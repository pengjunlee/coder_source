---
title: SpringBoot框架整合之--使用RedisTemplate操作Redis数据库
date: 2020-07-24 14:11:00
updated: 2020-07-24 14:11:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何使用RedisTemplate操作Redis数据库?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img11.jpg
aside: true
toc: true
toc_number: false
auto_open: true
copyright: false
mathjax: false
katex: false
aplayer:
highlight_shrink: false
top: false
---
# 一、Maven依赖
```Xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/>
    </parent>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- 引入 Redis 依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!-- 引入单元测试依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- 接下来三个是jackson的依赖 -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.9.8</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.8</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.9.8</version>
        </dependency>
    </dependencies>
```

# 二、application.yml配置
```Properties
	spring:
	  ### 配置Redis
	  # Redis数据库索引（默认为0）
	  redis:
	    database: 0
	    # Redis服务器地址
	    host: 172.16.250.238
	    # Redis服务器连接端口
	    port: 6379
	    # Redis服务器连接密码（默认为空）
	    password: 123456
	    # 配置连接池
	    jedis:
	      pool:
	        # 连接池最大连接数（使用负值表示没有限制）
	        max-active: 8
	        # 连接池最大阻塞等待时间（使用负值表示没有限制）
	        max-wait: -1
	        # 连接池中的最大空闲连接
	        max-idle: 500
	        # 连接池中的最小空闲连接
	        min-idle: 0
	    # 连接超时时间（毫秒）
	    timeout: 2000
	    lettuce:
	      shutdown-timeout: 0
```

# 三、StringRedisTemplate与RedisTemplate测试

完成上述两个步骤之后，Springboot就会在应用启动时自动地为我们装配好StringRedisTemplate和RedisTemplate的Bean。下面写一个JUNIT来测试一下：
```Java
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.context.ApplicationContext;
	import org.springframework.data.redis.connection.RedisConnectionFactory;
	import org.springframework.data.redis.core.RedisTemplate;
	import org.springframework.data.redis.core.StringRedisTemplate;
	import org.springframework.test.context.junit4.SpringRunner;
	 
	import javax.annotation.Resource;
	 
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class SpringRedisApplicationTests {
	 
	    @Autowired
	    private ApplicationContext context;
	 
	    @Resource
	    private RedisTemplate<String, Object> redisTemplate;
	 
	    @Autowired
	    private StringRedisTemplate stringRedisTemplate;
	 
	    @Test
	    public void RedisTemplateTest() {
	        System.out.println(redisTemplate);  // 结果： org.springframework.data.redis.core.RedisTemplate@314c8b4a
	        redisTemplate.opsForValue().set("db-type", "redis");
	        System.out.println(redisTemplate.opsForValue().get("db-type")); // 结果： redis
	    }
	 
	    @Test
	    public void StringRedisTemplateTest() {
	        System.out.println(stringRedisTemplate);  // 结果： org.springframework.data.redis.core.RedisTemplate@314c8b4a
	        stringRedisTemplate.opsForValue().set("db-type", "mongodb");
	        System.out.println(stringRedisTemplate.opsForValue().get("db-type")); // 结果： mongodb
	        System.out.println(redisTemplate.opsForValue().get("db-type")); // 结果： redis
	    }
	 
	    @Test
	    public void RedisConnectionFactoryTest() {
	        RedisConnectionFactory redisConnectionFactoryBean = context.getBean(RedisConnectionFactory.class);
	        System.out.println(redisConnectionFactoryBean); // 结果： org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory@59496961
	    }
	 
	 
	}
```
需要知道的是：

1. StringRedisTemplate继承自RedisTemplate；
2. 从测试结果可以看出 StringRedisTemplate和RedisTemplate所管理的数据是相互独立的，对其中一个的数据进行修改并不会对另一个的数据产生影响。
3. SDR默认采用的序列化策略有两种，一种是String的序列化策略，一种是JDK的序列化策略。
	- StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。
	- RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

# 四、自己创建RedisTemplate Bean
```Java
	public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {}
```
> **注意**：如果没特殊情况，切勿定义成RedisTemplate<Object, Object>，否则根据里氏替换原则，使用的时候会造成类型错误 。推荐定义成 RedisTemplate<String, Object>。

> **常见问题**：在使用@Autowired注解装配RedisTemplate<String, Object>时由于泛型不匹配会出现如下错误：

`Could not autowire. No beans of 'RedisTemplate<String, Object>' type found.`

<div align=center>

