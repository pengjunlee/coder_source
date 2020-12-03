---
title: 设计模式系列之--迭代器模式
date: 2020-07-18 12:22:00
updated: 2020-07-18 12:22:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 迭代器模式
type: 
description: 什么是迭代器模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img22.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img22.jpg
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
# 一、什么是迭代器模式

迭代器(Iterator)模式又叫作游标(Cursor)模式，是一种对象的行为模式。提供一种方法顺序访问一个聚合（指一组对象的组合结构，如：Java中的集合、数组等）对象中各个元素，而又不需暴露该对象的内部表示。

`迭代器模式的本质：控制访问聚合对象中的元素`

设计意图：无须暴露聚合对象的内部实现，就能够访问到聚合对象中的各个元素。  

# 二、迭代器模式的结构

<div align=center>

![迭代器应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/35.png "迭代器应用示意图")
<div align=left>

迭代器模式涉及的角色及其职责如下：

+ 抽象迭代器(Iterator)角色：一般定义为接口，用来定义访问和遍历元素的接口。
+ 具体迭代器(ConcreteIterator)角色：实现对聚合对象的遍历，并跟踪遍历时的当前位置。
+ 抽象聚合(Aggregate)角色：定义创建相应迭代器对象的接口。
+ 具体聚合(ConcreteAggregate)角色：实现创建相应的迭代器对象。

迭代器模式结构示意源代码如下：
先来看看抽象迭代器接口的定义。示例代码如下： 
```Java
	/**
	 * 迭代器接口，定义访问和遍历元素的操作
	 */
	public interface Iterator {
	 
		/**
		 * 移动到聚合对象中第一个元素
		 */
		public void first();
	 
		/**
		 * 移动到聚合对象中下一个元素
		 */
		public void next();
	 
		/**
		 * 判断是否已经移动到聚合对象的最后一个元素
		 */
		public boolean isDone();
	 
		/**
		 * 获取迭代的当前元素
		 */
		public Object currentItem();
	 
	}
```
接下来看看具体迭代器是如何实现的。示例代码如下：
```Java
	/**
	 * 具体的迭代器实现类，不同的聚合对象所对应的迭代器实现是不一样的 以下示意的是数组聚合对象的迭代器
	 */
	public class ConcreteIterator implements Iterator {
	 
		/**
		 * 持有被迭代的具体的聚合对象
		 */
		private ConcreteAggregate aggregate;
	 
		/**
		 * 内部索引，记录当前迭代到的位置
		 */
		private int index = -1;
	 
		/**
		 * 构造方法，传入被迭代的具体的聚合对象
		 * 
		 * @param aggregate 被迭代的具体的聚合对象
		 */
		public ConcreteIterator(ConcreteAggregate aggregate) {
			this.aggregate = aggregate;
		}
	 
		/**
		 * 移动到聚合对象中第一个元素
		 */
		@Override
		public void first() {
			index = 0;
	 
		}
	 
		/**
		 * 移动到聚合对象中下一个元素
		 */
		@Override
		public void next() {
			if (index < this.aggregate.size()) {
				index += 1;
			}
	 
		}
	 
		/**
		 * 判断是否已经移动到聚合对象的最后一个元素
		 */
		@Override
		public boolean isDone() {
			if (index == this.aggregate.size()) {
				return true;
			}
			return false;
		}
	 
		/**
		 * 获取迭代的当前元素
		 */
		@Override
		public Object currentItem() {
			return this.aggregate.get(index);
		}
	 
	}
```
再来看看抽象聚合类的定义。示例代码如下：
```Java
	/**
	 * 聚合对象的接口，定义创建相应的迭代器对象的接口
	 */
	public abstract class Aggregate {
		/**
		 * 工厂方法，创建相应的迭代器对象的接口
		 */
		public abstract Iterator createIterator();
	}
```
下面该来看看具体聚合类是如何实现的了，这里以数组聚合对象的迭代器实现为例。示例代码如下： 
```Java
	/**
	 * 具体的聚合对象，实现创建相应迭代器对象的功能
	 *
	 */
	public class ConcreteAggregate extends Aggregate {
	 
		/**
		 * 示意，表示聚合对象具体的内容
		 */
		private String[] ss = null;
	 
		/**
		 * 构造方法，传入聚合对象具体的内容
		 * 
		 * @param ss 聚合对象具体的内容
		 */
		public ConcreteAggregate(String[] ss) {
			super();
			this.ss = ss;
		}
	 
		/**
		 * 工厂方法，创建相应的迭代器对象的接口
		 */
		@Override
		public Iterator createIterator() {
			// 实现创建迭代器的工厂方法
			return new ConcreteIterator(this);
		}
	 
		/**
		 * 根据索引位置，获取所对应的元素
		 */
		public Object get(int index) {
			Object retObj = null;
			if (index < ss.length) {
				retObj = ss[index];
			}
			return retObj;
		}
	 
		/**
		 * 获取聚合对象的容量大小
		 */
		public int size() {
			return this.ss.length;
		}
	 
	}
```
在客户端中测试一下，示例代码如下。  
```Java
	public class Client {
	 
		public void someOperation() {
			// 创建一个数组
			String[] names = { "张三", "李四", "王五" };
			// 创建聚合对象
			ConcreteAggregate aggregate = new ConcreteAggregate(names);
			// 获取聚合对象的迭代器
			Iterator iterator = aggregate.createIterator();
			// 移动到聚合对象中第一个元素
			iterator.first();
			int index = 1;
			// 循环输出聚合对象中的值
			while (!iterator.isDone()) {
				Object obj = iterator.currentItem();
				System.out.println("聚合对象中第" + (index++) + "个元素是：" + obj);
				iterator.next();
			}
		}
	 
		public static void main(String[] args) {
			Client client = new Client();
			client.someOperation();
	 
		}
	 
	}
```
运行程序打印结果如下：  
```
聚合对象中第1个元素是：张三 
聚合对象中第2个元素是：李四
聚合对象中第3个元素是：王五
```
从以上示例可以看出，迭代器模式为客户端提供了一个统一访问聚合对象的接口，通过这个接口就可以顺序地访问聚合对象的元素。对于客户端而言，只是面向这个接口在访问，根本不知道聚合对象内部的实现细节（聚合对象可以是集合，也可以是数组，客户端无从得知）。

