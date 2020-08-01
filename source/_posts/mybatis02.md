---
title: MyBatis系列之--SQL映射文件
date: 2020-07-25 14:02:00
updated: 2020-07-25 14:02:00
tags: MyBatis
categories: MyBatis
keywords: MyBatis
type: 
description: 认识MyBatis的SQL映射文件。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
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
在上一章《MyBatis系列之--HelloWorld》中我们搭建了一个MyBatis的HelloWorld项目，对如何在项目中使用MyBatis有了一个初步的认识。

这一章我们延续上一章，对如何编写Mapper配置文件进行简单介绍，代码中所用到的ClassEntity和StudentEntity两个实体类请参照上一章节。  

# MyBatis中的CRUD
在MyBatis中，对数据进行CRUD使用以下四个标签：insert、select、update、delete，对于增删改操作，MyBatis会将操作所影响的数据库记录条数封装为结果返回，可以返回如下类型：Long（long）、Integer（int）、Boolean（boolean）、Void（void）。

以对班级的CRUD为例：
```Java
	//新建班级
	public boolean insertClass(ClassEntity classEntity);
```
<br/>
```Xml
	<!-- useGeneratedKeys 使用主键自增策略 -->
	<!-- keyProperty 将获取到的自增主键值赋值给Java对象的classId属性 -->
	<!-- keyColumn  将获取到的自增主键值赋值给数据库的CLASS_ID字段 -->
	<insert id="insertClass" parameterType="org.mybatis.entity.ClassEntity"
		useGeneratedKeys="true" keyProperty="classId" keyColumn="CLASS_ID">
		insert into
		tbl_class(CLASS_NAME) values(#{className})
	</insert>
```
<br/>
```Java
	//查询班级
	public ClassEntity selectClassById(Long classId);
```
<br/>
```Xml
	<select id="selectClassById" parameterType="long"
		resultType="org.mybatis.entity.ClassEntity">
		select CLASS_ID
		classId,CLASS_NAME className from tbl_class
		where
		CLASS_ID=#{classId}
	</select>
```
<br/>
```Java
	//更新班级
	public int updateClass(ClassEntity classEntity);
```
<br/>
```Xml
	<update id="updateClass" parameterType="org.mybatis.entity.ClassEntity">
		update tbl_class set
		CLASS_NAME=#{className} where CLASS_ID=#{classId}
	</update>
```
<br/>
```Java
	//删除班级
	public long deleteClassById(Long classId);
```
<br/>
```Xml
	<delete id="deleteClassById" parameterType="long">
		delete from tbl_class where CLASS_ID=#{classId}
	</delete>
```

# MyBatis中的参数处理
## 单个基本类型参数
对于操作方法中传入单个基本类型参数的情况，例如上例中的查询班级方法，MyBatis不会做任何特殊处理，直接使用 `#{ 任意变量名 }` 均可获取到该参数的值。
 
## 多个基本类型参数
对于操作方法中传入多个基本类型参数的情况，例如下面这个根据班级Id和学生性别来统计学生人数的方法，MyBatis会自动将这些参数按照其传入的先后顺序封装到一个Map中，Map的key为param1...paramN。
```Java
	// 根据班级ID和学生的性别统计学生人数
	// @Param注解用来指定参数在Map中所对应的key
	public int selectCountByParams(@Param("classId") Long classId,
				@Param("studentGender") String studentGender);
```
<br/>
```Xml
	<select id="selectCountByParams" resultType="int">
			select
			count(STUDENT_ID) from tbl_student where CLASS_ID=#{classId} and
			STUDENT_GENDER=#{studentGender}
	</select>
```

这种情况，可以通过以下三种方式来获取参数值：
1.使用 `#{ paramN }` 来获取第N个参数的值；
2.使用 `#{ n }` 来获取索引为n的参数的值，此时 n 从 0 开始；
3.使用`@Param`注解来指定参数在Map中所对应的key，并使用 `#{ key }`来获取对应的值，如示例所示；

## 传入Map或POJO参数
在参数较多的情况下推荐将参数值放入Map或POJO中进行传参，例如之前的班级更新接口就传入了一个班级POJO对象，可以直接通过 `#{ 属性名 }`来获取对象的属性值。

