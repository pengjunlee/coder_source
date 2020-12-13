---
title: Java知识点系列之--Instrumentation
date: 2020-07-20 13:15:00
updated: 2020-07-20 13:15:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 一篇文章带你了解Java中的Instrumentation。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
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
利用 Java 代码，即 `java.lang.instrument` 做`动态Instrumentation` 是 Java SE 5 的新特性，它把 Java 的 instrument 功能从本地代码中解放出来，使之可以用 Java 代码的方式解决问题。使用 Instrumentation，开发者可以构建一个独立于应用程序的代理程序（Agent），用来监测和协助运行在 JVM 上的程序，甚至能够替换和修改某些类的定义。有了这样的功能，开发者就可以实现更为灵活的运行时虚拟机监控和Java 类操作了，这样的特性实际上提供了一种虚拟机级别支持的 AOP 实现方式，使得开发者无需对 JDK 做任何升级和改动，就可以实现某些 AOP 的功能了。 

在 Java SE 6 里面，instrumentation 包被赋予了更强大的功能：`启动后的 instrument、本地代码（native code）instrument，以及动态改变 classpath`等等。这些改变，意味着 Java 具有了更强的动态控制、解释能力，它使得 Java 语言变得更加灵活多变。

在 Java SE6 里面，最大的改变使运行时的 Instrumentation 成为可能。在 Java SE 5 中，Instrument 要求在运行前利用命令行参数或者系统参数来设置代理类，在实际的运行之中，虚拟机在初始化之时（在绝大多数的 Java 类库被载入之前），instrumentation 的设置已经启动，并在虚拟机中设置了回调函数，检测特定类的加载情况，并完成实际工作。但是在实际的很多的情况下，我们没有办法在虚拟机启动之时就为其设定代理，这样实际上限制了 instrument 的应用。而 Java SE 6 的新特性改变了这种情况，通过 Java Tool API 中的 attach 方式，我们可以很方便地在运行过程中动态地设置加载代理类，以达到 instrumentation 的目的。

另外，对native 的 Instrumentation也是 Java SE 6 的一个崭新的功能，这使以前无法完成的功能 —— 对 native 接口的 instrumentation 可以在 Java SE 6 中，通过一个或者一系列的 prefix 添加而得以完成。

最后，Java SE 6 里的 Instrumentation 也增加了动态添加 class path的功能。所有这些新的功能，都使得 instrument 包的功能更加丰富，从而使 Java 语言本身更加强大。  

# Instrumentation 的基本功能和用法
“`java.lang.instrument`”包的具体实现，依赖于 `JVMTI`。`JVMTI（Java Virtual Machine Tool Interface）`是一套由 Java 虚拟机提供的，为 JVM 相关的工具提供的本地编程接口集合。JVMTI 是从 Java SE 5 开始引入，整合和取代了以前使用的 Java Virtual Machine Profiler Interface(JVMPI)和 the Java Virtual Machine Debug Interface(JVMDI)，而在 Java SE 6 中，JVMPI 和 JVMDI 已经消失了。JVMTI 提供了一套”代理”程序机制，可以支持第三方工具程序以代理的方式连接和访问 JVM，并利用 JVMTI 提供的丰富的编程接口，完成很多跟 JVM 相关的功能。事实上，java.lang.instrument 包的实现，也就是基于这种机制的：在 Instrumentation 的实现当中，存在一个 JVMTI 的代理程序，通过调用 JVMTI 当中 Java 类相关的函数来完成Java 类的动态操作。除开 Instrumentation 功能外，JVMTI 还在虚拟机内存管理，线程控制，方法和变量操作等等方面提供了大量有价值的函数。关于 JVMTI 的详细信息，请参考 Java SE 6 文档中的介绍。

Instrumentation 的最大作用，就是类定义动态改变和操作。在 Java SE 5 及其后续版本当中，开发者可以在一个普通 Java 程序（带有 main 函数的 Java 类）运行时，通过 -javaagent参数指定一个特定的 jar 文件（包含 Instrumentation 代理）来启动 Instrumentation 的代理程序。

在 Java SE 5 当中，开发者可以让 Instrumentation 代理在 main 函数运行前执行。简要说来就是如下几个步骤： 

**(1) 编写 premain 函数**

编写一个 Java 类，包含如下两个方法当中的任何一个： 
```Java
	public static void premain(String agentArgs, Instrumentation inst); // [1]
	public static void premain(String agentArgs);                       // [2]
```
其中，[1] 的优先级比 [2] 高，将会被优先执行（[1] 和 [2] 同时存在时，[2] 被忽略）。在这个 premain 函数中，开发者可以进行对类的各种操作。

