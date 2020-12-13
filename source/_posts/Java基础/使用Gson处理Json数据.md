---
title: Java知识点系列之--使用Gson处理Json数据
date: 2020-07-20 13:01:00
updated: 2020-07-20 13:01:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 如何使用HttpClient发送GET和POST请求？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
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
# Gson简介

Gson（又称Google Json）是Google公司发布的一个开放源代码的Java库，用于序列化Java对象为JSON字符串，或反序列化JSON字符串成Java对象。

目前，在Java中处理JSON数据的类库中有以下三个比较流行：`FastJSON`、`Gson`和`Jackson`。与FastJSON和Jackson相比，Gson虽然在性能方面略显不足，但在功能上，它无疑是三者之中功能最强大的（使用Gson我们可以更加灵活地配置对象的哪些字段需要序列化）。  

# Gson的用法

Gson.jar的MVN下载地址：<http://mvnrepository.com/artifact/com.google.code.gson/gson>

在Gson类库中有一个Gson类，这个Gson类提供了两个方法：`toJson()` 和 `fromJson()`，我们主要就是调用这两个方法来分别实现Java对象的序列化和JSON字符串的反序列化。

下面我们通过几个简单的代码示例来对Gson类的常用方法做一个简单介绍。

首先，创建两个JavaBean类：Student和Teacher，来充当需要被转换的Java对象。 
```Java
	import java.io.Serializable;
	import java.util.Date;
	 
	import com.google.gson.annotations.Expose;
	import com.google.gson.annotations.SerializedName;
	import com.google.gson.annotations.Since;
	import com.google.gson.annotations.Until;
	 
	public class Student implements Serializable {
	 
		private static final long serialVersionUID = 6L;
	 
		@SerializedName("number")
		@Expose(serialize = true, deserialize = true)
		private int id;
		@Expose(serialize = true, deserialize = true)
		private String name;
		@Expose(serialize = true, deserialize = true)
		private int age;
		@Expose(serialize = true, deserialize = true)
		private Gender gender;
		@Until(1.1)
		private Date enrollmentDate;
		@Since(1.8)
		private Date graduationDate;
	 
		public Student(int id, String name, int age) {
			super();
			this.id = id;
			this.name = name;
			this.age = age;
		}
	 
		public int getId() {
			return id;
		}
	 
		public void setId(int id) {
			this.id = id;
		}
	 
		public String getName() {
			return name;
		}
	 
		public void setName(String name) {
			this.name = name;
		}
	 
		public int getAge() {
			return age;
		}
	 
		public void setAge(int age) {
			this.age = age;
		}
	 
		public Gender getGender() {
			return gender;
		}
	 
		public Date getEnrollmentDate() {
			return enrollmentDate;
		}
	 
		public void setEnrollmentDate(Date enrollmentDate) {
			this.enrollmentDate = enrollmentDate;
		}
	 
		public Date getGraduationDate() {
			return graduationDate;
		}
	 
		public void setGraduationDate(Date graduationDate) {
			this.graduationDate = graduationDate;
		}
	 
		public void setGender(Gender gender) {
			this.gender = gender;
		}
	 
		public String toString() {
			return "{number:" + this.id + ",name:" + this.name + ",age:" + this.age
					+ (null == this.gender ? "" : ",gender:"+this.gender+",") + "}";
		}
	}
```
<br/>
```Java
	import java.util.List;
	 
	import com.google.gson.annotations.SerializedName;
	 
	public class Teacher {
	 
		@SerializedName("teacherId")
		private int id;
	 
		private String name;
	 
		private List<Student> students;
	 
		public Teacher(int id, String name, List<Student> students) {
			super();
			this.id = id;
			this.name = name;
			this.students = students;
		}
	 
		public int getId() {
			return id;
		}
	 
		public void setId(int id) {
			this.id = id;
		}
	 
		public String getName() {
			return name;
		}
	 
		public void setName(String name) {
			this.name = name;
		}
	 
		public List<Student> getStudents() {
			return students;
		}
	 
		public void setStudents(List<Student> students) {
			this.students = students;
		}
	 
		public String toString() {
			return "{teacherId:" + this.id + ",name:" + this.name + ",students:" + this.students + "}";
		}
	 
	}
```
其中，Gender（性別）是一个枚举类，定义如下。  
```Java
	public enum Gender {
		MALE/* 男 */, FEMALE/* 女 */
	}
```
在使用Gson前需要先创建一个Gson对象，有以下两种创建方式：  
```Java
	private static final Gson gson1 = new Gson();                            //方式一
	private static final Gson gson2 = new GsonBuilder().create();            //方式二
```

