---
title: 设计模式系列之--原型模式
date: 2020-07-18 12:05:00
updated: 2020-07-18 12:05:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 原型模式
type: 
description: 什么是原型模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
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
# 一、什么是原型模式

原型（Prototype）模式是一种对象创建型模式，它通过原型实例指定创建对象的种类，并采用拷贝原型实例的方法来创建新的对象。所以，使用原型模式创建的实例，具有与原型实例一样的数据。  

# 二、原型模式实现

原型模式主要用于对象的复制，Prototype类需要具备以下两个条件()：

- 实现Cloneable接口。在java语言有一个Cloneable接口，它的作用只有一个，就是在运行时通知虚拟机可以安全地在实现了此接口的类上使用clone方法。在java虚拟机中，只有实现了这个接口的类才可以被拷贝，否则在运行时会抛出CloneNotSupportedException异常。
- 重写Object类中的clone方法。Java中，所有类的父类都是Object类，Object类中有一个clone方法，作用是返回对象的一个拷贝，但是其作用域protected类型的，一般的类无法调用，因此，Prototype类需重写clone方法并将的方法的作用域修改为public类型。

原型模式是一种比较简单的模式，也非常容易理解，实现一个接口，重写一个方法即完成了原型模式。在实际应用中，原型模式很少单独出现，经常与其他模式混用，他的原型类Prototype也常用抽象类来替代。

原型模式的表现形式：

- 简单形式
- 登记形式

## 简单形式
Prototype类为抽象时，示例代码如下。
抽象原型角色：需实现Cloneable接口，并为子类提供公共方法。
```Java
	public abstract class Prototype implements Cloneable {
		/**
		 * 为子类提供一个复制方法
		 */
		protected abstract Prototype clone();
		/**
		 * 以下两个方法为测试用，为每一个Prototype取上名字，方便区分
		 */
		protected abstract String getName();
		protected abstract void setName(String name);
	}
```
具体原型角色：需继承自抽象原型类，并实现父类的抽象方法。
```Java
	public class ConcretePrototypeA extends Prototype {
		/**
		 * 为ConcretePrototypeA的实例增加一个name属性，方便区分
		 */
		protected String name;
		@Override
		public String getName() {
			return name;
		}
		@Override
		public void setName(String name) {
			this.name = name;
		}
		/**
		 * 实现clone方法
		 */
		@Override
		public Prototype clone() {
			Prototype prototype = new ConcretePrototypeA();
			prototype.setName(this.name);
			System.out.println("ConcretePrototypeA创建完成！");
			return prototype;
		}
	}
```
<br/>
```Java
	public class ConcretePrototypeB extends Prototype {
		/**
		 * 为ConcretePrototypeA的实例增加一个name属性，方便区分
		 */
		protected String name;
		@Override
		public String getName() {
			return name;
		}
		@Override
		public void setName(String name) {
			this.name = name;
		}
		/**
		 * 实现clone方法
		 */
		@Override
		protected Prototype clone() {
			Prototype prototype = new ConcretePrototypeB();
			prototype.setName(this.name);
			System.out.println("ConcretePrototypeB创建完成！");
			return prototype;
		}
	}
```
创建一个客户端测试一下：
```Java
	public class Client{
		public static void main(String[] args) {
			/**
			 * 测试代码，首先创建好两个原型实例，以后创建新的Prototype对象就靠复制它了
			 */
			Prototype prototypeA=null;
			Prototype prototypeB=null;
			/**
			 * 为原型实例起个名字
			 */
			prototypeA=new ConcretePrototypeA();
			prototypeA.setName("A1");
			/**
			 * 为原型实例起个名字
			 */
			prototypeB=new ConcretePrototypeB();
			prototypeB.setName("B1");
			/**
			 * 复制实例
			 */
			Prototype copyA=prototypeA.clone();
			Prototype copyB=prototypeB.clone();
			/**
			 * 看看新复制的实例名字是否和原型一样
			 */
			System.out.println("原型A的名字："+prototypeA.getName());
			System.out.println("原型B的名字："+prototypeB.getName());
			System.out.println("拷贝A的名字："+copyA.getName());
			System.out.println("拷贝B的名字："+copyB.getName());
			/**
			 * 看看是不是真正的复制了一份
			 */
			copyA.setName("备份A1");
			System.out.println("拷贝A的名字："+copyA.getName());	
			System.out.println("原型A的名字："+prototypeA.getName());	
		}
	}
```
运行程序打印结果如下：
```
ConcretePrototypeA创建完成！
ConcretePrototypeB创建完成！
原型A的名字：A1
原型B的名字：B1
拷贝A的名字：A1
拷贝B的名字：B1
拷贝A的名字：备份A1
原型A的名字：A1
```

