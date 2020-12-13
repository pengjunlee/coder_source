---
title: SpringBoot框架整合之--如何优雅地集成Shiro与JWT
date: 2020-07-24 14:17:00
updated: 2020-07-24 14:17:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何如何优雅地集成Shiro与JWT?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img17.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img17.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
关于 JWT 是什么，请参考[官网](https://jwt.io/introduction/ "官网")或者 [《10分钟了解JSON Web令牌（JWT）》](https://blog.csdn.net/pengjunlee/article/details/95338554 "10分钟了解JSON Web令牌（JWT）") 。这里就不多做介绍了，直接开始我们今天的项目。

# 鉴权流程

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/springboot/61.png "Shiro示意图")
<div align=left>

# 项目搭建
项目的完整目录层次如下图所示。
<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/springboot/62.png "Shiro示意图")
<div align=left>

## pom.xml
在工程的POM文件中引入Shiro和JWT依赖。
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
		<!-- 配置 JWT -->
		<dependency>
			<groupId>com.auth0</groupId>
			<artifactId>java-jwt</artifactId>
			<version>3.4.1</version>
		</dependency>
	</dependencies>
```

## 配置Shiro
本例中的 ShiroConfig.java 与一般Shiro项目的配置有以下几点不同：

- 禁用Session；
- 增加自定义jwtFilter过滤器，用来拦截并处理携带JWT token的请求；
- 使用自定义的MultiRealmAuthenticator多Realm认证器，解决认证异常无法正常返回的问题；
- JwtRealm和ShiroRealm双Realm，其中，JwtRealm用来处理使用JWT token验证身份的请求；

```Java
	import java.util.ArrayList;
	import java.util.LinkedHashMap;
	import java.util.List;
	import java.util.Map;
	 
	import javax.servlet.Filter;
	 
	import org.apache.shiro.authc.credential.CredentialsMatcher;
	import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
	import org.apache.shiro.authc.pam.AuthenticationStrategy;
	import org.apache.shiro.authc.pam.FirstSuccessfulStrategy;
	import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
	import org.apache.shiro.mgt.DefaultSessionStorageEvaluator;
	import org.apache.shiro.mgt.DefaultSubjectDAO;
	import org.apache.shiro.mgt.SecurityManager;
	import org.apache.shiro.mgt.SessionStorageEvaluator;
	import org.apache.shiro.realm.Realm;
	import org.apache.shiro.spring.LifecycleBeanPostProcessor;
	import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
	import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
	import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
	import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
	import org.springframework.boot.web.servlet.FilterRegistrationBean;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	 
	import com.pengjunlee.jwt.JwtFilter;
	 
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
		 * 为 Spring-Bean 开启对 Shiro 注解的支持
		 */
		@Bean
		public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
			AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
			authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
			return authorizationAttributeSourceAdvisor;
		}
	 
		@Bean
		public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
			DefaultAdvisorAutoProxyCreator app = new DefaultAdvisorAutoProxyCreator();
			app.setProxyTargetClass(true);
			return app;
	 
		}
	 
		/**
		 * 不向 Spring容器中注册 JwtFilter Bean，防止 Spring 将 JwtFilter 注册为全局过滤器
		 * 全局过滤器会对所有请求进行拦截，而本例中只需要拦截除 /login 和 /logout 外的请求 
		 * 另一种简单做法是：直接去掉 jwtFilter()上的 @Bean 注解
		 */
		@Bean
		public FilterRegistrationBean<Filter> registration(JwtFilter filter) {
			FilterRegistrationBean<Filter> registration = new FilterRegistrationBean<Filter>(filter);
			registration.setEnabled(false);
			return registration;
		}
	 
		@Bean
		public JwtFilter jwtFilter() {
			return new JwtFilter();
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
	 
			// 添加 jwt 专用过滤器，拦截除 /login 和 /logout 外的请求
			Map<String, Filter> filterMap = new LinkedHashMap<>();
			filterMap.put("jwtFilter", jwtFilter());
			shiroFilterFactoryBean.setFilters(filterMap);
	 
			LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
			filterChainDefinitionMap.put("/login", "anon"); // 可匿名访问
			filterChainDefinitionMap.put("/logout", "logout"); // 退出登录
			filterChainDefinitionMap.put("/**", "jwtFilter,authc"); // 需登录才能访问
			shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
			return shiroFilterFactoryBean;
		}
	 
		/**
		 * 配置 ModularRealmAuthenticator
		 */
		@Bean
		public ModularRealmAuthenticator authenticator() {
			ModularRealmAuthenticator authenticator = new MultiRealmAuthenticator();
			// 设置多 Realm的认证策略，默认 AtLeastOneSuccessfulStrategy
			AuthenticationStrategy strategy = new FirstSuccessfulStrategy();
			authenticator.setAuthenticationStrategy(strategy);
			return authenticator;
		}
	 
		/**
		 * 禁用session, 不保存用户登录状态。保证每次请求都重新认证
		 */
		@Bean
		protected SessionStorageEvaluator sessionStorageEvaluator() {
			DefaultSessionStorageEvaluator sessionStorageEvaluator = new DefaultSessionStorageEvaluator();
			sessionStorageEvaluator.setSessionStorageEnabled(false);
			return sessionStorageEvaluator;
		}
	 
		/**
		 * JwtRealm 配置，需实现 Realm 接口
		 */
		@Bean
		JwtRealm jwtRealm() {
			JwtRealm jwtRealm = new JwtRealm();
			// 设置加密算法
			CredentialsMatcher credentialsMatcher = new JwtCredentialsMatcher();
			// 设置加密次数
			jwtRealm.setCredentialsMatcher(credentialsMatcher);
			return jwtRealm;
		}
	 
		/**
		 * ShiroRealm 配置，需实现 Realm 接口
		 */
		@Bean
		ShiroRealm shiroRealm() {
			ShiroRealm shiroRealm = new ShiroRealm();
			// 设置加密算法
			HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher("SHA-1");
			// 设置加密次数
			credentialsMatcher.setHashIterations(16);
			shiroRealm.setCredentialsMatcher(credentialsMatcher);
			return shiroRealm;
		}
	 
		/**
		 * 配置 SecurityManager
		 */
		@Bean
		public SecurityManager securityManager() {
			DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
	 
			// 1.Authenticator
			securityManager.setAuthenticator(authenticator());
	 
			// 2.Realm
			List<Realm> realms = new ArrayList<Realm>(16);
			realms.add(jwtRealm());
			realms.add(shiroRealm());
			securityManager.setRealms(realms);
	 
			// 3.关闭shiro自带的session
			DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
			subjectDAO.setSessionStorageEvaluator(sessionStorageEvaluator());
			securityManager.setSubjectDAO(subjectDAO);
	 
			return securityManager;
		}
	}
