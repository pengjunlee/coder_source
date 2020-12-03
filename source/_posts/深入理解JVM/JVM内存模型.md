---
title: 深入理解JVM系列之--JVM内存模型
date: 2020-07-18 13:04:00
updated: 2020-07-18 13:04:00
tags: Java虚拟机
categories: 深入理解JVM
keywords: Java, JVM
type: 
description: 初识JVM内存模型。
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
> 参考书籍：《深入理解Java虚拟机——JVM高级特性与最佳实践(第2版)》

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则是依赖用户线程的启动和结束而建立和销毁。根据《Java虚拟机规范（Java SE 7 版）》的规定，Java虚拟机所管理的内存将会包括以下几个运行时数据区域，如下图所示：

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/63.png "JVM内存模型图")
<div align=left>


# 一、运行时数据区域

## 1、程序计数器
程序计数器（Program Counter Register）是一块较小的内存空间，它的作用可以看作是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

由于Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

如果线程正在执行的是一个Java 方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie 方法，这个计数器值则为空（Undefined）。此内存区域是唯一一个在Java 虚拟机规范中没有规定任何OutOfMemoryError 情况的区域。 

## 2、Java 虚拟机栈
与程序计数器一样，Java 虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java 方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

经常有人把Java 内存粗略地区分为堆内存（Heap）和栈内存（Stack），这里的“栈内存”就是指的虚拟机栈，或者说是虚拟机栈中的局部变量表部分。

局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不等同于对象本身，根据不同的虚拟机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress 类型（指向了一条字节码指令的地址）。

其中64 位长度的long 和double 类型的数据会占用2 个局部变量空间（Slot），其余的数据类型只占用1 个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

在Java 虚拟机规范中，对这个区域规定了两种异常状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError 异常；如果虚拟机栈可以动态扩展（当前大部分的Java 虚拟机都可动态扩展，只不过Java 虚拟机规范中也允许固定长度的虚拟机栈），当扩展时无法申请到足够的内存时会抛出OutOfMemoryError 异常。

## 3、本地方法栈
本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java 方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native 方法服务。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun HotSpot 虚拟机）直接就把本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError 和OutOfMemoryError异常。

## 4、Java 堆
对于大多数应用来说，Java 堆（Java Heap）是Java 虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。这一点在Java 虚拟机规范中的描述是：所有的对象实例以及数组都要在堆上分配，但是随着JIT 编译器的发展与逃逸分析技术的逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化发生，所有的对象都分配在堆上也渐渐变得不是那么“绝对”了。

Java 堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC 堆”（Garbage Collected Heap，幸好国内没翻译成“垃圾堆”）。从内存回收的角度来看，由于现在收集器基本都是采用的分代收集算法，所以Java 堆中还可以细分为：新生代和老年代；再细致一点的有Eden 空间、From Survivor 空间、To Survivor 空间等。从内存分配的角度来看，线程共享的Java 堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB）。不过，无论如何划分，都与存放内容无关，无论哪个区域，存储的都仍然是对象实例，进一步划分的目的是为了更好地回收内存，或者更快地分配内存。

根据Java 虚拟机规范的规定，Java 堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘空间一样。在实现时，既可以实现成固定大小的，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的（通过-Xmx和-Xms 控制）。如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError 异常。

## 5、方法区
方法区（Method Area）与Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是与Java 堆区分开来。

对于习惯在HotSpot 虚拟机上开发和部署程序的开发者来说，很多人愿意把方法区称为“永久代”（Permanent Generation），本质上两者并不等价，仅仅是因为HotSpot 虚拟机的设计团队选择把GC 分代收集扩展至方法区，或者说使用永久代来实现方法区而已。对于其他虚拟机（如BEA JRockit、IBM J9 等）来说是不存在永久代的概念的。即使是HotSpot 虚拟机本身，根据官方发布的路线图信息，现在也有放弃永久代并“搬家”至Native Memory 来实现方法区的规划了。

Java 虚拟机规范对这个区域的限制非常宽松，除了和Java 堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是有必要的。在Sun 公司的BUG 列表中，曾出
现过的若干个严重的BUG 就是由于低版本的HotSpot 虚拟机对此区域未完全回收而导致内存泄漏。

根据Java 虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError 异常。

## 6、运行时常量池
运行时常量池（Runtime Constant Pool）是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述等信息外，还有一项信息是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

Java 虚拟机对Class 文件的每一部分（自然也包括常量池）的格式都有严格的规定，每一个字节用于存储哪种数据都必须符合规范上的要求，这样才会被虚拟机认可、装载和执行。但对于运行时常量池，Java 虚拟机规范没有做任何细节的要求，不同的提供商实现的虚拟机可以按照自己的需要来实现这个内存区域。不过，一般来说，除了保存Class 文件中描述的符号引用外，还会把翻译出来的直接引用也存储在运行时常量池中。

