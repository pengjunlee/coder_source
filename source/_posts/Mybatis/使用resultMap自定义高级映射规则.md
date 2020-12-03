---
title: MyBatis系列之--使用resultMap自定义高级映射规则
date: 2020-07-25 14:03:00
updated: 2020-07-25 14:03:00
tags: MyBatis
categories: MyBatis
keywords: MyBatis
type: 
description: MyBatis如何使用resultMap自定义高级映射规则？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
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
# resultMap简介
在前面两篇文章中，我们都是通过使用`select`元素的`resultType`属性指定查询结果的返回值类型，来让MyBatis自动将查询结果集封装成我们希望的类型进行返回。

`resultType`属性非常有用，但在返回结果类型比较复杂的情况下却无能为力，为此，MyBatis在select元素中还为我们提供了另外一个resultMap属性，用于引用一个使用`<resultMap/>`元素定义好的自定义结果集映射规则。

`<resultMap/>`算是MyBatis中最重要、最强大的元素了，它可以让你比使用JDBC调用结果集省掉90%的代码，也可以让你做许多JDBC不支持的事。 事实上，编写类似的对复杂语句联合映射的代码也许会需要上千行。resultMap的设计就是简单语句不需要明确的结果映射，而很多复杂语句确实需要描述它们的关系。  

# resultMap的用法
`<resultMap/>`元素有很多子元素： 
```Xml
	<constructor>		 /*用来将查询结果作为参数注入到实例的构造方法中*/
		<idArg />	 /*标记结果作为 ID*/
	 
		<arg />		 /*标记结果作为普通参数*/
	 
	</constructor>
	<id/>			 /*一个ID结果，标记结果作为 ID*/
	<result/>		 /*一个普通结果，JavaBean的普通属性或字段*/
	<association>		 /*关联其他的对象*/
	</association>
	<collection>		 /*关联其他的对象集合*/
	</collection>
	<discriminator>	         /*鉴别器，根据结果值进行判断，决定如何映射*/
		<case></case>	 /*结果值的一种情况，将对应一种映射规则*/
	</discriminator>
```
在一个`<resultMap/>`元素中，这些子元素出现的先后顺序是有严格规定的，它们从前到后依次是：`constructor`-->`id` --> `result`--> `association`-->`collection` -->`discriminator`。  

## id & result
`<id/>`和`<result/>`是resultMap中最基本的映射内容，它们都可以将查询结果中一个单独列的值映射为返回结果对象中的简单数据类型(字符串,整型,双精度浮点数,日期等)的单独属性或字段。这两者之间的唯一不同是id 表示的结果将是当比较对象实例时用到的标识属性。这帮助来改进整体表现，特别是缓存和嵌入结果映射(也就是联合映射) 。 
```Xml
	<!-- 返回javaBean对象 -->
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_GENDER" property="studentGender" />
		<result column="STUDENT_BIRTHDAY" property="studentBirthday" />
		<result column="STUDENT_AGE" property="studentAge" />
	</resultMap>
	<!-- 返回Map对象 -->
	<resultMap type="map" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" javaType="long"/>
		<result column="STUDENT_NAME" property="studentName" javaType="string"/>
		<result column="STUDENT_GENDER" property="studentGender" javaType="string"/>
		<result column="STUDENT_BIRTHDAY" property="studentBirthday" javaType="date"/>
		<result column="STUDENT_AGE" property="studentAge" javaType="int"/>
	</resultMap>
```
`<id/>`和`<result/>`都支持以下几个属性： 
<div align=center>

