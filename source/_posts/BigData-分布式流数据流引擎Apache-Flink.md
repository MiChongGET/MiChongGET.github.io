---
title: BigData--分布式流数据流引擎Apache Flink
date: 2020-09-11 10:59:30
tags:
- Flink
- Apache
categories:
- BigData
description: Apache Flink® — Stateful Computations over Data Streams.
top_img: https://file.buildworld.cn/img/20200913210327.jpg
cover: https://file.buildworld.cn/img/20200911110402.png
---

![](https://flink.apache.org/img/flink-header-logo.svg)



[官网：https://flink.apache.org/](https://flink.apache.org/)

### 一、Flink的重要特点

#### 1）事件驱动型（Event-driven）

> - `事件驱动的应用程序`是一个有状态的应用程序，它从一个或多个事件流接收事件，并通过触发计算、状态更新或外部操作对传入事件作出反应。
> - `事件驱动应用程序`是传统应用程序设计的一种发展，它具有分离的计算和数据存储层。在这种体系结构中，应用程序从远程事务数据库读取数据并将其持久化。
> - 相反，`事件驱动应用程序`基于有状态流处理应用程序。在这个设计中，数据和计算被放在同一个位置，从而产生本地（内存或磁盘）数据访问。容错是通过定期将检查点写入远程持久存储来实现的。下图描述了传统应用程序体系结构与事件驱动应用程序之间的区别。

![](https://file.buildworld.cn/img/20200911125128.png)

**kafka作为消息队列就是一种典型的事件驱动型应用。**

#### 2） 流、批（stream，micro-batching）

> `Spark`中，一切都是批次组成的，离线数据是一个大批次，实时数据是一个个无限的小批次组成的。
>
> `Flink`中，一切都是由流组成的，离线数据是有界限的流，实时数据是一个没有界限的流，这就是所谓的有界流和无界流。

#### 3）分层API

![](https://file.buildworld.cn/img/20200911155213.png)



>越顶层越抽象，最高层级的抽象是SQL。
>
>越底层越具体



### 二、Flink使用（word count）

#### 1、设置pom文件

> 注意下面的依赖设置，使用的是scala 2.12.x版本，Flink版本为1.10.1

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.buildworld.flink</groupId>
    <artifactId>FlinkTrain</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-streaming-scala -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- 该插件用于将Scala代码编译成class文件 -->
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>4.4.0</version>
                <executions>
                    <execution>
                        <!-- 声明绑定到maven的compile阶段 -->
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

#### 2、编写scala代码

##### 1）批处理 wordcount

```scala
package cn.buildworld.flink

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment}
import org.apache.flink.api.scala._

// 批处理的word count
object WordCount {
  def main(args: Array[String]): Unit = {

    //创建一个批处理的执行环境
    val env: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment

    //从文件中读取数据
    val inputPath = "D:\\Java\\project\\Scala\\FlinkTrain\\src\\main\\resources\\hello.txt"

    val dataSet: DataSet[String] = env.readTextFile(inputPath)

    // 对数据进行转换处理统计，先分词，再按照word进行分组，最后进行聚合统计

    val resultDataSet: DataSet[(String, Int)] = dataSet
      .flatMap(_.split(" "))
      .map((_, 1))
      .groupBy(0) //以第一个元素为key进行分组
      .sum(1) //对所有数据的第二个元素求和

    resultDataSet.print()
  }
}
```

##### 2）流处理wordcount

**超级简单，比sparkstreaming的流式处理简单多了！！！**

```scala
import org.apache.flink.streaming.api.scala._

/**
 * 流处理的word count
 *
 */
object WordCountByStream {
  def main(args: Array[String]): Unit = {

    //创建一个批处理的执行环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

    // 设置并行度
    env.setParallelism(6)

    //从端口中读取数据
    val dataSet: DataStream[String] = env.socketTextStream("192.168.162.102", 7777)

    // 对数据进行转换处理统计，先分词，再按照word进行分组，最后进行聚合统计

    val resultDataSet = dataSet
      .flatMap(_.split(" "))
      .filter(_.nonEmpty)
      .map((_, 1))
      .keyBy(0) //以第一个元素为key进行分组
      .sum(1) //对所有数据的第二个元素求和

    resultDataSet.print()

    // 启动任务执行
    env.execute()
  }
}
```

**补充**

```scala
import org.apache.flink.api.java.utils.ParameterTool

//可以冲启动参数里面读取指定的参数
val parameterTool: ParameterTool = ParameterTool.fromArgs(args)
val host: String = parameterTool.get("host")
val port: Int = parameterTool.getInt("port")
```

###  三、Flink 运行架构

#### 1、Flink运行时组件

![](https://file.buildworld.cn/img/20200914200131.png)

> - **作业管理器（JobManager）**
>   控制一个应用程序执行的主进程，也就是说，每个应用程序都会被一个不同的JobManager 所控制执行。JobManager 会先接收到要执行的应用程序，这个应用程序会包括：作业图（JobGraph）、逻辑数据流图（logical dataflow graph）和打包了所有的类、库和其它资源的 JAR 包。JobManager 会把 JobGraph 转换成一个物理层面的数据流图，这个图被叫做“执行图”（ExecutionGraph），包含了所有可以并发执行的任务。JobManager 会向资源管理器（ResourceManager）请求执行任务必要的资源，也就是任务管理器（TaskManager）上的插槽（slot）。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的
>   TaskManager 上。而在运行过程中，JobManager 会负责所有需要中央协调的操作，比如说检查点（checkpoints）的协调。
>
> - **资源管理器（ResourceManager）**
>   主要负责管理任务管理器（TaskManager）的插槽（slot），TaskManger 插槽是 Flink 中
>   定义的处理资源单元。Flink 为不同的环境和资源管理工具提供了不同资源管理器，比如
>   YARN、Mesos、K8s，以及 standalone 部署。当 JobManager 申请插槽资源时，ResourceManager会将有空闲插槽的 TaskManager 分配给 JobManager。如果 ResourceManager 没有足够的插槽来满足 JobManager 的请求，它还可以向资源提供平台发起会话，以提供启动 TaskManager进程的容器。另外，ResourceManager 还负责终止空闲的 TaskManager，释放计算资源。
>
> - **任务管理器（TaskManager）**
>
>   Flink 中的工作进程。通常在 Flink 中会有多个 TaskManager 运行，每一个 TaskManager都包含了一定数量的插槽（slots）。插槽的数量限制了 TaskManager 能够执行的任务数量。
>   启动之后，TaskManager 会向资源管理器注册它的插槽；收到资源管理器的指令后，TaskManager 就会将一个或者多个插槽提供给 JobManager 调用。JobManager 就可以向插槽分配任务（tasks）来执行了。在执行过程中，一个 TaskManager 可以跟其它运行同一应用程
>   序的 TaskManager 交换数据。
>
> - **分发器（Dispatcher）**
>
>   可以跨作业运行，它为应用提交提供了 REST 接口。当一个应用被提交执行时，分发器
>   就会启动并将应用移交给一个 JobManager。由于是 REST 接口，所以 Dispatcher 可以作为集
>   群的一个 HTTP 接入点，这样就能够不受防火墙阻挡。Dispatcher 也会启动一个 Web UI，用
>   来方便地展示和监控作业执行的信息。Dispatcher 在架构中可能并不是必需的，这取决于应
>   用提交运行的方式。

  #### 2、任务提交流程

![](https://file.buildworld.cn/img/20200914205020.png)

#### 3、任务调度原理

Task Slot  是静态的概念，是指 TaskManager  具有的并发执行能力，可以通过参数 taskmanager.numberOfTaskSlots 进行配置；而 并行度 parallelism  是动态概念 ，即 即 TaskManager  运行程序时实际使用的并发能力，可以通过参数 parallelism.default进行配置。

### 四、Flink流处理API

#### 1、三种不同方式读取数据

bin/kafka-topics.sh  --zookeeper localhost:2181 --list

bin/kafka-topics.sh  --zookeeper localhost:2181 --create --replication-factor 3 --partitions 1 --topic sensor

bin/kafka-console-producer.sh --broker-list 10.81.1.56:9092 --topic sensor

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic sensor

```
sensor_1, 1547718199, 35.8
```

```scala
import java.util.Properties

import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer011


// 定义样例类，温度传感器
case class SensorReading(id: String, timestamp: Long, temperature: Double)


object SourceTest {
  def main(args: Array[String]): Unit = {

    //创建执行环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

//    //1、从集合中读取数据
//    val data = List(
//      SensorReading("sensor_1", 1547718199, 35.8),
//      SensorReading("sensor_6", 1547718201, 15.4),
//      SensorReading("sensor_7", 1547718202, 6.7),
//      SensorReading("sensor_10", 1547718205, 38.1)
//    )
//
//    val stream1: DataStream[SensorReading] = env.fromCollection(data)
//    stream1.print()
//
//    //2、从文件中读取数据
//    val stream2 = env.readTextFile("YOUR_FILE_PATH")

    //3、从kafka中读取数据
    val properties = new Properties()
    properties.setProperty("bootstrap.servers", "localhost:9092")
    properties.setProperty("group.id", "consumer-group")
    properties.setProperty("auto.offset.reset", "latest")

    val stream3: DataStream[String] = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))

    val value: DataStream[String] = stream3.flatMap(_.split(",")).filter(_.nonEmpty)

    value.print()
    //执行
    env.execute()
  }
}
```

#### 2、自定义 Source

```scala
class MySensorSource extends SourceFunction[SensorReading] {

  // flag: 表示数据源是否还在正常运行
  var running: Boolean = true
  var num = 0
    
  override def run(sourceContext: SourceFunction.SourceContext[SensorReading]): Unit = {
    //定义一个随机数发生器
    val rand = new Random()

    //随机生成一组传感器的初始温度
    var curTemp = 1.to(10).map(
      i => ("sensor_" + i, 65 + rand.nextGaussian() * 20)
    )

    while (running) {
      //更新温度值
      curTemp = curTemp.map(
        t => (t._1, t._2 + rand.nextGaussian())
      )

      //获取当前的时间戳
      val curTime: Long = System.currentTimeMillis()

      curTemp.foreach(
        t => sourceContext.collect(SensorReading(t._1, curTime, t._2))
      )

      num+=1

      if (num == 5){
        cancel()
      }

      Thread.sleep(1000)
    }
  }

  override def cancel(): Unit = {
    running = false
  }
}
```

#### 使用自定义Source

```scala
val stream4: DataStream[SensorReading] = env.addSource(new MySensorSource())
stream4.print()
```

#### 3、Transform转换算子

##### map

##### flatMap

##### Filter

##### KeyBy

##### 滚动聚合算子（Rolling Aggregation） 

> 这些算子可以针对 KeyedStream 的每一个支流做聚合。
>
> sum()
> min()
> max()
> minBy()
> maxBy()

##### Reduce

> `KeyedStream` → → `DataStream`：一个分组数据流的聚合操作，合并当前的元素和上次聚合的结果，产生一个新的值，返回的流中包含每一次聚合的结果，而不是只返回最后一次聚合的最终结果。

```scala
val resultStream = dataStream
  .keyBy("id")
  .reduce((curState, newData) =>
    SensorReading(curState.id, newData.timestamp, curState.temperature.min(newData.temperature))
  )
```

#####  Split 和 和 Select

![DataStream → → SplitStream：根据某些特征把一个 DataStream 拆分成两个或者多个 DataStream。](https://file.buildworld.cn/img/20201011165829.png)



![SplitStream →DataStream：从一个 SplitStream 中获取一个或者多个DataStream。](https://file.buildworld.cn/img/20201011165909.png)

```scala
case class SensorReading(id: String, timestamp: Long, temperature: Double)

val resultStream = dataStream
  .keyBy("id")
  .reduce((curState, newData) =>
    SensorReading(curState.id, newData.timestamp, curState.temperature.min(newData.temperature))
  )

// 数据过滤，按照指定条件将数据分开来
val splitStream = resultStream.split(data => {
  if (data.temperature > 30)
    Seq("high")
  else Seq("low")
})

val high = splitStream.select("high")
val low = splitStream.select("low")
val all = splitStream.select("high", "low")
```

##### Connect和CoMap

![](https://file.buildworld.cn/img/20201012100125.png)

![](https://file.buildworld.cn/img/20201012100146.png)

```scala
val warning: DataStream[(String, Double)] = high.map(sensorData => (sensorData.id, sensorData.temperature))

// DataStream,DataStream → → ConnectedStreams：连接两个保持他们类型的数据流，两个数据流被 Connect 之后，只是被放在了一个同一个流中，内部依然保持各自的数据和形式不发生任何变化，两个流相互独立。
val connected: ConnectedStreams[(String, Double), SensorReading] = warning.connect(low)

// ConnectedStreams → DataStream：作用于 ConnectedStreams 上，功能与 map和 flatMap 一样，对 ConnectedStreams 中的每一个 Stream 分别进行 map 和 flatMap处理。
val coMap: DataStream[Product] = connected.map(
  warningData => (warningData._1, warningData._2, "WARNING"),
  lowData => (lowData.id, lowData.temperature, "SAFE")
)

coMap.print()


// 打印结果
10> (sensor_7,6.7,SAFE)
8> (sensor_6,15.4,SAFE)
6> (sensor_10,38.1,WARNING)
7> (sensor_1,30.8,WARNING)
7> (sensor_1,30.8,WARNING)
7> (sensor_1,30.8,WARNING)
7> (sensor_1,30.8,WARNING)
7> (sensor_1,30.8,WARNING)
7> (sensor_1,30.8,WARNING)
```

##### Union

![](https://file.buildworld.cn/img/20201012101514.png)

> - Union 之前两个流的类型必须是一样，Connect 可以不一样，在之后的 coMap中再去调整成为一样的。
> - Connect 只能操作两个流，Union 可以操作多个。

```scala
val unionStream: DataStream[SensorReading] = high.union(low)
unionStream.print("union:::")
```

#### 4、支持的数据类型

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setParallelism(1)

//基本数据类型
val numbers: DataStream[Long] = env.fromElements(1L, 2L, 3L, 4L)
val res: DataStream[Long] = numbers.map(n => n + 1)
res.print("相加：")

// 元组
val persons1: DataStream[(String, Int)] = env.fromElements(
  ("michong", 25),
  ("lili", 15)
)
//元组数据过滤
val res1: DataStream[(String, Int)] = persons1.filter(p => p._2 > 18)

// Scala样例类
val persons2: DataStream[Person] = env.fromElements(
  Person("MiChong", 25),
  Person("Lili", 15)
)

val res2: DataStream[Person] = persons2.filter(
  p => p.age > 18
)

res2.print("成年人： ")
env.execute()
```

#### 5、实现 UDF 函数——更细粒度的控制流

###### 函数类（Function Classes）

```scala
val res2: DataStream[Person] = persons2.filter(new MyFilter)

class MyFilter extends FilterFunction[Person] {
  override def filter(p: Person): Boolean = {
    p.age > 18
  }
}
```

 ###### 富函数（Rich Functions）

> “富函数”是 DataStream API 提供的一个函数类的接口，所有 Flink 函数类都有其 Rich 版本。它与常规函数的不同在于，可以获取运行环境的上下文，并拥有一些生命周期方法，所以可以实现更复杂的功能。
>
> **生命周期**
>
> - `open()`方法是 rich function 的初始化方法，当一个算子例如 map 或者 filter被调用之前 open()会被调用。
> - `close()`方法是生命周期中的最后一个调用的方法，做一些清理工作。
> - `getRuntimeContext()`方法提供了函数的 RuntimeContext 的一些信息，例如函数执行的并行度，任务的名字，以及 state 状态

```scala
class MyFlatMap extends RichFlatMapFunction[Int, (Int, Int)] {

  var subTaskIndex = 0

  override def open(parameters: Configuration): Unit = {
    subTaskIndex = getRuntimeContext.getIndexOfThisSubtask
    //做一些初始化工作，建立HDFS的连接

  }

  override def flatMap(in: Int, collector: Collector[(Int, Int)]): Unit = {
    if (in % 2 == subTaskIndex) {
      collector.collect((subTaskIndex, in))
    }
  }
    
  override def close(): Unit = {
    // 以下做一些清理工作，断开HDFS的连接
  }
}
```

#### 6、Sink

> Flink的对外输出操作

```scala
import java.util.Properties

import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.{FlinkKafkaConsumer011, FlinkKafkaProducer011}

// 定义样例类，温度传感器
case class SensorReading(id: String, timestamp: Long, temperature: Double)

object SourceTest {
  def main(args: Array[String]): Unit = {

    //创建执行环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

    //3、从kafka中读取数据
    val properties = new Properties()
    properties.setProperty("bootstrap.servers", "10.12.42.174:9092")
    properties.setProperty("group.id", "consumer-group")
    //    properties.setProperty("auto.offset.reset", "latest")


    val stream3: DataStream[String] = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))

    // 将从kafka获取的数据处理
    val outputStream: DataStream[String] = stream3.map(
      data => {
        var arr = data.split(",")
        SensorReading(arr(0), arr(1).toLong, arr(2).toDouble).toString
      }
    )

    // 将处处之后的数据重新发送到kafka中
    outputStream.addSink(new FlinkKafkaProducer011[String]("10.12.42.174:9092", "sensor_res", new SimpleStringSchema()))
    outputStream.print()

    //执行
    env.execute()
  }
}
```

```xml
<dependency>
    <groupId>org.apache.flink</groupId>

    <artifactId>flink-connector-kafka-0.11_2.12</artifactId>
    <version>1.10.1</version>
</dependency>
```

###### JDBC自定义sink

> 数据从Kafka获取，然后进行dataStream转换，最后将结果保存在mysql中

```scala
import java.sql.{Connection, DriverManager, PreparedStatement}
import java.util.Properties

import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.functions.sink.{RichSinkFunction, SinkFunction}
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.{FlinkKafkaConsumer011, FlinkKafkaProducer011}
// 定义样例类，温度传感器
case class SensorReading(id: String, timestamp: Long, temperature: Double)

object SourceTest {
  def main(args: Array[String]): Unit = {

    //创建执行环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

    //3、从kafka中读取数据
    val properties = new Properties()
    properties.setProperty("bootstrap.servers", "10.12.42.174:9092")
    properties.setProperty("group.id", "consumer-group")
    //    properties.setProperty("auto.offset.reset", "latest")


    val stream3: DataStream[String] = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))

    // 将从kafka获取的数据处理
    val outputStream: DataStream[SensorReading] = stream3.map(
      data => {
        var arr = data.split(",")
        SensorReading(arr(0), arr(1).toLong, arr(2).toDouble)
      }
    )

    // 将处处之后的数据重新发送到kafka中
    //    outputStream.addSink(new FlinkKafkaProducer011[String]("10.12.42.174:9092", "sensor_res", new SimpleStringSchema()))
    //    outputStream.print()

    outputStream.addSink(new MyJdbcSink())
    //执行
    env.execute()
  }
}


/**
 *
 * 自定义关于Sink的JDBC连接处理
 *
 */

class MyJdbcSink() extends RichSinkFunction[SensorReading] {

  // 初始化自定义参数
  var conn: Connection = _
  var insertStmt: PreparedStatement = _
  var updateStmt: PreparedStatement = _

  // 创建数据库连接
  override def open(parameters: Configuration): Unit = {

    conn = DriverManager.getConnection("jdbc:mysql://10.12.42.174/flink", "root", "root")
    insertStmt = conn.prepareStatement("insert into sensor_temp(id,temp) values (?,?)")
    updateStmt = conn.prepareStatement("update sensor_temp set temp =? where id = ? ")
  }

  override def invoke(value: SensorReading, context: SinkFunction.Context[_]): Unit = {

    // 先执行更新程序，查到就更新
    updateStmt.setDouble(1, value.temperature)
    updateStmt.setString(2, value.id)
    updateStmt.execute()

    // 如果没有更新的内容就插入
    if (updateStmt.getUpdateCount == 0) {
      insertStmt.setString(1, value.id)
      insertStmt.setDouble(2, value.temperature)
      insertStmt.execute()
    }
  }

  override def close(): Unit = {
    insertStmt.close()
    updateStmt.close()
    conn.close()
  }
}
```

### 五、Flink中的Window

> Window是一种**切割无限数据**为**有限块**进行处理的手段。

#### 1、Window类型

- CountWindow:按照指定的数据条数生成一个window，和时间没有关系
- TimeWindow:按照时间生成window

#### 2、窗口实现原理的不同分成三类

##### 1）滚动窗口（Tumbling Windows)

>将数据依据`固定的窗口长度`对数据进行`切片`。
>
>**特点：时间对齐，窗口长度固定，没有重叠。**

##### 2）滑动窗口（Sliding Windows）

> 滑动窗口由`固定的窗口长度`和`滑动间隔`组成。
>
> **特点：时间对齐，窗口长度固定，可以有重叠**

##### 3）会话窗口（Session Windows)

>由一系列事件组合一个指定时间长度的 `timeout` 间隙组成，类似于 web 应用的session，也就是一段时间没有接收到新数据就会生成新的窗口。
>
>**特点：时间无对齐。**
>
>![](https://file.buildworld.cn/img/20201013095706.png)

#### 3、Window API

##### TimeWindow

```scala
import org.apache.flink.streaming.api.windowing.time.Time

// 将从kafka获取的数据处理
val outputStream: DataStream[SensorReading] = stream3.map(
  data => {
    val arr = data.split(",")
    SensorReading(arr(0), arr(1).toLong, arr(2).toDouble)
  }
)

outputStream
  .map(data => (data.id, data.temperature))
  .keyBy(_._1) //安装二元组的第一个元素（id）分组
  .timeWindow(Time.seconds(10)) // 一个参数就是滚动窗口，两个参数就是滑动窗口
  .window(SlidingEventTimeWindows.of(Time.seconds(15),Time.seconds(5)))  // 会话窗口
  .reduce((curRes, newData) => (curRes._1, curRes._2.min(newData._2), newData._3))
```

##### CountWindow

```scala 
outputStream
  .map(data => (data.id, data.temperature))
  .keyBy(_._1) //安装二元组的第一个元素（id）分组
  .countWindow(5) // 一个参数就是滚动窗口，两个参数就是滑动窗口
  .reduce((curRes, newData) => (curRes._1, curRes._2.min(newData._2), newData._3))
```

#### 4、window function

> - **增量聚合函数（incremental aggregation functions）**
>   每条数据到来就进行计算，保持一个简单的状态。典型的增量聚合函数有ReduceFunction, AggregateFunction。
> - **全窗口函数（full window functions）**
>   先把窗口所有数据收集起来，等到计算的时候会遍历所有数据。ProcessWindowFunction 就是一个全窗口函数。

### 六、时间语义与 Wartermark

**在 Flink  的流式处理中，绝大部分的业务都会使用 `eventTime`**

```scala
//创建执行环境
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
// 从调用时刻开始给 env 创建的每一个 stream 追加时间特征
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

####  Watermark

- Watermark 是一种衡量 Event Time 进展的机制。
- **Watermark  是用于处理乱序事件的**，而正确的处理乱序事件，通常用Watermark 机制结合 window 来实现。
- 数据流中的 Watermark 用于表示 timestamp 小于 Watermark 的数据，都已经到达了，因此，window 的执行也是由 Watermark 触发的。
- Watermark 可以理解成一个延迟触发机制，我们可以设置 Watermark 的延时时长 t，每次系统会校验已经到达的数据中最大的 maxEventTime，然后认定 eventTime小于 maxEventTime - t 的所有数据都已经到达，如果有窗口的停止时间等于maxEventTime – t，那么这个窗口被触发执行。



### 七、ProcessFunction API（底层 API）

> Process Function 用来构建事件驱动的应用以及实现自定义的业务逻辑(使用之前的window 函数和转换算子无法实现)。
>
> Flink 提供了 8 个 Process Function：
> •  ProcessFunction
> •  KeyedProcessFunction
> •  CoProcessFunction
> •  ProcessJoinFunction
> •  BroadcastProcessFunction
> •  KeyedBroadcastProcessFunction
> •  ProcessWindowFunction
> •  ProcessAllWindowFunction

#### TimerService 和 定时器（Timers）案例

>监控温度传感器的温度值，如果温度值在 10 秒钟之内(processing time)连续上升，则报警

```java
import cn.buildworld.flink.processfunc.bean.SensorReading;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;

/**
 * @author MiChong
 * @date 2020-12-07 15:02
 */
public class ProcessFunction_App {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        //socket 文本流
        DataStreamSource<String> inputStream = env.socketTextStream("localhost", 7777);

        //转换为SensorReading类型
        DataStream<SensorReading> dataStream = inputStream.map(line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        });


        //测试keyedProcessFunction，先分组再定义
        dataStream.keyBy("id")
                .process(new TempConsIncreWarning(10))
                .print();

        env.execute();
    }

    // 实现自定义处理函数，检测一段时间内的温度连续上升，输出报警
    public static class TempConsIncreWarning extends KeyedProcessFunction<Tuple, SensorReading, String> {

        // 定义私有属性，当前统计的时间间隔
        private Integer interval;

        //定义状态，保存上一次的温度值，定时器时间戳
        private ValueState<Double> lastTempState;
        private ValueState<Long> timerTsState;


        public TempConsIncreWarning(Integer interval) {
            this.interval = interval;
        }

        @Override
        public void open(Configuration parameters) throws Exception {
            lastTempState = getRuntimeContext().getState(new ValueStateDescriptor<Double>("last-temp", Double.class, Double.MIN_VALUE));
            timerTsState = getRuntimeContext().getState(new ValueStateDescriptor<Long>("time-ts", Long.class));
        }

        @Override
        public void processElement(SensorReading sensorReading, Context context, Collector<String> collector) throws Exception {

            //取出状态
            Double lastTemp = lastTempState.value();
            Long timerTs = timerTsState.value();
            
            //更新温度状态
            lastTempState.update(sensorReading.getTemperature());

            //如果温度上升并且没有定时器的时候，注册10秒之后的定时器，开始等待
            if (sensorReading.getTemperature() > lastTemp && timerTs == null) {
                //计算出定时器的时间戳
                Long ts = context.timerService().currentProcessingTime() + interval * 1000L;
                context.timerService().registerProcessingTimeTimer(ts);
                timerTsState.update(ts);
                System.out.println("温度上升");

            }

            //如果温度下降，删除定时器
            else if (sensorReading.getTemperature() < lastTemp && timerTs != null) {
                context.timerService().deleteProcessingTimeTimer(timerTs);
                timerTsState.clear();
                System.out.println("温度下降");
            }
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            out.collect("传感器：" + ctx.getCurrentKey().getField(0) + "温度值连续" + interval + "秒上升" + ",当前温度为：" + lastTempState.value());
            timerTsState.clear();
        }

        @Override
        public void close() throws Exception {
            lastTempState.clear();
        }
    }
}
```

#### 侧输出流（SideOutput）

> 案例：用来监控传感器温度值，将温度值低于 30 度的数据输出到 side output。

```java
public class ProcessFunction_SideOutputCase {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        //socket 文本流
        DataStreamSource<String> inputStream = env.socketTextStream("localhost", 7777);

        //转换为SensorReading类型
        DataStream<SensorReading> dataStream = inputStream.map(line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        });


        final OutputTag<SensorReading> lowTempTag = new OutputTag<SensorReading>("lowTemp") {
        };

        //测试keyedProcessFunction，自定义侧输出流实现分流操作
        SingleOutputStreamOperator<SensorReading> highTempStream = dataStream.keyBy("id")
                .process(new ProcessFunction<SensorReading, SensorReading>() {

                    @Override
                    public void processElement(SensorReading sensorReading, Context context, Collector<SensorReading> collector) throws Exception {

                        if (sensorReading.getTemperature() < 30) {
                            context.output(lowTempTag, sensorReading);
                        } else {
                            collector.collect(sensorReading);
                        }
                    }
                });

        DataStream<SensorReading> lowTempStream = highTempStream.getSideOutput(lowTempTag);

        // 分别输出
        highTempStream.print("high");
        lowTempStream.print("low");

        env.execute();
    }
}
```

### 八、状态编程和容错机制

> 流式计算分为`有状态`和`无状态`两种情况。无状态的计算观察每个独立事件，并根据最后一个事件输出结果（**无状态流处理**每次只转换一条输入记录，并且仅根据最新的输入记录输出结果）。有状态的计算则会基于多个事件输出结果（**有状态流处理**维护所有已处理记录的状态值，并根据每条新输入的记录更新状态，因此输出记录(灰条)反映的是综合考虑多个事件之后的结果。）。

#### 1、Flink检查点算法--检查点分界线（Checkpoint Barrier）

>- Flink 的检查点算法用到了一种称为分界线（barrier）的特殊数据形式，用来把一条流上数据按照不同的检查点分开。
>- 分界线之前到来的数据导致的状态更改，都会被包含在当前分界线所属的检查点中；而基于分界线之后的数据导致的所有更改，就会被包含在之后的检查点中。

#### 2、保存点（Savepoints）

>- Flink 还提供了可以自定义的镜像保存功能，就是`保存点（savepoints）`
>- 原则上，创建保存点使用的算法与检查点完全相同，因此保存点可以认为就是具有一些额外元数据的检查点
>- Flink不会自动创建保存点，因此用户（或者外部调度程序）必须明确地触发创建操作
>- 保存点是一个强大的功能。除了故障恢复外，保存点可以用于：有计划的手动备份，更新应用程序，版本迁移，暂停和重启应用，等等

#### 3、容错机制配置项

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//2、检查点配置
env.enableCheckpointing(300);
//高级选项
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
env.getCheckpointConfig().setCheckpointTimeout(60000L);
env.getCheckpointConfig().setMaxConcurrentCheckpoints(2);
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(100L);

//3、重启策略配置
// 固定延迟重启
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 10000L));
//失败率重启
env.setRestartStrategy(RestartStrategies.failureRateRestart(3, Time.minutes(10),Time.minutes(1)));
```

#### 4、状态一致性分类

> Flink 的一个重大价值在于， 它既保证了 `exactly-once` ，也具有`低延迟`和`高吞吐力`的处理能力。

![](https://file.buildworld.cn/img/20201208182144.png)

#### 5、Flink和kafka实现端到端的 exactly-once  语义

![](https://file.buildworld.cn/img/20201209123608.png)

### 九、Table API和Flink SQL

#### 1、引入pom依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>1.10.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
        
        <!--阿里巴巴贡献出来的Blink-->
		<dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner-blink_2.12</artifactId>
            <version>1.10.1</version>
        </dependency>
    </dependencies>
```

#### 2、简单的例子

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(1);

//1、读取数据
DataStreamSource<String> inputStream = env.readTextFile("D:\\Java\\project\\Flink_Java\\src\\main\\resources\\sensor.txt");

//2、转化成pojo
DataStream<SensorReading> dataStream = inputStream.map(
        line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        }
);

//3、创建表环境
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

//4、基于流创建一张表
Table dataTable = tableEnv.fromDataStream(dataStream);

//5、调用table API进行转化操作
Table resTable = dataTable.select("id,temperature")
        .where(" id = 'sensor_1'");

//6、执行SQL
tableEnv.createTemporaryView("sensor", dataTable);
String sql = "select id,temperature from sensor where id = 'sensor_1'";
Table resultSqlTable = tableEnv.sqlQuery(sql);

//7、打印出查询结果
tableEnv.toAppendStream(resTable, Row.class).print("result");
tableEnv.toAppendStream(resultSqlTable, Row.class).print("resultSql");

env.execute();
```

#### 3、新老版本的流、批处理方法

```java
// 1、创建环境
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(1);

//1.1 基于老版本planner的流处理
EnvironmentSettings oldStreamSettings = EnvironmentSettings
        .newInstance()
        .useOldPlanner()
        .inStreamingMode()
        .build();

StreamTableEnvironment oldStreamTableEnv = StreamTableEnvironment.create(env, oldStreamSettings);

//1.2 基于老版本planner的批处理
ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment();
BatchTableEnvironment oldBatchTableEnv = BatchTableEnvironment.create(batchEnv);

//1.3 基于Blink的流处理
EnvironmentSettings blinkStreamSettings = EnvironmentSettings
        .newInstance()
        .useBlinkPlanner()
        .inStreamingMode()
        .build();
StreamTableEnvironment blinkStreamTableEnv = StreamTableEnvironment.create(env, blinkStreamSettings);

//1.4 基于Blink的批处理
EnvironmentSettings blinkBatchSettings = EnvironmentSettings
        .newInstance()
        .useBlinkPlanner()
        .inBatchMode()
        .build();
TableEnvironment blinkBatchTableEnv = TableEnvironment.create(blinkBatchSettings);


env.execute();
```

#### 4、连接外部系统创建一张表

```java
// 1、创建环境
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(1);

StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// 2、表的创建：连接外部系统，读取数据
// 2.1 读取文件
String filePath = "D:\\Java\\project\\Flink_Java\\src\\main\\resources\\sensor.txt";
tableEnv.connect(new FileSystem().path(filePath))
.withFormat(new Csv())
.withSchema(new Schema()
        .field("id", DataTypes.STRING())
        .field("timestamp",DataTypes.BIGINT())
        .field("temp",DataTypes.DOUBLE())
).createTemporaryTable("inputTable");

Table inputTable = tableEnv.from("inputTable");
inputTable.printSchema();
tableEnv.toAppendStream(inputTable, Row.class).print();

env.execute();
```

#### 5、查询转换

```java
/ 3 查询转换
// 3.1 Table API
// 简单转换
Table resultTable = inputTable.select("id,temperature")
        .filter("id = 'sensor_6'");
// 聚合统计
Table aggTable = inputTable.groupBy("id")
        .select("id,id.count as count,temperature.avg as avgTemp");

// 3.2 SQL
Table sqlTable = tableEnv.sqlQuery("select id,temperature from inputTable where id = 'sensor_6'");
Table sqlQuery = tableEnv.sqlQuery("select id,count(id) as cnt,avg(temperature) as avgTemp from inputTable group by id");

// 打印输出
tableEnv.toAppendStream(resultTable,Row.class).print("resultTable");
tableEnv.toRetractStream(aggTable,Row.class).print("aggTable");
tableEnv.toAppendStream(sqlTable,Row.class).print("sqlTable");
tableEnv.toRetractStream(sqlQuery,Row.class).print("sqlQuery");

 // 4、输出到文件
// 连接外部文件注册输出表
String outputFilePath = "D:\\Java\\project\\Flink_Java\\src\\main\\resources\\output_sensor.txt";

 tableEnv.connect(new FileSystem().path(outputFilePath))
                .withFormat(new Csv())
                .withSchema(new Schema()
                        .field("id", DataTypes.STRING())
//                        .field("cnt",DataTypes.BIGINT())
                        .field("temperature", DataTypes.DOUBLE()))
                .createTemporaryTable("outputTable");

Table outputTable = tableEnv.from("outputTable");
resultTable.insertInto("outputTable");
```

#### 6、Table&&Kafka

```java
 		// 1、创建环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 2、连接kafka，读取数据
        tableEnv.connect(new Kafka()
                .version("0.11")
                .topic("sensor")
                .property("zookeeper.connect", "10.12.42.174:2181")
                .property("bootstrap.servers", "10.12.42.174:9092"))
                .withFormat(new Csv())
                .withSchema(new Schema()
                        .field("id", DataTypes.STRING())
                        .field("timestamp", DataTypes.BIGINT())
                        .field("temp", DataTypes.DOUBLE()))
        .createTemporaryTable("inputTable");

        // 3、查询转换
        // 简单转换
        Table sensorTable = tableEnv.from("inputTable");
        Table resultTable = sensorTable.select("id,temp")
                .filter("id = 'sensor_6'");
        // 聚合统计
        Table aggTable = sensorTable.groupBy("id")
                .select("id,id.count as count,temp.avg as avgTemp");

        // 4、输出到文件
        // 连接外部文件注册输出表

        tableEnv.connect(new Kafka()
                .version("0.11")
                .topic("flink")
                .property("zookeeper.connect", "10.12.42.174:2181")
                .property("bootstrap.servers", "10.12.42.174:9092"))
                .withFormat(new Csv())
                .withSchema(new Schema()
                        .field("id", DataTypes.STRING())
//                        .field("timestamp", DataTypes.BIGINT())
                        .field("temp", DataTypes.DOUBLE()))
                .createTemporaryTable("outputTable");


        // 5、写入到kafka中
        resultTable.insertInto("outputTable");

        env.execute();
```

#### 7、更新模式

>- 对于流式查询，需要声明如何在表和外部连接器之间执行转换
>- 与外部系统交换的消息类型，由更新模式（Update Mode）指定

![](https://file.buildworld.cn/img/20201209210053.png)

#### 8、输出到MySQL

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-jdbc_2.12</artifactId>
    <version>1.10.1</version>
</dependency
```

```java
// 4、连接MySQL
String sinkDDL = "create table jdbcOutputTable (" +
        " id varchar(20) not null, " +
        " avgTemp double not null " +
        ") with (" +
        " 'connector.type' = 'jdbc', " +
        " 'connector.url' = 'jdbc:mysql://localhost:3306/flink', " +
        " 'connector.table' = 'sensor_count', " +
        " 'connector.driver' = 'com.mysql.jdbc.Driver', " +
        " 'connector.username' = 'root', " +
        " 'connector.password' = 'root' )";
tableEnv.sqlUpdate(sinkDDL);
aggTable.insertInto("jdbcOutputTable")
```

#### 9、动态表和持续查询

![](https://file.buildworld.cn/img/20201211151959.png)

> - 流被转换为动态表
> - 对动态表计算连续查询，生成新的动态表
> - 生成的动态表被转换回流

#### 10、Group Windows

> 滚动窗口（Tumbling windows）-- 滚动窗口要用 `Tumble` 类来定义

```java
// Tumbling Event-time Window
.window(Tumble.over("10.minutes").on("rowtime").as("w"))
// Tumbling Processing-time Window
.window( Tumble.over(" 10.minutes ").on(" proctime ").as("w"))
// Tumbling Row-count Window
.window( Tumble.over(" 10.rows ").on(" proctime ").as("w"))	
```

> 滑动窗口（Sliding windows）-- 滑动窗口要用 `Slide` 类来定义

```java
// Sliding Event-time Window
.window(Slide.over("10.minutes").every("5.minutes").on("rowtime").as("w"))
// Sliding Processing-time window
.window(Slide.over("10.minutes").every("5.minutes").on("proctime").as("w"))
// Sliding Row-count window
.window(Slide.over("10.rows").every("5.rows").on("proctime").as("w"))
```

#### 11、自定义UDF

##### 标量函数（Scalar Functions）

>• 用户定义的标量函数，可以将0、1或多个标量值，映射到新的标量值
>• 为了定义标量函数，必须在 `org.apache.flink.table.functions` 中扩展基类`ScalarFunction`，并实现（一个或多个）求值（eval）方法
>• 标量函数的行为由求值方法决定，求值方法必须公开声明并命名为 `eval`

```JAVA
public class ScalarFunctionTest {
    public static void main(String[] args) throws Exception {
        // 1、创建环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);


        // 2、表的创建：连接外部系统，读取数据
        // 2.1 读取文件
        String filePath = "D:\\Java\\project\\Flink_Java\\src\\main\\resources\\sensor.txt";
        DataStream<String> inputStream = env.readTextFile(filePath);

        // 3. 转换成POJO
        DataStream<SensorReading> dataStream = inputStream.map(line -> {
            String[] fields = line.split(",");
            return new SensorReading(fields[0], new Long(fields[1]), new Double(fields[2]));
        })
                .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<SensorReading>(Time.seconds(2)) {
                    @Override
                    public long extractTimestamp(SensorReading element) {
                        return element.getTimestamp() * 1000L;
                    }
                });

        // 4、将流转换为表
        Table sensorTable = tableEnv.fromDataStream(dataStream, "id,timestamp as ts,temperature as temp");

        // 自定义标量函数，实现求id的hash值
        // 4.1 table API
        HashCode hashCode = new HashCode(21);
        // 需要在环境中注册UDF
        tableEnv.registerFunction("hashCode",hashCode);
        Table resultTable = sensorTable.select("id,ts,hashCode(id)");

        // 4.2 SQL
        tableEnv.createTemporaryView("sensor",sensorTable);
        Table resultSqlTable = tableEnv.sqlQuery("select id,ts,hashCode(id) from sensor");

        tableEnv.toAppendStream(resultTable, Row.class).print("resultTable");
        tableEnv.toRetractStream(resultSqlTable, Row.class).print("resultSqlTable");

        env.execute();
    }

    // 实现自定义的ScalarFunction
    public static class HashCode extends ScalarFunction {
        private int factor = 13;

        public HashCode(int factor) {
            this.factor = factor;
        }

        public int eval(String s) {
            return s.hashCode() * factor;
        }
    }
}
```

##### 表函数（Table Functions）

>• 用户定义的表函数，也可以将0、1或多个标量值作为输入参数；与标量函数不同的是，它可以返回任意数量的行作为输出，而不是单个值
>• 为了定义一个表函数，必须扩展 `org.apache.flink.table.functions` 中的基类`TableFunction` 并实现（一个或多个）求值方法
>• 表函数的行为由其求值方法决定，求值方法必须是 `public` 的，并命名为 `eval`

```JAVA

    // 实现自定义TableFunction
    public static class Split extends TableFunction<Tuple2<String, Integer>>{
        // 定义属性，分隔符
        private String separator = ",";

        public Split(String separator) {
            this.separator = separator;
        }

        // 必须实现一个eval方法，没有返回值
        public void eval( String str ){
            for( String s: str.split(separator) ){
                collect(new Tuple2<>(s, s.length()));
            }
        }
    }	


// 使用
// 4. 自定义表函数，实现将id拆分，并输出（word, length）
        // 4.1 table API
        Split split = new Split("_");

        // 需要在环境中注册UDF
        tableEnv.registerFunction("split", split);
        Table resultTable = sensorTable
                .joinLateral("split(id) as (word, length)")
                .select("id, ts, word, length");

        // 4.2 SQL
        tableEnv.createTemporaryView("sensor", sensorTable);
        Table resultSqlTable = tableEnv.sqlQuery("select id, ts, word, length " +
                " from sensor, lateral table(split(id)) as splitid(word, length)");

```

#####  聚合函数（Aggregate Functions）

> **AggregateFunction 的工作原理如下：**
>
> 首先，它需要一个累加器（Accumulator），用来保存聚合中间结果的数据结构；可以通过调用 createAccumulator() 方法创建空累加器
> 随后，对每个输入行调用函数的 accumulate() 方法来更新累加器
> 处理完所有行后，将调用函数的 getValue() 方法来计算并返回最终结果



```java
   // 实现自定义的AggregateFunction
    public static class AvgTemp extends AggregateFunction<Double, Tuple2<Double, Integer>>{
        @Override
        public Double getValue(Tuple2<Double, Integer> accumulator) {
            return accumulator.f0 / accumulator.f1;
        }

        @Override
        public Tuple2<Double, Integer> createAccumulator() {
            return new Tuple2<>(0.0, 0);
        }

        // 必须实现一个accumulate方法，来数据之后更新状态
        public void accumulate( Tuple2<Double, Integer> accumulator, Double temp ){
            accumulator.f0 += temp;
            accumulator.f1 += 1;
        }
    }


// 使用
        // 4. 自定义聚合函数，求当前传感器的平均温度值
        // 4.1 table API
        AvgTemp avgTemp = new AvgTemp();

        // 需要在环境中注册UDF
        tableEnv.registerFunction("avgTemp", avgTemp);
        Table resultTable = sensorTable
                .groupBy("id")
                .aggregate( "avgTemp(temp) as avgtemp" )
                .select("id, avgtemp");

```

##### 表聚合函数（Table Aggregate Functions）

>**TableAggregateFunction 的工作原理如下:**
>– 首先，它同样需要一个累加器（Accumulator），它是保存聚合中间结果的数据结构。通过调用 createAccumulator() 方法可以创建空累加器。
>– 随后，对每个输入行调用函数的 accumulate() 方法来更新累加器。
>– 处理完所有行后，将调用函数的 emitValue() 方法来计算并返回最终结果。