---
title: Java知识点系列之--使用Quartz进行作业任务调度
date: 2020-07-20 13:06:00
updated: 2020-07-20 13:06:00
tags: Java知识点
categories: Java知识点
keywords: Java, 知识点
type: 
description: 如何使用Quartz进行作业任务调度？
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img6.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img6.jpg
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
**Quartz**是一个完全由java编写的开源作业调度框架，为在**Java**应用程序中进行作业调度提供了简单却强大的机制。

**Quartz**允许开发人员根据时间间隔（或天数）来调度作业。它实现了作业和触发器的多对多关系，还能把多个作业与不同的触发器关联。整合了 **Quartz** 的应用程序可以重用来自不同事件的作业，还可以为一个事件组合多个作业。 

由 **James House** 创建并最初于2001年春天被加入**sourceforge**工程。之后归入**OpenSymphony**开源组织(2010年11月份关闭)。**Terracotta**公司在2009年收购了著名的**Java**开源缓存项目**Ehcache**以及**Java**任务调度项目**Quartz**。 

官网下载地址：<http://www.quartz-scheduler.org/downloads/>

# 1、设置开始时间
```Java
	import org.quartz.Job;
	import org.quartz.JobExecutionContext;
	import org.quartz.JobExecutionException;
	 
	public class MyJob implements Job {
	 
		@Override
		public void execute(JobExecutionContext arg0) throws JobExecutionException {
			System.out.println("执行定时任务:MyJob.execute()...，当前时间："+System.currentTimeMillis());
		}
	}
```
<br/>
```Java
	public static void testStartAt() throws SchedulerException {
		SchedulerFactory sf = new StdSchedulerFactory();
		Scheduler sched = sf.getScheduler();
		Date runTime = DateBuilder.evenSecondDate(new Date());
		JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity("jobName1", "jobGroup1").build();
		Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName1", "triggerGroup1").startAt(runTime)
				.build();
		sched.scheduleJob(job, trigger);
		sched.start();
	}
```

# 2、设置结束时间
```Java
	public static void testEndAt() throws SchedulerException {
		SchedulerFactory sf = new StdSchedulerFactory();
		Scheduler sched = sf.getScheduler();
		Date runTime = DateBuilder.evenSecondDate(new Date());
		Date endTime = DateBuilder.evenMinuteDate(new Date());
		JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity("jobName2", "jobGroup2").build();
		Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName2", "triggerGroup2").startAt(runTime)
				.withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(1).repeatForever())
				.endAt(endTime).build();
		sched.scheduleJob(job, trigger);
		sched.start();
	}
```

# 3、简单触发器
```Java
`SimpleScheduleBuilder.simpleSchedule()` 可以设置作业执行的时间间隔和重复执行的次数。  

	public static void testSimJob() throws SchedulerException {
		SchedulerFactory sf = new StdSchedulerFactory();
		Scheduler sched = sf.getScheduler();
		JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity("jobName3", "jobGroup3").build();
		Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName3", "triggerGroup3")
				.withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(1).repeatForever()).build();
		sched.scheduleJob(job, trigger);
		sched.start();
	}
```

# 4、Cron触发器
```Java
`CronScheduleBuilder.cronSchedule("0/5 * * * * ?")` 使用Unix cron表达式对作业执行周期进行设置。  

	public static void testCronJob() throws SchedulerException {
		SchedulerFactory sf = new StdSchedulerFactory();
		Scheduler sched = sf.getScheduler();
		JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity("jobName4", "jobGroup4").build();
		Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName4", "triggerGroup4")
				.withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")).build();
		sched.scheduleJob(job, trigger);
		sched.start();
	}
```

