---
title: 设计模式系列之--动态代理模式（下）
date: 2020-07-18 12:12:00
updated: 2020-07-18 12:12:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 动态代理模式
type: 
description: 什么是动态代理模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img12.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img12.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# 一、前章回顾

在前一章`动态代理模式（上）`中我们分别使用JDK自带的动态代理和CGLIB动态代理为数据库增加了日志记录功能。然而，生成的动态代理类到底是个什么样子呢？本章我们就一起来揭开它的庐山真面目。  

# 二、深入剖析JAVA动态代理类
我们先来看看生成的动态代理类是什么类型，对前一章第二节中示例的客户端进行如下简单修改，将生成的动态代理类实例的类型打印到控制台。 
```Java
	import java.lang.reflect.Proxy;
	 
	public class MainClass {
		public static void main(String[] args) {
			// 创建用来处理代理对象请求的调用处理程序userHandler
			UserDao userDao = new ImpUserDao();
			DataBaseLogHandler userHandler = new DataBaseLogHandler(userDao);
			// 创建用来处理代理对象请求的调用处理程序documentHandler
			DocumentDao doucumentDao = new ImpDocumentDao();
			DataBaseLogHandler documentHandler = new DataBaseLogHandler(doucumentDao);
			// 开始创建代理对象，分别实现接口UserDao和DocumentDao
			UserDao userProxy = (UserDao) Proxy.newProxyInstance(UserDao.class.getClassLoader(),
					new Class[] { UserDao.class }, userHandler);
			DocumentDao documentProxy = (DocumentDao) Proxy.newProxyInstance(DocumentDao.class.getClassLoader(),
					new Class[] { DocumentDao.class }, documentHandler);
			// 将两个代理类的类型打印到控制台
			System.out.println("实现了UserDao接口的动态代理类型为：" + userProxy.getClass());
			System.out.println("实现了DocumentDao接口的动态代理类型为：" + documentProxy.getClass());
			// 接下来我们再创建两个动态代理类对象，二者都实现UserDao和DocumentDao两个接口，只是接口实现的先后顺序不同
			UserDao udProxy = (UserDao) Proxy.newProxyInstance(UserDao.class.getClassLoader(),
					new Class[] { UserDao.class, DocumentDao.class }, userHandler);
			DocumentDao duProxy = (DocumentDao) Proxy.newProxyInstance(DocumentDao.class.getClassLoader(),
					new Class[] { DocumentDao.class, UserDao.class }, documentHandler);
			// 将这两个代理类的类型也打印到控制台
			System.out.println("实现了UserDao和DocumentDao接口的动态代理类型为：" + udProxy.getClass());
			System.out.println("实现了DocumentDao和UserDao接口的动态代理类型为：" + duProxy.getClass());
		}
	}
```
运行程序打印结果如下：
```
实现了UserDao接口的动态代理类型为：class $Proxy0
实现了DocumentDao接口的动态代理类型为：class $Proxy1
实现了UserDao和DocumentDao接口的动态代理类型为：class $Proxy2
实现了DocumentDao和UserDao接口的动态代理类型为：class $Proxy3
```
原来生成的动态代理类都是形似 $Proxy0、$Proxy1、$Proxy2、$Proxy3 这种类型，同时，貌似生成的动态代理类的类型与传入方法newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) 中的interfaces接口数组参数有关，而且接口数组中接口的先后顺序也会对最后生成的动态代理类的类型产生影响（这一点貌似sun做得不是很好）。

