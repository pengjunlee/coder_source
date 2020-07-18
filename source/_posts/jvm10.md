---
title: 深入理解JVM系列之--性能监控与故障诊断
date: 2020-07-18 13:10:00
updated: 2020-07-18 13:10:00
tags: Java虚拟机
categories: 深入理解JVM
keywords: Java, javac, class, JVM
type: 
description: 虚拟机性能监控与故障诊断。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img10.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img10.jpg
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
> 参考书籍：《深入理解Java虚拟机——JVM高级特性与最佳实践(第2版)》

SUN公司在JDK的bin目录中为Java开发人员提供的命令行工具，除了大家熟知的`java.exe`、`javac.exe`和`javap.exe`之外，还包含一些功能强大且稳定的虚拟机性能监控与故障处理工具，能在处理应用程序性能问题、 定位故障时发挥很大的作用，例如下面这几个。

<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/34.png "性能监控与故障诊断")
<div align=left>


还是先来看看它们都是干什么用的吧，稍后再对各个工具的作用及用法进行详细介绍。 
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/35.png "性能监控与故障诊断")
<div align=left>


# jps：查看虚拟机进程状况

jps（JVM Process Status  Tool）的作用是：列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class,main（）函数所在的类）名称以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier,LVMID），对于本地虚拟机进程来说，LVMID与操作系统的进程ID（Process Identifier,PID）是一致的。
```Bash
	# jps 命令格式：
	jps [options] [hostid]
```
jps可以通过RMI协议查询开启了RMI服务的远程虚拟机进程状态，hostid为RMI注册表中注册的主机名。 jps的其他常用选项见下表。

<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/36.png "性能监控与故障诊断")
<div align=left>

jps执行样例：

在Eclipse IDE中将main()方法传入参数及JVM启动参数进行如下设置，并执行自定义类jvm.JpsTest中的main()方法。

<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/37.png "性能监控与故障诊断")
<div align=left>

使用命令 jps -mlv 查看所有的JVM进程、main()方法传入参数、JVM启动参数，并显示类的包全名。
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/38.png "性能监控与故障诊断")
<div align=left>


# jstat：监视虚拟机运行状态

jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。 它可以显示本地或者远程虚拟机进程中的类装载、 内存、 垃圾收集、 JIT编译等运行数据，在没有GUI图形界面，只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的首选工具。
```Bash
	# jstat 命令格式为：
	jstat [ option vmid [ interval[s|ms] [count]] ]
```
对于命令格式中的VMID与LVMID需要特别说明一下：如果是本地虚拟机进程，VMID与LVMID是一致的，如果是远程虚拟机进程，那VMID的格式应当是：`[protocol：][//]lvmid[@hostname[：port]/servername]`

参数interval和count代表查询间隔和次数，如果省略这两个参数，说明只查询一次。 假设需要每250毫秒查询一次进程2764垃圾收集状况，一共查询20次，那命令应当是：`jstat -gc 2764 250 20`

选项option代表着用户希望查询的虚拟机信息，主要分为3类：类装载、 垃圾收集、 运行期编译状况，具体选项及作用如下表。 
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/39.png "性能监控与故障诊断")
<div align=left>

jstat执行样例：

我们以监视Java堆的状况为例，使用命令：`jstat -gcutil [进程ID]`
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/40.png "性能监控与故障诊断")
<div align=left>


监视结果表明：这台服务器的新生代Eden区（E，表示Eden）使用了6.01%的空间，两个Survivor区（S0、 S1，表示Survivor0、 Survivor1）里面都是空的，老年代（O，表示Old）和永久代（P，表示Permanent）则分别使用了1.88%和14.28%的空间。 程序运行以来共发生Minor GC（YGC，表示Young GC）14次，总耗时0.037秒，发生Full GC（FGC，表示FullGC）0次，Full GC总耗时（FGCT，表示Full GC Time）为0.000秒，所有GC总耗时（GCT，表示GC Time）为0.037秒。



# jinfo：管理JVM配置信息

jinfo（Configuration Info for Java）的作用是实时地查看和调整虚拟机各项参数。 
```Bash
	# jinfo 命令格式：
	jinfo [option] pid
```
选项option的基本用法如下表。 
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/41.png "性能监控与故障诊断")
<div align=left>

jinfo执行样例：
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/42.png "性能监控与故障诊断")
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/43.png "性能监控与故障诊断")
<div align=left>

# jmap：Java内存映像工具

jmap（Memory Map for Java）命令用于生成堆转储快照（一般称为heapdump或dump文件）
```Bash
	# jmap 命令格式：
	jmap [option] vmid
```
具体选项及作用如下表。
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/44.png "性能监控与故障诊断")
<div align=left>

jmap执行样例：生成一个正在执行的eclipse进程的dump快照文件。
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/45.png "性能监控与故障诊断")
<div align=left>

使用 clstats 参数，以ClassLoader为统计口径显示永久代内存状态
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/46.png "性能监控与故障诊断")
<div align=left>

# jhat：JVM快照分析工具

Sun JDK提供jhat（JVM Heap Analysis Tool）命令与jmap搭配使用，来分析jmap生成的堆转储快照。 jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，可以在浏览器中查看。

jhat执行样例：使用jhat分析上例中采用jmap生成的Eclipse IDE的内存快照文件。 
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/47.png "性能监控与故障诊断")
<div align=left>

屏幕显示“Server is ready.”的提示后，用户在浏览器中键入`http://localhost：7000/`就可以看到分析结果，分析结果默认是以包为单位进行分组显示，如下图所示。 
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/48.png "性能监控与故障诊断")
<div align=left>

# jstack：Java堆栈跟踪工具

jstack（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照（一般称为threaddump或者javacore文件）。 线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、 死循环、 请求外部资源导致的长时间等待等都是导致线程长时间停顿的常见原因。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者等待着什么资源。
```Bash
	# jstack 命令格式：
	jstack [option] vmid
```
option选项的合法值与具体含义下表。 
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/49.png "性能监控与故障诊断")
<div align=left>

jstack执行样例：使用jstack查看一个java线程的堆栈。
<div align=center>

![性能监控与故障诊断](http://pengjunlee.3vzhuji.net/static/jvm/50.png "性能监控与故障诊断")
<div align=left>

在JDK 1.5中，java.lang.Thread类新增了一个getAllStackTraces（）方法用于获取虚拟机中所有线程的StackTraceElement对象。
```Java
	import java.util.Map;
	 
	public class JstackTrace {
	 
		public void printThreadTrace() {
			for (Map.Entry<Thread, StackTraceElement[]> stackTrace : Thread
					.getAllStackTraces().entrySet()) {
				Thread thread = (Thread) stackTrace.getKey();
				StackTraceElement[] stack = (StackTraceElement[]) stackTrace
						.getValue();
				if (thread.equals(Thread.currentThread())) {
					continue;
				}
				System.out.print("\n线程:" + thread.getName() + "\n");
				for (StackTraceElement element : stack) {
					System.out.print("\t" + element + "\n");
				}
			}
		}
	 
	}
```