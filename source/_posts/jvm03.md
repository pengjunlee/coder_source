---
title: 深入理解JVM系列之--class类文件
date: 2020-07-18 13:03:00
updated: 2020-07-18 13:03:00
tags: Java虚拟机
categories: 深入理解JVM
keywords: Java, javac, class, JVM
type: 
description: 初识class类文件。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
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
> <font color=red >参考书籍：《深入理解Java虚拟机——JVM高级特性与最佳实践(第2版)》</font>

Java在刚刚诞生之时曾经提出过一个非常著名的宣传口号：`一次编写，到处运行（Write Once,Run Anywhere）`，这句话充分表达了软件开发人员对冲破平台界限的渴求。而实现这个“与平台无关”理想的基础就是虚拟机和字节码（ByteCode）格式存储。 

Java虚拟机不和包括Java在内的任何语言绑定，它只与“Class文件”这种特定的二进制文件格式所关联，Class 文件中包含了 Java 虚拟机指令集（或者称为字节码、 Bytecodes）和符号表，还有一些其他辅助信息。使用不同语言所编写的代码只要能够被正确编译为符合虚拟机规范要求的Class文件，虚拟机就能够执行它，虚拟机并不关心Class文件的来源是何种语言。  
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/8.png "类文件结构示意图")
<div align=left>

# 一、Class类文件的结构

Class文件格式所具备的平台中立（不依赖于特定硬件及操作系统）、紧凑、稳定和可扩展的特点，是Java技术体系实现平台无关、 语言无关两项特性的重要支柱。

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，中间没有添加任何分隔符，这使得整个Class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。 当遇到需要占用8位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8位字节进行存储。

根据Java虚拟机规范的规定，Class文件格式采用一种类似于C语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：`无符号数`和`表`。

- 无符号数属于基本的数据类型，以u1、 u2、 u4、 u8来分别代表1个字节、 2个字节、 4个字节和8个字节的无符号数，无符号数可以用来描述数字、 索引引用、 数量值或者按照UTF-8编码构成字符串值。
- 表是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以“_info”结尾，用于描述有层次关系的复合结构的数据。

整个Class文件本质上就是一张表，它由下表所示的数据项构成。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/9.png "类文件结构示意图")
<div align=left>

样例源码：  
```Java
	package jvm;
	 
	public class SimpleClass implements Comparable<SimpleClass> {
	 
		private static final int magic = 0xCAFEBABE;
	 
		private int number;
	 
		public void setNumber(int number) {
			this.number = number;
		}
	 
		public int compareTo(SimpleClass o) {
			if (this.number == o.number) {
				return 0;
			}
			int ret = this.number > o.number ? 1 : -1;
			return ret;
		}
	 
	}
```
将以上代码使用JDK 1.6编译输出的Class文件在UltraEdit中打开，其前64个字节的内容如下图： 
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/10.png "类文件结构示意图")
<div align=left>

# 二、魔数与Class文件的版本

每个Class文件的头4个字节称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。 很多文件存储标准中都使用魔数来进行身份识别，譬如图片格式，如gif或者jpeg等在文件头中都存有魔数。 使用魔数而不是扩展名来进行识别主要是基于安全方面的考虑，因为文件扩展名可以随意地改动。 如上图所示，Class文件的魔数值为：`0xCAFEBABE`。

紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6个字节是次版本号（Minor Version，图中为`0x0000`），第7和第8个字节是主版本号（Major Version，图中为`0x0031`）。

# 三、常量池

由于常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数值（constant_pool_count）。 与Java中语言习惯不一样的是，这个容量计数是从1而不是0开始的，如上图所示，常量池容量（偏移地址：`0x00000008`）为十六进制数**0x0024**，即十进制的**36**，这就代表常量池中有**35**项常量，索引值范围为**1～35**。 在Class文件格式规范制定之时，设计者将第0项常量空出来是有特殊考虑的，这样做的目的在于满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义，这种情况就可以把索引值置为0来表示。Class文件结构中只有常量池的容量计数是从1开始，对于其他集合类型，包括接口索引集合、 字段表集合、 方法表集合等的容量计数都与一般习惯相同，是从0开始的。

