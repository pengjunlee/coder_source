---
title: 安全框架系列之--Shiro开启身份验证
date: 2020-08-01 14:08:00
updated: 2020-08-01 14:08:00
tags: Shiro
categories: 安全框架
keywords: 安全, Shiro
type: 
description: Shiro 开启身份验证。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img8.jpg
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
项目的完整目录层次如下图所示。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/04.png "Shiro示意图")
<div align=left>

项目地址：<https://github.com/pengjunlee/shiro-authenticate.git>

# 添加依赖与配置
在本工程中，Shiro使用了Ehchache作缓存，因此需要在工程POM文件中引入它们的Maven依赖。
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.0.RELEASE</version>
		<relativePath />
	</parent>
 
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
 
	<dependencies>
 
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
 
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
 
		<!-- 配置 Shiro -->
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-core</artifactId>
			<version>1.4.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring</artifactId>
			<version>1.4.1</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.2</version>
		</dependency>
 
		<!-- 配置 Shiro-Ehchache -->
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-ehcache</artifactId>
			<exclusions>
				<exclusion>
					<groupId>net.sf.ehcache</groupId>
					<artifactId>ehcache-core</artifactId>
				</exclusion>
			</exclusions>
			<version>1.4.0</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<dependency>
			<groupId>net.sf.ehcache</groupId>
			<artifactId>ehcache</artifactId>
		</dependency>
	</dependencies>
```
EhCacheManager使用默认的 `classpath:/ehcache.xml` 加载配置，文件内容如下。
```Xml
	<?xml version="1.0" encoding="UTF-8"?>
	<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
		updateCheck="false">
		<diskStore path="java.io.tmpdir/Tmp_EhCache" />
		<defaultCache eternal="false" maxElementsInMemory="1000"
			overflowToDisk="false" diskPersistent="false" timeToIdleSeconds="0"
			timeToLiveSeconds="600" memoryStoreEvictionPolicy="LRU" />
		<cache name="user" eternal="false" maxElementsInMemory="10000"
			overflowToDisk="false" diskPersistent="false" timeToIdleSeconds="0"
			timeToLiveSeconds="0" memoryStoreEvictionPolicy="LFU" />
```
在`application.properties`核心配置文件中加入Thymeleaf相关配置，方便前端页面演示：
```Properties
	# thymelea模板配置
	spring.thymeleaf.prefix=classpath:/templates/
	spring.thymeleaf.suffix=.html
	spring.thymeleaf.mode=HTML5
	spring.thymeleaf.encoding=UTF-8
	# 热部署文件，页面不产生缓存，及时更新
	spring.thymeleaf.cache=false
	spring.resources.chain.strategy.content.enabled=true
	spring.resources.chain.strategy.content.paths=/**
```

# 创建用户实体类
```Java
	import java.io.Serializable;
	 
	public class UserEntity implements Serializable {
	 
		private static final long serialVersionUID = 1L;
	 
		private Long id; // 主键ID
	 
		private String name; // 登录用户名
	 
		private String password; // 登录密码
	 
		private String nickName; // 昵称
	 
		private Boolean locked; // 账户是否被锁定
	 
		public UserEntity() {
			super();
		}
	 
		public UserEntity(Long id, String name, String password, String nickName, Boolean locked) {
			super();
			this.id = id;
			this.name = name;
			this.password = password;
			this.nickName = nickName;
			this.locked = locked;
		}
	 
		// 此处省略各属性的 getXXX() 和 setXXX() 方法
	 
	}
```

# 自定义Realm
<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/05.png "Shiro示意图")
<div align=left>

如果你只是想使用Shiro 进行身份验证，而不需要它的授权功能。那么，你只需让你的Realm继承 AuthenticatingRealm 并实现其抽象方法 doGetAuthenticationInfo() 即可。
```Java
	import java.util.HashMap;
	import java.util.Map;
	 
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.AuthenticationInfo;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.authc.LockedAccountException;
	import org.apache.shiro.authc.SimpleAuthenticationInfo;
	import org.apache.shiro.authc.UnknownAccountException;
	import org.apache.shiro.realm.AuthenticatingRealm;
	 
	import com.pengjunlee.domain.UserEntity;
	 
	/**
	 * 只开启身份验证，继承 AuthenticatingRealm 并实现 doGetAuthenticationInfo() 方法即可
	 */
	public class LoginRealm extends AuthenticatingRealm {
	 
		private static Map<String, UserEntity> users = new HashMap<String, UserEntity>(16);
	 
		static {
			UserEntity user1 = new UserEntity(1L, "graython", "123456", "灰先生", false);
			UserEntity user2 = new UserEntity(2L, "plum", "123456", "李先生", false);
			UserEntity user3 = new UserEntity(3L, "nightking", "123456", "夜王", false);
			UserEntity user4 = new UserEntity(4L, "guomeimei", "123456", "郭妹妹", true);
	 
			users.put("graython", user1);
			users.put("plum", user2);
			users.put("nightking", user3);
			users.put("guomeimei", user4);
		}
	 
		/**
		 * 查询数据库，将获取到的用户安全数据封装返回
		 */
		@Override
		protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
			// 从 AuthenticationToken 中获取当前用户
			String username = (String) token.getPrincipal();
			// 查询数据库获取用户信息，此处使用 Map 来模拟数据库
			UserEntity user = users.get(username);
	 
			// 用户不存在
			if (user == null) {
				throw new UnknownAccountException("用户不存在！");
			}
	 
			// 用户被锁定
			if (user.getLocked()) {
				throw new LockedAccountException("该用户已被锁定,暂时无法登录！");
			}
	 
			/**
			 * 将获取到的用户数据封装成 AuthenticationInfo 对象返回，此处封装为 SimpleAuthenticationInfo
			 * 对象。
			 *  参数1. 认证的实体信息，可以是从数据库中获取到的用户实体类对象或者用户名 
			 *  参数2. 查询获取到的登录密码 
			 *  参数3. 当前  Realm 对象的名称，直接调用父类的 getName() 方法即可
			 */
			SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getPassword(), getName());
			return info;
		}
	 
	}
