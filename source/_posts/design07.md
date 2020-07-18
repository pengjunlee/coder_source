---
title: 设计模式系列之--装饰模式
date: 2020-07-18 12:07:00
updated: 2020-07-18 12:07:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 装饰模式
type: 
description: 什么是装饰模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
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
# 一、什么是装饰模式

装饰（ Decorator ）模式又叫做包装( Wrapper )模式，它通过一种对客户端透明的方式来扩展对象的功能，是继承关系的一个替换方案。 

# 二、装饰模式实现

装饰模式通过一种对客户端透明的方式动态地为一个对象附加上新的功能，它可以使客户端在不需要创建更多子类的情况下，自由地对创建对象的功能进行扩展，装饰模式是继承关系的一个替换方案。 

举个简单的例子：沙发可以坐、可以躺，我们还可以为它添加上按摩功能使它成为保健沙发，我们还可以为它添加上可以展开供人睡觉的功能让它变成沙发床，接下来我们就使用子类继承方式来模拟为沙发添加上这些新功能。 

首先我们要定义一个沙发Sofa的接口用以声明所有沙发都应该具有的功能，此处额外增加了一个show()方法用来展示沙发的功能。  
```Java
	public interface Sofa { 
	    // 测试方法，用以展示沙发具有的功能 
	    public void show(); 
	    // 坐 
	    public void sit(); 
	    // 躺 
	    public void lie(); 
	}
```
接下来我们就要声明普通沙发CommonSofa、SofaBed和HealthCareSofa三种沙发了。 

普通沙发只能坐和躺。
```Java
		public class CommonSofa implements Sofa { 
		    @Override 
		    public void show() { 
		        this.sit(); this.lie(); 
		    } 
		    @Override public void sit() { 
		        System.out.println("可以坐"); 
		    } 
		    @Override public void lie() { 
		        System.out.println("可以躺"); 
		    } 
		}
```
沙发床除了能坐和躺外，还可以睡觉。
```Java
	public class SofaBed implements Sofa {
		@Override
		public void show() {
			this.sit();
			this.lie();
			this.sleep();
		}
	 
		@Override
		public void sit() {
			System.out.println("可以坐");
		}
	 
		@Override
		public void lie() {
			System.out.println("可以躺");
		}
	 
		public void sleep() {
			System.out.println("可以睡觉");
		}
	}
```
保健沙发除了坐和躺外，还可以按摩。
```Java
	public class HealthCareSofa implements Sofa {
		@Override
		public void show() {
			this.sit();
			this.lie();
			this.massage();
		}
	 
		@Override
		public void sit() {
			System.out.println("可以坐");
		}
	 
		@Override
		public void lie() {
			System.out.println("可以躺");
		}
	 
		public void massage() {
			System.out.println("可以按摩");
		}
	}
```
创建一个客户端来进行测试。
```Java
	public class Client {
		public static void main(String[] args) {
			System.out.println("---------普通沙发功能如下：---------");
			Sofa commonSofa = new CommonSofa();
			commonSofa.show();
			System.out.println();
			System.out.println("---------沙发床功能如下：---------");
			Sofa sofaBed = new SofaBed();
			sofaBed.show();
			System.out.println();
			System.out.println("---------保健沙发功能如下：---------");
			Sofa healthCareSofa = new HealthCareSofa();
			healthCareSofa.show();
		}
	}
```
运行程序打印结果如下：
```
---------普通沙发功能如下：---------
可以坐 可以躺
---------沙发床功能如下：---------
可以坐 可以躺 可以睡觉
---------保健沙发功能如下：---------
可以坐 可以躺 可以按摩
```
以上是通过继承来实现沙发Sofa功能的扩展，功能实现了但却有个严重的缺陷：通过继承实现的功能扩展是静态的，我们必须提前预知需要为对象添加哪些功能，并创建好相应的子类，对象能够扩展哪些功能这是在java虚拟机对类进行加载编译时就已经决定了的，因而是静态的。还是以沙发Sofa为例，如果我们需要创建一个既能睡觉又能按摩的保健沙发床对象时，我们就不得不再新添加一个Sofa的子类（保健沙发床）并为其附加上sleep()和massage()两个方法，倘若以后沙发又增加上可拆洗的功能，对各个功能进行组合我们就要再增加可拆洗沙发、可拆洗沙发床、可拆洗保健沙发、可拆洗保健沙发床四个子类，功能若继续增加势必造成子类数量的成倍增加，出现"类膨胀"情况。 

装饰模式的设计初衷：以一种更为灵活的方式动态地为对象附加一些新功能，装饰模式的实现结构图如下： 
<div align=center>

![装饰模式结构示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/3.png "装饰模式结构示意图")
<div align=left>
装饰模式涉及的角色及其职责如下： 

