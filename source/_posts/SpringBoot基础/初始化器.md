---
title: SpringBoot基础之--初始化器
date: 2020-07-25 14:13:00
updated: 2020-07-25 14:13:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: 如何使用SpringBoot提供的初始化器？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img13.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img13.jpg
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
Spring 是一个扩展性很强的容器框架，为开发者提供了丰富的扩展入口，其中一个扩展点便是 `ApplicationContextInitializer` （应用上下文初始化器 ）。

ApplicationContextInitializer 是 Spring 在执行 `ConfigurableApplicationContext.refresh()` 方法对应用上下文进行刷新之前调用的一个回调接口，用来完成对 Spring 应用上下文个性化的初始化工作，该接口定义在 `org.springframework.context` 包中，其内部仅包含一个 `initialize()` 方法，其定义代码如下。 
```Java
	package org.springframework.context;
	 
	/**
	 * Callback interface for initializing a Spring {@link ConfigurableApplicationContext}
	 * prior to being {@linkplain ConfigurableApplicationContext#refresh() refreshed}.
	 */
	public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
	 
		/**
		 * Initialize the given application context.
		 * @param applicationContext the application to configure
		 */
		void initialize(C applicationContext);
	 
	}
```

# 自定义初始化器

在 Springboot 中使用自定义初始化器大致可以分为以下两个步骤：

- 自定义初始化器，一般是实现 ApplicationContextInitializer 接口。
- 注册初始化器。

## 自定义初始化器
第一步：自定义初始化器，此处为了测试初始化器的执行顺序定义了如下3个初始化器。 
```Java
	import org.springframework.context.ApplicationContextInitializer;
	import org.springframework.context.ConfigurableApplicationContext;
	import org.springframework.core.annotation.Order;
	 
	@Order(1)
	public class MyFirstApplicationContextInitializer
			implements ApplicationContextInitializer<ConfigurableApplicationContext> {
	 
		@Override
		public void initialize(ConfigurableApplicationContext applicationContext) {
			System.out.println("Application Display Name:" + applicationContext.getDisplayName());
		}
	 
	}
	import org.springframework.context.ApplicationContextInitializer;
	import org.springframework.context.ConfigurableApplicationContext;
	import org.springframework.core.annotation.Order;
	 
	@Order(2)
	public class MySecondApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
	 
		@Override
		public void initialize(ConfigurableApplicationContext applicationContext) {
			System.out.println("BeanDefinitionCount:" + applicationContext.getBeanFactory().getBeanDefinitionCount());
		}
	 
	}
	import org.springframework.context.ApplicationContextInitializer;
	import org.springframework.context.ConfigurableApplicationContext;
	import org.springframework.core.annotation.Order;
	 
	@Order(3)
	public class MyThirdApplicationContextInitializer
			implements ApplicationContextInitializer<ConfigurableApplicationContext> {
	 
		@Override
		public void initialize(ConfigurableApplicationContext applicationContext) {
			System.out.println("BeanDefinitionNames" + applicationContext.getBeanDefinitionNames());
		}
	 
	}
```

## 注册初始化器
第二步：注册初始化器，有以下三种方式。

### 方式一
在启动类中，使用 `SpringApplication.addInitializers()` 方法注册。  
```Java
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.context.ConfigurableApplicationContext;
	 
	import com.pengjunlee.initializer.MyFirstApplicationContextInitializer;
	 
	@SpringBootApplication
	public class MyApplication {
		public static void main(String[] args) {
			SpringApplication springApplication = new SpringApplication(MySpringBootApplication.class);
			springApplication.addInitializers(new MyFirstApplicationContextInitializer());
			ConfigurableApplicationContext context = springApplication.run(args);
			context.close();
		}
	}
```

### 方式二
在 Springboot 核心配置文件 `application.properties` 中增加 `context.initializer.classes = [ 初始化器全类名 ]` 进行注册。  

`context.initializer.classes=com.pengjunlee.initializer.MySecondApplicationContextInitializer`

