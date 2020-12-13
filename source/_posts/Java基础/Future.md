---
title: Java知识点系列之--Future
date: 2020-07-20 13:04:00
updated: 2020-07-20 13:04:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 带你领略Java的Future特性。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
> 参考书籍：《Java特种兵（上册）》 

接口 `Future<V>` 可用于获取异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。计算完成后只能使用 `get()` 方法来获取结果，如有必要，计算完成前可以阻塞此方法。取消则由 `cancel()` 方法来执行。还提供了其他方法，以确定任务是正常完成还是被取消了。一旦计算完成，就不能再取消计算。如果为了可取消性而使用 `Future` 但又不提供可用的结果，则可以声明 `Future<?>` 形式类型，并返回 `null` 作为底层任务的结果。 
```Java
	package java.util.concurrent;
	 
	import java.util.concurrent.ExecutionException;
	import java.util.concurrent.TimeUnit;
	import java.util.concurrent.TimeoutException;
	 
	public interface Future<V> {
	 
		/*
		 * 尝试将任务的状态修改为CANCELLED
		 * 该动作若将状态成功修改为CANCELLED，则会释放AQS的锁，成功返回true，失败返回false
		 * 传入的boolean参数如果为true，则判定如果有正在运行的线程 runner，则会将其中断
		 */
		boolean cancel(boolean mayInterruptIfRunning);
	 
		/*
		 * 判定任务的状态是否为CANCELLED
		 */
		boolean isCancelled();
	 
		/*
		 * 判定任务是否结束
		 * 该方法不会阻塞，若返回true，则表示任务已经结束
		 */
		boolean isDone();
	 
		/*
		 * 尝试获取任务结束的返回值
		 * 如果是Runnable 则通常返回null，如果是Callable 则返回对应的return值
		 * 若任务尚未结束，则当前线程将会等待直到任务结束才返回
		 */
		V get() throws InterruptedException, ExecutionException;
	 
		/*
		 * 尝试获取任务结束的返回值
		 * 达到设置的超时时间后，则会自动放弃等待，并抛出TimeoutException异常
		 */
		V get(long timeout, TimeUnit unit) throws InterruptedException,
				ExecutionException, TimeoutException;
	}
```
`Future` 接口有很多实现类，用得较多的是`FutureTask`，用于实现可取消的异步计算功能：利用开始和取消计算的方法、查询计算是否完成的方法和获取计算结果的方法。此类提供了对 `Future` 的基本实现，仅在计算完成时才能获取结果，如果计算尚未完成，则阻塞 `get` 方法。一旦计算完成，就不能再重新开始或取消计算。 

下面是一个使用`FutureTask`进行异步计算的简单样例。
```Java
	package com.pengjunlee;
	 
	import java.util.concurrent.Callable;
	import java.util.concurrent.ExecutionException;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	 
	public class FutureTaskTest {
	 
		static class CallableImpl implements Callable<Integer> {
	 
			private int[] addends;
	 
			public CallableImpl(int[] addends) {
				this.addends = addends;
			}
	 
			public Integer call() throws Exception {
				int sum = 0;
				// 简单的数学计算，求出addends数组内的数字总和
				for (int i = 0; i < addends.length; i++) {
					sum += addends[i];
				}
				return sum;
			}
		}
	 
		public static void main(String[] args) throws InterruptedException,
				ExecutionException {
			ExecutorService executorService = Executors.newFixedThreadPool(3);
			// 以数学计算获取(1+2+3)*(4+5+6)*(7+8+9+10)的结果为例
			int sum1 = executorService.submit(
					new CallableImpl(new int[] { 1, 2, 3 })).get();
			int sum2 = executorService.submit(
					new CallableImpl(new int[] { 4, 5, 6 })).get();
			int sum3 = executorService.submit(
					new CallableImpl(new int[] { 7, 8, 9, 10 })).get();
			int result = sum1 * sum2 * sum3;
			System.out.println("(1+2+3)*(4+5+6)*(7+8+9+10)= " + result);
	 
		}
	}
```
运算结果： 
```
(1+2+3)*(4+5+6)*(7+8+9+10)= 3060
```
`FutureTask`内部有一个`sync`属性，这个属性所在的类是`AQS（AbstractQueuedSynchronizer）`的子类，即它是基于`AQS`来实现的。