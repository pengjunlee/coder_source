---
title: 设计模式系列之--职责链模式
date: 2020-07-18 12:17:00
updated: 2020-07-18 12:17:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 职责链模式
type: 
description: 什么是职责链模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img17.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img17.jpg
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
# 一、什么是职责链模式

职责链(Chain of Responsibility)模式是一种对象的行为模式。在职责链模式里，很多对象由每一个对象对其下家的引用而连接起来形成一条链。请求在这个链上传递，直到链上的某一个对象决定处理此请求。发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织和分配职责。

`职责链模式的本质：分离职责、动态组合`

设计意图：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。  

# 二、职责链模式的结构

<div align=center>

![职责链应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/24.png "职责链应用示意图")
<div align=left>

职责链模式涉及的角色及其职责如下：

+ 抽象处理者(Handler)角色：定义职责的接口，通常在这里定义处理请求的方法，如果需要，接口可以定义出一个方法以设定和返回对后继处理者的引用。
+ 具体处理者(ConcreteHandler)角色：实现职责的类，在这个类中，实现对在它职责范围内请求的处理，如果不处理，就继续转发请求给后继处理者。
+ 客户端(Client)角色：职责链的客户端，向链上的具体处理对象提交请求，让职责链负责处理。

职责链模式结构示意源代码如下：
抽象处理者的示意代码如下。
```Java
	/**
	 * 抽象处理者，定义职责的接口，也就是处理请求的接口
	 */
	public abstract class Handler {
	 
		/**
		 * 持有后继的处理者对象
		 */
		protected Handler successor;
	 
		/**
		 * 赋值方法，设置后继的处理者对象
		 * 
		 * @param successor
		 */
		public void setSuccessor(Handler successor) {
			this.successor = successor;
		}
	 
		/**
		 * 示意处理请求的方法，虽然这个示意方法是没有传入参数的 但实际是可以传入参数的，根据具体需要来选择是否传递参数
		 */
		public abstract void handleRequest();
	 
	}
```
具体处理者示意代码如下。
```Java
	public class ConcreteHandlerA extends Handler {
	 
		/**
		 * 处理方法，调用此方法处理请求
		 */
		@Override
		public void handleRequest() {
			// 根据某些条件来判断是否属于自己处理的职责范围,下面這句話只是个示意
			boolean someCondition = false;
	 
			if (someCondition) {
				// 如果属于自己处理的职责范围，就在这里处理请求
				// 具体的处理代码
			} else {
				// 如果不属于自己处理的职责范围，那就判断是否还有后继的职责对象
				// 如果有，就转发请求给后继的职责对象
				// 如果没有，什么都不做，自然结束
				if (null != this.successor) {
					this.successor.handleRequest();
				}
			}
		}
	}
```
<br/>
```Java
	public class ConcreteHandlerB extends Handler {
	 
		/**
		 * 处理方法，调用此方法处理请求
		 */
		@Override
		public void handleRequest() {
			// 根据某些条件来判断是否属于自己处理的职责范围,下面這句話只是个示意
			boolean someCondition = false;
	 
			if (someCondition) {
				// 如果属于自己处理的职责范围，就在这里处理请求
				// 具体的处理代码
			} else {
				// 如果不属于自己处理的职责范围，那就判断是否还有后继的职责对象
				// 如果有，就转发请求给后继的职责对象
				// 如果没有，什么都不做，自然结束
				if (null != this.successor) {
					this.successor.handleRequest();
				}
			}
		}
	}
```
客户端示意代码如下。
```Java
	public class Client {
	 
		public static void main(String[] args) {
			// 组装职责链
			Handler handlerA = new ConcreteHandlerA();
			Handler handlerB = new ConcreteHandlerB();
			handlerA.setSuccessor(handlerB);
			// 提交请求
			handlerA.handleRequest();
		}
	}
```

# 三、职责链模式应用举例

《研磨设计模式》中给出这么一个场景：申请聚餐费用。
很多公司都有这样的福利，就是项目组或者是部门可以向公司申请一些聚餐费用，用于组织项目组成员或者是部门成员进行聚餐活动，以增进员工之间的感情，更有利于工作中的相互合作。

