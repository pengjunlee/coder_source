---
title: 设计模式系列之--访问者模式
date: 2020-07-18 12:15:00
updated: 2020-07-18 12:15:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 访问者模式
type: 
description: 什么是访问者模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
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
# 一、什么是访问者模式

访问者（Visitor）模式是一种对象的行为模式。在访问者模式里，每个访问者表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

在面向对象的系统开发和设计过程中，经常遇到的一种情况就是需求变更，针对已经开发完成的系统，客户又会提出新的需求。因此，我们不得不去修改已有的设计，最常见就是解决方案就是给已经设计、实现好的类添加新的方法去实现客户新的需求，这样就陷入了设计变更的梦魇：不停地打补丁，其最大的问题就是设计根本就不可能封闭。

访问者模式则为这种情况提供了一种解决方案：将更新（变更）封装到一个类中（访问操作），并由待更改类提供一个接收接口，则可达到效果。 

# 二、访问者模式的结构
  
<div align=center>

![访问者模式结构示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/16.png "访问者模式结构示意图")
<div align=left>

访问者模式涉及的角色及其职责如下：

- 抽象访问者(Visitor)角色：为该对象结构中具体元素角色声明一个访问操作接口，用来代表为对象结构添加的新功能，理论上可以代表任意的功能。
- 具体访问者(ConcreteVisitor)角色：实现每个由抽象访问者角色（Visitor）声明的操作接口。
- 抽象元素(Element)角色：定义一个accept()操作，它以一个访问者（Visitor）对象作为参数。
- 具体元素(ConcreteElement)角色：实现由抽象元素(Element)角色提供的accept()操作。
- 对象结构(ObjectStructure)角色：这是使用访问者模式必备的角色。一般具备以下特征：
	
	1. 能枚举它的元素。  
	2. 可以提供一个高层的接口以允许该访问者访问它的元素。
	3. 可以是一个复合（组合模式）或是一个集合，如一个列表或一个无序集合。

访问者模式结构示意源代码如下：