由测试结果可以看出，在对拷贝A的name属性进行修改并未影响原型对象，拷贝A确实是复制而来的新对象。

使用原型模式创建对象无需关心这个实例本身的类型，只要它实现了clone方法，就可以通过这个方法来获取新的对象，而无须再去通过new来创建。

## 登记形式
该形式与简单形式的区别在于多增加了一个PrototypeManager(原型管理器)角色，该角色对要复制的原型对象进行登记管理，这个角色提供必要的方法供外界增加新的原型对象和取得已经登记过的原型对象。

该形式的抽象原型角色Prototype、具体原型角色ConcretePrototypeA、具体原型角色ConcretePrototypeB与简单形式完全相同。

原型管理器角色：此处以Map为容器对要复制的原型进行存储登记，PrototypeManager示例代码如下。
```Java
	import java.util.HashMap;
	import java.util.Map;
	public class PrototypeManager {
		/**
		 * 此处以Map为例，为每一个要复制的原型实例进行存储登记
		 */
		private static Map<String, Prototype> map = new HashMap<String, Prototype>();
		/**
		 * 私有化构造方法，避免外部创建管理器
		 */
		private PrototypeManager() {
		}
		/**
		 * 向原型管理器里面登记原型对象或是修改某个原型对象登记信息
		 * 
		 * @param prototypeIndex
		 *            原型索引
		 * @param prototype
		 *            原型实例
		 */
		public synchronized static void setPrototype(String prototypeIndex,
				Prototype prototype) {
			map.put(prototypeIndex, prototype);
		}
		/**
		 * 从原型管理器里面删除某个登记好的原型对象
		 * 
		 * @param prototypeIndex
		 *            原型索引
		 */
		public synchronized static void removePrototype(String prototypeIndex) {
			map.remove(prototypeIndex);
		}
		/**
		 * 获取某个原型索引对应的原型实例
		 * 
		 * @param prototypeIndex
		 *            原型索引
		 * @return 原型索引对应的原型实例
		 * @throws Exception
		 *             如果原型索引对应的实例不存在，则抛出异常
		 */
		public synchronized static Prototype getPrototype(String prototypeIndex)
				throws Exception {
			Prototype prototype = map.get(prototypeIndex);
			if (prototype == null) {
				throw new Exception("该原型实例不存在！");
			}
			return prototype;
		}
	}
```
再创建一个客户端来测试一下：
```Java
	public class MainClass {
		public static void main(String[] args) throws Exception {
			/**
			 * 测试代码，首先创建好两个原型实例，以后创建新的Prototype对象就靠复制它了
			 */
			Prototype prototypeA = null;
			Prototype prototypeB = null;
			/**
			 * 为原型实例起个名字
			 */
			prototypeA = new ConcretePrototypeA();
			prototypeA.setName("A");
			/**
			 * 为原型实例起个名字
			 */
			prototypeB = new ConcretePrototypeB();
			prototypeB.setName("B");
			/**
			 * 将创建好的原型实例在PrototypeManager中进行登记
			 */
			PrototypeManager.setPrototype("A", prototypeA);
			PrototypeManager.setPrototype("B", prototypeB);
			/**
			 * 使用PrototypeManager来对登记好的原型进行选择复制 我想要以 name 为 "A" 的原型为模板进行复制
			 */
			Prototype copyA = PrototypeManager.getPrototype("A").clone();
			/**
			 * 使用PrototypeManager来对登记好的原型进行选择复制 我想要以 name 为 "B" 的原型为模板进行复制
			 */
			Prototype copyB = PrototypeManager.getPrototype("B").clone();
			/**
			 * 看看新复制的实例名字是否和你想要的一样
			 */
			System.out.println("原型A的名字：" + prototypeA.getName());
			System.out.println("原型B的名字：" + prototypeB.getName());
			System.out.println("拷贝A的名字：" + copyA.getName());
			System.out.println("拷贝B的名字：" + copyB.getName());
			/**
			 * 看看是不是真正的复制了一份
			 */
			copyA.setName("备份A1");
			System.out.println("拷贝A的名字：" + copyA.getName());
			System.out.println("原型A的名字：" + prototypeA.getName());
		}
	}
```
运行程序打印结果如下：
```Java
ConcretePrototypeA创建完成！
ConcretePrototypeB创建完成！
原型A的名字：A
原型B的名字：B
拷贝A的名字：A
拷贝B的名字：B
拷贝A的名字：备份A1
原型A的名字：A
```

