---
title: SpringBoot基础之--配置数据源
date: 2020-07-25 14:15:00
updated: 2020-07-25 14:15:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何配置数据源？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img15.jpg
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
# 默认数据源

Springboot默认支持4种数据源类型，定义在 `org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration` 中，分别是：

- org.apache.tomcat.jdbc.pool.DataSource
- com.zaxxer.hikari.HikariDataSource
- org.apache.commons.dbcp.BasicDataSource
- org.apache.commons.dbcp2.BasicDataSource

对于这4种数据源，当 `classpath` 下有相应的类存在时，Springboot 会通过自动配置为其生成`DataSource Bean`，DataSource Bean默认只会生成一个，四种数据源类型的生效先后顺序如下：`Tomcat--> Hikari --> Dbcp --> Dbcp2` 。 

## 添加依赖与配置

在Springboot 使用JDBC可直接添加官方提供的 `spring-boot-start-jdbc` 或者 `spring-boot-start-data-jpa` 依赖。 
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.1.RELEASE</version>
	</parent>
	<dependencies>
		<!-- 添加MySQL依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- 添加JDBC依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
	</dependencies>
```
在核心配置`application.properties`或者`application.yml`文件中添加数据源相关配置。 
```Properties
	# application.properties文件中添加如下配置：
	spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
	spring.datasource.driverClassName=com.mysql.jdbc.Driver
	spring.datasource.username=root
	spring.datasource.password=123456
``` 
```Yml
	# application.yml文件中添加如下配置：
	spring:
	  datasource:
	    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
	    driverClassName: com.mysql.jdbc.Driver
	    username: root
	    password: 123456
```

## 切换默认数据源

- 方式一 排除其他的数据源依赖，仅保留需要的数据源依赖； 
- 方式二 通过在核心配置中通过spring.datasource.type属性指定数据源的类型； 

### 方式一
Springboot默认支持的4种数据源Maven依赖如下：
```Xml
		<!-- 添加Tomcat-JDBC依赖 -->
		<dependency>
			<groupId>org.apache.tomcat</groupId>
			<artifactId>tomcat-jdbc</artifactId>
		</dependency>
		<!-- 添加HikariCP依赖 -->
		<dependency>
			<groupId>com.zaxxer</groupId>
			<artifactId>HikariCP</artifactId>
		</dependency>
		<!-- 添加DBCP依赖 -->
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
		</dependency>
		<!-- 添加DBCP2依赖 -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-dbcp2</artifactId>
		</dependency>
```
当我们引入spring-boot-start-jdbc依赖时，其实里面就包含了 Tomcat-JDBC 的依赖，如果想要切换为其他的数据源类型，需要先将Tomcat-JDBC 依赖排除，再添加上需要的数据源的依赖，以使用HikariCP数据源为例，依赖配置如下。
```Xml
		<!-- 添加JDBC依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
			<exclusions>
				<!-- 排除Tomcat-JDBC依赖 -->
				<exclusion>
					<groupId>org.apache.tomcat</groupId>
					<artifactId>tomcat-jdbc</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- 添加HikariCP依赖 -->
		<dependency>
			<groupId>com.zaxxer</groupId>
			<artifactId>HikariCP</artifactId>
		</dependency>
```

### 方式二 
 此外，还可以通过在核心配置中通过添加`spring.datasource.type = [数据源类型]` 来指定数据源的类型； 
```Properties
	spring.datasource.type=com.zaxxer.hikari.HikariDataSource
	# spring.datasource.type=org.apache.tomcat.jdbc.pool.DataSource
	# spring.datasource.type=org.apache.commons.dbcp.BasicDataSource
	# spring.datasource.type=org.apache.commons.dbcp2.BasicDataSource
```

# 第三方数据源

如果不想使用Springboot默认支持的4种数据源，还可以选择使用其他第三方的数据源，例如：Druid、c3p0等。
以使用Druid数据源为例。 

## 添加依赖与配置
在pom文件中引入第三方数据源依赖。 
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.1.RELEASE</version>
	</parent>
	<dependencies>
		<!-- 添加MySQL依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- 添加JDBC依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<!-- 添加Druid依赖 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.6</version>
		</dependency>
	</dependencies>
```
核心配置文件中的添加数据源相关配置与使用默认数据源时的配置相同，此处不再重复贴出。  

## 定义数据源
使用注解@Bean 创建一个DataSource Bean并将其纳入到Spring容器中进行管理即可。 
```Java
	@Configuration
	public class DataSourceConfig {
	 
		@Autowired
		private Environment env;
	 
		@Bean
		public DataSource getDataSource() {
			DruidDataSource dataSource = new DruidDataSource();
			dataSource.setUrl(env.getProperty("spring.datasource.url"));
			dataSource.setUsername(env.getProperty("spring.datasource.username"));
			dataSource.setPassword(env.getProperty("spring.datasource.password"));
			return dataSource;
		}
	}
```
或者： 
```Java
	@Configuration
	@ConfigurationProperties(prefix = "spring.datasource")
	public class DataSource2Config {
	 
		private String url;
		private String username;
		private String password;
	 
		@Bean
		public DataSource getDataSource() {
			DruidDataSource dataSource = new DruidDataSource();
			dataSource.setUrl(url);
			dataSource.setUsername(username);// 用户名
			dataSource.setPassword(password);// 密码
			return dataSource;
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
	}
```