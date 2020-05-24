---
title: BigData--Hadoop技术
date: 2020-05-22 09:37:03
tags:
- Hadoop
- HDFS
- YARN
- MapReduce
categories:
- BigData
description: Hadoop是一个由Apache基金会所开发的分布式系统基础架构。用户可以在不了解分布式底层细节的情况下，开发分布式程序。充分利用集群的威力进行高速运算和存储。
top_img: https://ae01.alicdn.com/kf/Hec817c4f0dc645feb89c62cf831f6ecem.jpg
cover: https://file.buildworld.cn/img/b7e15dd6cb0a651fd54bd82bf625939d_Cg-4WVPSElWIGdy1AABtPWOy4kQAAP6HAKPjYgAAG1V585.jpg
---
## 一、Hadoop组成
![image](http://myfile.buildworld.cn/Hadoop组成.png)
### 1、HDFS架构
- 1）NameNode（nn）：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等
- 2）DataNode(dn)：在本地文件系统存储文件块数据，以及块数据的校验和。
- 3）Secondary NameNode(2nn)：用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照

### 2、YARN架构
![image](http://myfile.buildworld.cn/YARN架构.png)

### 3、MapReduce架构
- 1）Map阶段并行处理输入数据
- 2）Reduce阶段对Map结果进行汇总
![image](http://myfile.buildworld.cn/mapreduce.png)

### 4、大数据技术生态体系
![](https://file.buildworld.cn/img/20200522094812.png)


## 二、HDFS(Hadoop Distribution File System)
### 1、优点
![image](http://myfile.buildworld.cn/hdfs优点.png)
### 2、缺点
![image](http://myfile.buildworld.cn/hdfs缺点.png)

### 3、组织架构
![image](http://myfile.buildworld.cn/hdfs组织架构.png)
![image](http://myfile.buildworld.cn/hdfs组织架构2.png)


## 三、HDFS的数据流
### 1、HDFS写数据流程
![](https://file.buildworld.cn/img/20200521213715.png)

- 1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。
- 2）NameNode返回是否可以上传。
- 3）客户端请求第一个 Block上传到哪几个DataNode服务器上。
- 4）NameNode返回3个DataNode节点，分别为dn1、dn2、dn3。
- 5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。
- 6）dn1、dn2、dn3逐级应答客户端。
- 7）客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。
- 8）当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。

### 2、HDFS读数据流程
![](https://file.buildworld.cn/img/20200521213817.png)

### 3、网络拓扑-节点距离计算
> 节点距离：两个节点到达最近的共同祖先的距离总和。

### 常用命令实操
（0）启动Hadoop集群（方便后续的测试）

```
[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
[atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
```

（1）-help：输出这个命令参数

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -help rm
```

（2）-ls: 显示目录信息

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -ls /
```

（3）-mkdir：在HDFS上创建目录

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -mkdir -p /sanguo/shuguo
```

（4）-moveFromLocal：从本地剪切粘贴到HDFS

```
[atguigu@hadoop102 hadoop-2.7.2]$ touch kongming.txt
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs  -moveFromLocal  ./kongming.txt  /sanguo/shuguo
```

（5）-appendToFile：追加一个文件到已经存在的文件末尾

```
[atguigu@hadoop102 hadoop-2.7.2]$ touch liubei.txt
[atguigu@hadoop102 hadoop-2.7.2]$ vi liubei.txt
```

输入

```
san gu mao lu
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -appendToFile liubei.txt /sanguo/shuguo/kongming.txt
```

（6）-cat：显示文件内容

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -cat /sanguo/shuguo/kongming.txt
```

（7）-chgrp 、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs  -chmod  666  /sanguo/shuguo/kongming.txt
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs  -chown  atguigu:atguigu   /sanguo/shuguo/kongming.txt
```

（8）-copyFromLocal：从本地文件系统中拷贝文件到HDFS路径去

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -copyFromLocal README.txt /
```

（9）-copyToLocal：从HDFS拷贝到本地

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -copyToLocal /sanguo/shuguo/kongming.txt ./
```

（10）-cp ：从HDFS的一个路径拷贝到HDFS的另一个路径

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -cp /sanguo/shuguo/kongming.txt /zhuge.txt
```

（11）-mv：在HDFS目录中移动文件

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -mv /zhuge.txt /sanguo/shuguo/
```

（12）-get：等同于copyToLocal，就是从HDFS下载文件到本地

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -get /sanguo/shuguo/kongming.txt ./
```

（13）-getmerge：合并下载多个文件，比如HDFS的
目录 /user/atguigu/test下有多个文件:log.1, log.2,log.3,...

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -getmerge /user/atguigu/test/* ./zaiyiqi.txt
```

（14）-put：等同于copyFromLocal

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -put ./zaiyiqi.txt /user/atguigu/test/
```

（15）-tail：显示一个文件的末尾

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -tail /sanguo/shuguo/kongming.txt
```

（16）-rm：删除文件或文件夹

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -rm /user/atguigu/test/jinlian2.txt
```

（17）-rmdir：删除空目录

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -mkdir /test
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -rmdir /test
```

（18）-du统计文件夹的大小信息

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -du -s -h /user/atguigu/test
2.7 K  /user/atguigu/test

[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -du  -h /user/atguigu/test
1.3 K  /user/atguigu/test/README.txt
15     /user/atguigu/test/jinlian.txt
1.4 K  /user/atguigu/test/zaiyiqi.txt
```

（19）-setrep：设置HDFS中文件的副本数量

```
[atguigu@hadoop102 hadoop-2.7.2]$ hadoop fs -setrep 10 /sanguo/shuguo/kongming.txt
```


**假设有数据中心d1机架r1中的节点n1。该节点可以表示为/d1/r1/n1。**

![](https://file.buildworld.cn/img/20200521221124.png)

## 四、NN && 2NN

### 1、NameNode工作机制
**NameNode中的元数据是存储在哪里的？**
```
    首先，我们做个假设，如果存储在NameNode节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。
因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的FsImage。

    这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，
一旦NameNode节点断电，就会产生数据丢失。因此，引入Edits文件(只进行追加操作，效率很高)。每当元数据有更新或者添加元数据时，
修改内存中的元数据并追加到Edits中。这样，一旦NameNode节点断电，可以通过FsImage和Edits的合并，合成元数据。

    但是，如果长时间添加数据到Edits中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。
因此，需要定期进行FsImage和Edits的合并，如果这个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode，
专门用于FsImage和Edits的合并。
```
**Fsimage：NameNode内存中元数据序列化后形成的文件。**
**Edits：记录客户端更新元数据信息的每一步操作（可通过Edits运算出元数据）。**

##### 下图为NN和2NN工作机制
![NN和2NN工作机制](http://myfile.buildworld.cn/NameNode工作机制.png)  

## 五、DataNode
### 1、DataNode工作机制

![](https://file.buildworld.cn/img/20200522143452.png)

### 2、数据完整性
- 1）当DataNode读取Block的时候，它会计算CheckSum。
- 2）如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。
- 3）Client读取其他DataNode上的Block。
- 4）DataNode在其文件创建后周期验证CheckSum。

![](https://file.buildworld.cn/img/20200522152850.png)