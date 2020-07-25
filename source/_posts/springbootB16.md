---
title: SpringBoot基础之--配置文件
date: 2020-07-25 14:16:00
updated: 2020-07-25 14:16:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何使用配置文件指定配置？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
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
# 核心配置文件

Spring-Boot 核心配置文件的默认位置是在`classpath根目录`或`classpath/config目录`下，文件名为 `application.properties`或`application.yml` ，如果两个文件同时存在，则都会被加载。

其中，application.properties采用键值对 [ property-name= property-vale] 格式进行配置，如下所示。
```Properties
server.port=9080
```

application.yml采用缩进格式（勿使用 Tab 键缩进）进行配置，如下所示。  
```Yml
	server:
	  port: 9080
```
如果你不想使用 Spring-Boot 默认的核心配置文件，可以通过修改启动参数来进行指定，有以下两种情况：

- 新配置文件在`classpath`根目录或`classpath/config`目录下，通过在程序启动参数Program arguments中添加 `--spring.config.name=[新配置文件名]`进行指定，只需要指定新配置文件的名称，无需指定扩展名。例如：  
 `--spring.config.name=jdbc <!-- 核心配置文件名可以是 jdbc.properties 或 jdbc.yml ，两者均可 -->`
- 新配置文件不在`classpath`根目录或`classpath/config`目录下，通过在程序启动参数Program arguments 中添加 `--spring.config.location=[新配置文件全路径]` 进行指定，此时的新配置文件可以从classpath 或本地的文件系统中指定，全路径需包含扩展名。  
```Properties
<!-- 从 classpath 中读取配置文件 -->
--spring.config.location=classpath:conf/application.yml
<!-- 从本地文件系统中读取配置文件 -->
--spring.config.location=file:/d:/src/app.properties
<!-- 可以同时指定多个配置文件，多个文件之间使用 “,”进行分隔 -->
--spring.config.location=classpath:conf/application.yml,file:/d:/src/jdbc.properties
```

# 如何读取配置

在 Spring-Boot 中读取配置文件非常容易，常用的有以下两种方式：

- 通过@Value注解获取配置
- 通过 Environment对象获取配置

接下来，我们以读取 application.properties 配置文件为例，配置文件的内容如下。 
```Properties
	server.ip=192.168.80.47
	server.port=9080
	server.name=pengjunlee
	# Use ${property-name} As Placeholder
	server.welcome=Hello ${server.name}
	#server.password=
```
读取配置的示例代码如下。  
```Java
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.core.env.Environment;
	import org.springframework.stereotype.Component;
	 
	@Component
	public class PropertyTestOne {
	 
		@Autowired
		private Environment env;
		
		// 读取字符串
		@Value("${server.ip}")
		private String ip;
		
		// 读取整型
		@Value("${server.port}")
		private Integer port;
		
		@Value("${server.name}")
		private String name;
		
		@Value("${server.welcome}")
		private String welcome;
		
		// 属性不存在时(不包括值为空)，设置默认值
		@Value("${server.password:123456}")
		private String password;
	 
		public void show() {
	 
			System.out.println("----------通过 @Value 注解获取配置------------");
			System.out.println("server.ip= "+ip);
			System.out.println("server.port= "+port);
			System.out.println("server.name= "+name);
			System.out.println("server.welcome= "+welcome);
			System.out.println("server.password= "+password);
			System.out.println("----------通过 Environment 对象获取配置------------");
			System.out.println("server.ip= "+env.getProperty("server.ip"));
			System.out.println("server.port= "+env.getProperty("server.port", Integer.class));
			System.out.println("server.name= "+env.getProperty("server.name"));
			System.out.println("server.welcome= "+env.getProperty("server.welcome"));
			System.out.println("server.password= "+env.getProperty("server.password","123456"));
			
		}
	}
```
应用启动类代码如下。  
```Java
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.context.ConfigurableApplicationContext;
	 
	@SpringBootApplication
	public class MyApplication {
	 
		public static void main(String[] args) {
			
			ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
			// 通过 context.getEnvironment().getProperty() 获取配置
			System.out.println(context.getEnvironment().getProperty("server.ip"));
			context.getBean(PropertyTestOne.class).show();
			context.close();
		}
	}
```
启动程序，打印结果如下： 
```
	192.168.80.47
	----------通过 @Value 注解获取配置------------
	server.ip= 192.168.80.47
	server.port= 9080
	server.name= pengjunlee
	server.welcome= Hello pengjunlee
	server.password= 123456
	----------通过 Environment 对象获取配置------------
	server.ip= 192.168.80.47
	server.port= 9080
	server.name= pengjunlee
	server.welcome= Hello pengjunlee
	server.password= 123456
```
> **提示**：除了通过以上两种方式为配置赋默认值以外，还可以通过 `SpringApplication.setDefaultProperties()` 方法为配置赋默认值。  
```Java
	SpringApplication springApplication = new SpringApplication(MyApplication.class);
	Map<String, Object> map = new HashMap<String, Object>();
	springApplication.setDefaultProperties(map);
```

# 加载指定配置文件