常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。字面量比较接近于Java语言层面的常量概念，如文本字符串、 声明为final的常量值等。 而符号引用则属于编译原理方面的概念，包括了下面三类常量：

- 类和接口的全限定名（Fully Qualified Name）
- 字段的名称和描述符（Descriptor）
- 方法的名称和描述符


常量池中每一项常量都是一个表，从JDK 1.7开始，共分为以下14种表类型。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/11.png "类文件结构示意图")
<div align=left>
还是以上面编译的class文件的UltraEdit截图进行示例。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/10.png "类文件结构示意图")
<div align=left>
UltraEdit截图中常量池的第一项常量的标志位（偏移地址：`0x0000000a`）是`0x07`，表示这个常量属于CONSTANT_Class_info类型，此类型的常量代表一个类或者接口的符号引用。 CONSTANT_Class_info的结构比较简单，见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/12.png "类文件结构示意图")
<div align=left>

tag是标志位，用于区分常量类型；name_index是一个索引值，它指向常量池中一个CONSTANT_Utf8_info类型常量，此常量代表了这个类（或者接口）的全限定名，这里name_index值（偏移地址：`0x0000000b`）为`0x0002`，也即是指向了常量池中的第二项常量。 继续从UltraEdit截图中查找第二项常量，它的标志位（地址：`0x0000000d`）是`0x01`，表示这个常量属于CONSTANT_Utf8_info类型，CONSTANT_Utf8_info类型的结构见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/13.png "类文件结构示意图")
<div align=left>

length值说明了这个UTF-8编码的字符串长度是多少字节，它后面紧跟着的长度为length字节的连续数据是一个使用UTF-8缩略编码表示的字符串。 UTF-8缩略编码与普通UTF-8编码的区别是：

- 从'\u0001'到'\u007f'之间的字符（相当于1～127的ASCII码）的缩略编码使用一个字节表示；
- 从'\u0080'到'\u07ff'之间的所有字符的缩略编码用两个字节表示；
- 从'\u0800'到'\uffff'之间的所有字符的缩略编码就按照普通UTF-8编码规则使用三个字节表示。


本例中这个字符串的length值（偏移地址：`0x0000000e`）为`0x000F`，也就是长15字节，往后15字节正好都在1～127的ASCII码范围以内，内容为“jvm/SimpleClass”，换算结果如下图选中的部分所示。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/14.png "类文件结构示意图")
<div align=left>


其余的34个常量，我们可以通过类似的方法计算出来，或使用javap工具帮助我们进行计算，以下是使用javap工具的-verbose参数输出的SimpleClass.class的常量池内容。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/15.png "类文件结构示意图")
<div align=left>


常量池中14种常量项的结构总表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/16.png "类文件结构示意图")
<div align=left>

# 四、访问标志

在常量池结束之后，紧接着的两个字节代表访问标志（access_flags），这个标志用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被声明为final等。 具体的标志位以及标志的含义下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/17.png "类文件结构示意图")
<div align=left>

access_flags中一共有16个标志位可以使用，当前只定义了其中8个，没有使用到的标志位要求一律为0。UltraEdit截图中的access_flags标志（偏移地址：`0x000001b2`）值为`0x0021`，代表SimpleClass是一个public访问权限的普通java类。

# 五、类索引、 父类索引与接口索引集合

类索引（this_class）和父类索引（super_class）都是一个u2类型的数据，而接口索引集合（interfaces）是一组u2类型的数据的集合，Class文件中由这三项数据来确定这个类的继承关系。 类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。 由于Java语言不允许多重继承，所以父类索引只有一个，除了`java.lang.Object`之外，所有的Java类都有父类，因此除了`java.lang.Object`外，所有Java类的父类索引都不为**0**。 接口索引集合就用来描述这个类实现了哪些接口，这些被实现的接口将按implements语句（如果这个类本身是一个接口，则应当是extends语句）后的接口顺序从左到右排列在接口索引集合中。

