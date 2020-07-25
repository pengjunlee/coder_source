---
title: SpringBoot基础之--设置跨域请求（下）
date: 2020-07-25 14:18:00
updated: 2020-07-25 14:18:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何设置支持跨域请求？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img18.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img18.jpg
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
**跨域**：现代浏览器出全的考虑，在http/https请求时必须遵守同源策略，否则即使跨域的http/https 请求，默认情况下是被禁止的，ip(域名)不同、或者端口不同、协议不同(比如http、https) 都会造成跨域问题。

# 一、前端解决方案
使用 JSONP 来支持跨域的请求，JSONP 实现跨域请求的原理简单的说，就是动态创建 script 标签，然后利用 script 的 SRC 不受同源策略约束来跨域获取数据。缺点是需要后端配合输出特定的返回信息。
利用反应代理的机制来解决跨域的问题，前端请求的时候先将请求发送到同源地址的后端，通过后端请求转发来避免跨域的访问。
后来 HTML5 支持了 `CORS` 协议。CORS 是一个 W3C 标准，全称是`”跨域资源共享”`（`Cross-origin resource sharing`），允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 AJAX 只能同源使用的限制。它通过服务器增加一个特殊的 Header[Access-Control-Allow-Origin]来告诉客户端跨域的限制，如果浏览器支持 CORS、并且判断 Origin 通过的话，就会允许 XMLHttpRequest 发起跨域请求。

前端使用了 CORS 协议，就需要后端设置支持非同源的请求，对于SpringBoot 对于CORS 同样有着良好的支持，首先附上官网地址：

<https://docs.spring.io/spring-boot/docs/2.0.2.RELEASE/reference/htmlsingle/#boot-features-cors>

# 二、java跨域的三种配置方式

## 方式一：配置过滤器（全局配置）
```Java
	@Configuration
	public class GlobalCorsConfig {
	    @Bean
	    public CorsFilter corsFilter() {
	        CorsConfiguration config = new CorsConfiguration();
	          config.addAllowedOrigin("*");
	          config.setAllowCredentials(true);
	          config.addAllowedMethod("*");
	          config.addAllowedHeader("*");
	          config.addExposedHeader("*");
	 
	        UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
	        configSource.registerCorsConfiguration("/**", config);
	 
	        return new CorsFilter(configSource);
	    }
	}
```

## 方式二：配置拦截器 （全局配置）
```Java
	@Configuration
	public class MyConfiguration extends WebMvcConfigurerAdapter  {
	 
	    @Override  
	    public void addCorsMappings(CorsRegistry registry) {  
	        registry.addMapping("/**")  
	                .allowCredentials(true)  
	                .allowedHeaders("*")  
	                .allowedOrigins("*")  
	                .allowedMethods("*");  
	 
	    }  
	}
```

## 方式三： 单个请求的跨域通过 @CrossOrigin 注解来实现
```Java
	@RequestMapping("/hello")
	@CrossOrigin("http://localhost:8080") 
	public String hello( ){
	    return "Hello World";
	}
```
参考资料：<https://blog.csdn.net/fxbin123/article/details/80603678>