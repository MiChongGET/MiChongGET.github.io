---
title: BigData--大数据技术之SparkSQL
date: 2020-09-03 15:44:59
tags:
- Spark
- SparkSQL
categories:
- BigData
description: Spark SQL是Spark用来处理结构化数据的一个模块，它提供了2个编程抽象：DataFrame和DataSet，并且作为分布式SQL查询引擎的作用。
top_img: https://file.buildworld.cn/img/20200903155309.jpeg
cover: https://file.buildworld.cn/img/20200903155458.png
---

![](https://file.buildworld.cn/img/20200810132517.png)

### 一、Spark SQL概述

#### 1、DataFrame

> 与RDD类似，DataFrame也是一个分布式数据容器。然而DataFrame更像传统数据库的二维表格，除了数据以外，还记录数据的结构信息，即schema。同时，与Hive类似，DataFrame也支持嵌套数据类型（struct、array和map）。从API易用性的角度上看，DataFrame API提供的是一套高层的关系操作，比函数式的RDD API要更加友好，门槛更低。

#### 2、DataSet

> - 1）是`Dataframe API`的一个扩展，是Spark最新的数据抽象。
> - 2）用户友好的API风格，既具有类型安全检查也具有Dataframe的查询优化特性。
> - 3）Dataset支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。
> - 4）样例类被用来在Dataset中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称。
> - 5） Dataframe是Dataset的特列，`DataFrame=Dataset[Row]` ，所以可以通过as方法将Dataframe转换为Dataset。Row是一个类型，跟Car、Person这些的类型一样，所有的表结构信息我都用Row来表示。
> - 6）DataSet是强类型的。比如可以有Dataset[Car]，Dataset[Person].
> - 7）DataFrame只是知道字段，但是不知道字段的类型，所以在执行这些操作的时候是没办法在编译的时候检查是否类型失败的，比如你可以对一个String进行减法操作，在执行的时候才报错，而DataSet不仅仅知道字段，而且知道字段类型，所以有更严格的错误检查。就跟JSON对象和类对象之间的类比。

![](https://file.buildworld.cn/img/20200810163552.png)

### 二、SparkSQL程序

#### 1、user.json

```json
{"id" : "1201", "name" : "satish", "age" : "25"}
{"id" : "1202", "name" : "krishna", "age" : "28"}
{"id" : "1203", "name" : "amith", "age" : "39"}
{"id" : "1204", "name" : "javed", "age" : "23"}
{"id" : "1205", "name" : "prudvi", "age" : "23"}
```

#### 2、基本使用

```scala
def main(args: Array[String]): Unit = {

  //设置配置
  val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Spark SQL")

  //创建SparkSession
  val spark = SparkSession
    .builder()
    .appName("Spark SQL basic example")
    .config(sparkConf)
    .getOrCreate()

  //加载json数据
  val dataFrame = spark.read.json("data\\user.json")

  //创建user视图
  dataFrame.createOrReplaceTempView("user")

  //执行SQL语句，并打印结果
  spark.sql("select * from user where age > 25").show()

  //关闭
  spark.stop
}
```

#### 3、相互转换

```scala
//设置配置
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Spark SQL")

//创建SparkSession
val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config(sparkConf)
  .getOrCreate()

//进行转换之前，需要引入隐式转换规则
import spark.implicits._

// 创建RDD
val rdd = spark.sparkContext.makeRDD(List((1, "michong", 20), (2, "qjzxzxd", 21), (3, "米虫", 18)))

// 转换为DF
val df = rdd.toDF("id", "name", "age")
df.show()

// 转换为DS
val ds = df.as[User]

// 转换为DF
val df1 = ds.toDF()

// 转换为RDD
val rdd1 = df1.rdd
rdd1.foreach(row=>{
  println(row.getString(1))
})

//释放资源
spark.stop
```

#### 4、RDD和DataSet之间相互转换

```scala
//设置配置
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Spark SQL")

//创建SparkSession
val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config(sparkConf)
  .getOrCreate()

//进行转换之前，需要引入隐式转换规则
import spark.implicits._

// 创建RDD
val rdd = spark.sparkContext.makeRDD(List((1, "michong", 20), (2, "qjzxzxd", 21), (3, "米虫", 18)))

val userRDD = rdd.map{
  case (id,name,age)=>{
    User(id,name,age)
  }
}

//RDD转换为DataSet
val userDS = userRDD.toDS()

//Represents the content of the Dataset as an `RDD` of `T`.
val rdd1 = userDS.rdd

rdd1.foreach(println)

//释放资源
spark.stop
```

  

#### 5、用户自定义聚合函数

##### 方式一

```scala
object hello4 {
  def main(args: Array[String]): Unit = {
    //设置配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Spark SQL")
    //创建SparkSession
    val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config(sparkConf)
      .getOrCreate()
    //创建聚合函数
    val udaf = new MyAgeAvgFunction
    spark.udf.register("avgAge",udaf)
    //使用聚合函数
    val frame = spark.read.json("data/user.json")
    frame.createOrReplaceTempView("user")
    spark.sql("select avgAge(age) from user").show

    spark.stop
  }
}

// 声明用户自定义聚合函数
// 1）继承UserDefinedAggregateFunction
// 2）实现方法
class MyAgeAvgFunction extends UserDefinedAggregateFunction {

  // 函数输入的数据结构
  override def inputSchema: StructType = {
    new StructType().add("age", LongType)
  }

  // 计算时的数据结构
  override def bufferSchema: StructType = {
    new StructType().add("sum", LongType).add("count", LongType)
  }

  // 函数返回的数据类型
  override def dataType: DataType = DoubleType

  // 函数是否稳定
  override def deterministic: Boolean = true

  //计算之前的缓冲区的初始化
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0) = 0L
    buffer(1) = 0L
  }

  // 根据查询结果更新缓冲区的数据
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    buffer(0) = buffer.getLong(0) + input.getLong(0)
    buffer(1) = buffer.getLong(1) + 1
  }

  // 将多个节点的缓冲区合并
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    // sum
    buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
    // count
    buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
  }

  // 计算
  override def evaluate(buffer: Row): Any = {
    buffer.getLong(0).toDouble / buffer.getLong(1)
  }
}
```

##### 方式二（强类型）

```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.{DataFrame, Dataset, Encoder, Encoders, SparkSession}
import org.apache.spark.sql.expressions.Aggregator

object hello5 {
  def main(args: Array[String]): Unit = {
    //设置配置
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Spark SQL")

    //创建SparkSession
    val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config(sparkConf)
      .getOrCreate()

    // 引入隐式转换
    import spark.implicits._

    //创建聚合函数
    val udaf = new MyAgeAvgClassFunction

    //将聚合函数转化为查询列
    val avgCol = udaf.toColumn.name("avgAge")

    //使用聚合函数
    val frame:DataFrame = spark.read.json("data/user.json")
    val userDS :Dataset[UserBean]= frame.as[UserBean]

    //应用函数
    userDS.select(avgCol).show()
    spark.stop
  }
}

case class UserBean(name: String, age: BigInt)
case class AvgBuffer(var sum: BigInt, var count: Int)

// 声明用户自定义聚合函数(强类型)
// 1）继承Aggregator
// 2）实现方法
class MyAgeAvgClassFunction extends Aggregator[UserBean, AvgBuffer, Double] {
  //初始化
  override def zero: AvgBuffer = {
    AvgBuffer(0, 0)
  }

  /**
   * 聚合数据
   *
   * @param b
   * @param a
   * @return
   */
  override def reduce(b: AvgBuffer, a: UserBean): AvgBuffer = {
    b.sum = b.sum + a.age
    b.count = b.count + 1
    b
  }

  /**
   * 缓冲区合并操作
   *
   * @param b1
   * @param b2
   * @return
   */
  override def merge(b1: AvgBuffer, b2: AvgBuffer): AvgBuffer = {
    b1.sum = b1.sum + b2.sum
    b1.count = b1.count + b2.count
    b1
  }

  /**
   * 完成计算
   *
   * @param reduction
   * @return
   */
  override def finish(reduction: AvgBuffer): Double = {
    reduction.sum.toDouble / reduction.count
  }

  override def bufferEncoder: Encoder[AvgBuffer] = Encoders.product

  override def outputEncoder: Encoder[Double] = Encoders.scalaDouble
}
```

#### 6、Spark连接MySQL数据库

```scala
//设置配置
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Spark SQL")

//创建SparkSession
val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config(sparkConf)
  .getOrCreate()

val jdbcDF = spark.read
  .format("jdbc")
  .option("url", "jdbc:mysql://127.0.0.1:3306/qiniuyun?serverTimezone=CTT&useUnicode=true&characterEncoding=UTF8")
  .option("dbtable", "myfile")
  .option("user", "root")
  .option("password", "root")
  .load()

// 创建视图
jdbcDF.createOrReplaceTempView("myfile")
// 查询出数据
spark.sql("select * from myfile").show
```