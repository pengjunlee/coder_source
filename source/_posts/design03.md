---
title: 设计模式系列之--抽象工厂模式
date: 2020-07-18 12:03:00
updated: 2020-07-18 12:03:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 抽象工厂
type: 
description: 什么是抽象工厂模式？
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
# 一、什么是抽象工厂模式

抽象工厂模式是所有形态的工厂模式中最为抽象和最具一般性的。抽象工厂模式可以向客户端提供一个接口，使得客户端在不必指定产品的具体类型的情况下，能够创建多个产品族的产品对象。

此处引入了一个新的概念产品族，那什么是产品族呢？百度一下：产品族是以产品平台为基础，通过添加不同的个性模块，以满足不同客户个性化需求的一组相关产品。
<div align=center>

![1.png](http://pengjunlee.3vzhuji.net/static/design_pattern/1.jpg "产品簇示意图")  
<div align=left>
所谓产品族通俗来说即是：具有某一共性的一系列相关产品.以前面的Apple(苹果),Banana(香蕉),Pear(梨)为例，Apple(苹果),Banana(香蕉),Pear(梨)这三种水果对应上图中的产品等级结构。

这三种水果有产自南方的，也有产自北方的，北方和南方则对应上图中的产品族，产自北方的Apple(苹果),Banana(香蕉),Pear(梨)就构成一个产品族，它们的共性是产自北方，同样产自南方的Apple(苹果),Banana(香蕉),Pear(梨)也构成了一个产品族。 

# 二、模式中包含的角色及其职责

+ 抽象工厂（Factory）角色：抽象工厂模式的核心，包含对多个产品等级结构的声明，任何工厂类都必须实现这个接口。
+ 具体工厂（ConcreteFactory）角色：具体工厂类是抽象工厂的一个实现，负责实例化某个产品族中的产品对象。
+ 抽象（Product）角色：抽象模式所创建的所有对象的父类，或声明所有具体产品所共有的公共接口。
+ 具体产品（ConcreteProduct）角色：抽象工厂模式所创建的真正实例。 

> 总结：抽象工厂中的方法对应产品等级结构，具体工厂对应产品族。

接下来用代码进行说明：保留之前工厂方法模式中的Fruit接口，用来负责描述所有水果实例应该共有的方法。
```Java
	public interface Fruit {
		/*
		 * 采集
		 */
		public void get();
	}
```
此时Apple(苹果)和Banana(香蕉)将不再是具体的产品类而是抽象类，因为它们还需要进一步划分北方和南方两个产品族。
```Java
	public abstract class Apple implements Fruit{
		/*
		 * 采集
		 */
		public abstract void get();
	}
```
<br/>
```Java
	public abstract class Banana implements Fruit{
		/*
		 * 采集
		 */
		public abstract void get();
	}
```
再进一步细分，苹果（Apple）被具体化为北方苹果(NorthApple)和南方苹果(SouthApple)。
```Java
	public class NorthApple extends Apple {
	 
		public void get() {
			System.out.println("采集北方苹果");
		}
	 
	}
```
<br/>
```Java
	public class SouthApple extends Apple {
	 
		public void get() {
			System.out.println("采集南方苹果");
		}
	 
	}
```
香蕉(Banana)被具体化为北方香蕉(NorthBanana)和南方香蕉(SouthBanana).
```Java
	public class NorthBanana extends Banana {
	 
		public void get() {
			System.out.println("采集北方香蕉");
		}
	 
	}
```
<br/>
```Java
	public class SouthBanana extends Banana {
	 
		public void get() {
			System.out.println("采集南方香蕉");
		}
	 
	}
```
继续写工厂，与之前的FruitFactory有所不同，此时的FruitFactory需为每一个产品等级结构添加获取方法声明。
```Java
	public interface FruitFactory {
		//实例化Apple
		public Fruit getApple();
		//实例化Banana
		public Fruit getBanana();
	}
```
为每一个产品族添加相应的工厂，NorthFruitFactory负责生产所有北方的水果，SouthFruitFactory负责生产所有南方的水果。
```Java
	public class NorthFruitFactory implements FruitFactory {
	 
		public Fruit getApple() {
			return new NorthApple();
		}
	 
		public Fruit getBanana() {
			return new NorthBanana();
		}
	 
	}
```
<br/>
```Java
	public class SouthFruitFactory implements FruitFactory {
	 
		public Fruit getApple() {
			return new SouthApple();
		}
	 
		public Fruit getBanana() {
			return new SouthBanana();
		}
	 
	}
```
在客户端中进行测试，代码如下。
```Java
	public class Client {
		public static void main(String[] args) {
	 
			FruitFactory ff1 = new NorthFruitFactory();
			Fruit apple1 = ff1.getApple();
			apple1.get();
	 
			Fruit banana1 = ff1.getBanana();
			banana1.get();
	 
			FruitFactory ff2 = new SouthFruitFactory();
			Fruit apple2 = ff2.getApple();
			apple2.get();
	 
			Fruit banana2 = ff2.getBanana();
			banana2.get();
		}
	}
```
运行程序打印结果如下：
```
采集北方苹果
采集北方香蕉
采集南方苹果
采集南方香蕉
```

# 三、抽象工厂模式的优缺点

抽象工厂模式有以下优点：

1. 抽象工厂模式隔离了具体产品类的生产，使得客户并不需要知道即将创建的对象的具体类型。
2. 当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。
3. 增加新的具体工厂和产品族很方便，无须修改已有系统，符合“开闭原则”。

抽象工厂模式有以下缺点：

+ 增加新的产品等级结构很复杂，需要修改抽象工厂和所有的具体工厂类，对“开闭原则”的支持呈现倾斜性。

优点代码举例：需求更改需要增加温室产品族。

先增加温室苹果和温室香蕉两个类。
```Java
	public class WenshiApple extends Apple {
	 
		public void get() {
			System.out.println("采集温室苹果");
		}
	 
	}
```
<br/>
```Java
	public class WenshiBanana extends Banana {
	 
		public void get() {
			System.out.println("采集温室香蕉");
		}
	 
	}
```
再增加生产温室水果的温室工厂。
```Java
	public class WenshiFruitFactory implements FruitFactory {
	 
		public Fruit getApple() {
			return new WenshiApple();
		}
	 
		public Fruit getBanana() {
			return new WenshiBanana();
		}
	 
	}
```
在客户端中测试一下，创建温室水果测试。
```Java
	public class Client{
		public static void main(String[] args) {
			
			FruitFactory ff3 = new WenshiFruitFactory();
			Fruit apple3 = ff3.getApple();
			apple3.get();
			
			Fruit banana3 = ff3.getBanana();
			banana3.get();
		}
	}
```
运行程序打印结果如下：
```
采集温室苹果  
采集温室香蕉  
```

产自温室的苹果和香蕉被正确创建，只是增加了新产品族的具体产品类和负责生产该产品族所有产品的工厂，无需对现有代码进行修改，很好的符合“开闭原则”。