- 抽象组件(Component)角色： 一个抽象接口，是被装饰类（ConcreteComponent）和装饰类（Decorator）的父接口。 
- 具体组件(ConcreteComponent)角色：抽象组件的实现类。
- 抽象装饰(Decorator)角色：包含一个组件的引用，并定义了与抽象组件一致的接口。
- 具体装饰(ConcreteDecorator)角色：为抽象装饰角色的实现类，负责具体的装饰(为组件附加上新的功能)。 

接下来我们使用装饰模式来实现沙发的功能扩展，首先定义一个沙发Sofa的接口同上，担当装饰模式中的抽象组件角色。
```Java
	public interface Sofa {
	 
		// 测试方法，用以展示沙发所具有的功能
		public void show();
	 
		// 坐
		public void sit();
	 
		// 躺
		public void lie();
	}
```
再定义一个Sofa接口的实现类，扮演装饰模式中的具体组件角色。 
```Java
	public class CommonSofa implements Sofa {
		@Override
		public void show() {
			this.sit();
			this.lie();
		}
	 
		@Override
		public void sit() {
			System.out.println("可以坐");
		}
	 
		@Override
		public void lie() {
			System.out.println("可以躺");
		}
	}
```
再定义一个抽象装饰角色，其内部要包含一个被装饰对象的引用，并实现抽象组件的所有接口
```Java
	public class SofaDecorator implements Sofa {
		// 私有的被装饰对象的引用
		private Sofa sofa; 
	    
	        // 公有的获取被装饰对象的方法
		public Sofa getSofa() {
			return sofa;
		}
	 
		public void setSofa(Sofa sofa) {
			this.sofa = sofa;
		}
	 
		public SofaDecorator(Sofa sofa) {
			this.sofa = sofa;
		}
	 
		public void show() {
			this.sit();
			this.lie();
		}
	 
		@Override
		public void sit() {
			System.out.println("可以坐");
		}
	 
		@Override
		public void lie() {
			System.out.println("可以躺");
		}
	}
```
最后创建具体装饰角色，为被装饰对象附件上新功能
```Java
	public class BedDecorator extends SofaDecorator {
		public BedDecorator(Sofa sofa) {
			super(sofa);
		}
	 
		public void show() {
			this.getSofa().show();
			this.sleep();
		} // 为沙发添加睡觉功能
	 
		public void sleep() {
			System.out.println("可以睡觉");
		}
	}
```
<br/>
```Java
	public class HealthCareDecorator extends SofaDecorator {
		public HealthCareDecorator(Sofa sofa) {
			super(sofa);
		}
		public void show() {
			this.getSofa().show();
			this.massage();
		}
		// 为沙发添加按摩功能
		public void massage() {
			System.out.println("可以按摩");
		}
	}
```
创建一个客户端测试一下，其代码如下：
```Java
	public class MainClass {
		public static void main(String[] args) { // 先创建一个要被装饰的普通沙发对象
			Sofa sofa = new CommonSofa();
			System.out.println("---------普通沙发功能如下：---------");
			sofa.show();
			System.out.println(); // 用BedDecorator对创建好的普通沙发对象进行装饰，为其增加睡觉功能
			System.out.println("---------沙发床功能如下：---------");
			Sofa sofaBed = new BedDecorator(sofa);
			sofaBed.show();
			System.out.println(); // 用HealthCareDecorator对创建好的普通沙发对象进行装饰，为其增加按摩功能
			System.out.println("---------保健沙发功能如下：---------");
			Sofa healthCareSofa = new HealthCareDecorator(sofa);
			healthCareSofa.show();
			System.out.println(); // 用HealthCareDecorator对创建好的沙发床对象进行装饰，为其增加按摩功能
			System.out.println("---------保健沙发床功能如下：---------");
			Sofa healthCareSofaBed = new HealthCareDecorator(sofaBed);
			healthCareSofaBed.show();
		}
	}
```
运行程序打印结果如下：
```Java
---------普通沙发功能如下：---------
可以坐 可以躺
---------沙发床功能如下：---------
可以坐 可以躺 可以睡觉
---------保健沙发功能如下：---------
可以坐 可以躺 可以按摩
---------保健沙发床功能如下：---------
可以坐 可以躺 可以睡觉 可以按摩
```
装饰模式通过对普通沙发的功能进行层层装饰，最终连具有保健和睡觉功能的沙发也创建成功了。 

# 三、装饰模式的特点

- 装饰对象和真实对象有相同的接口。
- 装饰对象包含一个真实对象的引用（reference） 
- 装饰对象接受所有的来自客户端的请求。它把这些请求转发给真实的对象。
- 装饰对象可以在转发这些请求以前或以后增加一些附加功能。这样就确保了在运行时，不用修改给定对象的结构就可以在外部增加附加的功能。  

# 四、参考文章

[http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html](http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html "《JAVA与模式》之装饰模式")