```

接下来，本文将安装整个鉴权流程对各部分的代码进行详细讲解。

# 用户登录
## LoginController.java
根据ShiroConfig中FilterChainDefinitionMap的配置，/login 请求是不会被 jwtFilter 过滤器拦截的。Shiro验证用户名和密码正确后完成登录，同时生成 JWT token 签名，并随Response一起返回。
```Java
	import javax.servlet.ServletResponse;
	import javax.servlet.http.HttpServletResponse;
	 
	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.IncorrectCredentialsException;
	import org.apache.shiro.authc.LockedAccountException;
	import org.apache.shiro.authc.UnknownAccountException;
	import org.apache.shiro.authc.UsernamePasswordToken;
	import org.apache.shiro.subject.Subject;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.PostMapping;
	import org.springframework.web.bind.annotation.RequestParam;
	import org.springframework.web.bind.annotation.RestController;
	 
	import com.pengjunlee.domain.BaseResponse;
	import com.pengjunlee.jwt.JwtUtils;
	 
	@RestController
	public class LoginController {
	 
		@PostMapping(value = "/login")
		public Object userLogin(@RequestParam(name = "username", required = true) String userName,
				@RequestParam(name = "password", required = true) String password, ServletResponse response) {
	 
			// 获取当前用户主体
			Subject subject = SecurityUtils.getSubject();
			String msg = null;
			boolean loginSuccess = false;
			// 将用户名和密码封装成 UsernamePasswordToken 对象
			UsernamePasswordToken token = new UsernamePasswordToken(userName, password);
			try {
				subject.login(token);
				msg = "登录成功。";
				loginSuccess = true;
			} catch (UnknownAccountException uae) { // 账号不存在
				msg = "用户名与密码不匹配，请检查后重新输入！";
			} catch (IncorrectCredentialsException ice) { // 账号与密码不匹配
				msg = "用户名与密码不匹配，请检查后重新输入！";
			} catch (LockedAccountException lae) { // 账号已被锁定
				msg = "该账户已被锁定，如需解锁请联系管理员！";
			} catch (AuthenticationException ae) { // 其他身份验证异常
				msg = "登录异常，请联系管理员！";
			}
			BaseResponse<Object> ret = new BaseResponse<Object>();
			if (loginSuccess) {
				// 若登录成功，签发 JWT token
				String jwtToken = JwtUtils.sign(userName, JwtUtils.SECRET);
				// 将签发的 JWT token 设置到 HttpServletResponse 的 Header 中
				((HttpServletResponse) response).setHeader(JwtUtils.AUTH_HEADER, jwtToken);
				// 
				ret.setErrCode(0);
				ret.setMsg(msg);
				return ret;
			} else {
				ret.setErrCode(401);
				ret.setMsg(msg);
				return ret;
			}
	 
		}
	 
		@GetMapping("/logout")
		public Object logout() {
			BaseResponse<Object> ret = new BaseResponse<Object>();
			ret.setErrCode(0);
			ret.setMsg("退出登录");
			return ret;
		}
	}
