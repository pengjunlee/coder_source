---
title: 安全框架系列之--Shiro常用技巧
date: 2020-08-01 14:11:00
updated: 2020-08-01 14:11:00
tags: Shiro
categories: 安全框架
keywords: 安全, Shiro
type: 
description: Shiro常用技巧。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
aside: true
toc: true
toc_number: true
auto_open: true
copyright: true
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# 会话管理
Shiro 提供了完整的企业级会话管理功能，不依赖于底层容器（如web容器tomcat），不管 JavaSE 还是 JavaEE 环境都可以使用，提供了会话管理、会话事件监听、会话存储/持久化、容器无关的集群、失效/过期支持、对Web 的透明支持、 SSO 单点登录的支持等特性。 

为了方便我们使用Session，Shiro提供了下面几个类：

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/20.png "Shiro示意图")
<div align=left>

- AbstractSessionDAO 提供了 SessionDAO 的基础实现，如生成会话ID等
- CachingSessionDAO 提供了对开发者透明的会话缓存的功能，需要设置相应的 CacheManager
- MemorySessionDAO 直接在内存中进行会话维护
- EnterpriseCacheSessionDAO 提供了带缓存功能的会话维护，默认情况下使用 MapCache 实现，内部使用ConcurrentHashMap 保存缓存的会话。

由于 AbstractSessionDAO 需要使用它内部的 SessionIdGenerator 实例来生成会话 ID。因此，在创建SessionDAO 时必须为其指定一个 SessionIdGenerator ，可以使用 Shiro 为我们提供的下面这两个实现类。

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/21.png "Shiro示意图")
<div align=left>

## 启用会话管理
```Java
	/**
	 * 第一步 创建 SessionDAO 的同时设置其 SessionIdGenerator 属性
	 */
	@Bean
	public SessionDAO sessionDAO() {
		EnterpriseCacheSessionDAO sessionDAO = new EnterpriseCacheSessionDAO();
		// 设置 SessionIdGenerator
		sessionDAO.setSessionIdGenerator(new JavaUuidSessionIdGenerator());
		// 需要使用缓存时，设置 CacheManager
		EhCacheManager em = new EhCacheManager();
		em.setCacheManager(CacheManager.create());
		sessionDAO.setCacheManager(em);
		return sessionDAO;
	}
 
	/**
	 * 第二步 创建 SessionManager 的同时设置其 SessionDAO 属性
	 */
	@Bean
	public DefaultSessionManager sessionManager() {
		DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
		sessionManager.setGlobalSessionTimeout(1800 * 1000);
		sessionManager.setDeleteInvalidSessions(true);
		sessionManager.setSessionValidationSchedulerEnabled(true);
		sessionManager.setSessionDAO(sessionDAO());
		// 添加监听器 统计在线人数
		Collection<SessionListener> listeners = new ArrayList<SessionListener>();
		listeners.add(new BDSessionListener());
		sessionManager.setSessionListeners(listeners);
		return sessionManager;
	}
 
	/**
	 * 第三步 创建 SecurityManager 的同时设置其 SessionManager 属性
	 */
	@Bean
	public SecurityManager securityManager() {
		DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
 
		// 1.CacheManager
		securityManager.setCacheManager(ehCacheManager());
 
		// 2. Authenticator
		securityManager.setAuthenticator(authenticator());
 
		// 3.Realm
		List<Realm> realms = new ArrayList<Realm>(16);
		realms.add(loginRealm());
		realms.add(userRealm());
		securityManager.setRealms(realms);
 
		// 4.SessionManager
		securityManager.setSessionManager(sessionManager());
		return securityManager;
	}
```
BDSessionListener 用来监听当前 Session 的存活数量，实现在线人数统计的目的。 
```Java
	import java.util.concurrent.atomic.AtomicInteger;
	 
	import org.apache.shiro.session.Session;
	import org.apache.shiro.session.SessionListener;
	 
	public class BDSessionListener implements SessionListener {
	 
		private final AtomicInteger sessionCount = new AtomicInteger(0);
	 
		@Override
		public void onStart(Session session) {
			sessionCount.incrementAndGet();
		}
	 
		@Override
		public void onStop(Session session) {
			sessionCount.decrementAndGet();
		}
	 
		@Override
		public void onExpiration(Session session) {
			sessionCount.decrementAndGet();
	 
		}
	 
		public int getSessionCount() {
			return sessionCount.get();
		}
	 
	}
```

