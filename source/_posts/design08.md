---
title: 设计模式系列之--策略模式
date: 2020-07-18 12:08:00
updated: 2020-07-18 12:08:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 策略模式
type: 
description: 什么是策略模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
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
# 一、什么是策略模式

策略（Strategy）模式定义了一系列的算法，并将每一个算法封装起来，而且使它们还可以相互替换。策略模式让算法独立于使用它的客户而独立变化。（原文：The Strategy Pattern defines a family of algorithms,encapsulates each one,and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.） 

# 二、策略模式的结构

<div align=center>

![策略模式结构示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/4.png "策略模式结构示意图")
<div align=left>
在策略模式中的角色有： 

- 抽象策略(Strategy)角色:这是一个抽象角色，通常由一个接口或抽象类实现,用以声明所有具体策略类要实现的方法。 
- 具体策略(ConcreteStrategy)角色:各种策略（算法）的具体实现。 
- 环境(Context)角色:策略的外部封装类，或者说策略的容器类，它持有一个Strategy的引用，Strategy策略由客户端决定，不同Strategy策略可以执行不同的行为。 

策略模式结构示意源代码如下： 

**抽象策略(Strategy)角色** 
```Java
	public interface Strategy {
		// 抽象策略角色
		public void strategyInterface();
	}
```
 **具体策略(ConcreteStrategy)角色**
```Java
	public class ConcreteStrategyA implements Strategy {
		// 具体策略角色
		@Override
		public void strategyInterface() {
			// 相关的业务
		}
	}
```
<br/>
```Java
	public class ConcreteStrategyB implements Strategy {
		// 具体策略角色
		@Override
		public void strategyInterface() {
			// 相关的业务
		}
	}
```
<br/>
```Java
	public class ConcreteStrategyC implements Strategy {
		// 具体策略角色
		@Override
		public void strategyInterface() {
			// 相关的业务
		}
	}
```
**环境(Context)角色 **
```Java
	public class Context {
		// 持有一个具体策略的对象
		private Strategy strategy;
		public Context(Strategy strategy) {
			this.strategy = strategy;
		}
	 
		// 环境角色策略方法
		public void contextInterface() {
			strategy.strategyInterface();
		}
	}
```
从策略模式中环境(Context)角色的功能可以看出，策略模式的重心不是如何实现算法，而是如何组织、调用这些算法，从而让程序结构更灵活，具有更好的维护性和扩展性，它体现了面向对象程序设计中两个非常重要的原则：

1. 封装变化 把一个类中经常改变或者将来可能改变的部分提取出来，作为一个接口（也可以是抽象类），然后在类中包含实现了这个接口的实例，这样这个类就可以通过其内包含的实例来实现接口的行为。
2. 依赖倒转(Dependence Inversion Principle ) 编程中更多地使用抽象，而不是具体的实现。 

# 三、策略模式应用场景
超市出售商品的折扣是一个很复杂的问题，对于不同类型的顾客会有不同的折扣，在不同的时间购买商品也有可能折扣不同，我们假设超市的折扣有以下几种情况：

1. 对普通顾客，没有折扣。
2. 对普通会员，所有商品9.5折。
3. 对VIP会员，所有商品9.0折。

使用策略模式模拟以上折扣的算法：

**抽象策略（Strategy）角色**
```Java
	public interface DiscountStrategy {
		/**
		 * 计算打折后的价格
		 * 
		 * @param num
		 *            商品原价
		 * @return 折后价格
		 */
		public double calcPrice(double num);
	}
```
实现DiscountStrategy（折扣策略）接口的具体策略角色。

**普通顾客策略类**
```Java
	public class CommonCustomerStrategy implements DiscountStrategy {
		@Override
		public double calcPrice(double num) {
			// 普通会员没有折扣
			return num;
		}
	}
```
**普通会员策略类**
```Java
	public class CommonMemberStrategy implements DiscountStrategy {
		@Override
		public double calcPrice(double num) {
			// 普通会员9.5折
			return num * 0.95;
		}
	}
```
**VIP会员策略类**
```Java
	public class VipMemberStrategy implements DiscountStrategy {
		@Override
		public double calcPrice(double num) {
			// 普通会员9.0折
			return num * 0.9;
		}
	}
```
**环境角色类**
```Java
	public class Price {
		// 持有一个具体的策略对象
		private DiscountStrategy strategy;
		// 构造方法中传入一个具体的策略对象
		public Price(DiscountStrategy strategy) {
			this.strategy = strategy;
		}
		public double calcPrice(double num) {
			// 调用内部包含的策略对象方法计算价格
			return this.strategy.calcPrice(num);
		}
	}
```
创建一个MainClass类测试一番
```Java
	public class MainClass {
		public static void main(String[] args) {
			// 商品原价
			double num = 200;
			// 普通顾客策略计算折后价格
			Price price = new Price(new CommonCustomerStrategy());
			double newPrice = price.calcPrice(num);
			System.out.println("普通顾客购买该商品的折后价格是：" + newPrice);
	 
			// 普通会员策略计算折后价格
			price = new Price(new CommonMemberStrategy());
			newPrice = price.calcPrice(num);
			System.out.println("普通会员购买该商品的折后价格是：" + newPrice);
	 
			// VIP会员策略计算折后价格
			price = new Price(new VipMemberStrategy());
			newPrice = price.calcPrice(num);
			System.out.println("VIP会员购买该商品的折后价格是：" + newPrice);
		}
	}
```
运行程序打印结果：
```
普通顾客购买该商品的折后价格是：200.0
普通会员购买该商品的折后价格是：190.0
VIP会员购买该商品的折后价格是：180.0
```
从以上示例可以看出策略模式的意图是针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以互相替换。策略模式自身并不能决定何时该采用哪种策略算法，该使用何种算法是由客户端来决定的。

# 四、策略模式的特点

**优点：**

- 策略模式提供了管理相关的算法族的办法。策略类的等级结构定义了一个算法或行为族。恰当使用继承可以把公共的代码移到父类里面，从而避免代码重复。
- 使用策略模式可以避免使用（if-else）多重条件转移语句。多重条件转移语句不易维护，它把采取哪一种算法或采取哪一种行为的逻辑与算法或行为的逻辑混合在一起，统统列在一个多重转移语句里面，比使用继承的办法还要原始和落后。

**缺点：**

- 客户端必须知道所有的策略类，并自行决定使用哪一个策略类。这就意味着客户端必须理解这些算法的区别，以便适时选择恰当的算法类。换言之，策略模式只适用于客户端知道算法或行为的情况。
- 由于策略模式把每个具体的策略实现都单独封装成为类，如果备选的策略很多的话，那么具体策略类的数目就会很可观。 

# 五、参考文章

[http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html](http://www.cnblogs.com/java-my-life/archive/2012/04/20/2455726.html "《JAVA与模式》之装饰模式")