---
title: BigData--Apache Flume框架
date: 2020-07-01 17:22:48
tags:
- Hadoop
- Flume
- Apache
categories:
- BigData
description: Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。Flume基于流式架构，灵活简单。
top_img: http://p5.qhimg.com/bdr/__85/t011fe8fb82651dfd2c.jpg
cover: https://ae01.alicdn.com/kf/H3cae0a1814934caa9a43090c0c041653e.jpg
---

## Flume进阶

### 1、Flume基本框架

![](https://file.buildworld.cn/img/20200701104704.png)

**1. Event**

`Event` 是 Flume NG 数据传输的基本单元。类似于 JMS 和消息系统中的消息。一个 Event 由标题和正文组成：前者是键/值映射，后者是任意字节数组。

**2. Source**

数据收集组件，从外部数据源收集数据，并存储到 Channel 中。

**Source类型**

- （1）监控后台日志：exec
- （2）监控后台产生日志的端口：netcat

**3. Channel**

`Channel` 是源和接收器之间的管道，用于临时存储数据。可以是内存或持久化的文件系统：

- `Memory Channel` : 使用内存，优点是速度快，但数据可能会丢失 (如突然宕机)；
- `File Channel` : 使用持久化的文件系统，优点是能保证数据不丢失，但是速度慢。

**4. Sink**

`Sink` 的主要功能从 `Channel` 中读取 `Event` ，并将其存入外部存储系统或将其转发到下一个`Source` ，成功后再从 `Channel` 中移除 `Event` 。

**5. Agent**

是一个独立的 (JVM) 进程，包含 `Source` 、 `Channel` 、 `Sink` 等组件。



### 2、Flume事务  

![](https://file.buildworld.cn/img/20200701104347.png)

> **Flume 的事务机制（类似数据库的事务机制）**：Flume 使用两个独立的事务分别负责从`Soucrce` 到 `Channel`，以及从 `Channel` 到 `Sink` 的事件传递。比如 `spooling directory source` 为文件的每一行创建一个事件，一旦事务中所有的事件全部传递到 `Channel` 且提交成功，那么 `Soucrce` 就将该文件标记为完成。同理，事务以类似的方式处理从 `Channel` 到 `Sink` 的传递过程，如果因为某种原因使得事件无法记录，那么事务将会回滚。且所有的事件都会保持到 `Channel` 中，等待重新传递。

### 3、**Flume Agent** 内部原理

![](https://file.buildworld.cn/img/20200701113850.png)

**重要组件：** 

**1）ChannelSelector** 

`ChannelSelector` 的作用就是选出 `Event` 将要被发往哪个 `Channel`。其共有两种类型，分别是 `Replicating`（复制）和 `Multiplexing`（多路复用）。

`ReplicatingSelector` 会将同一个 `Event` 发往所有的 `Channel`，`Multiplexing` 会根据相应的原则，将不同的 `Event` 发往不同的 `Channel`。 

**2）SinkProcessor** 

`SinkProcessor` 共 有 三 种 类 型 ， 分 别 是 `DefaultSinkProcessor` 、`LoadBalancingSinkProcessor` 和 `FailoverSinkProcessor`

`DefaultSinkProcessor` 对 应 的 是 单 个 的 `Sink` ， `LoadBalancingSinkProcessor` 和 `FailoverSinkProcessor` 对应的是 `Sink Group`，`LoadBalancingSinkProcessor` 可以实现负载均衡的功能，`FailoverSinkProcessor` 可以实现故障转移的功能。

### 4、**Flume** 拓扑结构

#### （1）、简单串联

![](https://file.buildworld.cn/img/20200701145041.png)

> 这种模式是将多个 `flume` 顺序连接起来了，从最初的 `source` 开始到最终 `sink` 传送的目的存储系统。此模式不建议桥接过多的 `flume` 数量，`flume` 数量过多不仅会影响传输速率，而且一旦传输过程中某个节点 flume 宕机，会影响整个传输系统。

#### （2）、复制和多路复用

![](https://file.buildworld.cn/img/20200701145300.png)

>`Flume` 支持将事件流向一个或者多个目的地。这种模式可以将相同数据复制到多个`channel` 中，或者将不同数据分发到不同的 `channel` 中，`sink` 可以选择传送到不同的目的地。

#### （3）、负载均衡和故障转移

![](https://file.buildworld.cn/img/20200701145526.png)

> `Flume`支持使用将多个`sink`逻辑上分到一个`sink`组，`sink`组配合不同的`SinkProcessor`可以实现负载均衡和错误恢复的功能。

#### (4)、聚合

![](https://file.buildworld.cn/img/20200701145800.png)

> 这种模式是我们最常见的，也非常实用，日常 web 应用通常分布在上百个服务器，大者甚至上千个、上万个服务器。产生的日志，处理起来也非常麻烦。用 `flume` 的这种组合方式能很好的解决这一问题，每台服务器部署一个 `flume` 采集日志，传送到一个集中收集日志的`flume`，再由此 `flume` 上传到 `hdfs`、`hive`、`hbase` 等，进行日志分析。 



### 5、 **Flume** 参数调优

####  （1）. Source

> 增加 `Source` 个（使用 `Tair Dir Source` 时可增加 `FileGroups` 个数）可以增大 Source 的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个文件目录，同时配置好多个 Source 以保证 Source 有足够的能力获取到新产生的数据。

> `batchSize` 参数决定 Source 一次批量运输到 Channel 的 event 条数，适当调大这个参数可以提高 Source 搬运 Event 到 Channel 时的性能。

#### （2）. Channel

> `type` 选择 memory 时 Channel 的性能最好，但是如果 Flume 进程意外挂掉可能会丢失数据。type 选择 file 时 Channel 的容错性更好，但是性能上会比 memory channel 差。

> 使用 `file Channel` 时 `dataDirs` 配置多个不同盘下的目录可以提高性能。

> `Capacity` 参数决定 `Channel` 可容纳最大的 `event` 条数。`transactionCapacity` 参数决定每次 Source 往 channel 里面写的最大 event 条数和每次 Sink 从 channel 里面读的最大 event条数。**transactionCapacity 需要大于 Source 和 Sink 的 batchSize 参数。**

#### （3）. Sink

> 增加 `Sink` 的个数可以增加 Sink 消费 event 的能力。Sink 也不是越多越好够用就行，过多的 Sink 会占用系统资源，造成系统资源不必要的浪费。

> `batchSize` 参数决定 Sink 一次批量从 Channel 读取的 event 条数，适当调大这个参数可以提高 Sink 从 Channel 搬出 event 的性能。



### 6、**Flume** **的** Channel Selectors

![](https://file.buildworld.cn/img/20200701190722.png)