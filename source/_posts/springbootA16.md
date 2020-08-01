---
title: SpringBoot框架整合之--MyBatis-Plus3.1
date: 2020-07-24 14:16:00
updated: 2020-07-24 14:16:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何整合MyBatis-Plus3.1?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img16.jpg
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
> 转载自：<https://juejin.im/post/5cfa6e465188254ee433bc69>

# 一.说明
Mybatis-Plus是一个Mybatis框架的增强插件,根据官方描述,MP只做增强不做改变,引入它不会对现有工程产生影响,如丝般顺滑.并且只需简单配置,即可快速进行 CRUD 操作,从而节省大量时间.代码生成,分页,性能分析等功能一应俱全,最新已经更新到了3.1.1版本了,3.X系列支持lambda语法,让我在写条件构造的时候少了很多的"魔法值",从代码结构上更简洁了.

# 二.项目环境

- MyBatis-Plus版本: 3.1.0
- SpringBoot版本:2.1.5
- JDK版本:1.8

Maven依赖如下：
```Xml
	<dependencies>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>mysql</groupId>
	            <artifactId>mysql-connector-java</artifactId>
	            <scope>runtime</scope>
	        </dependency>
	        <dependency>
	            <groupId>org.projectlombok</groupId>
	            <artifactId>lombok</artifactId>
	            <optional>true</optional>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-test</artifactId>
	            <scope>test</scope>
	        </dependency>
	        <!-- mybatisPlus 核心库 -->
	        <dependency>
	            <groupId>com.baomidou</groupId>
	            <artifactId>mybatis-plus-boot-starter</artifactId>
	            <version>3.1.0</version>
	        </dependency>
	        <!-- 引入阿里数据库连接池 -->
	        <dependency>
	            <groupId>com.alibaba</groupId>
	            <artifactId>druid</artifactId>
	            <version>1.1.6</version>
	        </dependency>
	</dependencies>
```
配置如下:
```Yml
	# 配置端口
	server:
	  port: 8081
	spring:
	  # 配置数据源
	  datasource:
	    driver-class-name: com.mysql.cj.jdbc.Driver
	    url: jdbc:mysql://localhost:3306/mp_student?useUnicode=true&characterEncoding=utf-8
	    username: root
	    password: root
	    type: com.alibaba.druid.pool.DruidDataSource
	# mybatis-plus相关配置
	mybatis-plus:
	  # xml扫描，多个目录用逗号或者分号分隔（告诉 Mapper 所对应的 XML 文件位置）
	  mapper-locations: classpath:mapper/*.xml
	  # 以下配置均有默认值,可以不设置
	  global-config:
	    db-config:
	      #主键类型  auto:"数据库ID自增" 1:"用户输入ID",2:"全局唯一ID (数字类型唯一ID)", 3:"全局唯一ID UUID";
	      id-type: auto
	      #字段策略 IGNORED:"忽略判断"  NOT_NULL:"非 NULL 判断")  NOT_EMPTY:"非空判断"
	      field-strategy: NOT_EMPTY
	      #数据库类型
	      db-type: MYSQL
	  configuration:
	    # 是否开启自动驼峰命名规则映射:从数据库列名到Java属性驼峰命名的类似映射
	    map-underscore-to-camel-case: true
	    # 如果查询结果中包含空值的列，则 MyBatis 在映射的时候，不会映射这个字段
	    call-setters-on-nulls: true
	    # 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
	    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```
