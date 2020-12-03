---
title: SpringCloud系列之--Zuul高级配置(上)
date: 2020-07-18 14:07:00
updated: 2020-07-18 14:07:00
tags: SpringCloud
categories: SpringCloud
keywords: Java, SpringCloud, 微服务
type: 
description: 介绍一下关于Zuul的高级配置。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img7.jpg
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
# 自定义路由规则
在《Zuul路由和过滤》一章中，我们并未对Zuul的路由规则进行设置，默认会使用服务的 ID 对服务进行路由，即：在源服务的URI之前增加 /service-id 前缀。

	# Zuul 默认路由地址
	http://<zuul-host>:<zuul-port>/service-id/[service-URI]
除了可以直接使用默认的路由规则外，Zuul还提供了很多配置项允许我们对路由映射规则进行自定义，例如：
```Yml
	zuul:
	  ignoredServices: '*'
	  routes:
	    message-service: /zuul-msg/**
```
示例中这个配置的作用是：忽略除 message-service 外的所有服务（不对它们进行路由），只将message-service 服务路由到 /zuul-msg/** 地址上。
```
	zuul.ignored-services  # 忽略指定微服务，多个微服务之间使用逗号分隔
	zuul.routes.service-id # 指定service-id服务的路由地址
```
> <font color=red>注意：在 zuul.routes 中配置了的服务，即使被包含在 zuul.ignored-services 配置中也不会被忽略。</font>

若将上例中的配置改成如下，则表示除 message-service 外的所有服务都使用默认的路由规则。
```Yml
	zuul:
	  routes:
	    message-service: /zuul-msg/**
```
如果你希望对某一个路由进行更精细的控制，可以独立地指定该服务的路由地址和它的service-id，例如：
```Yml
	zuul:
	  routes:
	    message: 
	      path: /zuul-msg/**
	      serviceId: message-service
```
或者，像下面这样指定一个该服务的物理地址来替代上例中的service-id：
```Yml
	zuul:
	  routes:
	    message: 
	      path: /zuul-msg/**
	      url: http://localhost:8771/
```
连续多次访问 <http://localhost:8791/zuul-msg/api/v1/msg/get> 返回结果相同，见下图：
<div align=center>

![Zuul示意图](http://pengjunlee.3vzhuji.net/static/springcloud/21.png "Zuul示意图")
<div align=left>


可以看出，通过url配置的路由不会被当作HystrixCommand执行，自然也就不会使用Ribbon在多个Url之间进行负载均衡。所以，推荐使用serviceId进行配置。或者，指定一个包含有多个可用服务列表的serviceId，例如：
```Yml
	zuul:
	  routes:
	    message: 
	      path: /zuul-msg/**
	      serviceId: msg-servers
	      
	hystrix:
	  command:
	    msg-servers:
	      execution:
	        isolation:
	          thread:
	            timeoutInMilliseconds: 1000
	      
	msg-servers:
	  ribbon:
	    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
	    listOfServers: http://localhost:8771/,http://localhost:8772/
	    ConnectTimeout: 1000
	    ReadTimeout: 3000
	    MaxTotalHttpConnections: 500
	    MaxConnectionsPerHost: 100
```
还有另外一种方法也可以达到上述目标：使用service-id为要路由的服务指定一个Ribbon客户端，并设置Ribbon禁用Eureka，例如：
```Yml
	zuul:
	  routes:
	    message: 
	      path: /zuul-msg/**
	      serviceId: msg-servers
	      
	ribbon:
	  eureka:
	    enabled: false
	 
	msg-servers:
	  ribbon:
	    listOfServers: http://localhost:8771/,http://localhost:8772/
```
如果你的service-id命名满足一定的规则，可以使用正则表达式从service-id中提取一些变量作为你的路由地址，例如：
```Java
	@Bean
	public PatternServiceRouteMapper serviceRouteMapper() {
		return new PatternServiceRouteMapper("(?<name>^.+)-(?<version>v.+$)", "${version}/${name}");
	}
```
在这个示例中，一个名称为myusers-v1的服务会被路由到 /v1/myusers/** 地址上。在这里你可以使用任意正则表达式，但是要保证所有的命名组必须都包含servicePattern和routePattern两部分。如果servicePattern不匹配service-id，就会使用默认的规则，即在服务原有的URI之前增加 /service-id 前缀。Zuul的这一特性默认是关闭的，而且只能被应用到已经发现的服务。

对那些要忽略的微服务也可以采用模式匹配进行更加精细的控制，例如：
```Yml
	zuul:
	  ignoredPatterns: /**/v2/**
	  routes:
	    message-service: /zuul-msg/**
```
该例中的配置会忽略所有包含 /v2/ 的请求。 

使用 zuul.prefix 配置项可以为所有的路由地址都添加一个前缀，例如 /abc 。
```Yml
	zuul:
	  prefix: /abc
	  routes:
	    message-service: /zuul-msg/**
```
添加前缀之后路由地址为：<http://localhost:8791/abc/zuul-msg/api/v1/msg/get> 。 

<div align=center>

![Zuul示意图](http://pengjunlee.3vzhuji.net/static/springcloud/22.png "Zuul示意图")
<div align=left>

默认情况下，Zuul代理会在将请求转发出去之前先将其中的前缀字符串剔除掉。如果你不想剔除前缀，可以设置 zuul.stripPrefix=false。如果你只是想不剔除指定某一个服务中的前缀，可以使用如下配置：
```Yml
	zuul:
	  prefix: /abc
	  # stripPrefix: false
	  routes:
	    message-service: /zuul-msg/**
	    stripPrefix: false
```
zuul.routes 中的配置条目最终都会被绑定到一个ZuulProperties类型的对象，在 ZuulProperties 中还有一个 retryable 标识，将该标识设置成 ture 能够让Ribbon客户端对失败的请求自动进行重试。

zuul.routes 中配置的路由地址最终都会被存储在一个Map中，路由的地址即为Map的key，Map的value是一个ZuulRoute类型的对象。在yaml配置文件中，路由地址相同的两个服务，后面的会覆盖前面的配置，从而导致先配置的服务不可用。

> <font color=red>**注意**：如果需要保留路由的顺序，需要使用yaml文件，因为使用properties文件时会丢失顺序。</font>

目前，Zuul已经使用 Apache HTTP Client 替换掉了已经过时的Ribbon RestClient 来作为自己的HTTP Client。你若想继续使用 RestClient 或者 okhttp3.OkHttpClient，只需要将 ribbon.restclient.enabled 或者 ribbon.okhttp.enabled 设置为 true 即可。当然，你也可以使用自定义的 Apache HTTP client 或者 OK HTTP client，提供一个相应的 ClosableHttpClient 或者OkHttpClient 类型的 Bean即可。

# 设置请求头
默认情况下，Zuul会为转发的请求添加一些 X-Forwarded-* 请求头，可以通过配置项 zuul.addProxyHeaders = false 来关闭它。请求中的前缀默认会被剔除，同样地，请求中的X-Forwarded-Prefix 请求头也会被摘取出来。

在同一个系统中，请求头在多个服务之间是可以共享的，但是你可能并不希望某些敏感的请求头信息泄露到下游的外部服务器，你可以把这些敏感请求头配置到zuul.sensitiveHeaders列表，从而禁止它们被传递到下游。例如：
```Yml
	zuul:
	  sensitiveHeaders: Cookie,Set-Cookie,Authorization
	  routes:
	    message-service: /zuul-msg/**
	    sensitiveHeaders: token
```
sensitiveHeaders就相当于是一个向下游传递的请求头黑名单，默认包含了 Cookie，Set-Cookie和Authorization三个请求头。因此，如果你需要向下传递所有的请求头信息，需要明确地把sensitiveHeaders设置成一个空列表，如下：
```Yml
	zuul:
	  sensitiveHeaders: 
	  routes:
	    message-service: /zuul-msg/**
```
另外，你还可以把那些不需要传递到下游的请求头或响应头配置到 zuul.ignoredHeaders 中，可以达到相同的作用。默认情况下，当类路径中不包含 Spring Security 时，ignoredHeaders 列表是空的；当类路径中包含 Spring Security时，ignoredHeaders 列表会被初始化包含一些由Spring Security指定的 security 头信息，例如 involving caching。这种情况下如果你需要把这些 security 头信息传递到下游，可以添加 zuul.ignoreSecurityHeaders = false 配置项。

# 管理终端
默认情况下，当@EnableZuulProxy与Actuator一起使用时，Actuator健康监控页面会增加两个终端：Routes和Filters。

需要在pom.xml中引入Actuator依赖：
```Html
	<!-- Actuator 依赖 -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
```
并在application.yml文件中增加配置：
```Yml
	management:
	  security:
	    enabled: false
	  endpoints:
	    web:
	      exposure:
	        include: hystrix.stream,*
```
访问 /actuator/routes 终端会返回一个映射路由的列表，如下图。
<div align=center>

![Zuul示意图](http://pengjunlee.3vzhuji.net/static/springcloud/23.png "Zuul示意图")
<div align=left>



访问 /actuator/routes/details 终端会返回一个映射路由的详细信息列表，如下图。

<div align=center>

![Zuul示意图](http://pengjunlee.3vzhuji.net/static/springcloud/24.png "Zuul示意图")
<div align=left>


访问 /actuator/filters 终端会返回一个包含了过滤器类型和配置的Map对象。
<div align=center>

![Zuul示意图](http://pengjunlee.3vzhuji.net/static/springcloud/25.png "Zuul示意图")
<div align=left>

# 编码设置
在对进来的请求进行处理的时候，请求参数会被解码以便在Zuul过滤器中能够对它们进行一些修改，然后又会在路由过滤器中重新编码并发送出去，重新编码后的结果跟原来的输入可能会有所不同，这在多数情况下是没什么问题的，但是在一些对复杂的请求字符串编码比较挑剔的Web服务器中仍有可能会出问题。解决的办法就是添加如下的配置来强制使用原始编码的请求字符串，相当于直接使用 HttpServletRequest.getQueryString()方法来获取请求字符串。

在对进来的请求进行处理的时候，会先对请求的URI进行解码，再和路由地址进行匹配，然后又会在路由过滤器中对请求的URI重新编码并发送给后端，当你的URI中包含 “/” 字符时这可能会引发一些意想不到的问题。解决的办法就是添加如下的配置来强制使用原始编码的请求URI，相当于直接使用 HttpServletRequest.getRequestURI()方法来获取请求URI。

# 禁用Zuul过滤器
Spring Cloud的Zuul自带了一些过滤器Bean，默认情况下它们都是启用的。如果你想禁用其中的某一个过滤器，可以使用如下配置：
```
zuul.<SimpleClassName>.<filterType>.disable=true
```
按照惯例，过滤器所在包名中 “filters” 字符串后面的就是过滤器的类型了，例如，禁用 `org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter` ，配置如下：
```
zuul.SendResponseFilter.post.disable=true
```

# 超时设置

- 如果你需要设置通过Zuul路由的请求的Socket超时时间和Read超时时间，有两种配置方法，需要根据你的配置情况进行选择：
- 如果使用了服务发现，使用ribbon.ReadTimeout 和 ribbon.SocketTimeout 进行配置。
- 如果是通过指定URL的方式进行路由，使用zuul.host.connect-timeout-millis 和 zuul.host.socket-timeout-millis进行配置。

# 参考文章

<https://github.com/spring-projects/spring-retry>

<https://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#netflix-zuul-starter>