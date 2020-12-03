---
title: SpringCloud系列之--Feign模拟RPC调用
date: 2020-07-18 14:04:00
updated: 2020-07-18 14:04:00
tags: SpringCloud
categories: SpringCloud
keywords: Java, SpringCloud, 微服务
type: 
description: 使用Feign模拟RPC调用初体验。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img4.jpg
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
# Feign简介

Feign是一个声明式的Web Service客户端，它能够让Web Service客户端的编写变得更加容易（你只需创建一个接口，并在接口上添加相应注解即可）。除了Feign自带的注解外它还支持JAX-RS注解，SpringCloud又为Feign增加了对SpringMVC注解的支持，同时为了能够使用和Spring Web中默认使用的相同的httpMessageConverter，SpringCloud集成了Ribbon和Eureka，用来在使用Feign时能够为其提供一个负载均衡的HTTP客户端。

总起来说，Feign具有如下特性：

- 可插拔的注解支持，包括Feign注解和JAX-RS注解;
- 支持可插拔的HTTP编码器和解码器;
- 支持Hystrix和它的Fallback;
- 支持Ribbon的负载均衡;
- 支持HTTP请求和响应的压缩。

接下来我们将通过对上一章《Ribbon客户端负载均衡》中的 message-center 项目进行改造，演示如何使用Feign。

# message-center改造
## 引入Feign依赖
由于Feign依赖中默认包含了Ribbon，所以只需要在 pom.xml 文件中引入Feign依赖即可，Ribbon依赖无需重复引入：
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
		<!-- Feign 依赖 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
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

## 修改启动类
在MessageCenterApplication启动类上增加@EnableFeignClients注解：
```Java
	import org.springframework.boot.WebApplicationType;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.boot.builder.SpringApplicationBuilder;
	import org.springframework.cloud.openfeign.EnableFeignClients;
	 
	@SpringBootApplication
	@EnableFeignClients
	public class MessageCenterApplication {
	 
		public static void main(String[] args) {
			new SpringApplicationBuilder(MessageCenterApplication.class).web(WebApplicationType.SERVLET).run(args);
		}
	 
	}
```
这里我们在启动类中增加了@EnableFeignClients注解，用来开启Feign客户端发现功能。

如果你的Feign客户端类文件不在Spring的包扫描路径之中，可以在@EnableFeignClients注解中对Feign客户端的包路径进行指定。
```Java
	@SpringBootApplication
	@EnableFeignClients(basePackages = "com.pengjunlee.client.**")
	public class MessageCenterApplication {
	 
		public static void main(String[] args) {
			new SpringApplicationBuilder(MessageCenterApplication.class).web(WebApplicationType.SERVLET).run(args);
		}
	 
	}
```

## 创建Feign客户端
在上一章《Ribbon客户端负载均衡》中，对外提供服务的HTTP接口定义在MessageController中。
```Java
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
接下来，我们在消费端message-center中为它创建一个Feign客户端。新建一个接口取名MessageServiceClient，并在上面添加`@FeignClient`注解，完整代码如下：
```Java
	import org.springframework.cloud.openfeign.FeignClient;
	import org.springframework.web.bind.annotation.GetMapping;
	 
	@FeignClient(name = "message-service")
	public interface MessageServiceClient {
	 
		@GetMapping("/api/v1/msg/get")
		public String getMsg();
	}
```

> 说明：此处@FeignClient注解的name属性应与message-service应用的spring.application.name属性相同，表示为message-service服务创建一个Feign客户端。接口的映射地址路径以及接口入参都必须与MessageController中的方法完全相同。

## 调用Feign客户端
接下来，我们来看一看如何在消费端使用创建好的Feign客户端对message-service服务进行调用，示例代码如下：
```Java
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	 
	import com.pengjunlee.service.MessageServiceClient;
	 
	@RestController
	@RequestMapping("/api/v1/center")
	public class MessageCenterController {
	 
		@Autowired
		private MessageServiceClient messageService;
	 
		@GetMapping("/msg/get")
		public Object getMsg() {
			String msg = messageService.getMsg();
			return msg;
		}
	}
