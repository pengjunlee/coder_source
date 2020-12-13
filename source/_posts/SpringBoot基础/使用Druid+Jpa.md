---
title: SpringBoot基础之--使用Druid+Jpa
date: 2020-07-25 14:21:00
updated: 2020-07-25 14:21:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何使用Druid+Jpa？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img21.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img21.jpg
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
# Druid 简介

Druid是阿里巴巴开源的数据库连接池，号称是Java语言中最好的数据库连接池，能够提供强大的监控和扩展功能。GitHub地址：<https://github.com/alibaba/druid>。

**Druid有以下优点**：

- 可以监控数据库访问性能，Druid内置提供了一个功能强大的StatFilter插件，能够详细统计SQL的执行性能，这对于线上分析数据库访问性能有帮助。 
- 替换DBCP和C3P0，Druid提供了一个高效、功能强大、可扩展性好的数据库连接池。 
- 数据库密码加密。直接把数据库密码写在配置文件中，这是不好的行为，容易导致安全问题。DruidDriver和DruidDataSource都支持PasswordCallback。 
- SQL执行日志，Druid提供了不同的LogFilter，能够支持Common-Logging、Log4j和JdkLog，你可以按需要选择相应的LogFilter，监控你应用的数据库访问情况。 
- 扩展JDBC，如果你要对JDBC层有编程的需求，可以通过Druid提供的Filter-Chain机制，很方便编写JDBC层的扩展插件。 

本文将对如何在Springboot中使用Druid数据库连接池进行简单示例和介绍，为简单起见，本文使用了Spring Jpa来进行数据库操作，项目的完整目录层次如下图所示。

<div align=center>

