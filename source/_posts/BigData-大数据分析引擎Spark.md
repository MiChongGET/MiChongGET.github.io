---
title: BigData--大数据分析引擎Spark
date: 2020-08-03 10:23:37
tags:
- Spark
categories:
- BigData
description: Apache Spark™ is a unified analytics engine for large-scale data processing.
top_img: https://file.buildworld.cn/img/573b864c042fecfea1a71d9845896376_t011159af9f862fc82e.jpg
cover: https://file.buildworld.cn/img/20200803194840.png
---
## 一、Spark运行

### 1、Spark内置模块

![](https://file.buildworld.cn/img/20200803102434.png)

> - `Spark Core`：实现了Spark的基本功能，包含任务调度、内存管理、错误恢复、与存储系统交互等模块。Spark Core中还包含了对弹性分布式数据集(Resilient Distributed DataSet，简称RDD)的API定义。 
> - `Spark SQL`：是Spark用来操作结构化数据的程序包。通过Spark SQL，我们可以使用 SQL或者Apache Hive版本的SQL方言(HQL)来查询数据。Spark SQL支持多种数据源，比如Hive表、Parquet以及JSON等。 
> - `Spark Streaming`：是Spark提供的对实时数据进行流式计算的组件。提供了用来操作数据流的API，并且与Spark Core中的 RDD API高度对应。 
> - `Spark MLlib`：提供常见的机器学习(ML)功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据 导入等额外的支持功能。 
> - `集群管理器`：Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计 算。为了实现这样的要求，同时获得最大灵活性，Spark支持在各种集群管理器(Cluster Manager)上运行，包括Hadoop YARN、Apache Mesos，以及Spark自带的一个简易调度 器，叫作**独立调度器**。

###  2、spark通用运行建议流程

![](https://file.buildworld.cn/img/20200726140437.png)

> - textFile("input")：读取本地文件input文件夹数据；
> - flatMap(_.split(" "))：压平操作，按照空格分割符将一行数据映射成一个个单词；
> - map((_,1))：对每一个元素操作，将单词映射为元组；
> - reduceByKey(_+_)：按照key将值进行聚合，相加；
> - collect：将数据收集到Driver端展示。

### 3、Spark和yarn联动

![](https://file.buildworld.cn/img/20200726174442.png)

## 二、RDD转换

### 1、 Value类型

#### 1）map(func)

> 返回一个新的RDD，该RDD由每一个输入元素经过func函数转换后组成

#### 2）mapPartitions(func)

> 类似于map，但独立地在RDD的每一个分片上运行，因此在类型为T的RDD上运行时，func的函数类型必须是Iterator[T] => Iterator[U]。假设有N个元素，有M个分区，那么map的函数的将被调用N次,而mapPartitions被调用M次,一个函数一次处理所有分区。

#### 3）mapPartitionsWithIndex(func)

> 类似于mapPartitions，但func带有一个整数参数表示分片的索引值，因此在类型为T的RDD上运行时，func的函数类型必须是(Int, Interator[T]) => Iterator[U]；

**map()和mapPartition()的区别**

> - map()：每次处理一条数据。
> - mapPartition()：每次处理一个分区的数据，这个分区的数据处理完后，原RDD中分区的数据才能释放，可能导致OOM。

#### 4）flatMap(func)

> 类似于map，但是每一个输入元素可以被映射为0或多个输出元素（所以func应该返回一个序列，而不是单一元素）

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD: RDD[List[Int]] = sc.makeRDD(Array(List(1, 2), List(3, 4)))

val flatMapRDD = listRDD.flatMap(datas => datas)

flatMapRDD.collect().foreach(println)
```

#### 5）glom

> 将每一个分区形成一个数组，形成新的RDD类型时RDD[Array[T]]

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD = sc.makeRDD(1 to 16, 4)

val flatMapRDD = listRDD.glom()

flatMapRDD.collect().foreach(array=>{
  println(array.mkString(","))
})
```

#### 6）groupBy(func)

> 分组，按照传入函数的返回值进行分组。将相同的key对应的值放入一个迭代器。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD = sc.makeRDD(1 to 16)

val groupByRDD = listRDD.groupBy(i => i % 2)

groupByRDD.collect().foreach(println)


//打印结果
(0,CompactBuffer(2, 4, 6, 8, 10, 12, 14, 16))
(1,CompactBuffer(1, 3, 5, 7, 9, 11, 13, 15))
```

#### 7）filter(func) 

> 过滤。返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD = sc.makeRDD(1 to 16)

val filterRDD = listRDD.filter(i => (i % 2) == 0)

filterRDD.collect().foreach(println)
```

#### 8）sample(withReplacement, fraction, seed)

> 以指定的随机种子随机抽样出数量为fraction的数据，withReplacement表示是抽出的数据是否放回，true为有放回的抽样，false为无放回的抽样，seed用于指定随机数生成器种子。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD = sc.makeRDD(1 to 16)

val sampleRDD = listRDD.sample(false, 0.4, 1)

sampleRDD.collect().foreach(println)
```

#### 9）distinct([numTasks]))

> 对源RDD进行去重后返回一个新的RDD。默认情况下，只有8个并行任务来操作，但是可以传入一个可选的numTasks参数改变它。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD = sc.makeRDD(1 to 16)

val distinctRDD = listRDD.distinct()

distinctRDD.collect().foreach(println)
```

#### 10）coalesce(numPartitions)

> 缩减分区数，用于大数据集过滤后，提高小数据集的执行效率。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD = sc.makeRDD(1 to 16, 4)

println("缩减分区前=" + listRDD.partitions.size)

val coalesceRDD = listRDD.coalesce(3)

println("缩减分区后=" + coalesceRDD.partitions.size)
```

#### 11） repartition(numPartitions)

> 根据分区数，重新通过网络随机洗牌所有数据。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val listRDD = sc.makeRDD(1 to 16, 4)

val repartitionRDD = listRDD.repartition(2)

repartitionRDD.collect().foreach(println)
```

**coalesce和repartition的区别**

- 1. coalesce重新分区，可以选择是否进行shuffle过程。由参数`shuffle: Boolean = false/true`决定。
- 2. repartition实际上是调用的coalesce，默认是进行shuffle的。源码如下：

```scala
def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] = withScope {
 coalesce(numPartitions, shuffle = true)
}
```

#### 12）sortBy(func,[ascending], [numTasks])

> 使用func先对数据进行处理，按照处理后的数据比较结果排序，默认为正序(从小到大)。

```scala
    val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

    val sc = new SparkContext(config)

    val listRDD = sc.makeRDD(1 to 16, 4)
    
    //安装除以2余数来排序
    val sortByRDD = listRDD.sortBy(x=>x%2)

    sortByRDD.collect().foreach(println)
```

#### 13）pipe(command, [envVars])

> 管道，针对每个分区，都执行一个shell脚本，返回输出的RDD。



### 2、双Value类型

#### 1）union(otherDataset) 

> 对源RDD和参数RDD求并集后返回一个新的RDD。



#### 2）subtract (otherDataset)

> 计算差的一种函数，去除两个RDD中相同的元素，不同的RDD将保留下来。

#### 3）intersection(otherDataset)

> 对源RDD和参数RDD求交集后返回一个新的RDD。

#### 4）cartesian(otherDataset)

> 笛卡尔积**（尽量避免使用）**

#### 5）zip(otherDataset)

> 将两个RDD组合成Key/Value形式的RDD,这里默认两个RDD的partition数量以及元素数量都相同，否则会抛出异常。

### 3、Key-Value类型

#### 1）partitionBy

> 对pairRDD进行分区操作，如果原有的partionRDD和现有的partionRDD是一致的话就不进行分区， 否则会生成ShuffleRDD，即会产生shuffle过程。

```scala
def main(args: Array[String]): Unit = {

  val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

  val sc = new SparkContext(config)

  val listRDD = sc.makeRDD(List(("a",1),("b",2),("c",3)))

  val partRDD = listRDD.partitionBy(new MyPartitioner(3))

  partRDD.saveAsTextFile("output")

}

//自定义
class MyPartitioner(partitions:Int) extends Partitioner{
  //设置分区数目
  override def numPartitions: Int = {
    partitions
  }

  //设置数据存放位置
  override def getPartition(key: Any): Int = {
    1
  }
}
```

#### 2）groupByKey

> groupByKey也是对每个key进行操作，但只生成一个`sequence`。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val words = Array("one", "two", "two", "three", "three", "three")

val wordPairsRDD = sc.parallelize(words).map(word => (word, 1))

val group = wordPairsRDD.groupByKey()

//将相同key对应值聚合到一个sequence中
group.collect().foreach(println)
//计算相同key对应值的相加结果
group.map(t=>(t._1,t._2.sum)).foreach(println)
```

#### 3）reduceByKey(func, [numTasks])

> 在一个(K,V)的RDD上调用，返回一个(K,V)的RDD，使用指定的reduce函数，将相同key的值聚合到一起，reduce任务的个数可以通过第二个可选的参数来设置。



**reduceByKey和groupByKey的区别**

> - 1. reduceByKey：按照key进行聚合，在shuffle之前有combine（预聚合）操作，返回结果是RDD[k,v].
> - 2. groupByKey：按照key进行分组，直接进行shuffle。
> - 3. 开发指导：reduceByKey比groupByKey快，建议使用。但是需要注意是否会影响业务逻辑。

![](https://file.buildworld.cn/img/20200803194634.png)

#### 4）aggregateByKey

> 参数：`(zeroValue:U,[partitioner: Partitioner]) (seqOp: (U, V) => U,combOp: (U, U) => U)`
>
> - 1. 作用：在kv对的RDD中，，按key将value进行分组合并，合并时，将每个value和初始值作为seq函数的参数，进行计算，返回的结果作为一个新的kv对，然后再将结果按照key进行合并，最后将每个分组的value传递给combine函数进行计算（先将前两个value进行计算，将返回结果和下一个value传给combine函数，以此类推），将key与计算结果作为一个新的kv对输出。
> - 2. 参数描述：
>
> （1）`zeroValue`：给每一个分区中的每一个key一个初始值；
>
> （2）`seqOp`：函数用于在每一个分区中用初始值逐步迭代value；
>
> （3）`combOp`：函数用于合并每个分区中的结果。

![](https://file.buildworld.cn/img/20200804105505.png)

**创建一个pairRDD，取出每个分区相同key对应值的最大值，然后相加**

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val rdd = sc.parallelize(List(("a", 3), ("a", 2), ("c", 4), ("b", 3), ("c", 6), ("c", 8)), 2)

rdd.glom().collect().foreach(println)

val value = rdd.aggregateByKey(0)(math.max(_, _), _ + _)

value.collect().foreach(println)
```

#### 5）foldByKey

> 参数：`(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]`

- 1. 作用：aggregateByKey的简化操作，seqop和combop相同
- 2. 需求：创建一个pairRDD，计算相同key对应值的相加结果

```scala
val agg = rdd.foldByKey(0)(_+_)
```

#### 6）combineByKey[C] 

> 参数：`(createCombiner: V => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C)` 

> - 1. 作用：对相同K，把V合并成一个集合。
>
> - 2. 参数描述：
>
>   （1）createCombiner: combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就和之前的某个元素的键相同。如果这是一个新的元素,combineByKey()会使用一个叫作createCombiner()的函数来创建那个键对应的累加器的初始值
>
>   
>
>   （2）mergeValue: 如果这是一个在处理当前分区之前已经遇到的键，它会使用mergeValue()方法将该键的累加器对应的当前值与这个新的值进行合并
>
>   
>
>   （3）mergeCombiners: 由于每个分区都是独立处理的， 因此对于同一个键可以有多个累加器。如果有两个或者更多的分区都有对应同一个键的累加器， 就需要使用用户提供的 mergeCombiners() 方法将各个分区的结果进行合并。

![](https://file.buildworld.cn/img/20200804152509.png)

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val input = sc.parallelize(Array(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98)),2)

val value = input.combineByKey(x => x, (x: Int, y: Int) => x + y, (x: Int, y: Int) => x + y)

value.collect().foreach(println)
```

#### 7）sortByKey([ascending], [numTasks])

> 作用：在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD

```scala
val input = sc.parallelize(Array(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98)),2)
val value1 = input.sortByKey(true)
```

#### 8）mapValues
 
> 针对于(K,V)形式的类型只对V进行操作

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")
val sc = new SparkContext(config)
val input = sc.parallelize(Array(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98)), 2)
val value1 = input.mapValues(_ + 100)
value1.collect().foreach(println)
```

#### 9）join(otherDataset, [numTasks])

> 在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD



####  10）cogroup(otherDataset, [numTasks])

> 在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的RDD


### 三、Action（行动算子）

#### 1）reduce(func)

> 通过func函数聚集RDD中的所有元素，先聚合分区内数据，再聚合分区间数据。

#### 2）collect()

> 在驱动程序中，以数组的形式返回数据集的所有元素。

#### 3） count()

> 返回RDD中元素的个数

#### 4）first()

> 返回RDD中的第一个元素

#### 5）take(n)

> 返回一个由RDD的前n个元素组成的数组

#### 6）takeOrdered(n)

> 返回该RDD排序后的前n个元素组成的数组

#### 7）aggregate(num)(func1)(func2)

> - 1. 参数：(zeroValue: U)(seqOp: (U, T) ⇒ U, combOp: (U, U) ⇒ U)
> - 2. 作用：aggregate函数将每个分区里面的元素通过seqOp和初始值进行聚合，然后用combine函数将每个分区的结果和初始值(zeroValue)进行combine操作。这个函数最终返回的类型不需要和RDD中元素类型一致。

#### 8）fold(num)(func)

> 作用：折叠操作，aggregate的简化操作，seqop和combop一样。

#### 9）saveAsTextFile(path)

> 将数据集的元素以textfile的形式保存到HDFS文件系统或者其他支持的文件系统，对于每个元素，Spark将会调用toString方法，将它装换为文件中的文本

#### 10）saveAsSequenceFile(path)

> 将数据集中的元素以Hadoop sequencefile的格式保存到指定的目录下，可以使HDFS或者其他Hadoop支持的文件系统。

#### 11）saveAsObjectFile(path)

> 用于将RDD中的元素序列化成对象，存储到文件中。

#### 12） countByKey()

> 针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数。

```scala
val rdd = sc.parallelize(List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8)),3)

val total = rdd.countByKey()

println(total)

//打印结果
Map(3 -> 2, 1 -> 3, 2 -> 1)
```

#### 13） foreach(func)

> 在数据集的每一个元素上，运行函数func进行更新。

### 四、RDD依赖关系

#### 1）Lineage

> RDD只支持粗粒度转换，即在大量记录上执行的单个操作。将创建RDD的一系列`Lineage`（血统）记录下来，以便恢复丢失的分区。RDD的`Lineage`会记录RDD的元数据信息和转换行为，当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。

#### 2）窄依赖

> 窄依赖指的是每一个父RDD的Partition最多被子RDD的一个Partition使用.

#### 3）宽依赖

> 宽依赖指的是多个子RDD的Partition会依赖同一个父RDD的Partition，会引起shuffle.

#### 4）DAG(Directed Acyclic Graph)

> DAG(Directed Acyclic Graph)叫做有向无环图，原始的RDD通过一系列的转换就就形成了DAG，根据RDD之间的依赖关系的不同将DAG划分成不同的Stage，对于窄依赖，partition的转换处理在Stage中完成计算。对于宽依赖，由于有Shuffle的存在，只能在parent RDD处理完成后，才能开始接下来的计算，因此**宽依赖是划分Stage的依据**。

![](https://file.buildworld.cn/img/20200807164413.png)

### 五、累加器

> 累加器用来对信息进行聚合，通常在向 Spark传递函数时，比如使用 map() 函数或者用 filter() 传条件时，可以使用驱动器程序中定义的变量，但是集群中运行的每个任务都会得到这些变量的一份新的副本，更新这些副本的值也不会影响驱动器中的对应变量。如果我们想实现所有分片处理时更新共享变量的功能，那么累加器可以实现我们想要的效果。

```scala
val config = new SparkConf().setMaster("local[*]").setAppName("WordCount")

val sc = new SparkContext(config)

val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4), 2)

//创建累加器对象
val accumulator: LongAccumulator = sc.longAccumulator

rdd.foreach {
  case i => {
    //执行累加器的累加功能
    accumulator.add(i)
  }
}
//获取累加器保存的值
println(accumulator.value)
```

#### 自定义累加器

> 自定义累加器类型的功能在1.X版本中就已经提供了，但是使用起来比较麻烦，在2.0版本后，累加器的易用性有了较大的改进，而且官方还提供了一个新的抽象类：`AccumulatorV2`来提供更加友好的自定义类型累加器的实现方式。实现自定义类型累加器需要继承`AccumulatorV2`并至少覆写下例中出现的方法。

### 六、广播变量（调优策略）

> 广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的**只读**值，以供一个或多个Spark操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，甚至是机器学习算法中的一个很大的特征向量，广播变量用起来都很顺手。 在多个并行操作中使用同一个变量，但是 Spark会为每个任务分别发送。

> 使用广播变量的过程如下：
>
> - (1) 通过对一个类型 T 的对象调用 SparkContext.broadcast 创建出一个 Broadcast[T] 对象。 任何可序列化的类型都可以这么实现。 
> - (2) 通过 value 属性访问该对象的值(在 Java 中为 value() 方法)。 
> - (3) 变量只会被发到各个节点一次，应作为只读值处理(修改这个值不会影响到别的节点)。