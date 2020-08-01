---
title: MyBatis系列之--缓存
date: 2020-07-25 14:06:00
updated: 2020-07-25 14:06:00
tags: MyBatis
categories: MyBatis
keywords: MyBatis
type: 
description: MyBatis如何开启并使用缓存？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img6.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img6.jpg
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
# 缓存简介
MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制缓存，以改善数据库的性能并提升查询效率。 

# 一级缓存、二级缓存
MyBatis 中定义了两级缓存：一级缓存和二级缓存。

- 一级缓存也称为本地缓存或局部缓存，是一个SqlSession级别的缓存。在与数据库的一次会话期间执行查询得到的数据都会被存储到当前会话的一级缓存（本质是一个Map）中，以后如果再次执行相同的查询，就会直接从这块缓存中去获取数据，而不必再去查询数据库了；
- 二级缓存也叫作全局缓存，是一个namespace级别或Mapper级别的缓存，即一个namespace或Mapper（通常一个Mapper会定义一个namespace）会对应一个二级缓存。当会话关闭，会话的一级缓存中存储的数据会被保存到该会话所对应的namespace的二级缓存（本质是一个Map）中，新的会话如果再次执行相同的查询，就可以直接从二级缓存中去获取；

# 开启缓存
默认情况下MyBatis 只为我们开启了一级缓存，若要开启二级缓存,需要以下三个步骤：

1.修改MyBatis的核心配置文件，在<settings/>元素中增加cacheEnabled配置。 
```Xml
	<settings>
		<!-- 开启二级缓存 -->
		<setting name="cacheEnabled" value="true" />
	</settings>
```

2.为每一个需要使用二级缓存的mapper.xml文件增加cache配置。  
```Xml
	<cache 
		eviction="LRU"                      /*缓存回收策略，LRU、FIFO、SOFT、WEAK*/
		flushInterval="60000"               /*刷新间隔，单位毫秒*/
		readOnly="true"                     /*是否只读、true、false*/
		size="1024"                         /*缓存对象引用数量*/
		type="org.mybatis.MyCustomCache">   /*缓存实现类的全类名*/
	</cache>
```
其中，`eviction`属性用来指定缓存的回收策略，可用的有以下几个回收策略： 

- LRU – 最近最少使用的:移除最长时间不被使用的对象，默认值。
- FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
- SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
- WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

<br/>

- `flushInterval`用来指定缓存的数据多长时间清空一次（以毫秒为单位），它可以被设置为任意的正数，默认情况是不设置,也就是不需要定时清空缓存,只在执行增删改操作的时候才清空缓存。
- `size`用来指定最多可以缓存的对象引用数量，一旦需要缓存的对象引用数量超过该值，MyBatis将会根据eviction中指定的缓存回收策略对缓存的数据进行回收。它可以被设置为任意正整数，默认值是 1024。
- `readOnly`用来指定从缓存中获取的数据是否只用于只读场景，一旦该值被设置为 true，从缓存中获取数据时，MyBatis会认为你只会对查询出来的内容进行只读操作而不会修改里面的内容，于是会将缓存中的数据对象的引用直接返回，这种情况下一旦有人对这块数据内容进行了修改很容易引发数据冲突和安全问题。当该值被设置为false时，MyBatis会返回缓存对象的拷贝(通过序列化)，这会慢一些,但是安全,默认是 false。
- `type`用来指定所使用的缓存实现类，它可以是任何一个实现了`org.mybatis.cache.Cache`接口的Java类。

3.当cache配置中的`readOnly`属性设置为false时，POJO需实现`java.io.Serializable`接口

<font color=red>注意：</font>

- 要使用二级缓存，所有的会话对象SqlSession必须使用同一个SqlSessionFactory对象的openSession()方法来创建。
- cacheEnabled配置仅对二级缓存有效，对一级缓存无影响。
- 除了使用<cache/>元素对二级缓存进行配置，还可以使用<cache-ref/>元素来引用其他namespace的<cache/>配置。
- 每一个select元素都有一个useCache="true"的默认属性，代表该查询SQL会使用二级缓存，将该属性值改为false，将不再使用二级缓存，一级缓存依然使用。
- 每一个增删改元素都有一个flushCache="true"的默认属性，代表执行该操作会将一级缓存和二级缓存同时清空。
- SqlSession.clearCache()方法只会清空当前SqlSession的一级缓存，对二级缓存无影响。
- 在MyBatis3.3版本以后，新增加了一个localCacheScope的全局配置项，用来指定本地缓存的作用域，有SESSION和STATEMENT两个取值。 

# 整合Ehcache

除了使用MyBatis自定义缓存的方式, 你也可以通过实现`org.mybatis.cache.Cache`接口来定义属于你自己的缓存方式或者直接引入其他第三方的缓存方案，接下来以MyBatis整合Ehcache为例来对如何在MyBatis中引入第三方的缓存方案做一个简单介绍。

在进行整合之前，需先下载好相关的Jar包，下载地址：<https://github.com/mybatis/ehcache-cache/releases>

## 导入Jar包
<div align=center>

![MyBatis图](http://pengjunlee.3vzhuji.net/static/mybatis/02.png "MyBatis示意图")
<div align=left>
将下载好的压缩包解压之后，将所需的上述3个Jar包添加到项目的classpath构建路径。

## 开启二级缓存
首先确保在MyBatis的核心配置文件中已经开启了二级缓存，并在Mapper文件中增加如下配置： 
```XMl
	<cache type="org.mybatis.caches.ehcache.EhcacheCache">
	</cache>
```
## 在classpath下添加ehcache.xml
在classpath下添加ehcache.xml文件，其内容如下： 
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
	 <!-- 磁盘保存路径 -->
	 <diskStore path="D:\cache\ehcache" />
	 
	 <defaultCache 
	   maxElementsInMemory="1" 
	   maxElementsOnDisk="10000000"
	   eternal="false" 
	   overflowToDisk="true" 
	   timeToIdleSeconds="120"
	   timeToLiveSeconds="120" 
	   diskExpiryThreadIntervalSeconds="120"
	   memoryStoreEvictionPolicy="LRU">
	 </defaultCache>
	</ehcache>
 
	<!-- 
	属性说明：
	diskStore：指定数据在磁盘中的存储位置。
	defaultCache：当借助CacheManager.add("demoCache")创建Cache时，EhCache便会采用<defalutCache/>指定的的管理策略
	 
	以下属性是必须的：
	maxElementsInMemory - 在内存中缓存的element的最大数目 
	maxElementsOnDisk - 在磁盘上缓存的element的最大数目，若是0表示无穷大
	eternal - 设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断
	overflowToDisk - 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上
	 
	以下属性是可选的：
	timeToIdleSeconds - 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时，这些数据便会删除，默认值是0,也就是可闲置时间无穷大
	timeToLiveSeconds - 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大
	 diskSpoolBufferSizeMB 这个参数设置DiskStore(磁盘缓存)的缓存区大小.默认是30MB.每个Cache都应该有自己的一个缓冲区.
	diskPersistent - 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。
	diskExpiryThreadIntervalSeconds - 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，相应的线程会进行一次EhCache中数据的清理工作
	memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）
	 -->
```