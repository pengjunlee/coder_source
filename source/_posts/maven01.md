---
title: Maven系列之--Maven引入org.apache.tools.zip
date: 2020-07-20 14:01:00
updated: 2020-07-20 14:01:00
tags: Maven
categories: Maven
keywords: Java, Maven
type: 
description: Maven中如何引入org.apache.tools.zip依赖?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
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
可以看出 `org.apache.tools.zip` 是 `ant-**.jar` 里面的。

<div align=center>

![zip示意图](http://pengjunlee.3vzhuji.net/static/javacore/35.png "zip示意图")
<div align=left>

所以要引入`org.apache.tools.zip`，直接maven引入`ant`即可。
```Xml
	<!-- https://mvnrepository.com/artifact/org.apache.ant/ant -->
	<dependency>
		<groupId>org.apache.ant</groupId>
		<artifactId>ant</artifactId>
		<version>1.10.5</version>
	</dependency>
```