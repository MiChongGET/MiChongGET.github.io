---
title: BigData--Hive数据仓库工具
date: 2020-06-08 12:38:51
tags:
- Hive
- SQL
categories:
- BigData
description: Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。
top_img: https://yanxuan.nosdn.127.net/14c292de64bccbb469cefc2b94f9ab42.png
cover: https://file.buildworld.cn/img/20200608124301.png
---

## Hive

### 一、Hive入门

#### 1、Hive功能

![](https://file.buildworld.cn/img/20200608124512.png)

- 1）Hive处理的数据存储在HDFS
- 2）Hive分析数据底层的实现是MapReduce
- 3）执行程序运行在Yarn上



#### 2、Hive的优缺点

##### 优点

- （1) 操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）。
- （2) 避免了去写MapReduce，减少开发人员的学习成本。
- （3) Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。
- （4) Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。
- （5) Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。

##### 缺点

- （1）迭代式算法无法表达
- （2）数据挖掘方面不擅长，由于MapReduce数据处理流程的限制，效率更高的算法却无法实现。
- （3）Hive自动生成的MapReduce作业，通常情况下不够智能化
- （4）Hive调优比较困难，粒度较粗

#### 3、Hive架构

![](https://file.buildworld.cn/img/20200613100142.png)

- （1）**解析器（SQL Parser）**：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
- （2）**编译器（Physical Plan）：**将AST编译生成逻辑执行计划。
- （3）**优化器（Query Optimizer）：**对逻辑执行计划进行优化。
- （4）**执行器（Execution）：**把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。



#### 4、Hive的运行机制

**Hive通过给用户提供的一系列交互接口，接收到用户的`指令(SQL)`，使用自己的`Driver`，结合`元数据(MetaStore)`，将这些指令翻译成MapReduce，提交到Hadoop中执行，最后，将执行返回的结果输出到用户交互接口。**

![](https://file.buildworld.cn/img/20200613102931.png)



### 二、Hive数据类型

#### 1、基本数据类型

| Hive数据类型 | Java数据类型 | 长度                                                 | 例子                                 |
| :----------: | :----------: | ---------------------------------------------------- | ------------------------------------ |
|   TINYINT    |     byte     | 1byte有符号整数                                      | 20                                   |
|   SMALINT    |    short     | 2byte有符号整数                                      | 20                                   |
|   **INT**    |     int      | 4byte有符号整数                                      | 20                                   |
|  **BIGINT**  |     long     | 8byte有符号整数                                      | 20                                   |
|   BOOLEAN    |   boolean    | 布尔类型，true或者false                              | TRUE  FALSE                          |
|    FLOAT     |    float     | 单精度浮点数                                         | 3.14159                              |
|  **DOUBLE**  |    double    | 双精度浮点数                                         | 3.14159                              |
|  **STRING**  |    string    | 字符系列。可以指定字符集。可以使用单引号或者双引号。 | ‘now is the time’ “for all good men” |
|  TIMESTAMP   |              | 时间类型                                             |                                      |
|    BINARY    |              | 字节数组                                             |                                      |

#### 2、集合数据类型

| 数据类型 | 描述                                                         | 语法示例                                       |
| :------: | ------------------------------------------------------------ | :--------------------------------------------- |
|  STRUCT  | 和c语言中的struct类似，都可以通过“点”符号访问元素内容。例如，如果某个列的数据类型是STRUCT{first STRING, last STRING},那么第1个元素可以通过字段.first来引用。 | struct()例如struct<street:string, city:string> |
|   MAP    | MAP是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 | map()例如map<string, int>                      |
|  ARRAY   | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’, ‘Doe’]，那么第2个元素可以通过数组名[1]进行引用。 | Array()例如array<string>                       |

#### 3、类型转化

> Hive的原子数据类型是可以进行隐式转换的，类似于Java的类型转换，例如某表达式使用`INT`类型，`TINYINT`会自动转换为`INT`类型，但是Hive不会进行反向转化，例如，某表达式使用TINYINT类型，INT不会自动转换为TINYINT类型，它会返回错误，除非使用`CAST`操作。

- （1）任何整数类型都可以隐式地转换为一个范围更广的类型，如TINYINT可以转换成INT，INT可以转换成BIGINT。
- （2）所有整数类型、FLOAT和STRING类型都可以隐式地转换成DOUBLE。
- （3）TINYINT、SMALLINT、INT都可以转换为FLOAT。
- （4）BOOLEAN类型不可以转换为任何其它的类型。

### 三、DDL数据定义

​	

#### 1、建表语法

```shell
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 

[(col_name data_type [COMMENT col_comment], ...)] 

[COMMENT table_comment] 

[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 

[CLUSTERED BY (col_name, col_name, ...) 

[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 

[ROW FORMAT row_format] 

[STORED AS file_format] 

[LOCATION hdfs_path]

[TBLPROPERTIES (property_name=property_value, ...)]

[AS select_statement]
```

- （1）CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。
- （2）EXTERNAL关键字可以让用户创建一个外部表，在建表的同时可以指定一个指向实际数据的路径（LOCATION），在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。
- （3）COMMENT：为表和列添加注释。
- （4）PARTITIONED BY创建分区表
- （5）CLUSTERED BY创建分桶表
- （6）SORTED BY不常用，对桶中的一个或多个列另外排序
- （7）ROW FORMAT 

DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char]

    [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char] 

  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]