运行时常量池相对于Class 文件常量池的另外一个重要特征是具备动态性，Java 语言并不要求常量一定只能在编译期产生，也就是并非预置入Class 文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String 类的intern() 方法。

既然运行时常量池是方法区的一部分，自然会受到方法区内存的限制，当常量池无法再申请到内存时会抛出OutOfMemoryError 异常。

## 7、直接内存
直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError 异常出现，所以我们放到这里一起讲解。

在JDK 1.4 中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O 方式，它可以使用Native 函数库直接分配堆外内存，然后通过一个存储在Java 堆里面的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java 堆和Native 堆中来回复制数据。

显然，本机直接内存的分配不会受到Java 堆大小的限制，但是，既然是内存，则肯定还是会受到本机总内存（包括RAM 及SWAP 区或者分页文件）的大小及处理器寻址空间的限制。服务器管理员配置虚拟机参数时，一般会根据实际内存设置-Xmx等参数信息，但经常会忽略掉直接内存，使得各个内存区域的总和大于物理内存限制（包括物理上的和操作系统级的限制），从而导致动态扩展时出现OutOfMemoryError异常。 

# 二、OutOfMemoryError异常

在Java虚拟机规范的描述中，除了程序计数器外，虚拟机内存的其他几个运行时区域都有发生OutOfMemoryError（OOM）异常的可能，以下将通过实例来模拟各个区域发生OOM异常的场景。

## 1、Java堆溢出
Java堆用于存储对象实例，只要不断地创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么在对象数量到达最大堆的容量限制后就会产生内存溢出。

修改Eclipse IDE的虚拟机启动参数如下：`-Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 `

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/64.png "JVM内存模型图")
<div align=left>


将Java堆的大小限制为20MB，不可扩展（将堆的最小值-Xms参数与最大值-Xmx参数设置为一样即可避免堆自动扩展），其中年轻代大小为10M（-Xmn参数指定），Eden空间与单个Survivor空间（From Survivor空间或To Survivor空间）大小比值为 8:1。

此时JVM堆内存各个空间大小分配如下：

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/65.png "JVM内存模型图")
<div align=left>

```
（Heap总大小）20MB = （Eden空间）8MB+（Survivor空间）1MB*2+（老年代）10MB
```

测试代码如下：
```Java
	import java.util.ArrayList;
	import java.util.List;
	 
	/**
	 * VM Args:-Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
	 */
	public class HeapOOM {
	 
		static class OOMObject {
		}
	 
		public static void main(String[] args) {
	 
			List<OOMObject> list = new ArrayList<OOMObject>();
			while (true) {
				list.add(new OOMObject());
			}
		}
	}
```
运行结果： 

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/66.png "JVM内存模型图")
<div align=left>


## 2、虚拟机栈和本地方法栈溢出
由于在HotSpot虚拟机中并不区分虚拟机栈和本地方法栈，因此，对于hotSpot来说，虽然-Xoss参数（设置本地方法栈大小）存在，但实际上是无效的，栈容量只由-Xss参数设定。关于虚拟机栈和本地方法栈，在Java虚拟机规范中描述了两种异常：

- 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。
- 如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常。

测试代码如下：
```Java
	/**
	 * VM Args:-Xss128k
	 */
	 
	public class JavaVMStackSOF {
		private int stackLength = 1;
	 
		public void stackLeak() {
			stackLength++;
			stackLeak();
		}
	 
		public static void main(String[] args) throws Throwable {
			JavaVMStackSOF sof = new JavaVMStackSOF();
			try {
				sof.stackLeak();
			} catch (Throwable e) {
				System.out.println("stack length:" + sof.stackLength);
				throw e;
			}
		}
	}
```
运行结果：

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/67.png "JVM内存模型图")
<div align=left>


注意：

- 以上测试代码在单线程中进行操作，若使用-Xss参数减少栈内存容量或在方法中定义本地变量，均会导致抛出StackOverflowError异常时输出的堆栈深度减小。
- 在单线程下，无论是由于栈帧太大还是虚拟机栈容量太小，当内存无法分配的时候，虚拟机抛出的都是StackOverflowError异常。

理论上通过不断地创建线程的方式是可以产生内存溢出异常的，例如下面的测试代码，但是这样产生的内存溢出与栈空间是否足够大并不存在任何联系，或者准确地说，在这种情况下为每个线程分配的内存越大，反而越容易产生内存溢出异常。