**使用方式一**创建的Gson对象一般多用在那些不需要对序列化的字段进行详细配置的情况；

**使用方式二**创建的Gson对象能够根据字段的类型、字段名称、版本、注解等信息去决定哪些字段需要进行序列化和反序列化，同时还能实现许多其他的个性化配置功能，配置方法详见GsonTest。

以下是GsonTest测试类的完整代码。  
```Java
	import java.util.ArrayList;
	import java.util.Date;
	import java.util.HashMap;
	import java.util.LinkedHashMap;
	import java.util.List;
	import java.util.Map;
	import java.util.Map.Entry;
	 
	import org.junit.Test;
	 
	import com.google.gson.FieldNamingPolicy;
	import com.google.gson.Gson;
	import com.google.gson.GsonBuilder;
	import com.google.gson.reflect.TypeToken;
	 
	public class GsonTest {
		private static final Gson gson1 = new Gson();
		private static final Gson gson2 = new GsonBuilder()
				// 美化输出结果
				.setPrettyPrinting()
				// 设置日期格式
				.setDateFormat("yyyy年MM月dd日-HH时mm分ss秒")
				// 为Student类注册StudentTypeAdapter,定制序列化和反序列化规则
				.registerTypeAdapter(Student.class, new StudentTypeAdapter())
				// 设置字段的命名规则，对使用@SerializedName注解的字段无效
				.setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE)
				// 设置排除策略
				.setExclusionStrategies(new MyExclusionStrategy(Gender.class))
				// 仅序列化和反序列化带有@Expose注解的字段
				// 注解格式：@Expose (serialize = false, deserialize = false)
				// .excludeFieldsWithoutExposeAnnotation()
				// 序列化空值，默认空值不会序列化
				// .serializeNulls()
				// 开启版本控制，与@Until、@Since注解配合使用
				.setVersion(1.7)
				// 启用对Map中复杂对象的序列化支持
				.enableComplexMapKeySerialization().create();
		private static final Gson gson3 = new GsonBuilder()
				// 美化输出结果
				.setPrettyPrinting()
				// 设置日期格式
				.setDateFormat("yyyy年MM月dd日-HH时mm分ss秒").registerTypeAdapter(Gender.class, new GenderSerializer())
				// 仅序列化和反序列化带有@Expose注解的字段
				// 注解格式：@Expose (serialize = false, deserialize = false)
				.excludeFieldsWithoutExposeAnnotation().create();
	 
		@Test
		public void testGson1() {
			Student user1 = new Student(9527, "瓦力", 16);
			Student user2 = new Student(89757, "雪莉", 14);
	 
			// 将对象转换为json字符串
			String userStr = gson1.toJson(user1);
			System.out.println(userStr);
	 
			List<Student> userList = new ArrayList<Student>();
			userList.add(user1);
			userList.add(user2);
	 
			// 将对象集合转换为json字符串
			String listStr = gson1.toJson(userList);
			System.out.println(listStr);
	 
			Teacher teacher = new Teacher(200802, "潮歌", userList);
			// 将对象转换为json字符串
			String jsonStr = gson1.toJson(teacher);
			System.out.println(jsonStr);
	 
			// 将json字符串还原为对象
			Student user = gson1.fromJson(userStr, Student.class);
			System.out.println(user);
	 
			// 将json字符串还原为对象集合
			List<Student> users = gson1.fromJson(listStr, new TypeToken<List<Student>>() {
			}.getType());
			System.out.println(users);
	 
		}
	 
		@Test
		public void testGson2() {
			Student user1 = new Student(9527, "瓦力", 16);
			Student user2 = new Student(89757, "雪莉", 14);
	 
			user1.setEnrollmentDate(new Date());
			user1.setGraduationDate(new Date());
	 
			// 将对象转换为json字符串
			String userStr = gson2.toJson(user1);
			System.out.println(userStr);
	 
			List<Student> userList = new ArrayList<Student>();
			userList.add(user1);
			userList.add(user2);
	 
			// 将对象集合转换为json字符串
			String listStr = gson2.toJson(userList);
			System.out.println(listStr);
	 
			// 为保证对象序列化时的顺序，此处使用LinkedHashMap
			Map<String, Student> map = new LinkedHashMap<String, Student>();
			map.put(user1.getName(), user1);
			map.put(user2.getName(), user2);
	 
			// 将对象Map转换为json字符串
			String mapStr = gson2.toJson(map);
			System.out.println(mapStr);
	 
			Map<String, Student> retMap = gson2.fromJson(mapStr, new TypeToken<Map<String, Student>>() {
			}.getType());
			for (Entry<String, Student> entry : retMap.entrySet()) {
				System.out.println(entry.getKey() + ":" + entry.getValue());
			}
	 
		}
	 
		@Test
		public void testGson3() {
			Student user1 = new Student(9527, "瓦力", 16);
			Student user2 = new Student(89757, "雪莉", 14);
	 
			user1.setGender(Gender.MALE);
			user2.setGender(Gender.FEMALE);
	 
			// 将对象转换为json字符串
			String userStr = gson3.toJson(user1);
			System.out.println(userStr);
	 
			List<Student> userList = new ArrayList<Student>();
			userList.add(user1);
			userList.add(user2);
	 
			// 将对象集合转换为json字符串
			String listStr = gson3.toJson(userList);
			System.out.println(listStr);
	 
			Map<String, Student> map = new HashMap<String, Student>();
			map.put(user1.getName(), user1);
			map.put(user2.getName(), user2);
	 
			// 将对象Map转换为json字符串
			String mapStr = gson3.toJson(map);
			System.out.println(mapStr);
	 
			Map<String, Student> retMap = gson3.fromJson(mapStr, new TypeToken<Map<String, Student>>() {
			}.getType());
			for (Entry<String, Student> entry : retMap.entrySet()) {
				System.out.println(entry.getKey() + ":" + entry.getValue());
			}
		}
	}
```
其中**StudentTypeAdapter**用来自定义**Student**对象的序列化和反序列化过程。 
```Java
	import java.io.IOException;
	import java.util.regex.Matcher;
	import java.util.regex.Pattern;
	 
	import com.google.gson.TypeAdapter;
	import com.google.gson.stream.JsonReader;
	import com.google.gson.stream.JsonToken;
	import com.google.gson.stream.JsonWriter;
	 
	public class StudentTypeAdapter extends TypeAdapter<Student> {
	 
		private static Pattern pattern = Pattern.compile(":{1}([^\\{]*?)(,{1}|\\}{1})");
	 
		@Override
		public Student read(JsonReader reader) throws IOException {
			if (reader.peek() == JsonToken.NULL) {
				reader.nextNull();
				return null;
			}
			String jsonStr = reader.nextString();
			Matcher m = pattern.matcher(jsonStr);
			String[] arr = new String[3];
			int i = 0;
			while (m.find()) {
				arr[i++] = m.group(1);
			}
			Student student = new Student(Integer.parseInt(arr[0]), arr[1], Integer.parseInt(arr[2]));
			return student;
		}
	 
		@Override
		public void write(JsonWriter writer, Student student) throws IOException {
	 
			if (null == student) {
				writer.nullValue();
				return;
			}
	 
			String jsonStr = "{student:{number:" + student.getId() + ",name:" + student.getName() + ",age:"
					+ student.getAge() + "}}";
			writer.value(jsonStr);
		}
	 
	}
```
**MyExclusionStrategy**通过**shouldSkipClass()**和**shouldSkipField()**两个方法来指定某个类或者字段是否需要被序列化或者反序列化。  
```Java
	import com.google.gson.ExclusionStrategy;
	import com.google.gson.FieldAttributes;
	 
	public class MyExclusionStrategy implements ExclusionStrategy {
	 
		private Class<?> excludedClass;
	 
		public MyExclusionStrategy(Class<?> clazz) {
			this.excludedClass = clazz;
		}
	 
		@Override
		public boolean shouldSkipClass(Class<?> clazz) {
			return excludedClass.equals(clazz);
		}
	 
		@Override
		public boolean shouldSkipField(FieldAttributes f) {
			return f.getName().equalsIgnoreCase("age");
		}
	 
	}
```
**GenderSerializer**通过**GenderSerializer()**和**deserialize()**两个方法来对Gender枚举类型的序列化和反序列化过程进行设置。  
```Java
	import java.lang.reflect.Type;
	 
	import com.google.gson.JsonDeserializationContext;
	import com.google.gson.JsonDeserializer;
	import com.google.gson.JsonElement;
	import com.google.gson.JsonParseException;
	import com.google.gson.JsonPrimitive;
	import com.google.gson.JsonSerializationContext;
	import com.google.gson.JsonSerializer;
	 
	public class GenderSerializer implements JsonSerializer<Gender>, JsonDeserializer<Gender> {
	 
		@Override
		public JsonElement serialize(Gender gender, Type genderType, JsonSerializationContext context) {
			if (null == gender) {
				return new JsonPrimitive("");
			}
			return new JsonPrimitive(gender.equals(Gender.MALE) ? "男" : "女");
		}
	 
		@Override
		public Gender deserialize(JsonElement gender, Type genderType, JsonDeserializationContext context)
				throws JsonParseException {
			if ("男".equals(gender.getAsString())) {
				return Gender.MALE;
			} else if ("女".equals(gender.getAsString())) {
				return Gender.FEMALE;
			} else {
				return null;
			}
		}
	 
	}
```
testGson1()的执行结果：  
```
	{"number":9527,"name":"瓦力","age":16}
	[{"number":9527,"name":"瓦力","age":16},{"number":89757,"name":"雪莉","age":14}]
	{"teacherId":200802,"name":"潮歌","students":[{"number":9527,"name":"瓦力","age":16},{"number":89757,"name":"雪莉","age":14}]}
	{number:9527,name:瓦力,age:16}
	[{number:9527,name:瓦力,age:16}, {number:89757,name:雪莉,age:14}]
```
testGson2()的执行结果：  
```
	"{student:{number:9527,name:瓦力,age:16}}"
	[
	  "{student:{number:9527,name:瓦力,age:16}}",
	  "{student:{number:89757,name:雪莉,age:14}}"
	]
	{
	  "瓦力": "{student:{number:9527,name:瓦力,age:16}}",
	  "雪莉": "{student:{number:89757,name:雪莉,age:14}}"
	}
	瓦力:{number:9527,name:瓦力,age:16}
	雪莉:{number:89757,name:雪莉,age:14}
```
testGson3()的执行结果：  
```
	{
	  "number": 9527,
	  "name": "瓦力",
	  "age": 16,
	  "gender": "男"
	}
	[
	  {
	    "number": 9527,
	    "name": "瓦力",
	    "age": 16,
	    "gender": "男"
	  },
	  {
	    "number": 89757,
	    "name": "雪莉",
	    "age": 14,
	    "gender": "女"
	  }
	]
	{
	  "雪莉": {
	    "number": 89757,
	    "name": "雪莉",
	    "age": 14,
	    "gender": "女"
	  },
	  "瓦力": {
	    "number": 9527,
	    "name": "瓦力",
	    "age": 16,
	    "gender": "男"
	  }
	}
	雪莉:{number:89757,name:雪莉,age:14,gender:FEMALE,}
	瓦力:{number:9527,name:瓦力,age:16,gender:MALE,}
```
项目源码下载地址：<http://download.csdn.net/download/pengjunlee/9977536>