- agentArgs 是 premain 函数得到的程序参数，随同 “-javaagent”一起传入。与 main 函数不同的是，这个参数是一个字符串而不是一个字符串数组，如果程序参数有多个，程序将自行解析这个字符串。
- Inst 是一个 java.lang.instrument.Instrumentation 的实例，由 JVM 自动传入。java.lang.instrument.Instrumentation 是 instrument 包中定义的一个接口，也是这个包的核心部分，集中了其中几乎所有的功能方法，例如类定义的转换和操作等等。 

**(2) jar 文件打包** 

将这个 Java 类打包成一个 jar 文件，并在其中的 manifest 属性当中加入” Premain-Class”来指定步骤 1 当中编写的那个带有 premain 的 Java 类。（可能还需要指定其他属性以开启更多功能）

**(3) 运行** 

用如下方式运行带有 Instrumentation 的 Java 程序：

`java -javaagent:jar 文件的位置 [= 传入 premain 的参数 ]`

对 Java 类文件的操作，可以理解为对一个 byte 数组的操作（将类文件的二进制字节流读入一个 byte 数组）。开发者可以在“ClassFileTransformer”的 transform 方法当中得到，操作并最终返回一个类的定义（一个 byte 数组）。这方面，Apache 的 BCEL 开源项目提供了强有力的支持，具体的字节码操作并非本文的重点，所以，本文中所举的例子，只是采用简单的类文件替换的方式来演示 Instrumentation 的使用。

下面，我们通过简单的举例，来说明 Instrumentation 的基本使用方法。

首先，我们有一个简单的类，TransClass， 可以通过一个静态方法返回一个整数 1。
```Java
	public class TransClass { 
	     public int getNumber() { 
	     return 1; 
	    } 
	}
```
我们运行如下类，可以得到输出 ”1“。
```Java
	public class TestMainInJar { 
	    public static void main(String[] args) { 
	        System.out.println(new TransClass().getNumber()); 
	    } 
	}
```
然后，我们将 TransClass 的 getNumber 方法改成如下 :
```Java
public int getNumber() { 
    return 2; 
}
```
再将这个返回 2 的 Java 文件编译成类文件，为了区别开原有的返回 1 的类，我们将返回 2 的这个类文件命名为 TransClass2.class.2。 

接下来，我们建立一个 Transformer 类：  
```Java
	import java.lang.instrument.ClassFileTransformer; 
	import java.io.FileInputStream; 
	import java.io.IOException; 
	import java.io.InputStream; 
	import java.lang.instrument.ClassFileTransformer; 
	import java.lang.instrument.IllegalClassFormatException; 
	import java.security.ProtectionDomain; 
	 
	class Transformer implements ClassFileTransformer { 
	 
	    public static final String classNumberReturns2 = "TransClass.class.2"; 
	 
	    public static byte[] getBytesFromFile(String fileName) { 
	        try { 
	            // precondition 
	            File file = new File(fileName); 
	            InputStream is = new FileInputStream(file); 
	            long length = file.length(); 
	            byte[] bytes = new byte[(int) length]; 
	 
	            // Read in the bytes 
	            int offset = 0; 
	            int numRead = 0; 
	            while (offset <bytes.length 
	                    && (numRead = is.read(bytes, offset, bytes.length - offset)) >= 0) { 
	                offset += numRead; 
	            } 
	 
	            if (offset < bytes.length) { 
	                throw new IOException("Could not completely read file "
	                        + file.getName()); 
	            } 
	            is.close(); 
	            return bytes; 
	        } catch (Exception e) { 
	            System.out.println("error occurs in _ClassTransformer!"
	                    + e.getClass().getName()); 
	            return null; 
	        } 
	    } 
	 
	    public byte[] transform(ClassLoader l, String className, Class<?> c, 
	            ProtectionDomain pd, byte[] b) throws IllegalClassFormatException { 
	        if (!className.equals("TransClass")) { 
	            return null; 
	        } 
	        return getBytesFromFile(classNumberReturns2); 
	    } 
	}
```
这个类实现了 ClassFileTransformer 接口。其中，getBytesFromFile 方法根据文件名读入二进制字符流，而 ClassFileTransformer 当中规定的 transform 方法则完成了类定义的替换转换。