使用登记形式原型模式在要复制的原型实例种类比较多而且要使用的原型实例经常变化的情况下比较方便，你只要写好原型实例并在原型管理器中登记好(起一个自己能够区分的名字作索引)，要复制哪个原型就从管理器直接获取。 

# 三、Java中的深拷贝与浅拷贝(或深度克隆与浅度克隆)

Java的所有类都是从java.lang.Object类继承而来的，而Object类提供protected Object clone()方法对对象进行复制，但Object类的clone方法只会拷贝对象中的基本的数据类型，对于数组、容器对象、引用类型对象等都不会拷贝，这就是所谓浅拷贝。如果要实现深拷贝，必须将原型模式中的数组、容器对象、引用对象等另行拷贝。

注意事项:

- Java语言提供的Cloneable接口只起一个作用，就是在运行时期通知Java虚拟机可以安全地在这个类上使用clone()方法。通过调用这个clone()方法可以得到一个对象的复制。由于Object类本身并不实现Cloneable接口，因此如果所考虑的类没有实现Cloneable接口时，调用clone()方法会抛出CloneNotSupportedException异常。
- 使用原型模式复制对象不会调用类的构造方法。因为对象的复制是通过调用Object类的clone方法来完成的，它直接在内存中复制数据，因此不会调用到类的构造方法。不但构造方法中的代码不会执行，甚至连访问权限都对原型模式无效。还记得单例模式吗？单例模式中，只要将构造方法的访问权限设置为private型，就可以实现单例。但是clone方法直接无视构造方法的权限，所以，单例模式与原型模式是冲突的，在使用时要特别注意。

把对象写到流里的过程是序列化(Serialization)过程；而把对象从流中读出来的过程则叫反序列化(Deserialization)过程。应当指出的是，写到流里的是对象的一个拷贝，而原对象仍然存在于JVM里面。

在Java语言里深度复制一个对象，常常可以先使对象实现Serializable接口，然后把对象（实际上只是对象的拷贝）写到一个流里（序列化），再从流里读回来（反序列化），便可以重建对象。

