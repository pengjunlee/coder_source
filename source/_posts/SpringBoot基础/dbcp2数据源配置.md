---
title: SpringBoot基础之--dbcp2数据源配置
date: 2020-07-25 14:09:00
updated: 2020-07-25 14:09:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何使用dbcp2数据源?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# DBCP2详细的配置表

## 常用链接配置
<div align=center>

![初始化器示意图](http://pengjunlee.3vzhuji.net/static/springboot/24.png "初始化器示意图")
<div align=left>

## 数据源连接数量配置
<div align=center>

![初始化器示意图](http://pengjunlee.3vzhuji.net/static/springboot/25.png "初始化器示意图")
<div align=left>

## 事务属性配置
<div align=center>

![初始化器示意图](http://pengjunlee.3vzhuji.net/static/springboot/26.png "初始化器示意图")
<div align=left>

## 数据源连接健康状况检查
<div align=center>

![初始化器示意图](http://pengjunlee.3vzhuji.net/static/springboot/27.png "初始化器示意图")
<div align=left>

## 缓存语句
<div align=center>

![初始化器示意图](http://pengjunlee.3vzhuji.net/static/springboot/28.png "初始化器示意图")
<div align=left>

## 连接泄露回收
<div align=center>

![初始化器示意图](http://pengjunlee.3vzhuji.net/static/springboot/29.png "初始化器示意图")
<div align=left>

> **注意**：

- Java数据库连接有“8小时问题”，所以`destroy-method="close"`一定要加上。“8小时问题”是指一个连接空闲8小时数据库会自动关闭，而数据源并不知道。  
- 高并发下，可以`testOnBorrow`设置`false`，`testWhileIdle`设置为`true`，这样就会定时对后台空链接进行检测发现无用连接就会清除掉，不会每次都去都去检测是否8小时的空链接。

参考： <http://blog.csdn.net/initphp/article/details/8255793>