```
在自定义Realm中，我们可以对一些业务相关的用户身份验证异常进行处理。除了可以直接使用 Shiro 为我们提供好的下面这些身份验证异常类，我们也可以使用其他自定义的验证异常类。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/06.png "Shiro示意图")
<div align=left>

# 集成Shiro
```Java
	import java.util.LinkedHashMap;
	 
	import org.apache.shiro.cache.ehcache.EhCacheManager;
	import org.apache.shiro.mgt.SecurityManager;
	import org.apache.shiro.spring.LifecycleBeanPostProcessor;
	import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
	import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
	import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
	import org.apache.shiro.web.session.mgt.DefaultWebSessionManager;
	import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	 
	import net.sf.ehcache.CacheManager;
	 
	@Configuration
	public class ShiroConfig {
	 
		/**
		 * 交由 Spring 来自动地管理 Shiro-Bean 的生命周期
		 */
		@Bean
		public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
			return new LifecycleBeanPostProcessor();
		}
	 
		/**
		 * 配置访问资源需要的权限
		 */
		@Bean
		ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
			ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
			shiroFilterFactoryBean.setSecurityManager(securityManager);
			shiroFilterFactoryBean.setLoginUrl("/login");
			shiroFilterFactoryBean.setSuccessUrl("/authorized");
			shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorized");
			LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
			filterChainDefinitionMap.put("/login", "anon"); // 可匿名访问
			filterChainDefinitionMap.put("/logout", "logout"); // 退出登录
			filterChainDefinitionMap.put("/**", "authc"); // 需登录才能访问
			shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
			return shiroFilterFactoryBean;
		}
	 
		/**
		 * 配置 SecurityManager，通常需要配置以下属性： 
		 * 1.CacheManager 
		 * 2.Realm 
		 * 3.SessionManager
		 */
		@Bean
		public SecurityManager securityManager() {
			DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
			// 1.CacheManager
			securityManager.setCacheManager(ehCacheManager());
			// 2.Realm
			securityManager.setRealm(loginRealm());
			// 3.SessionManager
			securityManager.setSessionManager(sessionManager());
			return securityManager;
		}
	 
		/**
		 * EhCacheManager缓存配置，默认使用 classpath:/ehcache.xml
		 */
		@Bean("cacheManager")
		public EhCacheManager ehCacheManager() {
			EhCacheManager em = new EhCacheManager();
			em.setCacheManager(cacheManager());
			return em;
		}
	 
		@Bean("cacheManager2")
		CacheManager cacheManager() {
			return CacheManager.create();
		}
	 
		/**
		 * Realm配置，需实现 Realm 接口
		 */
		@Bean
		LoginRealm loginRealm() {
			LoginRealm loginRealm = new LoginRealm();
			return loginRealm;
		}
	 
		/**
		 * SessionManager配置
		 */
		@Bean
		public DefaultWebSessionManager sessionManager() {
			DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
			sessionManager.setGlobalSessionTimeout(1800 * 1000);
			sessionManager.setDeleteInvalidSessions(true);
			sessionManager.setSessionValidationSchedulerEnabled(true);
			return sessionManager;
		}
	}
```
<br/>
```Java
	public enum DefaultFilter {
	 
	    anon(AnonymousFilter.class), // 可以匿名访问，示例：/static/**=anon
	    authc(FormAuthenticationFilter.class), // 需要经过身份验证才能访问，示例：/**=authc
	    logout(LogoutFilter.class), // 退出登录，示例：/logout=logout
	    perms(PermissionsAuthorizationFilter.class), // 需要用户具有相应权限才能访问，示例：/user/**=perms["user:*"]
	    port(PortFilter.class), // 只能通过指定端口访问，端口错误会重定向到相应端口，，示例：/test=port[80]
	    rest(HttpMethodPermissionFilter.class), // rest风格拦截器，会自动根据请求方法构建权限字符串
	    roles(RolesAuthorizationFilter.class), // 需要用户具有相应角色才能访问，示例：/admin/**=roles[admin]
	    ssl(SslFilter.class), // 只能通过HTTPS协议访问，否则自动跳转回HTTPS端口（443），示例：/admin/**=ssl
	    user(UserFilter.class); // 用户通过身份验证或者记住我登录后都能访问，示例：/admin/**=user
		
	}