申请聚餐费用的大致流程一般是：由申请人先填写申请单，然后交给领导审批，如果申请批准下来，领导会通知申请人审批通过，然后申请人去财务领取费用，如果没有批准下来，领导会通知申请人审批未通过，此事也就此作罢。

不同级别的领导，对于审批的额度是不一样的，比如，项目经理只能审批500元以内的申请；部门经理能审批1000元以内的申请；而总经理可以审核任意额度的申请。

也就是说，当某人提出聚餐费用申请的请求后，该请求会经由项目经理、部门经理、总经理之中的某一位领导来进行相应的处理，但是提出申请的人并不知道最终会由谁来处理他的请求，一般申请人是把自己的申请提交给项目经理，或许最后是由总经理来处理他的请求，但是申请人并不知道应该由总经理来处理他的申请。    

上述功能可以使用职责链模式来实现：当某人提出聚餐费用申请的请求后，该请求会在 项目经理—〉部门经理—〉总经理 这样一条领导处理链上进行传递，发出请求的人并不知道谁会来处理他的请求，每个领导会根据自己的职责范围，来判断是处理请求还是把请求交给更高级别的领导，只要有领导处理了，传递就结束了。

需要把每位领导的处理独立出来，实现成单独的职责处理对象，然后为它们提供一个公共的、抽象的父职责对象，这样就可以在客户端来动态地组合职责链，实现不同的功能要求了。 

以下是使用职责链模式实现示例的类图结构。 

<div align=center>

![职责链应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/25.png "职责链应用示意图")
<div align=left>

先定义抽象处理者角色，在这个类中持有下一个处理请求的对象，同时还要定义业务处理方法，示例代码如下：
```Java
	/**
	 * 抽象处理者，定义职责的接口，也就是处理请求的接口
	 */
	public abstract class Handler {
	 
		/**
		 * 持有后继的处理者对象
		 */
		protected Handler successor=null;
	 
		/**
		 * 赋值方法，设置后继的处理者对象
		 * 
		 * @param successor
		 */
		public void setSuccessor(Handler successor) {
			this.successor = successor;
		}
	 
		/**
		 * 处理聚餐费用的申请
		 * @param user 申请人
		 * @param fee 申请的钱数
		 */
		public abstract String handleFeeRequest(String user,double fee);
	 
	}
```
假设现在的费用申请处理流程如下：申请人提出的申请交给项目经理处理，项目经理的处理权限是500元内，超过500元，把申请转给部门经理处理，部门经理的处理权限是1000元以内，超过1000元，把申请转给总经理处理。
具体处理者代码实现如下：
```Java
	public class ProjectManager extends Handler {
	 
		public String handleFeeRequest(String user,double fee) {
			String str="";
			//项目经理处理权限比较小，只能处理500元以内的费用申请
			if(fee<500){
				//为了测试，简单点，只同意小李的申请
				if("小李".equals(user)){
					str="项目经理同意"+user+"聚餐费用"+fee+"元的请求...";
				}
				else{
				//其他人的申请一律不同意
					str="项目经理不同意"+user+"聚餐费用"+fee+"元的请求...";
				}
			}
			else{
				//超过500元的费用申请，继续传递给级别更高的人处理
				if(this.successor!=null){
					return successor.handleFeeRequest(user, fee);
				}
				
			}
			return str;
		}
	}
```
<br/>
```Java
	public class DeptManager extends Handler {
	 
		public String handleFeeRequest(String user, double fee) {
			String str = "";
			// 部门经理只能处理500元以内的费用申请
			if (fee < 1000) {
				// 为了测试，简单点，只同意小李的申请
				if ("小李".equals(user)) {
					str = "部门经理同意" + user + "聚餐费用" + fee + "元的请求...";
				} else {
					// 其他人的申请一律不同意
					str = "部门经理不同意" + user + "聚餐费用" + fee + "元的请求...";
				}
			} else {
				// 超过1000元的费用申请，继续传递给级别更高的人处理
				if (this.successor != null) {
					return successor.handleFeeRequest(user, fee);
				}
	 
			}
			return str;
		}
	}
```
<br/>
```Java
	public class GeneralManager extends Handler {
	 
		public String handleFeeRequest(String user,double fee) {
			String str="";
			//总经理的权限很大，只要请求到了这里，他都可以处理
			if(fee>=1000){
				//为了测试，简单点，只同意小李的申请
				if("小李".equals(user)){
					str="总经理同意"+user+"聚餐费用"+fee+"元的请求...";
				}
				else{
				//其他人的申请一律不同意
					str="总经理不同意"+user+"聚餐费用"+fee+"元的请求...";
				}
			}
			else{
				//如果还有后继的处理者对象，继续传递
				if(this.successor!=null){
					return successor.handleFeeRequest(user, fee);
				}
				
			}
			return str;
		}
	}
```
客户端示意代码如下： 
```Java
	public class Client {
	 
		public static void main(String[] args) {
			// 先要组装责任链
			Handler h1 = new GeneralManager();
			Handler h2 = new DeptManager();
			Handler h3 = new ProjectManager();
			h3.setSuccessor(h2);
			h2.setSuccessor(h1);
	 
			// 开始测试
			String ret1 = h3.handleFeeRequest("小李", 300);
			System.out.println("the ret1 = " + ret1);
			String ret2 = h3.handleFeeRequest("小张", 300);
			System.out.println("the ret2 = " + ret2);
			System.out.println("===========================================");
	 
			String ret3 = h3.handleFeeRequest("小李", 700);
			System.out.println("the ret3 = " + ret3);
			String ret4 = h3.handleFeeRequest("小张", 700);
			System.out.println("the ret4 = " + ret4);
			System.out.println("===========================================");
	 
			String ret5 = h3.handleFeeRequest("小李", 1500);
			System.out.println("the ret5 = " + ret5);
			String ret6 = h3.handleFeeRequest("小张", 1500);
			System.out.println("the ret6 = " + ret6);
		}
	}
```
运行程序打印结果如下：  
```
the ret1 = 项目经理同意小李聚餐费用300.0元的请求...
the ret2 = 项目经理不同意小张聚餐费用300.0元的请求...
===========================================
the ret3 = 部门经理同意小李聚餐费用700.0元的请求...
the ret4 = 部门经理不同意小张聚餐费用700.0元的请求...
===========================================
the ret5 = 总经理同意小李聚餐费用1500.0元的请求...
the ret6 = 总经理不同意小张聚餐费用1500.0元的请求...
```

