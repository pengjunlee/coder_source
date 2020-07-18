---
title: 设计模式系列之--动态代理模式（上）
date: 2020-07-18 12:11:00
updated: 2020-07-18 12:11:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 动态代理模式
type: 
description: 什么是动态代理模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
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
# 一、什么是动态代理
在静态代理(Static Proxy)模式中，代理类都是真实存在的，由程序员提前创建好的java类，是静态的，每一个代理类在编译之后都会生成一个.class字节码文件，静态代理类所实现的接口和所代理的方法早在编译期都已被固定了。

动态代理(Dynamic Proxy)则不同：动态代理使用字节码动态生成和加载技术，在系统运行时动态地生成和加载代理类。

与静态代理相比，动态代理有以下优点：

- 首先，无需再为每一个真实主题写一个形式上完全一样的代理类，假如抽象主题接口中的方法很多的话，为每一个接口方法写一个代理方法也很麻烦，同样地，如果后期抽象主题接口发生变动，则真实主题和代理类都要修改，不利于系统维护；
- 其次，动态代理可以让系统根据实际需要来动态创建代理类，同一个代理类能够代理多个不同的真实主题类，并且可以代理多个不同的方法。

# 二、Java对动态代理的支持
从JDK 1.3版本开始，Java语言提供了对动态代理的支持，在Java中实现动态代理机制，需要用到 java.lang.reflect 包中的 InvocationHandler 接口和 Proxy 类,我们先来看看java的API帮助文档是怎么样对这两个类进行描述的：
## InvocationHandler
```
	InvocationHandler is the interface implemented by the invocation handler of a proxy instance. 
	Each proxy instance has an associated invocation handler. When a method is invoked on a proxy instance, the 
	method invocation is encoded and dispatched to the invoke method of its invocation handler.
```
大致的意思是：
```
	InvocationHandler 是代理实例的调用处理程序实现的接口。
	每个代理实例都具有一个与之关联的调用处理程序。对代理实例调用方法时，将对方法调用进行编码并将其指派到它的调用处理程
	序的 invoke() 方法。
```
invoke() 方法形式如下：  
```Java
	Object invoke(Object proxy,Method method,Object[] args) throws Throwable
```
InvocationHandler 接口只包含invoke()这唯一一个方法，该方法用于处理对代理类实例的方法调用并返回相应的结果，当一个代理实例中的业务方法被调用时将自动调用该方法。invoke()方法包含三个参数，其中第一个参数proxy表示代理类的实例，第二个参数method表示需要代理的方法，第三个参数args表示代理方法的参数数组。 

## Proxy 
```
	Proxy provides static methods for creating dynamic proxy classes and instances, and it is also the
	superclass of all dynamic proxy classes created by those methods.
```
大致的意思是：Proxy 提供用于创建动态代理类和实例的静态方法，它还是由这些方法创建的所有动态代理类的超类。

Proxy提供给我们的静态方法有以下四个：  
```Java
	//返回指定代理实例的调用处理程序。
	static InvocationHandler getInvocationHandler(Object proxy) 
	 
	//返回代理类的 java.lang.Class 对象，并向其提供类加载器和接口数组。
	static Class<?>	getProxyClass(ClassLoader loader, Class<?>[] interfaces) 
	 
	//当且仅当指定的类通过 getProxyClass 方法或 newProxyInstance 方法动态生成为代理类时，返回 true。
	static boolean	isProxyClass(Class<?> cl) 
	 
	//返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
	//方法中的 ClassLoader loader 参数用来指定动态代理类的类加载器，Class<?>[] interfaces用来指定动态代理类要实现的接口。
	//InvocationHandler h 用来指定与即将生成的动态代理对象相关联的调用处理程序
	static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) 
```
下面我们以为数据库增加日志记录功能（为简单起见，我们仅记录下所有操作的执行时间及操作执行的结果）为例来看一看如何使用这两个类实现动态代理：