用户在建表的时候可以自定义SerDe或者使用自带的SerDe。如果没有指定ROW FORMAT 或者ROW FORMAT DELIMITED，将会使用自带的SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的SerDe，Hive通过SerDe确定表的具体的列的数据。

SerDe是Serialize/Deserilize的简称， hive使用Serde进行行对象的序列与反序列化。

- （8）STORED AS指定存储文件类型

常用的存储文件类型：SEQUENCEFILE（二进制序列文件）、TEXTFILE（文本）、RCFILE（列式存储格式文件）

如果文件数据是纯文本，可以使用STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。

- （9）LOCATION ：指定表在HDFS上的存储位置。
- （10）AS：后跟查询语句，根据查询结果创建表。
- （11）LIKE允许用户复制现有的表结构，但是不复制数据。

#### 2、管理表与外部表的互相转换

- （1）查询表的类型

```shell
hive (default)> desc formatted student2;

Table Type:       MANAGED_TABLE
```

- （2）修改内部表student2为外部表

```shell
alter table student2 set tblproperties('EXTERNAL'='TRUE');
```

- （3）查询表的类型

```shell
hive (default)> desc formatted student2;

Table Type:       EXTERNAL_TABLE
```

- （4）修改外部表student2为内部表

```shell
alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

- （5）查询表的类型

```shell
hive (default)> desc formatted student2;

Table Type:       MANAGED_TABLE
```

**注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写！**

### 四、DML数据操作

#### 1、数据导入

```shell
hive> load data [local] inpath '/opt/module/datas/student.txt' [overwrite] into table student [partition (partcol1=val1,…)];
```

- （1）**load data:**表示加载数据
- （2）**local:**表示从本地加载数据到hive表；否则从HDFS加载数据到hive表
- （3）**inpath:**表示加载数据的路径
- （4）**overwrite:**表示覆盖表中已有数据，否则表示追加
- （5）**into table:**表示加载到哪张表
- （6）**student:**表示具体的表
- （7）**partition:**表示上传到指定分区



### 五、查询

```shell
[WITH CommonTableExpression (, CommonTableExpression)*]  (Note: Only available
 starting with Hive 0.13.0)
SELECT [ALL | DISTINCT] select_expr, select_expr, ...
 FROM table_reference
 [WHERE where_condition]
 [GROUP BY col_list]
 [ORDER BY col_list]
 [CLUSTER BY col_list
  | [DISTRIBUTE BY col_list] [SORT BY col_list]
 ]
 [LIMIT number]
```