```

## ShiroRealm.java
由于用户登录时 subject.login(token) 方法中 token 的类型为 UsernamePasswordToken ，所以会进入 ShiroRealm 查询数据库获取用户信息。
```Java
	import java.util.HashMap;
	import java.util.HashSet;
	import java.util.Map;
	import java.util.Set;
	 
	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.AuthenticationInfo;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.authc.LockedAccountException;
	import org.apache.shiro.authc.SimpleAuthenticationInfo;
	import org.apache.shiro.authc.UnknownAccountException;
	import org.apache.shiro.authc.UsernamePasswordToken;
	import org.apache.shiro.authz.AuthorizationInfo;
	import org.apache.shiro.authz.SimpleAuthorizationInfo;
	import org.apache.shiro.realm.AuthorizingRealm;
	import org.apache.shiro.subject.PrincipalCollection;
	import org.apache.shiro.util.ByteSource;
	 
	import com.pengjunlee.domain.UserEntity;
	 
	/**
	 * 同时开启身份验证和权限验证，需要继承 AuthorizingRealm 
	 * 并实现其  doGetAuthenticationInfo()和 doGetAuthorizationInfo 两个方法
	 */
	@SuppressWarnings("serial")
	public class ShiroRealm extends AuthorizingRealm {
	 
		public static Map<String, UserEntity> userMap = new HashMap<String, UserEntity>(16);
		public static Map<String, Set<String>> roleMap = new HashMap<String, Set<String>>(16);
		public static Map<String, Set<String>> permMap = new HashMap<String, Set<String>>(16);
	 
		static {
			UserEntity user1 = new UserEntity(1L, "graython", "dd524c4c66076d1fa07e1fa1c94a91233772d132", "灰先生", false);
			UserEntity user2 = new UserEntity(2L, "plum", "cce369436bbb9f0325689a3a6d5d6b9b8a3f39a0", "李先生", false);
	 
			userMap.put("graython", user1);
			userMap.put("plum", user2);
	 
			roleMap.put("graython", new HashSet<String>() {
				{
					add("admin");
	 
				}
			});
			
			roleMap.put("plum", new HashSet<String>() {
				{
					add("guest");
				}
			});
			permMap.put("plum", new HashSet<String>() {
				{
					add("article:read");
				}
			});
		}
	 
		/**
		 * 限定这个 Realm 只处理 UsernamePasswordToken
		 */
		@Override
		public boolean supports(AuthenticationToken token) {
			return token instanceof UsernamePasswordToken;
		}
		
		/**
		 * 查询数据库，将获取到的用户安全数据封装返回
		 */
		@Override
		protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
			// 从 AuthenticationToken 中获取当前用户
			String username = (String) token.getPrincipal();
			// 查询数据库获取用户信息，此处使用 Map 来模拟数据库
			UserEntity user = userMap.get(username);
	 
			// 用户不存在
			if (user == null) {
				throw new UnknownAccountException("用户不存在！");
			}
	 
			// 用户被锁定
			if (user.getLocked()) {
				throw new LockedAccountException("该用户已被锁定,暂时无法登录！");
			}
	 
			// 使用用户名作为盐值
			ByteSource credentialsSalt = ByteSource.Util.bytes(username);
	 
			/**
			 * 将获取到的用户数据封装成 AuthenticationInfo 对象返回，此处封装为 SimpleAuthenticationInfo 对象。
			 *  参数1. 认证的实体信息，可以是从数据库中获取到的用户实体类对象或者用户名 
			 *  参数2. 查询获取到的登录密码 
			 *  参数3. 盐值
			 *  参数4. 当前 Realm 对象的名称，直接调用父类的 getName() 方法即可
			 */
			SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getPassword(), credentialsSalt,
					getName());
			return info;
		}
	 
		/**
		 * 查询数据库，将获取到的用户的角色及权限信息返回
		 */
		@Override
		protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
			// 获取当前用户
			UserEntity currentUser = (UserEntity) SecurityUtils.getSubject().getPrincipal();
			// UserEntity currentUser = (UserEntity)principals.getPrimaryPrincipal();
			// 查询数据库，获取用户的角色信息
			Set<String> roles = roleMap.get(currentUser.getName());
			// 查询数据库，获取用户的权限信息
			Set<String> perms = permMap.get(currentUser.getName());
	 
			SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
			info.setRoles(roles);
			info.setStringPermissions(perms);
			return info;
		}
	 
	}
