---
title: SpringCloud系列之--Eureka服务注册中心
date: 2020-07-18 14:02:00
updated: 2020-07-18 14:02:00
tags: SpringCloud
categories: SpringCloud
keywords: Java, SpringCloud, 微服务
type: 
description: Eureka服务注册中心初体验。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img2.jpg
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
**Spring Cloud**是一系列框架的集合，它利用**Spring Boot**的开发便利性巧妙地简化了分布式系统基础设施的开发，构建了`服务治理(服务注册与发现)`、`配置中心`、`消息总线`、`负载均衡`、`断路器`、`数据监控`、`分布式会话`和`集群状态管理`等功能，为我们**提供一整套企业级分布式云应用的完美解决方案**。

Spring Cloud的服务治理等核心功能主要是通过Spring Cloud Netflix的相关产品来实现，包括：`服务发现（Eureka）`、`断路器（Hystrix）`、`智能路由（Zuul）`和`客户端负载均衡（Ribbon）`。

本文主要对如何使用Eureka搭建服务注册中心进行介绍，我们先从最简单的单机模式Eureka服务器搭建开始。

# 关于SpringCloud版本
由于Spring Cloud是诸多子项目集合的综合项目，原则上由其子项目维护自己的发布版本号，也就是我们常用的版本号，如:1.2.3.RELEASE、1.1.4.RELEASE等。因此Spring Cloud为了避免版本号与其子项目的版本号混淆，所以没有采用版本号的方式，而是采用命名的方式。这些版本名称采用了伦敦地铁站的名字，根据字母表的顺序来对应版本时间顺序。比如，最早的Release版本名称为Angel，第二个Release版本的名称为Brixton，以此类推……。而我们在本系列文章所使用的版本名称为:Finchley.SR2，也就是目前的最新版本，其中的`SR`是`service releases`的简写，而1则是该版本名称中的第1个版本。其对应的Springboot版本为2.0.6.RELEASE。

