---
title: SpringCloud系列之--Ribbon客户端负载均衡
date: 2020-07-18 14:03:00
updated: 2020-07-18 14:03:00
tags: SpringCloud
categories: SpringCloud
keywords: Java, SpringCloud, 微服务
type: 
description: Ribbon客户端负载均衡初体验。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img3.jpg
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
# 服务器端负载均衡
负载均衡是我们处理高并发、缓解网络压力和进行服务器扩容的重要手段之一，但是一般情况下我们所说的负载均衡通常都是指服务器端负载均衡，服务器端负载均衡又分为两种，一种是硬件负载均衡，还有一种是软件负载均衡。

硬件负载均衡主要通过在服务器节点之前安装专门用于负载均衡的设备，常见的如：F5。

软件负载均衡则主要是在服务器上安装一些具有负载均衡功能的软件来完成请求分发进而实现负载均衡，常见的如：LVS 、 Nginx 、Haproxy。

无论是硬件负载均衡还是软件负载均衡，它的工作原理都不外乎下面这张图：
<div align=center>

![Ribbon示意图](http://pengjunlee.3vzhuji.net/static/springcloud/5.png "Ribbon示意图")
<div align=left>



# 客户端负载均衡
而微服务的出现，则为负载均衡的实现提供了另外一种思路：把负载均衡的功能以库的方式集成到服务的消费方，而不再是由一台指定的负载均衡设备集中提供。这种方案称为软负载均衡（Soft Load Balancing）或者客户端负载均衡。常见的如：`Spring Cloud`中的 `Ribbon`。

Ribbon是一个基于HTTP和TCP的客户端负载均衡器，当我们将Ribbon和Eureka一起使用时，Ribbon会到Eureka注册中心去获取服务端列表，然后进行轮询访问以到达负载均衡的作用，客户端负载均衡也需要心跳机制去维护服务端清单的有效性，当然这个过程需要配合服务注册中心一起完成。

服务器端负载均衡 VS 客户端负载均衡的特点如下：

- 服务器端负载均衡 客户端先发送请求到负载均衡服务器，然后由负载均衡服务器通过负载均衡算法，在众多可用的服务器之中选择一个来处理请求。
- 客户端负载均衡 客户端自己维护一个可用服务器地址列表，在发送请求前先通过负载均衡算法选择一个将用来处理本次请求的服务器，然后再直接将请求发送至该服务器。

接下来我们将延续上一章的Eureka注册中心，继续搭建一个基于Ribbon的客户端负载均衡示例。

# Ribbon负载均衡示例搭建
## 创建服务提供者
### 引入依赖
新建一个 Maven 工程，取名 message-service 用来提供消息服务，在其 pom.xml 文件中引入依赖，内容如下：
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
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka-Client 依赖 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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

### 添加配置
`application.yml `添加配置如下： 
```Yml
	spring:
	  application:
	    name: message-service
	eureka:
	  client:
	    serviceUrl:
	      defaultZone: http://localhost:8761/eureka/
```

### 服务提供者
创建一个 MessageController 控制器对外提供一个http接口服务。
```Java
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	 
	@RestController
	@RequestMapping("/api/v1/msg")
	public class MessageController {
	 
		@Value("${server.port}")
		private String port;
	 
		/**
		 * 返回一条消息
		 */
		@GetMapping("/get")
		public String getMsg() {
			return "This message is sent from port: " + port;
		}
	}
```

### 创建启动类
```Java
	import org.springframework.boot.WebApplicationType;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.boot.builder.SpringApplicationBuilder;
	import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
	 
	@SpringBootApplication
	@EnableEurekaClient
	public class MessageApplication {
	 
		public static void main(String[] args) {
			new SpringApplicationBuilder(MessageApplication.class).web(WebApplicationType.SERVLET).run(args);
		}
	 
	}
```
### 启动服务
`右键-->Run As --> Run Configurations...`，分别使用 8771、8772、8773 三个端口各启动一个MessageApplication应用。
```
	-Dserver.port=8771
	-Dserver.port=8772
	-Dserver.port=8773
```
<div align=center>

![Ribbon示意图](http://pengjunlee.3vzhuji.net/static/springcloud/6.png "Ribbon示意图")
<div align=left>

启动完成后，浏览器输入：<http://localhost:8761/> 效果图如下：

<div align=center>

![Ribbon示意图](http://pengjunlee.3vzhuji.net/static/springcloud/7.png "Ribbon示意图")
<div align=left>

## 服务消费者
### 引入Ribbon依赖
新建一个 Maven 工程，取名 message-center 用来调用消息服务，在其 pom.xml 文件中引入依赖，内容如下：
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
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- Eureka-Client 依赖 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
 
		<!-- Ribbon 依赖 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
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
### 添加配置
application.yml 添加配置如下： 
```Yml
	server:
	  port: 8781
	spring:
	  application:
	    name: message-center
	eureka:
	  client:
	    serviceUrl:
	      defaultZone: http://localhost:8761/eureka/
```
### 使用Ribbon客户端
**方式一：**

在启动类中向Spring容器中注入一个带有@LoadBalanced注解的RestTemplate Bean。
```Java
	import org.springframework.boot.WebApplicationType;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.boot.builder.SpringApplicationBuilder;
	import org.springframework.cloud.client.loadbalancer.LoadBalanced;
	import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
	import org.springframework.context.annotation.Bean;
	import org.springframework.web.client.RestTemplate;
	 
	@SpringBootApplication
	@EnableEurekaClient
	public class MessageCenterApplication {
	 
		@Bean
	 
		@LoadBalanced
	 
		public RestTemplate restTemplate() {
	 
			return new RestTemplate();
	 
		}
	 
		public static void main(String[] args) {
	 
			new SpringApplicationBuilder(MessageCenterApplication.class).web(WebApplicationType.SERVLET).run(args);
	 
		}
	 
	}
```
在调用那些需要做负载均衡的服务时，使用上面注入的RestTemplate Bean进行调用即可。
```Java
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.client.RestTemplate;
	 
	@RestController
	@RequestMapping("/api/v1/center")
	public class MessageCenterController {
	 
		@Autowired
		private RestTemplate restTemplate;
	 
		@GetMapping("/msg/get")
		public Object getMsg() {
	 
			String msg = restTemplate.getForObject("http://message-service/api/v1/msg/get", String.class);
			return msg;
	 
		}
	 
	}
```
**方式二：**

直接使用 LoadBalancerClient 中的负载均衡策略获取一个可用的服务地址，然后再进行请求。
```Java
	import java.net.URI;
	 
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.cloud.client.ServiceInstance;
	import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	import org.springframework.web.client.RestTemplate;
	 
	@RestController
	@RequestMapping("/api/v1/center")
	public class MessageCenterController {
	 
		@Autowired
		private LoadBalancerClient loadBalancer;
	 
		@GetMapping("/msg/get")
		public Object getMsg() {
	 
			ServiceInstance instance = loadBalancer.choose("message-service");
			URI url = URI.create(String.format("http://%s:%s/api/v1/msg/get", instance.getHost(), instance.getPort()));
			RestTemplate restTemplate = new RestTemplate();
			String msg = restTemplate.getForObject(url, String.class);
			return msg;
	 
		}
	 
	}
```
待应用启动之后，连续三次请求地址 <http://localhost:8781/api/v1/center/msg/get> ，返回的结果如图所示：
<div align=center>

![Ribbon示意图](http://pengjunlee.3vzhuji.net/static/springcloud/8.png "Ribbon示意图")
<div align=left>

# 切换Ribbon负载均衡策略
Ribbon本身提供了下面几种负载均衡策略：
 
<div align=center>

![Ribbon示意图](http://pengjunlee.3vzhuji.net/static/springcloud/9.png "Ribbon示意图")
<div align=left>

- RoundRobinRule: 轮询策略，Ribbon以轮询的方式选择服务器，这个是默认值。所以示例中所启动的两个服务会被循环访问;
- RandomRule: 随机策略，也就是说Ribbon会随机从服务器列表中选择一个进行访问;
- BestAvailableRule: 最大可用策略，即先过滤出故障服务器后，选择一个当前并发请求数最小的;
- WeightedResponseTimeRule: 带有加权的轮询策略，对各个服务器响应时间进行加权处理，然后在采用轮询的方式来获取相应的服务器;
- AvailabilityFilteringRule: 可用过滤策略，先过滤出故障的或并发请求大于阈值的一部分服务实例，然后再以线性轮询的方式从过滤后的实例清单中选出一个;
- ZoneAvoidanceRule: 区域感知策略，先使用主过滤条件（区域负载器，选择最优区域）对所有实例过滤并返回过滤后的实例清单，依次使用次过滤条件列表中的过滤条件对主过滤条件的结果进行过滤，判断最小过滤数（默认1）和最小过滤百分比（默认0），最后对满足条件的服务器则使用RoundRobinRule(轮询方式)选择一个服务器实例。

我们可以将上例中的message-service的负载均衡策略设置为随机访问RandomRule，application.yml配置如下：
```Yml
	message-service:
	  ribbon:
	    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```
当然，我们也可以通过继承ClientConfigEnabledRoundRobinRule，来实现自己的负载均衡策略。

# 自定义Ribbon客户端
您可以使用外部属性按照 <clientName>.<nameSpace>.<propertyName>=<value> 的格式对Ribbon客户端的某些特性进行配置，这与使用Netflix API类似。当然，你也可以在Springboot配置文件中进行配置。所有的配置项均以静态字段的形式定义在 CommonClientConfigKey 类（Ribbon核心的一部分）中。

SpringCloud还允许你在RibbonClientConfiguration的基础之上使用@RibbonClient声明一些额外配置，从而实现对Ribbon客户端的完全控制，如下例所示：
```Java
	import org.springframework.cloud.netflix.ribbon.RibbonClient;
	import org.springframework.cloud.netflix.ribbon.ZonePreferenceServerListFilter;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	 
	import com.netflix.loadbalancer.IPing;
	import com.netflix.loadbalancer.PingUrl;
	import com.pengjunlee.TestConfiguration.MessageConfiguration;
	 
	@Configuration
	@RibbonClient(name = "message-service", configuration = MessageConfiguration.class)
	public class TestConfiguration {
	 
		@Configuration
		protected static class MessageConfiguration {
			@Bean
			public ZonePreferenceServerListFilter serverListFilter() {
				ZonePreferenceServerListFilter filter = new ZonePreferenceServerListFilter();
				filter.setZone("myTestZone");
				return filter;
			}
	 
			@Bean
			public IPing ribbonPing() {
				return new PingUrl();
			}
		}
	}
```
在这个例子中，这个Ribbon客户端将由RibbonClientConfiguration和MessageConfiguration中的组件一起组成（后者会覆盖前者的配置）。

> **注意**：本例中，MessageConfiguration必须用@Configuration注解标注，但是它不应该被包含在Spring的组件扫描路径之中，否则它将被所有的Ribbon客户端共享。如果你使用@ComponentScan（或者@SpringBootApplication），那么你应该采取措施来避免它被包含到扫描范围之中。

下表列出了 Spring Cloud Netflix 缺省为Ribbon提供的所有 Bean：

<div align=center>

![Ribbon示意图](http://pengjunlee.3vzhuji.net/static/springcloud/10.png "Ribbon示意图")
<div align=left>

创建这些类型的一个Bean 并将它写到 @RibbonClient 声明的配置中，这样你就能够对这些Bean中的每一个进行重写。例如上面的 MessageConfiguration 中利用PingUrl替代了NoOpPing并提供了一个自定义的serverListFilter。

# 自定义Ribbon客户端的默认配置
通过@RibbonClients注解可以为所有的Ribbon客户端提供一个默认的配置。例如
```Java
	import org.springframework.cloud.netflix.ribbon.RibbonClients;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	 
	import com.netflix.client.config.IClientConfig;
	import com.netflix.loadbalancer.BestAvailableRule;
	import com.netflix.loadbalancer.ConfigurationBasedServerList;
	import com.netflix.loadbalancer.IPing;
	import com.netflix.loadbalancer.IRule;
	import com.netflix.loadbalancer.PingUrl;
	import com.netflix.loadbalancer.Server;
	import com.netflix.loadbalancer.ServerList;
	import com.netflix.loadbalancer.ServerListSubsetFilter;
	 
	@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
	public class RibbonClientDefaultConfigurationTestsConfig {
	 
		public static class BazServiceList extends ConfigurationBasedServerList {
			public BazServiceList(IClientConfig config) {
				super.initWithNiwsConfig(config);
			}
		}
	}
	 
	@Configuration
	class DefaultRibbonConfig {
	 
		@Bean
		public IRule ribbonRule() {
			return new BestAvailableRule();
		}
	 
		@Bean
		public IPing ribbonPing() {
			return new PingUrl();
		}
	 
		@Bean
		public ServerList<Server> ribbonServerList(IClientConfig config) {
			return new RibbonClientDefaultConfigurationTestsConfig.BazServiceList(config);
		}
	 
		@Bean
		public ServerListSubsetFilter serverListFilter() {
			ServerListSubsetFilter filter = new ServerListSubsetFilter();
			return filter;
		}
	 
	}
```

# 通过配置属性自定义Ribbon客户端
Spring Cloud Netflix 从1.2.0版本开始支持通过配置属性自定义Ribbon客户端，这可以让你在应用启动时根据不同的环境来改变Ribbon的行为。

支持配置的属性列表如下：
```
	<clientName>.ribbon.NFLoadBalancerClassName: 需实现 ILoadBalancer
	<clientName>.ribbon.NFLoadBalancerRuleClassName: 需实现 IRule
	<clientName>.ribbon.NFLoadBalancerPingClassName: 需实现 IPing
	<clientName>.ribbon.NIWSServerListClassName: 需实现 ServerList
	<clientName>.ribbon.NIWSServerListFilterClassName: 需实现 ServerListFilter
```
> 注意：通过配置属性定义的类与上述@RibbonClient配置类中定义的Bean以及Spring Cloud Netflix 提供的缺省配置相比具有更高的优先级。

例如对 message-service 服务的 IRule 属性进行修改，可以在application.yml中进行如下配置：
```Yml
		message-service:
		  ribbon:
		    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
		    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```

# 脱离Eureka使用Ribbon
Eureka提供了一种抽象的发现远程服务的便捷的方式，这样你就不必在客户端中硬编码服务地址列表，但是如果你不用Eureka，也可以继续使用Ribbon和Feign。假设在不使用Eureka，你用@RibbonClient声明了一个"stores"服务，这个时候Ribbon Client 默认会引用一个配置好的服务列表，你可以在application.yml中对它进行配置：
```Yml
	stores:
	  ribbon:
	    listOfServers: example.com,google.com
```

# 在Ribbon中禁用Eureka
通过将 ribbon.eureka.enabled 属性设置为 false 可以在Ribbon中禁用Eureka。
```Yml
	ribbon:
	  eureka:
	   enabled: false
```

# Ribbon缓存配置
对于每一个Ribbon客户端，Spring Cloud都会维护一个与其相应的应用上下文，这个应用上下文会在相应服务第一次被请求时进行懒加载。你可以使用饥饿加载来取代懒加载，通过指定那些需要饥饿加载Ribbon客户端名称，在应用启动时就对其应用上下文进行加载。
```Yml
	ribbon:
	  eager-load:
	    enabled: true
	    clients: client1, client2, client3
```

# 参考文章

<https://www.jianshu.com/p/d32ae141f680>

<https://blog.csdn.net/u014401141/article/details/78676296>

<https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.0.2.RELEASE/single/spring-cloud-netflix.html#spring-cloud-ribbon>