类索引、 父类索引和接口索引集合都按顺序排列在访问标志之后，类索引和父类索引用两个u2类型的索引值表示，它们各自指向一个类型为`CONSTANT_Class_info`的类描述符常量，通过`CONSTANT_Class_info`类型的常量中的索引值可以找到定义在`CONSTANT_Utf8_info`类型的常量中的全限定名字符串。

 SimpleClass类仅实现了一个Comparable接口，其接口计数器偏移地址：`0x00001b07`，其后紧跟`0x0005`指向一个CONSTANT_Class_info类型的索引值，从常量#6可以看出，这个索引指向的就是Comparable接口。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/18.png "类文件结构示意图")
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/19.png "类文件结构示意图")
<div align=left>


# 六、字段表集合

字段表（field_info）用于描述接口或者类中声明的变量。 字段（field）包括类级变量以及实例级变量，但不包括在方法内部声明的局部变量。
字段表结构如下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/20.png "类文件结构示意图")
<div align=left>
其中access_flag可以设置的访问标志位和含义见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/21.png "类文件结构示意图")
<div align=left>
跟随access_flags标志的是两项索引值：name_index和descriptor_index。 它们都是对常量池的引用，分别代表着字段的简单名称以及字段和方法的描述符。

所谓全限定，仅仅是把类全名中的`.`替换成了`/`而已，例如：“java/lang/Comparable”，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个`；`表示全限定名结束。 简单名称是指没有类型和参数修饰的方法或者字段名称，例如，SimpleClass类中的setNumber()方法和magic字段的简单名称分别是“setNumber”和“magic”。

描述符的作用是用来描述字段的数据类型、 方法的参数列表（包括数量、 类型以及顺序）和返回值。 
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/22.png "类文件结构示意图")
<div align=left>

对于数组类型，每一维度将使用一个前置的`[`字符来描述，如一个定义为`java.lang.String[][]`类型的二维数组，将被记录为：`[[Ljava/lang/String；`，一个整型数组`int[]`将被记录为`[I`。

用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号`（）`之内。 如方法void inc()的描述符为`（）V`，方法java.lang.String toString()的描述符为`（）Ljava/lang/String；`，方法int indexOf（char[]source,int sourceOffset,int sourceCount,char[] target,int targetOffset,int targetCount,int fromIndex）的描述符为`（[CII[CIII）I`。

对于SimpleClass.class文件来说，字段表集合从地址`0x000001bb`开始，第一个u2类型的数据为容量计数器fields_count，如图所示，其值为`0x0002`，说明这个类有两个字段表数据。 接下来紧跟着容量计数器的是第一个字段的access_flags标志，值为`0x001A`，代表该字段被private+static+final修饰。 代表字段名称的name_index的值为`0x0007`，从常量池中可查得第7项常量名为`magic`，代表字段描述符的descriptor_index的值为`0x0008`，指向常量池的字符串`I`，即该字段为int类型。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/23.png "类文件结构示意图")
<div align=left>
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/24.png "类文件结构示意图")
<div align=left>

字段表集合中不会列出从超类或者父接口中继承而来的字段，但有可能列出原本Java代码之中不存在的字段，譬如在内部类中为了保持对外部类的访问性，会自动添加指向外部类实例的字段。

# 七、方法表集合

Class文件存储格式中对方法的描述与对字段的描述几乎采用了完全一致的方式，方法表的结构如同字段表一样，依次包括了访问标志（access_flags）、 名称索引（name_index）、 描述符索引（descriptor_index）、 属性表集合（attributes）几项，见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/25.png "类文件结构示意图")
<div align=left>

对于方法表，所有标志位及其取值可参见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/26.png "类文件结构示意图")
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/97.png "类文件结构示意图")
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/98.png "类文件结构示意图")
<div align=left>

对于SimpleClass.class文件来说，方法表集合的入口地址为：`0x00001d04`，第一个u2类型的数据（即是计数器容量）的值为`0x0004`，代表集合中有四个方法（这四个方法为编译器添加的实例构造器＜init＞、Comparable接口的构造方法以及源码中的方法compareTo()和setNumber()方法）。 第一个方法的访问标志值为`0x001`，也就是只有ACC_PUBLIC标志为真，名称索引值为`0x000C`，从常量池中可查得第12项常量池的方法名为“`＜init＞`，描述符索引值为`0x000D`，对应常量为`（）V`，属性表计数器attributes_count的值为`0x0001`就表示此方法的属性表集合有一项属性，属性名称索引为`0x000E`，对应常量为`Code`，说明此属性是方法的字节码描述。