该 `context.initializer.classes` 配置项名称在 `DelegatingApplicationContextInitializer` 类的属性常量中指定。

### 方式三
通过在`CLASSPATH/META-INF/spring.factories`中添加 `org.springframework.context.ApplicationContextInitializer` 配置项进行注册。 

	# Application Context Initializers
	org.springframework.context.ApplicationContextInitializer=com.pengjunlee.initializer.MyThirdApplicationContextInitializer

启动应用，程序打印结果如下：  
```
	BeanDefinitionCount:6
	Application Display Name:org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@3439f68d
	BeanDefinitionNames[Ljava.lang.String;@55a561cf
```
> <font color=red>注意</font>：虽然可以使用 `@Order` 注解来控制多个初始化器的执行顺序（数值越小越先执行），但是，通过不同方式注册的初始化器的执行顺序也有所不同，若多个初始化器注册的方式不同会导致 @Order 注解顺序无效，从以上程序执行后的打印结果来看，三种方式注册的初始化器的执行顺序依次是：`方式二 --> 方式一 --> 方式三`。  

# Springboot定义的初始化器

Springboot定义的 ApplicationContextInitializer 接口的实现类有下面几个，如图所示。  

<div align=center>

![初始化器示意图](http://pengjunlee.3vzhuji.net/static/springboot/23.png "初始化器示意图")
<div align=left>

## DelegatingApplicationContextInitializer
`DelegatingApplicationContextInitializer` 初始化器负责读取核心配置文件 `context.initializer.classes` 配置项指定的初始化器，并调用它们的 initialize() 方法来完成对`应用上下文的初始化工作`。  
```Java
	public class DelegatingApplicationContextInitializer
			implements ApplicationContextInitializer<ConfigurableApplicationContext>, Ordered {
	 
		private static final String PROPERTY_NAME = "context.initializer.classes";
	 
		private int order = 0;
	 
		/**
		 * 对应用上下文进行初始化
		 */
		@Override
		public void initialize(ConfigurableApplicationContext context) {
			ConfigurableEnvironment environment = context.getEnvironment();
			// 获取核心配置文件中指定的初始化器类
			List<Class<?>> initializerClasses = getInitializerClasses(environment);
			if (!initializerClasses.isEmpty()) {
				// 利用获取到的初始化器类对应用上下文进行初始化
				applyInitializerClasses(context, initializerClasses);
			}
		}
	 
		/**
		 * 读取核心配置文件中 context.initializer.classes 指定的初始化器类
		 */
		private List<Class<?>> getInitializerClasses(ConfigurableEnvironment env) {
			String classNames = env.getProperty(PROPERTY_NAME);
			List<Class<?>> classes = new ArrayList<Class<?>>();
			if (StringUtils.hasLength(classNames)) {
				for (String className : StringUtils.tokenizeToStringArray(classNames, ",")) {
					classes.add(getInitializerClass(className));
				}
			}
			return classes;
	 
		}
	 
		/**
		 * 使用指定的初始化器类对应用上下文进行初始化
		 */
		private void applyInitializerClasses(ConfigurableApplicationContext context, List<Class<?>> initializerClasses) {
			Class<?> contextClass = context.getClass();
			List<ApplicationContextInitializer<?>> initializers = new ArrayList<ApplicationContextInitializer<?>>();
			for (Class<?> initializerClass : initializerClasses) {
				initializers.add(instantiateInitializer(contextClass, initializerClass));
			}
			applyInitializers(context, initializers);
		}
		
		/**
		 * 使用指定的初始化器对应用上下文进行初始化
		 */
		@SuppressWarnings({ "unchecked", "rawtypes" })
		private void applyInitializers(ConfigurableApplicationContext context,
				List<ApplicationContextInitializer<?>> initializers) {
			// 对初始化器进行 Order 排序
			Collections.sort(initializers, new AnnotationAwareOrderComparator());
			for (ApplicationContextInitializer initializer : initializers) {
				initializer.initialize(context);
			}
		}
		// 此处省略其它内容
	 
	}
```

## ContextIdApplicationContextInitializer
`ContextIdApplicationContextInitializer` 初始化器的作用是给应用上下文设置一个ID，这个ID由 `name` 和 `index` 两个部分组成。

其中 `name` 会依次从读取核心配置的以下三个属性，如果存在则立即使用，若不存在则继续查找下一个，都不存在则取默认值 “application”。 

- spring.application.name
- vcap.application.name
- spring.config.name

`index` 会依次从核心配置的以下几个属性获取，如果存在则立即使用，若不存在则继续查找下一个，都不存在则为空。

- vcap.application.instance_index
- spring.application.index
- server.port
- PORT

所以，ID的格式应为 `[application.name][:application.index]` ，例如，application:8080 。 
```Java
	public class ContextIdApplicationContextInitializer implements
			ApplicationContextInitializer<ConfigurableApplicationContext>, Ordered {
	 
		private static final String NAME_PATTERN = "${spring.application.name:${vcap.application.name:${spring.config.name:application}}}";
	 
		private static final String INDEX_PATTERN = "${vcap.application.instance_index:${spring.application.index:${server.port:${PORT:null}}}}";
	 
		private final String name;
	 
		private int order = Ordered.LOWEST_PRECEDENCE - 10;
	 
		// 为应用上下文设置ID
		@Override
		public void initialize(ConfigurableApplicationContext applicationContext) {
			applicationContext.setId(getApplicationId(applicationContext.getEnvironment()));
		}
	 
		private String getApplicationId(ConfigurableEnvironment environment) {
			String name = environment.resolvePlaceholders(this.name);
			String index = environment.resolvePlaceholders(INDEX_PATTERN);
			String profiles = StringUtils
					.arrayToCommaDelimitedString(environment.getActiveProfiles());
			if (StringUtils.hasText(profiles)) {
				name = name + ":" + profiles;
			}
			if (!"null".equals(index)) {
				name = name + ":" + index;
			}
			return name;
		}
		// 此处省略其它内容
	 
	}
```
> **注意**：`ContextIdApplicationContextInitializer` 初始化器的 `Order` 为 `Ordered.LOWEST_PRECEDENCE - 10` ，若要在自定义初始化器中通过 `ConfigurableApplicationContext.getId()` 方法获取上下文ID值，自定义初始化器的 `Order` 需要`大于Ordered.LOWEST_PRECEDENCE - 10` 。  

## ConfigurationWarningsApplicationContextInitializer
`ConfigurationWarningsApplicationContextInitializer` 初始化器用来对常见的由于配置错误而引起的警告进行打印报告。 
```Java
	public class ConfigurationWarningsApplicationContextInitializer
			implements ApplicationContextInitializer<ConfigurableApplicationContext> {
	 
		private static final Log logger = LogFactory
				.getLog(ConfigurationWarningsApplicationContextInitializer.class);
	 
		@Override
		public void initialize(ConfigurableApplicationContext context) {
			// 添加一个 ConfigurationWarningsPostProcessor 用来打印警告信息
			context.addBeanFactoryPostProcessor(
					new ConfigurationWarningsPostProcessor(getChecks()));
		}
	 
		// 此处省略其它内容
	}
```
其中的 `ConfigurationWarningsPostProcessor` 是一个内部类，用来打印注册 BeanDefinition 过程中产生的配置错误警告信息。   
```Java
	protected final static class ConfigurationWarningsPostProcessor
				implements PriorityOrdered, BeanDefinitionRegistryPostProcessor {
	 
		private Check[] checks;
	 
		public ConfigurationWarningsPostProcessor(Check[] checks) {
			this.checks = checks;
		}
	 
		@Override
		public int getOrder() {
			return Ordered.LOWEST_PRECEDENCE - 1;
		}
	 
		@Override
		public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)
				throws BeansException {
			for (Check check : this.checks) {
				String message = check.getWarning(registry);
				if (StringUtils.hasLength(message)) {
					warn(message);
				}
			}
	 
		}
	 
		private void warn(String message) {
			if (logger.isWarnEnabled()) {
				logger.warn(String.format("%n%n** WARNING ** : %s%n%n", message));
			}
		}
	 
	}
```

## ServerPortInfoApplicationContextInitializer
`ServerPortInfoApplicationContextInitializer` 初始化器通过监听 `EmbeddedServletContainerInitializedEvent` 事件，来对内部服务器实际要监听的端口号进行属性设置。 
```Java
	public class ServerPortInfoApplicationContextInitializer
			implements ApplicationContextInitializer<ConfigurableApplicationContext> {
	 
		@Override
		public void initialize(ConfigurableApplicationContext applicationContext) {
			applicationContext.addApplicationListener(
					new ApplicationListener<EmbeddedServletContainerInitializedEvent>() {
	 
						@Override
						public void onApplicationEvent(
								EmbeddedServletContainerInitializedEvent event) {
							ServerPortInfoApplicationContextInitializer.this
									.onApplicationEvent(event);
						}
	 
					});
		}
	 
		protected void onApplicationEvent(EmbeddedServletContainerInitializedEvent event) {
			String propertyName = getPropertyName(event.getApplicationContext());
			setPortProperty(event.getApplicationContext(), propertyName,
					event.getEmbeddedServletContainer().getPort());
		}
	 
		protected String getPropertyName(EmbeddedWebApplicationContext context) {
			String name = context.getNamespace();
			if (StringUtils.isEmpty(name)) {
				name = "server";
			}
			return "local." + name + ".port";
		}
	 
		private void setPortProperty(ApplicationContext context, String propertyName,
				int port) {
			if (context instanceof ConfigurableApplicationContext) {
				setPortProperty(((ConfigurableApplicationContext) context).getEnvironment(),
						propertyName, port);
			}
			if (context.getParent() != null) {
				setPortProperty(context.getParent(), propertyName, port);
			}
		}
	 
		@SuppressWarnings("unchecked")
		private void setPortProperty(ConfigurableEnvironment environment, String propertyName,
				int port) {
			MutablePropertySources sources = environment.getPropertySources();
			PropertySource<?> source = sources.get("server.ports");
			if (source == null) {
				source = new MapPropertySource("server.ports", new HashMap<String, Object>());
				sources.addFirst(source);
			}
			((Map<String, Object>) source.getSource()).put(propertyName, port);
		}
	 
	}
```

## SharedMetadataReaderFactoryContextInitializer
`SharedMetadataReaderFactoryContextInitializer` 初始化器用来创建一个可以在 `ConfigurationClassPostProcessor` 和Spring Boot 之间共享的`CachingMetadataReaderFactory`。 
```Java
	@Override
	public void initialize(ConfigurableApplicationContext applicationContext) {
		applicationContext.addBeanFactoryPostProcessor(new CachingMetadataReaderFactoryPostProcessor());
	}
```

## AutoConfigurationReportLoggingInitializer
`AutoConfigurationReportLoggingInitializer` 初始化器用来将 `ConditionEvaluationReport` 记录的条件评估详情输出到日志，默认使用 `DEBUG` 级别，当有异常问题发生时会使用 `INFO` 级别。 
```Java
	@Override
	public void initialize(ConfigurableApplicationContext applicationContext) {
		this.applicationContext = applicationContext;
		applicationContext.addApplicationListener(new AutoConfigurationReportListener());
		if (applicationContext instanceof GenericApplicationContext) {
			// Get the report early in case the context fails to load
			this.report = ConditionEvaluationReport
					.get(this.applicationContext.getBeanFactory());
		}
	}
```
各个初始化器的执行顺序如下：
`DelegatingApplicationContextInitializer` --> `ContextIdApplicationContextInitializer` --> `ConfigurationWarningsApplicationContextInitializer` -->`ServerPortInfoApplicationContextInitializer`-->`SharedMetadataReaderFactoryContextInitializer`--> `AutoConfigurationReportLoggingInitializer`

本文项目源码已上传至CSDN，资源地址：<https://download.csdn.net/download/pengjunlee/10335994> 