表结构:
```Sql
	CREATE TABLE `user_info` (
	  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT 'ID',
	  `name` varchar(32) DEFAULT NULL COMMENT '姓名',
	  `age` int(11) DEFAULT NULL COMMENT '年龄',
	  `skill` varchar(32) DEFAULT NULL COMMENT '技能',
	  `evaluate` varchar(64) DEFAULT NULL COMMENT '评价',
	  `fraction` bigint(11) DEFAULT NULL COMMENT '分数',
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8mb4 COMMENT='学生信息表';
```
表数据:
```Sql
	INSERT INTO `user_info` VALUES (1, '小明', 20, '画画', '该学生在画画方面有一定天赋', 89);
	INSERT INTO `user_info` VALUES (2, '小兰', 19, '游戏', '近期该学生由于游戏的原因导致分数降低了', 64);
	INSERT INTO `user_info` VALUES (3, '张张', 18, '英语', '近期该学生参加英语比赛获得二等奖', 90);
	INSERT INTO `user_info` VALUES (4, '大黄', 20, '体育', '该学生近期由于参加篮球比赛,导致脚伤', 76);
	INSERT INTO `user_info` VALUES (5, '大白', 17, '绘画', '该学生参加美术大赛获得三等奖', 77);
	INSERT INTO `user_info` VALUES (7, '小龙', 18, 'JAVA', '该学生是一个在改BUG的码农', 59);
	INSERT INTO `user_info` VALUES (9, 'Sans', 18, '睡觉', 'Sans是一个爱睡觉,并且身材较矮骨骼巨大的骷髅小胖子', 60);
	INSERT INTO `user_info` VALUES (10, 'papyrus', 18, 'JAVA', 'Papyrus是一个讲话大声、个性张扬的骷髅，给人自信、有魅力的骷髅小瘦子', 58);
	INSERT INTO `user_info` VALUES (11, '删除数据1', 3, '画肖像', NULL, 61);
	INSERT INTO `user_info` VALUES (12, '删除数据2', 3, NULL, NULL, 61);
	INSERT INTO `user_info` VALUES (13, '删除数据3', 3, NULL, NULL, 61);
	INSERT INTO `user_info` VALUES (14, '删除数据4', 5, '删除', NULL, 10);
	INSERT INTO `user_info` VALUES (15, '删除数据5', 6, '删除', NULL, 10);
```

# 三.编写基础类
在启动类上添加扫描DAO的注解。
```Java
	@SpringBootApplication
	@MapperScan(basePackages = {"com.mp.demo.dao"}) //扫描DAO
	public class DemoApplication {
	    public static void main(String[] args) {
	        SpringApplication.run(DemoApplication.class, args);
	    }
	}
```
编写Config配置类。
```Java
	/**
	 * @Description MybatisPlus配置类
	 * @Author Sans
	 * @CreateTime 2019/5/26 17:20
	 */
	@Configuration
	public class MybatisPlusConfig {
	    /**
	     * mybatis-plus SQL执行效率插件【生产环境可以关闭】
	     */
	    @Bean
	    public PerformanceInterceptor performanceInterceptor() {
	        return new PerformanceInterceptor();
	    }
	    /**
	     * 分页插件
	     */
	    @Bean
	    public PaginationInterceptor paginationInterceptor() {
	        return new PaginationInterceptor();
	    }
	}
```
编写Entity类。
```Java
	/**
	 * @Description 学生信息实体类
	 * @Author Sans
	 * @CreateTime 2019/5/26 21:41
	 */
	@Data
	@TableName("user_info")//@TableName中的值对应着表名
	public class UserInfoEntity {
	 
	    /**
	     * 主键
	     * @TableId中可以决定主键的类型,不写会采取默认值,默认值可以在yml中配置
	     * AUTO: 数据库ID自增
	     * INPUT: 用户输入ID
	     * ID_WORKER: 全局唯一ID，Long类型的主键
	     * ID_WORKER_STR: 字符串全局唯一ID
	     * UUID: 全局唯一ID，UUID类型的主键
	     * NONE: 该类型为未设置主键类型
	     */
	    @TableId(type = IdType.AUTO)
	    private Long id;
	    /**
	     * 姓名
	     */
	    private String name;
	    /**
	     * 年龄
	     */
	    private Integer age;
	    /**
	     * 技能
	     */
	    private String skill;
	    /**
	     * 评价
	     */
	    private String evaluate;
	    /**
	     * 分数
	     */
	    private Long fraction;
	}
```
编写Dao类。
```Java
	/**
	 * @Description 用户信息DAO
	 * @Author Sans
	 * @CreateTime 2019/6/8 16:24
	 */
	public interface UserInfoDao extends BaseMapper<UserInfoEntity> {
	}
```
编写Service类。
```Java
	/**
	 * @Description 用户业务接口
	 * @Author Sans
	 * @CreateTime 2019/6/8 16:26
	 */
	public interface UserInfoService extends IService<UserInfoEntity> {
	}
```
编写ServiceImpl类。
```Java
	/**
	 * @Description 用户业务实现
	 * @Author Sans
	 * @CreateTime 2019/6/8 16:26
	 */
	@Service
	@Transactional
	public class UserInfoSerivceImpl extends ServiceImpl<UserInfoDao, UserInfoEntity> implements UserInfoService {
	}
```

