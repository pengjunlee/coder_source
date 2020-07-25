---
title: SpringBoot基础之--Spring AOP
date: 2020-07-25 14:12:00
updated: 2020-07-25 14:12:00
tags: SpringBoot基础
categories: SpringBoot基础
keywords: Java, SpringBoot
type: 
description: 一篇文章带你理解什么是AOP面向切面编程。
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img12.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img12.jpg
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
官网文档地址：<https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop>  

# 什么是AOP？

`AOP（Aspect Oriented Programming 面向切面编程）`，通过预编译方式和运行期动态代理实现程序功能统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率，常用于`日志记录`，`性能统计`，`安全控制`，`事务处理`，`异常处理`等等。  

# AOP概念

- 连接点（Join Point）：程序执行过程中某个特定的点，比如，调用方法或者处理异常。
- 切入点（Pointcut）：筛选出来的某些符合一定规则的连接点。
- 通知（Advice）：指在切面的某个特定的连接点上执行的动作。
- 切面（Aspect）：通知和切入点的组合。
- 引入（Introduction）：为某一个类型声明额外的方法或者属性。
- 目标对象（Target Object）：被通知的源对象。
- AOP代理（AOP Proxy）：由AOP框架创建的代理对象。
- 织入（Wearving）：将切面和目标对象进行连接形成代理对象的过程。

举个比较形象的例子，某银行系统需要对所有的转账业务进行日志记录，在这一需求中日志记录即是我们要添加的通知，银行提供的所有业务（开户，查询，取款，转账）都可以被看作连接点，而我们只需要为转账这一个连接点添加通知，此时转账连接点就是日志通知要应用到的目标切入点，两者组合构成一个切面。 

# 声明Pointcut

Spring AOP 支持下面几个AspectJ的切入点指示符（PointCut Designators ，PCD）用来声明切入点。

## execution
execution 是在 Spring AOP 中使用最多的一个切入点指示符，它通过对连接点中执行的方法进行匹配，其声明表达式格式如下。
```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)throws-pattern?)
```

翻译后：  
```
execution(修饰符匹配样式? 返回值类型匹配样式 声明类型匹配样式?方法名匹配样式(参数匹配样式)异常匹配样式?)
```
应用示例： 
```Java
    // 匹配所有以set开头的公有方法
    @Pointcut("execution(public * set*(..))")
    public void publicSetOperation()
    {
 
    }
 
    // 匹配 com.pengjunlee.service 包及其子包下的类的所有方法
    @Pointcut("execution(* com.pengjunlee.service..*.*(..))")
    public void servicePackageOperation()
    {
 
    }
 
    // 匹配 com.pengjunlee.controller 包及其子包下以 Controller 结尾的类中
    // 声明抛出 LoginException 异常，并且返回值类型为 String 的公有方法
    @Pointcut("execution(public java.lang.String com.pengjunlee.controller..*Controller.*(..)throws javax.security.auth.login.LoginException)")
    public void pulbicControllerOperation()
    {
 
    }
```

## within 
within 用来匹配指定类型内的全部方法执行或指定包内某些类的全部方法执行。

应用示例：
```Java
    // 匹配 LogServiceImpl 类的所有方法
    @Pointcut("within(com.pengjunlee.service.impl.LogServiceImpl)")
    public void logServiceImplOperation()
    {
 
    }
 
    // 匹配 com.pengjunlee.service 包及其子包下所有以 ServiceImpl 结尾的类的所有方法
    @Pointcut("within(com.pengjunlee.service..*ServiceImpl)")
    public void allServiceImplOperation()
    {
 
    }
```

## target
Spring AOP 是基于代理的，target 指的是被代理的应用程序对象 ，当被代理的应用程序对象是指定类型的实例时表示匹配。

应用示例：
```Java
    // 当被代理的目标对象是 LogService 类型的实例时表示匹配
    @Pointcut("target(com.pengjunlee.service.LogService)")
    public void logServiceOperation()
    {
 
    }
```

## this
Spring AOP 是基于代理的，this 指的就是这个生成的代理对象 Bean ，当this Bean 可以转换为指定类型的实例时表示匹配。

应用示例：
```Java
    // 当生成的代理对象可以转化为 LogService 类型的实例时表示匹配
    @Pointcut("this(com.pengjunlee.service.LogService)")
    public void logServiceOperation()
    {
 
    }
```

## args
args 用来对执行方法时传入的参数类型进行匹配。

应用示例：
```Java
    // 匹配所有不带参数的方法
    @Pointcut("args()")
    public void noArgsOperation()
    {
 
    }
 
    // 匹配第一个参数是 String 类型的方法
    @Pointcut("args(java.lang.String,..)")
    public void firstArgIsStringOperation()
    {
 
    }
 
    // 匹配所有带参数的方法
    @Pointcut("args(..)")
    public void anyArgsOperation()
    {
 
    }
```

