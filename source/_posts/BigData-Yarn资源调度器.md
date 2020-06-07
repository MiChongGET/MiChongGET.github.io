---
title: BigData--Yarn资源调度器
date: 2020-06-03 16:19:03
tags:
- Hadoop
- Yarn
categories:
- BigData
description: 狗子今天退役了💔
top_img: https://file.buildworld.cn/img/xiaohuichai.jpg
cover: https://file.buildworld.cn/img/15c04d5f9a84cf49ec8d98fa906a0eaf_346b-hezpzwu1456399.jpg
---
## Yarn资源调度器

**YARN主要由`ResourceManager`、`NodeManager`、`ApplicationMaster`和`Container`等组件构成**

### 1、YARN架构

![](https://file.buildworld.cn/img/20200603135942.png)

### 2、YARN工作机制

![](https://file.buildworld.cn/img/image-20200603142713125.png)

#### 工作机制详解

- ​    （1）MR程序提交到客户端所在的节点。

- ​	（2）`YarnRunner`向`ResourceManager`申请一个`Application`。

- ​	（3）RM将该应用程序的资源路径返回给`YarnRunner`。

- ​	（4）该程序将运行所需资源提交到HDFS上。

- ​	（5）程序资源提交完毕后，申请运行`mrAppMaster`。

- ​	（6）RM将用户的请求初始化成一个Task。

- ​	（7）其中一个`NodeManager`领取到Task任务。

- ​	（8）该`NodeManager`创建容器`Container`，并产生MRAppmaster。

- ​	（9）`Container`从HDFS上拷贝资源到本地。

- ​	（10）`MRAppmaster`向RM 申请运行`MapTask`资源。

- ​	（11）RM将运行`MapTask`任务分配给另外两个`NodeManager`，另两个`NodeManager`分别领取任务并创建容器。

- ​	（12）MR向两个接收到任务的`NodeManager`发送程序启动脚本，这两个`NodeManager`分别启动`MapTask`，`MapTask`对数据分区排序。

- ​    （13）`MrAppMaster`等待所有`MapTask`运行完毕后，向RM申请容器，运行`ReduceTask`。

- ​	（14）`ReduceTask`向`MapTask`获取相应分区的数据。

- ​	（15）程序运行完毕后，`MR`会向`RM`申请注销自己。

  

### 3、作业提交

#### 1）作业提交过程之YARN

![](https://file.buildworld.cn/img/image-20200603142713125.png)

**提交作业详解**

##### （1）作业提交

- 第1步：Client调用`job.waitForCompletion`方法，向整个集群提交MapReduce作业。
- 第2步：Client向RM申请一个作业id。
- 第3步：RM给Client返回该job资源的提交路径和作业id。
- 第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。
- 第5步：Client提交完资源后，向RM申请运行`MrAppMaster`。

##### （2）作业初始化

- 第6步：当RM收到Client的请求后，将该job添加到容量调度器中。
- 第7步：某一个空闲的NM领取到该Job。
- 第8步：该NM创建`Container`，并产生`MRAppmaster`。
- 第9步：下载Client提交的资源到本地。

##### （3）任务分配

- 第10步：`MrAppMaster`向RM申请运行多个MapTask任务资源。
- 第11步：RM将运行`MapTask`任务分配给另外两个`NodeManager`，另两个`NodeManager`分别领取任务并创建容器。

##### （4）任务运行

- 第12步：MR向两个接收到任务的`NodeManager`发送程序启动脚本，这两个`NodeManager`分别启动`MapTask`，`MapTask`对数据分区排序。
- 第13步：`MrAppMaster`等待所有`MapTask`运行完毕后，向RM申请容器，运行`ReduceTask`。
- 第14步：`ReduceTask`向`MapTask`获取相应分区的数据。
- 第15步：程序运行完毕后，MR会向RM申请注销自己。

##### （5）进度和状态更新

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过`mapreduce.client.progressmonitor.pollinterval`设置)向应用管理器请求进度更新, 展示给用户。

##### （6）作业完成

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用`waitForCompletion()`来检查作业是否完成。时间间隔可以通过`mapreduce.client.completion.pollinterva`l来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。



#### 2）作业提交过程之MapReduce

![](https://file.buildworld.cn/img/20200603145736.png)

### 4、资源调度器

**Hadoop作业调度器主要有三种：`FIFO`、`Capacity Scheduler`和`Fair Scheduler`。**

#### 1）先进先出调度器（FIFO）

![](https://file.buildworld.cn/img/20200603152207.png)

#### 2）容量调度器（Capacity Scheduler）

![](https://file.buildworld.cn/img/20200603153023.png)

- 1、支持多个队列，每个队列可配置一定的资源量，每个队列采用FIFO调度策略。
- 2、为了防止同一个用户的作业独占队列中的资源，该调度器会对同一用户提交的作业所占资源量进行限定。
- 3、首先，计算每个队列中正在运行的任务数与其应该分得的计算资源之间的比值，选择一个该比值最小的队列——最闲的。
- 4、其次，按照作业优先级和提交时间顺序，同时考虑用户资源量限制和内存限制对队列内任务排序。
- 5、三个队列同时按照任务的先后顺序依次执行，比如，job11、job21和job31分别排在队列最前面，先运行，也是并行运行。

#### 3）公平调度器（Fair Scheduler）

![](https://file.buildworld.cn/img/20200603154212.png)

**支持多队列多用户，每个队列中的资源量可以配置，同一队列中的作业公平共享队列中所有资源。**

**在同一个队列中，job的资源缺额越大，越先获得资源优先执行。作业是按照缺额的高低来先后执行的，而且可以看到上图有多个作业同时运行。**