# 四.MyBatis-Plus基础演示
这里我们看到,service中我们没有写任何方法,MyBatis-Plus官方封装了许多基本CRUD的方法,可以直接使用大量节约时间,MP共通方法详见IService,ServiceImpl,BaseMapper源码,写入操作在ServiceImpl中已有事务绑定,这里我们举一些常用的方法演示.
```Java
	/**
	 * @Description UserInfoController
	 * @Author Sans
	 * @CreateTime 2019/6/8 16:27
	 */
	@RestController
	@RequestMapping("/userInfo")
	public class UserInfoController {
	 
	    @Autowired
	    private UserInfoService userInfoService;
	 
	    /**
	     * 根据ID获取用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:34
	     * @Param  userId  用户ID
	     * @Return UserInfoEntity 用户实体
	     */
	    @RequestMapping("/getInfo")
	    public UserInfoEntity getInfo(String userId){
	        UserInfoEntity userInfoEntity = userInfoService.getById(userId);
	        return userInfoEntity;
	    }
	    /**
	     * 查询全部信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:35
	     * @Param  userId  用户ID
	     * @Return List<UserInfoEntity> 用户实体集合
	     */
	    @RequestMapping("/getList")
	    public List<UserInfoEntity> getList(){
	        List<UserInfoEntity> userInfoEntityList = userInfoService.list();
	        return userInfoEntityList;
	    }
	    /**
	     * 分页查询全部数据
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:37
	     * @Return IPage<UserInfoEntity> 分页数据
	     */
	    @RequestMapping("/getInfoListPage")
	    public IPage<UserInfoEntity> getInfoListPage(){
	        //需要在Config配置类中配置分页插件
	        IPage<UserInfoEntity> page = new Page<>();
	        page.setCurrent(5); //当前页
	        page.setSize(1);    //每页条数
	        page = userInfoService.page(page);
	        return page;
	    }
	    /**
	     * 根据指定字段查询用户信息集合
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:39
	     * @Return Collection<UserInfoEntity> 用户实体集合
	     */
	    @RequestMapping("/getListMap")
	    public Collection<UserInfoEntity> getListMap(){
	        Map<String,Object> map = new HashMap<>();
	        //kay是字段名 value是字段值
	        map.put("age",20);
	        Collection<UserInfoEntity> userInfoEntityList = userInfoService.listByMap(map);
	        return userInfoEntityList;
	    }
	    /**
	     * 新增用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:40
	     */
	    @RequestMapping("/saveInfo")
	    public void saveInfo(){
	        UserInfoEntity userInfoEntity = new UserInfoEntity();
	        userInfoEntity.setName("小龙");
	        userInfoEntity.setSkill("JAVA");
	        userInfoEntity.setAge(18);
	        userInfoEntity.setFraction(59L);
	        userInfoEntity.setEvaluate("该学生是一个在改BUG的码农");
	        userInfoService.save(userInfoEntity);
	    }
	    /**
	     * 批量新增用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:42
	     */
	    @RequestMapping("/saveInfoList")
	    public void saveInfoList(){
	        //创建对象
	        UserInfoEntity sans = new UserInfoEntity();
	        sans.setName("Sans");
	        sans.setSkill("睡觉");
	        sans.setAge(18);
	        sans.setFraction(60L);
	        sans.setEvaluate("Sans是一个爱睡觉,并且身材较矮骨骼巨大的骷髅小胖子");
	        UserInfoEntity papyrus = new UserInfoEntity();
	        papyrus.setName("papyrus");
	        papyrus.setSkill("JAVA");
	        papyrus.setAge(18);
	        papyrus.setFraction(58L);
	        papyrus.setEvaluate("Papyrus是一个讲话大声、个性张扬的骷髅，给人自信、有魅力的骷髅小瘦子");
	        //批量保存
	        List<UserInfoEntity> list =new ArrayList<>();
	        list.add(sans);
	        list.add(papyrus);
	        userInfoService.saveBatch(list);
	    }
	    /**
	     * 更新用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:47
	     */
	    @RequestMapping("/updateInfo")
	    public void updateInfo(){
	        //根据实体中的ID去更新,其他字段如果值为null则不会更新该字段,参考yml配置文件
	        UserInfoEntity userInfoEntity = new UserInfoEntity();
	        userInfoEntity.setId(1L);
	        userInfoEntity.setAge(19);
	        userInfoService.updateById(userInfoEntity);
	    }
	    /**
	     * 新增或者更新用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:50
	     */
	    @RequestMapping("/saveOrUpdateInfo")
	    public void saveOrUpdate(){
	        //传入的实体类userInfoEntity中ID为null就会新增(ID自增)
	        //实体类ID值存在,如果数据库存在ID就会更新,如果不存在就会新增
	        UserInfoEntity userInfoEntity = new UserInfoEntity();
	        userInfoEntity.setId(1L);
	        userInfoEntity.setAge(20);
	        userInfoService.saveOrUpdate(userInfoEntity);
	    }
	    /**
	     * 根据ID删除用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:52
	     */
	    @RequestMapping("/deleteInfo")
	    public void deleteInfo(String userId){
	        userInfoService.removeById(userId);
	    }
	    /**
	     * 根据ID批量删除用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:55
	     */
	    @RequestMapping("/deleteInfoList")
	    public void deleteInfoList(){
	        List<String> userIdlist = new ArrayList<>();
	        userIdlist.add("12");
	        userIdlist.add("13");
	        userInfoService.removeByIds(userIdlist);
	    }
	    /**
	     * 根据指定字段删除用户信息
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:57
	     */
	    @RequestMapping("/deleteInfoMap")
	    public void deleteInfoMap(){
	        //kay是字段名 value是字段值
	        Map<String,Object> map = new HashMap<>();
	        map.put("skill","删除");
	        map.put("fraction",10L);
	        userInfoService.removeByMap(map);
	    }
	}
```

