---
layout: '[layout]'
title: SpringBoot整合Quartz实现定时任务（单任务、多任务）
date: 2018-05-07 13:50:24
tags:
- SpringBoot
- Java
- Quartz
categories:
- 后端
cover: https://file.buildworld.cn/img/u=2551084964,3275034231&fm=26&gp=0.jpg
---

#### 前言
>Quartz是OpenSymphony开源组织在Job scheduling领域又一个开源项目，它可以与J2EE与J2SE应用程序相结合也可以单独使用。Quartz可以用来创建简单或为运行十个，百个，甚至是好几万个Jobs这样复杂的程序。Jobs可以做成标准的Java组件或 EJBs

#### 一、添加依赖

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.1</version>
        </dependency>

        <dependency><!-- 该依赖必加，里面有sping对schedule的支持 -->
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>

        <!--必须添加，要不然会出错，项目无法启动-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>

    </dependencies>
```
#### 二、相关配置
> 由于springboot是无xml配置，所以此处我们采用bean注解的方式实现quartz的配置

```java
@Configuration
public class QuartzConfigration {
    @Bean(name = "jobDetail")
    public MethodInvokingJobDetailFactoryBean detailFactoryBean(SchedulerTask task) {
        // ScheduleTask为需要执行的任务
        MethodInvokingJobDetailFactoryBean jobDetail = new MethodInvokingJobDetailFactoryBean();
        /*
         *  是否并发执行
         *  例如每5s执行一次任务，但是当前任务还没有执行完，就已经过了5s了，
         *  如果此处为true，则下一个任务会bing执行，如果此处为false，则下一个任务会等待上一个任务执行完后，再开始执行
         */
        jobDetail.setConcurrent(true);

        jobDetail.setName("scheduler");// 设置任务的名字
        jobDetail.setGroup("scheduler_group");// 设置任务的分组，这些属性都可以存储在数据库中，在多任务的时候使用

        /*
         * 这两行代码表示执行task对象中的scheduleTest方法。定时执行的逻辑都在scheduleTest。
         */
        jobDetail.setTargetObject(task);

        jobDetail.setTargetMethod("start");
        return jobDetail;
    }

    @Bean(name = "jobTrigger")
    public CronTriggerFactoryBean cronJobTrigger(MethodInvokingJobDetailFactoryBean jobDetail) {
        CronTriggerFactoryBean tigger = new CronTriggerFactoryBean();
        tigger.setJobDetail(jobDetail.getObject());
        tigger.setCronExpression("0/2 * * * * ?");// 表示每隔2秒钟执行一次
        //tigger.set
        tigger.setName("myTigger");// trigger的name
        return tigger;

    }

    @Bean(name = "scheduler")
    public SchedulerFactoryBean schedulerFactory(Trigger cronJobTrigger) {
        SchedulerFactoryBean bean = new SchedulerFactoryBean();
        //设置是否任意一个已定义的Job会覆盖现在的Job。默认为false，即已定义的Job不会覆盖现有的Job。
        bean.setOverwriteExistingJobs(true);
        // 延时启动，应用启动5秒后  ，定时器才开始启动
        bean.setStartupDelay(5);
        // 注册定时触发器
        bean.setTriggers(cronJobTrigger);
        return bean;
    }
    //多任务时的Scheduler，动态设置Trigger。一个SchedulerFactoryBean可能会有多个Trigger
    @Bean(name = "multitaskScheduler")
    public SchedulerFactoryBean schedulerFactoryBean(){
        SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
        return schedulerFactoryBean;
    }
}

```

#### 三、应用场景
##### 1、单任务执行，并且通过控制器的接口实现时间间隔的动态修改
###### 1）新建一个任务SchedulerTask.java

```java
<!--三个注释都要加上-->
@Configuration
@Component
@EnableScheduling
public class SchedulerTask {

    public void start() throws InterruptedException {
        System.out.println("活动开始！！！"+new Date());
    }
}

```
###### 2）控制器执行

```java
@Controller
public class QuartzController {

    @Resource(name = "jobDetail")
    private JobDetail jobDetail;

    @Resource(name = "scheduler")
    private Scheduler scheduler;

    @Resource(name = "jobTrigger")
    private CronTrigger cronTrigger;


