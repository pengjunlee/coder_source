---
title: Java知识点系列之--try语句块中的return语句
date: 2020-07-20 13:16:00
updated: 2020-07-20 13:16:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 一篇文章讨论一下当try语句块中含有return语句时的几个问题。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
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
对于做Java开发的程序员来说，异常捕获似乎已经是再稀松平常不过的事情了。我们都已经清楚的了解到：在try-catch-finally语法结构中无论异常是否发生，finally语句块中的内容都会被执行，所以，我们习惯把一些占用资源较多的对象的释放操作放到finally语句块中来做，例如：关闭数据库连接、关闭文件流。 

但当`try-catch-finally`语句块中包含`return`语句时，情况就会变得复杂，有以下几种情况值得注意。  

# 情况一：try中有return，catch和finally中没有return
```Java
	public class Test
	{
	    public static void main(String[] args)
	    {
	        System.out.println("返回结果 num = " + test());
	    }
	 
	    private static int test()
	    {
	        int num = 10;
	        try
	        {
	            System.out.println("进入try语句块");
	            System.out.println("try语句块中num = " + num);
	            return num;
	        }
	        catch (Exception e)
	        {
	            System.out.println("进入catch语句块");
	        }
	        finally
	        {
	 
	            num += 10;
	            System.out.println("进入finally语句块");
	            System.out.println("finally语句块中num = " + num);
	        }
	        return num;
	    }
	}
````
**执行结果**：  
```
	进入try语句块
	try语句块中num = 10
	进入finally语句块
	finally语句块中num = 20
	返回结果 num = 10
```
**结果分析**：

> 程序先进入try语句块，在执行return num;语句之前，发现程序后面还有finally语句块，于是，先将要返回的值num=10缓存起来，然后程序跳转到finally语句块执行，在finally语句块中num值被修改为了20，而最终返回的结果却是10，这说明finally语句块执行完毕后，程序直接将之前try语句块缓存的值返回了。  

# 情况二：try和finally中都有return
```Java
	public class Test
	{
	    public static void main(String[] args)
	    {
	        System.out.println("返回结果 num = " + test());
	    }
	 
	    private static int test()
	    {
	        int num = 10;
	        try
	        {
	            System.out.println("进入try语句块");
	            System.out.println("try语句块中num = " + num);
	            return num;
	        }
	        catch (Exception e)
	        {
	            System.out.println("进入catch语句块");
	        }
	        finally
	        {
	 
	            num += 10;
	            System.out.println("进入finally语句块");
	            System.out.println("finally语句块中num = " + num);
	            return num;
	        }
	 
	    }
	}
```
**执行结果**：  
```
	进入try语句块
	try语句块中num = 10
	进入finally语句块
	finally语句块中num = 20
	返回结果 num = 20
```
**结果分析**：

> try语句块的执行过程与情景一相同，所不同的是finally语句块中的return num;直接将finally语句块中的num值返回了。

>> <font color=blue>注</font>：SUN官方并不推荐在finally语句块中写return语句，否则，会提示警告：finally block does not complete normally。  

# 情况三：将情景一中的返回值类型改为引用类型
```Java
	public class Test
	{
	 
	    public static void main(String[] args)
	    {
	        System.out.println("返回结果Num.num = " + test().num);
	    }
	 
	    private static Num test()
	    {
	        Num num = new Num();
	        try
	        {
	            System.out.println("进入try语句块");
	            System.out.println("try语句块中Num.num = " + num.num);
	            return num;
	        }
	        catch (Exception e)
	        {
	            System.out.println("进入catch语句块");
	        }
	        finally
	        {
	 
	            num.num += 10;
	            System.out.println("进入finally语句块");
	            System.out.println("finally语句块中Num.num = " + num.num);
	 
	        }
	        return num;
	 
	    }
	 
	}
	 
	class Num
	{
	    public int num = 10;
	}
```
**执行结果**：  
```
	进入try语句块
	try语句块中Num.num = 10
	进入finally语句块
	finally语句块中Num.num = 20
	返回结果Num.num = 20
```
**结果分析**：

> try语句块的执行过程与情景一类似，只是此时缓存的是一个Num引用类型对象，finally语句块中的num.num += 10;修改了Num对象的num值，并作为最终的结果返回。  

# 结论
当try-catch-finally语句块中包含return语句时，有以下几点结论：

- 如果finally中有return语句，finally中的return语句会”覆盖“掉try中的return语句，直接将finally中的值作为结果返回。
- 如果finally中没有return语句，也没有改变要返回的结果值，则执行完finally中的语句后，会接着执行try中的return语句，返回try之前保留的结果值。
- 如果finally中没有return语句，但是改变了要返回的结果值，这里有点类似与引用传递和值传递的区别，分以下两种情况：
	1. 如果返回值是基本数据类型或字符串，try中的return语句依然会返回进入finally块之前保留的结果值。
	2. 如果返回值是引用数据类型，try中的return语句返回的就是在finally中改变后的该对象的值。 

# 参考文章

<http://blog.sina.com.cn/s/blog_6aefe42501018wmw.html>

<http://blog.csdn.net/ns_code/article/details/17485221>