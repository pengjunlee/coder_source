---
title: 设计模式系列之--观察者模式
date: 2020-07-18 12:09:00
updated: 2020-07-18 12:09:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 观察者模式
type: 
description: 什么是观察者模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
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
# 一、什么是观察者模式

 观察者(Observer)模式是行为模式之一，它的作用是当一个被观察对象的状态发生变化时，能够自动通知相关的观察者对象，自动更新观察者对象中被观察对象的状态。它提供给关联对象一种同步通信的手段，使某个对象与依赖它的其他对象之间保持状态同步。 

# 二、观察者模式的典型应用

观察者(Observer)模式多被应用于以下场景：

- 侦听事件驱动程序设计中的外部事件
- 侦听/监视某个对象的状态变化
- 发布者/订阅者(publisher/subscriber)模型中，当一个外部事件（新的产品，消息的出现等等）被触发时，通知邮件列表中的订阅者 

# 三、观察者模式的结构
<div align=center>

![策略模式结构示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/5.png "策略模式结构示意图")
<div align=left>

观察者模式中的角色及其职责如下：

- 抽象被观察者（Subject）角色    被观察对象的抽象类，为所有具体被观察者角色提供了一个统一接口，主要包括添加、删除、通知观察者对象方法，其内包含一个用以储存所有观察者对象的容器（Collection），当被观察者的状态发生变化时，需要通知容器中所有观察者对象，并维护（添加，删除，通知）观察者对象列表。
- 具体被观察者（ConcreteSubject）角色    被观察者的具体实现，包含一些基本的属性状态及其他操作，当具体被观察者的状态发生变化时，会给注册的所有观察者对象发送通知，以提示观察者对象们对其内被观察对象的状态进行更新。
- 抽象观察者（Observer）角色    这是一个抽象角色，通常被定义为接口，为所有的具体观察者定义一个接口，当收到被观察对象状态发生变化的通知时更新自己。
- 具体观察者（ConcreteObserver）角色    观察者的具体实现，得到通知后将对内部被观察对象的状态进行更新，并完成一些具体的业务逻辑处理。

观察者模式结构示意源代码如下：

**抽象被观察者（Subject）角色**
```Java
	public abstract class Subject {
	 
		//用来保存注册过的观察者
		private List<Observer> observers=new ArrayList<Observer>();
		//注册一个观察者
		public void registerObserver(Observer observer){
			this.observers.add(observer);
		}
		//删除一个观察者
		public void unregisterObserver(Observer observer){
			this.observers.remove(observer);
		}
		//通知所有观察者进行状态更新
		public void notifyObservers(Subject subject){
			for(Observer o:observers){
				o.update(subject);
			}
		}
	}
```
**具体被观察者（ConcreteSubject）角色**
```Java
	public class ConcreteSubject extends Subject {
	    //具体被观察者类可以具有自己的属性或状态
	    private String state;
	 
	    public String getState() {
	        return state;
	    }
	 
	    public void setState(String newState){
	 
	    	this.state = newState;
	        System.out.println("被观察者自身状态更新为：" + this.state);
	 
	        //状态发生改变，通知所有观察者
	        this.notifyObservers(this);
	    }
	}
```
**抽象观察者（Observer）角色**
```Java
	public interface Observer{
	 
		//更新观察者的状态
		public void update(Subject subject);
	 
	}
```
**具体观察者（ConcreteObserver）角色**
```Java
	public class ConcreteObserverA implements Observer {
	 
		private ConcreteSubject subject;
		public ConcreteSubject getSubject() {
			return subject;
		}
	 
		public void setSubject(ConcreteSubject subject) {
			this.subject = subject;
		}
	 
		@Override
		public void update(Subject subject) {
			this.subject=(ConcreteSubject)subject;
			System.out.println("观察者A中被观察对象的状态更新为："+this.subject.getState());
		}
	}
```
<br/>
```Java
	public class ConcreteObserverB implements Observer {
		private ConcreteSubject subject;
		public ConcreteSubject getSubject() {
			return subject;
		}
	 
		public void setSubject(ConcreteSubject subject) {
			this.subject = subject;
		}
	 
		@Override
		public void update(Subject subject) {
			this.subject=(ConcreteSubject)subject;
			System.out.println("观察者B中被观察对象的状态更新为："+this.subject.getState());
	 
		}
	}
```
编写一个MainClass类来测试一番。
```Java
	public class MainClass {
	 
		public static void main(String[] args) {
	 
			//创建被观察者对象
			ConcreteSubject subject=new ConcreteSubject();
	 
			//创建观察者对象
			ConcreteObserverA observerA=new ConcreteObserverA();
			ConcreteObserverB observerB=new ConcreteObserverB();
	 
			//为被观察者对象注册观察者
			subject.registerObserver(observerA);
			subject.registerObserver(observerB);
	 
			subject.setState("复活中...");
			System.out.println();
			System.out.println("----------一千年以后...----------");
			System.out.println();
			subject.setState("疯狂杀戮中...");
	 
		}
	}
```
运行MainClass打印结果如下：
```Java
被观察者自身状态更新为：复活中...
观察者A中被观察对象的状态更新为：复活中...
观察者B中被观察对象的状态更新为：复活中...
----------一千年以后...----------
被观察者自身状态更新为：疯狂杀戮中...
观察者A中被观察对象的状态更新为：疯狂杀戮中...
观察者B中被观察对象的状态更新为：疯狂杀戮中...
```