（1）首先定义一个接口来代表要新加入的功能，把它称作访问者（Visitor），访问谁呢？当然是访问对象结构中的对象了。 
```Java
	/** 
	 * 访问者接口 
	 */  
	public interface Visitor {  
	  
	    /** 
	     * 访问ConcreteElementA，相当于为ConcreteElementA添加的新功能 
	     */  
	    public void visitConcreteElementA(ConcreteElementA elementA);  
	  
	    /** 
	     * 访问ConcreteElementB，相当于为ConcreteElementB添加的新功能 
	     */  
	    public void visitConcreteElementB(ConcreteElementB elementB);  
	  
	}  
```
（2）再看看访问者的具体实现(ConcreteVisitor)。
```Java
	public class ConcreteVisitorA implements Visitor {  
	  
	    @Override  
	    public void visitConcreteElementA(ConcreteElementA elementA) {  
	        /** 
	         * 把访问ConcreteElementA时，需要执行的功能在这里进行实现 可能需要访问元素已有的功能，比如：operationA() 
	         */  
	        System.out.println("ConcreteVisitorA 访问 ==> ConcreteElementA 对象。");  
	  
	    }  
	  
	    @Override  
	    public void visitConcreteElementB(ConcreteElementB elementB) {  
	        /** 
	         * 把访问ConcreteElementB时，需要执行的功能在这里进行实现 可能需要访问元素已有的功能，比如：operationB() 
	         */  
	        System.out.println("ConcreteVisitorA 访问 ==> ConcreteElementB 对象。");  
	  
	    }  
	  
	}
```
<br/>
```Java
	public class ConcreteVisitorB implements Visitor {  
	  
	    @Override  
	    public void visitConcreteElementA(ConcreteElementA elementA) {  
	        /** 
	         * 把访问ConcreteElementA时，需要执行的功能在这里进行实现 可能需要访问元素已有的功能，比如：operationA() 
	         */  
	        System.out.println("ConcreteVisitorB 访问 ==> ConcreteElementA 对象。");  
	    }  
	  
	    @Override  
	    public void visitConcreteElementB(ConcreteElementB elementB) {  
	        /** 
	         * 把访问ConcreteElementB时，需要执行的功能在这里进行实现 可能需要访问元素已有的功能，比如：operationB() 
	         */  
	        System.out.println("ConcreteVisitorB 访问 ==> ConcreteElementB 对象。");  
	  
	    }  
	  
	}
```
（3）抽象元素(Element)的定义。 
```Java
	public abstract class Element {  
	  
	    // 接受访问者的访问  
	    public abstract void accept(Visitor visitor);  
	  
	}  
```
（4）再看看元素对象的具体实现(ConcreteElement)。 
```Java
	public class ConcreteElementA extends Element {  
	  
	    @Override  
	    public void accept(Visitor visitor) {  
	        // 回调访问者对象的相应方法  
	        visitor.visitConcreteElementA(this);  
	  
	    }  
	  
	    /** 
	     * 示例方法，表示元素已有的功能实现 
	     */  
	    public void operationA() {  
	        System.out.println("执行ConcreteElementA已有的operationA方法.");  
	  
	    }  
	}
```
<br/>
```Java
	public class ConcreteElementB extends Element {  
	  
	    @Override  
	    public void accept(Visitor visitor) {  
	        // 回调访问者对象的相应方法  
	        visitor.visitConcreteElementB(this);  
	  
	    }  
	  
	    /** 
	     * 示例方法，表示元素已有的功能实现 
	     */  
	    public void operationB() {  
	        System.out.println("执行ConcreteElementB已有的operationB方法.");  
	  
	    }  
	}  
​```
（5）对象结构(ObjectStructure)示例代码如下。
```Java
	import java.util.ArrayList;  
	import java.util.Collection;  
	  
	/** 
	 * 对象结构，通常在这里对元素对象进行遍历，让访问者能够访问到所有的元素 
	 */  
	public class ObjectStructure {  
	  
	    /** 
	     * 示意,表示对象结构，可以是一个组合结构或者集合 
	     */  
	    private Collection<Element> col = new ArrayList<Element>();  
	  
	    /** 
	     * 示意方法，提供给客户端操作的高层接口，让访问者对对象结构中的所有元素进行访问 
	     */  
	    public void handleRequest(Visitor visitor) {  
	        // 循环对象结构中的元素，进行访问  
	        for (Element element : col) {  
	            element.accept(visitor);  
	        }  
	    }  
	  
	    /** 
	     * 示意方法，组建对象结构，向对象结构中添加元素 
	     */  
	    public void addElement(Element element) {  
	        this.col.add(element);  
	    }  
	}  
```
（6）最后在客户端测试一下，示例代码如下。 
```Java
	public class Client {  
	    public static void main(String[] args) {  
	  
	        // 创建对象结构  
	        ObjectStructure os = new ObjectStructure();  
	  
	        // 为对象结构中添加元素对象  
	        os.addElement(new ConcreteElementA());  
	        os.addElement(new ConcreteElementB());  
	  
	        // 创建访问者  
	        Visitor visitor = new ConcreteVisitorA();  
	  
	        // 调用对象结构的业务处理方法  
	        os.handleRequest(visitor);  
	    }  
	  
	}  
```
 运行程序打印结果如下：
```
ConcreteVisitorA 访问 ==> ConcreteElementA 对象。   
ConcreteVisitorA 访问 ==> ConcreteElementB 对象。   
```
访问者模式中对象结构存储了不同类型的元素对象，以供不同访问者访问。访问者模式包括两个层次结构，一个是访问者层次结构，提供了抽象访问者和具体访问者，一个是元素层次结构，提供了抽象元素和具体元素。相同的访问者可以以不同的方式访问不同的元素，相同的元素可以接受不同访问者以不同访问方式访问。在访问者模式中，增加新的访问者无须修改原有系统，系统具有较好的可扩展性。 

