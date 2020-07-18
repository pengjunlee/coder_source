---
title: 设计模式系列之--简单工厂模式
date: 2020-07-18 12:01:00
updated: 2020-07-18 12:01:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 简单工厂
type: 
description: 什么是简单工厂模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
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
# 一、什么是简单工厂模式

简单工厂模式属于类的创建型模式。提供一个创建对象实例的功能，而无须关心其具体实现。被创建实例的类型可以是接口、抽象类，也可以是具体的类。 

`简单工厂模式的本质：选择实现 `

设计意图： 通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

# 二、简单工厂模式的结构

简单工厂模式涉及的角色及其职责如下：

- 工厂（Factory）角色：简单工厂模式的核心，它负责实现创建所有实例的内部逻辑，该类可以被外界直接调用，创建所需的产品对象。  
- 抽象（Product）角色：简单工厂模式所创建的所有对象的父类，它负责约定所有具体产品类所共有的公共接口。  
- 具体产品（ConcreteProduct）角色：简单工厂模式所创建的真正实例对象。

以具体代码为例： 
现在有两种水果：苹果(Apple)和香蕉(Banana)，为了演示，为这两种水果添加一个采集get()方法。

```Java
	public class Apple{
	    /*
	     * 采集
	     */
	     public void get(){
	        System.out.println("采集苹果");
	    }
	}
```
<br>
```Java
	public class Banana{
	    /*
	     * 采集
	     */
	    public void get(){
	        System.out.println("采集香蕉");
	    }
	}
```
如果要采集水果，在不使用任何设计模式的情况下，我们一般都会这样做。
```Java
	public class Client {
	    public static void main(String[] args){
	 
	        //实例化一个Apple
	        Apple apple = new Apple();
	        //实例化一个Banana
	        Banana banana = new Banana();
	        apple.get();
	        banana.get();
	    }
	}
```
客户端分别创建苹果和香蕉两个具体产品的实例并调用其采集的方法，这样做存在的问题是：客户端需要知道所有的具体水果种类，即客户端与所有的水果类都存在了耦合关系，不符合面向对象设计中的“低耦合”原则。

以上问题如何解决呢？  
答案：使用简单工厂，利用接口进行“封装隔离”。 

Apple和Banana类既然都是水果，且共有一个采集的get()方法，我们可以为其添加一个父类或让其实现一个共同的接口。在此，我们添加一个Fruit接口。  
```Java
	public interface Fruit {
	    /*
	     * 采集
	     */
	    public void get();
	}
```
让Apple和Banana类都实现Fruit接口。  
```Java
	public class Apple implements Fruit{
	    /*
	     * 采集
	     */
	     public void get(){
	        System.out.println("采集苹果");
	    }
	}
```
<br>
```Java
	public class Banana implements Fruit{
	    /*
	     * 采集
	     */
	     public void get(){
	        System.out.println("采集香蕉");
	    }
	}
```
接下来创建一个水果工厂类FruitFactory，在工厂中实现对Apple和Banana类的实例化。  
```Java
	public class FruitFactory {
	    /*
	     * 获得Apple类的实例
	     */
	    public static  Fruit getApple(){
	        return new Apple();
	    }
	     
	    /*
	     * 获得Banana类实例
	     */
	    public static Fruit getBanana(){
	        return new Banana();
	    }
	}
```
若FruitFactory中getXXX()方法未声明为static的话，每次调用方法需创建FruitFactory实例比较麻烦，我们直接声明方法为static。  
这样，我们就能够使用FruitFactory创建Apple和Banana实例对象了。 
```Java
	public class Client{
	    public static void main(String[] args){
	 
	     //实例化一个Apple
	        Fruit apple = FruitFactory.getApple();
	        Fruit banana = FruitFactory.getBanana();
	        apple.get();
	        banana.get();
	    }
	}
```
观察以上工厂，其为每一个水果具体类都添加了一个getXXX()方法，当水果种类增多时方法数量也会增加导致FruitFactory类比较臃肿，而且各个方法之间关系也不够紧密不符合“高内聚”原则，我们再对其进行优化。 
```Java
	public class FruitFactory {
	    /*
	     * getFruit方法，获得所有产品对象
	     */
	     public static Fruit getFruit(String type) throws InstantiationException, IllegalAccessException{
	        if(type.equalsIgnoreCase("apple")) {
	            return Apple.class.newInstance();
	        } elseif(type.equalsIgnoreCase("banana")) {
	            return Banana.class.newInstance();
	        } else{
	            System.out.println("找不到相应的实例化类");
	            return null;
	        }
	    }
	}
```
在客户端，使用改良后的FruitFactory来创建Apple和Banana实例对象。
```Java
	public class Client{
	    public static void main(String[] args){
	        Fruit apple = FruitFactory.getFruit("apple");
	        Fruit banana = FruitFactory.getFruit("banana");
	        apple.get();
	        banana.get();
	    }
	}
```
运行程序打印结果如下： 
```
采集苹果  
采集香蕉
```

# 三、简单工厂模式的优缺点
在这个模式中，工厂类是整个模式的关键所在。它包含必要的判断逻辑，能够根据外界给定的信息，决定究竟应该创建哪个具体类的对象。用户在使用时可以直接根据工厂类去创建所需的实例，而无需了解这些对象是如何创建以及如何组织的。有利于整个软件体系结构的优化。不难发现，简单工厂模式的缺点也正体现在其工厂类上，由于工厂类集中了所有实例的创建逻辑，所以“高内聚”方面做的并不好。另外，当系统中的具体产品类不断增多时，可能会出现要求工厂类也要做相应的修改，扩展性并不很好。例如新增一个Pear(梨)类时，FruitFactory需在getFruit()方法中增加type=="Pear"的逻辑判断，此时不符合开闭原则。

FruitFactory可通过反射来解决扩展性和高内聚缺陷，代码如下。
```Java
	public class FruitFactory{
	    /*
	     * getFruit方法，获得所有产品对象
	     */
	     public static Fruit getFruit(String type)throws ClassNotFoundException, InstantiationException,IllegalAccessException {
	        Class fruit = Class.forName(type);
	        return (Fruit) fruit.newInstance();
	    }
	}
```
客户端使用FruitFactory创建Apple和Banana实例对象，此时getFruit()方法传入的参数需注意区分大小写，否则会抛出
ClassNotFoundException异常，写法呆板，但可以解决扩展性和高内聚缺陷。
```Java
	public class Client{
	    public static void main(String[] args){
	 
	        Fruit apple = FruitFactory.getFruit("Apple");
	        Fruit banana = FruitFactory.getFruit("Banana");
	        apple.get();
	        banana.get();
	    }
	}
```