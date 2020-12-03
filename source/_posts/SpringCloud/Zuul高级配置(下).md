---
title: SpringCloud系列之--Zuul高级配置(下)
date: 2020-07-18 14:08:00
updated: 2020-07-18 14:08:00
tags: SpringCloud
categories: SpringCloud
keywords: Java, SpringCloud, 微服务
type: 
description: 介绍一下关于Zuul的高级配置。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
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
# 为路由提供HystrixFallback

当Zuul中某一个路由的断路器被断开时，你可以通过创建一个FallbackProvider类型的Bean来为它提供一个Fallback响应。在这个Bean中你需要指定Fallback响应所对应的路由的ID并提供一个ClientHttpResponse作为返回的Fallback响应，下面是一个FallbackProvider实现的实例。
```Java
	import java.io.ByteArrayInputStream;
	import java.io.IOException;
	import java.io.InputStream;
	 
	import org.springframework.cloud.netflix.zuul.filters.route.FallbackProvider;
	import org.springframework.http.HttpHeaders;
	import org.springframework.http.HttpStatus;
	import org.springframework.http.MediaType;
	import org.springframework.http.client.ClientHttpResponse;
	import org.springframework.stereotype.Component;
	 
	import com.netflix.hystrix.exception.HystrixTimeoutException;
	 
	@Component
	public class MessageFallbackProvider implements FallbackProvider {
	 
		@Override
		public String getRoute() {
			return "zuul-msg";
		}
	 
		@Override
		public ClientHttpResponse fallbackResponse(String route, final Throwable cause) {
			if (cause instanceof HystrixTimeoutException) {
				return response(HttpStatus.GATEWAY_TIMEOUT);
			} else {
				return response(HttpStatus.INTERNAL_SERVER_ERROR);
			}
		}
	 
		private ClientHttpResponse response(final HttpStatus status) {
			return new ClientHttpResponse() {
				@Override
				public HttpStatus getStatusCode() throws IOException {
					return status;
				}
	 
				@Override
				public int getRawStatusCode() throws IOException {
					return status.value();
				}
	 
				@Override
				public String getStatusText() throws IOException {
					return status.getReasonPhrase();
				}
	 
				@Override
				public void close() {
				}
	 
				@Override
				public InputStream getBody() throws IOException {
					return new ByteArrayInputStream("消息服务暂时不可用，请稍后重试！".getBytes());
				}
	 
				@Override
				public HttpHeaders getHeaders() {
					HttpHeaders headers = new HttpHeaders();
					headers.setContentType(MediaType.APPLICATION_JSON);
					return headers;
				}
			};
		}
	}
```
路由的配置信息如下：
```Yml
	zuul:
	  routes:
	    message-service: /zuul-msg/**
```
当message-service服务调用失败时，返回结果见下图。

<div align=center>