关于更多版本的介绍，请参考[官网](http://spring.io/projects/spring-cloud "官网")。

# 单机模式Eureka注册中心

## 引入Eureka-Server依赖
新建一个 Maven 工程，并在其` pom.xml `文件中引入依赖，内容如下：
```Html
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
	</parent>
 
	<properties>
		<spring-cloud.version>Finchley.SR2</spring-cloud.version>
	</properties>
 
	<dependencies>
		<!-- Eureka-Server 依赖 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>
	</dependencies>
 
	<dependencyManagement>
		<dependencies>
			<!-- SpringCloud 版本控制依赖 -->
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```

## 创建启动类
```Java
	import org.springframework.boot.WebApplicationType;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.boot.builder.SpringApplicationBuilder;
	import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
	 
	@SpringBootApplication
	@EnableEurekaServer
	public class EurekaServerApplication {
	 
		public static void main(String[] args) {
			new SpringApplicationBuilder(EurekaServerApplication.class).web(WebApplicationType.SERVLET).run(args);
		}
	}
```

## 添加配置
默认情况下，每一个Eureka服务端同时也是Eureka客户端，你需要为它提供（至少一个）service-url 让它用来定位它的同类（其他Eureka服务端），如果你一个service-url 都不提供，服务虽然能够运行并且正常工作，但是它会在你的日志文件中插入很多由于它不能成功注册到同类而生成的噪音日志。 在集群模式下，两个Eureka（一个服务端和一个客户端）通过注册表合并和心跳监控能够让一个独立的Eureka服务端从故障中完美复活（只要还有监控或者弹性运行环境使它保持存活）。在单机模式，我们需要关闭Eureka的这些客户端行为，这样的话它就不会再不停地去尝试连接它的同类并不停地失败了。

在 Springboot的核心配置文件 application.yml 中加入如下配置来关闭Eureka的客户端行为：
```Yml
	server:
	  port: 8761
	 
	eureka:
	  instance:
	    hostname: localhost
	  client:
	    registerWithEureka: false
	    fetchRegistry: false
	    serviceUrl:
	      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```
其中fetch-registry和register-with-eureka两个配置用来关闭Eureka的客户端行为。

- fetch-registry：表示是否从eureka server获取注册表信息，如果是单一节点，不需要同步其他eureka server节点，则可以设置为false；若是集群，则设置为true，默认为true。
- register-with-eureka：表示是否将自己注册到eureka server，默认为true。

# 高可用Eureka注册中心

由于Eureka服务端没有后台存储，但是所有的服务实例都需要不断地向Eureka服务端发送心跳来更新它们在注册表中的状态，所以，这一系列的功能只能在内存中完成的。同时，每一个Eureka客户端都有一个内存缓存存储了Eureka的注册表信息，对服务的请求可以直接从缓存的注册表中获取，并不需要每一次都到Eureka注册表中去获取。

你可以添加多个同类Eureka实例到你的系统中，只要它们彼此之间能够相互连接，它们就能够在彼此之间进行注册表同步，这能够让Eureka具有更高的弹性和可用性。事实上，这也是Eureka的默认行为，所以你唯一要做的就是配置一个有效的同类serviceUrl 来使它生效。

## 双节点注册中心

### 修改配置文件

修改上例中的application.yml文件，内容如下：
```Yml
	---
	server:
	  port: 8761
	spring:
	  profiles: peer1
	  application:
	    name: eureka-server
	eureka:
	  instance:
	    hostname: peer1
	  client:
	    serviceUrl:
	      defaultZone: http://peer2:8762/eureka/
	  
	---
	server:
	  port: 8762
	spring:
	  profiles: peer2
	  application:
	    name: eureka-server
	eureka:
	  instance:
	    hostname: peer2
	  client:
	    serviceUrl:
	      defaultZone: http://peer1:8761/eureka/
```
有了这个YAML文件，我们就能够在一台服务器上通过在启动时指定不同的`Spring profile`来模拟启动两个主机（**peer1**和**peer2**）了。事实上，如果你是在一个知晓自己主机名（默认主机是通过`java.net.InetAddress` 进行查找）的服务器上运行程序，`eureka.instance.hostname `配置项并不是必须的。

### 修改hosts文件

修改Windows系统的hosts文件:
```
	# Windows：C:\Windows\System32\drivers\etc\hosts
	# Linux：/etc/hosts
```
在hosts文件中加入如下配置：
```
	127.0.0.1 peer1
	127.0.0.1 peer2
	127.0.0.1 peer3
```

### 启动测试
`右键-->Run As --> Run Configurations...`，分别以peer1和peeer2 配置信息启动EurekaServerApplication。
```
	--spring.profiles.active=peer1
	--spring.profiles.active=peer2
```

<div align=center>

![Eureka示意图](http://pengjunlee.3vzhuji.net/static/springcloud/2.png "Eureka示意图")
<div align=left>


 依次启动完成后，浏览器输入：`<http://peer1:8761/> ` 效果图如下：
<div align=center>

![Eureka示意图](http://pengjunlee.3vzhuji.net/static/springcloud/3.png "Eureka示意图")
<div align=left>


## 多节点注册中心
### 修改配置文件
在生产中我们可能需要三台或者大于三台的注册中心来保证服务的稳定性，配置的原理其实都一样，将注册中心分别指向其它的注册中心。这里只介绍三台集群的配置情况，其实和双节点的注册中心类似，修改上例中的application.yml文件，再添加一个 profiles 配置，内容如下：
```Yml
	---
	server:
	  port: 8761
	spring:
	  profiles: peer1
	  application:
	    name: eureka-server
	eureka:
	  instance:
	    hostname: peer1
	  client:
	    serviceUrl:
	      defaultZone: http://peer2:8762/eureka/,http://peer3:8763/eureka/
	  
	---
	server:
	  port: 8762
	spring:
	  profiles: peer2
	  application:
	    name: eureka-server
	eureka:
	  instance:
	    hostname: peer2
	  client:
	    serviceUrl:
	      defaultZone: http://peer1:8761/eureka/,http://peer3:8763/eureka/
	          
	---
	server:
	  port: 8763
	spring:
	  profiles: peer3
	  application:
	    name: eureka-server
	eureka:
	  instance:
	    hostname: peer3
	  client:
	    serviceUrl:
	      defaultZone: http://peer1:8761/eureka/,http://peer2:8762/eureka/
```
### 启动测试
分别加载三个profiles的配置启动EurekaServerApplication。
<div align=center>

![Eureka示意图](http://pengjunlee.3vzhuji.net/static/springcloud/4.png "Eureka示意图")
<div align=left>


# 常见问题  
<font color=red>EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.</font>

**出现原因**：默认情况下，为了防止由于网络问题而造成的已经正常启动的Eureka实例无法成功注册，Eureka会开启自我保护模式，这样即使Eureka实例续约失败也不会从可用列表中被剔除，可继续从注册表中返回并对外提供服务。

**解决办法**：关闭自我保护模式，将配置 eureka.server.enable-self-preservation 设置为 false 。关闭自我保护之后，提示信息将变为如下内容：

<font color=red>THE SELF PRESERVATION MODE IS TURNED OFF.THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.</font>

# 参考文章

<https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.0.2.RELEASE/single/spring-cloud-netflix.html>

<https://www.jianshu.com/p/d32ae141f680>

<http://spring.io/projects/spring-cloud>