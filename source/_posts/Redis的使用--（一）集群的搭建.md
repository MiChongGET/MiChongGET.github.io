---
layout: '[layout]'
title: Redis的使用--（一）集群的搭建
date: 2018-01-31 16:12:10
tags:
- Redis
categories:
- 集群
---
##### 主题词：负载均衡高可用、redis集群

* 需求：负载均衡高可用的概念

1. 什么是负载均衡高可用

> Nginx一般用作负载均衡服务器，可见处于网络中非常重要的位置，一旦Nginx服务器宕机无法提供服务，那么将影响严重。所以需要负载均衡高可用。

> 高可用——主从备份


2. keepalived+nginx实现主从备份

> Keepalived的作用是检测服务器的状态，如果有一台web服务器宕机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。

3. keepalived工作原理

> keepalived是以VRRP协议为实现基础的，VRRP全称Virtual Router Redundancy Protocol，即虚拟路由冗余协议。

> 虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master和多个backup，master上面有一个对外提供服务的vip（该路由器所在局域网内其他机器的默认路由为该vip），master会发组播，当backup收不到vrrp包时就认为master宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master。这样的话就可以保证路由器的高可用了。

> keepalived主要有三个模块，分别是core、check和vrrp。core模块为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析。check负责健康检查，包括常见的各种检查方式。vrrp模块是来实现VRRP协议的。

4. 搭建过程可参考《keepalived权威指南中文.pdf》


* 需求：完成redis集群的搭建

1. 了解集群和主从的区别

2. redis集群基本概念

> redis集群的详细贴子：http://blog.csdn.net/sanwenyublog/article/details/52942236

> redis集群中至少应该有三个节点。要保证集群的高可用，每个节点需要有一个备份机。因此redis集群至少需要六台服务器

> 这里搭建的是伪分布模式，可以使用一台服务器运行6个redis实例，修改redis的端口号为7001-7006


> 相关算法：http://blog.csdn.net/u014490157/article/details/52244378

3. redis集群的搭建

* 安装ruby环境

```
yum install ruby
yum install rubygems
```

* 安装ruby脚本运行使用的包

```
# 离线安装
gem install redis-3.0.7.gem
```

```
# 在线安装
gem install redis -v 3.0.7
```

* 创建6台服务器，将6台的端口号修改7001——7006


```
1、将端口修改为7001-7006
2、将集群前面#注释去除 cluster-enabled yes
3、如果是云服务器，例如端口是7001，则将7001和17001加入安全组

```

* 清除每一个节点的缓存数据

```shell
bin/redis-cli -p 7002 -c
127.0.0.1:7002> flushall
OK
127.0.0.1:7002> cluster reset
OK
127.0.0.1:7002> quit
[root@VM_241_143_centos redis02]# cd ..
[root@VM_241_143_centos redis-cluster]# cd redis03
[root@VM_241_143_centos redis03]# bin/redis-cli -p 7003 -c
127.0.0.1:7003> flushall
OK
127.0.0.1:7003> cluster reset
OK
127.0.0.1:7003> quit
```

* 自定义shell脚本启动6台服务器

```shell

cd /usr/local/redis-cluster/redis01
bin/redis-server redis.conf
cd ../redis02
bin/redis-server redis.conf
cd ../redis03
bin/redis-server redis.conf
cd ../redis04
bin/redis-server redis.conf
cd ../redis05
bin/redis-server redis.conf
cd ../redis06
bin/redis-server redis.conf
```

* 自定义shell脚本关闭6台服务器交给大家来做

* 运行如下代码搭建集群环境


```
./redis-trib.rb create --replicas 1 10.31.152.30:7001 10.31.152.30:7002 10.31.152.30:7003 10.31.152.30:7004 10.31.152.30:7005 10.31.152.30:7006
```


```
./redis-trib.rb create --replicas 1 10.31.166.22:9001 10.31.166.22:9002 10.31.166.22:9003 10.31.166.22:9004 10.31.166.22:9005 10.31.166.22:9006
```
```
./redis-trib.rb create --replicas 1 119.29.181.95:7001 119.29.181.95:7002 119.29.181.95:7003 119.29.181.95:7004 119.29.181.95:7005 119.29.181.95:7006
```
* 集群创建成功的两张截图

![image](https://i.loli.net/2017/09/14/59ba40edb0422.jpg)
![image](https://i.loli.net/2017/09/14/59ba40edc85f6.jpg)



* 客户端如何连接集群中的机器


```
# -p 端口号
# -c 开启reidis cluster模式,连接redis cluster节点时候使用
bin/redis-cli -p 7004 -c
```

* 往集群节点存入数据进行测试，查看数据到底存入到哪个节点

> redis集群中内置了16384个哈希槽，当需要往集群中存放键值对的时候，redis先对key使用CRC16算法算出一个结果，然后拿这个结果对16384求余，这样每个key都会对应一个编号为0-16383之间的哈希槽，redis会根据节点数量大致均等的将哈希槽映射到不同的节点上