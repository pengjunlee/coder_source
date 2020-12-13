---
title: Java知识点系列之--元注解详解
date: 2020-07-20 13:19:00
updated: 2020-07-20 13:19:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 一篇关于带你深入理解Java中的元注解。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img19.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img19.jpg
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
# Annotation(注解)
从JDK 1.5开始, Java增加了对元数据(MetaData)的支持，也就是 Annotation(注解)。 

注解其实就是代码里的特殊标记，它用于替代配置文件：传统方式通过配置文件告诉类如何运行，有了注解技术后，开发人员可以通过注解告诉类如何运行。在Java技术里注解的典型应用是：可以通过反射技术去得到类里面的注解，以决定怎么去运行类。 

注解可以标记在包、类、属性、方法，方法参数以及局部变量上，且同一个地方可以同时标记多个注解。 
```Java
	// 抑制编译期的未指定泛型、未使用和过时警告
	@SuppressWarnings({ "rawtypes", "unused", "deprecation" })
	// 重写
	@Override
	meta-annotation（元注解）
```
除了直接使用JDK 定义好的注解，我们还可以自定义注解，在JDK 1.5中提供了4个标准的用来对注解类型进行注解的注解类，我们称之为 meta-annotation（元注解），他们分别是：

- @Target
- @Retention
- @Documented
- @Inherited

我们可以使用这4个元注解来对我们自定义的注解类型进行注解，接下来，我们挨个对这4个元注解的作用进行介绍。  

# @Target注解
Target注解的作用是：`描述注解的使用范围（即：被修饰的注解可以用在什么地方）` 。

Target注解用来说明那些被它所注解的注解类可修饰的对象范围：注解可以用于修饰 packages、types（类、接口、枚举、注解类）、类成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数），在定义注解类时使用了@Target 能够更加清晰的知道它能够被用来修饰哪些对象，它的取值范围定义在ElementType 枚举中。  
```Java
	public enum ElementType {
	 
	    TYPE, // 类、接口、枚举类
	 
	    FIELD, // 成员变量（包括：枚举常量）
	 
	    METHOD, // 成员方法
	 
	    PARAMETER, // 方法参数
	 
	    CONSTRUCTOR, // 构造方法
	 
	    LOCAL_VARIABLE, // 局部变量
	 
	    ANNOTATION_TYPE, // 注解类
	 
	    PACKAGE, // 可用于修饰：包
	 
	    TYPE_PARAMETER, // 类型参数，JDK 1.8 新增
	 
	    TYPE_USE // 使用类型的任何地方，JDK 1.8 新增
	 
	}
```

# @Retention注解
Reteniton注解的作用是：`描述注解保留的时间范围（即：被描述的注解在它所修饰的类中可以被保留到何时）` 。

Reteniton注解用来限定那些被它所注解的注解类在注解到其他类上以后，可被保留到何时，一共有三种策略，定义在RetentionPolicy枚举中。
```Java
	public enum RetentionPolicy {
	 
	    SOURCE,    // 源文件保留
	    CLASS,       // 编译期保留，默认值
	    RUNTIME   // 运行期保留，可通过反射去获取注解信息
	}
```
为了验证应用了这三种策略的注解类有何区别，分别使用三种策略各定义一个注解类做测试。  
```Java
	@Retention(RetentionPolicy.SOURCE)
	public @interface SourcePolicy {
	 
	}
	@Retention(RetentionPolicy.CLASS)
	public @interface ClassPolicy {
	 
	}
	@Retention(RetentionPolicy.RUNTIME)
	public @interface RuntimePolicy {
	 
	}
```
用定义好的三个注解类分别去注解一个方法。  
```Java
	public class RetentionTest {
	 
		@SourcePolicy
		public void sourcePolicy() {
		}
	 
		@ClassPolicy
		public void classPolicy() {
		}
	 
		@RuntimePolicy
		public void runtimePolicy() {
		}
	}
```
<div align=center>

