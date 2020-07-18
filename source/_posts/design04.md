---
title: 设计模式系列之--单例模式
date: 2020-07-18 12:04:00
updated: 2020-07-18 12:04:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 单例模式
type: 
description: 什么是单例模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
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
# 一、什么是单例模式

单例（Singleton）模式是一种对象创建型模式，保证一个类只有一个实例存在，同时该类提供能对该实例加以访问的全局访问方法。

`单例模式的本质：控制实例个数`

设计意图：使用单例模式，可以保证为一个类只生成唯一的实例对象。也就是说，在整个程序空间中，该类只存在一个实例对象。 

# 二、单例模式的应用场景

如果碰到以下情况，可以考虑使用单例模式：

- 在多个线程之间，比如servlet环境，需共享同一个资源或者操作同一个对象。
- 在整个程序空间使用的全局变量，共享资源等。
- 大规模系统中，为了性能的考虑，需要节省对象的创建时间，等等...。  

# 三、单例模式实现

- 饿汉式。
- 懒汉式。
- 双重检查。 

## 饿汉式 
为保证单例类只能创建唯一实例需将该类的构造方法私有化以防止其他类通过构造方法显式创建单例类实例，另外单例类需对外提供一个公共的全局访问方法以使其他类可以通过该方法获取到单例类的唯一实例，饿汉式简单示例代码如下。 
```Java
	public class Singleton {
	    private static Singleton singleton= new Singleton();
	    /**
	     * 私有构造方法
	     */
	    private Singleton(){}
	    /**
	     * 静态的全局访问方法
	     */
	    public static Singleton getSingleton(){
	        return singleton;
	    }
	}
```
在饿汉式中，在类加载时，静态变量singleton会调用类的私有构造方法进行初始化。这时候，单例类的唯一实例就被创建出来了。

饿汉式是典型的空间换时间，当类装载的时候就会创建类的实例，不管你用不用，先创建出来，然后每次调用的时候，就不需要再判断，节省了运行时间。

## 懒汉式
与饿汉式不同，懒汉式唯一实例采取了延迟实例化的方法，先不创建实例，等到需要用到实例的时候再创建。真的很懒，有木有？其代码示例如下。
```Java
	public class Singleton {
		private static Singleton singleton = null;
		/**
		 * 私有默认构造方法
		 */
		private Singleton() {
		}
		/**
		 * 静态的全局访问方法
		 */
		public static Singleton getSingleton() {
			if (null == singleton) {
				singleton = new Singleton();
			}
			return singleton;
		}
	}
```
懒汉式是典型的时间换空间,就是每次获取实例都会进行判断，看是否需要创建实例，浪费时间去判断。当然，如果一直没有人使用的话，那就不会创建实例，则节约内存空间。

上述懒汉式Singleton类在单线程中是安全的，但在多线程中存在隐患：首次运行<font color='red'> singleton=null </font>时，当有线程 **A、B** 需调用 **getSingleton()** 获取**singleton**实例时，<font color='red'>线程A运行-->进行判断if (null == singleton)*-->进入if语句块中，还未来得及初始化此时线程A的运行时间突然结束，线程B开始运行，线程B也进行判断if (null == singleton)此时线程A还未对singleton完成初始化，singleton仍为null，线程B也进入if语句块中，线程B对singleton进行初始化并获取到singleton实例，在线程B结束运行之后线程A又被唤醒，线程A继续执行它之前未完成的singleton初始化工作，初始化完成后线程A也获得了singleton实例</font>。在这种情况下线程A，B获取到的singleton实例就会指向两个不同的Singleton对象而不是同一实例。

既然上述懒汉式Singleton类在多线程中是不安全的，可以通过将getSingleton()方法同步化来解决，代码如下：
```Java
	public class Singleton {
		private static Singleton singleton = null;
		/**
		 * 私有默认构造方法
		 */
		private Singleton() {
		}
		/**
		 * 静态的全局访问方法
		 */
		public static synchronized Singleton getSingleton() {
			if (null == singleton) {
				singleton = new Singleton();
			}
			return singleton;
		}
	}
```
这样Singleton类虽然解决了线程安全问题，可又带来了新的问题，由于getSingleton()方法被同步化，当多个线程同时需要获取singleton实例时只能一个一个线程排队执行，这样会降低整个访问的速度。

那么，有没有一种办法可以既可以实现懒汉式的延迟实例化，又线程安全且不会降低整个系统的访问速度呢？答案是：双重检查。

## 双重检查
双重检查单例模式的产生是为了解决懒汉式单例模式同步化后系统执行效率慢的问题，其代码示例如下，
```Java
	public class Singleton {
		private volatile static Singleton singleton = null;
		/**
		 * 私有默认构造方法
		 */
		private Singleton() {
		}
		/**
		 * 静态的全局访问方法
		 */
		public static Singleton getSingleton() {
			//先检查实例是否已创建，如果未创建才进入同步块
			if (null == singleton) {
				synchronized (Singleton.class) {
					//再次检查实例是否已创建，如果真的未创建才创建实例
					if (null == singleton) { 
						singleton = new Singleton();
					}
				}
			}
			return singleton;
		}
	}
```
所谓的双重检查即：<font color='red'>调用getSingleton()方法时先检查singleton实例是否已被创建，如果未创建才进入同步块，这是第一重检查，进入同步块之后再次检查确认singleton实例是否已被创建，如果真的未被创建才创建实例，这是第二重检查。这样一来，同步只在第一次调用getSingleton()方法singleton==null时进行，之后每次获取实例都无需进行同步判断，减少了同步判断浪费的时间</font>。

> “双重检查”机制的实现使用了关键字volatile，它的意思是：被volatile修饰的变量的值，将不会被本地线程缓存，所有对该变量的读写都是直接操作共享内存，从而确保多个线程能正确的处理该变量。

# 四、单例模式的特点

+ 单例类只能有一个实例。
+ 单例类必须自己创建自己的唯一实例。
+ 单例类必须向整个系统提供对该实例加以访问的全局访问方法。  