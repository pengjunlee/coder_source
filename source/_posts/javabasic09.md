---
title: Java知识点系列之--Thread
date: 2020-07-20 13:09:00
updated: 2020-07-20 13:09:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 一起来了解一下Java中的Thread线程。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
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
# 一、什么是线程？

线程是一个程序的多个执行路径，执行调度的单位，依托于进程而存在。 线程不仅可以共享进程的内存，而且还拥有一个属于自己的内存空间，这段内存空间也叫做线程栈，是在建立线程时由系统分配的，主要用来保存线程内部所使用的数据，如线程执行函数中所定义的变量。

> <font color=red>注意</font>：Java中的多线程是一种抢占机制而不是分时机制。抢占机制指的是有多个线程处于可运行状态，但是只允许一个线程在运行，他们通过竞争的方式抢占CPU。  

# 二、线程的创建

在Java中可以通过以下两种方式定义线程：

- 继承java.lang.Thread类 。
- 实现java.lang.Runnable接口。

## 继承java.lang.Thread类
```Java
	/**
	 * 通过继承java.lang.Thread类来定义线程
	 */
	public class MyThread extends Thread {
	 
		private boolean flag = true;
	 
		public MyThread(String threadName) {
			super(threadName);
		}
	 
		@Override
		public void run() {
			while (flag) {
				System.out.println(Thread.currentThread().getName()
						+ " Is Running...");
			}
		}
	 
		public void shutDown() {
			this.flag = false;
		}
	 
		public static void main(String[] args) throws InterruptedException {
			MyThread myThread = new MyThread("MyThread");
			myThread.start();
			Thread.sleep(1000);
			myThread.shutDown();
	 
		}
	}
```
## 实现java.lang.Runnable接口
```Java
	/**
	 * 通过实现Runnable接口来定义线程
	 */
	public class MyThread implements Runnable {
		private boolean flag = true;
	 
		@Override
		public void run() {
			while (flag) {
				System.out.println(Thread.currentThread().getName()
						+ " Is Running...");
			}
		}
	 
		public void shutDown() {
			this.flag = false;
		}
	 
		public static void main(String[] args) throws InterruptedException {
	 
			MyThread myThread = new MyThread();
			Thread thread = new Thread(myThread, "MyThread");
			thread.start();
			Thread.sleep(1000);
			myThread.shutDown();
	 
		}
	}
```
在线程的`start()`方法被调用后，JVM会自动调用线程的`run()`方法来执行任务。start()方法结束，线程也就终止了。

> <font color=red>注意</font>：推荐使用实现Runnable接口的方式来定义线程，因为Java中的类只支持单继承，你一旦继承了Thread类就不能再继承其他的类了，会大大降低程序的可扩展性和灵活性。  

# 三、线程的状态

<div align=center>