如果传入的多个参数不是业务模型中的数据，没有与之对应的POJO，为了方便我们也可以将其封装到一个Map进行传参，使用 `#{ key }` 来进行取值，例如下面这个查询示例。
```Java
	// 传入map进行查询
	public List<StudentEntity> selectStudentByMap(Map<String, Object> map);
```
<br/>
```Xml
	<!-- 查询出一个班中指定性别的所有学生 -->
	<select id="selectStudentByMap" parameterType="map"
			resultType="org.mybatis.entity.StudentEntity">
			select STUDENT_ID
			studentId,STUDENT_NAME
			studentName,STUDENT_GENDER studentGender,
			STUDENT_BIRTHDAY
			studentBirthday,STUDENT_AGE studentAge,CLASS_ID
			classId from
			tbl_student where CLASS_ID =
			#{classId} and STUDENT_GENDER 
			=#{studentGender}
	</select>
```
如果多个参数不是业务模型中的数据，却要经常用到，推荐创建一个`TO`(Transfer Object)数据传输对象来传参，例如分页查询中的分页TO。

## 集合类参数
如果方法中传入了集合类型（List、Set）或者数组类型的参数，MyBatis也会做特殊处理，将传入的集合或者数组封装到Map中，集合类型参数的key为collection，数组类型的key为array。

此外List还可以使用 list 作为key来获取值，例如下面这个根据id集合来查询学生的接口。
```Java
	// 传入List进行查询
	public List<StudentEntity> selectStudentByList(List<Long> ids);
```
<br/>
```Xml
	<select id="selectStudentByList" parameterType="list"
			resultType="org.mybatis.entity.StudentEntity">
			select STUDENT_ID studentId,STUDENT_NAME studentName,STUDENT_GENDER
			studentGender,
			STUDENT_BIRTHDAY studentBirthday,STUDENT_AGE
			studentAge,CLASS_ID classId from
			tbl_student where STUDENT_ID in
			<foreach collection="list" index="index" item="item"
				open="(" close=")" separator=",">
				#{item}
			</foreach>
	</select>
```

# `#{}`与`${}`取值对比

`#{}`是以预编译的形式将参数设置到sql语句中，可以有效地防止sql注入，一般情况下我们都应当使用`#{}`来取值。

`${}`多用于在某些原生jdbc不支持占位符的地方取值（比如：表名、排序字段），例如以下映射语句。 
```Xml
	<select id="selectClass" parameterType="string"
		resultType="org.mybatis.entity.ClassEntity">
		select CLASS_ID
		classId,CLASS_NAME className from ${ tableName }
	</select>
```

# 返回List和Map类型
使用MyBatis返回List类型的结果非常容易，只需将select标签中的resultType属性的值设置为List中将要存放的元素的类型即可，MyBatis会自动将查询结果中的每一条记录都自动映射成这种类型的对象。 
```Java
	// 查询班级，返回ClassEntity的List集合
	public List<ClassEntity> selectClassReturnList();
```
<br/>
```Xml
	<select id="selectClassReturnList" resultType="org.mybatis.entity.ClassEntity">
		select
		CLASS_ID
		classId,CLASS_NAME className from tbl_class
	</select>
```
<br/>
```Java
	// 查询班级，返回Map的List集合
	public List<Map<String, Object>> selectClassReturnListOfMap();
```
<br/>
```Xml
	<select id="selectClassReturnListOfMap" resultType="map">
		select
		CLASS_ID
		classId,CLASS_NAME className from tbl_class
	</select>
```
如果想要把返回的对象集合封装到一个Map中，类似Map<Long,ClassEntity>这种类型，可以使用`@MapKey`注解来指定使用对象中的哪一个属性值来作为Map的key。  
```Java
	// 查询班级，返回Map
	@MapKey("classId")
	public Map<Long, ClassEntity> selectClassReturnMap();
```
<br/>
```Xml
	<select id="selectClassReturnMap" resultType="org.mybatis.entity.ClassEntity">
		select
		CLASS_ID
		classId,CLASS_NAME className from tbl_class
	</select>
```
<br/>
```Java
	// 按ID查询班级,返回Map
	public Map<String, Object> selectClassByIdReturnMap(Long classId);
```
<br/>
```Xml
	<select id="selectClassByIdReturnMap" parameterType="long"
		resultType="map">
		select CLASS_ID
		classId,CLASS_NAME className from tbl_class
		where
		CLASS_ID=#{classId}
	</select>
```