最后，我们建立一个 Premain 类，写入 Instrumentation 的代理方法 premain：  
```Java
	public class Premain { 
	    public static void premain(String agentArgs， Instrumentation inst)  throws ClassNotFoundException， UnmodifiableClassException { 
	        inst.addTransformer(new Transformer()); 
	    } 
	}
```
可以看出，**addTransformer** 方法并没有指明要转换哪个类。转换发生在 **premain** 函数执行之后，main 函数执行之前，这时每装载一个类，**transform** 方法就会执行一次，看看是否需要转换，所以，在 **transform**（Transformer 类中）方法中，程序用 `className.equals(“TransClass”)` 来判断当前的类是否需要转换。

代码完成后，我们将他们打包为 `TestInstrument1.jar`。返回 1 的那个 **TransClass** 的类文件保留在 jar 包中，而返回 2 的那个 `TransClass.class.2` 则放到 jar 的外面。在 manifest 里面加入如下属性来指定 premain 所在的类： 
```
	Manifest-Version: 1.0 
	Premain-Class: Premain
```
在运行这个程序的时候，如果我们用普通方式运行这个 jar 中的 main 函数，可以得到输出“1”。如果用下列方式运行 :  
```
java -javaagent:TestInstrument1.jar -cp TestInstrument1.jar TestMainInJar
```
则会得到输出“2”。

当然，程序运行的 main 函数不一定要放在 premain 所在的这个 jar 文件里面，这里只是为了例子程序打包的方便而放在一起的。 

除了用 addTransformer 的方式，Instrumentation 当中还有另外一个方法“redefineClasses”来实现 premain 当中指定的转换。用法类似，如下：  
```Java
	public class Premain { 
	    public static void premain(String agentArgs， Instrumentation inst)  throws ClassNotFoundException， UnmodifiableClassException { 
	        ClassDefinition def = new ClassDefinition(TransClass.class， Transformer.getBytesFromFile(Transformer.classNumberReturns2)); 
	        inst.redefineClasses(new ClassDefinition[] { def }); 
	        System.out.println("success"); 
	    } 
	}
```
redefineClasses 的功能比较强大，可以批量转换很多类。  

# JDK6的新特性：虚拟机启动后的动态instrument

在 Java SE 5 当中，开发者只能在 premain 当中施展想象力，所作的 Instrumentation 也仅限与 main 函数执行前，这样的方式存在一定的局限性。

在 Java SE 5 的基础上，Java SE 6 针对这种状况做出了改进，开发者可以在main 函数开始执行以后，再启动自己的 Instrumentation 程序。

在 Java SE 6 的 Instrumentation 当中，有一个跟 premain“并驾齐驱”的“agentmain”方法，可以在 main 函数开始运行之后再运行。跟 premain 函数一样， 开发者可以编写一个含有“agentmain”函数的 Java 类： 
```Java
	public static void agentmain (String agentArgs, Instrumentation inst); //[1] 
	public static void agentmain (String agentArgs);                       //[2]
```
同样，[1] 的优先级比 [2] 高，将会被优先执行。跟 premain 函数一样，开发者可以在 agentmain 中进行对类的各种操作。其中的 agentArgs 和 Inst 的用法跟 premain 相同。

与“Premain-Class”类似，开发者必须在 manifest 文件里面设置“Agent-Class”来指定包含 agentmain 函数的类。
可是，跟 premain 不同的是，agentmain 需要在 main 函数开始运行后才启动，这样的时机应该如何确定呢，这样的功能又如何实现呢？

在 Java SE 6 文档当中，开发者也许无法在 java.lang.instrument 包相关的文档部分看到明确的介绍，更加无法看到具体的应用 agnetmain 的例子。不过，在 Java SE 6 的新特性里面，有一个不太起眼的地方，揭示了 agentmain 的用法。这就是Java SE 6 当中提供的 Attach API。

Attach API 不是 Java 的标准 API，而是 Sun 公司提供的一套扩展 API，用来向目标 JVM ”附着”（Attach）代理工具程序的。有了它，开发者可以方便的监控一个 JVM，运行一个外加的代理程序。

Attach API 很简单，只有 2 个主要的类，都在 com.sun.tools.attach 包里面：VirtualMachine 代表一个 Java 虚拟机，也就是程序需要监控的目标虚拟机，提供了 JVM 枚举，Attach 动作和 Detach 动作（Attach 动作的相反行为，从 JVM 上面解除一个代理）等等 ;VirtualMachineDescriptor 则是一个描述虚拟机的容器类，配合 VirtualMachine 类完成各种功能。

