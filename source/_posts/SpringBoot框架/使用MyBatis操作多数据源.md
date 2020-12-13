---
title: SpringBoot框架整合之--使用MyBatis操作多数据源
date: 2020-07-24 14:10:00
updated: 2020-07-24 14:10:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何整合MyBatis?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img10.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img10.jpg
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
本文将对如何使用Springboot+MyBatis操作多数据源进行简单示例和介绍，项目的完整目录层次如下图所示。  

<div align=center>

![数据源示意图](http://pengjunlee.3vzhuji.net/static/springboot/44.png "数据源示意图")
<div align=left>

# 添加依赖与配置

首先，需要在工程POM文件中引入MyBatis和MySQL的Maven依赖。
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.1.RELEASE</version>
	</parent>
	<dependencies>
		<!-- 添加 MyBatis -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.2.0</version>
		</dependency>
		<!-- 添加 MySQL -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
	</dependencies>
```
其中`mybatis-spring-boot-starter`是 MyBatis 官方提供的Starter， 其Github 源码地址: <https://github.com/mybatis/spring-boot-starter> 。

此外，mybatis-spring-boot-stater 的官方文档地址：<http://www.mybatis.org/spring-boot-starter/>，MyBatis 官方中文文档地址：<http://www.mybatis.org/mybatis-3/zh/java-api.html> 。

然后，在核心配置文件`application.properties`中添加数据源相关配置。  
```Properties
	#########################################################
	### Primary DataSource -- DataSource 1 configuration  ###
	#########################################################
	primary.datasource.url=jdbc:mysql://localhost:3306/dev1?useUnicode=true&characterEncoding=utf8
	primary.datasource.driverClassName=com.mysql.jdbc.Driver
	primary.datasource.username=root
	primary.datasource.password=123456
	
	#########################################################
	### Secondary DataSource -- DataSource 2 configuration ##
	#########################################################
	secondary.datasource.url=jdbc:mysql://localhost:3306/dev2?useUnicode=true&characterEncoding=utf8
	secondary.datasource.driverClassName=com.mysql.jdbc.Driver
	secondary.datasource.username=root
	secondary.datasource.password=123456
```

# 配置数据源

首先，分别为每一个数据源各定义一个配置类，在配置类中通过注解@MapperScan来为不同包下的Mapper分别创建不同`SqlSessionFactory`、`SqlSessionTemplate`和`DataSourceTransactionManager`，用来操作不同的数据源。 
```Java
	/**
	 * 数据源 1 配置类
	 */
	@Configuration
	@MapperScan(basePackages = "com.pengjunlee.primary.mapper", sqlSessionTemplateRef = "primarySqlSessionTemplate")
	public class PrimaryDataSourceConfig {
	 
		@Bean(name = "primaryDataSource")
		@ConfigurationProperties(prefix = "primary.datasource")
		public DataSource primaryDataSource() {
			return DataSourceBuilder.create().build();
		}
	 
		@Bean
		@ConfigurationProperties(prefix = "primary.datasource")
		@Primary
		public DataSource createDataSource() {
			return DataSourceBuilder.create().build();
		}
	 
		@Primary
		@Bean(name = "primarySqlSessionFactory")
		public SqlSessionFactory primarySqlSessionFactory(@Qualifier("primaryDataSource") DataSource dataSource)
				throws Exception {
			SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
			bean.setDataSource(dataSource);
			bean.setMapperLocations(
					new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/primary/*.xml"));
			return bean.getObject();
		}
	 
		@Primary
		@Bean(name = "primaryTransactionManager")
		public DataSourceTransactionManager primaryTransactionManager(
				@Qualifier("primaryDataSource") DataSource dataSource) {
			return new DataSourceTransactionManager(dataSource);
		}
	 
		@Primary
		@Bean(name = "primarySqlSessionTemplate")
		public SqlSessionTemplate primarySqlSessionTemplate(
				@Qualifier("primarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
			return new SqlSessionTemplate(sqlSessionFactory);
		}
	 
	}
```
<br/>
```Java
	/**
	 * 数据源 2 配置类
	 */
	@Configuration
	@MapperScan(basePackages = "com.pengjunlee.secondary.mapper", sqlSessionTemplateRef = "secondarySqlSessionTemplate")
	public class SecondaryDataSourceConfig {
	 
		@Bean(name = "secondaryDataSource")
		@ConfigurationProperties(prefix = "secondary.datasource")
		public DataSource secondaryDataSource() {
			return DataSourceBuilder.create().build();
		}
	 
		@Bean(name = "secondarySqlSessionFactory")
		public SqlSessionFactory secondarySqlSessionFactory(@Qualifier("secondaryDataSource") DataSource dataSource)
				throws Exception {
			SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
			bean.setDataSource(dataSource);
			bean.setMapperLocations(
					new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/secondary/*.xml"));
			return bean.getObject();
		}
	 
		@Bean(name = "secondaryTransactionManager")
		public DataSourceTransactionManager secondaryTransactionManager(
				@Qualifier("secondaryDataSource") DataSource dataSource) {
			return new DataSourceTransactionManager(dataSource);
		}
	 
		@Bean(name = "secondarySqlSessionTemplate")
		public SqlSessionTemplate secondarySqlSessionTemplate(
				@Qualifier("secondarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
			return new SqlSessionTemplate(sqlSessionFactory);
		}
	 
	}
```

# 定义Mapper

由于注解@MapperScan是通过包名来区分哪些Mapper用来操作哪个数据源，故而需要将不同数据源的Mapper接口类定义在不同的包中。

此处为了示例，定义了User（用户）和Department（部门）两个实体类，分别对应数据源一和数据源二。 
```Java
	/**
	 * 用户
	 */
	public class User {
	 
		private Long id;
	 
		private String name;
	 
		private Integer age;
	 
		// 此处省略get和set方法
	}
```
<br/>
```Java
	/**
	 * 部门
	 */
	public class Department {
	 
		private Long id;
	 
		private String name;
	 
		// 此处省略get和set方法
	 
	}
```
<br/>
```Java
	package com.pengjunlee.primary.mapper;
	 
	import org.apache.ibatis.annotations.Mapper;
	import org.apache.ibatis.annotations.Param;
	 
	import com.pengjunlee.bean.User;
	 
	/**
	 * 数据源 1 Mapper
	 */
	@Mapper
	public interface UserMapper {
	 
		User selectById(@Param("id") Long id);
	 
		void deleteById(@Param("id") Long id);
	}
```
<br/>
```Java
	package com.pengjunlee.secondary.mapper;
	 
	import org.apache.ibatis.annotations.Mapper;
	import org.apache.ibatis.annotations.Param;
	import com.pengjunlee.bean.Department;
	 
	/**
	 * 数据源 2 Mapper
	 */
	@Mapper
	public interface DepartmentMapper {
	 
		Department selectById(@Param("id") Long id);
	 
		void deleteById(@Param("id") Long id);
	}
```
> **注意**：在定义Mapper接口的时候，其所在包一定要与配置文件中指定的包路径一致。  

# 使用Mapper

在实际项目中我们一般都会将定义好的Mapper自动装配到Service层进行调用，此时要格外注意事务。

以使用UserMapper为例，UserService接口定义如下。 
```Java
	public interface UserService {
	 
		// 根据主键查询用户
		User getUserById(Long id);
	 
		// 根据主键删除用户
		void deleteUserById(Long id);
	}
```
在接口实现类UserServiceImpl中自动装配UserMapper来完成对数据源1的操作。  
```Java
	@Service("userService")
	public class UserServiceImpl implements UserService {
	 
		@Autowired
		private UserMapper userMapper;
	 
		@Override
		public User getUserById(Long id) {
			return userMapper.selectById(id);
		}
	 
		@Override
		@Transactional("primaryTransactionManager")// 注意事务
		public void deleteUserById(Long id) {
			userMapper.deleteById(id);
			throw new RuntimeException();
		}
	 
	}
```

# 启动类测试

在启动类中进行测试，测试代码如下。 
```Java
	@SpringBootApplication
	public class MyApplication {
	 
		public static void main(String[] args) {
			ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
	 
			UserMapper userMapper = context.getBean(UserMapper.class);
			User user = userMapper.selectById(3L);
			System.out.println(user.getName());
	 
			DepartmentMapper deptMapper = context.getBean(DepartmentMapper.class);
			Department dept = deptMapper.selectById(2L);
			System.out.println(dept.getName());
	 
			// 通过删除操作测试事务
			// DepartmentService deptService =
			// context.getBean(DepartmentService.class);
			// deptService.deleteDepartmentById(2L);
	 
			UserService userService = context.getBean(UserService.class);
			userService.deleteUserById(3L);
		}
	}
```
启动程序，各数据源中的数据都能查询成功且支持事务。 

本文项目源码已上传至CSDN，资源地址：<https://download.csdn.net/download/pengjunlee/10612419>