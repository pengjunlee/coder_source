---
title: SpringBoot框架整合之--使用数据库配置Quartz定时任务
date: 2020-07-24 14:13:00
updated: 2020-07-24 14:13:00
tags: SpringBoot框架
categories: SpringBoot框架
keywords: Java, SpringBoot
type: 
description: SpringBoot中如何使用数据库配置Quartz定时任务?
top_img: http://pengjunlee.3vzhuji.net/static/img/top_img13.jpg
comments: true
cover: http://pengjunlee.3vzhuji.net/static/img/top_img13.jpg
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
在实际项目开发过程中，定时任务几乎是必不可少的。作为Java程序员用的最多的任务调度框架非Quartz莫属了。 

在Quartz中配置任务的方式很多，比较常见的就有`基于注解配置`、`基于XML等配置文件进行配置`和`通过数据库进行配置`三种配置方式，具体应该使用哪种方式来对定时任务进行配置需要根据你的实际业务场景来进行选择，这不是本文要讨论的重点，本文仅对如何使用数据库实现对定时任务的动态灵活配置进行简单示例和介绍。

为简单起见，本文使用了Spring Jpa来进行数据库操作。当应用启动时，TaskInitService类会从数据库中读取定时任务并进行加载。由于所有的定时任务都被存储在数据库中，用户可以通过相应的前端任务展示页面方便地对定时任务进行查看和管理，一旦有任务被修改，Scheduler调度器中的任务也会同步更新并立刻生效。项目的完整目录层次如下图所示。

<div align=center>

