---
title: 深入理解JVM系列之--javac命令
date: 2020-07-18 13:02:00
updated: 2020-07-18 13:02:00
tags: Java虚拟机
categories: 深入理解JVM
keywords: Java, javac, JVM
type: 
description: 初识javac编译器。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
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
javac命令用于将 .java 源文件编译成 .class 字节码文件，在windows命令行中使用"javac -help"命令查看其用法： 
<div align=center>

![javac命令用法](http://pengjunlee.3vzhuji.net/static/jvm/1.png "javac命令用法")
<div align=left>
```Bash
	语法格式: javac <options> <source files>
	options                       # 命令行选项，可指定多个，多个选项可按任意顺序排列
	source files                  # 用于指定一个或多个要编译的 .java 源文件
```

# 一、指定编译源文件 
如果需要编译的源文件数量较少，可以直接在命令行上将所有文件名（必要时包含路径）列出，多个文件名之间用空格分隔。
```Bash
	javac src/FirstClass.java src/SecondClass.java src/ThirdClass.java # 例1
```
> <font color=red>注意：如果文件路径中包含有空格，需用双引号把该文件名括起来。</font> 
```Bash
	javac "src/Program Files/MyClass.java"
```
在没使用分号的情况下，对相同路径下的多个 .java 源码文件进行编译，可以使用`*`通配符，此时例1可以写成如下形式：
```Bash
	javac src/*.java
	javac src/*Class.java
```
如果需要编译的 .java 源文件数量较多，为缩短和简化javac命令，可以把要编译的 .java 源文件的文件名（必要时包含路径）存储到一个文件中，多个 .java 源文件名之间用空格或回车进行分隔。然后在javac命令行中，用'@' 字符指定该文件。

比如，我们把例1中要编译的 .java 源码文件名写到classes.txt文件中，classes.txt文件内容如下： 
```
	src\FirstClass.java src\SecondClass.java
	src\ThirdClass.java
```
我们也可以在classes.txt中用双引号把单个要编译的 .java 源文件名括起来，但是这时路径之间接的分隔符“\”就要写成"\\"的形式了。classes.txt文件内容如下： 
```
	"src\\FirstClass.java" "src\\SecondClass.java"
	"src\\ThirdClass.java"
```
此时，执行如下的javac命令：
```Bash
	javac @classes.txt
```

# 二、命令选项
```Bash
	-d <目录> # 指定放置生成的类文件的位置
```
该选项用于指定生成的.class文件存放的位置。如果某个类是一个包的组成部分，则javac将把生成的.class文件放入反映包名的子目录中，必要时创建目录。 
```Java
	package com.pengjunlee;
		public class MyClass {
	}
```
使用命令：**`javac -d bin src\MyClass.java`**，对以上MyClass.java文件进行编译，将会将生成的MyClass.class文件存放到 `bin\com\pengjunlee` 目录下。

若未指定 -d 选项，则 javac 将把生成的 .class 文件放到与 .java 源文件相同的目录中。
**.class类文件的搜索路径**
```Bash
	-bootclasspath <路径>                   覆盖引导类文件的位置
	-extdirs <路径>                         覆盖所安装扩展的位置
	-classpath <路径>                       指定查找用户类文件和注释处理程序的位置
	-cp <路径>                              指定查找用户类文件和注释处理程序的位置
```
JDK在编译一个java源文件时，搜索依赖的.class类文件的顺序如下：
```Bash
	Bootstrap classes-->Extension classes-->User classes
```
- Bootstrap classes 默认的是JDK自带的jar或zip文件，它包括jre\lib下rt.jar等文件，JDK首先搜索这些文件。可以通过-bootclasspath来设置它，文件之间用分号";"进行分隔。
- Extension classes 默认的是位于jre\lib\ext目录下的jar文件，JDK在搜索完Bootstrap后就搜索该目录下的jar文件.可以通过-extdirs来设置。文件之间用分号";"来进行分隔。
- User classes 搜索顺序为当前目录、环境变量 CLASSPATH、-classpath。

-cp 和 -classpath 是同义词，参数意义是一样的。classpath参数太长了，所以提供cp作为缩写形式。它们用于告知JDK搜索目录名、jar文档名、zip文档名，用分号";"进行分隔。例如当你自己开发了公共类并包装成一个`common.jar`包，在使用 `common.jar`中的类时，就需要用`-classpath common.jar `告诉JDK从`common.jar`中查找该类，否则JDK就会抛出`java.lang.NoClassDefFoundError`异常，表明未找到类定义。

使用-classpath后JDK将不再使用CLASSPATH中的类搜索路径，如果-classpath和CLASSPATH都没有设置，则JDK使用当前路径(.)作为类搜索路径。

推荐使用-classpath来定义JDK要搜索的类路径，而不要使用环境变量 CLASSPATH的搜索路径，以减少多个项目同时使用CLASSPATH时存在的潜在冲突。例如应用1要使用`a1.0.jar`中的类G，应用2要使用`a2.0.jar`中的类G,`a2.0.jar`是`a1.0.jar`的升级包，当`a1.0.jar`，`a2.0.jar`都在CLASSPATH中，JDK搜索到第一个包中的类G时就停止搜索，如果应用1应用2的虚拟机都从CLASSPATH中搜索，就会有一个应用得不到正确版本的类G。 
```Bash
	javac -cp bin -d bin MyClass.java
	javac -classpath bin -d bin MyClass.java
```
如果需要指定各个JAR文件具体的存放路径，相同路径有多个可使用通配符。
```Bash
	-sourcepath <路径> # 指定查找输入源文件的位置
```
在编译时，JDK需要两方面的路径，一个是查找java源码文件的路径，一个是查找 .class（类）文件的路径。

关于.class文件的路径上文已经已经介绍过，可以通过`-bootclasspath`，`-extdirs`，`-classpath`和`-cp`来设定。java源码文件的路径则可以通过`-sourcepath`来设定，默认情况下`-sourcepath`和`-classpath`的路径一样。在编译的过程中，若需要相关java类的则首先在sourcefiles或@files列出的java源码文件中查找并编译，如果没找到，就在-`sourcepath`指定的路径中查找java源码文件，这时无论找没找到都会继续在类路径中进行查找。如果在sourcepath中找到了java源码文件，但是在类路径中没有找到了相关的类，或找的类位于包文件（jar或zip）中,或找的类并不是在包文件中，但源码文件比该类文件新，这时会对源码文件进行编译，而且编译生成的类文件将会和你指定要进行编译的java源码所生成的类文件位于同一根目录。否则，除了既没找到java源码文件也没找到相关类就编译失败外，直接载入相关类就可以了。因此你得至少要指定一个要编译的java源文件。它并不是指定sourecfiles或@files中指定的要编译的java源码文件的根目录。与类路径一样，java源码路径项用分号 (;) 进行分隔，它们可以是class文件的根目录、JAR 归档文件或 ZIP 归档文件。
```Bash
	javac -sourcepath src -d bin MyClass.java
```
<br/>
```Bash
	-source <发行版> # 提供与指定发行版的源兼容性
```
当你从sun安装了某个版本的JDK，而其实该JDK却包含多个版本的编译器。-source参数就是指定用哪个版本的编译器对java源码进行编译。如果你的java源码不符合该版本编译器的规范的话，当然就不能编译通过。
```Bash
	-target <发行版> # 生成特定 VM 版本的类文件
```
-target 命令用于指定生成的class文件将保证和哪个版本的虚拟机进行兼容。我们可以通过-target 1.2来保证生成的class文件能在1.2虚拟机上进行运行，但是1.1的虚拟机就不能保证了。因为java虚拟机的向前兼容行，1.5的虚拟机当然也可以运行通过-target 1.2生成的class文件。每个版本编译器的默认-target版本是不太一样的。
<br/>
```Bash
	javac -source 1.2 -target 1.1 -sourcepath src -d bin MyClass.java
	javac -source 1.2 -target 1.5 -sourcepath src -d bin MyClass.java
```
<br/>
```Bash
	-deprecation # 输出使用已过时的 API 的源位置
```
如果java源码中使用了不鼓励使用的类或方法，那么如果使用了该参数，将显示关于此警告的详细信息,否则只有个简单的Note.
```Java
	public class MyClass {
	 
		public static void main(String[] args) {
			String str=new String(new byte[3],100);
		}
	}
```
<br/>
<div align=center>

![javac命令用法](http://pengjunlee.3vzhuji.net/static/jvm/3.png "javac命令用法")
<div align=left>
```Bash
	-verbose  #　输出有关编译器正在执行的操作的消息
```
使用该参数，你可以看到编译器编译java源文件的详细过程。 

<div align=center>

![javac命令用法](http://pengjunlee.3vzhuji.net/static/jvm/4.png "javac命令用法")
<div align=left>
```Bash
	-encoding<编码> ＃　指定源文件使用的字符编码
```
设置源文件编码名称，例如UTF-8。若未指定 -encoding 选项，则使用平台缺省的编码方式。
```Bash
	-g ＃　生成所有调试信息
```
生成所有的调试信息，包括局部变量。缺省情况下，只生成行号和源文件信息。
```Bash
	-g:none 　　　　　　　　　　　　＃　不生成任何调试信息
	-g:{lines,vars,source} 　　　　＃　只生成某些调试信息
	-nowarn 　　　　　　　　　　　　＃　禁用警告信息。 
```

# 三、非标准选项 -X
使用该参数，可以显示所有的非标准选项的有关信息。 

<div align=center>

![javac命令用法](http://pengjunlee.3vzhuji.net/static/jvm/5.png "javac命令用法")
<div align=left>
```Bash
	-Xlint                     ＃　启用建议的警告
	-Xlint:{all,none，其他选项} ＃　启用或禁用特定的警告
```
通过该命令我们将看到你java源码文件的一些危险代码，关键字有：
```Bash
	{all,auxiliaryclass,cast,classfile,deprecation,dep-ann,divzero,empty,fallthrough,finally,options,overloads,
	overrides,path,processing,rawtypes,serial,static,try,unchecked,varargs,
	-auxiliaryclass,-cast,-classfile,-deprecation,-dep-ann,-divzero,-empty,-fallthrough,-finally,-options,-overloads,-overrides,-path,-processing,-rawtypes,-serial,-static,-try,-unchecked,-varargs,none}
```
没有"-"前缀的表示开启，有的该前缀的表示关闭，all表示开启所有，none表示都不开启。
```Java
	public class MyClass {
	 
		public static void main(String[] args) {
			String str=new String(new byte[3],100);
		}
	}
```
<br/>
<div align=center>

![javac命令用法](http://pengjunlee.3vzhuji.net/static/jvm/6.png "javac命令用法")
<div align=left>
```Bash
	-Xstdout <文件名>            ＃ 重定向标准输出
```
javac命令执行信息默认将在当前控制台进行显示，我们可以用该参数进行重新定义。比如将前一示例的编译过程信息输出到"stdout.log"文件中：
```Bash
	javac -Xstdout stdout.log -Xlint:all MyClass.java
```
命令执行完成后，查看stdout.log中内容如下： 

<div align=center>

![javac命令用法](http://pengjunlee.3vzhuji.net/static/jvm/7.png "javac命令用法")
<div align=left>
```Bash
	-Xmaxerrs <编号>           ＃  设置要输出的错误的最大数目
	-Xmaxwarns <编号>          ＃  设置要输出的警告的最大数目
```