    @ResponseBody
    @GetMapping("/{second}/quart")
    public Object quartzTest(@PathVariable("second")Integer second) throws SchedulerException {
        CronTrigger cron  = (CronTrigger) scheduler.getTrigger(cronTrigger.getKey());
        String currentCron = cron.getCronExpression();// 当前Trigger使用的
        System.err.println("当前trigger使用的-"+currentCron);

        //修改每隔?秒执行任务
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0/"+second+" * * * * ?");

        // 按新的cronExpression表达式重新构建trigger
        cron = cron.getTriggerBuilder().withIdentity(cronTrigger.getKey())
                .withSchedule(scheduleBuilder).build();

        scheduler.rescheduleJob(cronTrigger.getKey(),cron);

        return "-这是quartz测试！";
    }
}

```

##### 2、多任务场景
##### ==Part1==
> :新建多个Tast.java，也就是一开始就设定好了任务，我们假设为 伪多任务

###### 1）新建多个任务

```java
public class SchedulerJob1 implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.err.println("这是第一个任务"+new Date());

    }
}


public class SchedulerJob2 implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.err.println("这是第二个任务"+new Date());
    }
}

```
###### 2）控制器

> 通过下面的代码就可以实现两个任务交替执行，但是我们一般的应用场景是不确定的任务和执行时间，请看下一个解决方案
```java
@Controller
public class QuartzController2 {
    @Resource(name = "multitaskScheduler")
    private Scheduler scheduler;

    @ResponseBody
    @RequestMapping("task1")
    public Object task1() throws SchedulerException {
        //配置定时任务对应的Job，这里执行的是ScheduledJob类中定时的方法
        JobDetail jobDetail = JobBuilder.newJob(SchedulerJob1.class).withIdentity("job1", "group1").build();
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0/3 * * * * ?");
        // 每3s执行一次
        CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "group1").withSchedule(scheduleBuilder).build();
        scheduler.scheduleJob(jobDetail, cronTrigger);

        return "任务1";
    }

    @ResponseBody
    @RequestMapping("task2")
    public Object task1() throws SchedulerException {
        //配置定时任务对应的Job，这里执行的是ScheduledJob类中定时的方法
        JobDetail jobDetail = JobBuilder.newJob(SchedulerJob2.class).withIdentity("job2", "group1").build();
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0/6 * * * * ?");
        // 每3s执行一次
        CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity("trigger2", "group1").withSchedule(scheduleBuilder).build();
        scheduler.scheduleJob(jobDetail, cronTrigger);

        return "任务1";
    }
}
```

##### ==Part2:==
> 有时候我们有新建活动之类的场景，这种场景就是活动数目不确定，活动开始时间不确定，所以我们需要用其他的方案来解决！


```
思路：
主要是通过逻辑代码实现任务开始时间的修改，但是必须要修改任务名称和触发器（trigger）名称的修改，确保多个任务之间名称不一致，否则会报错！

根据任务我们也可以定制使用数据库轮询的方式，确保任务的开启！
主要是为了解决服务器关起和其它因素导致任务终止！
```
###### 1）任务类

```java
public class SchedulerJob2 implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {

        //这里可以获取控制器绑定的值，实际应用中可以设置为某个活动的id,以便进行数据库操作
        Object jobName = jobExecutionContext.getJobDetail().getKey();
        System.err.println("这是"+jobName+"任务"+new Date());
    }
}

```
###### 2）控制器类

```java
 @ResponseBody
    @RequestMapping("task2/{jobName}")
    public Object task2(@PathVariable(value = "jobName") String jobName) throws SchedulerException {
        //配置定时任务对应的Job，这里执行的是ScheduledJob类中定时的方法
        JobDetail jobDetail = JobBuilder
                .newJob(SchedulerJob2.class)
                .usingJobData("jobName",jobName)
                .withIdentity(jobName, "group1")
                .build();

        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0/2 * * * * ?");
        // 每3s执行一次
        CronTrigger cronTrigger = TriggerBuilder.newTrigger()
                .withIdentity("trigger2"+jobName, "group1")
                .withSchedule(scheduleBuilder)
                .build();

        scheduler.scheduleJob(jobDetail,cronTrigger);

        return jobName;
    }

```
###### 3）获取所有的在线job

```java
 @ResponseBody
    @RequestMapping("jobs")
    public Object Jobs() throws SchedulerException {

        Set<TriggerKey> triggerKeys = scheduler.getTriggerKeys(GroupMatcher.anyTriggerGroup());

        //获取所有的job集合
        Set<JobKey> jobKeys = scheduler.getJobKeys(GroupMatcher.anyJobGroup());

        //可以在这进行线上任务和数据库任务匹配操作，确保该进行的活动进行活动
        
        return jobKeys;
    }
```
job集合

![](https://ws1.sinaimg.cn/large/005EneYkgy1fr2q86loa8j3056067dfn.jpg)`