使用迭代器模式，还可以实现很多更加丰富的功能。比如：

+ 以不同的方式遍历聚合对象，比如向前、向后等。
+ 对同一个聚合同时进行多个遍历。
+ 以不同的遍历策略来遍历聚合，比如是否需要过滤等。
+ 多态迭代，含义是：为不同的聚合结构提供统一的迭代接口，也就是说通过一个迭代接口可以访问不同的聚合结构。（多态迭代可能会带来类型安全的问题，可以考虑使用泛型）  

# 三、翻页迭代

在实际开发中，经常会碰到需要一次迭代多条数据的情况，比如常用的翻页功能。翻页功能有如下几种实现方式。

**（1）纯数据库实现**  
依靠SQL提供的功能实现翻页，用户每次请求翻页的数据，就会到数据库中获取相应的数据。  

**（2）纯内存实现**  
就是一次性从数据库中把需要的所有数据都取出来放到内存中，然后用户请求翻页时，从内存中获取相应的数据。

**两种方案各有优缺点**：

- 第一种方案明显是时间换空间的策略，每次获取翻页的数据都要访问数据库，运行速度相对比较慢，而且很耗数据库资源，但是节省了内存空间。
- 第二种方案是典型的空间换时间，每次是直接从内存中获取翻页的数据，运行速度快，但是很耗内存。        

**（3）纯数据库实现+纯内存实现**  
思路是这样的：如果每页显示10条记录，根据判断，用户很少翻到10页以后，那好，第一次访问的时候，就一次性从数据库中获取前10页的数据，也就是100条记录，把这100条记录放在内存里面。这样一来，当用户在前10页内进行翻页操作的时候，就不用再访问数据库了，而是直接从内存中获取数据，速度就快了。当用户想要获取第11页的数据时，才会再次访问数据库。  

# 四、迭代器模式的适用性

在以下条件下可以考虑使用迭代器模式：

- 如果你希望提供访问一个聚合对象的内容，但是又不想暴露它的内部表示的时候，可以使用迭代器模式来提供迭代器接口，从而让客户端只是通过迭代器的接口来访问聚合对象，而无须关心聚合对象的内部实现。
- 如果你希望有多种遍历方式可以访问聚合对象，可以使用迭代器模式。
- 如果你希望为遍历不同的聚合对象提供一个统一的接口，可以使用迭代器模式。  

# 五、迭代器模式的优点

**更好的封装性**

- 迭代器模式可以让你访问一个聚合对象的内容，而无须暴露该聚合对象的内部细节，从而提高聚合对象的封装性。

**可以以不同的遍历方式来遍历一个聚合**

- 使用迭代器模式，使得聚合对象的内容和具体的迭代算法分离开。这样就可以通过使用不同的迭代器的实例、不同的遍历方式来遍历一个聚合对象了。

**实现功能分离、简化聚合的接口**

- 有了迭代器的接口，则聚合对象只需要实现自身的基本功能，把迭代的功能委让给外部的迭代器去实现，实现了功能分离，符合“单一职责”原则。

**简化客户端调用**

- 迭代器为遍历不同的聚合对象提供了一个统一的接口，一方面方便调用；另一方面使得客户端不必关注迭代器的实现细节。

**同一个聚合上可以有多个遍历**

- 每个迭代器保持它自己的遍历状态，比如前面示例中的迭代索引位置，因此可以对同一个聚合对象同时进行多个遍历。  

# 六、总结

聚合对象的类型很多，如果对聚合对象的迭代访问跟聚合对象本身融合在一起的话，会严重影响到聚合对象的可扩展性和可维护性。

迭代器模式通过把对聚合对象的遍历和访问从聚合对象中分离出来，放入到单独的迭代器中，使得聚合对象变得简单；而且迭代器和聚合对象可以独立地变化和发展，大大加强了系统的灵活性。 