## @target
@target 用来对当前执行对象的类型上的注解进行匹配，当当前执行的对象类型上有指定注解时表示匹配。

应用示例：
```Java
    // 匹配 com.pengjunlee.controller 包及其子包下带有 Logable 注解的类的所有方法
    @Pointcut("execution(* com.pengjunlee.controller..*.*(..)) && @target(com.pengjunlee.annotation.Logable)")
    public void logableOperation()
    {
 
    }
```

## @within
@within 与 @target 类似，当执行带有指定注解的类型内声明的方法时表示匹配。

应用示例：
```Java
    // 匹配 com.pengjunlee.controller 包及其子包下带有 Logable 注解的类的所有方法
    @Pointcut("within(com.pengjunlee.controller..*) && @within(com.pengjunlee.annotation.Logable)")
    public void logableOperation()
    {
 
    }
```

## @args
@args 用来对执行方法传入的参数类型上的注解进行匹配，当参数类型上有指定注解时表示匹配。

应用示例：
```Java
    // 匹配 com.pengjunlee.controller 包及其子包下所有类中第一个参数的类型上有 Logable 注解的方法
    @Pointcut("execution(* com.pengjunlee.controller..*.*(..)) && @args(com.pengjunlee.annotation.Logable)")
    public void logableArgsOperation()
    {
 
    }
```

## @annotation
@annotation 用来对执行方法上的注解进行匹配，当方法上有指定注解时表示匹配。

应用示例：
```Java
    // 匹配 com.pengjunlee.controller 包及其子包下的类中带有 Countable 注解的方法
    @Pointcut("execution(* com.pengjunlee.controller..*.*(..)) && @annotation(com.pengjunlee.annotation.Countable)")
    public void countableOperation()
    {
 
    }
```

## bean
除此以上9个AspectJ的切入点指示符之外，Spring AOP还提供了另外一个切入点指示符 bean ，用来与Spring容器中的Bean的名称进行匹配。

应用示例：
```Java
   // 当被代理的目标对象是Spring容器中名称为 userService 或 deptService 的Bean表示匹配
    @Pointcut("bean(userService||deptService)")
    public void userOrDeptServiceOperation()
    {
 
    }
```

> **注意**：以上所有切入点指示符可以使用符号 `“&&”`、`“||”`、 `“!”` 进行组合使用。  

# 声明Advice

在Spring AOP中，通知（Advice）是和切入点（Pointcut）紧密相关的，通知可以执行在切入点之前或者之后，亦或者在切入点之前和之后都执行，相应的通知分为以下5种：

- 前置通知（Before）：在某个连接点被调用前执行；
- 后置通知（After）：在某个连接点被调用完成后执行；
- 返回通知（After-returning）：在某个连接点被调用并成功返回后执行；
- 异常通知（After-throwing）：在某个连接点被调用并抛出异常后执行；
- 环绕通知（Around）：指包围一个连接点的通知，在连接点被调用前和调用完成后都执行。

## 前置通知
前置通知使用 @Before 注解进行声明。

应用示例：
```Java
	@Aspect
	@Component
	public class BeforeAspect
	{
	    private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"); // HH ~ 24小时制
	 
	    private static final Logger logger=LoggerFactory.getLogger(BeforeAspect.class);
	    
	    // 记录请求时间和请求的方法
	    @Before("within(com.pengjunlee.controller..*)")
	    public void doBeforeOperation(JoinPoint point)
	    {
	        logger.info(sdf.format(new Date()) + " 访问" + "：" + point.getSignature());
	    }
	 
	}
```

## 后置通知
后置通知使用 @After 注解进行声明。

应用示例：
```Java
	//@Aspect("perthis(com.pengjunlee.aspect.AfterAspect.countableController())")
	@Aspect("pertarget(com.pengjunlee.aspect.AfterAspect.countableController())")
	@Order(10)
	@Component
	@Scope("prototype")
	public class AfterAspect
	{
	 
	    private static final Logger logger = LoggerFactory.getLogger(AfterAspect.class);
	 
	    private int count = 0;
	 
	    // 匹配 com.pengjunlee.controller 包及其子包下有 Countable 注解的类的所有方法
	    @Pointcut("within(com.pengjunlee.controller..*) && @within(com.pengjunlee.annotation.Countable)")
	    public void countableController()
	    {
	 
	    }
	 
	    // 统计各个控制器处理的请求次数
	    @After("countableController()")
	    public void doAfterOperation(JoinPoint point)
	    {
	        logger.info(point.getTarget().getClass() + " 访问次数：" + (++count));
	    }
	 
	}
```

## 返回通知
返回通知使用 @AfterReturning 注解进行声明。