# 五.MyBatis-Plus的QueryWrapper条件构造器
当查询条件复杂的时候,我们可以使用MP的条件构造器,请参考下面的QueryWrapper条件参数说明
<div align=center>

![MyBatis图](http://pengjunlee.3vzhuji.net/static/springboot/60.png "MyBatis-Plus语法示意图")
<div align=left>
下面我们来举一些常见的示例。
```Java
	/**
	 * @Description UserInfoPlusController
	 * @Author Sans
	 * @CreateTime 2019/6/9 14:52
	 */
	@RestController
	@RequestMapping("/userInfoPlus")
	public class UserInfoPlusController {
	 
	    @Autowired
	    private UserInfoService userInfoService;
	 
	    /**
	     * MP扩展演示
	     * @Author Sans
	     * @CreateTime 2019/6/8 16:37
	     * @Return Map<String,Object> 返回数据
	     */
	    @RequestMapping("/getInfoListPlus")
	    public Map<String,Object> getInfoListPage(){
	        //初始化返回类
	        Map<String,Object> result = new HashMap<>();
	        //查询年龄等于18岁的学生
	        //等价SQL: SELECT id,name,age,skill,evaluate,fraction FROM user_info WHERE age = 18
	        QueryWrapper<UserInfoEntity> queryWrapper1 = new QueryWrapper<>();
	        queryWrapper1.lambda().eq(UserInfoEntity::getAge,18);
	        List<UserInfoEntity> userInfoEntityList1 = userInfoService.list(queryWrapper1);
	        result.put("studentAge18",userInfoEntityList1);
	        //查询年龄大于5岁的学生且小于等于18岁的学生
	        //等价SQL: SELECT id,name,age,skill,evaluate,fraction FROM user_info WHERE age > 5 AND age <= 18
	        QueryWrapper<UserInfoEntity> queryWrapper2 = new QueryWrapper<>();
	        queryWrapper2.lambda().gt(UserInfoEntity::getAge,5);
	        queryWrapper2.lambda().le(UserInfoEntity::getAge,18);
	        List<UserInfoEntity> userInfoEntityList2 = userInfoService.list(queryWrapper2);
	        result.put("studentAge5",userInfoEntityList2);
	        //模糊查询技能字段带有"画"的数据,并按照年龄降序
	        //等价SQL: SELECT id,name,age,skill,evaluate,fraction FROM user_info WHERE skill LIKE '%画%' ORDER BY age DESC
	        QueryWrapper<UserInfoEntity> queryWrapper3 = new QueryWrapper<>();
	        queryWrapper3.lambda().like(UserInfoEntity::getSkill,"画");
	        queryWrapper3.lambda().orderByDesc(UserInfoEntity::getAge);
	        List<UserInfoEntity> userInfoEntityList3 = userInfoService.list(queryWrapper3);
	        result.put("studentAgeSkill",userInfoEntityList3);
	        //模糊查询名字带有"小"或者年龄大于18的学生
	        //等价SQL: SELECT id,name,age,skill,evaluate,fraction FROM user_info WHERE name LIKE '%小%' OR age > 18
	        QueryWrapper<UserInfoEntity> queryWrapper4 = new QueryWrapper<>();
	        queryWrapper4.lambda().like(UserInfoEntity::getName,"小");
	        queryWrapper4.lambda().or().gt(UserInfoEntity::getAge,18);
	        List<UserInfoEntity> userInfoEntityList4 = userInfoService.list(queryWrapper4);
	        result.put("studentOr",userInfoEntityList4);
	        //查询评价不为null的学生,并且分页
	        //等价SQL: SELECT id,name,age,skill,evaluate,fraction FROM user_info WHERE evaluate IS NOT NULL LIMIT 0,5
	        IPage<UserInfoEntity> page = new Page<>();
	        page.setCurrent(1);
	        page.setSize(5);
	        QueryWrapper<UserInfoEntity> queryWrapper5 = new QueryWrapper<>();
	        queryWrapper5.lambda().isNotNull(UserInfoEntity::getEvaluate);
	        page = userInfoService.page(page,queryWrapper5);
	        result.put("studentPage",page);
	        return result;
	    }
	}
```

# 六.自定义SQL
引入Mybatis-Plus不会对项目现有的 Mybatis 构架产生任何影响，而且Mybatis-Plus支持所有 Mybatis 原生的特性,这也是我喜欢使用它的原因之一,由于某些业务复杂,我们可能要自己去写一些比较复杂的SQL语句,我们举一个简单的例子来演示自定义SQL.

示例:查询大于设置分数的学生(分数为动态输入,且有分页)

编写Mapper.xml文件：
```Xml
	<mapper namespace="com.mp.demo.dao.UserInfoDao">
	    <!-- Sans 2019/6/9 14:35 -->
	    <select id="selectUserInfoByGtFraction" resultType="com.mp.demo.entity.UserInfoEntity" parameterType="long">
		SELECT * FROM user_info WHERE fraction > #{fraction}
	    </select>
	</mapper>
```
在DAO中加入方法：
```Java
    /**
     * 查询大于该分数的学生
     * @Author Sans
     * @CreateTime 2019/6/9 14:28
     * @Param  page  分页参数
     * @Param  fraction  分数
     * @Return IPage<UserInfoEntity> 分页数据
     */
    IPage<UserInfoEntity> selectUserInfoByGtFraction(IPage<UserInfoEntity> page, Long fraction);
```
在service加入方法：
```Java
    /**
     * 查询大于该分数的学生
     * @Author Sans
     * @CreateTime 2019/6/9 14:27
     * @Param  page  分页参数
     * @Param  fraction  分数
     * @Return IPage<UserInfoEntity> 分页数据
     */
    IPage<UserInfoEntity> selectUserInfoByGtFraction(IPage<UserInfoEntity> page,Long fraction);
```
在serviceImpl加入方法：
```Java
    /**
     * 查询大于该分数的学生
     * @Author Sans
     * @CreateTime 2019/6/9 14:27
     * @Param  page  分页参数
     * @Param  fraction  分数
     * @Return IPage<UserInfoEntity> 分页数据
     */
    @Override
    public IPage<UserInfoEntity> selectUserInfoByGtFraction(IPage<UserInfoEntity> page, Long fraction) {
        return this.baseMapper.selectUserInfoByGtFraction(page,fraction);
    }
```
在Controller中测试：
```Java
    /**
     * MP自定义SQL
     * @Author Sans
     * @CreateTime 2019/6/9 14:37
     * @Return IPage<UserInfoEntity> 分页数据
     */
    @RequestMapping("/getInfoListSQL")
    public IPage<UserInfoEntity> getInfoListSQL(){
        //查询大于60分以上的学生,并且分页
        IPage<UserInfoEntity> page = new Page<>();
        page.setCurrent(1);
        page.setSize(5);
        page = userInfoService.selectUserInfoByGtFraction(page,60L);
        return page;
    }
```

# 七.项目源码
项目源码: <https://gitee.com/liselotte/spring-boot-mp-demo>

个人确实很喜欢用MyBatis-Plus,不仅节约时间,代码也简洁干净,它给了我那时候从SSM到SpringBoot过度的那种感觉

嗯,这玩意真香~

谢谢大家阅读,如果喜欢,请收藏点赞,文章不足之处,也请给出宝贵意见.