![RedisTemplate示意图](http://pengjunlee.3vzhuji.net/static/springboot/57.png "RedisTemplate示意图")
<div align=left>

> **解决办法**：改用 @Resource 注解（@Autowired根据类型装配Bean，@Resource根据名称进行装配）。

<div align=center>

![RedisTemplate示意图](http://pengjunlee.3vzhuji.net/static/springboot/58.png "RedisTemplate示意图")
<div align=left>

或者，自己定义一个RedisTemplate<String, Object> Bean，而不再让Spring为我们创建RedisTemplate<Object, Object> Bean。这样即使使用@Autowired注解也可以成功装配RedisTemplate<String, Object> Bean。
```Java
	import com.fasterxml.jackson.annotation.JsonAutoDetect;
	import com.fasterxml.jackson.annotation.PropertyAccessor;
	import com.fasterxml.jackson.databind.ObjectMapper;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.data.redis.connection.RedisConnectionFactory;
	import org.springframework.data.redis.core.RedisTemplate;
	import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
	import org.springframework.data.redis.serializer.StringRedisSerializer;
	 
	/**
	 * @author pengjunlee
	 * @create 2019-09-10 9:59
	 */
	@Configuration
	public class RedisConfig {
	    @Bean
	    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
	        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
	        template.setConnectionFactory(factory);
	        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
	        ObjectMapper om = new ObjectMapper();
	        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
	        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
	        jackson2JsonRedisSerializer.setObjectMapper(om);
	        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
	        // key采用String的序列化方式
	        template.setKeySerializer(stringRedisSerializer);
	        // hash的key也采用String的序列化方式
	        template.setHashKeySerializer(stringRedisSerializer);
	        // value序列化方式采用jackson
	        template.setValueSerializer(jackson2JsonRedisSerializer);
	        // hash的value序列化方式采用jackson
	        template.setHashValueSerializer(jackson2JsonRedisSerializer);
	        template.afterPropertiesSet();
	        return template;
	    }
	}
```

# 五、Redis工具类

将RedisTemplate的常用API封装成一个工具类RedisUtil.java。
```Java
	import org.springframework.data.redis.core.RedisTemplate;
	import org.springframework.stereotype.Component;
	import org.springframework.util.CollectionUtils;
	 
	import javax.annotation.Resource;
	import java.util.List;
	import java.util.Map;
	import java.util.Set;
	import java.util.concurrent.TimeUnit;
	 
	/**
	 * @author pengjunlee
	 * @create 2019-09-10 10:02
	 */
	@Component
	public class RedisUtil {
	 
	    @Resource
	    private RedisTemplate<String, Object> redisTemplate;
	 
	    /**
	     * 指定缓存失效时间
	     *
	     * @param key
	     * @param time 时间(秒)
	     * @return
	     */
	    public boolean expire(String key, long time) {
	        try {
	            if (time > 0) {
	                redisTemplate.expire(key, time, TimeUnit.SECONDS);
	            }
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 根据key 获取过期时间
	     *
	     * @param key
	     * @return 时间(秒) 返回0代表为永久有效
	     */
	    public long getExpire(String key) {
	        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
	    }
	 
	    /**
	     * @param key
	     * @return true 存在 false不存在
	     */
	    public boolean hasKey(String key) {
	        try {
	            return redisTemplate.hasKey(key);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 删除键值
	     *
	     * @param key 可以传一个值 或多个
	     */
	    @SuppressWarnings("unchecked")
	    public void del(String... key) {
	        if (key != null && key.length > 0) {
	            if (key.length == 1) {
	                redisTemplate.delete(key[0]);
	            } else {
	                redisTemplate.delete(CollectionUtils.arrayToList(key));
	            }
	        }
	    }
	 
	    /**
	     * 获取键值
	     *
	     * @param key
	     * @return
	     */
	    public Object get(String key) {
	        return key == null ? null : redisTemplate.opsForValue().get(key);
	    }
	 
	    /**
	     * 设置键值
	     *
	     * @param key
	     * @param value
	     * @return true成功 false失败
	     */
	    public boolean set(String key, Object value) {
	        try {
	            redisTemplate.opsForValue().set(key, value);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 设置键值同时指定过期时间
	     *
	     * @param key
	     * @param value
	     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
	     * @return true成功 false 失败
	     */
	    public boolean set(String key, Object value, long time) {
	        try {
	            if (time > 0) {
	                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
	            } else {
	                set(key, value);
	            }
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 键值递增
	     *
	     * @param key
	     * @param delta 要增加几(大于0)
	     * @return
	     */
	    public long incr(String key, long delta) {
	        if (delta < 0) {
	            throw new RuntimeException("递增因子必须大于0");
	        }
	        return redisTemplate.opsForValue().increment(key, delta);
	    }
	 
	    /**
	     * 键值递减
	     *
	     * @param key
	     * @param delta 要减少几(小于0)
	     * @return
	     */
	    public long decr(String key, long delta) {
	        if (delta < 0) {
	            throw new RuntimeException("递减因子必须大于0");
	        }
	        return redisTemplate.opsForValue().increment(key, -delta);
	    }
	 
	    /**
	     * 获取HashGet
	     *
	     * @param key
	     * @param item
	     * @return
	     */
	    public Object hget(String key, String item) {
	        return redisTemplate.opsForHash().get(key, item);
	    }
	 
	    /**
	     * 设置HashGet
	     *
	     * @param key
	     * @return
	     */
	    public Map<Object, Object> hmget(String key) {
	        return redisTemplate.opsForHash().entries(key);
	    }
	 
	    /**
	     * 设置多个HashGet
	     *
	     * @param key
	     * @param map
	     * @return
	     */
	    public boolean hmset(String key, Map<String, Object> map) {
	        try {
	            redisTemplate.opsForHash().putAll(key, map);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 设置多个HashGet时指定过期时间
	     *
	     * @param key
	     * @param map
	     * @param time 时间(秒)
	     * @return
	     */
	    public boolean hmset(String key, Map<String, Object> map, long time) {
	        try {
	            redisTemplate.opsForHash().putAll(key, map);
	            if (time > 0) {
	                expire(key, time);
	            }
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 向一张hash表中放入数据,如果不存在将创建
	     *
	     * @param key
	     * @param item
	     * @param value
	     * @return true 成功 false失败
	     */
	    public boolean hset(String key, String item, Object value) {
	        try {
	            redisTemplate.opsForHash().put(key, item, value);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 向一张hash表中放入数据,如果不存在将创建
	     *
	     * @param key
	     * @param item
	     * @param value
	     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
	     * @return true 成功 false失败
	     */
	    public boolean hset(String key, String item, Object value, long time) {
	        try {
	            redisTemplate.opsForHash().put(key, item, value);
	            if (time > 0) {
	                expire(key, time);
	            }
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 删除hash表中的值
	     *
	     * @param key
	     * @param item
	     */
	    public void hdel(String key, Object... item) {
	        redisTemplate.opsForHash().delete(key, item);
	    }
	 
	    /**
	     * 判断hash表中是否存在item
	     *
	     * @param key
	     * @param item
	     * @return true 存在 false不存在
	     */
	    public boolean hHasKey(String key, String item) {
	        return redisTemplate.opsForHash().hasKey(key, item);
	    }
	 
	    /**
	     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
	     *
	     * @param key
	     * @param item
	     * @param by   要增加的值(大于0)
	     * @return
	     */
	    public double hincr(String key, String item, double by) {
	        return redisTemplate.opsForHash().increment(key, item, by);
	    }
	 
	    /**
	     * hash递减
	     *
	     * @param key
	     * @param item
	     * @param by   要减少的值(小于0)
	     * @return
	     */
	    public double hdecr(String key, String item, double by) {
	        return redisTemplate.opsForHash().increment(key, item, -by);
	    }
	 
	    /**
	     * 获取set的内容
	     *
	     * @param key
	     * @return
	     */
	    public Set<Object> sGet(String key) {
	        try {
	            return redisTemplate.opsForSet().members(key);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return null;
	        }
	    }
	 
	    /**
	     * 判断set中是否存在某个值
	     *
	     * @param key
	     * @param value
	     * @return
	     */
	    public boolean sHasKey(String key, Object value) {
	        try {
	            return redisTemplate.opsForSet().isMember(key, value);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 设置set
	     *
	     * @param key
	     * @param values
	     * @return
	     */
	    public long sSet(String key, Object... values) {
	        try {
	            return redisTemplate.opsForSet().add(key, values);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return 0;
	        }
	    }
	 
	    /**
	     * 设置set同时指定过期时间
	     *
	     * @param key
	     * @param time   时间(秒)
	     * @param values
	     * @return
	     */
	    public long sSetAndTime(String key, long time, Object... values) {
	        try {
	            Long count = redisTemplate.opsForSet().add(key, values);
	            if (time > 0)
	                expire(key, time);
	            return count;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return 0;
	        }
	    }
	 
	    /**
	     * 获取set的元素个数
	     *
	     * @param key
	     * @return
	     */
	    public long sGetSetSize(String key) {
	        try {
	            return redisTemplate.opsForSet().size(key);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return 0;
	        }
	    }
	 
	    /**
	     * 从set中移除元素
	     *
	     * @param key
	     * @param values
	     * @return
	     */
	    public long setRemove(String key, Object... values) {
	        try {
	            Long count = redisTemplate.opsForSet().remove(key, values);
	            return count;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return 0;
	        }
	    }
	 
	    /**
	     * 获取List的内容
	     *
	     * @param key
	     * @param start 开始索引
	     * @param end   结束索引 0 到 -1代表所有值
	     * @return
	     */
	    public List<Object> lGet(String key, long start, long end) {
	        try {
	            return redisTemplate.opsForList().range(key, start, end);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return null;
	        }
	    }
	 
	    /**
	     * 获取list的长度
	     *
	     * @param key
	     * @return
	     */
	    public long lGetListSize(String key) {
	        try {
	            return redisTemplate.opsForList().size(key);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return 0;
	        }
	    }
	 
	    /**
	     * 通过索引 获取list中的值
	     *
	     * @param key
	     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
	     * @return
	     */
	    public Object lGetIndex(String key, long index) {
	        try {
	            return redisTemplate.opsForList().index(key, index);
	        } catch (Exception e) {
	            e.printStackTrace();
	            return null;
	        }
	    }
	 
	    /**
	     * 设置List
	     *
	     * @param key
	     * @param value
	     * @return
	     */
	    public boolean lSet(String key, Object value) {
	        try {
	            redisTemplate.opsForList().rightPush(key, value);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 设置List
	     *
	     * @param key
	     * @param value
	     * @param time  时间(秒)
	     * @return
	     */
	    public boolean lSet(String key, Object value, long time) {
	        try {
	            redisTemplate.opsForList().rightPush(key, value);
	            if (time > 0)
	                expire(key, time);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 设置List
	     *
	     * @param key
	     * @param value
	     * @return
	     */
	    public boolean lSet(String key, List<Object> value) {
	        try {
	            redisTemplate.opsForList().rightPushAll(key, value);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 设置List同时指定过期时间
	     *
	     * @param key
	     * @param value
	     * @param time  时间(秒)
	     * @return
	     */
	    public boolean lSet(String key, List<Object> value, long time) {
	        try {
	            redisTemplate.opsForList().rightPushAll(key, value);
	            if (time > 0)
	                expire(key, time);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 根据索引修改list中的某条数据
	     *
	     * @param key
	     * @param index
	     * @param value
	     * @return
	     */
	    public boolean lUpdateIndex(String key, long index, Object value) {
	        try {
	            redisTemplate.opsForList().set(key, index, value);
	            return true;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return false;
	        }
	    }
	 
	    /**
	     * 移除N个值为value
	     *
	     * @param key
	     * @param count
	     * @param value
	     * @return 移除的个数
	     */
	    public long lRemove(String key, long count, Object value) {
	        try {
	            Long remove = redisTemplate.opsForList().remove(key, count, value);
	            return remove;
	        } catch (Exception e) {
	            e.printStackTrace();
	            return 0;
	        }
	    }
	 
	}
```
在需要操作Redis的类中通过@Autowired注解引人RedisUtil Bean进行使用。
```Java
    @Autowired
    private RedisUtil redisUtil;
 
    @Test
    public void MapTest() {
        Map<String, String> map = new HashMap<>(16);
        map.put("db-type", "redis");
        redisUtil.set("data", map); // 结果： ["java.util.HashMap",{"db-type":"redis"}]
        Map<String, String> data = (Map<String, String>) redisUtil.get("data");
        System.out.println(data.get("db-type")); // 结果： redis
    }
 
    @Test
    public void ListTest() {
        List<User> list = new ArrayList<>(16);
        list.add(new User("tracy", 18));
        list.add(new User("graython", 24));
        redisUtil.set("data", list); // 结果： ["java.util.ArrayList",[["com.pengjunlee.springredis.domain.User",{"name":"tracy","age":18}],["com.pengjunlee.springredis.domain.User",{"name":"graython","age":24}]]]
        List<User> users = (List<User>) redisUtil.get("data");
        System.out.println(users.get(1).getName()); // 结果： graython
    }
```