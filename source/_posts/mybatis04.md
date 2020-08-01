---
title: MyBatis系列之--动态SQL
date: 2020-07-25 14:04:00
updated: 2020-07-25 14:04:00
tags: MyBatis
categories: MyBatis
keywords: MyBatis
type: 
description: MyBatis如何编写动态SQL？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
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
# 动态SQL简介
MyBatis 的强大特性之一便是它的动态 SQL。如果你有使用 JDBC 或其他类似框架的经验，你就能体会到根据不同条件拼接 SQL 语句有多么痛苦。拼接的时候要确保不能忘了必要的空格，还要注意省掉列名列表最后的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

MyBatis常用的动态 SQL 元素有如下几个：

- if
- choose (when, otherwise)
- trim (where, set)
- foreach 

# if

`<if/>`元素根据传入的参数是否满足指定的条件来决定其内部所包含的子SQL语句是否生效，当它的test属性值判定结果为true时，其内部所包含的子SQL语句将会生效。

例如下面这个SQL，当传入的studentName属性不为空时会增加学生姓名的筛选条件，当studentGender属性不为空且值为“男”或“女”时会增加学生性别的筛选条件。 
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_GENDER" property="studentGender" />
		<result column="STUDENT_AGE" property="studentAge" />
	</resultMap>
	 
	<select id="selectStudentsByCond" parameterType="org.mybatis.entity.StudentEntity"
		resultMap="studentResultMap">
		select * from tbl_student
		<where>
			<if test="studentName!=null and studentName!=""">
				and STUDENT_NAME like #{studentName}
			</if>
			<if
				test="studentGender!=null &&studentGender in ("女","男")">
				and STUDENT_GENDER = #{studentGender}
			</if>
		</where>
	</select>
```

# choose, when, otherwise
`<choose/>`元素与`<if/>`元素类似，同样是根据传入的参数是否满足指定的条件来决定其内部所包含的子SQL语句是否生效，但它又与`<if/>`元素有所不同，`<choose/>`元素中使用`<when/>`和`<otherwise/>`元素声明的多个子条件语句有且只能有一个生效。

例如下面这个SQL，当传入的studentId属性不为空且其值大于0时将根据学生ID进行查询，若条件不满足，则会再去判断传入的studentName属性是否为空，若studentName属性不为空则会根据学生姓名进行查询，若两次判断条件都不满足，则查询全部。 
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_GENDER" property="studentGender" />
		<result column="STUDENT_AGE" property="studentAge" />
	</resultMap>
	 
	<select id="selectStudentsByCond" parameterType="org.mybatis.entity.StudentEntity"
		resultMap="studentResultMap">
		select * from tbl_student
		<where>
			<choose>
				<when test="studentId!=null and studentId > 0">
					STUDENT_ID = #{studentId}
				</when>
				<when test="studentName!=null and studentName!=""">
					STUDENT_NAME like #{studentName}
				</when>
				<otherwise>
					1=1
				</otherwise>
			</choose>
		</where>
	</select>
```

# trim, where, set
`<where/>`元素在有查询条件时能够自动帮我们插入“WHERE”子句，并且能够自动剔除里面多余的“AND”或“OR”前缀，例如之前介绍`<if/>`元素时的示例SQL。但`<where/>`元素只能够帮我们剔除掉查询条件中多余的前缀“AND”或“OR”，如果要剔除后缀就需要使用`<trim/>`元素了。

`<trim/>`元素通过配置prefix、prefixOverrides、suffix、suffixOverrides四个属性能够更加灵活的定制SQL语句。
使用`<trim/>`元素改写之前介绍`<if/>`元素时使用的示例SQL内容如下。 
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_GENDER" property="studentGender" />
		<result column="STUDENT_AGE" property="studentAge" />
	</resultMap>
	 
	<select id="selectStudentsByCond" parameterType="org.mybatis.entity.StudentEntity"
		resultMap="studentResultMap">
		select * from tbl_student
		<trim prefix="when" prefixOverrides="and | or">
			<if test="studentName!=null and studentName!=""">
				and STUDENT_NAME like #{studentName}
			</if>
			<if
				test="studentGender!=null &&studentGender in ("女","男")">
				and STUDENT_GENDER = #{studentGender}
			</if>
		</trim>
	</select>
```
`<set/>`元素可以被用于动态包含需要更新的列，它会动态添加前置SET关键字，同时也会剔除多余的逗号。  
```Xml
	<update id="updateStudent" parameterType="org.mybatis.entity.StudentEntity">
		update tbl_student
		<set>
			<if test="studentName!=null and studentName!=""">
				STUDENT_NAME = #{studentName},
			</if>
			<if test="studentAge!=null and studentAge > 0">
				STUDENT_AGE = #{studentAge},
			</if>
			<if
				test="studentGender!=null &&studentGender in ("女","男")">
				STUDENT_GENDER = #{studentGender},
			</if>
		</set>
		<where>
			STUDENT_ID=#{studentId}
		</where>
	</update>