为了简单起见，我们举例简化如下：依然用类文件替换的方式，将一个返回 1 的函数替换成返回 2 的函数，Attach API 写在一个线程里面，用睡眠等待的方式，每隔半秒时间检查一次所有的 Java 虚拟机，当发现有新的虚拟机出现的时候，就调用 attach 函数，随后再按照 Attach API 文档里面所说的方式装载 Jar 文件。等到 5 秒钟的时候，attach 程序自动结束。而在 main 函数里面，程序每隔半秒钟输出一次返回值（显示出返回值从 1 变成 2）。

TransClass 类和 Transformer 类的代码不变，参看上一节介绍。 含有 main 函数的 TestMainInJar 代码为：
```Java
	public class TestMainInJar { 
	    public static void main(String[] args) throws InterruptedException { 
	        System.out.println(new TransClass().getNumber()); 
	        int count = 0; 
	        while (true) { 
	            Thread.sleep(500); 
	            count++; 
	            int number = new TransClass().getNumber(); 
	            System.out.println(number); 
	            if (3 == number || count >= 10) { 
	                break; 
	            } 
	        } 
	    } 
	}
```
含有 agentmain 的 AgentMain 类的代码为：
```Java
	import java.lang.instrument.ClassDefinition; 
	import java.lang.instrument.Instrumentation; 
	import java.lang.instrument.UnmodifiableClassException; 
	 
	public class AgentMain { 
	    public static void agentmain(String agentArgs, Instrumentation inst)  throws ClassNotFoundException, UnmodifiableClassException, InterruptedException { 
	        inst.addTransformer(new Transformer (), true); 
	        inst.retransformClasses(TransClass.class); 
	        System.out.println("Agent Main Done"); 
	    } 
	}
```
其中，retransformClasses 是 Java SE 6 里面的新方法，它跟 redefineClasses 一样，可以批量转换类定义，多用于 agentmain 场合。

Jar 文件跟 Premain 那个例子里面的 Jar 文件差不多，也是把 main 和 agentmain 的类，TransClass，Transformer 等类放在一起，打包为“TestInstrument1.jar”，而 Jar 文件当中的 Manifest 文件为 :
```
	Manifest-Version: 1.0 
	Agent-Class: AgentMain
```
另外，为了运行 Attach API，我们可以再写一个控制程序来模拟监控过程：（代码片段）
```Java
	import com.sun.tools.attach.VirtualMachine; 
	import com.sun.tools.attach.VirtualMachineDescriptor; 
	……
	// 一个运行 Attach API 的线程子类
	static class AttachThread extends Thread { 
	 
	        private final List<VirtualMachineDescriptor> listBefore; 
	 
	        private final String jar; 
	 
	        AttachThread(String attachJar, List<VirtualMachineDescriptor> vms) { 
	            listBefore = vms;  // 记录程序启动时的 VM 集合
	            jar = attachJar; 
	        } 
	 
	        public void run() { 
	            VirtualMachine vm = null; 
	            List<VirtualMachineDescriptor> listAfter = null; 
	            try { 
	                int count = 0; 
	                while (true) { 
	                    listAfter = VirtualMachine.list(); 
	                    for (VirtualMachineDescriptor vmd : listAfter) { 
	                        if (!listBefore.contains(vmd)) { 
	                            // 如果 VM 有增加，我们就认为是被监控的 VM 启动了
	                            // 这时，我们开始监控这个 VM 
	                            vm = VirtualMachine.attach(vmd); 
	                            break; 
	                        } 
	                    } 
	                    Thread.sleep(500); 
	                    count++; 
	                    if (null != vm || count >= 10) { 
	                        break; 
	                    } 
	                } 
	                vm.loadAgent(jar); 
	                vm.detach(); 
	            } catch (Exception e) { 
	                 ignore 
	            } 
	        } 
	    } 
	……
		public static void main(String[] args) throws InterruptedException {     
		     new AttachThread("TestInstrument1.jar", VirtualMachine.list()).start(); 
		}
	}
```
运行时，可以首先运行上面这个启动新线程的 main 函数，然后，在 5 秒钟内（仅仅简单模拟 JVM 的监控过程）运行如下命令启动测试 Jar 文件 :
```
java – javaagent:TestInstrument2.jar – cp TestInstrument2.jar TestMainInJar
```

如果时间掌握得不太差的话，程序首先会在屏幕上打出 1，这是改动前的类的输出，然后会打出一些 2，这个表示 agentmain 已经被 Attach API 成功附着到 JVM 上，代理程序生效了，当然，还可以看到“Agent Main Done”字样的输出。

以上例子仅仅只是简单示例，简单说明这个特性而已。真实的例子往往比较复杂，而且可能运行在分布式环境的多个 JVM 之中。