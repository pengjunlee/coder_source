---
title: SpringBoot基础之--Spring IO Platform
date: 2020-07-25 14:02:00
updated: 2020-07-25 14:02:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目中如何使用Spring IO Platform解决依赖版本冲突?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
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
# 版本冲突现状

在使用Spring的时候，经常会使用到第三方库，一般大家都是根据经验挑选一个版本号或挑选最新的，随意性较大，其实这是有问题的，除非做过完整的测试，保证集成该版本的依赖不会出现问题，且后续集成其它第三方库的时候也不会出现问题，否则风险较大，且后续扩展会越来越困难，因为随着业务复杂度的增加，集成的第三方组件会越来会多，依赖之间的关联也会也来越复杂。

好消息是，`Spring IO Platform` 能很好地解决这些问题，我们在添加第三方依赖的时候，不需要写版本号，它能够自动帮我们挑选一个最优的版本，保证最大限度的扩展，而且该版本的依赖是经过测试的，可以完美的与其它组件结合使用。

# 什么是Spring IO Platform

`Spring IO platform` 包括 `Foundation Layer modules` 和`Execution Layer domain-specific runtimes(DSR)`。

- Foundation Layer modules：代表了核心Spring模块和相关的第三方依赖关系，这些模块和依赖关系已经得到协调，以确保开发过程的顺利进行。
- Execution Layer： 提供的DSR极大地简化了构建基于JVM的生产准备工作负载。
简而言之，Spring IO Platform，简单的可以认为是一个依赖维护平台，该平台将相关依赖汇聚到一起，针对每个依赖，都提供了一个版本号；这些版本对应的依赖都是经过测试的，可以保证一起正常使用。

# Spring IO Platform中维护了哪些依赖

完整的依赖列表：<http://docs.spring.io/platform/docs/current/reference/html/appendix-dependency-versions.html>

# Spring 相关BOM

当然SpringSource为了解决这些Jar冲突，推出了各种BOM，当然最著名的就是`spring platform io bom`，其中最核心的三个是：`spring-framework-bom`、`spring-boot-dependencies`、`platform-bom`。

对于Spring工程来说，直接在pom.xml文件中添加如下配置代码，即可免去管理版本冲突的难题。
```Xml
	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-framework-bom</artifactId>
	            <version>4.2.0.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-dependencies</artifactId>
	            <version>1.3.0.M2</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	        <dependency>
	            <groupId>io.spring.platform</groupId>
	            <artifactId>platform-bom</artifactId>
	            <version>1.1.3.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
```

# 在Maven中使用Spring IO Platform

有两种方式，一种是`使用import导入`，另一种是`继承parent`。

## import导入
将Platform的bom导入到应用程序的pom。
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <groupId>com.example</groupId>
	    <artifactId>your-application</artifactId>
	    <version>1.0.0-SNAPSHOT</version>
	    <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>io.spring.platform</groupId>
	                <artifactId>platform-bom</artifactId>
	                <version>Brussels-SR6</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
	    </dependencyManagement>
	    <!-- Dependency declarations -->
	</project>
```

## 继承parent
将它用作pom的父项，而不是导入平台的bom。
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <groupId>com.example</groupId>
	    <artifactId>your-application</artifactId>
	    <version>1.0.0-SNAPSHOT</version>
	    <parent>
	        <groupId>io.spring.platform</groupId>
	        <artifactId>platform-bom</artifactId>
	        <version>Brussels-SR6</version>
	        <relativePath/>
	    </parent>
	    <!-- Dependency declarations -->
	</project>
```
使用继承的话，除了从父pom中引入Spring IO Platform之外，我们的应用还会引入一些插件管理的配置，如Spring Boot的Maven插件，我们可以利用这一点，然后只需要在<plugins>代码块中添加如下代码即可使用插件：
```Xml
	<build> 
	    <plugins> 
	        <plugin> 
	            <groupId>org.springframework.boot</groupId> 
	            <artifactId>spring-boot-maven-plugin</artifactId> 
	        </plugin> 
	    </plugins> 
	</build>
```
另外，使用继承的话，还可以重写Maven中的属性，你可以用你<properties>想要的值在你的pom的部分声明属性。

如下所示--覆盖父类提供的依赖版本号：
```Xml
	<properties> 
	    <foo.version> 1.1.0.RELEASE </foo.version> 
	</ properties>
```
如果你想一起使用Platform和Spring Boot，你不用必须使用Platform的pom作为父级（参考：<http://blog.csdn.net/fly910905/article/details/79420326>）。

相反，您可以按照上面所述导入平台的pom（import方式），然后手动执行其余配置。Spring Boot的Maven使用它的文档将告诉你如何。

无论您选择哪种方法，都不会在您的应用程序中添加任何依赖项。但是，如果确实声明了对平台中某些内容的依赖关系，则现在可以省略版本号。例如：
```Xml
	<dependencies> 
	    <dependency> 
	        <groupId>org.springframework</groupId> 
	        <artifactId>spring-core</artifactId> 
	    </dependency> 
	</dependencies>
```

# Spring IO Platform应用示例：
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <groupId>com.example</groupId>
	    <artifactId>helloworld</artifactId>
	    <version>0.0.1-SNAPSHOT</version>
	   <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>io.spring.platform</groupId>
	                <artifactId>platform-bom</artifactId>
	                <version>Athens-SR2</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	            
	            <dependency>
	                <!-- Import dependency management from Spring Boot -->
	                <groupId>org.springframework.boot</groupId>
	                <artifactId>spring-boot-dependencies</artifactId>
	                <version>1.4.3.RELEASE</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
	    </dependencyManagement>
	    
	    <!-- Additional lines to be added here... -->
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.google.code.gson</groupId>
	            <artifactId>gson</artifactId>
	        </dependency>
	    </dependencies>
	    
	    <build>
	        <plugins>
	            <plugin>
	                <groupId>org.springframework.boot</groupId>
	                <artifactId>spring-boot-maven-plugin</artifactId>
	                <version>1.4.3.RELEASE</version>
	                <executions>
	                    <execution>
	                        <goals>
	                            <goal>repackage</goal>
	                        </goals>
	                    </execution>
	                </executions>
	            </plugin>
	        </plugins>
	    </build>
	</project>
```
有几点需要注意，这里我们没有继承Spring Boot的父Pom，也没继承Spring IO Platform的父POM，都是选择导入的方式，所以使用spring-boot-maven-plugin插件的时候，就不能像上面那样自动继承父POM的配置了，需要自己添加配置，绑定repackage Goal；

另外，当你想要修改依赖版本号的时候，由于不是继承，所以不能使用直接覆盖properties属性的方法，其实也很简单，如果不想继承Spring IO Platform中的依赖版本号的话，自己直接写上版本号即可，Spring Boot的话，可采用如下方式，来对Spring Data release train进行升级（注意要放在spring-boot-dependencies的前面）：
```Xml
	<dependencyManagement>
	    <dependencies>
	        <!-- Override Spring Data release train provided by Spring Boot -->
	        <dependency>
	            <groupId>org.springframework.data</groupId>
	            <artifactId>spring-data-releasetrain</artifactId>
	            <version>Fowler-SR2</version>
	            <scope>import</scope>
	            <type>pom</type>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-dependencies</artifactId>
	            <version>1.4.3.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
```
参考来源：<https://spring.io/blog/2014/06/26/introducing-the-spring-io-platform>