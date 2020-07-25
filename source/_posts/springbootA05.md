---
title: SpringBoot框架整合之--Quartz中为Job自动装配Bean
date: 2020-07-24 14:05:00
updated: 2020-07-24 14:05:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: Quartz的作业类中无法注入Service等由Spring容器所管理的Bean如何解决?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img5.jpg
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
在《SpringBoot系列之--使用数据库配置Quartz定时任务》一文中我们通过将定时任务持久化到数据库实现了对定时任务的动态灵活配置，但**存在一个非常严重的缺陷**：<font color=red>定时任务Job的作业类中无法注入Service等由Spring容器所管理的Bean</font>。例如下面这种情况，TaskCronJobService就无法成功注入。 
```Java
	import java.util.Iterator;
	 
	import javax.annotation.Resource;
	 
	import org.quartz.Job;
	import org.quartz.JobExecutionContext;
	import org.quartz.JobExecutionException;
	import org.springframework.stereotype.Component;
	 
	import com.pengjunlee.task.bean.TaskCronJob;
	import com.pengjunlee.task.service.TaskCronJobService;
	 
	/**
	 * 定时任务的作业类，需实现Job接口
	 * 
	 * @author pengjunlee
	 *
	 */
	@Component
	public class MyJob implements Job {
	 
		@Resource
		private TaskCronJobService taskCronJobService;
	 
		@Override
		public void execute(JobExecutionContext arg0) throws JobExecutionException {
			System.out.println("执行定时任务:MyJob.execute()..."
					+ System.currentTimeMillis());
			Iterable<TaskCronJob> findAll = taskCronJobService.findAll();
			Iterator<TaskCronJob> iterator = findAll.iterator();
			while (iterator.hasNext()) {
				System.out.println(iterator.next().getJobClassName());
			}
		}
	 
	}
```
通过从网上搜索，发现**是以下原因所导致**：<font color=blue>定时任务Job对象的实例化过程是在Quartz中进行的，而TaskCronJobService Bean是由Spring容器管理的，Quartz根本就察觉不到TaskCronJobService Bean的存在，故而无法将TaskCronJobService Bean装配到Job对象中</font>。 

知道了问题出现的原因，**解决的办法**也就显而易见了：<font color=green>如果能够将Job Bean也纳入到Spring容器的管理之中的话，Spring容器自然能够为Job Bean自动装配好所需的依赖</font>。  

# SchedulerFactoryBean

通过查看Spring官方文档得知，Spring与Quartz集成使用的是SchedulerFactoryBean这个类，其所需引入Maven依赖如下。 
```Xml
	<dependency>
		<groupId>org.quartz-scheduler</groupId>
		<artifactId>quartz</artifactId>
		<version>2.3.0</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context-support</artifactId>
	</dependency>
```
以下是SchedulerFactoryBean类的部分关键代码。 
```Java
	package org.springframework.scheduling.quartz;
	 
	public class SchedulerFactoryBean extends SchedulerAccessor implements
			FactoryBean<Scheduler>, BeanNameAware, ApplicationContextAware,
			InitializingBean, DisposableBean, SmartLifecycle {
	 
		@Override
		public void afterPropertiesSet() throws Exception {
			//---------------------------省略部分代码--------------------------
			// Get Scheduler instance from SchedulerFactory.
			try {
				this.scheduler = createScheduler(schedulerFactory, this.schedulerName);
				populateSchedulerContext();
	 
				if (!this.jobFactorySet && !(this.scheduler instanceof RemoteScheduler)) {
					// Use AdaptableJobFactory as default for a local Scheduler, unless when
					// explicitly given a null value through the "jobFactory" bean property.
					this.jobFactory = new AdaptableJobFactory();
				}
				if (this.jobFactory != null) {
					if (this.jobFactory instanceof SchedulerContextAware) {
						((SchedulerContextAware) this.jobFactory).setSchedulerContext(this.scheduler.getContext());
					}
					this.scheduler.setJobFactory(this.jobFactory);
				}
			}
			//---------------------------省略部分代码--------------------------
		}
	}
```
从以上代码可以看出： SchedulerFactoryBean 使用 `AdaptableJobFactory` 对`Job对象`进行实例化，如果未指定则SchedulerFactoryBean会自动创建一个，在这里如果我们能够将 SchedulerFactoryBean 的 jobFactory 指定为我们自定义的工厂实例的话，我们就能够有机会在Job实例化完成之后对其进行处理，并将其纳入到Spring容器的管理之中。

