---
title: BigData--大数据技术之SparkStreaming
date: 2020-09-07 10:31:19
tags:
- Spark
- SparkStreaming
categories:
- BigData
description: Spark Streaming用于流式数据的处理。Spark Streaming支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ和简单的TCP套接字等等。
top_img: https://file.buildworld.cn/img/20200907215749.jpg
cover: https://file.buildworld.cn/img/20200907103306.png
---

> Spark Streaming用于流式数据的处理。Spark Streaming支持的数据输入源很多，例如：Kafka、Flume、Twitter、ZeroMQ和简单的TCP套接字等等。数据输入后可以用Spark的高度抽象原语如：map、reduce、join、window等进行运算。而结果也能保存在很多地方，如HDFS，数据库等。



![](https://file.buildworld.cn/img/20200906163217.png)

### 1、SparkStreaming架构

![SparkStreaming架构](https://file.buildworld.cn/img/20200907215002.png)



#### 依赖（采用scala 2.12.x版本）

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.12</artifactId>
    <version>2.4.6</version>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.12</artifactId>
    <version>2.4.6</version>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.12</artifactId>
    <version>2.4.6</version>
</dependency>

<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-10_2.12</artifactId>
    <version>2.4.6</version>
</dependency>

<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.11.0.0</version>
</dependency>

```

### 2、WordCount案例实操

```scala
package cn.buildworld.spark.streaming

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

object WordCount {

  def main(args: Array[String]): Unit = {

    // 使用SparkStreaming完成WordCount

    // Spark配置对象
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

    //实时数据分析环境对象
    //采集周期：以指定的时间为周期采集实时数据
    val streamingContext = new StreamingContext(sparkConf, Seconds(3))

    //从指定的端口中采集数据
    val socketLineDStream = streamingContext.socketTextStream("hadoop102", 9999)

    //将采集的书籍进行分解，扁平化
    val wordDStream = socketLineDStream.flatMap(line => line.split(" "))

    //将数据进行结构的转换，方便统计分析
    val mapDStream = wordDStream.map((_, 1))

    //将转换结构之后的数据进行聚合处理
    val wordToSumDStream = mapDStream.reduceByKey(_ + _)

    //将结果打印出来
    wordToSumDStream.print()

    //启动采集器
    streamingContext.start()

    //Driver等待采集器的执行
    streamingContext.awaitTermination()

  }
}
```

### 3、自定义数据源

> 除了可以从socket中读取数据，我们还可以从mysql中读取数据，具体看自己的业务需求

#### 1）声明采集器

```scala
// 声明采集器
// 1) 继承Receiver
// 2) 重写onStart,onStop方法
class MyReceiver(host: String, port: Int) extends Receiver[String](StorageLevel.MEMORY_ONLY) {

  var socket: Socket = null

  def receive(): Unit = {
    socket = new Socket(host, port)
    val reader = new BufferedReader(new InputStreamReader(socket.getInputStream, "UTF-8"))

    var line: String = null

    while ((line = reader.readLine()) != null) {
      //将采集的数据存储到采集器的内部进行转换
      this.store(line)
    }
  }

  override def onStart(): Unit = {
    new Thread(new Runnable {
      override def run(): Unit = {
        receive()
      }
    }).start()
  }

  override def onStop(): Unit = {
    socket.close()
    socket = null
  }
}
```

#### 2）使用自定义的采集器

```scala
//从指定的端口中采集数据
val receiverStream = streamingContext.receiverStream(new MyReceiver("hadoop102", 9999))
```

### 4、Kafka数据源（版本kafka0.11.x）

> 两个版本的代码不太一样：
>
> spark官网kafka0.10版本样例：http://spark.apache.org/docs/2.3.0/streaming-kafka-0-10-integration.html
>
> spark官网kafka0.8.x版本样例：http://spark.apache.org/docs/2.3.0/streaming-kafka-0-8-integration.html

**以下代码在kafka0.11.x上面的运行的**

#### 模拟产生数据（通过代码自动生成消息）

```scala
/**
 * 产生数据
 */
object KafkaWordProducer {
  def main(args: Array[String]): Unit = {
    // kafka服务器地址
    val brokers = "hadoop102:9092"
    // kafka topic
    val topic = "michong"
    // 每秒发送三次消息
    val messagesPerSec = 1
    // 每次发送五个数字
    val wordsPerMessage = 5
    val props = new util.HashMap[String, Object]()
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, brokers)
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer")
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer")
    val producer = new KafkaProducer[String, String](props)
    while (true){
      (1 to messagesPerSec.toInt).foreach{
        messageNum => val str = (1 to wordsPerMessage.toInt).map(x => Random.nextInt(30).toString).mkString(" ")
          println(str)
          val message = new ProducerRecord[String, String](topic, null, str)
          producer.send(message)
      }
      Thread.sleep(1000)
    }
  }
```

#### 控制台生成Topic以及消息

```shell
# 生成一个名字叫michong 的topic
bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 2 --partitions 3 --topic michong

# 进入kafka控制台手动输入消息
bin/kafka-console-producer.sh --broker-list hadoop102:9092 --topic michong
```



#### Wordcount词频统计

```scala
al sparkConf = new SparkConf().setAppName("kafkaDirectWordCount").setMaster("local[*]")
  val ssc = new StreamingContext(sparkConf, Seconds(5))

  val KafkaTopic = List("michong")
  val kafkaParams = Map[String, Object](
    "bootstrap.servers" -> "hadoop102:9092",
    "key.deserializer" -> classOf[StringDeserializer],
    "value.deserializer" -> classOf[StringDeserializer],
    "group.id" -> "michong",
    "auto.offset.reset" -> "latest",
    "enable.auto.commit" -> (false: java.lang.Boolean)
  )

  // TODO ... 消费数据
  // messages 是全部的数据
 /**
     * 指定kafka数据源
     * ssc：StreamingContext的实例
     * LocationStrategies：位置策略，如果kafka的broker节点跟Executor在同一台机器上给一种策略，不在一台机器上给另外一种策略
     * 设定策略后会以最优的策略进行获取数据
     * 一般在企业中kafka节点跟Executor不会放到一台机器的，原因是kakfa是消息存储的，Executor用来做消息的计算，
     * 因此计算与存储分开，存储对磁盘要求高，计算对内存、CPU要求高
     * 如果Executor节点跟Broker节点在一起的话使用PreferBrokers策略，如果不在一起的话使用PreferConsistent策略
     * 使用PreferConsistent策略的话，将来在kafka中拉取了数据以后尽量将数据分散到所有的Executor上
     * ConsumerStrategies：消费者策略（指定如何消费）
     *
     */
  val messages = KafkaUtils.createDirectStream(ssc,
    LocationStrategies.PreferConsistent,
    ConsumerStrategies.Subscribe[String, String](KafkaTopic, kafkaParams))
  // x.value获得数据里的值
  messages.map(x => x.value())
    .flatMap(_.split(" "))
    .map(x => (x, 1))
    .reduceByKey(_ + _)
    .print()
  ssc.start()
  ssc.awaitTermination()
}
```

#### 统计结果如下

![](https://file.buildworld.cn/img/20200907202538.png)

### 5、DStream转换

#### 无状态转化操作

**上面的Wordcount词频统计代码就是使用的无状态转化操作。**

> 无状态转化操作就是把简单的RDD转化操作应用到每个批次上，也就是转化DStream中的每一个RDD。部分无状态转化操作列在了下表中。注意，针对键值对的DStream转化操作(比如 reduceByKey())要添加`import StreamingContext._`才能在Scala中使用。
>
> * map(func) ：对源DStream的每个元素，采用func函数进行转换，得到一个新的DStream；
> * flatMap(func)： 与map相似，但是每个输入项可用被映射为0个或者多个输出项；
> * filter(func)： 返回一个新的DStream，仅包含源DStream中满足函数func的项；
> * repartition(numPartitions)： 通过创建更多或者更少的分区改变DStream的并行程度；
> * union(otherStream)： 返回一个新的DStream，包含源DStream和其他DStream的元素；
> * count()：统计源DStream中每个RDD的元素数量；
> * reduce(func)：利用函数func聚集源DStream中每个RDD的元素，返回一个包含单元素RDDs的新DStream；
> * countByValue()：应用于元素类型为K的DStream上，返回一个（K，V）键值对类型的新DStream，每个键的值是在原DStream的每个RDD中的出现次数；
> * reduceByKey(func, [numTasks])：当在一个由(K,V)键值对组成的DStream上执行该操作时，返回一个新的由(K,V)键值对组成的DStream，每一个key的值均由给定的recuce函数（func）聚集起来；
> * join(otherStream, [numTasks])：当应用于两个DStream（一个包含（K,V）键值对,一个包含(K,W)键值对），返回一个包含(K, (V, W))键值对的新DStream；
> * cogroup(otherStream, [numTasks])：当应用于两个DStream（一个包含（K,V）键值对,一个包含(K,W)键值对），返回一个包含(K, Seq[V], Seq[W])的元组；
> * transform(func)：通过对源DStream的每个RDD应用RDD-to-RDD函数，创建一个新DStream。支持在新的DStream中做任何RDD操作。

![](https://file.buildworld.cn/img/20200907210350.png)

#### 有状态转化操作

**将历史数据也拿过来分析**

##### 追踪状态变化(updateStateByKey)的转换

> `UpdateStateByKey`原语用于记录历史记录，有时，我们需要在 DStream 中跨批次维护状态(例如流计算中累加wordcount)。针对这种情况，`updateStateByKey()` 为我们提供了对一个状态变量的访问，用于键值对形式的 DStream。给定一个由(键，事件)对构成的 DStream，并传递一个指定如何根据新的事件 更新每个键对应状态的函数，它可以构建出一个新的 DStream，其内部数据为(键，状态) 对。 
>
> `updateStateByKey()` 的结果会是一个新的 DStream，其内部的 RDD 序列是由每个时间区间对应的(键，状态)对组成的。

```scala
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010._
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{HashPartitioner, SparkConf}


object KafkaDirectStream {
   
  // 更新方法
  val updateFunc = (it: Iterator[(String, Seq[Int], Option[Int])]) => {
    it.map { case (w, s, o) => (w, s.sum + o.getOrElse(0)) }
  }

  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setAppName("KafkaDirectStream").setMaster("local[*]")
    val ssc: StreamingContext = new StreamingContext(conf, Seconds(5))
    ssc.checkpoint("./ck")
    //跟Kafka整合（直连方式，调用Kafka底层的消费数据的API）
    val brokerList = "hadoop102:9092"

    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> brokerList,
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "michong",
      //这个代表，任务启动之前产生的数据也要读
      "auto.offset.reset" -> "earliest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    val topics = Array("michong")
    /**
     * 指定kafka数据源
     * ssc：StreamingContext的实例
     * LocationStrategies：位置策略，如果kafka的broker节点跟Executor在同一台机器上给一种策略，不在一台机器上给另外一种策略
     * 设定策略后会以最优的策略进行获取数据
     * 一般在企业中kafka节点跟Executor不会放到一台机器的，原因是kakfa是消息存储的，Executor用来做消息的计算，
     * 因此计算与存储分开，存储对磁盘要求高，计算对内存、CPU要求高
     * 如果Executor节点跟Broker节点在一起的话使用PreferBrokers策略，如果不在一起的话使用PreferConsistent策略
     * 使用PreferConsistent策略的话，将来在kafka中拉取了数据以后尽量将数据分散到所有的Executor上
     * ConsumerStrategies：消费者策略（指定如何消费）
     *
     */
    val directStream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream(ssc,
      LocationStrategies.PreferConsistent,
      ConsumerStrategies.Subscribe[String, String](topics, kafkaParams)
    )

    //打印出词频统计结果
    val result: DStream[(String, Int)] = directStream.map(_.value()).flatMap(_.split(" "))
      .map((_, 1))
      .updateStateByKey(updateFunc, new HashPartitioner(ssc.sparkContext.defaultParallelism), true)
    result.print()

    directStream.foreachRDD(rdd => {
      val offsetRange: Array[OffsetRange] = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      val maped: RDD[(String, String)] = rdd.map(record => (record.key, record.value))
      //计算逻辑
      maped.foreach(println)
      //循环输出
      println("************"+System.currentTimeMillis()+"************")
      for (o <- offsetRange) {
        println(s"${o.topic} ${o.partition} ${o.fromOffset} ${o.untilOffset}")
      }
    })

    ssc.start()
    ssc.awaitTermination()
  }
}
```

#####  Window Operations

> Window Operations可以设置窗口的大小和滑动窗口的间隔来动态的获取当前Steaming的允许状态。基于窗口的操作会在一个比 StreamingContext 的批次间隔更长的时间范围内，通过整合多个批次的结果，计算出整个窗口的结果。

![](https://file.buildworld.cn/img/20200908094728.png)

**所有基于窗口的操作都需要两个参数，分别为窗口时长以及滑动步长，两者都必须是 StreamContext 的批次间隔的整数倍。**

###### 关于Window的操作有如下原语：

> - （1）`window(windowLength, slideInterval):` 基于对源DStream窗化的批次进行计算返回一个新的Dstream
> - （2）`countByWindow(windowLength, slideInterval)：`返回一个滑动窗口计数流中的元素。
> - （3）`reduceByWindow(func, windowLength, slideInterval)：`通过使用自定义函数整合滑动区间流元素来创建一个新的单元素流。
> - （4）`reduceByKeyAndWindow(func, windowLength, slideInterval, [numTasks])：`当在一个(K,V)对的DStream上调用此函数，会返回一个新(K,V)对的DStream，此处通过对滑动窗口中批次数据使用reduce函数来整合每个key的value值。Note:默认情况下，这个操作使用Spark的默认数量并行任务(本地是2)，在集群模式中依据配置属性(spark.default.parallelism)来做grouping。你可以通过设置可选参数numTasks来设置不同数量的tasks。
> - （5）`reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, [numTasks])：`这个函数是上述函数的更高效版本，每个窗口的reduce值都是通过用前一个窗的reduce值来递增计算。通过reduce进入到滑动窗口数据并”反向reduce”离开窗口的旧数据来实现这个操作。一个例子是随着窗口滑动对keys的“加”“减”计数。通过前边介绍可以想到，这个函数只适用于”可逆的reduce函数”，也就是这些reduce函数有相应的”反reduce”函数(以参数invFunc形式传入)。如前述函数，reduce任务的数量通过可选参数来配置。注意：为了使用这个操作，[检查点](#checkpointing)必须可用。 
> - （6）`countByValueAndWindow(windowLength,slideInterval, [numTasks])：`对(K,V)对的DStream调用，返回(K,Long)对的新DStream，其中每个key的值是其在滑动窗口中频率。如上，可配置reduce任务数量。
>
> `reduceByWindow()` 和 `reduceByKeyAndWindow()` 让我们可以对每个窗口更高效地进行归约操作。它们接收一个归约函数，在整个窗口上执行，比如 +。除此以外，它们还有一种特殊形式，通过只考虑新进入窗口的数据和离开窗口的数据，让 Spark 增量计算归约结果。这种特殊形式需要提供归约函数的一个逆函数，比 如 + 对应的逆函数为 -。对于较大的窗口，提供逆函数可以大大提高执行效率  

```scala
//窗口大小应该为采集周期的整数倍，窗口滑动的步长也应该为采集周期的整数倍
val windowDStream: DStream[ConsumerRecord[String, String]] = directStream.window(Seconds(10), Seconds(5))
```

### 6、DStream输出

> 输出操作指定了对流数据经转化操作得到的数据所要执行的操作(例如把结果推入外部数据库或输出到屏幕上)。与RDD中的惰性求值类似，如果一个DStream及其派生出的DStream都没有被执行输出操作，那么这些DStream就都不会被求值。如果StreamingContext中没有设定输出操作，整个context就都不会启动。 
>
> **输出操作如下：**
>
> - （1）`print()：`在运行流程序的驱动结点上打印DStream中每一批次数据的最开始10个元素。这用于开发和调试。在Python API中，同样的操作叫print()。
>
> - （2）`saveAsTextFiles(prefix, [suffix])：`以text文件形式存储这个DStream的内容。每一批次的存储文件名基于参数中的prefix和suffix。”prefix-Time_IN_MS[.suffix]”. 
>
> - （3）`saveAsObjectFiles(prefix, [suffix])：`以Java对象序列化的方式将Stream中的数据保存为 SequenceFiles . 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". Python中目前不可用。
>
> - （4）`saveAsHadoopFiles(prefix, [suffix])：`将Stream中的数据保存为 Hadoop files. 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]"。
>   Python API Python中目前不可用。
>
> - （5）`foreachRDD(func)：`这是最通用的输出操作，即将函数 func 用于产生于 stream的每一个RDD。其中 参数传入的函数func应该实现将每一个RDD中数据推送到外部系统，如将RDD存入文件或者通过网络将其写入数据库。注意：函数func在运行流应用的驱动中被执行，同时其中一般函数RDD操作从而强制其对于流RDD的运算。
>
>   ```scala
>   windowDStream.foreachRDD(rdd => {
>     val offsetRange: Array[OffsetRange] = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
>     val maped: RDD[(String, String)] = rdd.map(record => (record.key, record.value))
>     //计算逻辑
>     maped.foreach(println)
>   
>     //循环输出
>     println("************"+System.currentTimeMillis()+"************")
>     for (o <- offsetRange) {
>       println(s"${o.topic} ${o.partition} ${o.fromOffset} ${o.untilOffset}")
>     }
>   })
>   ```
>
> 
>
> 通用的输出操作foreachRDD()，它用来对DStream中的RDD运行任意计算。这和transform() 有些类似，都可以让我们访问任意RDD。在foreachRDD()中，可以重用我们在Spark中实现的所有行动操作。
>
> 比如，常见的用例之一是把数据写到诸如MySQL的外部数据库中。 注意：
>
> - （1）连接不能写在driver层面；
> - （2）如果写在foreach则每个RDD都创建，得不偿失；
> - （3）增加foreachPartition，在分区创建。