访问者模式在不破坏类的前提下，为类提供增加新的新操作，其实现的关键是双分派（ Double-Dispatch）的技术。 在访问者模式中 accept()操作是一个双分派的操作。具体调用哪一个具体的accept()操作，有两个决定因素： 

1. Element的类型。因为 accept()是多态的操作，需要具体的 Element 类型的子类才可以决定到底调用哪一个accept()实现；
2. Visitor的类型。accept()操作有一个参数（ Visitor visitor），要决定了实际传进来的 Visitor 的实际类别才可以决定具体是调用哪个 VisitConcrete（）实现。

# 三、访问者模式的适用性

从访问者模式的定义可以看出对象结构是使用访问者模式的先决条件，在以下情况可以考虑使用访问者模式：

1. 对象结构内包含多种类型的对象，我们希望对这些对象实施一些依赖于其具体类型的操作。
2. 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作“污染”这些对象的类，也不希望在增加新操作时修改这些类。
3. 对象结构中对象的类型很少改变，但经常需要在此对象结构上定义新的操作。

# 四、 场景举例

一个商场（SuperMarket），通常都会包括（当然还会包含一些其他的组成部分）：商店（Store）、监控室（MonitoringRoom）、卫生间（WaterCloset）。商场的访问者大致可以分为两大类：顾客（Customer）、商场工作人员（MarketStaff）。顾客可以逛商店、上卫生间，但却不能进入监控室；工作人员可以进入监控室、上卫生间，但却不能像顾客一样逛商店（除非他不想干了），也就是说对于商场的同一个地点，不同的访问者有不同的行为权限，而且访问者的种类很有可能需要根据时间的推移发生变化（没准哪天，工商局的人要来视察呢！此时就需要增加工商局人员的访问者了。）。

使用访问者模式实现，其类图如下： 

<div align=center>

![访问者模式应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/17.png "访问者模式应用示意图")
<div align=left>