![性能监控与故障处理示意图](http://pengjunlee.3vzhuji.net/static/javacore/13.png "垃圾收集器示意图")
<div align=left>

# 四、线程的基本方法和属性

## 1）优先级（priority）

每个线程都有一个优先级(用1-10的整数表示,默认优先级是5，高优先级线程的执行优先于低优先级线程。默认一个线程的优先级和创建他的线程优先级相同。

以下是Thread定义的三个与优先级相关的静态常量： 
```
MAX_PRIORITY =10
NORM_PRIORITY =5
MIN_PRIORITY=1
```

## 2）Thread.sleep(long millis)/sleep(long millis)

使当前线程休眠（暂停执行）millis毫秒的时间（millis指定的休眠时间是其最小的不执行时间，因为在sleep()休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级）；sleep()是Thread类的static(静态)的方法；因此他不能改变对象的机锁，所以当在一个Synchronized块中调用sleep()方法时，线程虽然休眠了，但是对象的机锁并没有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁）。 

<font color=blue>作用</font>：保持对象锁，让出CPU，调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留一定的执行机会给其他线程；

## 3）Thread.yield()

让出CPU的使用权，让其他同等优先级或更高优先级的线程可以获取到运行机会，线程yield()时也不会释放对象锁。

**sleep()和yield()的区别**：

- sleep()方法会给其他线程运行的机会，而不考虑其他线程的优先级，因此会给较低优先级线程一个运行的机会；yield()方法只会给相同优先级或者更高优先级的线程一个运行的机会。 
- 当线程执行了sleep(long millis)方法后，将转到阻塞状态，参数millis指定休眠时间；当线程执行了yield()方法后，将转到就绪状态。 
- sleep()方法声明抛出InterruptedException异常，而yield()方法没有声明抛出任何异常 。

## 4）thread.join()

线程合并，使用该方法的线程会在此线程执行完毕后才往下继续执行，可使异步线程变为同步线程。

## 5）object.wait()

wait()方法是Object类里的方法；当一个线程执行到wait()方法时，它就进入到一个和该对象相关的等待池中，同时失去（释放）了对象的机锁（暂时失去机锁，wait(long timeout)超时时间到后还需要返还对象锁），其他线程可以访问。执行wait()方法的线程必须拥有当前对象的锁，如果当前线程不是此锁的拥有者，会抛出IllegalMonitorStateException异常,所以wait()必须在synchronized block中调用。

wait()使用notify或者notifyAlll或者指定睡眠时间来唤醒当前等待池中的线程。

## 6）object.notify()/notifyAll()

唤醒在当前对象等待池中等待的一个线程/所有线程。notify()/notifyAll()也必须拥有相同对象锁，否则也会抛出IllegalMonitorStateException异常。

## 7）Synchronized Block

Java中的每一个对象都有唯一的一个内置的锁，每个Synchronized Block或同步的方法只有持有调用该方法被锁定对象的锁的线程才可以访问，否则所属线程阻塞；机锁具有独占性、一旦被一个Thread持有，其他的Thread就不能再拥有（不能访问其他同步方法），方法一旦执行，就独占该锁，直到从该方法返回时才将锁释放，此后被阻塞的线程方能获得该锁，重新进入可执行状态。

# 五、线程资源共享示例
```Java
	/**
	 * 车站售票示例
	 */
	public class TicketsThread implements Runnable {
	 
		// 车票总数
		private Integer tickets = 100;
		private boolean flag = true;
		private Object object = new Object();
	 
		@Override
		public void run() {
			while (flag) {
				/*	synchronized (object) {
					if (tickets > 0) {
						try {
							Thread.sleep(100);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
						System.out.println(Thread.currentThread().getName() + "售出第"
								+ tickets + "张车票");
						tickets--;
					} else {
						System.out.println(Thread.currentThread().getName()
								+ "提醒广大乘客：车票已经售完，停止售票！");
						this.flag = false;
					}
				}*/
				sell();
			}
	 
		}
	 
		private  synchronized void sell(){
			if (tickets > 0) {
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().getName() + "售出第"
						+ tickets + "张车票");
				tickets--;
			} else {
				System.out.println(Thread.currentThread().getName()
						+ "提醒广大乘客：车票已经售完，停止售票！");
				this.flag = false;
			}
		}
		
		public void shutDown() {
			this.flag = false;
		}
	 
		public static void main(String[] args) {
			TicketsThread ticketsThread = new TicketsThread();
			Thread thread1 = new Thread(ticketsThread, "售票处1");
			Thread thread2 = new Thread(ticketsThread, "售票处2");
			Thread thread3 = new Thread(ticketsThread, "售票处3");
	 
			thread1.start();
			thread2.start();
			thread3.start();
	 
		}
	 
	}
```
<br/>
```Java
	/**
	 * 死锁示例
	 */
	public class DeadThread implements Runnable {
	 
		private Object obj1 = new Object();
		private Object obj2 = new Object();
	 
		private boolean flag = true;
	 
		@Override
		public void run() {
			if (flag) {
				synchronized (obj1) {
					try {
						Thread.sleep(500);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					synchronized (obj2) {
	 
					}
				}
			} else {
				synchronized (obj2) {
					try {
						Thread.sleep(500);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					synchronized (obj1) {
	 
					}
				}
			}
	 
		}
	 
		public static void main(String[] args) throws InterruptedException {
			DeadThread deadThread = new DeadThread();
			Thread thread1 = new Thread(deadThread);
			Thread thread2 = new Thread(deadThread);
			thread1.start();
			Thread.sleep(100);
			deadThread.flag = false;
			thread2.start();
	 
		}
	 
	}
```