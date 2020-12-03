---
title: 深入理解JVM系列之--虚拟机类加载机制
date: 2020-07-18 13:11:00
updated: 2020-07-18 13:11:00
tags: Java虚拟机
categories: 深入理解JVM
keywords: Java, javac, class, JVM
type: 
description: 虚拟机类加载机制。
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
> 参考书籍：《深入理解Java虚拟机——JVM高级特性与最佳实践(第2版)》

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、 转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。 

# 类加载的时机

类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、 验证（Verification）、 准备（Preparation）、 解析（Resolution）、 初始化（Initialization）、 使用（Using）和卸载（Unloading）7个阶段。 其中验证、 准备、 解析3个部分统称为连接（Linking），这7个阶段的发生顺序如下图。 
<div align=center>

![虚拟机类加载机制图](http://pengjunlee.3vzhuji.net/static/jvm/92.png "虚拟机类加载机制图")
<div align=left>


Java虚拟机规范中并没有明确指明在什么情况下需要加载（Loading）类，各个虚拟机可自由把握。 但是对于初始化阶段，虚拟机规范则是严格规定了有且只有5种情况必须立即对类进行“初始化”（而加载、 验证、 准备自然需要在此之前开始）：

- 遇到new、 getstatic、 putstatic或invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。 生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、 读取或设置一个类的静态字段（被final修饰、 已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
- 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含main（）方法的那个类），虚拟机会先初始化这个主类。
- 当使用JDK 1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、 REF_putStatic、 REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

对于这5种会触发类进行初始化的场景，虚拟机规范中使用了一个很强烈的限定语：“有且只有”，这5种场景中的行为称为对一个类进行主动引用。 除此之外，所有引用类的方式都不会触发初始化，称为被动引用。

以下是三个被动引用的代码示例。
```Java
	/**
	 * 被动使用类字段演示一： 通过子类引用父类的静态字段，不会导致子类初始化
	 **/
	public class SuperClass {
		static {
			System.out.println("SuperClass init！");
		}
		public static int value = 123;
	}
	 
	public class SubClass extends SuperClass {
		static {
			System.out.println("SubClass init！");
		}
	}
	 
	public class NotInitialization {
	 
		public static void main(String[] args) {
			System.out.println(SubClass.value);
		}
	}
```
执行结果：
<div align=center>

![虚拟机类加载机制图](http://pengjunlee.3vzhuji.net/static/jvm/93.png "虚拟机类加载机制图")
<div align=left>
```Java
	/**
	 * 被动使用类字段演示二： 通过数组定义来引用类，不会触发此类的初始化
	 **/
	public class NotInitialization {
		public static void main(String[] args) {
			SuperClass[] sca = new SuperClass[10];
		}
	}
```
执行结果：`未执行SuperClass的类初始化，即未打印任何文字。`  
```Java
	/**
	 * 被动使用类字段演示三： 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。
	 **/
	public class ConstantClass {
		static{
			System.out.println("ConstClass init！");
			}
		public static final String HELLOWORLD="hello world";
	}
 
	/**
	 * 非主动使用字段演示
	 **/
	public class NotInitialization {
		public static void main(String[] args) {
			System.out.println(ConstantClass.HELLOWORLD);
		}
	}
```
执行结果：
<div align=center>

![虚拟机类加载机制图](http://pengjunlee.3vzhuji.net/static/jvm/94.png "虚拟机类加载机制图")
<div align=left>

# 类加载的过程

## 加载
在加载阶段，虚拟机需要完成以下3件事情：

- 通过一个类的全限定名来获取定义此类的二进制字节流。
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区之中，方法区中的数据存储格式由虚拟机实现自行定义，虚拟机规范未规定此区域的具体数据结构。 然后在内存中实例化一个java.lang.Class类的对象（并没有明确规定是在Java堆中，对于HotSpot虚拟机而言，Class对象比较特殊，它虽然是对象，但是存放在方法区里面），这个对象将作为程序访问方法区中的这些类型数据的外部接口。

加载阶段与连接阶段的部分内容（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的内容，这两个阶段的开始时间仍然保持着固定的先后顺序。

## 验证
验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

从整体上来看，验证阶段大致上会完成下面4个阶段的检验动作：文件格式验证、元数据验证、 字节码验证、 符号引用验证。

- 文件格式验证  
第一阶段要验证字节流是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理。

- 元数据验证  
第二阶段是对字节码描述的信息进行语义分析，以保证其描述的信息符合Java语言规范的要求。

- 字节码验证  
第三阶段是整个验证过程中最复杂的一个阶段，主要目的是通过数据流和控制流分析，确定程序语义是合法的、 符合逻辑的。

- 符号引用验证  
最后一个阶段的校验发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在连接的第三阶段——解析阶段中发生。

## 准备
准备阶段是正式为类变量分配内存并设置类变量初始值（“通常情况”下是数据类型的零值）的阶段，这些变量所使用的内存都将在方法区中进行分配。 

## 解析
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

**符号引用（Symbolic References）**：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。 符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到内存中。 各种虚拟机实现的内存布局可以各不相同，但是它们能接受的符号引用必须都是一致的，因为符号引用的字面量形式明确定义在Java虚拟机规范的Class文件格式中。

**直接引用（Direct References）**：直接引用可以是直接指向目标的指针、 相对偏移量或是一个能间接定位到目标的句柄。 直接引用是和虚拟机实现的内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。 如果有了直接引用，那引用的目标必定已经在内存中存在。

虚拟机规范之中并未规定解析阶段发生的具体时间，只要求了在执行anewarray、checkcast、 getfield、 getstatic、 instanceof、 invokedynamic、 invokeinterface、 invokespecial、invokestatic、 invokevirtual、 ldc、 ldc_w、 multianewarray、 new、 putfield和putstatic这16个用于操作符号引用的字节码指令之前，先对它们所使用的符号引用进行解析。

除invokedynamic指令以外，虚拟机实现可以对第一次解析的结果进行缓存（在运行时常量池中记录直接引用，并把常量标识为已解析状态）从而避免解析动作重复进行。

## 初始化
类初始化阶段是类加载过程的最后一步，前面的类加载过程中，除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全由虚拟机主导和控制。 到了初始化阶段，才真正开始执行类中定义的Java程序代码（或者说是字节码）。

在准备阶段，变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序员通过程序制定的主观计划去初始化类变量和其他资源，或者可以从另外一个角度来表达：初始化阶段是执行类构造器＜clinit＞（）方法的过程。 

＜clinit＞（）方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问，如以下代码所示。
```Java
	public class Test {
		static {
			i = 0;                  // 给变量赋值可以正常编译通过
			System.out.print(i);    // 这句编译器会提示"非法向前引用"
		}
		static int i = 1;
	}
```
- ＜clinit＞（）方法与类的构造函数（或者说实例构造器＜init＞（）方法）不同，它不需要显式地调用父类构造器，虚拟机会保证在子类的＜clinit＞（）方法执行之前，父类的＜clinit＞（）方法已经执行完毕。 因此在虚拟机中第一个被执行的＜clinit＞（）方法的类肯定是java.lang.Object。
- ＜clinit＞（）方法对于类或接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成＜clinit＞（）方法。
- 接口中不能使用静态语句块，但仍然有变量初始化的赋值操作，因此接口与类一样都会生成＜clinit＞（）方法。 但接口与类不同的是，执行接口的＜clinit＞（）方法不需要先执行父接口的＜clinit＞（）方法。 只有当父接口中定义的变量使用时，父接口才会初始化。 另外，接口的实现类在初始化时也一样不会执行接口的＜clinit＞（）方法。
- 虚拟机会保证一个类的＜clinit＞（）方法在多线程环境中被正确地加锁、 同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的＜clinit＞（）方法，其他线程都需要阻塞等待，直到活动线程执行＜clinit＞（）方法完毕。

# 类加载器

虚拟机设计团队把类加载阶段中的“通过一个类的全限定名来获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需要的类。 实现这个动作的代码模块称为“类加载器”。

## 类与类加载器
类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远远不限于类加载阶段。 对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。 这句话可以表达得更通俗一些：比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个Class文件，被同一个虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。 
```Java
	import java.io.IOException;
	import java.io.InputStream;
	 
	import com.pengjunlee.MyClass;
	 
	/**
	 * 类加载器与instanceof关键字演示
	 */
	public class ClassLoaderTest {
	 
		public static void main(String[] args) {
			ClassLoader myLoader = new ClassLoader() {
				public Class<?> loadClass(String name)
						throws ClassNotFoundException {
					try {
						String fileName = name.substring(name.lastIndexOf(".") + 1)
								+ ".class";
						InputStream is = getClass().getResourceAsStream(fileName);
						if (is == null) {
							return super.loadClass(name);
						}
						byte[] b = new byte[is.available()];
						is.read(b);
						return defineClass(name, b, 0, b.length);
					} catch (IOException e) {
						throw new ClassNotFoundException(name);
					}
				}
			};
			try {
				Object obj = myLoader.loadClass("jvm.ClassLoaderTest")
						.newInstance();
				System.out.println(obj.getClass());
				System.out.println(obj instanceof jvm.ClassLoaderTest);
				System.out.println(obj.getClass().getClassLoader().getClass());
				System.out.println(jvm.ClassLoaderTest.class.getClassLoader()
						.getClass());
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
```
执行结果：
<div align=center>

![虚拟机类加载机制图](http://pengjunlee.3vzhuji.net/static/jvm/96.png "虚拟机类加载机制图")
<div align=left>


## 双亲委派模型
从Java虚拟机的角度来讲，只存在两种不同的类加载器：一种是启动类加载器（Bootstrap ClassLoader），这个类加载器使用C++语言实现，是虚拟机自身的一部分；另一种就是所有其他的类加载器，这些类加载器都由Java语言实现，独立于虚拟机外部，并且全都继承自抽象类java.lang.ClassLoader。

- 启动类加载器（Bootstrap ClassLoader）：前面已经介绍过，这个类将器负责将存放在＜JAVA_HOME＞\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。 启动类加载器无法被Java程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器，那直接使用null代替即可。
- 扩展类加载器（Extension ClassLoader）：这个加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载＜JAVA_HOME＞\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。
- 应用程序类加载器（Application ClassLoader）：这个类加载器由sun.misc.Launcher $AppClassLoader实现。 由于这个类加载器是ClassLoader中的getSystemClassLoader（）方法的返回值，所以一般也称它为系统类加载器。 它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

我们的应用程序都是由这3种类加载器互相配合进行加载的，如果有必要，还可以加入自己定义的类加载器。 这些类加载器之间的关系一般如图所示。 

<div align=center>

![虚拟机类加载机制图](http://pengjunlee.3vzhuji.net/static/jvm/95.png "虚拟机类加载机制图")
<div align=left>

图中展示的类加载器之间的这种层次关系，称为类加载器的双亲委派模型（Parents Delegation Model）。 双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。 这里类加载器之间的父子关系一般不会以继承（Inheritance）的关系来实现，而是都使用组合（Composition）关系来复用父加载器的代码。

双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。

java.lang.ClassLoader的loadClass（）方法之中实现双亲委派的相关代码如下。
```Java
    protected synchronized Class<?> loadClass(String name, boolean resolve)
	throws ClassNotFoundException
    {
	//首先，检查请求的类是否已经被加载过了
	Class c = findLoadedClass(name);
	if (c == null) {
	    try {
		if (parent != null) {
		    c = parent.loadClass(name, false);
		} else {
		    c = findBootstrapClassOrNull(name);
		}
	    } catch (ClassNotFoundException e) {
               //如果父类加载器抛出ClassNotFoundException
               //说明父类加载器无法完成加载请求
            }
            if (c == null) {
	        //在父类加载器无法加载的时候
                //再调用本身的findClass方法来进行类加载
	        c = findClass(name);
	    }
	}
	if (resolve) {
	    resolveClass(c);
	}
	return c;
    }
```