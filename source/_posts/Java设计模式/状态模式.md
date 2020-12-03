---
title: 设计模式系列之--状态模式
date: 2020-07-18 12:20:00
updated: 2020-07-18 12:20:00
tags: Java设计模式
categories: 设计模式
keywords: Java, 设计模式, 状态模式
type: 
description: 什么是状态模式？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img20.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img20.jpg
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
# 一、什么是状态模式

状态(State)）模式，又称状态对象(Pattern of Objects for States)模式，是一种对象的行为模式。状态模式允许一个对象在其内部状态改变的时候改变其行为。这个对象看上去就像是改变了它的类一样。

`状态模式的本质：根据状态来分离和选择行为`

# 二、状态模式的结构

<div align=center>

![状态模式应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/30.png "状态模式应用示意图")
<div align=left>

状态模式涉及的角色及其职责如下：

+ 环境(Context)角色:也称上下文,通常用来定义客户端感兴趣的接口，同时维护一个来具体处理当前状态的对象示例。
+ 抽象状态(State)角色：定义一个接口，用来封装与环境（Context）对象的一个特定的状态所对应的行为。
+ 具体状态(ConcreteState)角色：每一个具体状态类都实现了一个跟环境（Context）相关的状态的具体处理。

状态模式结构示意源代码如下：
```Java
	/**
	 * 抽象状态(State)角色，用来封装与环境（Context）对象的一个特定的状态所对应的行为
	 */
	public interface State {
	 
		/**
		 * 状态对应的处理
		 * 
		 * @param sampleParameter 示例参数
		 */
		public void handle(String sampleParameter);
	}
```
<br/>
```Java
	/**
	 * 具体状态(ConcreteState)角色，实现一个与Context的一个特定状态相关的行为
	 */
	public class ConcreteStateA implements State {
	 
		@Override
		public void handle(String sampleParameter) {
			// 实现具体的处理
	 
		}
	 
	}
```
<br/>
```Java
	/**
	 * 具体状态(ConcreteState)角色，实现一个与Context的一个特定状态相关的行为
	 */
	public class ConcreteStateB implements State {
	 
		@Override
		public void handle(String sampleParameter) {
			// 实现具体的处理
	 
		}
	 
	}
```
<br/>
```Java
	/**
	 * 环境(Context)角色
	 */
	public class Context {
	 
		// 持有一个State类型的对象实例
		private State state;
	 
		public void setState(State state) {
			this.state = state;
		}
	 
		public void request(String sampleParameter) {
			// 处理操作，会转调State来处理
			state.handle(sampleParameter);
		}
	}
```

# 三、状态模式应用举例

《研磨设计模式》中给出这么一个场景：实现在线投票。

考虑一个在线投票的应用，要实现控制同一个用户只能投一票，如果一个用户重复投票，而且投票次数超过5次，则判定为恶意刷票，要取消该用户投票的资格，当然同时也要取消他所投的票；如果一个用户的投票次数超过8次，将进入黑名单，禁止再登录和使用系统。

以上功能我们使用状态模式来实现：

首先需要把投票过程的各种状态定义出来，根据以上描述大致分为四种状态：正常投票、重复投票、恶意刷票、进入黑名单。

设计好的程序结构如图所示：

<div align=center>

![状态模式应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/31.png "状态模式应用示意图")
<div align=left>