```
<br/>
```Java
	//加密密码
    String password = new SimpleHash("MD5", user.getPassword(), user.getUserName(), 1024).toString();
```

## BaseResponse.java
为了方便演示，本例中所有请求均返回Json响应。
```Java
	import java.io.Serializable;
	 
	public class BaseResponse<T> implements Serializable {
	 
		private static final long serialVersionUID = 1L;
	 
		/**
		 * 状态码
		 */
		private int errCode;
	 
		/**
		 * 消息内容
		 */
		private String msg;
	 
		/**
		 * 返回数据
		 */
		private T data;
	 
		public BaseResponse() {
			super();
		}
	 
		public BaseResponse(int errCode, String msg, T data) {
			super();
			this.errCode = errCode;
			this.msg = msg;
			this.data = data;
		}
	 
		public int getErrCode() {
			return errCode;
		}
	 
		public void setErrCode(int errCode) {
			this.errCode = errCode;
		}
	 
		public String getMsg() {
			return msg;
		}
	 
		public void setMsg(String msg) {
			this.msg = msg;
		}
	 
		public T getData() {
			return data;
		}
	 
		public void setData(T data) {
			this.data = data;
		}
	 
	}
```

## UserEntity.java
UserEntity 用户实体类定义如下。
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
	 
		public String getPassword() {
			return password;
		}
	 
		public void setPassword(String password) {
			this.password = password;
		}
	 
		public String getNickName() {
			return nickName;
		}
	 
		public void setNickName(String nickName) {
			this.nickName = nickName;
		}
	 
		public Boolean getLocked() {
			return locked;
		}
	 
		public void setLocked(Boolean locked) {
			this.locked = locked;
		}
	 
	}
```

# 签发Token
我们将 Token 的常用操作封装到 JwtUtils 工具类中。
```Java
	import java.util.Calendar;
	import java.util.Date;
	import java.util.Map;
	import java.util.Map.Entry;
	 
	import org.apache.shiro.crypto.SecureRandomNumberGenerator;
	 
	import com.auth0.jwt.JWT;
	import com.auth0.jwt.JWTCreator.Builder;
	import com.auth0.jwt.JWTVerifier;
	import com.auth0.jwt.algorithms.Algorithm;
	import com.auth0.jwt.exceptions.JWTCreationException;
	import com.auth0.jwt.exceptions.JWTDecodeException;
	import com.auth0.jwt.exceptions.JWTVerificationException;
	import com.auth0.jwt.interfaces.Claim;
	import com.auth0.jwt.interfaces.DecodedJWT;
	 
	public class JwtUtils {
	 
		// 过期时间5分钟
		private static final long EXPIRE_TIME = 5 * 60 * 1000;
	 
		// 私钥
		public static final String SECRET = "SECRET_VALUE";
	 
		// 请求头
		public static final String AUTH_HEADER = "X-Authorization-With";
	 
		/**
		 * 验证token是否正确
		 */
		public static boolean verify(String token, String username, String secret) {
			try {
				Algorithm algorithm = Algorithm.HMAC256(secret);
				JWTVerifier verifier = JWT.require(algorithm).withClaim("username", username).build();
				verifier.verify(token);
				return true;
			} catch (JWTVerificationException exception) {
				return false;
			}
		}
	 
		/**
		 * 获得token中的自定义信息，无需secret解密也能获得
		 */
		public static String getClaimFiled(String token, String filed) {
			try {
				DecodedJWT jwt = JWT.decode(token);
				return jwt.getClaim(filed).asString();
			} catch (JWTDecodeException e) {
				return null;
			}
		}
	 
		/**
		 * 生成签名
		 */
		public static String sign(String username, String secret) {
			try {
				Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
				Algorithm algorithm = Algorithm.HMAC256(secret);
				// 附带username，nickname信息
				return JWT.create().withClaim("username", username).withExpiresAt(date).sign(algorithm);
			} catch (JWTCreationException e) {
				return null;
			}
		}
	 
		/**
		 * 获取 token的签发时间
		 */
		public static Date getIssuedAt(String token) {
			try {
				DecodedJWT jwt = JWT.decode(token);
				return jwt.getIssuedAt();
			} catch (JWTDecodeException e) {
				return null;
			}
		}
	 
		/**
		 * 验证 token是否过期
		 */
		public static boolean isTokenExpired(String token) {
			Date now = Calendar.getInstance().getTime();
			DecodedJWT jwt = JWT.decode(token);
			return jwt.getExpiresAt().before(now);
		}
	 
		/**
		 * 刷新 token的过期时间
		 */
		public static String refreshTokenExpired(String token, String secret) {
			DecodedJWT jwt = JWT.decode(token);
			Map<String, Claim> claims = jwt.getClaims();
			try {
				Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
				Algorithm algorithm = Algorithm.HMAC256(secret);
				Builder builer = JWT.create().withExpiresAt(date);
				for (Entry<String, Claim> entry : claims.entrySet()) {
					builer.withClaim(entry.getKey(), entry.getValue().asString());
				}
				return builer.sign(algorithm);
			} catch (JWTCreationException e) {
				return null;
			}
		}
	 
		/**
		 * 生成16位随机盐
		 */
		public static String generateSalt() {
			SecureRandomNumberGenerator secureRandom = new SecureRandomNumberGenerator();
			String hex = secureRandom.nextBytes(16).toHex();
			return hex;
		}
	}
```