# 5、作业监听器
```Java
	import org.quartz.JobDataMap;
	import org.quartz.JobExecutionContext;
	import org.quartz.JobExecutionException;
	import org.quartz.JobKey;
	import org.quartz.JobListener;
	 
	public class MyJobListener implements JobListener {
	 
		@Override
		public String getName() {
			return "MyJobListener";
		}
	 
		/**
		 * (1) 任务执行之前执行 Called by the Scheduler when a JobDetail is about to be
		 * executed (an associated Trigger has occurred).
		 */
		@Override
		public void jobToBeExecuted(JobExecutionContext context) {
			System.out.println("MyJobListener.jobToBeExecuted()");
			// 获取usingJobData中传入的数据
			JobKey key = context.getJobDetail().getKey();
	 
			JobDataMap dataMap = context.getJobDetail().getJobDataMap();
	 
			String jobSays = dataMap.getString("jobSays");
			int currentYear = dataMap.getInt("currentYear");
	 
			System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and currentYear is: " + currentYear);
		}
	 
		/**
		 * (2)
		 * 这个方法正常情况下不执行,但是如果当TriggerListener中的vetoJobExecution方法返回true时,那么执行这个方法.
		 * 需要注意的是 如果方法(2)执行 那么(1),(3)这个俩个方法不会执行,因为任务被终止了嘛. Called by the Scheduler
		 * when a JobDetail was about to be executed (an associated Trigger has
		 * occurred), but a TriggerListener vetoed it's execution.
		 */
		@Override
		public void jobExecutionVetoed(JobExecutionContext context) {
			System.out.println("MyJobListener.jobExecutionVetoed()");
		}
	 
		/**
		 * (3) 任务执行完成后执行,jobException如果它不为空则说明任务在执行过程中出现了异常 Called by the Scheduler
		 * after a JobDetail has been executed, and be for the associated Trigger's
		 * triggered(xx) method has been called.
		 */
		@Override
		public void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException) {
			System.out.println("MyJobListener.jobWasExecuted()");
	 
			// 获取usingJobData中传入的数据
			JobKey key = context.getJobDetail().getKey();
	 
			JobDataMap dataMap = context.getJobDetail().getJobDataMap();
	 
			String jobSays = dataMap.getString("jobSays");
			int currentYear = dataMap.getInt("currentYear");
	 
			System.out.println("Instance " + key + " of DumbJob says: " + jobSays + " and current year is  " + currentYear);
		}
	 
	}
```
<br/>
```Java
	public static void testJobListener() throws SchedulerException {
		JobKey jobKey = new JobKey("jobName5", "jobGroup5");
		JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity(jobKey).build();
 
		Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName5", "triggerGroup5")
				.withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")).build();
 
		Scheduler scheduler = new StdSchedulerFactory().getScheduler();
 
		// Listener attached to jobKey
		// scheduler.getListenerManager().addJobListener(new MyJobListener(),
		// KeyMatcher.keyEquals(jobKey));
 
		// Listener attached to group
		scheduler.getListenerManager().addJobListener(new MyJobListener(), GroupMatcher.jobGroupEquals("jobGroup5"));
 
		scheduler.start();
		scheduler.scheduleJob(job, trigger);
	}
```

# 6、启动多个作业
```Java
	import org.quartz.Job;
	import org.quartz.JobExecutionContext;
	import org.quartz.JobExecutionException;
	 
	public class YourJob implements Job {
	 
		@Override
		public void execute(JobExecutionContext arg0) throws JobExecutionException {
			System.out.println("执行定时任务:YourJob.execute()...，当前时间：" + System.currentTimeMillis());
		}
	 
	}
```
<br/>
```Java
	public static void testMultiJobs() throws SchedulerException {
		JobKey jobKeyA = new JobKey("jobName6A", "jobGroup6");
		JobDetail jobA = JobBuilder.newJob(MyJob.class).withIdentity(jobKeyA).build();
 
		JobKey jobKeyB = new JobKey("jobName6B", "jobGroup6");
		JobDetail jobB = JobBuilder.newJob(YourJob.class).withIdentity(jobKeyB).build();
 
		Trigger trigger1 = TriggerBuilder.newTrigger().withIdentity("triggerName6A", "triggerGroup6")
				.withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")).build();
 
		Trigger trigger2 = TriggerBuilder.newTrigger().withIdentity("triggerName6B", "triggerGroup6")
				.withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")).build();
 
		Scheduler scheduler = new StdSchedulerFactory().getScheduler();
 
		scheduler.start();
		scheduler.scheduleJob(jobA, trigger1);
		scheduler.scheduleJob(jobB, trigger2);
	}
```

# 7、查看所有作业
```Java
	@SuppressWarnings("unchecked")
	public static void testQueryJobs() throws SchedulerException {
		Scheduler scheduler = new StdSchedulerFactory().getScheduler();
		for (String groupName : scheduler.getJobGroupNames()) {
			for (JobKey jobKey : scheduler.getJobKeys(GroupMatcher.jobGroupEquals(groupName))) {
				String jobName = jobKey.getName();
				String jobGroup = jobKey.getGroup();
				List<Trigger> triggers = (List<Trigger>) scheduler.getTriggersOfJob(jobKey);
				Date nextFireTime = triggers.get(0).getNextFireTime();
				System.out.println("[jobName] : " + jobName + " [groupName] : " + jobGroup + " - " + nextFireTime);
			}
		}
	}
```

