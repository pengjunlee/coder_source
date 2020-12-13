---
title: SpringBoot基础之--使用Junit进行单元测试
date: 2020-07-25 14:23:00
updated: 2020-07-25 14:23:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: SpringBoot项目如何使用Junit进行单元测试？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img23.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img23.jpg
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
本文将对在Springboot中如何使用Junit进行单元测试进行简单示例和介绍，项目的完整目录层次如下图所示。 

<div align=center>

![Junit示意图](http://pengjunlee.3vzhuji.net/static/springboot/45.png "Junit示意图")
<div align=left>

# 添加依赖与配置

为了保证测试的完整性，本工程POM文件中除引入Junit单元测试依赖外，还额外引入了用来测试JDBC和Controller的JPA和WEB依赖。 
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.6.RELEASE</version>
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
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- 引入单元测试依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```
同时，在`src/main/resources`目录下添加核心配置文件`application.properties`，内容如下。  
```Properties
	#########################################################
	###  Spring DataSource  -- DataSource configuration   ###
	#########################################################
	spring.datasource.url=jdbc:mysql://localhost:3306/dev1?useUnicode=true&characterEncoding=utf8
	spring.datasource.driverClassName=com.mysql.jdbc.Driver
	spring.datasource.username=root
	spring.datasource.password=123456
	 
	#########################################################
	### Java Persistence Api --  Spring jpa configuration ###
	#########################################################
	# Specify the DBMS
	spring.jpa.database = MYSQL
	# Show or not log for each sql query
	spring.jpa.show-sql = true
	# Hibernate ddl auto (create, create-drop, update)
	spring.jpa.hibernate.ddl-auto = update
	# Naming strategy
	#[org.hibernate.cfg.ImprovedNamingStrategy  #org.hibernate.cfg.DefaultNamingStrategy]
	spring.jpa.hibernate.naming-strategy = org.hibernate.cfg.ImprovedNamingStrategy
	# stripped before adding them to the entity manager)
	spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
```

# ApplicationContext测试

在Springboot中使用Junit进行单元测试的方法很简单，只需要在编写的单元测试类上添加两个注解：`@RunWith(SpringRunner.class)`和`@SpringBootTest`。 
```Java
	@RunWith(SpringRunner.class) // 等价于使用 @RunWith(SpringJUnit4ClassRunner.class)
	@SpringBootTest(classes = { MyApplication.class, TestConfig.class })
	public class ApplicationContextTest {
	 
		@Autowired
		private ApplicationContext context;
	 
		@Autowired
		private UserDao userDao;
	 
		@Test
		public void testUserDao() {
			userDao.addUser(18, "pengjunlee");
		}
	 
		@Test
		public void testConfiguration() {
			Runnable bean = context.getBean(Runnable.class);
			Assert.assertNotNull(bean);
			bean.run();
		}
	 
	}
```
UserDao定义如下。  
```Java
	@Repository
	public class UserDao {
	 
		@Autowired
		private JdbcTemplate jdbcTemplate;
	 
		@Transactional
		public void addUser(Integer userAge, String userName) {
			String sql = "insert into tbl_user (age,name) values ('" + userAge + "','" + userName + "');";
			jdbcTemplate.execute(sql);
		}
	}
```
TestConfig定义如下。 
```Java
	/**
	 * @TestConfiguration注解的配置内的Bean仅在测试时装配
	 */
	@TestConfiguration
	public class TestConfig {
	 
		@Bean
		public Runnable createRunnable(){
			return ()->{
				System.out.println("This is a test Runnable bean...");
			};
		}
	}
```
> **提示**：@SpringBootTest注解的classes可以指定用来加载Spring容器所使用的配置类，@TestConfiguration注解修饰的配置类内的Bean仅在测试的时候才会装配。  

# Environment测试

我们可以通过@SpringBootTest注解的properties属性向Environment中设置新的属性，也可以通过使用EnvironmentTestUtils工具类来向ConfigurableEnvironment中添加新的属性。 
```Java
	@RunWith(SpringRunner.class)
	@SpringBootTest(properties = { "app.token=pengjunlee" })
	public class EnvironmentTest {
	 
		@Autowired
		private Environment env;
	 
		@Autowired
		private ConfigurableEnvironment cenv;
	 
		@Before
		public void init() {
			EnvironmentTestUtils.addEnvironment(cenv, "app.secret=55a4b77eda");
		}
	 
		@Test
		public void testEnvironment() {
			System.out.println(env.getProperty("spring.datasource.url"));
			Assert.assertEquals("pengjunlee", env.getProperty("app.token"));
			Assert.assertEquals("55a4b77eda", cenv.getProperty("app.secret"));
		}
	 
		@Test
		@Ignore // 忽略测试方法
		public void testIgnore() {
	 
			System.out.println("你看不见我...");
		}
	 
	}
```
> **扩展**：Junit测试用例执行顺序：`@BeforeClass` ==> `@Before` ==> `@Test` ==> `@After` ==> `@AfterClass` 。

> **注意**：在使用Junit对Spring容器进行单元测试时，若在src/test/resources 目录下存在核心配置文件，Spring容器将会只加载`src/test/resources` 目录下的核心配置文件，而不再加载`src/main/resources` 目录下的核心配置文件。  

# MockBean测试

我们可以通过`@MockBean`注解来对那些未添加实现的接口进行模拟测试，预先设定好调用方法期待的返回值，然后再进行测试。

例如，有如下的IUserService接口，定义了一个getUserAge()方法用来根据用户的ID来查询用户的年龄。 
```Java
	public interface IUserService {
	 
		Integer getUserAge(Long userId);
	 
	}
```
使用@MockBean来对IUserService接口进行模拟测试，测试代码如下。  
```Java
	@RunWith(SpringRunner.class)
	public class MockBeanTest {
	 
		@MockBean
		private IUserService userService;
	 
		@SuppressWarnings("unchecked")
		@Test(expected = NullPointerException.class)
		public void testMockBean() {
	 
			BDDMockito.given(userService.getUserAge(2L)).willReturn(Integer.valueOf(18));
			BDDMockito.given(userService.getUserAge(0L)).willReturn(Integer.valueOf(0));
			BDDMockito.given(userService.getUserAge(null)).willThrow(NullPointerException.class);
	 
			Assert.assertEquals(Integer.valueOf(18), userService.getUserAge(2L));
			Assert.assertEquals(Integer.valueOf(0), userService.getUserAge(0L));
			Assert.assertEquals(Integer.valueOf(0), userService.getUserAge(null));
	 
		}
	}
```

# Controller测试

在Springboot中可以通过TestRestTemplate和MockMvc来对Controller进行测试，有以下两种情况。

## 情况一
Controller中未装配任何其他Spring容器中的Bean，例如下面这个控制器。 
```Java
	@RestController
	@RequestMapping("/user")
	public class UserController01 {
	 
		@GetMapping("/home")
		public String homeUser(@RequestParam(name = "name", required = true) String userName) {
			if (null == userName || userName.trim() == "") {
				return "you are nobody...";
			}
			return "This is " + userName + "'s home...";
		}
	}
```
此时无需启动Spring容器，可直接使用MockMvc来对Controller进行模拟测试，测试代码如下。  
```Java
	@RunWith(SpringRunner.class)
	@WebMvcTest(controllers = { UserController01.class })
	public class ControllerTest01 {
	 
		@Autowired
		private MockMvc mvc;
	 
		@Test
		public void testAddUser() throws Exception {
			mvc.perform(MockMvcRequestBuilders.get("/user/home").param("name", ""))
					.andExpect(MockMvcResultMatchers.status().isOk())
					.andExpect(MockMvcResultMatchers.content().string("you are nobody..."));
			mvc.perform(MockMvcRequestBuilders.get("/user/home").param("name", "pengjunlee"))
					.andExpect(MockMvcResultMatchers.status().isOk())
					.andExpect(MockMvcResultMatchers.content().string("This is pengjunlee's home..."));
		}
	 
	}
```

## 情况二
Controller中需装配其他Spring容器中的Bean，例如下面这个控制器。 
```Java
	@RestController
	@RequestMapping("/user")
	public class UserController02 {
	 
		@Autowired
		private UserDao userDao;
	 
		@GetMapping("/add")
		public String addUser(@RequestParam(name = "age", required = false, defaultValue = "0") Integer userAge,
				@RequestParam(name = "name", required = true) String userName) {
			if (userAge <= 0 || null == userName || userName.trim() == "") {
				return "0";
			}
			userDao.addUser(userAge, userName);
			return "1";
		}
	 
	}
```
此时除了要启动Spring容器，还需要启动内嵌的WEB环境，有以下两种方法。

### 方法一
利用TestRestTemplate进行测试。 
```Java
	@RunWith(SpringRunner.class)
	@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
	public class ControllerTest02 {
	 
		@Autowired
		private TestRestTemplate template;
	 
		@Test
		public void testAddUser() {
			String result1 = template.getForObject("/user/add?name=pengjunlee", String.class);
			Assert.assertEquals("0", result1);
			String result2 = template.getForObject("/user/add?age=20&name=Tracy", String.class);
			Assert.assertEquals("1", result2);
		}
	 
	}
```

### 方法二
利用MockMvc进行测试。 
```Java
	@RunWith(SpringRunner.class)
	@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
	@AutoConfigureMockMvc
	public class ControllerTest03 {
	 
		@Autowired
		private MockMvc mvc;
	 
		@Test
		public void testAddUser() throws Exception {
			mvc.perform(MockMvcRequestBuilders.get("/user/add").param("name", ""))
					.andExpect(MockMvcResultMatchers.status().isOk())
					.andExpect(MockMvcResultMatchers.content().string("0"));
			mvc.perform(MockMvcRequestBuilders.get("/user/add").param("age", "22").param("name", "pengjunlee"))
					.andExpect(MockMvcResultMatchers.status().isOk())
					.andExpect(MockMvcResultMatchers.content().string("1"));
		}
	 
	}
```
本文项目源码已上传至CSDN，资源地址：<https://download.csdn.net/download/pengjunlee/10394302>