先来看看状态接口的代码实现，示例代码如下。 
```Java
	public interface VoteState {
		/**
		 * 处理状态对应的行为
		 * 
		 * @param user
		 *            投票人
		 * @param voteItem
		 *            投票项
		 * @param voteManager
		 *            投票上下文，用来在实现状态对应的功能处理的时候， 可以回调上下文的数据
		 */
		public void vote(String user, String voteItem, VoteManager voteManager);
	}
```
接下来看看各个状态对应的处理，示例代码如下。 
```Java
	/**
	 * 正常投票状态
	 */
	public class NormalVoteState implements VoteState {
	 
		@Override
		public void vote(String user, String voteItem, VoteManager voteManager) {
			// 正常投票，记录到投票记录中
			voteManager.getMapVote().put(user, voteItem);
			System.out.println("恭喜您，投票成功。");
		}
	 
	}
```
<br/>
```Java
	/**
	 * 重复投票状态
	 */
	public class RepeatVoteState implements VoteState {
		@Override
		public void vote(String user, String voteItem, VoteManager voteManager) {
			// 重复投票，此处仅打印一个警告作为示意
			System.out.println("警告：请不要重复投票!");
		}
	}
```
<br/>
```Java
	/**
	 * 恶意刷票状态
	 */
	public class SpiteVoteState implements VoteState {
		@Override
		public void vote(String user, String voteItem, VoteManager voteManager) {
			// 恶意投票，取消用户的投票资格，并取消投票记录
			String str = voteManager.getMapVote().get(user);
			if (str != null) {
				voteManager.getMapVote().remove(user);
			}
			System.out.println("你有恶意刷屏行为，取消投票资格！");
		}
	}
```
<br/>
```Java
	/**
	 * 黑名单状态
	 */
	public class BlackVoteState implements VoteState {
		@Override
		public void vote(String user, String voteItem, VoteManager voteManager) {
			// 记录在黑名单中，禁止登录系统
			System.out.println("提示：您已进入投票黑名单，禁止登录和使用本系统！");
		}
	 
	}
```
最后看看投票管理，相当于状态模式中的上下文，示例代码如下。 
```Java
	import java.util.HashMap;
	import java.util.Map;
	 
	public class VoteManager {
		// 持有状体处理对象实例
		private VoteState state = null;
		// 记录用户投票的结果，Map<String,String>对应Map<用户名称，投票的选项>
		private Map<String, String> mapVote = new HashMap<String, String>();
		// 记录用户投票次数，Map<String,Integer>对应Map<用户名称，投票的次数>
		private Map<String, Integer> mapVoteCount = new HashMap<String, Integer>();
	 
		/**
		 * 获取用户投票结果的Map
		 */
		public Map<String, String> getMapVote() {
			return mapVote;
		}
	 
		/**
		 * 投票
		 * 
		 * @param user
		 *            投票人
		 * @param voteItem
		 *            投票的选项
		 */
		public void vote(String user, String voteItem) {
			// 1.为该用户增加投票次数
			// 从记录中取出该用户已有的投票次数
			Integer oldVoteCount = mapVoteCount.get(user);
			if (oldVoteCount == null) {
				oldVoteCount = 0;
			}
			oldVoteCount += 1;
			mapVoteCount.put(user, oldVoteCount);
			// 2.判断该用户的投票类型，就相当于判断对应的状态
			// 到底是正常投票、重复投票、恶意投票还是上黑名单的状态
			if (oldVoteCount == 1) {
				state = new NormalVoteState();
			} else if (oldVoteCount > 1 && oldVoteCount < 5) {
				state = new RepeatVoteState();
			} else if (oldVoteCount >= 5 && oldVoteCount < 8) {
				state = new SpiteVoteState();
			} else if (oldVoteCount > 8) {
				state = new BlackVoteState();
			}
			// 3.然后转调状态对象来进行相应的操作
			state.vote(user, voteItem, this);
		}
	}
```
写个客户端来测试一下，示例代码如下。 
```Java
	public class Client {
	 
		public static void main(String[] args) {
			// 创建一个投票管理对象实例
			VoteManager vm = new VoteManager();
	 
			// 通过一个循环操作来模仿用户连续投票9次的行为
			for (int i = 0; i < 9; i++) {
				vm.vote("u1", "A");
			}
		}
	 
	}
```
运行程序打印结果如下： 
```
恭喜您，投票成功。
警告：请不要重复投票!
警告：请不要重复投票!
警告：请不要重复投票! 
你有恶意刷屏行为，取消投票资格！
你有恶意刷屏行为，取消投票资格！
你有恶意刷屏行为，取消投票资格！
你有恶意刷屏行为，取消投票资格！
提示：您已进入投票黑名单，禁止登录和使用本系统！
```