![MyBatis图](http://pengjunlee.3vzhuji.net/static/mybatis/03.png "MyBatis示意图")
<div align=left>
为了未来的参考,MyBatis 通过包含的 jdbcType 枚举型,支持下面的 JDBC 类型。
<div align=center>

![MyBatis图](http://pengjunlee.3vzhuji.net/static/mybatis/04.png "MyBatis示意图")
<div align=left>

## constructor
使用`<constructor/>`元素可以在对象实例化时，将查询到结果集作为参数注入到实例的构造方法中，以实现对实例化对象的特殊处理。

例如，为了防止由于数据库学生表中的年龄字段长期未更新，而导致查询出来的学生年龄信息不准确，我们希望查询学生信息时根据学生的生日计算出学生的年龄，而不再直接从数据库中获取学生的年龄信息。

首先在学生实体类StudentEntity中增加如下构造方法：
```Java
	private static final SimpleDateFormat yearFormat = new SimpleDateFormat(
				"yyyy");
	public StudentEntity(Long studentId, Date studentBirthday) {
		super();
		this.studentId = studentId;
		if (null != studentBirthday) {
	        	this.studentBirthday = studentBirthday;
			int startYear = Integer
					.parseInt(yearFormat.format(studentBirthday));
			int nowYear = Integer.parseInt(yearFormat.format(new Date()));
			this.studentAge = nowYear - startYear;
		}
	}
```
相应的Mapper配置如下。
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<constructor>
			<idArg column="STUDENT_ID" javaType="java.lang.Long" />
			<arg column="STUDENT_BIRTHDAY" javaType="java.util.Date" />
		</constructor>
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_GENDER" property="studentGender" />
	</resultMap>
	<select id="selectStudentById" resultMap="studentResultMap">
		select * from
		tbl_student where
		STUDENT_ID =
		#{studentId}
	</select>
```
`<constructor/>`元素的`<idArg/>`和`<arg/>`除了支持示例中的`column`和`javaType`属性，还支持以下几个属性：`jdbcType`、`typeHandler`、`select`、`resultMap`、`name`，其中的name属性从3.4.3版本起开始引入，用来指定该在构造方法中形参的名字。
> <font color=red>注意</font>：使用`<constructor/>`元素在构造方法中初始化的参数若使用resultMap的其他子元素进行映射，则构造方法为对象实例赋予的属性值将被覆盖。

## association
`<association/>`元素用于处理查询结果中关联其他对象的情况，比如：一个学生属于一个班级，我们在查询一个学生的信息时，想要把他所在的班级也一并查出。
```Java
	public class StudentEntity {
	 
		private Long studentId;
	 
		private String studentName;
	 
		private ClassEntity studentClass;
		
		//此处省略get和set方法
	 
	}
```
MyBatis提供了两种不同的方式用来处理这种对象关联查询:嵌套查询和嵌套结果。

### 嵌套查询
所谓嵌套查询，指的是通过执行另外一个 SQL 映射语句来返回所关联的对象类型。
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<association column="CLASS_ID" property="studentClass"
			select="selectClassById" javaType="org.mybatis.entity.ClassEntity" />
	</resultMap>
	<select id="selectStudentById" resultMap="studentResultMap">
		select * from
		tbl_student where
		STUDENT_ID =
		#{studentId}
	</select>
	<resultMap type="org.mybatis.entity.ClassEntity" id="classResultMap">
		<id column="CLASS_ID" property="classId" />
		<result column="CLASS_NAME" property="className" />
	</resultMap>
	<select id="selectClassById" resultMap="classResultMap">
		select * from
		tbl_class
		where
		CLASS_ID =
		#{classId}
	</select>
```
> 注：`<association/>`元素可以通过columnPrefix属性可以祛除查询结果内列名的指定前缀。

### 嵌套结果
所谓嵌套结果，指的是通过联表查询将所需的所有字段内容先查询出来，再通过级联属性映射来创建复杂类型的结果对象。
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="CLASS_ID" property="studentClass.classId" />
		<result column="CLASS_NAME" property="studentClass.className" />
	</resultMap>
	<sql id="studentAndClassFields">
		s.STUDENT_ID AS STUDENT_ID,
		s.STUDENT_NAME AS STUDENT_NAME,
		c.CLASS_ID AS CLASS_ID,
		c.CLASS_NAME AS CLASS_NAME
	</sql>
	<select id="selectStudentById" resultMap="studentResultMap">
		SELECT
		<include refid="studentAndClassFields"></include>
		FROM
		tbl_student s
		LEFT OUTER JOIN
		tbl_class c ON s.CLASS_ID=c.CLASS_ID
		WHERE
		s.STUDENT_ID=#{studentId}
	</select>
```
此时的resultMap还可以写为如下两种形式。
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<association property="studentClass" javaType="org.mybatis.entity.ClassEntity">
			<id column="CLASS_ID" property="classId" />
			<result column="CLASS_NAME" property="className" />
		</association>
	</resultMap>
```
<br/>
```
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<association property="studentClass" resultMap="classResultMap">
		</association>
	</resultMap>
	<resultMap type="org.mybatis.entity.ClassEntity" id="classResultMap">
		<id column="CLASS_ID" property="classId" />
		<result column="CLASS_NAME" property="className" />
	</resultMap>
```

## collection
<collection/>元素用于处理查询结果中关联其他对象集合的情况，比如：一个班级包含多个学生，我们在查询一个班级的信息时，想要把这个班级里的学生集合也一并查出。
```Java
	public class ClassEntity {
	 
		private Long classId;
	 
		private String className;
	 
		private List<StudentEntity> students;
	 
		//此处省略get和set方法
	}
```
### 嵌套查询
```Xml
	<resultMap type="org.mybatis.entity.ClassEntity" id="classResultMap">
		<id column="CLASS_ID" property="classId" />
		<result column="CLASS_NAME" property="className" />
		<collection column="CLASS_ID" property="students" javaType="list"
			select="listStudentsByClass">
		</collection>
	</resultMap>
 
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
	</resultMap>
 
	<select id="selectClassById" resultMap="classResultMap">
		select * from
		tbl_class
		where
		CLASS_ID =
		#{classId}
	</select>
 
	<select id="listStudentsByClass" parameterType="long"
		resultMap="studentResultMap">
		select * from
		tbl_student
		where
		CLASS_ID =
		#{classId}
	</select>
```

### 嵌套结果
```Xml
	<resultMap type="org.mybatis.entity.ClassEntity" id="classResultMap">
		<id column="CLASS_ID" property="classId" />
		<result column="CLASS_NAME" property="className" />
		<collection column="CLASS_ID" property="students" javaType="list"
			ofType="org.mybatis.entity.StudentEntity">
			<id column="STUDENT_ID" property="studentId" />
			<result column="STUDENT_NAME" property="studentName" />
		</collection>
	</resultMap>
	<sql id="classAndStudentsFields">
		c.CLASS_ID AS CLASS_ID,
		c.CLASS_NAME AS CLASS_NAME,
		s.STUDENT_ID AS STUDENT_ID,
		s.STUDENT_NAME AS STUDENT_NAME
	</sql>
	<select id="selectClassById" parameterType="long" resultMap="classResultMap">
		SELECT
		<include refid="classAndStudentsFields"></include>
		FROM tbl_class c
		LEFT JOIN tbl_student s ON c.CLASS_ID=s.CLASS_ID
		WHERE
		c.CLASS_ID=#{classId}
	</select>
```
此时的resultMap也可以写为如下形式。
```Xml
	<resultMap type="org.mybatis.entity.ClassEntity" id="classResultMap">
		<id column="CLASS_ID" property="classId" />
		<result column="CLASS_NAME" property="className" />
		<collection property="students" javaType="list"
			ofType="org.mybatis.entity.StudentEntity" resultMap="studentResultMap">
		</collection>
		</resultMap>
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
	</resultMap>
```

## discriminator
<discriminator/>元素很像Java语言中的switch 语句，允许用户根据查询结果中指定字段的不同取值来执行不同的映射规则，比如：在查询一个班级的学生信息时，如果学生是男生则将其年龄信息一并查出，如果学生是女生则不查询其年龄信息而是将班级信息一并查出。
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentSResultMap">
		<discriminator column="STUDENT_GENDER" javaType="string">
			<case value="男" resultMap="maleResultMap"/>
			<case value="女" resultMap="femaleResultMap"/>
		</discriminator>
	</resultMap>
	<resultMap type="org.mybatis.entity.StudentEntity" id="maleResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<result column="STUDENT_AGE" property="studentAge" />
	</resultMap>
	<resultMap type="org.mybatis.entity.StudentEntity" id="femaleResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<association column="CLASS_ID" property="studentClass"
			javaType="org.mybatis.entity.ClassEntity">
			<id column="CLASS_ID" property="classId" />
			<result column="CLASS_NAME" property="className" />
		</association>
	</resultMap>
```
此时的resultMap也可以写为如下形式。
```Xml
	<resultMap type="org.mybatis.entity.StudentEntity" id="studentSResultMap">
		<id column="STUDENT_ID" property="studentId" />
		<result column="STUDENT_NAME" property="studentName" />
		<discriminator column="STUDENT_GENDER" javaType="string">
			<case value="男" resultType="org.mybatis.entity.StudentEntity">
				<result column="STUDENT_AGE" property="studentAge" />
			</case>
			<case value="女" resultType="org.mybatis.entity.StudentEntity">
				<association column="CLASS_ID" property="studentClass"
					javaType="org.mybatis.entity.ClassEntity">
					<id column="CLASS_ID" property="classId" />
					<result column="CLASS_NAME" property="className" />
				</association>
			</case>
		</discriminator>
	</resultMap>
```
> 注：在使用`<resultMap/>`元素进行分步查询时，如果需要传递多个参数到关联的查询语句中，可以使用column="{classId=CLASS_ID}"将参数封装到Map进行传参，接收参数的查询语句需将parameterType设置为Map类型。