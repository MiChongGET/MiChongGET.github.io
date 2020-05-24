---
title: BigData--Hadoop2.x新特性之HA
date: 2020-05-23 22:19:53
tags:
- Hadoop
- HDFS
- YARN
- HA
categories:
- BigData
description: Hadoop2.X的两大新特性：YARN和HA
top_img: https://ae01.alicdn.com/kf/H1f97727a7e774ebdbeca1ac0d703e6dak.jpg
cover: https://ae01.alicdn.com/kf/H326912a669954a64891d011f0bee71368.jpg
---
## HDFS HA高可用

> Hadoop2.X的两大新特性：YARN和HA

### 1、概述

**HA即High Available,高可用的意思**

- NameNode主要在以下两个方面影响HDFS集群

  ```
  NameNode机器发生意外，如宕机，集群将无法使用，直到管理员重启
  
  NameNode机器需要升级，包括软件、硬件升级，此时集群也将无法使用
  
  HDFS HA功能通过配置Active/Standby两个NameNodes实现在集群中对NameNode的热备来解决上述问题。如果出现故障，如机器崩溃或机器需要升级维护，这时可通过此种方式将NameNode很快的切换到另外一台机器。
  ```

  

### 2、HDFS-HA工作机制

> 通过双NameNode消除单点故障

### 3、HDFS-HA工作要点

- 元数据管理方式需要改变

```
内存中各自保存一份元数据；

Edits日志只有Active状态的NameNode节点可以做写操作；

两个NameNode都可以读取Edits；

共享的Edits放在一个共享存储中管理（qjournal和NFS两个主流实现）；
```

- 需要一个状态管理功能模块

```
实现了一个zkfailover，常驻在每一个namenode所在的节点，每一个zkfailover负责监控自己所在NameNode节点，利用zk进行状态标识，当需要进行状态切换时，由zkfailover来负责切换，切换时需要防止brain split现象的发生。
```

- 必须保证两个NameNode之间能够ssh无密码登录

- 隔离（Fence），即同一时刻仅仅有一个NameNode对外提供服务



### 4、HDFS-HA自动故障转移工作机制

#### 1）HA的自动故障转移依赖于ZooKeeper的以下功能：
**故障检测**：每个NameNode在ZooKeeper中维护了一个持久会话，如果机器崩溃，ZooKeeper中的会话将终止，ZooKeeper通知另一个NameNode需要触发故障转移。

**现役NameNode选择：**ooKeeper提供了一个简单的机制用于唯一的选择一个节点为active状态。如果目前现役NameNode崩溃，另一个节点可能从ZooKeeper获得特殊的排外锁以表明它应该成为现役NameNode。



#### 2）ZKFC

> ZKFC是自动故障转移中的另一个新组件，是ZooKeeper的客户端，也监视和管理NameNode的状态。每个运行NameNode的主机也运行了一个ZKFC进程，ZKFC负责：

**健康监测：**ZKFC使用一个健康检查命令定期地ping与之在相同主机的NameNode，只要该NameNode及时地回复健康状态，ZKFC认为该节点是健康的。如果该节点崩溃，冻结或进入不健康状态，健康监测器标识该节点为非健康的。

**ZooKeeper会话管理：**当本地NameNode是健康的，ZKFC保持一个在ZooKeeper中打开的会话。如果本地NameNode处于active状态，ZKFC也保持一个特殊的znode锁，该锁使用了ZooKeeper对短暂节点的支持，如果会话终止，锁节点将自动删除。

**基于ZooKeeper的选择：**如果本地NameNode是健康的，且ZKFC发现没有其它的节点当前持有znode锁，它将为自己获取该锁。如果成功，则它已经赢得了选择，并负责运行故障转移进程以使它的本地NameNode为Active。故障转移进程与前面描述的手动故障转移相似，首先如果必要保护之前的现役NameNode，然后本地NameNode转换为Active状态。



#### HDFS-HA故障转移机制流程图

![](https://file.buildworld.cn/img/20200523093038.png)

### 5、YARN-HA工作机制

![](https://file.buildworld.cn/img/20200523162851.png)

### 6、HDFS Federation架构设计

#### 1、NameNode架构局限性

- （1）Namespace（命名空间）的限制
- （2）隔离问题
- （3）性能的瓶颈

#### 2、HDFS Federation架构设计

![](https://file.buildworld.cn/img/20200523221153.png)

**不同应用可以使用不同NameNode进行数据管理,Hadoop生态系统中，不同的框架使用不同的NameNode进行管理NameSpace。（隔离性）**