应用示例：
```Java
	@Aspect
	@Component
	public class AfterReturningAspect
	{
	 
	    private static final Logger logger = LoggerFactory.getLogger(AfterReturningAspect.class);
	 
	    @AfterReturning(pointcut = "execution(* com.pengjunlee.controller.LoginController.login(..))", returning = "retVal", argNames = "retVal")
	    public void doAfterReturningOperation(JoinPoint point, Object retVal)
	    {
	        // 通知内容
	        logger.info("登录成功，请求返回值：" + retVal);
	    }
	 
	}
```
> **注意**：如果你希望在通知的方法体中能够获取到切入点执行完毕后的返回值，可以通过示例中的语法将切入点的执行结果绑定到通知中。 

## 异常通知
异常通知使用 @AfterThrowing 注解进行声明。

应用示例：
```Java
	@Aspect
	@Component
	public class AfterThrowingAspect
	{
	    private static final Logger logger = LoggerFactory.getLogger(AfterThrowingAspect.class);
	 
	    @AfterThrowing(pointcut = "execution(* com.pengjunlee.controller.LoginController.login(..))", throwing = "ex")
	    public void doAfterThrowingOperation(JoinPoint point, Exception ex)
	    {
	        // 通知内容
	        logger.error("登录失败！");
	    }
	 
	}
```
> **注意**：如果你希望在通知的方法体中能够获取到切入点执行时抛出的异常内容，可以通过示例中的语法将切入点抛出的异常绑定到通知中。 

## 环绕通知
环绕通知使用 @Around 注解进行声明。

应用示例：
```Java
	@Aspect
	@Component
	public class AroundAspect
	{
	    private static final Logger logger = LoggerFactory.getLogger(AroundAspect.class);
	 
	    // 记录请求的处理响应时间
	    @Around("execution(* com.pengjunlee.controller.LoginController.login(..))")
	    public Object doAroundOperation(ProceedingJoinPoint point) throws Throwable
	    {
	        long beginTime = System.currentTimeMillis();
	        // 执行方法
	        Object result = point.proceed();
	        // 执行时长(毫秒)
	        long time = System.currentTimeMillis() - beginTime;
	        logger.info("请求处理响应时间：" + time + " 毫秒");
	        return result;
	    }
	 
	}
```

# 通知参数
## 获取连接点信息
任何一个通知都可以传入一个 org.aspectj.lang.JoinPoint 类型的参数（环绕通知需要传入的参数类型为 org.aspectj.lang.ProceedingJoinPoint ，JoinPoint 是一个接口，ProceedingJoinPoint 是它的一个子类）作为通知方法的第一个参数。

JoinPoint 接口提供了一些非常有用的方法，比较常用的有下面几个：
```Java
    // 获取当前执行的代理对象
    Object getThis();
 
    // 获取被代理的目标对象
    Object getTarget();
 
    // 获取连接点的参数列表
    Object[] getArgs();
 
    // 获取连接点的签名信息
    Signature getSignature();
```

## 向通知传递参数
除了使用 JoinPoint 接口的 getArgs() 方法获取连接点的执行参数外，我们还可以通过以下方式在通知体中获取连接点的执行参数。 
```Java
    @Before("execution(* com.pengjunlee.service..*.*(..)) && args(str,..)")
    public void before(String str) throws Throwable
    {
        // ...
    }
```
此处的 args(str,..) 表达式有两个作用：

- 限制匹配的方法至少要有一个入参，并且第一个参数必须是 String 类型。
- 通过指定的 str 参数将参数值传递到通知中。

以上通知还可以书写为如下格式，两者效果相同。
```Java
    // 匹配 com.pengjunlee.service 包及其子包下所有类中第一个参数是 String 类型的方法
    @Pointcut("execution(* com.pengjunlee.service..*.*(..)) && args(str,..) ")
    public void allServiceOperation(String str)
    {
 
    }
 
    @Before("allServiceOperation(str)")
    public void before(String str) throws Throwable
    {
        // ...
    }
```
我们也能够将连接点的注解信息传递到通知中，示例代码如下。 
```Java
    @Before("execution(* com.pengjunlee.service..*.*(..)) && @annotation(auditable)")
    public void before(Auditable auditable) throws Throwable
    {
        // ...
    }
```
其中 Auditable 注解定义如下。 
```Java
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;
	 
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface Auditable
	{
	    String value() default "";
	}
```
类似地，我们还能够对泛型参数进行获取，例如下面的接口。 
```Java
	public interface Sample<T> {
	    
	    void sampleGenericMethod(T param);
	 
	    void sampleGenericCollectionMethod(Collection<T> param);
	}
```
如果要获取该接口的泛型参数，可以通过如下方式。 
```Java
    @Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
    public void beforeSampleMethod(MyType param)
    {
 
        // ...
    }
```
以上方法并不支持获取集合泛型参数，所以不能使用以下方式来获取集合泛型。 
```Java
    @Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
    public void beforeSampleMethod(Collection<MyType> param)
    {
 
        // ...
    }
```
本文示例源码已上传至CSDN，资源地址：<https://download.csdn.net/download/pengjunlee/10491809>