知晓了这一点，接下来我们再刨深点，看看这些代理类到底长什么样子，对MainClass再进行修改。
```Java
	import java.lang.reflect.Field;
	import java.lang.reflect.Method;
	import java.lang.reflect.Modifier;
	import java.lang.reflect.Proxy;
	 
	public class MainClass {
		public static void main(String[] args) {
			UserDao userDao = new ImpUserDao();
			DataBaseLogHandler userHandler = new DataBaseLogHandler(userDao);
			UserDao userProxy = (UserDao) Proxy.newProxyInstance(UserDao.class.getClassLoader(),
					new Class[] { UserDao.class }, userHandler);
			System.out.println("实现了UserDao接口的动态代理类的类型为：" + userProxy.getClass());
			// 接下来我们在创建两个动态代理类对象，二者都实现UserDao和DocumentDao两个接口，只是接口实现的先后顺序不同
			UserDao udProxy = (UserDao) Proxy.newProxyInstance(UserDao.class.getClassLoader(),
					new Class[] { UserDao.class, DocumentDao.class }, userHandler);
			System.out.println("实现了UserDao和DocumentDao接口的动态代理类的类型为：" + udProxy.getClass());
			System.out.println("");
			Class userProxyClass = userProxy.getClass();
			Class udProxyClass = udProxy.getClass();
			printClassDefinition(userProxyClass);
			printClassDefinition(udProxyClass);
		}
	 
		public static String getModifier(int modifier) {
			String result = "";
			switch (modifier) {
			case Modifier.PRIVATE:
				result = "private";
			case Modifier.PUBLIC:
				result = "public";
			case Modifier.PROTECTED:
				result = "protected";
			case Modifier.ABSTRACT:
				result = "abstract";
			case Modifier.FINAL:
				result = "final";
			case Modifier.NATIVE:
				result = "native";
			case Modifier.STATIC:
				result = "static";
			case Modifier.SYNCHRONIZED:
				result = "synchronized";
			case Modifier.STRICT:
				result = "strict";
			case Modifier.TRANSIENT:
				result = "transient";
			case Modifier.VOLATILE:
				result = "volatile";
			case Modifier.INTERFACE:
				result = "interface";
			}
			return result;
		}
	 
		public static void printClassDefinition(Class clz) {
			String clzModifier = getModifier(clz.getModifiers());
			if (clzModifier != null && !clzModifier.equals("")) {
				clzModifier = clzModifier + " ";
			}
			String superClz = clz.getSuperclass().getName();
			if (superClz != null && !superClz.equals("")) {
				superClz = "extends " + superClz;
			}
			Class[] interfaces = clz.getInterfaces();
			String inters = "";
			for (int i = 0; i < interfaces.length; i++) {
				if (i == 0) {
					inters += "implements " + interfaces[i].getName();
				} else {
					inters += "," + interfaces[i].getName();
				}
			}
			System.out.println(clzModifier + clz.getName() + " " + superClz + " " + inters);
			System.out.println("{");
			Field[] fields = clz.getDeclaredFields();
			for (int i = 0; i < fields.length; i++) {
				String modifier = getModifier(fields[i].getModifiers());
				if (modifier != null && !modifier.equals("")) {
					modifier = modifier + " ";
				}
				String fieldName = fields[i].getName();
				String fieldType = fields[i].getType().getName();
				System.out.println("    " + modifier + fieldType + " " + fieldName + ";");
			}
			System.out.println();
			Method[] methods = clz.getDeclaredMethods();
			for (int i = 0; i < methods.length; i++) {
				Method method = methods[i];
				String modifier = getModifier(method.getModifiers());
				if (modifier != null && !modifier.equals("")) {
					modifier = modifier + " ";
				}
				String methodName = method.getName();
				Class returnClz = method.getReturnType();
				String retrunType = returnClz.getName();
				Class[] clzs = method.getParameterTypes();
				String paraList = "(";
				for (int j = 0; j < clzs.length; j++) {
					paraList += clzs[j].getName();
					if (j != clzs.length - 1) {
						paraList += ", ";
					}
				}
				paraList += ")";
				clzs = method.getExceptionTypes();
				String exceptions = "";
				for (int j = 0; j < clzs.length; j++) {
					if (j == 0) {
						exceptions += "throws ";
					}
					exceptions += clzs[j].getName();
					if (j != clzs.length - 1) {
						exceptions += ", ";
					}
				}
				exceptions += ";";
				String methodPrototype = modifier + retrunType + " " + methodName + paraList + exceptions;
				System.out.println("    " + methodPrototype);
			}
			System.out.println("}");
		}
	}
```
运行程序打印结果如下：
```
实现了UserDao接口的动态代理类型为：class $Proxy0
实现了UserDao和DocumentDao接口的动态代理类型为：class $Proxy1
实现了UserDao接口的动态代理类型为：class $Proxy0
实现了UserDao和DocumentDao接口的动态代理类型为：class $Proxy1
```
```Java
	$Proxy0 extends java.lang.reflect.Proxy implements UserDao
	{
		java.lang.reflect.Method m1;
		java.lang.reflect.Method m3;
		java.lang.reflect.Method m4;
		java.lang.reflect.Method m0;
		java.lang.reflect.Method m2;
		java.lang.String login(java.lang.Long, java.lang.String);
		java.lang.String logout();
		boolean equals(java.lang.Object);
		java.lang.String toString();
		int hashCode();
	}
```
<br/>
```Java
	$Proxy1 extends java.lang.reflect.Proxy implements UserDao，DocumentDao
	{
	    java.lang.reflect.Method m1;
	    java.lang.reflect.Method m6;
	    java.lang.reflect.Method m3;
	    java.lang.reflect.Method m5;
	    java.lang.reflect.Method m4;
	    java.lang.reflect.Method m0;
	    java.lang.reflect.Method m2;
	    java.lang.String login(java.lang.Long, java.lang.String);
	    java.lang.String logout();
	    java.lang.String add(Document);
	    boolean equals(java.lang.Object);
	    java.lang.String toString();
	    int hashCode();
	    java.lang.String delete(Document);
	}
```
这下子似乎是清晰了一点，生成的代理类继承了java.lang.reflect.Proxy类，并实现了作为参数传入newProxyInstance()方法中的 Class<?>[] interfaces 接口数组，但由于Java的反射机制并不能获取到方法体的具体内容，所以动态代理类方法体的具体内容我们还不得而知。为此，我们不得不另辟蹊径：使用class字节码文件反编译方法，以期待能够看到动态代理类中方法体的具体内容。