Place相当于访问者模式中的Element。 
```Java
	public abstract class Place {
	 
		// 接受访问者的访问
		public abstract void accept(Visitor visitor);
	}
```
Place的具体实现类有三个，WaterCloset、Store和MonitoringRoom。  
```Java
	public class WaterCloset extends Place {
	 
		@Override
		public void accept(Visitor visitor) {
			// 回调访问者对象的相应方法
			visitor.visitWaterCloset(this);
		}
	 
		public void washing() {
			// 卫生间已有的方法
			System.out.println("洗洗手。。。*/");
		}
	}
```
<br/>
```Java
	public class Store extends Place {
	 
		@Override
		public void accept(Visitor visitor) {
			// 回调访问者对象的相应方法
			visitor.visitStore(this);
		}
	 
		public void shopping() {
			// 商店已有的方法
			System.out.println("欢迎光临，祝您购物愉快。。。*/");
		}
	}
```
<br/>
```Java
	public class MonitoringRoom extends Place {
	 
		@Override
		public void accept(Visitor visitor) {
			// 回调访问者对象的相应方法
			visitor.visitMonitoringRoom(this);
		}
	 
		public void watching() {
			// 监控室已有的方法
			System.out.println("查看监控录像...*/");
		}
	 
	}
```
抽象访问者Visitor的代码如下。  
```Java
	public interface Visitor {
	 
		// 进入卫生间
		public void visitWaterCloset(WaterCloset wc);
	 
		// 进入监控室
		public void visitMonitoringRoom(MonitoringRoom mr);
	 
		// 进入商店
		public void visitStore(Store store);
	 
	}
```
Visitor的具体实现类，MarketStaff和Customer。  
```Java
	public class MarketStaff implements Visitor {
	 
		@Override
		public void visitWaterCloset(WaterCloset wc) {
			System.out.println("/*工作人员来到卫生间。。。");
			wc.washing();
			System.out.println();
		}
	 
		@Override
		public void visitMonitoringRoom(MonitoringRoom mr) {
			System.out.println("/*工作人员来到监控室。。。");
			mr.watching();
			System.out.println();
	 
		}
	 
		@Override
		public void visitStore(Store store) {
			System.out.println("/*工作人员来到商店。。。");
			System.out.println("现在是工作时间，请专心工作。。。*/");
			System.out.println();
	 
		}
	}
```
<br/>
```Java
	public class Customer implements Visitor {
	 
		@Override
		public void visitWaterCloset(WaterCloset wc) {
			System.out.println("/*顾客来到卫生间。。。");
			wc.washing();
			System.out.println();
	 
		}
	 
		@Override
		public void visitMonitoringRoom(MonitoringRoom mr) {
			System.out.println("/*顾客来到监控室。。。");
			System.out.println("非工作人员禁止入内。。。*/");
			System.out.println();
	 
		}
	 
		@Override
		public void visitStore(Store store) {
			System.out.println("/*顾客来到商店。。。");
			store.shopping();
			System.out.println();
		}
	 
	}
```
SuperMarket就是结构对象。  
```Java
	import java.util.ArrayList;
	import java.util.Collection;
	 
	public class SuperMarket {
	 
		// 示意,表示对象结构，可以是一个组合结构或者集合
		private Collection<Place> col = new ArrayList<Place>();
	 
		public void handleRequest(Visitor visitor) {
			// 循环对象结构中的元素，进行访问
			for (Place place : col) {
				place.accept(visitor);
			}
		}
	 
		// 示意方法，组建对象结构，向对象结构中添加元素
		public void addPlace(Place place) {
			this.col.add(place);
		}
	}
```
Client中测试代码如下。  
```Java
	public class Client {
	 
		public static void main(String[] args) {
			SuperMarket superMarket = new SuperMarket();
			superMarket.addPlace(new WaterCloset());
			superMarket.addPlace(new Store());
			superMarket.addPlace(new MonitoringRoom());
	 
			// 创建一个顾客访问者
			Visitor customer = new Customer();
	 
			System.out.println("=====顾客来到商场=====");
			superMarket.handleRequest(customer);
			System.out.println("=====顾客离开商场=====");
			System.out.println();
			// 创建一个商场工作人员访问者
			Visitor visitor = new MarketStaff();
			System.out.println("=====工作人员来到商场=====");
			superMarket.handleRequest(visitor);
			System.out.println("=====工作人员离开商场=====");
		}
	 
	}
```
运行程序打印结果如下：  
```
=====顾客来到商场=====
/*顾客来到卫生间。。。
洗洗手。。。*/
 
/*顾客来到商店。。。
欢迎光临，祝您购物愉快。。。*/
 
/*顾客来到监控室。。。
非工作人员禁止入内。。。*/
 
=====顾客离开商场=====
 
=====工作人员来到商场=====
/*工作人员来到卫生间。。。
洗洗手。。。*/
 
/*工作人员来到商店。。。
现在是工作时间，请专心工作。。。*/
 
/*工作人员来到监控室。。。
查看监控录像...*/
 
=====工作人员离开商场=====
```

# 五、访问者模式的特点

## 访问者模式的优点　　

+ 好的扩展性：能够在不修改对象结构中的元素的情况下，为对象结构中的元素添加新的功能。 　　
+ 好的复用性：可以通过访问者来定义整个对象结构通用的功能，从而提高复用程度。 　　
+ 分离无关行为：可以通过访问者来分离无关的行为，把相关的行为封装在一起，构成一个访问者，这样每一个访问者的功能都比较单一。

## 访问者模式的缺点　　

+ 增加新的元素类很困难：在访问者类中，每一个元素类都有它对应的处理方法，也就是说，每增加一个元素类都需要修改访问者类（也包括访问者类的子类或者实现类），修改起来相当麻烦。 　　
+ 破坏封装：访问者模式通常需要对象结构开放内部数据给访问者和ObjectStructrue，这破坏了对象的封装性。 