```

# 视图Controller
```Java
	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.IncorrectCredentialsException;
	import org.apache.shiro.authc.LockedAccountException;
	import org.apache.shiro.authc.UnknownAccountException;
	import org.apache.shiro.authc.UsernamePasswordToken;
	import org.apache.shiro.subject.Subject;
	import org.springframework.stereotype.Controller;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PostMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	 
	import com.pengjunlee.domain.UserEntity;
	 
	@Controller
	public class LoginController {
	 
		@GetMapping("/login")
		public String login() {
			return "login";
		}
	 
		@PostMapping(value = "/login")
		public String userLogin(@RequestParam(name = "username") String userName,
				@RequestParam(name = "password") String password, ModelMap model) {
	 
			// 获取当前用户主体
			Subject subject = SecurityUtils.getSubject();
			String msg = null;
			// 判断是否已经验证身份，即是否已经登录
			if (!subject.isAuthenticated()) {
				// 将用户名和密码封装成 UsernamePasswordToken 对象
				UsernamePasswordToken token = new UsernamePasswordToken(userName, password);
				try {
					subject.login(token);
					System.out.println("用户 [ " + userName + " ] 登录成功。");
					addUserInfo2Model(model);
	 
				} catch (UnknownAccountException uae) { // 账号不存在
					msg = "用户名与密码不匹配，请检查后重新输入！";
				} catch (IncorrectCredentialsException ice) { // 账号与密码不匹配
					msg = "用户名与密码不匹配，请检查后重新输入！";
				} catch (LockedAccountException lae) { // 账号已被锁定
					msg = "该账户已被锁定，如需解锁请联系管理员！";
				} catch (AuthenticationException ae) { // 其他身份验证异常
					msg = "登录异常，请联系管理员！";
				}
			}
	 
			if (subject.isAuthenticated()) {
				return "redirect:/authorized";
			} else {
				model.addAttribute("msg", msg);
				return "403";
			}
	 
		}
	 
		@GetMapping("/logout")
		public String logout() {
			return "redirect:/login";
		}
	 
		@GetMapping("/unauthorized")
		public String unauthorized(ModelMap model) {
			return "403";
		}
	 
		@GetMapping("/authorized")
		public String authorized(ModelMap model) {
			addUserInfo2Model(model);
			return "success";
		}
	 
		// 将用户信息添加到 model
		private void addUserInfo2Model(ModelMap model) {
			Subject subject = SecurityUtils.getSubject();
			UserEntity currentUser = (UserEntity) subject.getPrincipal();
			model.addAttribute("user", currentUser);
		}
	}
```

# 前端页面
## login.html

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/07.png "Shiro示意图")
<div align=left>

完整代码：
```Html
	<!DOCTYPE HTML>
	<html>
	<style type="text/css">
	input {
		text-align: center;
	}
	</style>
	<head>
	<meta http-equiv="Content-Type" content="text/html;charset=utf-8">
	</head>
	<body>
		<form action="/login" method="post">
			<p style="padding: 5px;">
				账号：<input type="text" placeholder="请输入登录账号" name="username">
			</p>
			<p style="padding: 5px;">
				密码：<input type="password" placeholder="请输入登录密码" name="password">
			</p>
			<input type="submit" value="登录">
	 
		</form>
	</body>
	</html>
```

## success.html
```Html
	<!DOCTYPE HTML>
	<HTML>
	<head>
	<meta http-equiv="Content-Type" content="text/html;charset=utf-8">
	</head>
	<body>
		<h3>
			<span th:text="${user.nickName}"></span>,欢迎您!
		</h3>
		<p>
			<a href="/logout">退出登录</a>
		</p>
	</body>
	</HTML>
```

## 403.html
```Html
	<!DOCTYPE HTML>
	<HTML>
	<head>
	<meta http-equiv="Content-Type" content="text/html;charset=utf-8">
	</head>
	<body>
		<p>
			<span style="color:red;" th:text="${msg}"></span>
		</p>
		<h3>您无权访问该页面，请先登录！</h3>
		<p>
			<a href="/login">去登录</a>
		</p>
	</body>
	</HTML>
```

# 登录测试

## 账号与密码均正确
使用 graython 账号登录，输入正确的密码，登录成功。 

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/08.png "Shiro示意图")
<div align=left>

## 账号不存在
随便输入一个不存在的账号进行登录，登录失败。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/09.png "Shiro示意图")
<div align=left>

## 密码错误
使用 graython 账号登录，输入错误的密码，登录失败。 

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/10.png "Shiro示意图")
<div align=left>

## 账号被锁定

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/11.png "Shiro示意图")
<div align=left>

> 注意：由于启用了Shiro缓存，用户成功登录之后，在不退出登录的情况下，再次请求登录页面重新登录该用户，此时，无论密码是否输入正确都会登录成功。