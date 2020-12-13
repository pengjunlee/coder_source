---
title: SpringBoot框架整合之--使用MongoDB的连接池配置
date: 2020-07-24 14:09:00
updated: 2020-07-24 14:09:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何整合MongoDB?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img9.jpg
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
在SpringBoot中，我们可以通过引入 `spring-boot-starter-data-mongodb` 依赖来实现spring-data-mongodb 的自动配置。但是，默认情况下，该依赖并没有像使用MySQL或者Redis那样为我们提供连接池配置的功能。因此，我们需要自行重写 MongoDbFactory，实现MongoDB客户端连接的参数配置扩展。需要说明的是，MongoDB的客户端本身就是一个连接池，因此，我们只需要配置客户端即可。

# 引入依赖
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.2.RELEASE</version>
	</parent>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```

# 配置文件

为了方便对Mongodb进行统一管理，我们将相关的配置抽取到 mongo-pool.properties 中，前缀为spring.data.mongodb（前缀可自己随意配置）：
```Properties
	spring.data.mongodb.address=172.16.250.234:27017,172.16.250.239:27017,172.16.250.240:27017
	spring.data.mongodb.replica-set=rs0
	spring.data.mongodb.database=test
	spring.data.mongodb.username=admin
	spring.data.mongodb.password=admin
	 
	# Configure spring.data.mongodbDB Pool
	spring.data.mongodb.min-connections-per-host=10
	spring.data.mongodb.max-connections-per-host=100
	spring.data.mongodb.threads-allowed-to-block-for-connection-multiplier=5
	spring.data.mongodb.server-selection-timeout=30000
	spring.data.mongodb.max-wait-time=120000
	spring.data.mongodb.max-connection-idel-time=0
	spring.data.mongodb.max-connection-life-time=0
	spring.data.mongodb.connect-timeout=10000
	spring.data.mongodb.socket-timeout=0
	spring.data.mongodb.socket-keep-alive=false
	spring.data.mongodb.ssl-enabled=false
	spring.data.mongodb.ssl-invalid-host-name-allowed=false
	spring.data.mongodb.always-use-m-beans=false
	spring.data.mongodb.heartbeat-socket-timeout=20000
	spring.data.mongodb.heartbeat-connect-timeout=20000
	spring.data.mongodb.min-heartbeat-frequency=500
	spring.data.mongodb.heartbeat-frequency=10000
	spring.data.mongodb.local-threshold=15
	spring.data.mongodb.authentication-database=auth_dev
```

# 配置文件映射为JavaBean

为方便调用，将上述配置包装成一个配置实体类，代码如下:
```Java
	import java.util.List;
	 
	import org.springframework.boot.context.properties.ConfigurationProperties;
	import org.springframework.context.annotation.PropertySource;
	import org.springframework.stereotype.Component;
	 
	@Component
	@PropertySource(value = "classpath:mongo-pool.properties")
	@ConfigurationProperties(prefix = "spring.data.mongodb")
	public class MongoSettingsProperties {
	 
		private List<String> address;
		private String replicaSet;
		private String database;
		private String username;
		private String password;
		private Integer minConnectionsPerHost = 0;
		private Integer maxConnectionsPerHost = 100;
		private Integer threadsAllowedToBlockForConnectionMultiplier = 5;
		private Integer serverSelectionTimeout = 30000;
		private Integer maxWaitTime = 120000;
		private Integer maxConnectionIdleTime = 0;
		private Integer maxConnectionLifeTime = 0;
		private Integer connectTimeout = 10000;
		private Integer socketTimeout = 0;
		private Boolean socketKeepAlive = false;
		private Boolean sslEnabled = false;
		private Boolean sslInvalidHostNameAllowed = false;
		private Boolean alwaysUseMBeans = false;
		private Integer heartbeatConnectTimeout = 20000;
		private Integer heartbeatSocketTimeout = 20000;
		private Integer minHeartbeatFrequency = 500;
		private Integer heartbeatFrequency = 10000;
		private Integer localThreshold = 15;
		private String authenticationDatabase;
	 
		// 省略Getters和Setters方法
	 
	}
