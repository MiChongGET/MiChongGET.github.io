---
title: BigData-消息队列框架Apache Kafka入门、原理解析
date: 2020-07-02 15:11:20
tags:
- Hadoop
- Kafka
- Apache
categories:
- BigData
description: Kafka是一个分布式的基于发布/订阅模式的消息队列，主要应用于大数据实时处理领域。
top_img: https://ae01.alicdn.com/kf/Hf2a6711250954d3a8677efe3baf1c1b25.jpg
cover: https://file.buildworld.cn/img/20200702193913.png
---

## Kafka--消息队列框架

### 1、**Kafka** 基础架构

![](https://ae01.alicdn.com/kf/Hbbbd7556065c46a79a68ebd9302ab08eR.jpg)

> 1）**Producer** ：消息生产者，就是向kafka broker发消息的客户端；
> 2）**Consumer** ：消息消费者，向kafka broker取消息的客户端；
> 3）**Consumer Group （CG）**：消费者组，由多个consumer组成。`消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个消费者消费；消费者组之间互不影响。`所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
> 4）**Broker ：**一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。
> 5）**Topic** ：可以理解为一个队列，`生产者和消费者面向的都是一个topic`；
> 6）**Partition**：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，**一个topic可以分为多个partition**，每个partition是一个有序的队列；
> 7）**Replica**：副本，为保证集群中的某个节点发生故障时，**该节点上的partition数据不丢失，且kafka仍然能够继续工作**，kafka提供了副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower。
> 8）**leader**：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader。
> 9）**follower**：每个分区多个副本中的“从”，实时从leader中同步数据，保持和leader数据的同步。leader发生故障时，某个follower会成为新的follower。

 

### 2、 **Kafka** 命令行操作

#### 0）启动kafka

```shell
[root@hadoop102 kafka]$ /opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties
```



#### 1）查看所有的topic

```shell
[root@hadoop102 bin]$ kafka-topics.sh --list --zookeeper hadoop102:2181
```

#### 2）创建topic

```shell
[root@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 3 --partitions 1 --topic first
```

> 选项说明：
>
> `--topic` 定义topic名
>
> `--replication-factor`  定义副本数
>
> `--partitions`  定义分区数

#### 3）删除

```shell
[root@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --delete --topic first
```

**需要server.properties中设置delete.topic.enable=true否则只是标记删除。**

#### 4）发送消息

```shell
[root@hadoop102 bin]$ kafka-console-producer.sh --broker-list hadoop102:9092 --topic first
```

#### 5）消费消息

```shell
[root@hadoop103 kafka]$ kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --from-beginning --topic first
```

**\--from\-beginning：会把主题中以往所有的数据都读取出来。**

#### 6）查看某个topic详情

```shell
[root@hadoop104 kafka]$ kafka-topics.sh --describe --topic first --zookeeper hadoop102:2181
```

#### 7）修改分区

```shell
[root@hadoop104 kafka]$ kafka-topics.sh --alter --topic first --zookeeper hadoop102:2181 --partitions 6
```

### 3、**Kafka** 工作流程

![](https://file.buildworld.cn/img/20200702172047.png)

> `Kafka`中消息是以`topic`进行分类的，生产者生产消息，消费者消费消息，都是面向`topic`的。
>
> `topic`是逻辑上的概念，而`partition`是物理上的概念，每个`partition`对应于一个`log`文件，该`log`文件中存储的就是`producer`生产的数据。`Producer`生产的数据会被不断追加到该`log`文件末端，且每条数据都有自己的`offset`。消费者组中的每个消费者，都会实时记录自己消费到了哪个`offset`，以便出错恢复时，从上次的位置继续消费。

### 4、Kafka文件存储机制

![](https://file.buildworld.cn/img/20200702193119.png)

> 由于生产者生产的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了**分片和索引**机制，将每个`partition`分为多个`segment`。每个`segment`对应两个文件——“`.index`”文件和“`.log`”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：`topic名称+分区序号`。例如，first这个topic有三个分区，则其对应的文件夹为first-0,first-1,first-2。

### 5、Kafka 生产者

#### 1）分区原因：

- （1） **方便在集群中扩展**，每个 Partition 可以通过调整以适应它所在的机器，而一个 topic又可以有多个 Partition 组成，因此整个集群就可以适应任意大小的数据了；
- （2） **可以提高并发**，因为可以以 Partition 为单位读写了。



#### 2）数据可靠保证

![](https://file.buildworld.cn/img/20200703102441.png)

##### （1）ack

> Topic的每个partition收到producer发送的数据后，  都需要向producer发送`ack`（`acknowledgement` 确认收到），如果producer 收到 ack，就会进行下一轮的发送，否则重新发送数据。

##### （2）in-sync replica set (ISR)

>Leader 维护了一个动态的 `in-sync replica set (ISR)`，意为和 leader 保持同步的 follower 集合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。如果follower长 时 间 未 向 leader 同 步 数 据 ， 则 该 follower 将 被 踢 出 ISR ， 该 时 间 阈 值 由`replica.lag.time.max.ms` 参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 leader。

**ack关系数据丢不丢失的问题，ISR关系数据一致性和存储问题**

### 6、Kafka消费者

#### 1）消费方式

> 消费者采用`pull（拉）`模式从broker中读取数据。
>
> `push（推）`模式难以适应消费速率不同的消费者！
>
> 如果kafka没有数据，消费者可能陷入到循环中，一直返回空数据。

#### 2）分配策略

- RoundRobin
- Range（默认的）

#### 3）offset

> 下图是记录在zookeeper中的数据结构

![](https://file.buildworld.cn/img/20200703152929.png)

### 7、kafka事务

#### 1）Producer  事务

> 为了实现跨分区跨会话的事务，需要引入一个全局唯一的 **Transaction ID（客户端给予的）**，并将 `Producer`获得的`PID`和`Transaction ID`绑定。这样当Producer重启后就可以通过正在进行的TransactionID 获得原来的 PID。
>
> 为了管理 Transaction，Kafka 引入了一个新的组件 `Transaction Coordinator`。Producer 就是通过和 Transaction Coordinator 交互获得 Transaction ID 对应的任务状态。TransactionCoordinator 还负责将事务所有写入 Kafka 的一个内部 Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。

#### 2）Consumer

>由于 `Consumer` 可以通过 `offset` 访问任意信息，而且不同的 `Segment File` 生命周期不同，同一事务的消息可能会出现重启后被删除的情况。