# 再次请求

## ArticleController.java
ArticleController有两个示例接口：访问 /article/delete 需要 admin 角色；访问 /article/read 需要 article:read 权限。
```Java
	import org.apache.shiro.authz.annotation.RequiresPermissions;
	import org.apache.shiro.authz.annotation.RequiresRoles;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.bind.annotation.GetMapping;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	 
	import com.pengjunlee.domain.BaseResponse;
	 
	@RestController
	@RequestMapping("/article")
	public class ArticleController {
	 
		@GetMapping("/delete")
		@RequiresRoles(value = { "admin" })
		public Object deleteArticle(ModelMap model) {
			BaseResponse<Object> ret = new BaseResponse<Object>();
			ret.setErrCode(0);
			ret.setMsg("文章删除成功！");
			return ret;
		}
	 
		@GetMapping("/read")
		@RequiresPermissions(value = { "article:read" })
		public Object readArticle(ModelMap model) {
			BaseResponse<Object> ret = new BaseResponse<Object>();
			ret.setErrCode(0);
			ret.setMsg("请您鉴赏！");
			return ret;
		}
	 
	}
```

## JwtFilter.java
根据ShiroConfig中FilterChainDefinitionMap的配置，/article/delete 和 /article/read 两个请求都会被 jwtFilter 过滤器拦截。

