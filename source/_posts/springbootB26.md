---
title: SpringBoot基础之--事件监听
date: 2020-07-25 14:26:00
updated: 2020-07-25 14:26:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何监听事件？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img26.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img26.jpg
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
Springboot 事件监听为 Bean 与 Bean 之间的消息通信提供支持：`当一个 Bean 做完一件事以后，通知另一个 Bean 知晓并做出相应处理，此时，需要后续 Bean 监听当前 Bean 所发生的事件`。  

# 自定义事件监听

在 Springboot 中实现自定义事件监听大致可以分为以下四个步骤：

- 自定义事件，一般是继承 ApplicationEvent 抽象类；
- 定义事件监听器，一般是实现 ApplicationListener 接口；
- 注册监听器；
- 发布事件。

## 第一步
自定义事件，一般是继承 `ApplicationEvent` 抽象类。 
```Java
	import org.springframework.context.ApplicationEvent;
	 
	/**
	 * 自定义事件，继承 ApplicationEvent
	 * @author pjli
	 */
	public class MyApplicationEvent extends ApplicationEvent {
	 
		private static final long serialVersionUID = 1L;
	 
		public MyApplicationEvent(Object source) {
			super(source);
			System.out.println("触发 MyApplicationEvent 事件...");
		}
	 
	}
```

## 第二步
定义事件监听器，一般是实现 `ApplicationListener` 接口。  
```Java
	import org.springframework.context.ApplicationListener;
	import org.springframework.stereotype.Component;
	 
	/**
	 * 事件监听器，实现 ApplicationListener 接口
	 * @author pjli
	 */
	 
	@Component
	public class MyApplicationListener implements ApplicationListener<MyApplicationEvent>{
	 
		
		@Override
		public void onApplicationEvent(MyApplicationEvent event) {
			System.out.println("监听到："+event.getClass().getName()+"事件...");
		}
	 
	}
```

## 第三步
注册监听器，有以下五种方式。

### 方式一
在启动类中，使用 `ConfigurableApplicationContext.addApplicationListener()` 方法注册。 
```Java
	ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
	// 注册 MyApplicationListener 事件监听器
	context.addApplicationListener(new MyApplicationListener());
```

### 方式二
使用 `@Component` 等注解将事件监听器纳入到 Spring 容器中管理。  
```Java
	@Component
	public class MyApplicationListener implements ApplicationListener<MyApplicationEvent>{ //内容同上，此处省略 }
```

### 方式三
在 Springboot 核心配置文件 application.properties 中增加 `context.listener.classes = [ 监听器全类名 ]`。 
```Properties
context.listener.classes=com.pengjunlee.listener.MyApplicationListener
```
该 `context.listener.classes` 配置项在 `DelegatingApplicationListener` 类的属性常量中指定。

### 方式四
使用 `@EventListener` 注解自动生成并注册事件监听器。 
```Java
	import org.springframework.context.event.EventListener;
	import org.springframework.stereotype.Component;
	 
	@Component
	public class MyEventHandler {
	 
		@EventListener
		public void listenMyApplicationEvent(MyApplicationEvent event) {
			System.out.println("监听到：" + event.getClass().getName() + "事件...");
		}
	 
	}
```

### 方式五
通过在 `CLASSPATH/META-INF/spring.factories` 中添加 `org.springframework.context.ApplicationListener = [ 监听器全类名 ]` 配置项进行注册.  
```Properties
	# Application Listeners
	org.springframework.context.ApplicationListener=com.pengjunlee.listener.MyApplicationListener
```
## 第四步
发布事件,可以在程序启动类中使用 ApplicationContext.publishEvent() 发布事件，完整代码如下。  
```Java
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.context.ConfigurableApplicationContext;
	 
	import com.pengjunlee.listener.MyApplicationEvent;
	import com.pengjunlee.listener.MyApplicationListener;
	 
	@SpringBootApplication
	public class MyApplication {
		public static void main(String[] args) {
			ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
			// 注册 MyApplicationListener 事件监听器
			context.addApplicationListener(new MyApplicationListener());
			// 发布 MyApplicationEvent 事件
			context.publishEvent(new MyApplicationEvent(new Object()));
			// context.getBean(MyEventHandler.class).publishMyApplicationEvent();
			context.close();
		}
	}
```
> **注意**：可以通过以下代码将 ApplicationContext 注入到 Bean 中用来发布事件。  
```Java
	@Autowired
	ApplicationContext applicationContext;
```

# Springboot 启动事件监听

除了可以对自定义的事件进行监听，我们还能够监听 Springboot 的启动事件，Springboot 共定义了以下5 种启动事件类型：

