---
title: 设计模式系列之--桥接模式
date: 2020-07-18 12:16:00
updated: 2020-07-18 12:16:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 桥接模式
type: 
description: 什么是桥接模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
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
# 一、什么是桥接模式

桥接(Bridge)模式是构造型的设计模式之一。桥接模式基于类的最小设计原则，通过使用封装，聚合以及继承等行为来让不同的类承担不同的责任。它的主要特点是把抽象(abstraction)与行为实现(implementation)分离开来，从而可以保持各部分的独立性以及应对它们的功能扩展。  

# 二、桥接模式的应用场景

《研磨设计模式》中给出这么一个场景：发送消息。

- 从消息的重要程度上来看，消息可以分为普通消息、加急消息两种，两种消息的业务功能处理是不一样的，比如普通消息直接发送即可，而加急消息除了需在发送的消息上添加加急标识外，还需提供监控功能，以方便客户端了解加急消息的处理进度。
- 从发送消息的方式上来看，消息又可以分为站内短消息和E-Mail两种。

在不使用设计模式的情况下，我们一般会这么做：
考虑到普通消息和加急消息都各有站内短消息和E-Mail两种发送方式，为了让外部能统一操作，因此把消息设计成接口，普通消息的两种发送方式分别作为该接口的两个实现，但加急消息除了有基本的发送消息的功能，还需要添加监控功能，因此将加急消息也抽取为接口，加急消息的两种发送方式做为接口的两个实现。

这个时候，系统的类图结构设计如下。

<div align=center>