# 四、职责链模式的适用性
在以下情况下可以考虑使用职责链模式：

+ 有多个的对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定。
+ 你想在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。
+ 可处理一个请求的对象集合应被动态指定。  

# 五、职责链模式的优缺点

**职责链模式有以下优点**：

+ 责任的分担。  
每个类只需要处理自己该处理的工作（不该处理的传递给下一个对象完成），明确各类的责任范围，符合类的最小封装原则。
+ 可以根据需要自由组合职责流程。  
如果职责流程发生变化，可以通过重新组装对象链便可适应新的职责流程。
+ 请求者和处理者松散耦合。  
请求者并不知道他发起的请求的处理者是谁，也不知道请求是如何被处理的，他只是负责向职责链发出请求就可以了，实现了请求者和处理者之间的解耦。

**职责链模式有以下缺点**：

+ 产生很多细粒度职责对象
职责链模式把功能分散到单独的处理者对象中，也就是每个处理者只处理一部分的请求，在请求处理流程很复杂的情况下，这样会产生大量的细粒度职责对象。
+ 不一定能被处理
职责链模式的每个职责对象只负责处理自己职责范围内的请求，因此可能会出现某个请求，在整个职责链上传递完了，都没有职责对象处理它。
+ 影响执行效率
因为处理时以链的形式在对象间传递消息，根据实现方式不同，有可能会影响处理的速度。  

# 六、总结

职责链模式以分离职责为前提，将复杂的请求处理进行拆分，并分配给一个个的职责类。每个职责类只负责处理请求的一部分，在运行期间职责类进行动态组合，形成一个职责链，请求在链上进行传递和处理。

动态组合才是职责类模式的精华所在，因为要实现请求者和处理者的解耦，请求者不知道谁才是真正的处理者，因此要动态地把可能的处理者组合起来，由于组合的方式是动态的，这就意味着可以很方便地修改和添加新的处理者，从而让系统更加灵活和具有更好的扩展性。  