```
启动应用，再次请求 <http://localhost:8781/api/v1/center/msg/get> ，返回如下结果表明服务调用成功：

<div align=center>

![Feign示意图](http://pengjunlee.3vzhuji.net/static/springcloud/11.png "Feign示意图")
<div align=left>

# 关于传参
Feign除了支持自带的注解和JAX-RS注解外，还支持 SpringMVC注解，常用的有：@RequestParam 、@PathVariable、@RequestBody 等。

例如，服务端提供如下两个服务接口： 
```Java
	/**
	 * 获取消息详情
	 */
	@GetMapping("/api/v1/msg/detail/{id}")
	public MessageEntity getDetail(@PathVariable(name = "id") Long id) {
		return messageService.getById(id);
	}
 
	/**
	 * 新建一条消息
	 */
	@PostMapping("/api/v1/msg/save")
	public MessageEntity save(@RequestBody MessageEntity message) {
		return messageService.save(message);
	}
```
相应的，在Feign客户端中可以进行如下定义：
```Java
	/**
	 * 获取消息详情
	 */
	@GetMapping("/api/v1/msg/detail/{id}")
	public MessageEntity getDetail(@PathVariable(name = "id") Long id) ;
 
	/**
	 * 新建一条消息
	 */
	@PostMapping("/api/v1/msg/save")
	public MessageEntity save(@RequestBody MessageEntity message) ;
```

# 重写Feign的默认配置
在Spring Cloud对Feign的支持实现中，一个核心的概念就是客户端命名，每一个Feign客户端都是整个组件系统的一部分，它们相互协同一起工作来按照需求与远程服务器取得联系。并且它们每一个都有自己的名字，应用程序开发人员可以使用@feignclient来给它取名。Spring Cloud按照自己的需要又使用FeignClientsConfiguration为每一个已命名的客户端创建了一个ApplicationContext，额外包含了一个feign.Decoder、一个 feign.Encoder 和一个 feign.Contract。你可以通过指定@FeignClient注解的contextId 属性来设置ApplicationContext的名字。

SpringCloud还允许你在FeignClientsConfiguration的基础之上使用@FeignClient声明一些额外的配置，从而实现对Feign客户端的完全控制，如下例所示：
```Java
	@FeignClient(name = "message-service", configuration = MessageConfiguration.class)
	public interface MessageServiceClient {
	    //..
	}
```
在这个例子中，这个Feign客户端将由FeignClientsConfiguration 和MessageConfiguration中的组件一起组成（后者会覆盖前者的配置）。

> **注意：**本例中，MessageConfiguration不必用@Configuration注解进行标注，如果确实要加上@Configuration注解，你需要注意把MessageConfiguration排除在@ComponentScan和@SpringBootApplication扫描的包路径之外，否则它将成为feign.Decoder、feign.Encoder 和 feign.Contract 等的默认值。

下表列出了 Spring Cloud Netflix 缺省为Feign提供的所有 Bean（Bean类型 Bean名称：Bean实现）：
```
	Decoder feignDecoder: ResponseEntityDecoder (包装了一个 SpringDecoder)
	Encoder feignEncoder: SpringEncoder
	Logger feignLogger: Slf4jLogger
	Contract feignContract: SpringMvcContract
	Feign.Builder feignBuilder: HystrixFeign.Builder
	Client feignClient: 启用 Ribbon 时是 LoadBalancerFeignClient，否则使用 feign.Client.Default。
```
你可以使用 OkHttpClient 或者 ApacheHttpClient 的Feign客户端，只需要将 feign.okhttp.enabled 或者 feign.httpclient.enabled 设置为 true ，并将相应类添加到项目的CLASSPATH即可。你也可以使用自定义的HTTP 客户端，使用 Apache 时提供一个ClosableHttpClient 类型Bean或者使用OK HTTP时提供一个OkHttpClient 类型Bean。

默认情况下，Spring Cloud Netflix 并没有为Feign提供下列的Bean，但依然会从Spring容器中查找这些类型的Bean用来创建Feign客户端。
```
	Logger.Level
	Retryer
	ErrorDecoder
	Request.Options
	Collection<RequestInterceptor>
	SetterFactory