为了使演示更清晰，在此先定义两个简单类，一个User类和一个Document类分别表示数据库中的用户记录和文档记录，其代码如下。  
```Java
	public class User {
	    // 用户在数据库中的ID
	    private Long id;
	    // 用户的姓名
	    private String name;
	    public User(Long id, String name) {
	        super();
	        this.id = id;
	        this.name = name;
	    }
	    public Long getId() {
	        return id;
	    }
	    public void setId(Long id) {
	        this.id = id;
	    }
	    public String getName() {
	        return name;
	    }
	    public void setName(String name) {
	        this.name = name;
	    }
	}
```
<br/>
```Java
	public class Document {
	    // 文档在数据库中的ID
	    private Long id;
	    // 文档的标题
	    private String title;
	    public Document(Long id, String title) {
	        super();
	        this.id = id;
	        this.title = title;
	    }
	    public Long getId() {
	        return id;
	    }
	    public void setId(Long id) {
	        this.id = id;
	    }
	    public String getTitle() {
	        return title;
	    }
	    public void setTitle(String title) {
	        this.title = title;
	    }
	}
```
另外还需定义一个DataBase类用来扮演数据库的功能，在此为简单起见，将数据库中的用户记录和文档记录分别存储在一个Map中，代码如下。 
```Java
	import java.util.HashMap;
	import java.util.Map;
	public class DataBase {
		private static Map<Long, User> userMap = null;
		private static Map<Long, Document> documentMap = null;
		// 用来记录当前登陆用户信息
		private static User currentUser = null;
	 
		private DataBase() {
			// 数据初始化，为数据库中增加几条用户记录。。。
			userMap = new HashMap<Long, User>();
			userMap.put(20160708L, new User(20160708L, "燕凌娇"));
			userMap.put(20160709L, new User(20160709L, "姬如雪"));
			userMap.put(20160710L, new User(20160710L, "百里登风"));
			// 数据初始化，为数据库中增加几条文档记录。。。
			documentMap = new HashMap<Long, Document>();
			documentMap.put(30160708L, new Document(30160708L, "C++常用算法手册"));
			documentMap.put(30160709L, new Document(30160709L, "深入理解Android内核设计思想"));
			documentMap.put(30160710L, new Document(30160710L, "Java从入门到放弃"));
		}
	 
		public User getCurrentUser() {
			return currentUser;
		}
	 
		public void setCurrentUser(User currentUser) {
			DataBase.currentUser = currentUser;
		}
	 
		public Map<Long, User> getUserMap() {
			return userMap;
		}
	 
		public Map<Long, Document> getDocumentMap() {
			return documentMap;
		}
	 
		public static DataBase getDataBaseInstance() {
			return DataBaseHolder.dataBase;
		}
	 
		public static class DataBaseHolder {
			private static DataBase dataBase = new DataBase();
		}
	}
```
数据库布置完成了，接下来开始写动态代理相关代码了，为了与静态代理进行比较，在此我们来创建两个接口（抽象主题角色）。 
```Java
	public interface UserDao {
	    // 登陆数据库，为了演示方便将字符串作为执行结果返回
	    String login(Long id);
	    // 退出数据库，为了演示方便将字符串作为执行结果返回
	    String logout();
	}
```
<br/>
```Java
	public interface DocumentDao {
	    // 新增文档，为了演示方便将字符串作为执行结果返回
	    String add(Document document);
	    // 删除文档，为了演示方便将字符串作为执行结果返回
	    String delete(Document document);
	}
```
接下来是两个真实主题角色，ImpUserDao 类和ImpDocumentDao类，为了使示例结果清晰，此处将接口的执行结果直接以字符串形式返回。
```Java
	public class ImpUserDao implements UserDao {
		@Override
		public String login(Long id) {
			User user = DataBase.getDataBaseInstance().getUserMap().get(id);
			if (null != user) {
				// 数据库有此用户的信息，则允许登陆...
				DataBase.getDataBaseInstance().setCurrentUser(user);
				return "用户[" + user.getName() + "]登陆成功...";
			} else {
				// 数据库没有此用户信息，则不让登陆...
				return "登陆失败，ID为\"" + id + "\"的用户不存在！";
			}
		}
	 
		@Override
		public String logout() {
			User user = DataBase.getDataBaseInstance().getCurrentUser();
			if (null != user) {
				// 若当前有用户登陆，则退出成功...
				DataBase.getDataBaseInstance().setCurrentUser(null);
				return "用户[" + user.getName() + "]退出登陆成功...";
			} else {
				// 若当前无用户登陆，则退出失败...
				return "退出登陆失败，当前无登陆用户！";
			}
		}
	}
```
<br/>
```Java
	public class ImpDocumentDao implements DocumentDao {
		@Override
		public String add(Document document) {
			User user = DataBase.getDataBaseInstance().getCurrentUser();
			if (null == user) {
				// 若当前用户未登陆，则新增文档失败...
				return "保存失败，未获取到登陆信息！";
			} else {
				Document dbDocument = DataBase.getDataBaseInstance().getDocumentMap().get(document.getId());
				if (null != dbDocument) {
					// 若数据库中已经存在该ID的文档，则新增文档失败...
					return "添加文档《" + document.getTitle() + "》失败，文档已存在！";
				} else {
					// 若该ID的文档在数据库不存在，则新增文档成功...
					DataBase.getDataBaseInstance().getDocumentMap().put(document.getId(), document);
					return "添加文档《" + document.getTitle() + "》成功...";
				}
			}
		}
	 
		@Override
		public String delete(Document document) {
			User user = DataBase.getDataBaseInstance().getCurrentUser();
			if (null == user) {
				// 若当前用户未登陆，则新增文档失败...
				return "保存失败，未获取到登陆信息！";
			} else {
				Document dbDocument = DataBase.getDataBaseInstance().getDocumentMap().get(document.getId());
				if (null == dbDocument) {
					// 若数据库中该文档不存在，则删除文档失败...
					return "删除文档《" + document.getTitle() + "》失败，文档不存在！";
				} else {
					// 若数据库中该文档存在，则删除文档成功...
					DataBase.getDataBaseInstance().getDocumentMap().remove(document.getId());
					return "删除文档《" + document.getTitle() + "》成功...";
				}
			}
		}
	}
```
最后，我们就要定义动态代理类了，前面说过，每一个动态代理类都必须要实现 InvocationHandler 这个接口，因此我们这个动态代理类也不例外。 
```Java
	import java.lang.reflect.InvocationHandler;
	import java.lang.reflect.Method;
	import java.util.Calendar;
	import java.util.GregorianCalendar;
	 
	public class DataBaseLogHandler implements InvocationHandler {
		private Object object;
		private Calendar calendar;
	 
		public DataBaseLogHandler(Object object) {
			super();
			this.object = object;
		}
	 
		// invoke()方法用于处理对代理类实例的方法调用并返回相应的结果
		@Override
		public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
			before(method);
			// 继续转发请求给内部真实主题角色
			Object result = method.invoke(object, args);
			after(result);
			if (method.getName().equalsIgnoreCase("logout")) {
				System.out.println("==================");
				System.out.println("");
			}
			return result;
		}
	 
		public void before(Method method) {
			calendar = new GregorianCalendar();
			int year = calendar.get(Calendar.YEAR);
			int month = calendar.get(Calendar.MONTH);
			int date = calendar.get(Calendar.DATE);
			int hour = calendar.get(Calendar.HOUR_OF_DAY);
			int minute = calendar.get(Calendar.MINUTE);
			int second = calendar.get(Calendar.SECOND);
			String time = hour + "时" + minute + "分" + second + "秒";
			System.out.println("北京时间：" + year + "年" + month + "月" + date + "日" + time + ",执行方法\"" + method.getName() + "\"");
		}
	 
		public void after(Object object) {
			System.out.println("执行结果:" + object);
		}
	}
```
至此，动态代理所需要的类就算创建完成了，接下来创建一个Client充当客户端来测试一下。
```Java
	import java.lang.reflect.Proxy;
	 
	public class MainClass {
		public static void main(String[] args) {
			// 此处来创建了两个动态代理类对象...
			UserDao userDao = new ImpUserDao();
			DataBaseLogHandler userHandler = new DataBaseLogHandler(userDao);
			DocumentDao doucumentDao = new ImpDocumentDao();
			DataBaseLogHandler documentHandler = new DataBaseLogHandler(doucumentDao);
			UserDao userProxy = (UserDao) Proxy.newProxyInstance(UserDao.class.getClassLoader(),
					new Class[] { UserDao.class }, userHandler);
			DocumentDao documentProxy = (DocumentDao) Proxy.newProxyInstance(DocumentDao.class.getClassLoader(),
					new Class[] { DocumentDao.class }, documentHandler);
			// 先输入一个不存在的用户Id登陆试试...
			userProxy.login(20160718L);
			documentProxy.add(new Document(30160711L, "转角遇见幸福"));
			userProxy.logout();
			// 再用一个真实用户Id登陆试试...
			userProxy.login(20160708L);
			documentProxy.add(new Document(30160711L, "转角遇见幸福"));
			documentProxy.add(new Document(30160711L, "转角遇见幸福"));
			userProxy.logout();
		}
	}
```
运行程序打印结果如下： 
```
北京时间：2016年6月11日19时33分46秒,执行方法"login"
执行结果:登陆失败，ID为"20160718"的用户不存在！
北京时间：2016年6月11日19时33分46秒,执行方法"add"
执行结果:保存失败，未获取到登陆信息！
北京时间：2016年6月11日19时33分46秒,执行方法"logout"
执行结果:退出登陆失败，当前无登陆用户！
==================
北京时间：2016年6月11日19时33分46秒,执行方法"login"
执行结果:用户[燕凌娇]登陆成功...
北京时间：2016年6月11日19时33分46秒,执行方法"add"
执行结果:添加文档《转角遇见幸福》成功...
北京时间：2016年6月11日19时33分46秒,执行方法"add"
执行结果:添加文档《转角遇见幸福》失败，文档已存在！
北京时间：2016年6月11日19时33分46秒,执行方法"logout"
执行结果:用户[燕凌娇]退出登陆成功...
================== 
```
从以上程序的最终运行结果可以看出：通过使用JDK自带的动态代理，我们同时实现了对ImpUserDao和ImpDocumentDao两个真实主题类的统一代理和集中控制。至于该动态代理类是如何被创建的？将在下一篇文章详细讨论，接下来我们先看看如何使用CGLIB实现动态代理。  