![类图结构1](http://pengjunlee.3vzhuji.net/static/design_pattern/18.png "类图结构1")
<div align=left>

仔细观察这个类图，会发现一个很明显的问题，通过这种继承的方式来扩展消息处理，会非常不方便。举个例子，假如我们要在此基础上添加一个特急消息的处理：特急消息不需要进行监控，只要没有完成，就直接催促，也就是说，对于特急消息，在普通消息的基础上，需要添加催促的功能。而特急消息的发送方式还是发送站内短消息和E-Mail两种，此时的类图结构如下。 

<div align=center>

![类图结构2](http://pengjunlee.3vzhuji.net/static/design_pattern/19.png "类图结构2")
<div align=left>

从以上过程可以看到，我们在实现加急消息处理的时候，必须实现站内短消息和E-Mail两种处理方式；在实现特急消息处理的时候，又必须实现站内短消息和E-Mail两种处理方式。

这意味着，以后每次扩展一种消息处理，都必须要实现站内短消息和E-Mail两种处理方式，是不是很痛苦？

这还不算完，如果要添加新的消息发送方式呢？继续扩展功能，添加手机短消息的发送方式，类图结构扩展如下。  

<div align=center>

![类图结构3](http://pengjunlee.3vzhuji.net/static/design_pattern/20.png "类图结构3")
<div align=left>

仔细观察现在的实现，如果此时再添加一种消息处理，则需要添加系统内短消息、邮件和手机消息三个实现。按照这种实现方式继续扩展下去，消息类的数量会急剧增加，极易出现“类膨胀”。

遇到类似以上这种情况，应该如何解决呢？答案：使用桥接模式。 

# 三、桥接模式的结构

仔细分析上面的示例，会发现示例中消息的变化具有两个维度，一个维度是抽象的消息这边，包括普通消息、加急消息和特急消息；另一个维度是在消息的发送方式上，包括站内短消息、E-Mail和手机短消息。这两个维度一共可以组合出9种不同的可能性来，它们的关系如下。 

<div align=center>

![类图结构4](http://pengjunlee.3vzhuji.net/static/design_pattern/21.png "类图结构4")
<div align=left>

现在出现问题的根本原因，就在于消息的抽象和实现是混杂在一起的，这就导致一个维度的变化会引起另一个维度进行相应的变化，从而使得程序扩展起来非常困难。 要解决这个问题，就必须把这两个维度分开，也就是将抽象部分和实现部分分开，让它们相互独立，这样就可以实现独立的变化，使扩展变得简单。桥接模式就可以达到这种效果，以下是桥接模式的结构示意图。 

<div align=center>

![桥接模式应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/22.png "桥接模式应用示意图")
<div align=left>

桥接模式涉及的角色及其职责如下：

+ Abstraction：抽象部分的接口。通常在这个对象中，要维护一个实现部分的对象引用，抽象对象里面的方法，需要调用实现部分的对象来完成，这个对象中的方法，通常都是和具体的业务相关的方法。
+ RefinedAbstraction：扩展抽象部分的接口。通常在这些对象中，定义跟实际业务相关的方法，这些方法实现通常会使用Abstraction中定义的方法，也可能会调用实现部分的对象来完成。
+ Implementor：定义实现部分的接口。这个接口不用和Abstraction中定义的方法一致，通常是由Implementor接口提供基本的操作。而 Abstraction中定义的是基于这些基本操作的业务方法，也就是说 Abstraction定义了基于这些基本操作的较高层次的操作。
+ ConcreteImplementor：真正实现 Implementor接口的对象。 
 
桥接模式结构示意源代码如下： 
```Java
	public abstract class Abstraction {
	 
		/**
		 * 持有一个实现部分的对象
		 */
		protected Implementor impl;
	 
		/**
		 * 构造方法，传入一个实现部分的对象
		 */
		public Abstraction(Implementor impl) {
			this.impl = impl;
		}
	 
		/**
		 * 示例操作，实现一定的功能，可能需要调用实现部分的具体实现方法
		 */
		public void operation() {
			impl.operationImpl();
		}
	}
```
<br/> 
```Java
	public interface Implementor {  
	  
	    /** 
	     * 示例方法，实现抽象部分需要的某些具体功能 
	     */  
	    public void operationImpl();  
	} 
```
<br/> 
```Java
	public class RefinedAbstraction extends Abstraction {  
	  
	    public RefinedAbstraction(Implementor impl) {  
	        super(impl);  
	    }  
	  
	    /** 
	     * 示例操作，实现一定的功能 
	     */  
	    public void otherOperation() {  
	        // 实现一定的功能，可能会使用具体实现部分的实现方法  
	        // 但是本方法更大的可能是使用Abstraction中定义的方法  
	        // 通过组合使用Abstraction中定义的方法来完成更多的功能  
	  
	    }  
	}
```
<br/> 
 ```Java 
	public class ConcreteImplementorA implements Implementor {
	 
		@Override
		public void operationImpl() {
			// 真正的实现
	 
		}
	 
	}
```
<br/> 
```Java
	public class ConcreteImplementorB implements Implementor {  
	  
	    @Override  
	    public void operationImpl() {  
	        // 真正的实现  
	  
	    }  
	  
	}
```

# 四、使用桥接模式重写消息

先实现普通消息和加急消息的功能，发送方式先实现站内短消息和E-mail两种，使用桥接模式来实现这些功能的类图结构如下。

<div align=center>

![重写消息示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/23.png "重写消息示意图")
<div align=left>

下面是各个类的实现代码。  
```Java
	/**
	 * 抽象的消息对象
	 */
	public abstract class AbstractMessage {
	 
		/**
		 * 持有一个实现部分的对象
		 */
		protected MessageImplementor impl;
	 
		/**
		 * 构造方法，传入实现部分的对象
		 * @param impl 实现部分的对象
		 */
		public AbstractMessage(MessageImplementor impl) {
			this.impl = impl;
		}
	 
		/**
		 * 发送消息，转调实现部分的方法
		 * @param message 消息的内容
		 * @param toUser  消息的接收人
		 */
		public void sendMessage(String message, String toUser) {
			this.impl.send(message, toUser);
		}
	}
```
<br/> 
```Java
	/**
	 * 实现发送消息的统一接口
	 */
	public interface MessageImplementor {
		
		/**
		 * 发送消息
		 * 
		 * @param message 消息的内容
		 * @param toUser  消息的接收人
		 */
		public void send(String message, String toUser);
	 
	}
```
<br/> 
```Java
	public class CommonMessage extends AbstractMessage {
	 
		public CommonMessage(MessageImplementor impl) {
			super(impl);
		}
		
		public void sendMessage(String message, String toUser){
			//对于普通消息，什么都不干，直接调用父类的方法，把消息发送出去
			super.sendMessage(message, toUser);
		}
	}
```
<br/> 
```Java
	public class UrgencyMessage extends AbstractMessage {
	 
		public UrgencyMessage(MessageImplementor impl) {
			super(impl);
		}
	 
		public void sendMessage(String message, String toUser) {
			// 对于加急消息，在消息上添加“加急”标识，再调用父类的方法，把消息发送出去
			message = "[加急]" + message;
			super.sendMessage(message, toUser);
		}
	 
		/**
		 * 扩展自己的新功能：监控某消息的处理过程
		 * @param message 被监控的消息
		 * @return 包含监控到的数据对象，此处用Object示意
		 */
		public Object watch(AbstractMessage message) {
			return null;
		}
	}
```
<br/> 
```Java
	public class MessageSMS implements MessageImplementor {
	 
		@Override
		public void send(String message, String toUser) {
			System.out.println("使用站内短消息的方式，发送消息:"+message+"给"+toUser);
	 
		}
	}
```
<br/> 
```Java
	public class MessageEmail implements MessageImplementor {
	 
		@Override
		public void send(String message, String toUser) {
			System.out.println("使用E-mail的方式，发送消息:" + message + "给" + toUser);
	 
		}
	}
```
继续添加对特急消息的处理，同时添加发送手机消息的方式，该如何实现？

很简单，只需要在抽象部分再添加一个特急消息的类，扩展抽象消息就可以把特急消息的处理功能加入到系统中；对于添加手机发送消息的方式也很简单，在实现部分新增加一个实现类，实现使用手机发送消息的方式就可以了。

新添加的两个类，代码如下。 
```Java
	public class SpecialUrgencyMessage extends AbstractMessage {
	 
		public SpecialUrgencyMessage(MessageImplementor impl) {
			super(impl);
		}
	 
		public void hurry(AbstractMessage message) {
			// 执行催促的功能，发出催促的信息
		}
	 
		public void sendMessage(String message, String toUser) {
			// 对于特急消息，在消息上添加“特急”标识，再调用父类的方法，把消息发送出去
			message = "[特急]" + message;
			super.sendMessage(message, toUser);
			// 还需要增加一条待催促的信息
		}
	}
```
<br/> 
```Java
	public class MessageMobile implements MessageImplementor {
	 
		@Override
		public void send(String message, String toUser) {
			System.out.println("使用手机短消息的方式，发送消息:" + message + "给" + toUser);
	 
		}
	}
```
写个客户端类来测试一下，代码如下。  
```Java
	public class Client {
	 
		public static void main(String[] args) {
			// 创建具体的实现对象
			MessageImplementor impl = new MessageSMS();
	 
			// 创建一个普通消息对象
			AbstractMessage m = new CommonMessage(impl);
			m.sendMessage("今晚8点，时代广场，不见不散！", "小乔");
	 
			// 创建一个紧急消息对象
			m = new UrgencyMessage(impl);
			m.sendMessage("半小时后召开紧急会议！", "小周");
	 
			// 创建一个特急消息对象
			m = new SpecialUrgencyMessage(impl);
			m.sendMessage("领导过来视察了，速回！", "小明");
	 
			// 把发送方式切换成手机短消息，将以上内容重发一遍
			impl = new MessageMobile();
	 
			// 创建一个普通消息对象
			m = new CommonMessage(impl);
			m.sendMessage("今晚8点，时代广场，不见不散！", "小乔");
	 
			// 创建一个紧急消息对象
			m = new UrgencyMessage(impl);
			m.sendMessage("半小时后召开紧急会议！", "小周");
	 
			// 创建一个特急消息对象
			m = new SpecialUrgencyMessage(impl);
			m.sendMessage("领导过来视察了，速回！", "小明");
	 
		}
	}
```
运行程序打印结果如下：  
```Java
使用站内短消息的方式，发送消息:今晚8点，时代广场，不见不散！给小乔
使用站内短消息的方式，发送消息:[加急]半小时后召开紧急会议！给小周
使用站内短消息的方式，发送消息:[特急]领导过来视察了，速回！给小明
使用手机短消息的方式，发送消息:今晚8点，时代广场，不见不散！给小乔
使用手机短消息的方式，发送消息:[加急]半小时后召开紧急会议！给小周
使用手机短消息的方式，发送消息:[特急]领导过来视察了，速回！给小明
```

# 五、桥接模式的适用性

在以下的情况下可以考虑使用桥接模式：

1．如果你不希望在抽象部分和实现部分采用固定的绑定关系，可以采用桥接模式，来把抽象部分和实现部分分开，然后在程序运行期间来动态地设置抽象部分需要用到的具体的实现，还可以动态地切换具体的实现。

2．如果出现抽象部分和实现部分都能够扩展的情况，可以采用桥接模式，让抽象部分和实现部分独立地变化，从而灵活地进行单独扩展，而不是搅在一起，扩展一边就会影响到另一边。

3．如果希望实现部分的修改不会对客户产生影响，可以采用桥接模式。由于客户是面向抽象的接口在运行，实现部分的修改可以独立于抽象部分，并不会对客户产生影响，也可以说对客户是透明的。

4．如果采用继承的实现方案，会导致产生很多子类，对于这种情况，可以考虑采用桥接模式，分析功能变化的原因，看看是否能分离成不同的维度，然后通过桥接模式来分离它们，从而减少子类的数目。 

# 六、桥接模式的特点

+ 分离抽象和实现部分

桥接模式分离了抽象部分和实现部分，从而极大地提高了系统的灵活性。让抽象部分和实现部分独立开来，分别定义接口，这有助于对系统进行分层，从而产生更好的结构化的系统。对于系统的高层部分，只需要知道抽象部分和实现部分的接口就可以了。

+ 更好的扩展性

由于桥接模式把抽象部分和实现部分分离开了，而且分别定义接口，这就使得抽象部分和实现部分可以分别独立地扩展，而不会相互影响，从而大大地提高了系统的可扩展性。

+ 可动态地切换实现

桥接模式把抽象部分和实现部分分离开了，所以在实现桥接的时候，就可以实现动态地选择和使用具体的实现。也就是说一个实现不再是固定地绑定在一个抽象接口上了，可以实现运行期间动态地切换。

+ 可减少子类的个数

对于有两个变化维度的情况，如果采用继承的实现方式，大约需要两个维度上的可变化数量的乘积个子类；而采用桥接模式来实现，大约需要两个维度上的可变化数量的和个子类。可以明显地减少子类的个数。

# 七、总结

桥接模式很好地实现了面向对象设计中的开闭原则和多用对象组合、少用对象继承原则，它的本质是：分离抽象和实现。只有把抽象部分和实现部分分离开了，才能够让它们独立地变化；只有抽象部分和实现部分可以独立地变化，系统才会有更好的可扩展性和可维护性。