```
创建这些类型的一个Bean 并将它写到 @FeignClient 声明的配置中，这样你就能够对这些Bean中的每一个进行重写。例如下面的 MessageConfiguration 利用feign.Contract.Default替代了SpringMvcContract 并向RequestInterceptor 集合中添加了一个RequestInterceptor 。
```Java
	@Configuration
	public class MessageConfiguration {
	    @Bean
	    public Contract feignContract() {
	        return new feign.Contract.Default();
	    }
	 
	    @Bean
	    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
	        return new BasicAuthRequestInterceptor("user", "password");
	    }
	}
```
当然，@FeignClient 也支持通过配置文件进行配置。
```Yml
	feign:
	  client:
	    config:
	      message-service:
	        connectTimeout: 5000
	        readTimeout: 5000
	        loggerLevel: full
	        errorDecoder: com.pengjunlee.SimpleErrorDecoder
	        retryer: com.pengjunlee.SimpleRetryer
	        requestInterceptors:
	          - com.pengjunlee.FooRequestInterceptor
	          - com.pengjunlee.BarRequestInterceptor
	        decode404: false
	        encoder: com.pengjunlee.SimpleEncoder
	        decoder: com.pengjunlee.SimpleDecoder
	        contract: com.pengjunlee.SimpleContract
```
默认配置可以通过@EnableFeignClients的defaultConfiguration属性进行指定，然后会被应用到所有的Feign客户端。如果你希望使用配置文件对所有的@FeignClient进行配置，可以使用 default 作为Feign客户端的名称。
```Yml
	feign:
	  client:
	    config:
	      default:
	        connectTimeout: 5000
	        readTimeout: 5000
	        loggerLevel: basic
```
如果我们同时使用了@Configuration Bean和文件配置，文件配置会优先生效。如果你希望优先使用 @Configuration Bean中的配置，可以将 feign.client.default-to-properties 设置为 false 。

如果你需要在RequestInterceptor中使用ThreadLocal 变量，你需要将Hystrix 的线程隔离策略设置为 SEMAPHORE 或者直接禁用Hystrix 。
```Yml
	# To disable Hystrix in Feign
	feign:
	  hystrix:
	    enabled: false
	 
	# To set thread isolation to SEMAPHORE
	hystrix:
	  command:
	    default:
	      execution:
	        isolation:
	          strategy: SEMAPHORE
```

# 关于超时
在启用Ribbon的情况下，Feign客户端是一个LoadBalancerFeignClient Bean，其内部有一个 execute() 方法用来发送一个HTTP请求并获取响应， 本质上其实还是使用的Ribbon做负载均衡，并使用RestTemplate发送的请求。execute() 接口声明如下：

public Response execute(Request request, Request.Options options) throws IOException;
其中，Request 用来封装HTTP请求的详细信息。
```Java
	/**
	 * 
	 * An immutable request to an http server.
	 * 
	 */
	 
	public final class Request {
	 
		private final String method;
		private final String url;
		private final Map<String, Collection<String>> headers;
		private final byte[] body;
		private final Charset charset;
	 
		// ...
	 
	}
```
Options 则封装了一些请求控制参数：
```Java
	public static class Options {
	 
		private final int connectTimeoutMillis;
		private final int readTimeoutMillis;
		private final boolean followRedirects;
		public Options(int connectTimeoutMillis, int readTimeoutMillis) {
			this(connectTimeoutMillis, readTimeoutMillis, true);
		}
	 
		public Options() {
			this(10 * 1000, 60 * 1000);
		}
	 
		// ...
	 
	}
```
从Options 源码来看，Feign客户端默认的读取超时时间为60秒。若同时使用了Hystrix，由于Hystrix 默认的读取超时时间为1秒，会导致Feign客户端默认的读取超时时间设置无效，即超过1秒即为读取超时。可使用如下配置同时对Feign客户端和Hystrix 的超时配置进行重写。
```Yml
	feign:
	  client:
	    config:
	      default:
	        connectTimeout: 5000
	        readTimeout: 5000
```

# 参考文章

<https://cloud.spring.io/spring-cloud-openfeign/single/spring-cloud-openfeign.html>

<https://www.jianshu.com/p/a0d50385e598>

<https://blog.csdn.net/chengqiuming/article/details/80713471>