# 三、使用CGLIB实现动态代理
生成动态代理类的方法很多，如上例中JDK自带的动态代理、CGLIB、Javassist 或者 ASM 库。JDK 的动态代理使用简单，它内置在 JDK 中，因此不需要引入第三方 Jar 包，但相对功能比较弱。CGLIB 和 Javassist 都是高级的字节码生成库，总体性能比 JDK 自带的动态代理好，而且功能十分强大。ASM 是低级的字节码生成工具，使用 ASM 已经近乎于在使用 Java bytecode 编程，对开发人员要求最高，当然，也是性能最好的一种动态代理生成工具。但 ASM 的使用很繁琐，而且性能也没有数量级的提升，与 CGLIB 等高级字节码生成工具相比，ASM 程序的维护性较差，如果不是在对性能有苛刻要求的场合，还是推荐 CGLIB 或者 Javassist。

接下来我们继续用上面的例子来体验一下如何使用CGLIB实现动态代理。

首先，使用CGLIB来实现动态代理需引入“asm.jar”(CGLIB的底层是使用ASM实现的)和“cglib.jar”两个第三方jar包,引入两个jar包时需注意其版本，若版本有冲突会出现以下异常：<font color='red'> Exception in thread "main" java.lang.NoSuchMethodError: org.objectweb.asm.ClassWriter.<init>(I)V </font>

