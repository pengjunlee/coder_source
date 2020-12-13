---
title: SpringBoot基础之--返回的Json中忽略空字段
date: 2020-07-25 14:01:00
updated: 2020-07-25 14:01:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目中如何设置返回的Json中忽略空字段?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img1.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# 第一种：
```Java
	import com.fasterxml.jackson.annotation.JsonInclude;
	
	@JsonInclude(JsonInclude.Include.NON_NULL)
```

# 第二种：
```Properties
	spring:
	  jackson:
	    default-property-inclusion: non_null
```