与字段表集合相对应的，如果父类方法在子类中没有被重写（Override），方法表集合中就不会出现来自父类的方法信息。 但同样的，有可能会出现由编译器自动添加的方法，最典型的便是类构造器“＜clinit＞”方法和实例构造器“＜init＞”方法。

# 八、属性表集合

属性表（attribute_info）在前面的讲解之中已经出现过数次，在Class文件、 字段表、 方法表都可以携带自己的属性表集合，以用于描述某些场景专有的信息。

属性表的预定义属性目前共有21项，例如：Code、ConstantValue、LocalVariableTable、LineNumberTable。

## Code属性

Java程序方法体中的代码经过Javac编译器处理后，最终变为字节码指令存储在Code属性内。 Code属性出现在方法表的属性集合之中，但并非所有的方法表都必须存在这个属性，譬如接口或者抽象类中的方法就不存在Code属性，如果方法表有Code属性存在，那么它的结构将如下表所示。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/27.png "类文件结构示意图")
<div align=left>

attribute_name_index是一项指向CONSTANT_Utf8_info型常量的索引，常量值固定为“Code”，它代表了该属性的属性名称，attribute_length指示了属性值的长度，由于属性名称索引与属性长度一共为6字节，所以属性值的长度固定为整个属性表长度减去6个字节。

max_stack代表了操作数栈（Operand Stacks）深度的最大值。 在方法执行的任意时刻，操作数栈都不会超过这个深度。 虚拟机运行的时候需要根据这个值来分配栈帧（Stack Frame）中的操作栈深度。

max_locals代表了局部变量表所需的存储空间。 在这里，max_locals的单位是Slot,Slot是虚拟机为局部变量分配内存所使用的最小单位。 对于byte、 char、 float、 int、 short、 boolean和returnAddress等长度不超过32位的数据类型，每个局部变量占用1个Slot，而double和long这两种64位的数据类型则需要两个Slot来存放。 方法参数（包括实例方法中的隐藏参数“this”）、 显式异常处理器的参数（Exception Handler Parameter，就是try-catch语句中catch块所定义的异常）、 方法体中定义的局部变量都需要使用局部变量表来存放。 另外，并不是在方法中用到了多少个局部变量，就把这些局部变量所占Slot之和作为max_locals的值，原因是局部变量表中的Slot可以重用，当代码执行超出一个局部变量的作用域时，这个局部变量所占的Slot可以被其他局部变量所使用，Javac编译器会根据变量的作用域来分配Slot给各个变量使用，然后计算出max_locals的大小。

code_length和code用来存储Java源程序编译后生成的字节码指令。 code_length代表字节码长度，code是用于存储字节码指令的一系列字节流。 

其中异常表的结构如下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/28.png "类文件结构示意图")
<div align=left>

## Exceptions属性
Exceptions属性的作用是列举出方法中可能抛出的受查异常（Checked Excepitons），也就是方法描述时在throws关键字后面列举的异常。 它的结构见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/29.png "类文件结构示意图")
<div align=left>

## LineNumberTable属性
LineNumberTable属性用于描述Java源码行号与字节码行号（字节码的偏移量）之间的对应关系。 它的结构见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/30.png "类文件结构示意图")
<div align=left>

## LocalVariableTable属性
LocalVariableTable属性用于描述栈帧中局部变量表中的变量与Java源码中定义的变量之间的关系。 它的结构见下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/31.png "类文件结构示意图")
<div align=left>
其中，local_variable_info项目代表了一个栈帧与源码中的局部变量的关联，结构下表。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/32.png "类文件结构示意图")
<div align=left>

## ConstantValue属性
ConstantValue属性的作用是通知虚拟机自动为静态变量赋值。 只有被static关键字修饰的变量（类变量）才可以使用这项属性。
<div align=center>

![类文件结构示意图](http://pengjunlee.3vzhuji.net/static/jvm/33.png "类文件结构示意图")
<div align=left>
