---
title: BigData-Apache HBase数据库
date: 2020-07-09 16:13:00
tags:
- HBase
categories:
- BigData
description: Apache HBase™ is the Hadoop database, a distributed, scalable, big data store.
top_img: http://p4.qhimg.com/bdr/__85/t01defc109c446643ce.jpg
cover: https://ae01.alicdn.com/kf/H439047697514463983c2a6699326c2c68.jpg
---
## HBase
## HBase

> Apache](https://www.apache.org/) HBase™ is the [Hadoop](https://hadoop.apache.org/) database, a distributed, scalable, big data store.



## 一、HBase原理

### 1、数据模型

#### 1 ） Name Space

> 命名空间，类似于关系型数据库的 DatabBase 概念，每个命名空间下有多个表。HBase有两个自带的命名空间，分别是 `hbase` 和 `default`，hbase 中存放的是 HBase 内置的表，default 表是用户默认使用的命名空间。

#### 2 ） Region

> 类似于关系型数据库的表概念。不同的是，HBase 定义表时只需要声明列族即可，不需要声明具体的列。这意味着，往 HBase 写入数据时，字段可以动态、按需指定。因此，和关系型数据库相比，HBase 能够轻松应对字段变更的场景。

#### 3 ） Row

> HBase 表中的每行数据都由一个 `RowKey` 和多个  `Column`（列）组成，数据是按照 `RowKey`的字典顺序存储的，并且查询数据时只能根据 `RowKey` 进行检索，所以 `RowKey` 的设计十分重要。

####  4 ） Column

> HBase 中的每个列都由 Column Family(列族)和 Column Qualifier（列限定符）进行限定，例如 `info：name，info：age`。建表时，只需指明列族，而列限定符无需预先定义。

### 2、HBase基础架构

![](https://file.buildworld.cn/img/20200705133837.png)

#### 1 ） Region Server

>`Region Server` 为 Region 的管理者，其实现类为 `HRegionServer`，主要作用如下:
>对于数据的操作：`get, put, delete`；
>对于 Region 的操作：`splitRegion、compactRegion`。



### 2 ） Master

> `Master` 是所有 `Region Server` 的管理者，其实现类为 `HMaster`，主要作用如下：
> 对于表的操作：`create, delete, alter`
> 对于`RegionServer`的操作：分配`regions`到每个`RegionServer`，监控每个`RegionServer`的状态，负载均衡和故障转移。

#### 3 ） Zookeeper

> `HBase` 通过 `Zookeeper` 来做 Master 的高可用、`RegionServer` 的监控、元数据的入口以及集群配置的维护等工作。

#### 4 ） HDFS

> `HDFS` 为 `HBase` 提供最终的底层数据存储服务，同时为 `HBase` 提供高可用的支持。



### 3、HBase架构详解

![](https://file.buildworld.cn/img/20200706133638.png)

#### 1）StoreFile

> 保存实际数据的物理文件，`StoreFile` 以 `HFile` 的形式存储在 HDFS 上。每个 `Store` 会有一个或多个 `StoreFile（HFile`），数据在每个 `StoreFile` 中都是有序的。

#### 2）MemStore

> 写缓存，由于 `HFile` 中的数据要求是有序的，所以数据是先存储在 `MemStore` 中，排好序后，等到达刷写时机才会刷写到 `HFile`，每次刷写都会形成一个新的 `HFile`。 

#### 3）WAL（Write-Ahead logfile）

> 由于数据要经 `MemStore` 排序后才能刷写到 `HFile`，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做 `Write-Ahead logfile` 的文件中，然后再写入 `MemStore` 中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。**（HLog）**

### 4、HBase写流程

![](https://file.buildworld.cn/img/20200706193753.png)

> - 1）`Client` 先访问 `zookeeper`，获取 `hbase:meta` 表位于哪个 `Region Server`。 
> - 2）访问对应的 `Region Server`，获取 `hbase:meta` 表，根据读请求的 `namespace:table/rowkey`，查询出目标数据位于哪个 `Region Server` 中的哪个 `Region` 中。并将该 `table` 的 `region` 信息以及 `meta` 表的位置信息缓存在客户端的 `meta cache`，方便下次访问。
> - 3）与目标 `Region Server` 进行通讯；
> - 4）将数据顺序写入（追加）到 WAL； 
> - 5）将数据写入对应的 `MemStore`，数据会在 `MemStore` 进行排序； 
> - 6）向客户端发送 `ack`； 
> - 7）等达到 `MemStore` 的刷写时机后，将数据刷写到 `HFile`。

### 5、读流程

![](https://file.buildworld.cn/img/20200706223000.png)

> - 1）`Client` 先访问 `zookeeper`，获取 `hbase:meta` 表位于哪个 `Region Server`。 
> - 2）访问对应的 `Region Server`，获取 `hbase:meta` 表，根据读请求的 `namespace:table/rowkey`，查询出目标数据位于哪个 `Region Server` 中的哪个 `Region` 中。并将该 `table` 的 `region` 信息以及 `meta` 表的位置信息缓存在客户端的 `meta cache`，方便下次访问。
> - 3）与目标 `Region Server` 进行通讯；
> - 4）分别在 `Block Cache`（读缓存），`MemStore` 和 `Store File（HFile）`中查询目标数据，并将查到的所有数据进行合并。此处所有数据是指同一条数据的不同版本`（time stamp）`或者不同的类型`（Put/Delete）`。
> - 5） 将从文件中查询到的数据块（Block，HFile 数据存储单元，默认大小为 64KB）缓存到`Block Cache`。 
> - 6）将合并后的最终结果返回给客户端。

**内存和磁盘同时读取，但是将两个数据进行对比，返回时间戳大的数据，所以说HBase读取比写入要慢得多**



### 6、**StoreFile Compaction**

> `Compaction` 分为两种，分别是 `Minor Compaction` 和 `Major Compaction`。`Minor Compaction`会将临近的若干个较小的 `HFile` 合并成一个较大的 `HFile`，但**不会清理过期和删除的数据**。`Major Compaction` 会将一个 `Store` 下的所有的 `HFile` 合并成一个大 `HFile`，并且**会清理掉过期和删除的数据**。 

![](https://ae01.alicdn.com/kf/Hfc24467fbef54cd39b2f87d5a53bd52cs.jpg)



## 二、HBase API使用（Java）

### 1、添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-server</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>1.3.1</version>
    </dependency>
</dependencies>
```

### 2、Java调用

```java
package cn.buildworld.hbase;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;
import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-07-09 19:03
 */
public class Test {

    private static Connection connection = null;
    private static Admin admin = null;

    static {
        //1、获取配置文件信息
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104");


        try {
            //2、创建连接对象
            connection = ConnectionFactory.createConnection(configuration);

            //3、创建Admin对象
            admin = connection.getAdmin();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void close() throws IOException {
        if (admin != null) {
            admin.close();
        }
        if (connection != null) {
            connection.close();
        }
    }

    //1、判断表是否存在
    public static boolean isTabExist(String tableName) throws IOException {
        //3、判断表是否存在
        boolean exists = admin.tableExists(TableName.valueOf(tableName));

        return exists;
    }

    // 2、创建表
    public static void createTable(String tableName, String... cfs) throws IOException {

        //判断参数是否为空
        if (cfs.length <= 0) {
            System.out.println("请设置");
            return;
        }

        //判断表是否存在
        if (isTabExist(tableName)) {
            System.out.println(tableName + "表已经存在");
            return;
        }

        //创建表描述器
        HTableDescriptor hTableDescriptor = new HTableDescriptor(TableName.valueOf(tableName));

        for (String cf : cfs) {
            //创建列族描述器
            HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(cf);

            //添加具体的列族信息
            hTableDescriptor.addFamily(hColumnDescriptor);
        }

        //创建表
        admin.createTable(hTableDescriptor);
        System.out.println(tableName + "表创建成功！");

    }

    //3、删除表
    public static void dropTable(String tableName) throws IOException {
        //1、判断表是否存在
        if (!isTabExist(tableName)) {
            System.out.println("要删除的表不存在！！！");
            return;
        }
        //2、使表下线
        admin.disableTable(TableName.valueOf(tableName));

        //3、删除表
        admin.deleteTable(TableName.valueOf(tableName));

        System.out.println(tableName + "表删除成功！！！");
    }

    //4、创建命名空间
    public static void createNameSpace(String ns) {
        //创建命名空间描述器
        NamespaceDescriptor build = NamespaceDescriptor.create(ns).build();

        try {
            //创建命名空间
            admin.createNamespace(build);
            System.out.println(ns + "命名空间已经创建完成！");
        } catch (NamespaceExistException ne) {
            System.out.println(ns + "命名空间已经存在！！！");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //5、往表里面插入数据
    public static void putData(String tableName, String rowKey, String cf, String cn, String value) throws IOException {

        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建put对象
        Put put = new Put(Bytes.toBytes(rowKey));

        //给put对象赋值
        put.addColumn(Bytes.toBytes(cf), Bytes.toBytes(cn), Bytes.toBytes(value));

        //插入数据
        table.put(put);

        //关闭连接
        table.close();
    }

    //6、获取数据
    public static void getData(String tableName, String rowKey, String cf, String cn) throws IOException {

        //1、获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //2、创建Get对象
        Get get = new Get(Bytes.toBytes(rowKey));

        //设置获取指定族的值
        //get.addFamily(Bytes.toBytes(cf));
        //获取指定族和指定列的值
        get.addColumn(Bytes.toBytes(cf), Bytes.toBytes(cn));

        //3、获取数据
        Result result = table.get(get);

        //4、解析result
        Cell[] cells = result.rawCells();
        for (Cell cell : cells) {
            System.out.println("CF:" + Bytes.toString(CellUtil.cloneFamily(cell))
                    + " --CN:" + Bytes.toString(CellUtil.cloneQualifier(cell))
                    + " --Value:" + Bytes.toString(CellUtil.cloneValue(cell)));
        }

        table.close();
    }

    //7、获取数据
    public static void scanTable(String tableName) throws IOException {

        //1、获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //2、获取扫描器
        Scan scan = new Scan(Bytes.toBytes("1001"));

        ResultScanner scanner = table.getScanner(scan);
        for (Result result : scanner) {
            for (Cell cell : result.rawCells()) {
                System.out.println("CF:" + Bytes.toString(CellUtil.cloneFamily(cell))
                        + " --CN:" + Bytes.toString(CellUtil.cloneQualifier(cell))
                        + " --Value:" + Bytes.toString(CellUtil.cloneValue(cell)));
            }
        }
        table.close();
    }

    //8、删除数据
    public static void deleteData(String tableName, String rowKey, String cf, String cn) throws IOException {

        Table table = connection.getTable(TableName.valueOf(tableName));

        //构建删除对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));

        //删除指定的列族
        //delete.addFamily(Bytes.toBytes(cf));
        delete.addColumns(Bytes.toBytes(cf), Bytes.toBytes(cn));


        //执行删除操作
        table.delete(delete);

        table.close();
    }

    public static void main(String[] args) throws IOException {

        //1、测试表是否存在
        // System.out.println(isTabExist("student"));

        //2、创建表测试
        //createTable("idea:java", "class", "desc");

        //3、删除表测试
        //dropTable("teacher");

        //4、测试命名空间创建
        //createNameSpace("idea");

        //5、测试插入如数据
        //putData("student","1003","info","name","michong");

        //6、获取表数据
        //getData("student", "1001", "info", "sex");

        //7、扫描表
        //scanTable("student");

        //8、测试删除
        deleteData("student", "1002", "info", "name");

        close();
    }
}
```