利用序列化实现深度克隆：
先创建一个Person类，该类需实现Serializable接口，为测试其是否为深拷贝为其添加一个List容器属性family，代码如下。 
```Java
	import java.io.ByteArrayInputStream;
	import java.io.ByteArrayOutputStream;
	import java.io.IOException;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutputStream;
	import java.io.Serializable;
	import java.util.List;
	@SuppressWarnings("serial")
	public class Person implements Serializable{
		// 姓名
		private String name;
		// 家庭成员
		private List<String> family;
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
		public List<String> getFamily() {
			return family;
		}
		public void setFamily(List<String> family) {
			this.family = family;
		}
		public Person serializationClone() throws IOException,
				ClassNotFoundException {
			// 将对象写到流里
			ByteArrayOutputStream bos = new ByteArrayOutputStream();
			ObjectOutputStream oos = new ObjectOutputStream(bos);
			oos.writeObject(this);
			// 从流里读回来
			ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
			ObjectInputStream ois = new ObjectInputStream(bis);
			return (Person) ois.readObject();
		}
	}
```
依旧创建一个客户端类来测试。
```Java
	import java.util.ArrayList;
	import java.util.List;
	public class MainClass {
		public static void main(String[] args) throws Exception {
			/**
			 * 创建一个原型实例对象，以后复制它
			 */
			Person person = new Person();
			/**
			 * 为原型实例对象起个名字
			 */
			person.setName("demo");
			/**
			 * 给原型实例对象添加好家人，以测试serializationClone()方法是否对引用类型对象进行复制
			 */
			List<String> family = new ArrayList<String>();
			family.add("wife");
			family.add("child");
			person.setFamily(family);
			/**
			 * 使用serializationClone()方法进行复制
			 */
			Person copyPerson=person.serializationClone();
			/**
			 * 查看复制好的对象是否和原型实例相同
			 */
			System.out.println("原型实例的名字:"+person.getName());
			System.out.println("原型实例的家人:"+person.getFamily());
			System.out.println("复制对象的名字:"+copyPerson.getName());
			System.out.println("复制对象的家人:"+copyPerson.getFamily());
			/**
			 * 对复制好的对象属性进行修改，进一步测试该对象是否是真正的备份
			 */
			copyPerson.setName("copy-demo");
			List<String> copyFamily = new ArrayList<String>();
			copyFamily.add("copy-wife");
			copyFamily.add("copy-child");
			copyPerson.setFamily(copyFamily);
			/**
			 * 打印修改后的复制对象和原型对象
			 */
			System.out.println("复制对象修改后的名字:"+copyPerson.getName());
			System.out.println("复制对象修改后的家人:"+copyPerson.getFamily());
			System.out.println("原型实例的名字:"+person.getName());
			System.out.println("原型实例的家人:"+person.getFamily());
		}
	}
```
运行程序打印结果如下：
```
原型实例的名字:demo
原型实例的家人:[wife, child]
复制对象的名字:demo
复制对象的家人:[wife, child]
复制对象修改后的名字:copy-demo
复制对象修改后的家人:[copy-wife, copy-child]
原型实例的名字:demo
原型实例的家人:[wife, child]
```

大功告成，person原型对象被完美深度复制了。

这样做的前提就是对象以及对象内部所有引用到的对象都是可序列化的，否则，就需要仔细考察那些不可序列化的对象可否设成transient，从而将之排除在复制过程之外。

浅拷贝显然比深拷贝更容易实现，因为Java语言的所有类都会继承一个clone()方法，而这个clone()方法所做的正是浅拷贝。

有一些对象，比如线程(Thread)对象或Socket对象，是不能简单复制或共享的。不管是使用浅度克隆还是深度克隆，只要涉及这样的间接对象，就必须把间接对象设成transient而不予复制；或者由程序自行创建出相当的同种对象，权且当做复制件使用。

# 四、原型模式应用场景

在以下情况可以考虑使用原型模式：

- 在创建对象的时候，我们不只是希望被创建的对象继承其基类的基本结构，还希望继承原型对象的数据。
- 希望对目标对象的修改不影响既有的原型对象（深度克隆的时候可以完全互不影响）。
- 隐藏克隆操作的细节。很多时候，对对象本身的克隆需要涉及到类本身的数据细节。   

# 五、原型模式的特点

1. 由原型对象自身创建目标对象。也就是说，对象创建这一动作发自原型对象本身。
2. 目标对象是原型对象的一个克隆。也就是说，通过Prototype模式创建的对象，不仅仅与原型对象具有相同的结构，还与原型对象具有相同的值。
3. 根据对象克隆深度层次的不同，有浅度克隆与深度克隆。

**优点：**

- 使用原型模式创建对象比直接new一个对象在性能上要好的多，因为Object类的clone方法是一个本地方法，它直接操作内存中的二进制流，特别是复制大对象时，性能的差别非常明显。
- 使用原型模式的另一个好处是简化对象的创建，使得创建对象就像我们在编辑文档时的复制粘贴一样简单。

因为以上优点，所以在需要重复地创建相似对象时可以考虑使用原型模式。比如需要在一个循环体内创建对象，假如对象创建过程比较复杂或者循环次数很多的话，使用原型模式不但可以简化创建过程，而且可以使系统的整体性能提高很多。

**缺点：**

- 原型模式最主要的缺点是每一个类都必须配备一个克隆方法。配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类来说不是很难，而对于已经有的类不一定很容易，特别是当一个类引用不支持序列化的间接对象，或者引用含有循环结构的时候。  

# 六、参考文章

[http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html](http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html "《JAVA与模式》之原型模式")  

[http://blog.csdn.net/jason0539/article/details/23158081](http://blog.csdn.net/jason0539/article/details/23158081 "JAVA设计模式之原型模式")