![Zuul示意图](http://pengjunlee.3vzhuji.net/static/springcloud/35.png "Zuul示意图")
<div align=left>

如果你想为所有的路由提供一个默认的Fallback，可以创建一个FallbackProvider Bean并将它的getRoute() 方法的返回值设置为 “*”或者 null。
```Java
	class MyFallbackProvider implements FallbackProvider {
	    @Override
	    public String getRoute() {
	        return "*";
	    }
	 
	    @Override
	    public ClientHttpResponse fallbackResponse(String route, Throwable throwable) {
	        return new ClientHttpResponse() {
	            @Override
	            public HttpStatus getStatusCode() throws IOException {
	                return HttpStatus.OK;
	            }
	 
	            @Override
	            public int getRawStatusCode() throws IOException {
	                return 200;
	            }
	 
	            @Override
	            public String getStatusText() throws IOException {
	                return "OK";
	            }
	 
	            @Override
	            public void close() {
	 
	            }
	 
	            @Override
	            public InputStream getBody() throws IOException {
	                return new ByteArrayInputStream("fallback".getBytes());
	            }
	 
	            @Override
	            public HttpHeaders getHeaders() {
	                HttpHeaders headers = new HttpHeaders();
	                headers.setContentType(MediaType.APPLICATION_JSON);
	                return headers;
	            }
	        };
	    }
	}
```

# 重写Location头信息
如果Zuul面向的是一个Web应用，那么你可能会碰到需要重写Location请求头的情况，你只需要创建一个LocationRewriteFilter类型Bean即可。
```Java
	import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
	import org.springframework.cloud.netflix.zuul.filters.post.LocationRewriteFilter;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	 
	@Configuration
	@EnableZuulProxy
	public class ZuulConfig {
		@Bean
		public LocationRewriteFilter locationRewriteFilter() {
			return new LocationRewriteFilter();
		}
	}
```
> **注意**：此过滤器会作用于所有响应状态码为 3XX 的Location头信息，这并不一定适用于所有的场景，比如重定向到外部URL。

# 启用跨域请求
默认情况下，Zuul会把所有的Cross Origin requests (CORS，跨域请求)路由给相应的服务，如果你想代替Zuul自己来处理这些请求，你可以提供一个自定义的 WebMvcConfigurer Bean：
```Java
	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurer() {
			public void addCorsMappings(CorsRegistry registry) {
				registry.addMapping("/path-1/**").allowedOrigins("http://allowed-origin.com").allowedMethods("GET",
						"POST");
			}
		};
	}
```
在这个例子中，我们允许从 <http://allowed-origin.com> 向Zuul中以 path-1 开头的终端发送 GET和POST类型的跨域请求。你既可以将 CORS 配置作用于指定的路径，也可以使用 /** 让它作用于整个应用。你可以在其中配置以下属性：allowedOrigins、allowedMethods、allowedHeaders、exposedHeaders、allowCredentials、maxAge。

# 创建前置过滤器
前置过滤器对RequestContext中的数据进行设置，并提供给下游的过滤器使用，它的最主要的用途就是对路由过滤器所需的信息进行设置。
```Java
	import javax.servlet.http.HttpServletRequest;
	 
	import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
	 
	import com.netflix.zuul.ZuulFilter;
	import com.netflix.zuul.context.RequestContext;
	 
	public class QueryParamPreFilter extends ZuulFilter {
		@Override
		public int filterOrder() {
			return FilterConstants.PRE_DECORATION_FILTER_ORDER - 1; // run before PreDecoration
		}
	 
		@Override
		public String filterType() {
			return FilterConstants.PRE_TYPE;
		}
	 
		@Override
		public boolean shouldFilter() {
			RequestContext ctx = RequestContext.getCurrentContext();
			return !ctx.containsKey(FilterConstants.FORWARD_TO_KEY) // a filter has already forwarded
					&& !ctx.containsKey(FilterConstants.SERVICE_ID_KEY); // a filter has already determined serviceId
		}
	 
		@Override
		public Object run() {
			RequestContext ctx = RequestContext.getCurrentContext();
			HttpServletRequest request = ctx.getRequest();
			if (request.getParameter("sample") != null) {
				// put the serviceId in `RequestContext`
				ctx.put(FilterConstants.SERVICE_ID_KEY, request.getParameter("foo"));
			}
			return null;
		}
	}
```

# 创建路由过滤器
路由过滤器在前置过滤器之后运行，并负责构造需要发送给其他服务的请求。它在这的主要工作是：把请求数据传送至客户端所需要的 model 中，并把客户端 model 中的内容转换成响应数据返回。
```Java
	import java.io.InputStream;
	import java.util.Enumeration;
	import java.util.List;
	import java.util.Map;
	 
	import javax.servlet.http.HttpServletRequest;
	 
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.cloud.netflix.zuul.filters.ProxyRequestHelper;
	import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
	import org.springframework.util.LinkedMultiValueMap;
	import org.springframework.util.StreamUtils;
	 
	import com.netflix.zuul.ZuulFilter;
	import com.netflix.zuul.context.RequestContext;
	 
	import okhttp3.Headers;
	import okhttp3.MediaType;
	import okhttp3.OkHttpClient;
	import okhttp3.Request;
	import okhttp3.RequestBody;
	import okhttp3.Response;
	import okhttp3.internal.http.HttpMethod;
	 
	public class OkHttpRoutingFilter extends ZuulFilter {
		@Autowired
		private ProxyRequestHelper helper;
	 
		@Override
		public String filterType() {
			return FilterConstants.ROUTE_TYPE;
		}
	 
		@Override
		public int filterOrder() {
			return FilterConstants.SIMPLE_HOST_ROUTING_FILTER_ORDER - 1;
		}
	 
		@Override
		public boolean shouldFilter() {
			return RequestContext.getCurrentContext().getRouteHost() != null
					&& RequestContext.getCurrentContext().sendZuulResponse();
		}
	 
		@Override
		public Object run() {
			OkHttpClient httpClient = new OkHttpClient.Builder().build(); // customize
					
	 
			RequestContext context = RequestContext.getCurrentContext();
			HttpServletRequest request = context.getRequest();
	 
			String method = request.getMethod();
	 
			String uri = this.helper.buildZuulRequestURI(request);
	 
			Headers.Builder headers = new Headers.Builder();
			Enumeration<String> headerNames = request.getHeaderNames();
			while (headerNames.hasMoreElements()) {
				String name = headerNames.nextElement();
				Enumeration<String> values = request.getHeaders(name);
	 
				while (values.hasMoreElements()) {
					String value = values.nextElement();
					headers.add(name, value);
				}
			}
			try {
				InputStream inputStream = request.getInputStream();
	 
				RequestBody requestBody = null;
				if (inputStream != null && HttpMethod.permitsRequestBody(method)) {
					MediaType mediaType = null;
					if (headers.get("Content-Type") != null) {
						mediaType = MediaType.parse(headers.get("Content-Type"));
					}
					requestBody = RequestBody.create(mediaType, StreamUtils.copyToByteArray(inputStream));
				}
	 
				Request.Builder builder = new Request.Builder().headers(headers.build()).url(uri).method(method,
						requestBody);
	 
				Response response = httpClient.newCall(builder.build()).execute();
	 
				LinkedMultiValueMap<String, String> responseHeaders = new LinkedMultiValueMap<>();
	 
				for (Map.Entry<String, List<String>> entry : response.headers().toMultimap().entrySet()) {
					responseHeaders.put(entry.getKey(), entry.getValue());
				}
	 
				this.helper.setResponse(response.code(), response.body().byteStream(), responseHeaders);
				context.setRouteHost(null); // prevent SimpleHostRoutingFilter from running
			} catch (Exception e) {
				e.printStackTrace();
			}
			return null;
		}
	}
```

# 创建后置过滤器
后置过滤器通常是用来对响应进行处理，例如下面这个过滤器添加了一个UUID作为 X-Sample 的头内容。
```Java
	import java.util.UUID;
	 
	import javax.servlet.http.HttpServletResponse;
	 
	import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
	 
	import com.netflix.zuul.ZuulFilter;
	import com.netflix.zuul.context.RequestContext;
	 
	public class AddResponseHeaderFilter extends ZuulFilter {
		@Override
		public String filterType() {
			return FilterConstants.POST_TYPE;
		}
	 
		@Override
		public int filterOrder() {
			return FilterConstants.SEND_RESPONSE_FILTER_ORDER - 1;
		}
	 
		@Override
		public boolean shouldFilter() {
			return true;
		}
	 
		@Override
		public Object run() {
			RequestContext context = RequestContext.getCurrentContext();
			HttpServletResponse servletResponse = context.getResponse();
			servletResponse.addHeader("X-Sample", UUID.randomUUID().toString());
			return null;
		}
	}
```

# Zuul错误处理
在Zuul过滤器生命周期的任一阶段一旦出现异常，错误过滤器就会执行。SendErrorFilter 只会在RequestContext.getThrowable() 不是 null 的时候运行。它会在请求中设置一些特定的 javax.servlet.error.* 属性并将请求转向Springboot的错误页面。

# 应用上下文饿加载
Zuul 在内部使用的是Ribbon 对远程url进行调用。默认情况下，Ribbon客户端会在Spring Cloud第一次调用时进行懒加载。如果需要在应用启动时就进行加载，可以做如下配置。
```Yml
	zuul:
	  ribbon:
	    eager-load:
	      enabled: true
```

# 请求失败重试
Spring Cloud Netflix 提供了很多种方法用来创建 HTTP 请求，你可以使用负载均衡的RestTemplate、Ribbon 或者 Feign ，但无论你选择了哪一种，都有可能会出现请求失败的情况。当一个请求失败时，你可能会希望它自动地进行重试，要实现这一点的话，你需要把 Spring Retry 添加到你的classpath ，然后，负载均衡的RestTemplate、Ribbon 、 Feign 以及 Zuul 就会对所有失败的请求自动进行重试（前提是你的配置允许它这样做的话）。

## 补偿策略
默认情况下，重试请求是没有补偿策略的，如果你想要配置一个补偿策略，你需要为指定的服务创建一个 LoadBalancedRetryFactory 类型的Bean，并重写它的 createBackOffPolicy 方法。
```Java
	@Configuration
	public class MyConfiguration {
	    @Bean
	    LoadBalancedRetryFactory retryFactory() {
	        return new LoadBalancedRetryFactory() {
	            @Override
	            public BackOffPolicy createBackOffPolicy(String service) {
	                return new ExponentialBackOffPolicy();
	            }
	        };
	    }
	}
```

## 相关配置
在使用Ribbon进行重试时，你可以使用下面前三个个Ribbon配置项对重试的功能进行控制。
```Properties
	# Max number of retries on the same server (excluding the first try)
	sample-client.ribbon.MaxAutoRetries=1
	 
	# Max number of next servers to retry (excluding the first server)
	sample-client.ribbon.MaxAutoRetriesNextServer=1
	 
	# Whether all operations can be retried for this client
	sample-client.ribbon.OkToRetryOnAllOperations=true
	 
	# Interval to refresh the server list from the source
	sample-client.ribbon.ServerListRefreshInterval=2000
	 
	# Connect timeout used by Apache HttpClient
	sample-client.ribbon.ConnectTimeout=3000
	 
	# Read timeout used by Apache HttpClient
	sample-client.ribbon.ReadTimeout=3000
	 
	# Initial list of servers, can be changed via Archaius dynamic property at runtime
	sample-client.ribbon.listOfServers=www.microsoft.com:80,www.yahoo.com:80,www.google.com:80
```
另外，如果你想当返回的响应的状态码为某些值的时候才进行重试，你可以使用 clientName.ribbon.retryableStatusCodes 把那些需要Ribbon客户端进行重试的状态码列举出来。
```Yml	
	clientName:
	  ribbon:
	    retryableStatusCodes: 404,502
```
或者创建一个 LoadBalancedRetryPolicy 类型的Bean ，并实现它的retryableStatusCode方法。

## 禁用重试
```Properties
	# 全部禁用
	zuul.retryable=false
	# 禁用指定路由
	zuul.routes.routename.retryable=false
```

# 参考文章

<https://github.com/spring-projects/spring-retry>

<https://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#netflix-zuul-starter>