接下来开始写代码，延用上例中的：User类、Document类、DataBase类、两个抽象主题接口UserDao和DocumentDao、两个真实主题角色类ImpUserDao和ImpDocumentDao，这几个类无需做任何修改，此处不再重复贴出。

去掉上例中的 DataBaseLogHandler 类，新增一个 CglibProxy 类，该类需实现CGLIB的 MethodInterceptor(方法拦截) 接口。 
```Java
	import java.lang.reflect.Method;
	import java.util.Calendar;
	import java.util.GregorianCalendar;
	import net.sf.cglib.proxy.Enhancer;
	import net.sf.cglib.proxy.MethodInterceptor;
	import net.sf.cglib.proxy.MethodProxy;
	 
	public class CglibProxy implements MethodInterceptor {
		private Calendar calendar;
	 
		/**
		 * 创建动态代理类对象
		 *
		 * @param clazz
		 *            需要创建子类代理的父类的类型
		 * 
		 * @return
		 */
		public Object getProxyInstance(Class<?> clazz) {
			Enhancer enhancer = new Enhancer();
			// 设置要创建的动态代理类的父类
			enhancer.setSuperclass(clazz);
			// 设置回调的对象，此句会导致调用动态代理类对象的方法会被指派到CglibProxy对象的intercept()方法
			enhancer.setCallback(this);
			// 通过字节码技术动态创建动态代理类实例
			return enhancer.create();
		}
	 
		@Override
		// 回调方法
		public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
			before(method);
			// 通过动态子类代理实例调用父类的方法
			Object result = proxy.invokeSuper(obj, args);
			after(result);
			if (method.getName().equalsIgnoreCase("logout")) {
				System.out.println("**********************************");
				System.out.println("");
			}
			return result;
		}
	 
		public void before(Method method) {
			calendar = new GregorianCalendar();
			int year = calendar.get(Calendar.YEAR);
			int month = calendar.get(Calendar.MONTH);
			int date = calendar.get(Calendar.DATE);
			int hour = calendar.get(Calendar.HOUR_OF_DAY);
			int minute = calendar.get(Calendar.MINUTE);
			int second = calendar.get(Calendar.SECOND);
			String time = hour + "时" + minute + "分" + second + "秒";
			System.out
					.println("北京时间：" + year + "年" + month + "月" + date + "日" + time + ",执行方法\"" + method.getName() + "\"");
		}
	 
		public void after(Object object) {
			System.out.println("执行结果:" + object);
		}
	}
```
对客户端进行相应修改，如下。 
```Java
	public class MainClass {
		public static void main(String[] args) {
			// 创建一个CglibProxy代理类对象，用来创建子类代理实例
			CglibProxy cglib = new CglibProxy();
			// 为ImpUserDao类添加一个动态代理类对象，即子类代理对象
			UserDao userProxy = (UserDao) cglib.getProxyInstance(ImpUserDao.class);
			// 为ImpDocumentDao类添加一个动态代理类对象，即子类代理对象
			DocumentDao documentProxy = (DocumentDao) cglib.getProxyInstance(ImpDocumentDao.class);
			// 先输入一个不存在的用户Id登陆试试...
			userProxy.login(20160718L);
			documentProxy.add(new Document(30160711L, "转角遇见幸福"));
			userProxy.logout();
			// 再用一个真实用户Id登陆试试...
			userProxy.login(20160708L);
			documentProxy.add(new Document(30160711L, "转角遇见幸福"));
			documentProxy.add(new Document(30160711L, "转角遇见幸福"));
			userProxy.logout();
		}
	}
```
运行程序结果打印如下，与之前使用JDK自带动态代理程序运行结果完全相同： 
```Java
北京时间：2016年6月12日20时22分35秒,执行方法"login"
执行结果:登陆失败，ID为"20160718"的用户不存在！
北京时间：2016年6月12日20时22分35秒,执行方法"add"
执行结果:保存失败，未获取到登陆信息！
北京时间：2016年6月12日20时22分35秒,执行方法"logout"
执行结果:退出登陆失败，当前无登陆用户！
==================
北京时间：2016年6月12日20时22分35秒,执行方法"login"
执行结果:用户[燕凌娇]登陆成功...
北京时间：2016年6月12日20时22分35秒,执行方法"add"
执行结果:添加文档《转角遇见幸福》成功...
北京时间：2016年6月12日20时22分35秒,执行方法"add"
执行结果:添加文档《转角遇见幸福》失败，文档已存在！
北京时间：2016年6月12日20时22分35秒,执行方法"logout"
执行结果:用户[燕凌娇]退出登陆成功...
==================  
```
> PS:CGLib创建的动态代理对象性能比JDK创建的动态代理对象的性能高不少，但是CGLib在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。同时，由于CGLib采用动态创建子类的方法来对被代理的父类的功能进行增强和代理，所以，无法对被声明为final的类或方法进行代理。  

