---
title: BigData--Zookeeper介绍和使用
date: 2020-06-07 18:15:19
tags:
- Zookeeper
categories:
- BigData
description: Apache ZooKeeper is an effort to develop and maintain an open-source server which enables highly reliable distributed coordination.
top_img: https://ae01.alicdn.com/kf/H413d865662f94a6da42a5af4721e59e87.jpg
cover: https://file.buildworld.cn/img/u=3576311305,4246803808&fm=26&gp=0.jpg
---
## Zookeeper

### 1、Zookeeper工作机制

![](https://file.buildworld.cn/img/20200606173824.png)



### 2、Zookeeper特点

![](https://file.buildworld.cn/img/20200606174415.png)

- 1）Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。
- **2）集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。**
- 3）全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
- 4）更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。
- 5）数据更新原子性，一次数据更新要么成功，要么失败。
- 6）实时性，在一定时间范围内，Client能读到最新数据。

### 3、客户端命令

| 命令基本语法     | 功能描述                                         |
| ---------------- | ------------------------------------------------ |
| help             | 显示所有操作命令                                 |
| ls path [watch]  | 使用 ls 命令来查看当前znode中所包含的内容        |
| ls2 path [watch] | 查看当前节点数据并能看到更新次数等数据           |
| create           | 普通创建-s  含有序列-e  临时（重启或者超时消失） |
| get path [watch] | 获得节点的值                                     |
| set              | 设置节点的具体值                                 |
| stat             | 查看节点状态                                     |
| delete           | 删除节点                                         |
| rmr              | 递归删除节点                                     |



### 4、Java相关开发

#### 1）POM文件设置

```xml
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>RELEASE</version>
	</dependency>
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-core</artifactId>
		<version>2.8.2</version>
	</dependency>
		<!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
	<dependency>
		<groupId>org.apache.zookeeper</groupId>
		<artifactId>zookeeper</artifactId>
		<version>3.4.10</version>
	</dependency>
</dependencies>
```

#### 2）Java代码使用

```java
package cn.buildworld.zookeeper;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.util.List;

/**
 * @author MiChong
 * @date 2020-06-07 14:13
 */
public class ZkClient {
    private ZooKeeper zkClient;
    private static final String CONNECT_URL = "192.168.162.102:2181,192.168.162.103:2181,192.168.162.104:2181";
    private static final int SESSION_TIME_OUT = 2000;

    /**
    ** 创建客户端
    **/
    @Before
    public void before() throws IOException {
        zkClient = new ZooKeeper(CONNECT_URL, SESSION_TIME_OUT, (e) -> {
            System.out.println("命令执行完成回调！");
        });
    }

    /**
     * 获取目录
     * @throws KeeperException
     * @throws InterruptedException
     */
    @Test
    public void ls() throws KeeperException, InterruptedException {
        List<String> children = zkClient.getChildren("/", event -> {
            System.out.println("目录获取完成！");
        });
        for (String path : children) {
            System.out.println(path);
        }
        Thread.sleep(Long.MAX_VALUE);
    }

    /**
     * 创建目录
     * @throws KeeperException
     * @throws InterruptedException
     */
    @Test
    public void create() throws KeeperException, InterruptedException {
        String path = zkClient.create("/Idea",
                "234567".getBytes(),
                ZooDefs.Ids.CREATOR_ALL_ACL,
                CreateMode.EPHEMERAL);

        System.out.println("创建的路径："+path);
    }

    /**
     * 获取指定目录下面的数据
     * @throws KeeperException
     * @throws InterruptedException
     */
    @Test
    public void get() throws KeeperException, InterruptedException {
        byte[] data = zkClient.getData("/test", true, new Stat());
        String s = new String(data);
        System.out.println(s);
    }

    /**
     * 更新数据
     * @throws KeeperException
     * @throws InterruptedException
     */
    @Test
    public void set() throws KeeperException, InterruptedException {
        Stat stat = zkClient.setData("/test", "hello world".getBytes(), 0);

        byte[] data = zkClient.getData("/test", true, stat);
        String s = new String(data);
        System.out.println("更新之后的数据："+s);
    }

    /**
     * 获取节点状态
     * @throws KeeperException
     * @throws InterruptedException
     */
    @Test
    public void stat() throws KeeperException, InterruptedException {
        Stat exists = zkClient.exists("/test", false);
        if (exists == null){
            System.out.println("节点不存在！");
        }else {
            System.out.println(exists.getDataLength());
        }
    }

    /**
     * 删除
     * @throws KeeperException
     * @throws InterruptedException
     */
    @Test
    public void delete() throws KeeperException, InterruptedException {
        Stat exists = zkClient.exists("/test", false);
        if (exists != null){
            zkClient.delete("/test",exists.getVersion());
        }else {
            System.out.println("节点不存在！");
        }
    }
}
```



### 5、Zookeeper内部原理

#### 1）节点类型

![](https://file.buildworld.cn/img/20200607122957.png)



####  2）Stat结构体

- 1）czxid-创建节点的事务zxid
**每次修改ZooKeeper状态都会收到一个zxid形式的时间戳，也就是ZooKeeper事务ID。**
**事务ID是ZooKeeper中所有修改总的次序。每个修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1在zxid2之前发生。**
- 2）ctime - znode被创建的毫秒数(从1970年开始)
- 3）mzxid - znode最后更新的事务zxid
- 4）mtime - znode最后修改的毫秒数(从1970年开始)
- 5）pZxid-znode最后更新的子节点zxid
- 6）cversion - znode子节点变化号，znode子节点修改次数
- 7）dataversion - znode数据变化号
- 8）aclVersion - znode访问控制列表的变化号
- 9）ephemeralOwner- 如果是临时节点，这个是znode拥有者的session id。如果不是临时节点则是0。
- 10）dataLength- znode的数据长度
- 11）numChildren - znode子节点数量



#### 3）监听原理

![](https://file.buildworld.cn/img/20200607165333.png)

#### 4）选举机制

- （1）半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器。
- （2）Zookeeper虽然在配置文件中并没有指定Master和Slave。但是，Zookeeper工作时，是有一个节点为Leader，其他则为Follower，Leader是通过内部的选举机制临时产生的。
- （3）以一个简单的例子来说明整个选举的过程。



**选举流程**

![](https://file.buildworld.cn/img/20200607171940.png)

（1）服务器1启动，发起一次选举。服务器1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器1状态保持为LOOKING；

（2）服务器2启动，再发起一次选举。服务器1和2分别投自己一票并交换选票信息：此时服务器1发现服务器2的ID比自己目前投票推举的（服务器1）大，更改选票为推举服务器2。此时服务器1票数0票，服务器2票数2票，没有半数以上结果，选举无法完成，服务器1，2状态保持LOOKING

（3）服务器3启动，发起一次选举。此时服务器1和2都会更改选票为服务器3。此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING；

（4）服务器4启动，发起一次选举。此时服务器1，2，3已经不是LOOKING状态，不会更改选票信息。交换选票信息结果：服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOWING；

（5）服务器5启动，同4一样当小弟。



#### 5）写数据流程

![](https://file.buildworld.cn/img/20200607172236.png)