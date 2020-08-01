---
title: MyBatis系列之--自定义TypeHandler处理枚举
date: 2020-07-25 14:05:00
updated: 2020-07-25 14:05:00
tags: MyBatis
categories: MyBatis
keywords: MyBatis
type: 
description: MyBatis如何自定义TypeHandler来处理枚举？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# TypeHandler简介
在MyBatis中，StatementHandler负责对需要执行的SQL语句进行预编译处理，主要完成以下两项工作：

- 调用参数处理器（ParameterHandler）来设置需要传入SQL的参数；
- 调用结果集处理器（ResultSetHandler）来处理查询到的结果数据；

而不管要完成其中的哪一项工作都需要使用类型处理器（TypeHandler）来进行数据类型处理，无论是ParameterHandler为预处理语句（PreparedStatement）设置一个参数，还是ResultSetHandler将结果集中的一条记录转换为合适的Java类型。在整个过程中，TypeHandler负责完成数据库类型与JavaBean类型的转换工作。  

# 自定义枚举类型处理器
以枚举类型的处理器为例，MyBatis提供了EnumTypeHandler和EnumOrdinalTypeHandler两个处理器来处理枚举类型。这两个处理器之间的区别在于EnumTypeHandler是将枚举的name值存到数据库，而EnumOrdinalTypeHandler会将枚举的序号值（>=0）存入数据库。

当然，我们还可以自己写一个TypeHandler来控制在设置参数或者取出结果集时应该如何对参数类型进行转换或封装。

自定义一个TypeHandler需要以下几个步骤：

1. 实现TypeHandler接口或者继承BaseTypeHandler。
2. 使用@MappedTypes(value = {})定义需要由该处理器处理的java类型。
  使用@MappedJdbcTypes(value = {})定义所对应的jdbcType类型。
3. 在自定义结果集标签或者参数处理的时候声明使用自定义TypeHandler进行处理或者在全局配置TypeHandler要处理的javaType。

以下是自定义的枚举类型处理器的源码。

## EnumTypeHandler类
```Java
	package org.mybatis.typehandler;
	 
	import java.sql.CallableStatement;
	import java.sql.PreparedStatement;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	 
	import org.apache.ibatis.type.BaseTypeHandler;
	import org.apache.ibatis.type.JdbcType;
	 
	/**
	 * mapper里字段到枚举类的映射。
	 * 用法一: 
	 * 	保存到数据库：#{enumDataField,typeHandler=com.mybatis.typehandler.EnumTypeHandler} 
	 *  从数据库查出：
	 *   <resultMap> <result property="enumDataField" column="enum_data_field"
	 *   javaType="com.xxx.MyEnum"
	 *   typeHandler="com.mybatis.typehandler.EnumTypeHandler"/>
	 *   </resultMap>
	 *
	 * 用法二： 
	 *  1）在mybatis-config.xml中指定handler: 
	 *   <typeHandlers> 
	 *   <typeHandler handler="com.mybatis.typehandler.EnumTypeHandler" javaType="com.xxx.MyEnum"/> 
	 *   </typeHandlers>
	 *  2)在MyClassMapper.xml里直接select/update/insert。
	 */
	 
	public class EnumTypeHandler<E extends Enum<?> & CodeBaseEnum> extends
			BaseTypeHandler<CodeBaseEnum> {
		private Class<E> clazz;
	 
		public EnumTypeHandler(Class<E> enumType) {
			if (enumType == null)
				throw new IllegalArgumentException("Type argument cannot be null");
	 
			this.clazz = enumType;
		}
	 
		@Override
		public void setNonNullParameter(PreparedStatement ps, int i,
				CodeBaseEnum parameter, JdbcType jdbcType) throws SQLException {
			ps.setInt(i, parameter.code());
		}
	 
		@Override
		public E getNullableResult(ResultSet rs, String columnName)
				throws SQLException {
			return CodeEnumUtil.codeOf(clazz, rs.getInt(columnName));
		}
	 
		@Override
		public E getNullableResult(ResultSet rs, int columnIndex)
				throws SQLException {
			return CodeEnumUtil.codeOf(clazz, rs.getInt(columnIndex));
		}
	 
		@Override
		public E getNullableResult(CallableStatement cs, int columnIndex)
				throws SQLException {
			return CodeEnumUtil.codeOf(clazz, cs.getInt(columnIndex));
		}
	}
```

## CodeBaseEnum类 
```Java
	package org.mybatis.typehandler;
	 
	public interface CodeBaseEnum {
		int code();
	}
```

## CodeEnumUtil类 
```Java
	package org.mybatis.typehandler;
	 
	public class CodeEnumUtil {
		/**
		 * @param enumClass
		 * @param code
		 * @param <E>
		 * @return
		 */
		public static <E extends Enum<?> & CodeBaseEnum> E codeOf(
				Class<E> enumClass, int code) {
			E[] enumConstants = enumClass.getEnumConstants();
			for (E e : enumConstants) {
				if (e.code() == code)
					return e;
			}
			return null;
		}
	}
```

## 测试Gender类 
```Java
	package org.mybatis.entity;
	 
	import org.mybatis.typehandler.CodeBaseEnum;
	 
	public enum Gender implements CodeBaseEnum {
	 
		MALE(0, "男"), FEMALE(1, "女");
		private Integer code;
		private String msg;
	 
		Gender(Integer code, String msg) {
			this.code = code;
			this.msg = msg;
		}
	 
		public String getMsg() {
			return msg;
		}
	 
		public void setMsg(String msg) {
			this.msg = msg;
		}
	 
		@Override
		public int code() {
			return code;
		}
	 
	}
```