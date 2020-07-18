---
title: 设计模式系列之--工厂方法模式
date: 2020-07-18 12:02:00
updated: 2020-07-18 12:02:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 工厂方法
type: 
description: 什么是工厂方法模式？
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
# 一、什么是工厂方法模式

工厂方法模式同样属于类的创建型模式又被称为多态工厂模式 。工厂方法模式的意义是定义一个创建产品对象的工厂接口，将实际创建工作推迟到子类当中。核心工厂类不再负责产品的创建，这样核心类成为一个抽象工厂角色，仅负责声明具体工厂子类必须实现的接口，这样进一步抽象化的好处是使得工厂方法模式可以使系统在不修改具体工厂角色的情况下引进新的产品。 

# 二、模式中包含的角色及其职责

抽象工厂（Factory）角色：工厂方法模式的核心，所有具体工厂类都必须实现这个接口。

- 具体工厂（ ConcreteFactory）角色：具体工厂类是抽象工厂的一个实现，负责实例化具体的产品。
- 抽象产品（Product）角色：工厂方法模式所创建的所有具体产品的父类，或约定所有具体产品类都应实现的公共接口。
- 具体产品（ConcreteProduct）角色：工厂方法模式所创建的真正对象。

在简单工厂模式中我们了解了简单工厂模式，我们使用如下具体的工厂FruitFactory对所有的水果对象进行实例化，缺点是显而易见的：当有新品种的水果需要产生时就需要修改工厂的getFruit()方法，加入新品种水果的逻辑判断和业务代码，这极不符合java“开放--封闭”原则。 
```Java
	public class FruitFactory {
	    /*
	     * get方法，获得所有产品对象
	     */
	     public static Fruit getFruit(String type) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
	        if(type.equalsIgnoreCase("apple")) {
	            return Apple.class.newInstance();
	            
	        } else if(type.equalsIgnoreCase("banana")) {
	            return Banana.class.newInstance();
	        } 
	        /*
	         * 如果有新品种的水果需要生产需在此处增加相应逻辑判断及业务代码，例如：
	         * else if(type.equalsIgnoreCase("pear")) {
	         *         return Pear.class.newInstance();
	         *    } 
	         */
	         else {
	            System.out.println("找不到相应的实例化类");
	            return null;
	        }
	    }
	}
```
接下来我们使用工厂方法模式对以上工厂进行改进，首先工厂方法模式中FruitFactory被设计为了抽象工厂(可以是抽象类或者接口),它不再负责具体的产品创建，而是将产品的创建工作推迟到了由它的子类来实现，接下来以具体的代码为例。

抽象后的FruitFactory仅用来声明其所有子类需实现的接口。
```Java
	public interface FruitFactory {
	    public Fruit getFruit();
	}
```
有了这个抽象工厂以后，假如我们需要苹果了该怎么办？很简单，创建一个生产苹果的工厂AppleFactory即可。
```Java
	public class AppleFactory implements FruitFactory{
	    public Fruit getFruit(){
	        return new Apple();
	    }
	}
```
假如我们又需要获取香蕉了该怎么办？依旧简单，创建一个生产香蕉的工厂BananaFactory即可。
```Java
	public class BananaFactory implements FruitFactory{
	    public Fruit getFruit(){
	        return new Banana();
	    }
	}
```
如果以后还有其他更多新品种的水果加入，只需要创建一个生产相应水果的工厂并让该工厂实现抽象工厂FruitFactory 的接口，需要哪种水果我们就调用相应工厂的getFruit()方法得到水果，这样做有一个好处，无论我们增加多少种水果，我们只需增加一个生产相应水果的工厂即可，无需对现有的代码进行修改，这就很好的符合了"开放--封闭"原则。

# 三、工厂方法模式和简单工厂模式比较
工厂方法模式与简单工厂模式在结构上的不同不是很明显。工厂方法类的核心是一个抽象工厂类，而简单工厂模式把核心放在一个具体类上。 工厂方法模式之所以有一个别名叫多态性工厂模式是因为具体工厂类都有共同的接口，或者有共同的抽象父类。当系统扩展需要添加新的产品对象时，仅仅需要添加一个具体对象以及一个具体工厂对象，原有工厂对象不需要进行任何修改，也不需要修改客户端，很好的符合了“开放－封闭”原则。而简单工厂模式在添加新产品对象后不得不修改工厂方法，扩展性不好。

工厂方法模式退化后可以演变成简单工厂模式。   