# 四、动态代理模式的特点
动态代理类使用字节码动态生成加载技术，在运行时生成并加载代理类。与静态代理相比，**动态代理具有以下优点：**

- 无需单独为每一个接口添加一个代理类，使用动态代理可以一次性为多个接口实现代理。
- 无需逐个为接口中的所有方法添加实现，使用动态代理可以一次性为多个接口中的所有方法实现代理，在接口方法数量比较多的时候，可以避免出现大量的重复代码。

**动态代理的缺点：**  

目前，JDK中提供的动态代理只能对接口实现代理，无法代理未实现接口的类。如果需要动态代理未实现接口的类，必须借助第三方工具，如：CGLib(Code Generation Library)等。

# 五、参考文章

[https://www.cnblogs.com/xiaoluo501395377/p/3383130.html](https://www.cnblogs.com/xiaoluo501395377/p/3383130.html "java的动态代理机制详解")

[https://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern/](https://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern/ "代理模式原理及实例讲解")

[http://www.lxway.com/4445954962.htm](http://www.lxway.com/4445954962.htm "代理模式详解包含原理详解")

[https://blog.csdn.net/lovelion/article/details/8116704](https://blog.csdn.net/lovelion/article/details/8116704 "Java动态代理的实现")

[https://blog.csdn.net/hejingyuan6/article/details/36203505](https://blog.csdn.net/hejingyuan6/article/details/36203505 "JAVA学习篇--静态代理VS动态代理")

[https://blog.csdn.net/rokii/article/details/4046098](https://blog.csdn.net/rokii/article/details/4046098 "深入理解Java Proxy机制")