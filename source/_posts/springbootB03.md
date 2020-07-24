---
title: SpringBoot基础之--查询mongodb只返回指定字段
date: 2020-07-25 14:03:00
updated: 2020-07-25 14:03:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目中查询mongodb只返回指定字段如何设置?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
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
> 转载自：<https://my.oschina.net/110NotFound/blog/3005716>

springboot使用mongodb查询的时候会遇到服务器出口带宽压力大的情况，原因可能是查询mongodb的时候把整个对象给拖下来了，事实上我们只需要其中的某些字段，多余的字段返回的话会给小水管的带宽加上压力，也就是说我们的mongodb查询时只需要返回某些字段。

# 方法一

直接使用mongoTemplate来查询，只返回主键，name，status字段。（主键是默认返回的）
```Java
	Query query = new Query();
	query.addCriteria(Criteria.where("status").is(3));
	query.skip(skipNumber);
	query.limit(pageSize);
	query.fields().include("name").include("status");
	return mongoTemplate.find(query, CompanyInformation.class);
```

# 方法二

网上流传比较多的另外一种方法：
```Java
	BasicDBObject dbObject = new BasicDBObject();
	dbObject.put("id", "123");
	dbObject.put("name", "haha");
	//指定返回的字段
	BasicDBObject fieldsObject = new BasicDBObject();
	fieldsObject.put("id", true);
	fieldsObject.put("name", true);
	fieldsObject.put("age", true);
	Query query = new BasicQuery(dbObject.toJson(), fieldsObject.toJson());
	List<JavaEntity> list= mongoTemplate.find(query, JavaEntity.class, "collectionName");
```