![元注解示意图](http://pengjunlee.3vzhuji.net/static/javacore/25.png "元注解示意图")
<div align=left>

如图所示，通过执行 `javap -verbose RetentionTest` 命令获取到的RetentionTest 的 class 字节码内容如下。 
```Java
	{
	  public retention.RetentionTest();
	    flags: ACC_PUBLIC
	    Code:
	      stack=1, locals=1, args_size=1
	         0: aload_0
	         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
	         4: return
	      LineNumberTable:
	        line 3: 0
	
	  public void sourcePolicy();
	    flags: ACC_PUBLIC
	    Code:
	      stack=0, locals=1, args_size=1
	         0: return
	      LineNumberTable:
	        line 7: 0
	
	  public void classPolicy();
	    flags: ACC_PUBLIC
	    Code:
	      stack=0, locals=1, args_size=1
	         0: return
	      LineNumberTable:
	        line 11: 0
	    RuntimeInvisibleAnnotations:
	      0: #11()
	
	  public void runtimePolicy();
	    flags: ACC_PUBLIC
	    Code:
	      stack=0, locals=1, args_size=1
	         0: return
	      LineNumberTable:
	        line 15: 0
	    RuntimeVisibleAnnotations:
	      0: #14()
	}
```
从 RetentionTest 的字节码内容我们可以得出以下两点结论：

1. 编译器并没有记录下 sourcePolicy() 方法的注解信息； 
2. 编译器分别使用了 RuntimeInvisibleAnnotations 和 RuntimeVisibleAnnotations 属性去记录了classPolicy()方法 和 runtimePolicy()方法 的注解信息；  

# @Documented注解
Documented注解的作用是：`描述在使用 javadoc 工具为类生成帮助文档时是否要保留其注解信息`。

为了验证Documented注解的作用到底是什么，我们创建一个带有 @Documented 的自定义注解类。 
```Java
	import java.lang.annotation.Documented;
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Target;
	 
	@Documented
	@Target({ElementType.TYPE,ElementType.METHOD})
	public @interface MyDocumentedtAnnotation {
	 
		public String value() default "这是@Documented注解为文档添加的注释";
	}
```
再创建一个 MyDocumentedTest 类。  
```Java
	@MyDocumentedtAnnotation
	public class MyDocumentedTest {
	 
		@Override
		@MyDocumentedtAnnotation
		public String toString() {
			return this.toString();
		}
	}
```
接下来，使用以下命令为 MyDocumentedTest 类生成帮助文档。  

<div align=center>

![元注解示意图](http://pengjunlee.3vzhuji.net/static/javacore/26.png "元注解示意图")
<div align=left>

命令执行完成之后，会在当前目录下生成一个 doc 文件夹，其内包含以下文件。 

<div align=center>

![元注解示意图](http://pengjunlee.3vzhuji.net/static/javacore/27.png "元注解示意图")
<div align=left>

查看 index.html 帮助文档，可以发现在类和方法上都保留了 MyDocumentedtAnnotation 注解信息。 

<div align=center>

![元注解示意图](http://pengjunlee.3vzhuji.net/static/javacore/28.png "元注解示意图")
<div align=left>

修改 MyDocumentedtAnnotation 注解类，去掉上面的 @Documented 注解。 
```Java
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Target;
	 
	@Target({ElementType.TYPE,ElementType.METHOD})
	public @interface MyDocumentedtAnnotation {
	 
		public String value() default "这是@Documented注解为文档添加的注释";
	}
```
重新生成帮助文档，此时类和方法上的 MyDocumentedtAnnotation 注解信息都不见了。 

<div align=center>

![元注解示意图](http://pengjunlee.3vzhuji.net/static/javacore/29.png "元注解示意图")
<div align=left>

# @Inherited注解
Inherited注解的作用是：`使被它修饰的注解具有继承性（如果某个类使用了被@Inherited修饰的注解，则其子类将自动具有该注解）`。

接下来我们使用代码来进行测试，首先创建一个被@Inherited修饰的注解类MyInheritedAnnotation。 
```Java
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Inherited;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	 
	@Inherited
	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface MyInheritedAnnotation {
	 
		public String name() default "pengjunlee";
	}
```
创建一个带有 MyInheritedAnnotation 注解的父类和一个无任何注解的子类。  
```Java
	@MyInheritedAnnotation(name="parent")
	public class Parent {
	 
	}
	public class Child extends Parent{
	 
		public static void main(String[] args) {
			Class<Child> child=Child.class;
			MyInheritedAnnotation annotation = child.getAnnotation(MyInheritedAnnotation.class);
			System.out.println(annotation.name());
		}
	}
```
运行程序，打印结果如下：  
```
parent
```

# 注解应用举例

首先自定义一个注解类。 
```Java
	package com.pengjunlee;
	 
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Inherited;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	 
	@Target({ ElementType.TYPE, ElementType.METHOD })
	@Retention(RetentionPolicy.RUNTIME)
	@Inherited
	public @interface MyAnnotation {
	 
		public String name() default "pengjunlee";
	}
```
在 AnnotationTest 中使用反射获取注解信息。  
```Java
	package com.pengjunlee;
	 
	import java.lang.annotation.Annotation;
	import java.lang.reflect.Method;
	 
	@MyAnnotation(name = "name of type")
	public class AnnotationTest {
	 
		@MyAnnotation(name = "name of method")
		public String hello() {
			return "hello";
		}
	 
		public static void main(String[] args) throws NoSuchMethodException, SecurityException {
	 
			Class<AnnotationTest> annotationTest = AnnotationTest.class;
			// 获取类上的所有注解
			Annotation[] annotations = annotationTest.getAnnotations();
			for (Annotation annotation : annotations) {
				// 获取注解的全类名
				System.out.println(annotation.annotationType().getName());
			}
	 
			// 获取 hello() 方法
			Method method = annotationTest.getMethod("hello", new Class[] {});
	 
			// hello() 方法上是否有 MyAnnotation 注解
			if (method.isAnnotationPresent(MyAnnotation.class)) {
	 
				// 获得注解
				MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
	 
				// 获取注解的内容
				System.out.println(annotation.name());
	 
			}
		}
	}
```
运行程序，打印结果如下：  
```
com.pengjunlee.MyAnnotation
name of method
```