jwtFilter 先检查请求头中是否包含 JWT token，若不包含直接拒绝访问。反之，jwtFilter 会将请求头中包含的 JWT token 封装成 JwtToken 对象，并调用 subject.login(token) 方法交给Shiro去进行登录判断。
```Java
	import java.io.PrintWriter;
	 
	import javax.servlet.ServletRequest;
	import javax.servlet.ServletResponse;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	 
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.subject.Subject;
	import org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter;
	import org.apache.shiro.web.util.WebUtils;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.http.HttpStatus;
	import org.springframework.web.bind.annotation.RequestMethod;
	 
	/**
	 * 自定义的认证过滤器，用来拦截Header中携带 JWT token的请求
	 */
	public class JwtFilter extends BasicHttpAuthenticationFilter {
	 
		private Logger log = LoggerFactory.getLogger(this.getClass());
	 
		/**
		 * 前置处理
		 */
		@Override
		protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
			HttpServletRequest httpServletRequest = WebUtils.toHttp(request);
			HttpServletResponse httpServletResponse = WebUtils.toHttp(response);
			// 跨域时会首先发送一个option请求，这里我们给option请求直接返回正常状态
			if (httpServletRequest.getMethod().equals(RequestMethod.OPTIONS.name())) {
				httpServletResponse.setStatus(HttpStatus.OK.value());
				return false;
			}
			return super.preHandle(request, response);
		}
	 
		/**
		 * 后置处理
		 */
		@Override
		protected void postHandle(ServletRequest request, ServletResponse response) {
			// 添加跨域支持
			this.fillCorsHeader(WebUtils.toHttp(request), WebUtils.toHttp(response));
		}
	 
		/**
		 * 过滤器拦截请求的入口方法 
		 * 返回 true 则允许访问 
		 * 返回false 则禁止访问，会进入 onAccessDenied()
		 */
		@Override
		protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
			// 原用来判断是否是登录请求，在本例中不会拦截登录请求，用来检测Header中是否包含 JWT token 字段
			if (this.isLoginRequest(request, response))
				return false;
			boolean allowed = false;
			try {
				// 检测Header里的 JWT token内容是否正确，尝试使用 token进行登录
				allowed = executeLogin(request, response);
			} catch (IllegalStateException e) { // not found any token
				log.error("Not found any token");
			} catch (Exception e) {
				log.error("Error occurs when login", e);
			}
			return allowed || super.isPermissive(mappedValue);
		}
	 
		/**
		 * 检测Header中是否包含 JWT token 字段
		 */
		@Override
		protected boolean isLoginAttempt(ServletRequest request, ServletResponse response) {
			return ((HttpServletRequest) request).getHeader(JwtUtils.AUTH_HEADER) == null;
		}
	 
		/**
		 * 身份验证,检查 JWT token 是否合法
		 */
		@Override
		protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
			AuthenticationToken token = createToken(request, response);
			if (token == null) {
				String msg = "createToken method implementation returned null. A valid non-null AuthenticationToken "
						+ "must be created in order to execute a login attempt.";
				throw new IllegalStateException(msg);
			}
			try {
				Subject subject = getSubject(request, response);
				subject.login(token); // 交给 Shiro 去进行登录验证
				return onLoginSuccess(token, subject, request, response);
			} catch (AuthenticationException e) {
				return onLoginFailure(token, e, request, response);
			}
		}
	 
		/**
		 * 从 Header 里提取 JWT token
		 */
		@Override
		protected AuthenticationToken createToken(ServletRequest servletRequest, ServletResponse servletResponse) {
			HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
			String authorization = httpServletRequest.getHeader(JwtUtils.AUTH_HEADER);
			JwtToken token = new JwtToken(authorization);
			return token;
		}
	 
		/**
		 * isAccessAllowed()方法返回false，会进入该方法，表示拒绝访问
		 */
		@Override
		protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
			HttpServletResponse httpResponse = WebUtils.toHttp(servletResponse);
			httpResponse.setCharacterEncoding("UTF-8");
			httpResponse.setContentType("application/json;charset=UTF-8");
			httpResponse.setStatus(HttpStatus.UNAUTHORIZED.value());
			PrintWriter writer = httpResponse.getWriter();
			writer.write("{\"errCode\": 401, \"msg\": \"UNAUTHORIZED\"}");
			fillCorsHeader(WebUtils.toHttp(servletRequest), httpResponse);
			return false;
		}
	 
		/**
		 * Shiro 利用 JWT token 登录成功，会进入该方法
		 */
		@Override
		protected boolean onLoginSuccess(AuthenticationToken token, Subject subject, ServletRequest request,
				ServletResponse response) throws Exception {
			HttpServletResponse httpResponse = WebUtils.toHttp(response);
			String newToken = null;
			if (token instanceof JwtToken) {
				newToken = JwtUtils.refreshTokenExpired(token.getCredentials().toString(), JwtUtils.SECRET);
			}
			if (newToken != null)
				httpResponse.setHeader(JwtUtils.AUTH_HEADER, newToken);
			return true;
		}
	 
		/**
		 * Shiro 利用 JWT token 登录失败，会进入该方法
		 */
		@Override
		protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, ServletRequest request,
				ServletResponse response) {
			// 此处直接返回 false ，交给后面的  onAccessDenied()方法进行处理
			return false;
		}
	 
		/**
		 * 添加跨域支持
		 */
		protected void fillCorsHeader(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) {
			httpServletResponse.setHeader("Access-control-Allow-Origin", httpServletRequest.getHeader("Origin"));
			httpServletResponse.setHeader("Access-Control-Allow-Methods", "GET,POST,OPTIONS,HEAD");
			httpServletResponse.setHeader("Access-Control-Allow-Headers",
					httpServletRequest.getHeader("Access-Control-Request-Headers"));
		}
	}
```