# 四、理解状态模式

## 状态和行为

所谓对象的状态，通常指的就是对象实例的属性的值；而行为指的就是对象的功能，再具体点说，行为大多可以对应到方法上。

状态模式的功能就是分离状态的行为，通过维护状态的变化，来调用不同状态对应的不同功能。也就是说，状态和行为是相关联的，它们的关系可以描述为：状态决定行为。

由于状态是在运行期间被改变的，因此行为也会在运行期间根据状态的改变而改变，看起来，同一个对象，在不同的运行时刻，行为是不一样的，就像是类被修改了一样。

## 行为的平行性

注意平行性而不是平等性。所谓平行性指的是各个状态的行为所处的层次是一样的，相互独立的、没有关联的，是根据不同的状态来决定到底走平行线的哪一条。行为是不同的，当然对应的实现也是不同的，相互之间是不可替换的，如下图所示。 

<div align=center>

![状态模式应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/32.png "状态模式应用示意图")
<div align=left>

而平等性强调的是可替换性，大家是同一行为的不同描述或实现，因此在同一个行为发生的时候，可以根据条件挑选任意一个实现来进行相应的处理，如下图所示。

<div align=center>

![状态模式应用示意图](http://pengjunlee.3vzhuji.net/static/design_pattern/33.png "状态模式应用示意图")
<div align=left>

大家可能会发现状态模式的结构和策略模式的结构完全一样，但是，它们的目的、实现、本质却是完全不一样的。还有行为之间的特性也是状态模式和策略模式一个很重要的区别，状态模式的行为是平行性的，不可相互替换的；而策略模式的行为是平等性的，是可以相互替换的。

## 环境和状态处理对象

在状态模式中，环境(Context)是持有状态的对象，但是环境自身并不处理跟特定的状态相关的行为，而是把处理状态的功能委托给了状态对应的状态处理类来处理。

在具体的状态处理类中经常需要获取环境自身的数据，甚至在必要的时候会回调环境的方法，因此，通常将环境自身当作一个参数传递给具体的状态处理类。

客户端一般只和环境交互。客户端可以用状态对象来配置一个环境，一旦配置完毕，就不再需要和状态对象打交道了。客户端通常不负责运行期间状态的维护，也不负责决定后续到底使用哪一个具体的状态处理对象。 

# 五、状态模式的适用性

在以下条件下可以考虑使用状态模式：

+ 一个对象的行为取决于它的状态,并且它必须在运行时刻根据状态改变它的行为。
+ 一个操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态。 

# 六、状态模式的优缺点

**使用状态模式的优点**：

+ 简化应用逻辑控制  
状态模式使用单独的类来封装一个状态的处理，可以把负责逻辑控制的代码分散到单独的状态类中去，这样就把着眼点从执行状态提高到整个对象的状态，使得代码结构化和意图更清晰，从而简化应用的逻辑控制。
+ 更好地分离状态和行为  
状态模式通过设置所有状态类的公共接口，把状态和状态对应的行为分离开，把所有与一个特定的状态相关的行为都放入一个对象中，使得应用程序在控制的时候，只需要关心状态的切换，而不用关心这个状态对应的真正处理。
+ 更好的扩展性  
引入了状态处理的公共接口后，使得扩展新的状态变得非常容易，只需要新增加一个实现状态处理的公共接口的实现类，然后在进行状态维护的地方，设置状态变化到这个新的状态即可。
+ 显式化进行状态转换  
状态模式为不同的状态引入独立的对象，使得状态的转换变得更加明确。而且状态对象可以保证上下文不会发生内部状态不一致的情况，因为上下文中只有一个变量来记录状态对象，只要为这一个变量赋值就可以了。

**使用状态模式的缺点**：
一个状态对应一个状态处理类，会使得程序引入太多的状态类，这样程序变得杂乱。 