# 四、Java对观察者模式的支持

在JAVA语言的java.util库里面有一个Observable类和一个Observer接口，通过两者配合使用可以实现观察者模式。

**Observable类：**

Observable中文意思“可以被观察的”，即Observable类是可以被观察的，想要实现观察者模式只需将你想要被观察的类继承自Observable类即可。

一个 Observable对象可以有一个或多个观察者，观察者可以是实现了Observer接口的任意对象。一个Observable对象状态改变后,会调用notifyObservers()方法来通知观察者。

以下是Observable类提供的一些常用方法：
```Java
	public void addObserver(Observer o)    //向观察者集合中添加观察者
	public void deleteObserver(Observer o)    //从观察者集合中删除某一个观察者
	public void notifyObservers(Object arg)    //如果hasChanged方法指示对象已改变，则通知其所有观察者，并调用 clearChanged 方法来清除对象的已改变标记。此方法可不带参数，仅将Observable对象传递给update()方法，此时update方法中arg参数为Null。
	public void deleteObservers()    //清除观察者列表，使此对象不再有任何观察者
	protected void setChanged()    //标记Observable对象已经改变
	protected void clearChanged()    //清除Observable对象已改变标记
	public boolean hasChanged()    //测试对象是否改变
	public int countObservers()    //返回Observable对象的观察者数目
```
**Observer接口：**

Observer接口只包含一个update()方法,该方法仅接受两个参数：继承自Observable类的被观察对象和传递给notifyObservers() 方法的参数。当Observable(被观察者)对象状态发生改变时将通过notifyObservers()方法向所有的Observer(观察者)发送更新通知，Observer(观察者)对象在收到通知后即调用此方法完成状态更新。 
```Java
	void update(Observable o, Object arg)
```
接下来我们举个简单的例子来演示一下如何使用Observable类和Observer接口实现观察者模式，去年2015年是牛市，炒股的基本都发家了，在此我们以炒股为例。 

在炒股这个例子中，不用我说，相信你也想的到：股票Stock就是我们的被观察者。
```Java
	import java.util.Observable;
	public class Stock extends Observable{
		//为股票的状态创建一个枚举类：RISE 涨,FALL 跌
		public enum StockState{
			RISE,FALL
		}
		//股票涨跌状态
		private StockState state;
		//股票的价格
		private double price;
		//股票的历史最低价格
		private double LowestPrice;
		//股票无参的构造方法
		public Stock(){
		}
		//股票的带参数构造方法
		public Stock(StockState state, double price, double lowestPrice) {
			super();
			this.state = state;
			this.price = price;
			LowestPrice = lowestPrice;
		}
		public StockState getState() {
			return state;
		}
		private void setState(StockState state) {
			this.state = state;
		}
		public double getPrice() {
			return price;
		}
		public void setPrice(double price) {
			if(price<this.price){
				setState(StockState.FALL);
			}
			else{
				setState(StockState.RISE);
			}
			if(price<this.LowestPrice){
				setLowestPrice(price);
			}
			this.price = price;
			//更新股票状态标记为已改变
			this.setChanged();
			//通知观察者
			notifyObservers();
		}
		public double getLowestPrice() {
			return LowestPrice;
		}
		private void setLowestPrice(double lowestPrice) {
			LowestPrice = lowestPrice;
		}
	}
```
我们这些只有韭菜命的股民自然就是观察者了。
```Java
	import java.util.Observable;
	import java.util.Observer;
	import com.ibeifeng.news.Stock.StockState;
	public class Investor implements Observer {
		public void update(Observable o, Object arg) {
			Stock stock=(Stock)o;
			if(stock.getPrice()==stock.getLowestPrice())
			{
				System.out.println("股票已经跌到历史最低了，我得赶紧抄底...");
			}
			else{
				if(stock.getState().equals(StockState.RISE)){
					System.out.println("股票在涨，目前价格："+stock.getPrice());
				}
				else{
					System.out.println("股票在跌，目前价格："+stock.getPrice());
				}
			}
		}
	}
```
在客户端里面测试一下。
```Java
	public class Client{
		public static void main(String[] args) {
			Stock stock=new Stock(StockState.FALL,14.7D,13.2D);
			stock.addObserver(new Investor());
			stock.setPrice(13.7);
			stock.setPrice(12.6);
			stock.setPrice(14.0);
		}
	}
```
运行程序打印结果如下：

股票在跌，目前价格：13.7  
股票已经跌到历史最低了，我得赶紧抄底...  
股票在涨，目前价格：14.0  

# 五、观察者模式的优点

观察者Observer模式解决了对象之间一对多的复杂依赖关系问题，大大地提高了程序的可维护性和可扩展性，也很好的符合了开放-封闭原则。 