## 自定义SessionDAO
自定义SessionDAO可以直接继承EnterpriseCacheSessionDAO，覆写它的doReadSession()、doUpdate()、doDelete() 三个空实现方法即可。 
```
	import java.io.Serializable;
	import java.util.List;
	 
	import org.apache.shiro.session.Session;
	import org.apache.shiro.session.mgt.ValidatingSession;
	import org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.jdbc.core.JdbcTemplate;
	 
	public class MySessionDao extends EnterpriseCacheSessionDAO {
	 
		@Autowired
		private JdbcTemplate jdbcTemplate = null;
	 
		@Override
		protected Serializable doCreate(Session session) {
			Serializable sessionId = generateSessionId(session);
			assignSessionId(session, sessionId);
			String sql = "insert into sessions(id, session) values(?,?)";
			jdbcTemplate.update(sql, sessionId, SerializableUtils.serialize(session));
			return session.getId();
		}
	 
		@Override
		protected Session doReadSession(Serializable sessionId) {
			String sql = "select session from sessions where id=?";
			List<String> sessionStrList = jdbcTemplate.queryForList(sql, String.class, sessionId);
			if (sessionStrList.size() == 0)
				return null;
			return SerializableUtils.deserialize(sessionStrList.get(0));
		}
	 
		@Override
		protected void doUpdate(Session session) {
			if (session instanceof ValidatingSession && !((ValidatingSession) session).isValid()) {
				return;
			}
			String sql = "update sessions set session=? where id=?";
			jdbcTemplate.update(sql, SerializableUtils.serialize(session), session.getId());
		}
	 
		@Override
		protected void doDelete(Session session) {
			String sql = "delete from sessions where id=?";
			jdbcTemplate.update(sql, session.getId());
		}
	}
```
<br/>
```Java
	import org.apache.shiro.codec.Base64;
	import org.apache.shiro.session.Session;
	 
	import java.io.ByteArrayInputStream;
	import java.io.ByteArrayOutputStream;
	import java.io.ObjectInputStream;
	import java.io.ObjectOutputStream;
	 
	public class SerializableUtils {
	 
		public static String serialize(Session session) {
			try {
				ByteArrayOutputStream bos = new ByteArrayOutputStream();
				ObjectOutputStream oos = new ObjectOutputStream(bos);
				oos.writeObject(session);
				return Base64.encodeToString(bos.toByteArray());
			} catch (Exception e) {
				throw new RuntimeException("serialize session error", e);
			}
		}
	 
		public static Session deserialize(String sessionStr) {
			try {
				ByteArrayInputStream bis = new ByteArrayInputStream(Base64.decode(sessionStr));
				ObjectInputStream ois = new ObjectInputStream(bis);
				return (Session) ois.readObject();
			} catch (Exception e) {
				throw new RuntimeException("deserialize session error", e);
			}
		}
	 
	}
```
## 会话验证
Shiro 提供了会话验证调度器，用于定期的验证会话是否已过期，如果过期将停止会话；
出于性能考虑，一般情况下都是获取会话时来验证会话是否过期并停止会话的；但是如在 web 环境中，如果用户不主动退出是不知道会话是否过期的，因此需要定期的检测会话是否过期，Shiro 提供了会话验证调度器SessionValidationScheduler ；
Shiro 也提供了使用Quartz会话验证调度器：QuartzSessionValidationScheduler ；

# 关于缓存
Shiro 会自动检测其内部组件（如Realm）是否实现了CacheManagerAware 接口，并自动为其注入相应的CacheManager。CachingRealm 为我们提供了缓存的一些基础实现，前面身份验证提到的AuthenticatingRealm 以及授权中的 AuthorizingRealm 就继承了 CachingRealm 。

因此，若 SecurityManager 实现了 SessionSecurityManager，Shiro 会判断其内部的 SessionManager 是否实现了CacheManagerAware 接口，如果实现了会把 CacheManager 设置给它。同时， SessionManager 也会判断其内部的 SessionDAO（如继承自CachingSessionDAO）是否实现了CacheManagerAware，如果实现了会把 CacheManager设置给它。设置了缓存的 SessionManager，查询时会先查缓存，如果找不到才查数据库。

# 记住我
Shiro 提供了记住我（RememberMe）的功能，比如访问如淘宝等一些网站时，关闭了浏览器，下次再打开时还是能记住你是谁，下次访问时无需再登录即可访问。

## 认证VS记住我
- subject.isAuthenticated() 表示用户进行了身份验证登录的，即使用 Subject.login 进行了登录；
- subject.isRemembered()：表示用户是通过记住我登录的，此时可能并不是真正的你在访问（可能是你的朋友使用你的电脑，或者你的cookie 被窃取）；
- 两者二选一，即 subject.isAuthenticated()==true，则 subject.isRemembered()==false；反之一样。

## 启用记住我
```Java
	/**
	 * 第一步 创建 SimpleCookie
	 */
	@Bean
	public SimpleCookie simpleCookie() {
		SimpleCookie simpleCookie = new SimpleCookie();
		simpleCookie.setName("shiro-cookies");
		simpleCookie.setHttpOnly(true);
		// 设置 Cookies 的过期时间
		simpleCookie.setMaxAge(60);
		return simpleCookie;
	}
	/**
	 * 第二步 创建 CookieRememberMeManager 的同时设置其 Cookie 属性
	 */
	@Bean
	public CookieRememberMeManager cookieRememberMeManager() {
		CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
		cookieRememberMeManager.setCipherKey(Base64.decode("4AvVhmFLUs0KTA3Kprsdag=="));
		cookieRememberMeManager.setCookie(simpleCookie());
		return cookieRememberMeManager;
	}
 
	/**
	 * 第三步 创建 SecurityManager 的同时设置其 CookieRememberMeManager 属性
	 */
	@Bean
	public SecurityManager securityManager() {
		DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
 
		// 1.CacheManager
		securityManager.setCacheManager(ehCacheManager());
 
		// 2. Authenticator
		securityManager.setAuthenticator(authenticator());
 
		// 3.Realm
		List<Realm> realms = new ArrayList<Realm>(16);
		realms.add(loginRealm());
		realms.add(userRealm());
		securityManager.setRealms(realms);
 
		// 4.SessionManager
		securityManager.setSessionManager(sessionManager());
 
		// 5. CookieRememberMeManager
		securityManager.setRememberMeManager(cookieRememberMeManager());
		return securityManager;
	}
```
 在用户登录校验时，还需要对 UsernamePasswordToken 进行如下设置，才能启用记住我。
```Java
	// 将用户名和密码封装成 UsernamePasswordToken 对象
	UsernamePasswordToken token = new UsernamePasswordToken(userName, password);
	token.setRememberMe(true);
```
在谷歌浏览器中查看用户登录后生成的 cookie 截图如下： 

<div align=center>

![Shiro图](http://pengjunlee.3vzhuji.net/static/security/22.png "Shiro示意图")
<div align=left>

 