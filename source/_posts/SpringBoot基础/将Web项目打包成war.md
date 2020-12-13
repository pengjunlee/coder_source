---
title: SpringBoot基础之--将Web项目打包成war
date: 2020-07-25 14:14:00
updated: 2020-07-25 14:14:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: 如何将SpringBoot的Web项目打包成war？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img14.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img14.jpg
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
> 转载自：<http://m.blog.csdn.net/article/details?id=52515226>

把spring-boot项目按照平常的web项目一样打成war包发布到tomcat容器下。

# 修改打包形式

在pom.xml里设置：`<packaging>war</packaging>`

# 移除嵌入式tomcat插件

在pom.xml里找到spring-boot-starter-web依赖节点，在其中添加如下代码：
```Xml
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-web</artifactId>
	    <!-- 移除嵌入式tomcat插件 -->
	    <exclusions>
	        <exclusion>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-tomcat</artifactId>
	        </exclusion>
	    </exclusions>
	</dependency>
```

# 添加servlet-api的依赖

下面两种方式都可以，任选其一：
```Xml
	<dependency>
	    <groupId>javax.servlet</groupId>
	    <artifactId>javax.servlet-api</artifactId>
	    <version>3.1.0</version>
	    <scope>provided</scope>
	</dependency>
```
或者
```Xml
	<dependency>
	    <groupId>org.apache.tomcat</groupId>
	    <artifactId>tomcat-servlet-api</artifactId>
	    <version>8.0.36</version>
	    <scope>provided</scope>
	</dependency>
```

# 修改启动类，并重写初始化方法
我们平常用main方法启动的方式，都有一个App的启动类，代码如下：
```Java
	@SpringBootApplication
	public class Application {
	    public static void main(String[] args) {
	        SpringApplication.run(Application.class, args);
	    }
	}
```
我们需要类似于`web.xml`的配置方式来启动spring上下文了，在Application类的同级添加一个`SpringBootStartApplication`类，其代码如下：
```Java
	/**
	 * 修改启动类，继承 SpringBootServletInitializer 并重写 configure 方法
	 */
	public class SpringBootStartApplication extends SpringBootServletInitializer {
	 
	    @Override
	    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
	        // 注意这里要指向原先用main方法执行的Application启动类
	        return builder.sources(Application.class);
	    }
	}
```

# 打包部署
在项目根目录下（即包含pom.xml的目录），在命令行里输入 `mvn clean package -Dmaven.test.skip=true` 即可， 等待打包完成，出现`[INFO] BUILD SUCCESS` 即为打包成功。 然后，把target目录下的war包放到tomcat的webapps目录下，启动tomcat，即可自动解压部署。 

最后在浏览器中输入：`http://localhost:[端口号]/[打包项目名]/`  
发布成功！