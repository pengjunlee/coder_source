---
title: SpringBoot基础之--使用spring-boot-maven-plugin插件打包应用
date: 2020-07-25 14:25:00
updated: 2020-07-25 14:25:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何使用spring-boot-maven-plugin插件打包应用？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img25.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img25.jpg
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
官方文档：<https://docs.spring.io/spring-boot/docs/current/maven-plugin/index.html>

`spring-boot-maven-plugin` 插件以Maven的方式为Springboot应用提供支持，能够将Springboot应用打包为可执行的jar或war文件，进行相应部署后即可启动Springboot应用。

`spring-boot-maven-plugin` 的构建目标：

- spring-boot:run 运行你的Springboot应用
- spring-boot:repackage 将mvn package 生成的 jar或者war 重新打包成可执行文件，同时修改原文件名，增加.origin 后缀
- spring-boot:start 与 spring-boot:stop 用来管理Springboot应用的生命周期（例如，mvn integration-test 集成测试阶段）
- spring-boot:build-info 生成构建信息build-info.properties 可供Actuator 使用

# 指定打包类型
在`pom.xml`文件中指定打包类型，指定生成的是jar还是war。 
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	 
		<!-- ... -->
		<packaging>jar</packaging>
		<!-- ... -->
	 
	</project>
```

# 插件设置
使用`spring-boot-maven-plugin`来对Springboot应用进行打包，需要在项目的 pom.xml 文件中引入插件。 

对于使用了 spring-boot-starter-parent 的项目， 只需在properties中指定start-class启动类即可。 
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.3.RELEASE</version>
		<relativePath />
	</parent>
 
	<properties>
		<start-class>com.bootdo.MyApplication</start-class>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.8</java.version>
	</properties>
```
否则，需要使用如下配置指定启动类和打包类型。 
```Xml
	<build>
		<plugins>
			<!-- ... -->
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<version>2.1.5.RELEASE</version>
				<configuration>
					<mainClass>${start-class}</mainClass>
					<layout>ZIP</layout>
				</configuration>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<!-- ... -->
		</plugins>
	</build>
```
layout 属性用来指定打成 jar 还是 war 文件，可用的值包括：ZIP 、JAR 、WAR、 NONE 。 

# 执行打包 
使用 `mvn package spring-boot:repackage` 来执行打包。
```
	mvn package spring-boot:repackage
```