由于 Proxy.newProxyInstance()方法在生成动态代理类字节码文件的时候会调用到 sun.misc.ProxyGenerator.generateProxyClass()方法，在 Proxy.newProxyInstance()方法调用之前可以进行设置,将generateProxyClass()生成的代理类的字节码文件以 .class 文件的形式备份到当前Java项目的根目录下,配置代码如下：
```Java
System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
```
使用软件对上例中实现了UserDao接口的代理类 $Proxy0 的字节码文件进行反编译，其完整内容如下。
```Java
	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.lang.reflect.Proxy;
	import java.lang.reflect.UndeclaredThrowableException;
	public final class $Proxy0
	extends Proxy
	implements UserDao {
	    private static Method m1;
	    private static Method m3;
	    private static Method m4;
	    private static Method m0;
	    private static Method m2;
	    public $Proxy0(InvocationHandler invocationHandler) throws  {
	        super(invocationHandler);
	    }
	    public final boolean equals(Object object) throws  {
	        try {
	            return (Boolean)this.h.invoke(this, m1, new Object[]{object});
	        }
	        catch (Error | RuntimeException v0) {
	            throw v0;
	        }
	        catch (Throwable var2_2) {
	            throw new UndeclaredThrowableException(var2_2);
	        }
	    }
	    public final String login(Long l) throws  {
	        try {
	            return (String)this.h.invoke(this, m3, new Object[]{l});
	        }
	        catch (Error | RuntimeException v0) {
	            throw v0;
	        }
	        catch (Throwable var2_2) {
	            throw new UndeclaredThrowableException(var2_2);
	        }
	    }
	    public final String logout() throws  {
	        try {
	            return (String)this.h.invoke(this, m4, null);
	        }
	        catch (Error | RuntimeException v0) {
	            throw v0;
	        }
	        catch (Throwable var1_1) {
	            throw new UndeclaredThrowableException(var1_1);
	        }
	    }
	    public final int hashCode() throws  {
	        try {
	            return (Integer)this.h.invoke(this, m0, null);
	        }
	        catch (Error | RuntimeException v0) {
	            throw v0;
	        }
	        catch (Throwable var1_1) {
	            throw new UndeclaredThrowableException(var1_1);
	        }
	    }
	    public final String toString() throws  {
	        try {
	            return (String)this.h.invoke(this, m2, null);
	        }
	        catch (Error | RuntimeException v0) {
	            throw v0;
	        }
	        catch (Throwable var1_1) {
	            throw new UndeclaredThrowableException(var1_1);
	        }
	    }
	    static {
	        try {
	            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
	            m3 = Class.forName("UserDao").getMethod("login", Class.forName("java.lang.Long"));
	            m4 = Class.forName("UserDao").getMethod("logout", new Class[0]);
	            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
	            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
	            return;
	        }
	        catch (NoSuchMethodException var1) {
	            throw new NoSuchMethodError(var1.getMessage());
	        }
	        catch (ClassNotFoundException var1_1) {
	            throw new NoClassDefFoundError(var1_1.getMessage());
	        }
	    }
	}
```
通过对以上动态代理类字节码文件反编译的内容的观察，我们不难得出以下结论：
所谓动态代理，其实就是java.lang.reflect.Proxy类动态地根据你所指定的接口生成一个class 字节码文件，该 class 会继承 Proxy 类，并实现所有你指定的接口（您在newProxyInstance()方法的参数中传入的 Class<?>[] interfaces 接口数组）中的所有方法，同时，还会从java.lang.Object继承equals()、toString()、hashCode()三个方法；然后再利用你指定的类加载器（您在newProxyInstance()方法的参数中传入的 ClassLoader loader）将 class 字节码文件加载进系统，最后生成这样一个类的对象，并初始化该对象的一些值、以及所有的 Method 成员， 初始化之后将对象返回给调用的客户端，这样客户端拿到的就是一个实现你指定的所有接口的Proxy对象了。然后，客户端在调用该动态代理对象的方法时会通过动态代理类中的 this.h.invoke()方法自动将请求转发给与动态代理对象关联的InvocationHandler对象(您在newProxyInstance()方法的参数中传入的 InvocationHandler h)的invoke()方法，由invoke()方法来实现对请求的统一处理并返回处理结果。

# 三、参考文章

[https://www.cnblogs.com/xiaoluo501395377/p/3383130.html](https://www.cnblogs.com/xiaoluo501395377/p/3383130.html "java的动态代理机制详解")

[https://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern/](https://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern/ "代理模式原理及实例讲解")

[http://www.lxway.com/4445954962.htm](http://www.lxway.com/4445954962.htm "代理模式详解包含原理详解")

[https://blog.csdn.net/lovelion/article/details/8116704](https://blog.csdn.net/lovelion/article/details/8116704 "Java动态代理的实现")

[https://blog.csdn.net/hejingyuan6/article/details/36203505](https://blog.csdn.net/hejingyuan6/article/details/36203505 "JAVA学习篇--静态代理VS动态代理")

[https://blog.csdn.net/rokii/article/details/4046098](https://blog.csdn.net/rokii/article/details/4046098 "深入理解Java Proxy机制")
