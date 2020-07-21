---
title: Java知识点系列之--对数组进行初始化填充
date: 2020-07-20 13:23:00
updated: 2020-07-20 13:23:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: Java中如何对数组进行初始化填充?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img23.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img23.jpg
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
# 对数组进行初始化填充
```Java
	import java.util.Arrays;
	 
	public class ArrayFilling {
	 
		public static void main(String[] args) {
	 
			int[] scoreArr = new int[8]; // 创建一个大小为8的数组
	 
			Arrays.fill(scoreArr, 0); // 将数组使用数字 0 进行填充
	 
			for (int i = 0; i < scoreArr.length; i++) {
				System.out.print(scoreArr[i] + " ");
			}
	                System.out.print("");
			Arrays.fill(scoreArr, 2, 6, 1);// 将索引从 2 到 6 使用数字 1 进行填充
			for (int i = 0; i < scoreArr.length; i++) {
				System.out.print(scoreArr[i] + " ");
			}
		}
	}
```
 打印结果：
```
0 0 0 0 0 0 0 0
0 0 1 1 1 1 0 0
```