```

# 覆盖MongoDbFactory

自定义创建一个MongoDbFactory用来替代Springboot为我们自动装配的MongoDbFactory，代码如下：
```Java
	import java.util.ArrayList;
	import java.util.List;
	 
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.data.mongodb.MongoDbFactory;
	import org.springframework.data.mongodb.core.SimpleMongoDbFactory;
	 
	import com.mongodb.MongoClient;
	import com.mongodb.MongoClientOptions;
	import com.mongodb.MongoCredential;
	import com.mongodb.ServerAddress;
	 
	@Configuration
	public class MongoConfig {
	 
		private static final Logger logger = LoggerFactory.getLogger(MongoConfig.class);
	 
		// 覆盖容器中默认的MongoDbFacotry Bean
		@Bean
		@Autowired
		public MongoDbFactory mongoDbFactory(MongoSettingsProperties properties) {
	 
			// 客户端配置（连接数，副本集群验证）
			MongoClientOptions.Builder builder = new MongoClientOptions.Builder();
			builder.connectionsPerHost(properties.getMaxConnectionsPerHost());
			builder.minConnectionsPerHost(properties.getMinConnectionsPerHost());
			if (properties.getReplicaSet() != null) {
				builder.requiredReplicaSetName(properties.getReplicaSet());
			}
			builder.threadsAllowedToBlockForConnectionMultiplier(
					properties.getThreadsAllowedToBlockForConnectionMultiplier());
			builder.serverSelectionTimeout(properties.getServerSelectionTimeout());
			builder.maxWaitTime(properties.getMaxWaitTime());
			builder.maxConnectionIdleTime(properties.getMaxConnectionIdleTime());
			builder.maxConnectionLifeTime(properties.getMaxConnectionLifeTime());
			builder.connectTimeout(properties.getConnectTimeout());
			builder.socketTimeout(properties.getSocketTimeout());
			// builder.socketKeepAlive(properties.getSocketKeepAlive());
			builder.sslEnabled(properties.getSslEnabled());
			builder.sslInvalidHostNameAllowed(properties.getSslInvalidHostNameAllowed());
			builder.alwaysUseMBeans(properties.getAlwaysUseMBeans());
			builder.heartbeatFrequency(properties.getHeartbeatFrequency());
			builder.minHeartbeatFrequency(properties.getMinHeartbeatFrequency());
			builder.heartbeatConnectTimeout(properties.getHeartbeatConnectTimeout());
			builder.heartbeatSocketTimeout(properties.getHeartbeatSocketTimeout());
			builder.localThreshold(properties.getLocalThreshold());
			MongoClientOptions mongoClientOptions = builder.build();
	 
			// MongoDB地址列表
			List<ServerAddress> serverAddresses = new ArrayList<ServerAddress>();
			for (String address : properties.getAddress()) {
				String[] hostAndPort = address.split(":");
				String host = hostAndPort[0];
				Integer port = Integer.parseInt(hostAndPort[1]);
				ServerAddress serverAddress = new ServerAddress(host, port);
				serverAddresses.add(serverAddress);
			}
	 
			logger.info("serverAddresses:" + serverAddresses.toString());
	 
			// 连接认证
			// MongoCredential mongoCredential = null;
			// if (properties.getUsername() != null) {
			// 	mongoCredential = MongoCredential.createScramSha1Credential(
			// 			properties.getUsername(), properties.getAuthenticationDatabase() != null
			// 					? properties.getAuthenticationDatabase() : properties.getDatabase(),
			// 			properties.getPassword().toCharArray());
			// }
	 
			// 创建认证客户端
			// MongoClient mongoClient = new MongoClient(serverAddresses, mongoCredential, mongoClientOptions);
	 
			// 创建非认证客户端
			MongoClient mongoClient = new MongoClient(serverAddresses, mongoClientOptions);
	 
			// 创建MongoDbFactory
			MongoDbFactory mongoDbFactory = new SimpleMongoDbFactory(mongoClient, properties.getDatabase());
			return mongoDbFactory;
		}
	}
```

# MongoDB测试

## 创建数据实体
```Java 
	import java.io.Serializable;
	 
	public class UserEntity implements Serializable {
	 
		private static final long serialVersionUID = 1L;
	 
		private Long id;
		private String userName;
		private String passWord;
	 
		public Long getId() {
			return id;
		}
	 
		public void setId(Long id) {
			this.id = id;
		}
	 
		public String getUserName() {
			return userName;
		}
	 
		public void setUserName(String userName) {
			this.userName = userName;
		}
	 
		public String getPassWord() {
			return passWord;
		}
	 
		public void setPassWord(String passWord) {
			this.passWord = passWord;
		}
	 
		public String toString() {
			return "id: " + id + ",userName: " + userName + ",passWord: " + passWord;
		}
	}
```

## 创建Dao接口及实现
```Java
	public interface UserDao {
	 
		void saveUser(UserEntity user);
	 
		UserEntity findUserByName(String userName);
	}
```
<br/>
```Java
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.mongodb.core.MongoTemplate;
	import org.springframework.data.mongodb.core.query.Criteria;
	import org.springframework.data.mongodb.core.query.Query;
	import org.springframework.stereotype.Component;
	 
	@Component
	public class UserDaoImpl implements UserDao {
	 
		@Autowired
		private MongoTemplate mongoTemplate;
	 
		@Override
		public void saveUser(UserEntity user) {
			mongoTemplate.save(user);
		}
	 
		@Override
		public UserEntity findUserByName(String userName) {
			Query query = new Query(Criteria.where("userName").is(userName));
			UserEntity user = mongoTemplate.findOne(query, UserEntity.class);
			return user;
		}
	 
	}
```

## 编写测试代码
```Java
	import java.util.Optional;
	 
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.data.domain.Example;
	import org.springframework.test.context.junit4.SpringRunner;
	 
	import com.pengjunlee.UserDao;
	import com.pengjunlee.UserEntity;
	import com.pengjunlee.UserRepository;
	 
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class MongoTest {
		@Autowired
		private UserDao userDao;
	 
		@Autowired
		private UserRepository userRepository;
	 
		@Test
		public void testSaveUser() {
			UserEntity user = new UserEntity();
			user.setId(88L);
			user.setUserName("XiaoMing");
			user.setPassWord("123456");
			userDao.saveUser(user);
		}
	 
		@Test
		public void testFindUser01() {
			UserEntity user = userDao.findUserByName("XiaoMing");
			System.out.println(user);
		}
	 
		@Test
		public void testFindUser02() {
			UserEntity queryUser = new UserEntity();
			queryUser.setUserName("XiaoMing");
			Example<UserEntity> example = Example.of(queryUser);
			Optional<UserEntity> optional = userRepository.findOne(example);
			System.out.println(optional.get());
		}
	}
```
查询结果：
```
id: 88,userName: XiaoMing,passWord: 123456
```