除了可以使用Spring-Boot  的核心配置文件之外，我们还可以利用`@PropertySource` 和`@PropertySources`两个注解实现让程序加载classpath和本地文件系统中任意指定的一个或者多个配置文件。 
```Java
	import org.springframework.context.annotation.Configuration;
	import org.springframework.context.annotation.PropertySource;
	import org.springframework.context.annotation.PropertySources;
	 
	@Configuration
	//@PropertySource("classpath:email/email.properties")
	//@PropertySource("file:/d:/src/jdbc.properties")
	@PropertySources({@PropertySource("classpath:email/email.properties"),@PropertySource("file:/d:/src/jdbc.properties")})
	public class PropertyFileConfig {
	 
	}
```
另外，我们也可以利用 `@ConfigurationProperties` 注解从指定的配置文件中获取带有指定前缀的配置并赋值给JavaBean 对象中同名的属性，若未指定配置文件位置，则会从已经加载的所有配置文件中查找并获取配置。
```Java
	import java.util.ArrayList;
	import java.util.Arrays;
	import java.util.List;
	 
	import org.springframework.boot.context.properties.ConfigurationProperties;
	import org.springframework.stereotype.Component;
	 
	@Component
	@ConfigurationProperties(prefix="jdbc",locations="classpath:mysql/jdbc.properties")
	public class JdbcProperties {
	 
		private String driver;
		
		private String url;
		
		private String username;
		
		private String password;
		
		//private List<String> hosts=new ArrayList<String>();
		
		private String[] hosts;
		
		public void show(){
			System.out.println("jdbc.driver= "+driver);
			System.out.println("jdbc.url= "+url);
			System.out.println("jdbc.username= "+username);
			System.out.println("jdbc.password= "+password);
			System.out.println("jdbc.host="+Arrays.asList(hosts));
		}
	 
		public String getDriver() {
			return driver;
		}
	 
		public void setDriver(String driver) {
			this.driver = driver;
		}
	 
		public String getUrl() {
			return url;
		}
	 
		public void setUrl(String url) {
			this.url = url;
		}
	 
		public String getUsername() {
			return username;
		}
	 
		public void setUsername(String username) {
			this.username = username;
		}
	 
		public String getPassword() {
			return password;
		}
	 
		public void setPassword(String password) {
			this.password = password;
		}
	 
		public String[] getHosts() {
			return hosts;
		}
	 
		public void setHosts(String[] hosts) {
			this.hosts = hosts;
		}
	 
		/*public List<String> getHosts() {
			return hosts;
		}
		public void setHosts(List<String> hosts) {
			this.hosts = hosts;
		}*/
		
		
	}
```
> **注意**：使用此种方式获取配置，需要在 JavaBean 中为配置相应的属性生成 get() 和 set() 方法。

# 动态加载配置

在很多情况下，我们经常会需要为不同的环境启用不同的配置文件，例如：开发环境下就使用 application-dev.properties 中的配置；测试环境下就使用application-test.properties 中的配置；生产环境下就使用application-prod.properties 中的配置，在 Spring-Boot中实现这一功能需要借助Profile 机制来完成。

在 Spring-Boot 中使用 Profile 来动态地加载配置文件，有以下几种实现途径：

1. 在程序启动类中，调用 SpringApplication.setAdditionalProfiles() 方法激活指定的 Profile。 
```Java
		SpringApplication application=new SpringApplication(MyApplication.class);
		// 指定需要激活的 profile
		application.setAdditionalProfiles("default","test");
		ConfigurableApplicationContext context = application.run(args);
```
2. 在程序启动参数 Program arguments 中添加 `--spring.profiles.active=[profile1,profile2...profileN ]` 激活指定的Profile。  
```
--spring.profiles.active=default,test`
```
3. 在核心配置文件 application.properties 中添加 `spring.profiles.active=[ profile1,profile2...profileN]` 激活指定的Profile。 
```Properties
spring.profiles.active=default,test`
```
在 Profile 的帮助下，我们还可以利用@Profile()注解来实现容器中Bean 的动态创建，只在相应的Profile 被激活时才创建Bean。  
```Java
	import org.springframework.boot.SpringBootConfiguration;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Profile;
	 
	@SpringBootConfiguration
	@Profile("test")
	public class ProfileConfig {
	 
		@Bean
		//@Profile("test")
		public Runnable createRunnable1(){
			System.out.println("--------test------");
			return ()->{};
		}
		
	}
```

# 配置扩展接口

为了能够更好的对配置文件进行扩展，Spring-Boot还额外提供了一个`EnvironmentPostProcessor`接口，其内部只包含一个`postProcessEnvironment(ConfigurableEnvironment env, SpringApplication app)`方法，通过实现该接口，我们可以在方法中实现很多丰富的功能，例如：根据条件加载本地或远程的配置文件、对已加载的属性进行管理，等等。

以使用该接口加载一个本地的 local.properties 配置文件为例，需要以下两个步骤：

## 编写接口的实现类。
```Java
	import java.io.FileInputStream;
	import java.io.InputStream;
	import java.util.Properties;
	 
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.env.EnvironmentPostProcessor;
	import org.springframework.core.env.ConfigurableEnvironment;
	import org.springframework.core.env.PropertiesPropertySource;
	import org.springframework.stereotype.Component;
	 
	@Component
	public class MyEnvironmentPostProcessor implements EnvironmentPostProcessor {
	 
		@Override
		public void postProcessEnvironment(ConfigurableEnvironment env, SpringApplication app) {
			try{
				InputStream in=new FileInputStream("d:/src/local.properties");
				Properties properties=new Properties();
				properties.load(in);
				
				PropertiesPropertySource pps=new PropertiesPropertySource("local",properties);
				env.getPropertySources().addLast(pps);
			}
			catch(Exception e)
			{
				
			}
		}
	 
	}
```

## 将编写好的接口实现类注册到 `classpath/META-INF` 目录下的 `spring.factories`文件中。  
```Properties
	org.springframework.boot.env.EnvironmentPostProcessor=com.pengjunlee.MyEnvironmentPostProcessor
```
本文项目源码已上传至CSDN，资源地址：<https://download.csdn.net/download/pengjunlee/10301077>