- ApplicationStartingEvent：应用启动事件，在调用 SpringApplication.run() 方法之前，可以从中获取到 SpringApplication 对象，进行一些启动前设置。
- ApplicationEnvironmentPreparedEvent：Environment准备完成事件，此时可以从中获取到 Environment 对象并对其中的配置项进行查看或者修改。
- ApplicationPreparedEvent：ApplicationContext准备完成事件，接下来 Spring 就能够向容器中加载 Bean 了 。
- ApplicationReadyEvent：应用准备完成事件，预示着应用可以接收和处理请求了。
- ApplicationFailedEvent：应用启动失败事件，可以从中捕获到启动失败的异常信息进行相应处理，例如：添加虚拟机对应的钩子进行资源的回收与释放。

为了验证各个启动事件发生的前后，编写一个测试类进行测试，测试代码如下。 
```Java
	import org.springframework.boot.Banner;
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.boot.context.event.ApplicationEnvironmentPreparedEvent;
	import org.springframework.boot.context.event.ApplicationFailedEvent;
	import org.springframework.boot.context.event.ApplicationPreparedEvent;
	import org.springframework.boot.context.event.ApplicationReadyEvent;
	import org.springframework.boot.context.event.ApplicationStartingEvent;
	import org.springframework.context.ApplicationListener;
	import org.springframework.context.ConfigurableApplicationContext;
	import org.springframework.core.env.ConfigurableEnvironment;
	 
	/**
	 * Springboot 启动事件测试
	 * @author pjli
	 */
	@SpringBootApplication
	public class MySpringBootApplication {
		public static void main(String[] args) {
			SpringApplication springApplication = new SpringApplication(MySpringBootApplication.class);
			// 添加 ApplicationStartingEvent 事件监听器
			springApplication.addListeners((ApplicationListener<ApplicationStartingEvent>) event -> {
				System.out.println("触发 ApplicationStartingEvent 事件...");
				SpringApplication application = event.getSpringApplication();
				application.setBannerMode(Banner.Mode.OFF);
			});
			// 添加 ApplicationEnvironmentPreparedEvent 事件监听器
			springApplication.addListeners((ApplicationListener<ApplicationEnvironmentPreparedEvent>) event -> {
				System.out.println("触发 ApplicationEnvironmentPreparedEvent 事件...");
				ConfigurableEnvironment environment = event.getEnvironment();
				System.out.println("event.name=" + environment.getProperty("event.name"));
			});
			// 添加 ApplicationPreparedEvent 事件监听器
			springApplication.addListeners((ApplicationListener<ApplicationPreparedEvent>) event -> {
				System.out.println("触发 ApplicationPreparedEvent 事件...");
				ConfigurableApplicationContext context = event.getApplicationContext();
				System.out.println("Application context name:" + context.getDisplayName());
			});
			// 添加 ApplicationReadyEvent 事件监听器
			springApplication.addListeners((ApplicationListener<ApplicationReadyEvent>) event -> {
				System.out.println("触发 ApplicationReadyEvent 事件...");
			});
			// 添加 ApplicationFailedEvent 事件监听器
			springApplication.addListeners((ApplicationListener<ApplicationFailedEvent>) event -> {
				System.out.println("触发 ApplicationFailedEvent 事件...");
				Throwable t = event.getException();
				System.out.println("启动失败：" + t.getMessage());
			});
			ConfigurableApplicationContext context = springApplication.run(args);
	 
			context.close();
		}
	}
```
当应用正常启动时，打印内容如下：  
```
	触发 ApplicationStartingEvent 事件...
	触发 ApplicationEnvironmentPreparedEvent 事件...
	event.name=pengjunlee
	触发 ApplicationPreparedEvent 事件...
	Application context name:org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@49e202ad
	触发 ApplicationReadyEvent 事件...
```
当应用重复启动两次，由于端口被占用而启动失败时，打印内容如下：  
```
	触发 ApplicationStartingEvent 事件...
	触发 ApplicationEnvironmentPreparedEvent 事件...
	event.name=pengjunlee
	触发 ApplicationPreparedEvent 事件...
	Application context name:org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@49e202ad
	触发 ApplicationFailedEvent 事件...
	启动失败：Connector configured to listen on port 8080 failed to start
```
由此可以看出 Springboot 启动事件的发生顺序是：`Starting` -->`EnvironmentPrepared` --> `Prepared`-->`Ready/Failed` 。

本文项目源码已上传至CSDN，资源地址：<https://download.csdn.net/download/pengjunlee/10332845> 