例如下面这段代码，创建一个SchedulerFactoryBean 实例，并将其Job实例化的工厂指定为一个Spring容器中一个自定义的 TaskSchedulerFactory 实例。
```Java
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.scheduling.quartz.SchedulerFactoryBean;
	 
	@Configuration
	public class QuartzConfig {
		@Autowired
		private TaskSchedulerFactory taskSchedulerFactory;
	 
		@Bean
		public SchedulerFactoryBean schedulerFactoryBean() {
			SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
			schedulerFactoryBean.setJobFactory(taskSchedulerFactory);
			return schedulerFactoryBean;
		}
	 
		@Bean
		public Scheduler scheduler() {
			return schedulerFactoryBean().getScheduler();
		}
	}
```
# TaskSchedulerFactory

Quartz为我们提供了一个JobFactory接口，允许我们自定义实现创建Job的逻辑。 
```Java
	package org.quartz.spi;
	public interface JobFactory {
		Job newJob(TriggerFiredBundle bundle, Scheduler scheduler) throws SchedulerException;
	}
```
AdaptableJobFactory 就是 Spring 为我们提供的一个该接口的实现类，提供了创建Job实例的基本方法。  

<div align=center>

![AdaptableJobFactory示意图](http://pengjunlee.3vzhuji.net/static/springboot/07.png "AdaptableJobFactory示意图")
<div align=left>

自定义的工厂类 TaskSchedulerFactory 只需要继承 AdaptableJobFactory ，通过调用父类 AdaptableJobFactory 的方法来实现对Job的实例化，在Job实例化完以后，再调用自身方法为创建好的Job实例进行属性自动装配并将其纳入到Spring容器的管理之中。

创建好的 TaskSchedulerFactory 代码如下，项目地址：<http://download.csdn.net/download/pengjunlee/10187160>。 
```Java
	import org.quartz.spi.TriggerFiredBundle;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
	import org.springframework.scheduling.quartz.AdaptableJobFactory;
	import org.springframework.stereotype.Component;
	 
	@Component
	public class TaskSchedulerFactory extends AdaptableJobFactory
	{
	 
	    // 需要使用这个BeanFactory对Qurartz创建好Job实例进行后续处理，属于Spring的技术范畴.
	    @Autowired
	    private AutowireCapableBeanFactory capableBeanFactory;
	 
	    protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception
	    {
	        // 首先，调用父类的方法创建好Quartz所需的Job实例
	        Object jobInstance = super.createJobInstance(bundle);
	        // 然后，使用BeanFactory为创建好的Job实例进行属性自动装配并将其纳入到Spring容器的管理之中，属于Spring的技术范畴.
	        capableBeanFactory.autowireBean(jobInstance);
	        return jobInstance;
	    }
	}
```

# 普通SpringMVC工程中配置方法

首先，在Spring配置文件中添加 TaskSchedulerFactory 工厂 Bean 。 
```Xml
	<bean id="jobFactory" class="com.pengjunlee.task.TaskSchedulerFactory"></bean>
```
然后，再添加一个 SchedulerFactoryBean 类的 Bean ，并将其的jobFactory设置成我们自定义的 TaskSchedulerFactory 。  
```Xml
	<bean name="MyScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
	　　<!-- 其他属性省略 -->
	　　<property name="jobFactory" ref="jobFactory"></property>
	</bean>
```