导致以上现象出现的原因如下：操作系统分配给每个进程的内存是有限制的，譬如32位的Windows限制为2GB。虚拟机提供了参数来控制Java堆和方法区的这两部分内存的最大值。剩余的内存为2GB（Windows操作系统限制）减去Xmx（最大堆容量），再减去MaxPermSize（最大方法区容量），程序计数器消耗内存很小，可以忽略掉。如果虚拟机进程本身耗费的内存不计算在内，剩下的内存就由虚拟机栈和本地方法栈“瓜分”了。每个线程分配到的栈容量越大，可以建立的线程数量自然就越少，建立线程时就越容易把剩下的内存耗尽。 
```Java
	/**
	 *VM Args:-Xss4M(这时候不妨设置大些)
	 */
	public class JavaVMStackOOM {
		
		private void dontStop() {
			while (true) {
			}
		}
	 
		public void stackLeakByThread() {
			while (true) {
				Thread thread = new Thread(new Runnable() {
					public void run() {
						dontStop();
					}
				});
				thread.start();
				System.out.println(Thread.activeCount());
			}
		}
	 
		public static void main(String[] args) throws Throwable {
			JavaVMStackOOM oom = new JavaVMStackOOM();
			oom.stackLeakByThread();
		}
	}
```
运行结果： 
<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/68.png "JVM内存模型图")
<div align=left>

> <font color=red>注意：由于Windows平台的虚拟机中，Java的线程是映射到操作系统的内核线程上的，因此上述代码执行时有较大的风险，可能会导致操作系统假死，并不一定会抛出OutOfMemoryError异常。</font>

## 3、方法区和运行时常量池溢出
String.intern()是一个Native方法，它的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。在JDK 1.６及以前的版本中，由于常量池分配在永久代内，我们可以通过-XX:PermSize和 -XX:MaxPermSize限制方法区大小，从而间接限制其中常量池的容量。

测试代码如下：
```Java
	import java.util.List;
	import java.util.ArrayList;
	 
	/**
	 * VM Args:-XX:PermSize=10M -XX:MaxPermSize=10M
	 */
	public class RuntimeConstantPoolOOM {
		public static void main(String[] args) {
			// 使用List保持常量池引用，避免Full GC回收常量池内的对象
			List<String> list = new ArrayList<String>();
			int i = 0;
			while (true) {
				list.add(String.valueOf((i++)).intern());
			}
		}
	 
	}
```
运行结果：

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/69.png "JVM内存模型图")
<div align=left>


方法区用于存放Class的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。对于这些区域的测试，基本思路是运行时产生大量的类去填满方法区，直到溢出。以下示例代码借助CGLIB直接操作字节码运行时生成大量的动态类进行测试。 
```Java
	import java.lang.reflect.Method;
	 
	import net.sf.cglib.proxy.Enhancer;
	import net.sf.cglib.proxy.MethodInterceptor;
	import net.sf.cglib.proxy.MethodProxy;
	/**
	 * VM Args:-XX:PermSize=10M -XX:MaxPermSize=10M
	 */
	public class JavaMethodAreaOOM {
	 
		static class OOMObject {
		}
	 
		public static void main(String[] args) {
			while (true) {
				Enhancer enhancer = new Enhancer();
				enhancer.setSuperclass(OOMObject.class);
				enhancer.setUseCache(false);
				enhancer.setCallback(new MethodInterceptor() {
					public Object intercept(Object obj, Method method,
							Object[] args, MethodProxy proxy) throws Throwable {
						return proxy.invokeSuper(obj, args);
					}
				});
				enhancer.create();
			}
		}
	}
```
运行结果：

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/70.png "JVM内存模型图")
<div align=left>


## 4、本机直接内存溢出
DirectMemory容量可通过-XX:MaxDirectMemorySize指定，如果不指定，则默认与Java堆最大值（-Xmx指定）一样，测试代码越过了DirectByteBuffer类，直接通过反射获取Unsafe示例进行内存分配（Unsafe类的getUnsafe()方法限制了只有引导类加载器才会返回实例，也就是设计者希望只有rt.jar中的类才能使用Unsafe的功能）。因为，虽然使用DirectByteBuffer分配内存也会抛出内存溢出异常，但它抛出异常时并没有真正向操作系统申请分配内存，而是通过计算得知内存无法分配，于是手动抛出异常，真正申请分配内存的方法是unsafe.allocateMemory()。

测试代码如下：
```Java
	import java.lang.reflect.Field;
	 
	import sun.misc.Unsafe;
	/**
	 * VM Args:-Xmx20M -XX:MaxDirectMemorySize=10M
	 */
	public class DirectMemoryOOM {
	 
		private static final int _1MB = 1024 * 1024;
	 
		public static void main(String[] args) throws Exception {
			Field unsafeField = Unsafe.class.getDeclaredFields()[0];
			unsafeField.setAccessible(true);
			Unsafe unsafe = (Unsafe) unsafeField.get(null);
			while (true) {
				unsafe.allocateMemory(_1MB);
			}
		}
	}
```
运行结果：

<div align=center>

![JVM内存模型意图](http://pengjunlee.3vzhuji.net/static/jvm/71.png "JVM内存模型图")
<div align=left>