```
如果将以上SQL使用`<trim/>`元素来实现，写法如下。  
```Xml
	<update id="updateStudent" parameterType="org.mybatis.entity.StudentEntity">
		update tbl_student
		<trim prefix="set" suffixOverrides=",">
			<if test="studentName!=null and studentName!=""">
				STUDENT_NAME = #{studentName},
			</if>
			<if test="studentAge!=null and studentAge > 0">
				STUDENT_AGE = #{studentAge},
			</if>
			<if
				test="studentGender!=null &&studentGender in ("女","男")">
				STUDENT_GENDER = #{studentGender},
			</if>
		</trim>
		<where>
			STUDENT_ID=#{studentId}
		</where>
	</update>
```

# foreach
`<foreach/>`元素用于对集合类型的参数进行遍历，它允许你指定开闭匹配的字符串以及在迭代中间放置分隔符，这个元素是很智能的，因此它不会偶然地附加多余的分隔符。

例如以下根据学生的ID列表查询学生的SQL。 
```Java
	/*
	 * 根据id列表查询学生
	 */
	public List<StudentEntity> selectStudentsByIds(List<Long> ids);
```
<br/>
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_GENDER" property="studentGender" />
		<result column="STUDENT_AGE" property="studentAge" />
	</resultMap>
	<select id="selectStudentsByIds" parameterType="list"
		resultMap="studentResultMap">
		select * from tbl_student
		<where>
			STUDENT_ID in
		</where>
		<foreach collection="collection" index="index" item="id" open="("
			separator="," close=")">
			#{id}
		</foreach>
	</select>
```

> 注意：你可以将任何可迭代对象（如列表、集合等）和任何的字典或者数组对象传递给`<foreach/>`作为集合参数。当使用可迭代对象或者数组时，index是当前迭代的次数，item的值是本次迭代获取的元素。当使用字典（或者Map.Entry对象的集合）时，index是键，item是值。  

# _parameter、_databaseId
_parameter和_databaseId是MyBatis为我们提供的两个内置参数，_parameter代表了该方法的整个参数，_databaseId代表了在databaseIdProvider标签中为数据库所起的别名。 
```Xml
	<databaseIdProvider type="DB_VENDOR">
		<property name="MySQL" value="mysql" />
		<property name="Oracle" value="oracle" />
		<property name="SQL Server" value="sqlserver" />
	</databaseIdProvider>
```

# bind
`<bind/>`元素可以从 OGNL 表达式中创建一个变量并将其绑定到上下文。比如： 
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_GENDER" property="studentGender" />
		<result column="STUDENT_AGE" property="studentAge" />
	</resultMap>
	 
	<select id="selectStudentsByCond" parameterType="org.mybatis.entity.StudentEntity"
		resultMap="studentResultMap">
		<bind name="likeName" value="'%'+_parameter.studentName+'%'" />
		select * from tbl_student
		<trim prefix="where" prefixOverrides="and | or">
			<if test="studentName!=null and studentName!=""">
				and STUDENT_NAME like #{likeName}
			</if>
			<if
				test="studentGender!=null &&studentGender in ("女","男")">
				and STUDENT_GENDER = #{studentGender}
			</if>
		</trim>
	</select>
```

# sql、include
`<sql/>`元素用来提取可重用的SQL片段，方便后面引用，`<include/>`元素用来引入已经定义好的`<sql/>`元素；`<include/>`元素还可以自定义一些property供引用的`<sql/>`元素使用。 
```Xml
	<sql id="studentFields">
		STUDENT_ID AS studentId,
		STUDENT_NAME AS studentName,
		STUDENT_GENDER AS studentGender,
		${studentAge} AS studentAge
	</sql>
	<select id="selectStudentsByCond" parameterType="org.mybatis.entity.StudentEntity"
		resultType="org.mybatis.entity.StudentEntity">
		<bind name="likeName" value="'%'+_parameter.studentName+'%'" />
		select
		<include refid="studentFields">
			<property name="studentAge" value="STUDENT_AGE" />
		</include>
		from tbl_student
		<trim prefix="where" prefixOverrides="and | or">
			<if test="studentName!=null and studentName!=""">
				and STUDENT_NAME like #{likeName}
			</if>
			<if
				test="studentGender!=null &&studentGender in ("女","男")">
				and STUDENT_GENDER = #{studentGender}
			</if>
		</trim>
	</select>
```