![Druid示意图](http://pengjunlee.3vzhuji.net/static/springboot/38.png "Druid示意图")
<div align=left>

# 添加依赖与配置

为了使用Druid和Spring Data JPA，需要在工程POM文件中引入它们的Maven依赖。 
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.1.RELEASE</version>
	</parent>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- 添加MySQL依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- 添加JPA依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<!-- 添加druid依赖 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>1.1.1</version>
		</dependency>
	</dependencies>
```
在`application.yml`核心配置文件中除了要定义MYSQL数据库连接信息外，还需要添加如下JPA相关配置。 
```Properties
	spring:
	  datasource:
	      url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
	      username: root
	      password: 123456
	      driver-class-name: com.mysql.jdbc.Driver
	      platform: mysql
	      type: com.alibaba.druid.pool.DruidDataSource
	      # 下面为连接池的补充设置，应用到上面所有数据源中
	      # 初始化大小，最小，最大
	      initialSize: 1
	      minIdle: 3
	      maxActive: 20
	      # 配置获取连接等待超时的时间
	      maxWait: 60000
	      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
	      timeBetweenEvictionRunsMillis: 60000
	      # 配置一个连接在池中最小生存的时间，单位是毫秒
	      minEvictableIdleTimeMillis: 30000
	      validationQuery: select 'x'
	      testWhileIdle: true
	      testOnBorrow: false
	      testOnReturn: false
	      # 打开PSCache，并且指定每个连接上PSCache的大小
	      poolPreparedStatements: true
	      maxPoolPreparedStatementPerConnectionSize: 20
	      # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
	      filters: stat,wall,log4j
	      # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
	      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
	      # 合并多个DruidDataSource的监控数据
	      #useGlobalDataSourceStat: true
	 
	  jpa:
	    # 配置 DBMS 类型
	    database: MYSQL
	    # 配置是否将执行的 SQL 输出到日志
	    show-sql: true
	    properties:
	      hibernate:
	        hbm2ddl:
	          # 配置开启自动更新表结构
	          auto: update
```

# 配置数据源

在Spring中使用Druid数据源非常简单方便，只需要创建一个DruidDataSource类型的数据源并将其纳入到Spring容器中进行管理即可。 
```Java
	@ConfigurationProperties(prefix = "spring.datasource")
	public class DruidDataSourceConfig {
		private String url;
		private String username;
		private String password;
		private String driverClassName;
		private int initialSize;
		private int minIdle;
		private int maxActive;
		private int maxWait;
		private int timeBetweenEvictionRunsMillis;
		private int minEvictableIdleTimeMillis;
		private String validationQuery;
		private boolean testWhileIdle;
		private boolean testOnBorrow;
		private boolean testOnReturn;
		private boolean poolPreparedStatements;
		private int maxPoolPreparedStatementPerConnectionSize;
		private String filters;
		private String connectionProperties;
	 
		// 解决 spring.datasource.filters=stat,wall,log4j 无法正常注册
		@Bean 
		@Primary // 在同样的DataSource中，首先使用被标注的DataSource
		public DataSource dataSource() {
			DruidDataSource datasource = new DruidDataSource();
			datasource.setUrl(url);
			datasource.setUsername(username);
			datasource.setPassword(password);
			datasource.setDriverClassName(driverClassName);
	 
			// configuration
			datasource.setInitialSize(initialSize);
			datasource.setMinIdle(minIdle);
			datasource.setMaxActive(maxActive);
			datasource.setMaxWait(maxWait);
			datasource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
			datasource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
			datasource.setValidationQuery(validationQuery);
			datasource.setTestWhileIdle(testWhileIdle);
			datasource.setTestOnBorrow(testOnBorrow);
			datasource.setTestOnReturn(testOnReturn);
			datasource.setPoolPreparedStatements(poolPreparedStatements);
			datasource.setMaxPoolPreparedStatementPerConnectionSize(maxPoolPreparedStatementPerConnectionSize);
			try {
				datasource.setFilters(filters);
			} catch (SQLException e) {
				System.err.println("druid configuration initialization filter: " + e);
			}
			datasource.setConnectionProperties(connectionProperties);
			return datasource;
		}
	 
		// 此处省略属性的get和set方法
	 
	}
```

# 配置Druid监控统计功能

基于Druid的Filter-Chain扩展机制，Druid提供了3个非常有用的具有监控统计功能的Filter：

- StatFilter 用于统计监控信息；
- WallFilter 基于SQL语义分析来实现防御SQL注入攻击；
- LogFilter 用于输出JDBC执行的日志。

如果在项目中需要使用Druid提供的这些监控统计功能，可以通过以下两种途径进行配置。

## 方式一（基于Servlet 3.0 注解的配置）
对于使用Servlet 3.0的项目，在启动类上加上注解 @ServletComponentScan 启用Servlet自动扫描，并在自定义的Servlet或Filter上加上注解@WebServlet或@WebFilter使其能够被自动发现。
```Java
	@SpringBootApplication
	@ServletComponentScan
	public class MyApplication {
	 
		public static void main(String[] args) {
			SpringApplication.run(MyApplication.class, args);
		}
	}
```
<br/>
```Java
	/**
	 * druid数据源状态监控.
	 */
	@WebServlet(urlPatterns = "/druid/*", initParams = {
			// IP白名单 (没有配置或者为空，则允许所有访问)
			@WebInitParam(name = "allow", value = "192.168.1.101,127.0.0.1"),
			// IP黑名单 (存在共同时，deny优先于allow)
			@WebInitParam(name = "deny", value = "192.168.1.100"),
			// 用户名
			@WebInitParam(name = "loginUsername", value = "admin"),
			// 密码
			@WebInitParam(name = "loginPassword", value = "admin"),
			// 禁用HTML页面上的“Reset All”功能
			@WebInitParam(name = "resetEnable", value = "false") })
	@SuppressWarnings("serial")
	public class DruidStatViewServlet extends StatViewServlet {
	 
	}
```
<br/>
```Java
	/**
	 * druid过滤器. 
	 */
	@WebFilter(filterName = "druidWebStatFilter", urlPatterns = "/*", initParams = {
			// 忽略资源
			@WebInitParam(name = "exclusions", value = "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*") })
	public class DruidStatFilter extends WebStatFilter {
	}
```

## 方式二（基于Spring注解的配置）
使用Spring的注解@Bean对自定义的Servlet或Filter进行注册，Servlet使用ServletRegistrationBean进行注册，Filter使用FilterRegistrationBean进行注册。 
```Java
	@SpringBootConfiguration
	public class DruidMonitorConfig {
	 
		private static final Logger logger = LoggerFactory.getLogger(DruidMonitorConfig.class);
	 
		@Bean
		public ServletRegistrationBean servletRegistrationBean() {
			logger.info("init Druid Monitor Servlet ...");
			ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(),
					"/druid/*");
			// IP白名单
			servletRegistrationBean.addInitParameter("allow", "192.168.1.101,127.0.0.1");
			// IP黑名单(共同存在时，deny优先于allow)
			servletRegistrationBean.addInitParameter("deny", "192.168.1.100");
			// 控制台管理用户
			servletRegistrationBean.addInitParameter("loginUsername", "admin");
			servletRegistrationBean.addInitParameter("loginPassword", "admin");
			// 是否能够重置数据 禁用HTML页面上的“Reset All”功能
			servletRegistrationBean.addInitParameter("resetEnable", "false");
			return servletRegistrationBean;
		}
	 
		@Bean
		public FilterRegistrationBean filterRegistrationBean() {
			FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
			filterRegistrationBean.addUrlPatterns("/*");
			filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
			return filterRegistrationBean;
		}
	 
	}
