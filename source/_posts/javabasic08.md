---
title: Java知识点系列之--RMI
date: 2020-07-20 13:08:00
updated: 2020-07-20 13:08:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 一起来了解一下Java中的RMI。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
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
> 参考书籍：《Head First Java （中文版）》第二版 

**Java**的远程方法调用 (`Remote Method Invocation,RMI`)技术，能够帮助我们实现：某一个**Java**虚拟机上的对象可以调用另一台计算机、另一个**Java**虚拟机上面的对象的方法，就像调用本地对象的方法一样。

使用**RMI**时，需要指定协议：**JRMP**（`Java远程消息交换协议JRMP，Java Remote Messaging Protocol`）或是**IIOP**（`互联网内部对象请求代理协议，Internet Inter-ORB Protocol`）。**JRMP**是**RMI**原生的协议，它是为了Java对Java间的远程调用而设计的。另一方面，**IIOP**是为了**CORBA**（`Common Object Request Broker Architecture`）而产生的，它让你能够调用Java对象或其它类型的远程方法。

RMI远程方法调用过程如下图所示： 
<div align=center>

![rmi示意图](http://pengjunlee.3vzhuji.net/static/javacore/7.png "rmi示意图")
<div align=left>

# 创建远程服务

创建远程服务包含以下5个步骤：

## 创建远程服务接口

远程的服务接口要继承`java.rmi.Remote`，用来定义客户端可以远程调用的方法，**stub**和**skeleton**都要实现此接口。它是个作为服务的多态化类，也就是说，客户端会调用实现此接口的**stub**，而此**stub**因为会执行网络和输入/输出工作，所以可能会发生各种问题，客户端必须处理或声明异常来认知这一类风险。并且，远程方法的参数和返回值必须都是**primitive**基本数据类型或**Serializable**的。 
```Java
	package rmi;
	 
	import java.rmi.Remote;
	import java.rmi.RemoteException;
	 
	/**
	 * 远程服务接口
	 */
	public interface IRemoteService extends Remote {
	 
		public String sayHello() throws RemoteException;
	}
```

## 实现远程服务接口

这是真正执行的类，它实现了定义在远程服务接口中的方法，它是客户端会调用的对象。为了要成为远程服务对象，这个实现类必须要包含与远程有关的功能。其中最简单的方式就是继承`java.rim.server.UnicastRemoteObject`，以调用父类的远程功能来处理这些工作。`UnicastRemoteObject`有个小问题：它的构造函数会抛出`RemoteException`异常，因此需要为你的实现类声明一个构造函数，并在其中抛出`RemoteException`异常。 
```Java
	package rmi;
	 
	import java.rmi.RemoteException;
	import java.rmi.server.UnicastRemoteObject;
	 
	/**
	 * 远程服务实现
	 */
	public class RemoteServiceImpl extends UnicastRemoteObject implements
			IRemoteService {
	 
		// 无需声明RemoteException异常
		public String sayHello() {
			return "Server syas , 'Hey'";
		}
	 
		// 父类的构造函数声明了异常，所以你也得在构造函数中声明异常
		public RemoteServiceImpl() throws RemoteException {
	 
		}
	 
	}
```
> <font color=red>注意</font>：要记得当类被初始化的时候，父类的构造函数一定会被调用，如果父类的构造函数抛出异常，你也得声明你的构造函数会抛出异常。

## 使用rmic产生stub和skeleton

伴随`Java Software Development Kit`而来的`rmic`工具会以远程服务的实现类（不是远程服务接口）产生出两个新的类：`stub`和`skeleton`。它会按照命名规则在你的远程服务实现类名称后面加上`_Stub`或`_Skel`（JDK 1.2以后将只需要`_Stub`文件）。

执行`rmic`命令时需要考虑到包目录结构和完整名称，我的所有示例代码的`.class`文件均放置在`D:\bin\rmi`目录下，`RemoteServiceImpl`类的完整名称为`rmi（包路径）.RemoteServiceImpl`。
<div align=center>

![rmi示意图](http://pengjunlee.3vzhuji.net/static/javacore/8.png "rmi示意图")
<div align=left>

由于未指定`rmic`的生成的目标文件的存放位置，所以会将生成的`stub`文件和`skeleton`文件放置在`RemoteServiceImpl.class`文件所在目录(D:\bin\rmi)下。  
<div align=center>

![rmi示意图](http://pengjunlee.3vzhuji.net/static/javacore/9.png "rmi示意图")
<div align=left>

## 启动RMI registry（执行rmiregistry）

`rmiregistry`就像是电话簿，用户会从此处取得代理（客户端的`stub`对象），因此需要在远程服务启动前先启动`RMI registry`。 

<div align=center>

![rmi示意图](http://pengjunlee.3vzhuji.net/static/javacore/10.png "rmi示意图")
<div align=left>

## 启动远程服务

远程服务实现类定义完成之后还需要使用`java.rmi.Naming的bind()`方法 来向`RMI registry`注册服务，当你注册对象时，**RMI**系统会把**stub**加到`RMI registry`中。 
```Java
	package rmi;
	 
	import java.rmi.Naming;
	 
	public class Server {
		public static void main(String[] args) {
			try {
				// 创建出远程对象，然后使用静态的Naming.bind()来产生关联
				// 所注册的名称会供客户端查询
				IRemoteService service = new RemoteServiceImpl();
				Naming.bind("RemoteHello", service);
				//Naming.rebind("Remote Hello", service);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
```
<div align=center>

![rmi示意图](http://pengjunlee.3vzhuji.net/static/javacore/11.png "rmi示意图")
<div align=left>

# 客户端如何取得stub对象？

客户端必须取得`stub`对象，因为客户端必须要调用它的方法，这就得靠`RMI registry`了。客户端会像查询电话簿一样地搜索，找出上面有相符名称的服务。

RMI会自动将stub解序列化，这就要求客户端在查询服务时一定要有stub类文件，否则将导致stub不能被解序列化。 
```Java
	package rmi;
	 
	import java.rmi.Naming;
	 
	public class Client {
	 
		public static void main(String[] args) {
			new Client().go();
		}
	 
		public void go() {
			try {
				//客户端必须使用与服务相同的类型
				//事实上，客户端不需要知道服务实际上的类型
				IRemoteService service = (IRemoteService) Naming.lookup("rmi://127.0.0.1/RemoteHello");
				String s = service.sayHello();
				System.out.println(s);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
```
<div align=center>

![rmi示意图](http://pengjunlee.3vzhuji.net/static/javacore/12.png "rmi示意图")
<div align=left>

> <font color=red>注意事项</font>：客户端是使用接口来调用stub上的方法，客户端的Java虚拟机必须要有stub类，但客户端不会在程序代码中引用stub类，客户端总是通过接口来操作真正的远程对象。
服务器上必须要有stub和skeleton，以及服务于远程的接口，它会需要stub类是因为stub会被代换成连接在RMI registry上真正的服务。 