## JwtToken.java
JwtToken 和 UsernamePasswordToken 差不多，都是 AuthenticationToken 接口的实现类。
```Java
	import org.apache.shiro.authc.AuthenticationToken;
	 
	public class JwtToken implements AuthenticationToken {
	 
		private static final long serialVersionUID = 1L;
	 
		// 加密后的 JWT token串
		private String token;
	 
		private String userName;
	 
		public JwtToken(String token) {
			this.token = token;
			this.userName = JwtUtils.getClaimFiled(token, "username");
		}
	 
		@Override
		public Object getPrincipal() {
			return this.userName;
		}
	 
		@Override
		public Object getCredentials() {
			return token;
		}
	}
```

## JwtRealm.java
这次由于用户登录时 subject.login(token) 方法中 token 的类型为 JwtToken ，所以会由 JwtRealm 进行处理。
```Java
	import java.util.Set;
	 
	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.authc.AccountException;
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.AuthenticationInfo;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.authc.LockedAccountException;
	import org.apache.shiro.authc.SimpleAuthenticationInfo;
	import org.apache.shiro.authc.UnknownAccountException;
	import org.apache.shiro.authz.AuthorizationInfo;
	import org.apache.shiro.authz.SimpleAuthorizationInfo;
	import org.apache.shiro.realm.AuthorizingRealm;
	import org.apache.shiro.subject.PrincipalCollection;
	 
	import com.pengjunlee.domain.UserEntity;
	import com.pengjunlee.jwt.JwtToken;
	 
	/**
	 * JwtRealm 只负责校验 JwtToken
	 */
	public class JwtRealm extends AuthorizingRealm {
	 
		/**
		 * 限定这个 Realm 只处理我们自定义的 JwtToken
		 */
		@Override
		public boolean supports(AuthenticationToken token) {
			return token instanceof JwtToken;
		}
	 
		/**
		 * 此处的 SimpleAuthenticationInfo 可返回任意值，密码校验时不会用到它
		 */
		@Override
		protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken)
				throws AuthenticationException {
			JwtToken jwtToken = (JwtToken) authcToken;
			if (jwtToken.getPrincipal() == null) {
				throw new AccountException("JWT token参数异常！");
			}
			// 从 JwtToken 中获取当前用户
			String username = jwtToken.getPrincipal().toString();
			// 查询数据库获取用户信息，此处使用 Map 来模拟数据库
			UserEntity user = ShiroRealm.userMap.get(username);
	 
			// 用户不存在
			if (user == null) {
				throw new UnknownAccountException("用户不存在！");
			}
	 
			// 用户被锁定
			if (user.getLocked()) {
				throw new LockedAccountException("该用户已被锁定,暂时无法登录！");
			}
	 
			SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, username, getName());
			return info;
		}
	 
		@Override
		protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
			// 获取当前用户
			UserEntity currentUser = (UserEntity) SecurityUtils.getSubject().getPrincipal();
			// UserEntity currentUser = (UserEntity) principals.getPrimaryPrincipal();
			// 查询数据库，获取用户的角色信息
			Set<String> roles = ShiroRealm.roleMap.get(currentUser.getName());
			// 查询数据库，获取用户的权限信息
			Set<String> perms = ShiroRealm.permMap.get(currentUser.getName());
			SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
			info.setRoles(roles);
			info.setStringPermissions(perms);
			return info;
		}
	 
	}
```

## JwtCredentialsMatcher.java
跟 ShiroRealm 不一样，JwtRealm 不需要拿传入的 JwtToken 和其他的 Token 去做比对，只需验证JwtToken自身的内容是否合法即可。所以，我们需要为 JwtRealm 自定义一个 CredentialsMatcher 实现。
```Java
	import org.apache.shiro.authc.AuthenticationInfo;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.authc.credential.CredentialsMatcher;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	 
	import com.auth0.jwt.JWT;
	import com.auth0.jwt.JWTVerifier;
	import com.auth0.jwt.algorithms.Algorithm;
	import com.auth0.jwt.exceptions.JWTVerificationException;
	import com.pengjunlee.jwt.JwtUtils;
	 
	public class JwtCredentialsMatcher implements CredentialsMatcher {
	 
		private Logger logger = LoggerFactory.getLogger(this.getClass());
	 
		/**
		 * JwtCredentialsMatcher只需验证JwtToken内容是否合法
		 */
		@Override
		public boolean doCredentialsMatch(AuthenticationToken authenticationToken, AuthenticationInfo authenticationInfo) {
	 
			String token = authenticationToken.getCredentials().toString();
			String username = authenticationToken.getPrincipal().toString();
			try {
				Algorithm algorithm = Algorithm.HMAC256(JwtUtils.SECRET);
				JWTVerifier verifier = JWT.require(algorithm).withClaim("username", username).build();
				verifier.verify(token);
				return true;
			} catch (JWTVerificationException e) {
				logger.error(e.getMessage());
			}
			return false;
		}
	 
	}
```