# 8、手动触发作业
```Java
	public static void testTriggerJob() throws SchedulerException, InterruptedException {
		SchedulerFactory sf = new StdSchedulerFactory();
		Scheduler sched = sf.getScheduler();
		Date runTime = DateBuilder.evenSecondDate(new Date());
		JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity("jobName8", "jobGroup8").build();
		Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName8", "triggerGroup8").startAt(runTime)
				.build();
		sched.scheduleJob(job, trigger);
		sched.start();
		// 手动触发两次
		sched.triggerJob(new JobKey("jobName8", "jobGroup8"));
		sched.triggerJob(new JobKey("jobName8", "jobGroup8"));
 
	}
```

# 9、传递参数
```Java
	public static void testJobData() throws SchedulerException {
		SchedulerFactory sf = new StdSchedulerFactory();
		Scheduler sched = sf.getScheduler();
		Date runTime = DateBuilder.evenSecondDate(new Date());
		JobDetail job = JobBuilder.newJob(MyJob.class).withIdentity("jobName9", "jobGroup9")
				.usingJobData("jobSays", "Hello Quartz!").usingJobData("currentYear", 2017).build();
		Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName9", "triggerGroup9").startAt(runTime)
				.build();
		sched.scheduleJob(job, trigger);
		sched.start();
 
		sched.getListenerManager().addJobListener(new MyJobListener(), GroupMatcher.jobGroupEquals("jobGroup9"));
 
	}
```

# 10、取消/删除作业
```Java
	public static void testCancelJob(Scheduler sched) throws SchedulerException {
		// removes the given trigger
		sched.unscheduleJob(new TriggerKey("triggerName10", "triggerGroup10"));
		// removes all triggers to the given job
		sched.deleteJob(new JobKey("jobName10", "jobGroup10"));
	}
```

# 11、作业出错时自动再执行
```Java
	import org.quartz.Job;
	import org.quartz.JobExecutionContext;
	import org.quartz.JobExecutionException;
	import org.quartz.JobKey;
	 
	public class ErrorJob implements Job {
	 
		private int count = 0;
	 
		@Override
		public void execute(JobExecutionContext context) throws JobExecutionException {
	 
			JobKey jobKey = context.getJobDetail().getKey();
	 
			try {
				int zero = 0;
				@SuppressWarnings("unused")
				int calculation = 1 / zero;
			} catch (Exception e) {
				System.out.println("Instance " + jobKey + "执行出错了!!!");
				JobExecutionException e2 = new JobExecutionException(e);
				// allow 5 retries
				if ((++count) >= 5) {
					// make sure it doesn't run again
					e2.setUnscheduleAllTriggers(true);
				} else {
					// fire it again
					e2.setRefireImmediately(true);
				}
				throw e2;
			}
	 
		}
	 
	}
```
<br/>
```Java
	public static void testJobRetry() throws SchedulerException {
		try {
			SchedulerFactory sf = new StdSchedulerFactory();
			Scheduler sched = sf.getScheduler();
			Date runTime = DateBuilder.evenSecondDate(new Date());
			JobDetail job = JobBuilder.newJob(ErrorJob.class).withIdentity("jobName11", "jobGroup11").build();
			Trigger trigger = TriggerBuilder.newTrigger().withIdentity("triggerName11", "triggerGroup11")
					.startAt(runTime).build();
			sched.scheduleJob(job, trigger);
			sched.start();
		} catch (JobExecutionException e) {
		}
	}
```

# 12、JWatch - A Quartz Monitor

JWatch官网地址：<http://code.google.com/p/jwatch/>
<div align=center>

![quartz示意图](http://pengjunlee.3vzhuji.net/static/javacore/3.png "quartz示意图")
<div align=center>

![quartz示意图](http://pengjunlee.3vzhuji.net/static/javacore/4.png "quartz示意图")
<div align=left>

# 13、CronMaker - Cron表达式生成器

CronMaker网址：<http://www.cronmaker.com/>
<div align=center>

![quartz示意图](http://pengjunlee.3vzhuji.net/static/javacore/5.png "quartz示意图")
<div align=left>

# 14、相关资源下载

JWatch管理工具：<http://download.csdn.net/download/pengjunlee/10145202>

Quartz中文API：<http://download.csdn.net/download/pengjunlee/10142840>

本文项目源码：<http://download.csdn.net/download/pengjunlee/10142832>

Quartz英文API：<http://download.csdn.net/download/pengjunlee/10142674>  