![quartz项目示意图](http://pengjunlee.3vzhuji.net/static/springboot/10.png "quartz项目示意图")
<div align=left>

# 添加依赖配置

为了使用Quartz和JPA，需要在工程POM文件中引入它们的Maven依赖，此外本示例工程中还使用到了`lombok`和`commons-lang`两个辅助工具包。 
```Xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.1.RELEASE</version>
	</parent>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- 添加MYSQL数据库驱动依赖 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<!-- 添加spring jpa依赖 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<!-- 添加quartz依赖 -->
		<dependency>
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz</artifactId>
			<version>2.3.0</version>
		</dependency>
		<!-- 添加lombok依赖 -->
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>
		<!-- 添加commons-lang3依赖 -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.7</version>
		</dependency>
	</dependencies>
```
在`application.properties`配置文件中除了要定义MYSQL数据库连接信息外，还需要添加如下JPA相关配置。 
```Properties
	########################################################
	### Java Persistence Api --  Spring jpa setting  #######
	########################################################
	# Specify the DBMS
	spring.jpa.database = MYSQL
	# Show or not log for each sql query
	spring.jpa.show-sql = true
	# Hibernate ddl auto (create, create-drop, update)
	spring.jpa.hibernate.ddl-auto = update
	# stripped before adding them to the entity manager)
	spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
```

# 创建定时任务实体类

定义两个任务实体类：`TaskCronJob`和`TaskSimJob`，用来与数据库中的表建立映射关系。  
```Java
	import javax.persistence.Entity;
	import javax.persistence.GeneratedValue;
	import javax.persistence.GenerationType;
	import javax.persistence.Id;
	import javax.persistence.Table;
	 
	import lombok.Data;
	 
	/**
	 * 基于Cron触发器的定时任务实体类
	 * 
	 * @author pengjunlee
	 *
	 */
	@Data
	@Entity
	@Table(name="task_cron_job")
	public class TaskCronJob {
	 
		// Job主键
		@Id
		@GeneratedValue(strategy = GenerationType.AUTO)
		private Long id;
	 
		// cron表达式
		private String cron;
	 
		// Job名称
		private String jobName;
	 
		// Job相关的类全名
		private String jobClassName;
	 
		// Job描述
		private String jobDescription;
	 
		// Job编号
		private String jobNumber;
	 
		// Job是否启用
		private Boolean enabled;
	 
		public TaskCronJob(String cron, String jobName, String jobClassName, String jobDescription, String jobNumber) {
			super();
			this.cron = cron;
			this.jobName = jobName;
			this.jobClassName = jobClassName;
			this.jobDescription = jobDescription;
			this.jobNumber = jobNumber;
		}
	 
		public TaskCronJob() {
		}
	}
```
<br/>
```Java
	import javax.persistence.Entity;
	import javax.persistence.GeneratedValue;
	import javax.persistence.GenerationType;
	import javax.persistence.Id;
	import javax.persistence.Table;
	 
	import lombok.Data;
	 
	/**
	 * 基于simple触发器的定时任务实体类
	 * 
	 * @author pengjunlee
	 *
	 */
	@Data
	@Entity
	@Table(name="task_sim_job")
	public class TaskSimJob {
	 
		// Job主键
		@Id
		@GeneratedValue(strategy = GenerationType.AUTO)
		private Long id;
	 
		// 间隔时间
		private Integer intervalTime;
	 
		// Job名称
		private String jobName;
	 
		// Job相关的类全名
		private String jobClassName;
	 
		// Job描述
		private String jobDescription;
	 
		// Job编号
		private String jobNumber;
	 
		// Job是否启用
		private Boolean enabled;
	 
		public TaskSimJob(Integer intervalTime, String jobName, String jobClassName, String jobDescription,
				String jobNumber) {
			super();
			this.intervalTime = intervalTime;
			this.jobName = jobName;
			this.jobClassName = jobClassName;
			this.jobDescription = jobDescription;
			this.jobNumber = jobNumber;
		}
	 
		public TaskSimJob() {
		}
	 
	}
```
对应的数据库字段及其格式如下图所示。 

<div align=center>

![quartz项目示意图](http://pengjunlee.3vzhuji.net/static/springboot/11.png "quartz项目示意图")
<div align=left>

# 任务实体持久化

通过上面定义的定义的两个任务实体类`TaskCronJob`和`TaskSimJob`，实现了使用Java的普通对象（POJO）与数据库表建立映射关系（ORM），接下来使用JPA来实现持久化。定义好的`TaskCronJobRepository`和`TaskSimJobRepository`均继承自`CrudRepository`，用来实现基本的持久化操作。 
```Java
	import org.springframework.data.repository.CrudRepository;
	import org.springframework.stereotype.Repository;
	 
	import com.pengjunlee.task.bean.TaskCronJob;
	 
	@Repository
	public interface TaskCronJobRepository extends CrudRepository<TaskCronJob, Long>
	{
	 
	}
```
<br/>
```Java
	import org.springframework.data.repository.CrudRepository;
	import org.springframework.stereotype.Repository;
	 
	import com.pengjunlee.task.bean.TaskSimJob;
	 
	@Repository
	public interface TaskSimJobRepository extends CrudRepository<TaskSimJob, Long>
	{
	 
	}
```

# 创建调度器工厂

TaskSchedulerFactory调度器工厂类用来生产Scheduler调度器对象，一般情况下应将其实现为单例。  
```Java
	import org.quartz.Scheduler;
	import org.quartz.SchedulerFactory;
	import org.quartz.impl.StdSchedulerFactory;
	import org.springframework.stereotype.Component;
	 
	import lombok.extern.slf4j.Slf4j;
	 
	/**
	 * 调度器工厂类，应当在Spring中将该类配置为单例
	 */
	@Component
	@Slf4j
	public class TaskSchedulerFactory {
	 
		private volatile Scheduler scheduler;
	 
		/**
		 * 获得scheduler实例
		 */
		public Scheduler getScheduler() {
			Scheduler s = scheduler;
			if (s == null) {
				synchronized (this) {
					s = scheduler;
					if (s == null) {
						// 双重检查
						try {
							SchedulerFactory sf = new StdSchedulerFactory();
							s = scheduler = sf.getScheduler();
						} catch (Exception e) {
							log.error("Get scheduler error :" + e.getMessage(), e);
						}
					}
				}
			}
	 
			return s;
		}
	}
```

# TaskUtils工具类

TaskUtils工具类负责为定时任务生产JobKey和TriggerKey。 
```Java
	import org.quartz.JobKey;
	import org.quartz.TriggerKey;
	 
	import com.pengjunlee.task.bean.TaskCronJob;
	import com.pengjunlee.task.bean.TaskSimJob;
	 
	/**
	 * 任务管理模块的工具类
	 */
	public class TaskUtils
	{
		
		/**
		 * 基于cron调度的Job的默认组名
		 */
		public static final String CRON_JOB_GROUP_NAME = "cron_task_group";
	 
		/**
		 * 基于simple调度的Job的默认组名
		 */
		public static final String SIM_JOB_GROUP_NAME = "sim_task_group";
	 
	    /**
	     * 产生JobKey
	     * 
	     * @param job
	     * @return
	     */
	    public static JobKey genCronJobKey(TaskCronJob job)
	    {
	        return new JobKey(job.getJobName().trim(), CRON_JOB_GROUP_NAME);
	    }
	 
	    /**
	     * 产生TriggerKey
	     * 
	     * @param job
	     * @return
	     */
	    public static TriggerKey genCronTriggerKey(TaskCronJob job)
	    {
	        return new TriggerKey("trigger_" + job.getJobName().trim(), CRON_JOB_GROUP_NAME);
	    }
	 
	    /**
	     * 产生JobKey
	     * 
	     * @param job
	     * @return
	     */
	    public static JobKey genSimJobKey(TaskSimJob job)
	    {
	        return new JobKey(job.getJobName().trim(), SIM_JOB_GROUP_NAME);
	    }
	 
	    /**
	     * 产生TriggerKey
	     * 
	     * @param job
	     * @return
	     */
	    public static TriggerKey genSimTriggerKey(TaskSimJob job)
	    {
	        return new TriggerKey("trigger_" + job.getJobName().trim(), SIM_JOB_GROUP_NAME);
	    }
	 
	    /**
	     * 判断是否两个trigger key是否相等
	     * 
	     * @param tk1
	     * @param tk2
	     * @return
	     */
	    public static boolean isTriggerKeyEqual(TriggerKey tk1, TriggerKey tk2)
	    {
	        return tk1.getName().equals(tk2.getName()) && ((tk1.getGroup() == null && tk2.getGroup() == null)
	                || (tk1.getGroup() != null && tk1.getGroup().equals(tk2.getGroup())));
	    }
	}
```

# 创建定时任务服务类

TaskCronJobService和TaskSimJobService通过调用上面定义好的TaskCronJobRepository和TaskSimJobRepository两个持久化类，在保存定时任务到数据库时实现了对调度器中定时任务的同步更新。 
```Java
	import javax.annotation.Resource;
	import javax.transaction.Transactional;
	 
	import org.quartz.CronScheduleBuilder;
	import org.quartz.CronTrigger;
	import org.quartz.Job;
	import org.quartz.JobBuilder;
	import org.quartz.JobDetail;
	import org.quartz.JobKey;
	import org.quartz.Scheduler;
	import org.quartz.Trigger;
	import org.quartz.TriggerBuilder;
	import org.quartz.TriggerKey;
	import org.springframework.stereotype.Service;
	 
	import com.pengjunlee.task.TaskSchedulerFactory;
	import com.pengjunlee.task.bean.TaskCronJob;
	import com.pengjunlee.task.repository.TaskCronJobRepository;
	import com.pengjunlee.task.util.TaskUtils;
	 
	import lombok.extern.slf4j.Slf4j;
	 
	/**
	 * 基于Cron的定时任务服务类
	 * 
	 * @author pengjunlee
	 */
	@Service
	@Slf4j
	public class TaskCronJobService
	{
	 
	    @Resource
	    private TaskCronJobRepository cronJobRepository;
	 
	    @Resource
	    private TaskSchedulerFactory schedulerFactory;
	 
	    // 在对任务进行保存时需同步更新调度器中的定时任务配置
	    @Transactional
	    public void save(TaskCronJob taskCronJob)
	    {
	        try
	        {
	            TaskCronJob job = findOne(taskCronJob.getId());
	            TriggerKey triggerKey = TaskUtils.genCronTriggerKey(job);
	            Scheduler scheduler = schedulerFactory.getScheduler();
	            JobKey jobKey = TaskUtils.genCronJobKey(job);
	            // 如果不同则代表着CRON表达式已经修改
	            if (!job.getCron().equals(taskCronJob.getCron()))
	            {
	                CronTrigger newTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey)
	                        .withSchedule(CronScheduleBuilder.cronSchedule(taskCronJob.getCron()).withMisfireHandlingInstructionDoNothing()).build();
	                // 更新任务
	                scheduler.rescheduleJob(triggerKey, newTrigger);
	            }
	            if (!job.getEnabled().equals(taskCronJob.getEnabled()))
	            {
	                // 如果状态为0则停止该任务
	                if (!taskCronJob.getEnabled())
	                {
	                    /* scheduler.unscheduleJob(triggerKey); */
	                    scheduler.pauseJob(jobKey);
	                    /* scheduler.deleteJob(jobKey); */
	                }
	                else
	                {
	                    Trigger trigger = scheduler.getTrigger(triggerKey);
	                    // trigger如果为null则说明scheduler中并没有创建该任务
	                    if (trigger == null)
	                    {
	                        Class<?> jobClass = Class.forName(job.getJobClassName().trim());
	                        @SuppressWarnings("unchecked")
	                        JobDetail jobDetail = JobBuilder.newJob((Class<? extends Job>) jobClass).withIdentity(jobKey)
	                                .withDescription(job.getJobDescription()).build();
	                        trigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey)
	                                .withSchedule(CronScheduleBuilder.cronSchedule(taskCronJob.getCron()).withMisfireHandlingInstructionDoNothing())
	                                .build();
	 
	                        scheduler.scheduleJob(jobDetail, trigger);
	                    }
	                    else
	                    {
	                        // 不为null则说明scheduler中有创建该任务,更新即可
	                        CronTrigger newTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey)
	                                .withSchedule(CronScheduleBuilder.cronSchedule(taskCronJob.getCron()).withMisfireHandlingInstructionDoNothing())
	                                .build();
	                        scheduler.rescheduleJob(triggerKey, newTrigger);
	                    }
	                }
	            }
	            job.setCron(taskCronJob.getCron());
	            job.setEnabled(taskCronJob.getEnabled());
	 
	            cronJobRepository.save(job);
	        }
	        catch (Exception e)
	        {
	            log.error("定时任务刷新失败...");
	            log.error(e.getMessage());
	        }
	    }
	 
	    public TaskCronJob findOne(Long id)
	    {
	        return cronJobRepository.findOne(id);
	    }
	 
	    public Iterable<TaskCronJob> findAll()
	    {
	        return cronJobRepository.findAll();
	    }
	 
	}
```
<br/>
```Java
	import javax.annotation.Resource;
	import javax.transaction.Transactional;
	 
	import org.quartz.Job;
	import org.quartz.JobBuilder;
	import org.quartz.JobDetail;
	import org.quartz.JobKey;
	import org.quartz.Scheduler;
	import org.quartz.SimpleScheduleBuilder;
	import org.quartz.SimpleTrigger;
	import org.quartz.Trigger;
	import org.quartz.TriggerBuilder;
	import org.quartz.TriggerKey;
	import org.springframework.stereotype.Service;
	 
	import com.pengjunlee.task.TaskSchedulerFactory;
	import com.pengjunlee.task.bean.TaskSimJob;
	import com.pengjunlee.task.repository.TaskSimJobRepository;
	import com.pengjunlee.task.util.TaskUtils;
	 
	import lombok.extern.slf4j.Slf4j;
	 
	/**
	 * 基于Cron的定时任务服务类
	 * 
	 * @author pengjunlee
	 */
	@Service
	@Slf4j
	public class TaskSimJobService
	{
	 
	    @Resource
	    private TaskSimJobRepository simJobRepository;
	 
	    @Resource
	    private TaskSchedulerFactory schedulerFactory;
	 
	    // 在对任务进行保存时需同步更新调度器中的定时任务配置
	    @Transactional
	    public void save(TaskSimJob taskSimJob)
	    {
	        try
	        {
	            TaskSimJob job = findOne(taskSimJob.getId());
	            TriggerKey triggerKey = TaskUtils.genSimTriggerKey(job);
	            Scheduler scheduler = schedulerFactory.getScheduler();
	            JobKey jobKey = TaskUtils.genSimJobKey(job);
	            // 如果不同则代表着CRON表达式已经修改
	            if (!job.getIntervalTime().equals(taskSimJob.getIntervalTime()))
	            {
	                SimpleTrigger newTrigger = TriggerBuilder.newTrigger()
	                        .withIdentity(triggerKey).forJob(jobKey).withSchedule(SimpleScheduleBuilder.simpleSchedule()
	                                .withIntervalInSeconds(taskSimJob.getIntervalTime()).withMisfireHandlingInstructionIgnoreMisfires().repeatForever())
	                        .build();
	                // 更新任务
	                scheduler.rescheduleJob(triggerKey, newTrigger);
	            }
	            if (!job.getEnabled().equals(taskSimJob.getEnabled()))
	            {
	                // 如果状态为0则停止该任务
	                if (!taskSimJob.getEnabled())
	                {
	                    /* scheduler.unscheduleJob(triggerKey); */
	                    scheduler.pauseJob(jobKey);
	                    /* scheduler.deleteJob(jobKey); */
	                }
	                else
	                {
	                    Trigger trigger = scheduler.getTrigger(triggerKey);
	                    // trigger如果为null则说明scheduler中并没有创建该任务
	                    if (trigger == null)
	                    {
	                        Class<?> jobClass = Class.forName(job.getJobClassName().trim());
	                        @SuppressWarnings("unchecked")
	                        JobDetail jobDetail = JobBuilder.newJob((Class<? extends Job>) jobClass).withIdentity(jobKey)
	                                .withDescription(job.getJobDescription()).build();
	                        trigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey)
	                                .withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(taskSimJob.getIntervalTime())
	                                        .withMisfireHandlingInstructionIgnoreMisfires().repeatForever())
	                                .build();
	                        scheduler.scheduleJob(jobDetail, trigger);
	                    }
	                    else
	                    {
	                        // 不为null则说明scheduler中有创建该任务,更新即可
	                        SimpleTrigger newTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey)
	                                .withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(taskSimJob.getIntervalTime())
	                                        .withMisfireHandlingInstructionIgnoreMisfires().repeatForever())
	                                .build();
	                        scheduler.rescheduleJob(triggerKey, newTrigger);
	                    }
	                }
	            }
	            job.setIntervalTime(taskSimJob.getIntervalTime());
	            job.setEnabled(taskSimJob.getEnabled());
	 
	            simJobRepository.save(job);
	        }
	        catch (Exception e)
	        {
	            log.error("定时任务刷新失败...");
	            log.error(e.getMessage());
	        }
	    }
	 
	    public TaskSimJob findOne(Long id)
	    {
	        return simJobRepository.findOne(id);
	    }
	 
	    public Iterable<TaskSimJob> findAll()
	    {
	        return simJobRepository.findAll();
	    }
	}
```

# 创建定时任务初始化服务类

TaskInitService类负责从数据库中获取到需要执行的定时任务并将任务添加到调度器的管理之中。 
```
	import static org.quartz.CronExpression.isValidExpression;
	 
	import java.util.List;
	 
	import javax.annotation.PostConstruct;
	import javax.annotation.Resource;
	 
	import org.apache.commons.lang3.StringUtils;
	import org.quartz.CronScheduleBuilder;
	import org.quartz.CronTrigger;
	import org.quartz.JobBuilder;
	import org.quartz.JobDetail;
	import org.quartz.JobKey;
	import org.quartz.Scheduler;
	import org.quartz.SchedulerException;
	import org.quartz.SimpleScheduleBuilder;
	import org.quartz.SimpleTrigger;
	import org.quartz.Trigger;
	import org.quartz.TriggerBuilder;
	import org.quartz.TriggerKey;
	import org.springframework.stereotype.Component;
	 
	import com.pengjunlee.task.TaskSchedulerFactory;
	import com.pengjunlee.task.bean.TaskCronJob;
	import com.pengjunlee.task.bean.TaskSimJob;
	import com.pengjunlee.task.service.TaskInitService;
	import com.pengjunlee.task.util.TaskUtils;
	 
	import lombok.extern.slf4j.Slf4j;
	 
	/**
	 * 定时任务初始化服务类
	 * @author pengjunlee
	 *
	 */
	@Component
	@Slf4j
	public class TaskInitService {
	 
		@Resource
		private TaskCronJobService taskCronJobService;
	 
		@Resource
		private TaskSimJobService taskSimJobService;
	 
		@Resource
		private TaskSchedulerFactory schedulerFactory;
	 
		/**
		 * 初始化
		 */
		@PostConstruct
		public void init() {
			Scheduler scheduler = schedulerFactory.getScheduler();
			if (scheduler == null) {
				log.error("初始化定时任务组件失败，Scheduler is null...");
				return;
			}
	 
			// 初始化基于cron时间配置的任务列表
			try {
				initCronJobs(scheduler);
			} catch (Exception e) {
				log.error("init cron tasks error," + e.getMessage(), e);
			}
	 
			// 初始化基于固定间隔时间配置的任务列表
			try {
				initSimJobs(scheduler);
			} catch (Exception e) {
				log.error("init sim tasks error," + e.getMessage(), e);
			}
	 
			try {
				log.info("The scheduler is starting...");
				scheduler.start(); // start the scheduler
			} catch (Exception e) {
				log.error("The scheduler start is error," + e.getMessage(), e);
			}
		}
	 
		/**
		 * 初始化任务（基于cron触发器）
		 * 
		 */
		private void initCronJobs(Scheduler scheduler) throws Exception {
			Iterable<TaskCronJob> jobList = taskCronJobService.findAll();
			if (jobList != null) {
				for (TaskCronJob job : jobList) {
					scheduleCronJob(job, scheduler);
				}
			}
		}
	 
		/**
		 * 初始化任务（基于simple触发器）
		 * 
		 */
		private void initSimJobs(Scheduler scheduler) throws Exception {
			Iterable<TaskSimJob> jobList = taskSimJobService.findAll();
			if (jobList != null) {
				for (TaskSimJob job : jobList) {
					scheduleSimJob(job, scheduler);
				}
			}
		}
	 
		/**
		 * 安排任务(基于simple触发器)
		 * 
		 * @param job
		 * @param scheduler
		 */
		private void scheduleSimJob(TaskSimJob job, Scheduler scheduler) {
			if (job != null && StringUtils.isNotBlank(job.getJobName()) && StringUtils.isNotBlank(job.getJobClassName())
					&& scheduler != null) {
				if (!job.getEnabled()) {
					return;
				}
	 
				try {
					JobKey jobKey = TaskUtils.genSimJobKey(job);
	 
					if (!scheduler.checkExists(jobKey)) {
						// This job doesn't exist, then add it to scheduler.
						log.info("Add new simple job to scheduler, jobName = " + job.getJobName());
						this.newJobAndNewSimTrigger(job, scheduler, jobKey);
					} else {
						log.info("Update simple job to scheduler, jobName = " + job.getJobName());
						this.updateSimTriggerOfJob(job, scheduler, jobKey);
					}
	 
				} catch (Exception e) {
					log.error("ScheduleCronJob is error," + e.getMessage(), e);
				}
			} else {
				log.error("Method scheduleSimJob arguments are invalid.");
			}
		}
	 
		/**
		 * 安排任务(基于cron触发器)
		 * 
		 * @param job
		 * @param scheduler
		 */
		private void scheduleCronJob(TaskCronJob job, Scheduler scheduler) {
			if (job != null && StringUtils.isNotBlank(job.getJobName()) && StringUtils.isNotBlank(job.getJobClassName())
					&& StringUtils.isNotBlank(job.getCron()) && scheduler != null) {
				if (!job.getEnabled()) {
					return;
				}
	 
				try {
					JobKey jobKey = TaskUtils.genCronJobKey(job);
	 
					if (!scheduler.checkExists(jobKey)) {
						// This job doesn't exist, then add it to scheduler.
						log.info("Add new cron job to scheduler, jobName = " + job.getJobName());
						this.newJobAndNewCronTrigger(job, scheduler, jobKey);
					} else {
						log.info("Update cron job to scheduler, jobName = " + job.getJobName());
						this.updateCronTriggerOfJob(job, scheduler, jobKey);
					}
				} catch (Exception e) {
					log.error("ScheduleCronJob is error," + e.getMessage(), e);
				}
			} else {
				log.error("Method scheduleCronJob arguments are invalid.");
			}
		}
	 
		/**
		 * 新建job和trigger到scheduler(基于simple触发器)
		 * 
		 * @param job
		 * @param scheduler
		 * @param jobKey
		 * @throws SchedulerException
		 * @throws ClassNotFoundException
		 */
		@SuppressWarnings({ "rawtypes", "unchecked" })
		private void newJobAndNewSimTrigger(TaskSimJob job, Scheduler scheduler, JobKey jobKey)
				throws SchedulerException, ClassNotFoundException {
			TriggerKey triggerKey = TaskUtils.genSimTriggerKey(job);
			// get a Class object by string class name of job;
			Class jobClass = Class.forName(job.getJobClassName().trim());
			JobDetail jobDetail = JobBuilder.newJob(jobClass).withIdentity(jobKey).withDescription(job.getJobDescription())
					.build();
	 
			SimpleTrigger trigger = null;
			int intervalInSec = job.getIntervalTime();
			if (intervalInSec > 0) {
				// repeat the job every interval seconds.
				trigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey).startNow()
						.withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(intervalInSec)).build();
			} else {
				// totally execute the job once.
				trigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey).startNow()
						.withSchedule(SimpleScheduleBuilder.repeatSecondlyForTotalCount(1)).build();
			}
	 
			scheduler.scheduleJob(jobDetail, trigger);
		}
	 
		/**
		 * 更新job的trigger(基于simple触发器)
		 * 
		 * @param job
		 * @param scheduler
		 * @param jobKey
		 * @throws SchedulerException
		 */
		private void updateSimTriggerOfJob(TaskSimJob job, Scheduler scheduler, JobKey jobKey) throws SchedulerException {
			TriggerKey triggerKey = TaskUtils.genSimTriggerKey(job);
			int intervalInSec = job.getIntervalTime();
	 
			List<? extends Trigger> triggers = scheduler.getTriggersOfJob(jobKey);
	 
			for (int i = 0; triggers != null && i < triggers.size(); i++) {
				Trigger trigger = triggers.get(i);
				TriggerKey curTriggerKey = trigger.getKey();
	 
				if (TaskUtils.isTriggerKeyEqual(triggerKey, curTriggerKey)) {
					SimpleTrigger newTrigger = null;
					if (intervalInSec > 0) {
						// repeat the job every interval seconds.
						newTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey).startNow()
								.withSchedule(SimpleScheduleBuilder.repeatSecondlyForever(intervalInSec)).build();
					} else {
						// totally execute the job once.
						newTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey).startNow()
								.withSchedule(SimpleScheduleBuilder.repeatSecondlyForTotalCount(1)).build();
					}
					scheduler.rescheduleJob(curTriggerKey, newTrigger);
				} else {
					// different trigger key // The trigger key is illegal,
					// unschedule this trigger
					scheduler.unscheduleJob(curTriggerKey);
				}
			}
		}
	 
		/**
		 * 新建job和trigger到scheduler(基于cron触发器)
		 * 
		 * @param job
		 * @param scheduler
		 * @param jobKey
		 * @throws SchedulerException
		 * @throws ClassNotFoundException
		 */
		@SuppressWarnings({ "rawtypes", "unchecked" })
		private void newJobAndNewCronTrigger(TaskCronJob job, Scheduler scheduler, JobKey jobKey)
				throws SchedulerException, ClassNotFoundException {
			TriggerKey triggerKey = TaskUtils.genCronTriggerKey(job);
	 
			String cronExpr = job.getCron();
			if (!isValidExpression(cronExpr)) {
				return;
			}
	 
			// get a Class object by string class name of job;
			Class jobClass = Class.forName(job.getJobClassName().trim());
			JobDetail jobDetail = JobBuilder.newJob(jobClass).withIdentity(jobKey).withDescription(job.getJobDescription())
					.build();
			CronTrigger trigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey)
					.withSchedule(CronScheduleBuilder.cronSchedule(cronExpr).withMisfireHandlingInstructionDoNothing())
					.build();
	 
			scheduler.scheduleJob(jobDetail, trigger);
		}
	 
		/**
		 * 更新job的trigger(基于cron触发器)
		 * 
		 * @param job
		 * @param scheduler
		 * @param jobKey
		 * @throws SchedulerException
		 */
		private void updateCronTriggerOfJob(TaskCronJob job, Scheduler scheduler, JobKey jobKey) throws SchedulerException {
			TriggerKey triggerKey = TaskUtils.genCronTriggerKey(job);
			String cronExpr = job.getCron().trim();
	 
			List<? extends Trigger> triggers = scheduler.getTriggersOfJob(jobKey);
	 
			for (int i = 0; triggers != null && i < triggers.size(); i++) {
				Trigger trigger = triggers.get(i);
				TriggerKey curTriggerKey = trigger.getKey();
	 
				if (TaskUtils.isTriggerKeyEqual(triggerKey, curTriggerKey)) {
					if (trigger instanceof CronTrigger
							&& cronExpr.equalsIgnoreCase(((CronTrigger) trigger).getCronExpression())) {
						// Don't need to do anything.
					} else {
						if (isValidExpression(job.getCron())) {
							// Cron expression is valid, build a new trigger and
							// replace the old one.
							CronTrigger newTrigger = TriggerBuilder.newTrigger().withIdentity(triggerKey).forJob(jobKey)
									.withSchedule(CronScheduleBuilder.cronSchedule(cronExpr)
											.withMisfireHandlingInstructionDoNothing())
									.build();
							scheduler.rescheduleJob(curTriggerKey, newTrigger);
						}
					}
				} else {
					// different trigger key ,The trigger key is illegal, unschedule
					// this trigger
					scheduler.unscheduleJob(curTriggerKey);
				}
	 
			}
	 
		}
	 
	}
```

# 创建定时任务实现类

具体的定时任务实现类需要实现Job接口。 
```Java
	import org.quartz.Job;
	import org.quartz.JobExecutionContext;
	import org.quartz.JobExecutionException;
	 
	public class MyJob implements Job {
	 
		@Override
		public void execute(JobExecutionContext arg0) throws JobExecutionException {
			System.out.println("执行定时任务:MyJob.execute()...");
		}
	 
	}
```
本文项目源码已上传至CSDN，资源地址：<http://download.csdn.net/download/pengjunlee/10142680>