## MultiRealmAuthenticator.java
MultiRealmAuthenticator 用来解决Shiro中出现的具体的认证异常无法正常返回，仅返回父类 AuthenticationException 的问题。
```Java
	import java.util.Collection;
	 
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.AuthenticationInfo;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.authc.pam.AuthenticationStrategy;
	import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
	import org.apache.shiro.realm.Realm;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	 
	/**
	 * 自定义认证器，解决 Shiro 异常无法返回的问题
	 */
	public class MultiRealmAuthenticator extends ModularRealmAuthenticator {
	 
		private static final Logger log = LoggerFactory.getLogger(MultiRealmAuthenticator.class);
	 
		@Override
		protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token)
				throws AuthenticationException {
			AuthenticationStrategy strategy = getAuthenticationStrategy();
	 
			AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);
	 
			if (log.isTraceEnabled()) {
				log.trace("Iterating through {} realms for PAM authentication", realms.size());
			}
			AuthenticationException authenticationException = null;
			for (Realm realm : realms) {
	 
				aggregate = strategy.beforeAttempt(realm, token, aggregate);
	 
				if (realm.supports(token)) {
	 
					log.trace("Attempting to authenticate token [{}] using realm [{}]", token, realm);
	 
					AuthenticationInfo info = null;
					try {
						info = realm.getAuthenticationInfo(token);
					} catch (AuthenticationException e) {
						authenticationException = e;
						if (log.isDebugEnabled()) {
							String msg = "Realm [" + realm
									+ "] threw an exception during a multi-realm authentication attempt:";
							log.debug(msg, e);
						}
					}
	 
					aggregate = strategy.afterAttempt(realm, token, info, aggregate, authenticationException);
	 
				} else {
					log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
				}
			}
			if (authenticationException != null) {
				throw authenticationException;
			}
			aggregate = strategy.afterAllAttempts(token, aggregate);
	 
			return aggregate;
		}
	}
```

## ExceptionController.java
ExceptionController 负责对 Controller中抛出的异常进行捕获处理。
```Java
	import javax.servlet.http.HttpServletRequest;
	 
	import org.apache.shiro.ShiroException;
	import org.springframework.web.bind.annotation.ExceptionHandler;
	import org.springframework.web.bind.annotation.RestControllerAdvice;
	 
	import com.pengjunlee.domain.BaseResponse;
	 
	@RestControllerAdvice
	/**
	 * 处理全局异常
	 */
	public class ExceptionController {
	 
		// 捕捉shiro的异常
		@ExceptionHandler(ShiroException.class)
		public Object handleShiroException(ShiroException e) {
			BaseResponse<Object> ret = new BaseResponse<Object>();
			ret.setErrCode(401);
			ret.setMsg(e.getMessage());
			return ret;
		}
	 
		// 捕捉其他所有异常
		@ExceptionHandler(Exception.class)
		public Object globalException(HttpServletRequest request, Throwable ex) {
			BaseResponse<Object> ret = new BaseResponse<Object>();
			ret.setErrCode(401);
			ret.setMsg(ex.getMessage());
			return ret;
		}
	}
```

# 请求测试
## 登录成功
输入正确的用户名和密码，登录成功。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/springboot/63.png "Shiro示意图")
<div align=left>

查看响应头中返回的 JWT token。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/springboot/64.png "Shiro示意图")
<div align=left>

## 登录失败
输入正确的用户名和错误的密码，登录失败。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/springboot/65.png "Shiro示意图")
<div align=left>

## 阅读文章
使用graython账号登录，不具备文章阅读权限。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/springboot/66.png "Shiro示意图")
<div align=left>

## 删除文章
使用graython账号登录，可以删除文章。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/springboot/67.png "Shiro示意图")
<div align=left>

项目地址：<https://github.com/pengjunlee/shiro-jwt.git>
参考文章:<https://www.jianshu.com/p/0b1131be7ace>