```

# 配置JPA
创建一个用户POJO实体类，用来进行示例。 
```Java
	@Entity
	@Table(name = "tbl_user") // 指定关联的数据库的表名
	public class User implements Serializable {
	 
		private static final long serialVersionUID = 1L;
	 
		@Id
		@GeneratedValue(strategy = GenerationType.IDENTITY)
		private Long id; // 主键ID
	 
		private String name; // 姓名
	 
		public Long getId() {
			return id;
		}
	 
		public void setId(Long id) {
			this.id = id;
		}
	 
		public String getName() {
			return name;
		}
	 
		public void setName(String name) {
			this.name = name;
		}
	 
		@Override
		public String toString() {
			return "User [id=" + id + ", name=" + name + "]";
		}
	 
	}
```
再创建一个用户的数据库访问资源库接口。  
```Java
	@Repository
	public interface UserRepository extends CrudRepository<User, Long> {
	 
	}
```
在UserController控制器中添加两个方法，一个用来添加用户，一个用来查看所有用户。  
```Java
	@RestController
	@RequestMapping("/user")
	public class UserController {
	 
		@Autowired
		private UserRepository userRepository;
	 
		@RequestMapping("/add")
		public Object add(@RequestParam(name = "name", required = true) String name) {
			User user = new User();
			user.setName(name);
			userRepository.save(user);
			return user;
		}
	 
		@RequestMapping("/list")
		public Object list() {
			Iterable<User> users = userRepository.findAll();
			return users;
		}
	}
```

# 应用测试

启动应用，先访问地址：<http://localhost:8080/user/add?name=Tracy>，为用户添加一个姓名为Tracy的用户。  

<div align=center>

![Druid示意图](http://pengjunlee.3vzhuji.net/static/springboot/39.png "Druid示意图")
<div align=left>

再访问地址：<http://localhost:8080/user/list>，查看所有用户。 

<div align=center>

![Druid示意图](http://pengjunlee.3vzhuji.net/static/springboot/40.png "Druid示意图")
<div align=left>

最后访问地址：<http://localhost:8080/druid/>，登录之后，即可查看Druid数据源的配置及SQL统计情况。 

<div align=center>

![Druid示意图](http://pengjunlee.3vzhuji.net/static/springboot/41.png "Druid示意图")
<div align=left>

本文项目源码已上传至CSDN，资源地址：<https://download.csdn.net/download/pengjunlee/10374589>