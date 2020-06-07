---
title: BigData--MapReduce进阶(二)之工作机制
date: 2020-06-01 15:01:19
tags:
- Hadoop
- MapReduce
- Shuffle
categories:
- BigData
description: 介绍了MapReduce工作流程，Shuffle机制，以及MapTask、ReduceTask工作机制等。
top_img: https://ae01.alicdn.com/kf/Hc6d06ee2bf1c493fab4ba83fe6fb0f54T.jpg
cover: https://ae01.alicdn.com/kf/H675a31a08d4b4015ab2bd5886ce81979C.jpg
---
## MapReduce进阶

### 一、MapReduce工作流程

#### 1、工作流程（1）

![](https://file.buildworld.cn/img/20200527100132.png)

#### 2、工作流程（2）

![](https://file.buildworld.cn/img/20200527100328.png)



**shuffle是从第七步开始的到第十六步结束的，下面是shuffle过程详解**

- 1）MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中
- 2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件
- 3）多个溢出文件会被合并成大的溢出文件
- 4）在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序
- 5）ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据
- 6）ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）
- 7）合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）



### 二、Shuffle机制

**Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。**

#### 1、Shuffle机制流程图

![](https://file.buildworld.cn/img/20200527140654.png)

#### 2、Partition分区

##### 1）自定义Partitioner步骤

- （1）自定义类继承`Partitioner`，重写`getPartition()`方法

```java
package cn.buildworld.mapreduce.partition;

import cn.buildworld.mapreduce.flow.FlowBean;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

/**
 * @author MiChong
 * @date 2020-05-27 14:40
 */
public class MyPartitioner extends Partitioner<Text, FlowBean> {
    @Override
    public int getPartition(Text text, FlowBean flowBean, int numPartitions) {
        String phone = text.toString();
        switch (phone.substring(0, 3)) {
            case "136":
                return 0;
            case "137":
                return 1;
            case "138":
                return 2;
            case "139":
                return 3;
            default:
                return 4;
        }
    }
}
```

- （2）在Job驱动中，设置自定义`Partitioner` 

```java
job.setPartitionerClass(MyPartitioner.class);
```

- （3）自定义`Partition`后，要根据自定义`Partitioner`的逻辑设置相应数量的`ReduceTask`

```java
job.setNumReduceTasks(5);
```

##### 2）分区总结

- （1）如果`ReduceTask`的数量> `getPartition`的结果数，则会多产生几个空的输出文件part-r-000xx；
- （2）如果1<`ReduceTask`的数量<`getPartition`的结果数，则有一部分分区数据无处安放，会Exception；
- （3）如果`ReduceTask`的数量=1，则不管MapTask端输出多少个分区文件，最终结果都交给这一个`ReduceTask`，最终也就只会产生一个结果文件 part-r-00000；
- （4）分区号必须从零开始，逐一累加。



#### 3、WritableComparable排序

**Bean对象实现`WritableComparable`几口，重写`compareTo（）`方法**

```java
package cn.buildworld.mapreduce.writablecomparable;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-05-24 18:15
 */

/**
 * 必须实现Writable接口
 */
public class FlowBean implements WritableComparable<FlowBean> {
    private long upFlow;
    private long downFlow;
    private long sumFlow;

    // 此处实现自定义排序
    @Override
    public int compareTo(FlowBean o) {
        return Long.compare(o.sumFlow, this.sumFlow);
    }
}
```

#### 4、Combiner合并

- （1）Combiner是MR程序中Mapper和Reducer之外的一种组件。

- （2）Combiner组件的父类就是Reducer。

- （3）Combiner和Reducer的区别在于运行的位置

  **Combiner是在每一个MapTask所在的节点运行;**

  **Reducer是接收全局所有Mapper的输出结果；**

- （4）Combiner的意义就是对每一个MapTask的输出进行局部汇总，以减小网络传输量。
- （5）Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟Reducer的输入kv类型要对应起来。

##### 实现步骤

**自定义一个Combiner继承Reducer，重写Reduce方法**

```java
public class WordcountCombiner extends Reducer<Text, IntWritable, Text,IntWritable>{
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {


 // 1 汇总操作
    int count = 0;
	for(IntWritable v :values){
		count += v.get();
	}
   // 2 写出
		context.write(key, new IntWritable(count));
	}
```

**在Job驱动类中设置**

```java
job.setCombinerClass(WordcountCombiner.class);
```

#### 5、GroupingComparator分组

- （1）自定义类继承`WritableComparator`
- （2）重写`compare()`方法
- （3）创建一个构造将比较对象的类传给父类

```java
package cn.buildworld.mapreduce.groupCompa;

import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

/**
 * @author MiChong
 * @date 2020-05-30 13:44
 */
public class OrderComparator extends WritableComparator {

    protected OrderComparator() {
        super(OrderBean.class, true);
    }

    @Override
    public int compare(WritableComparable a, WritableComparable b) {

        OrderBean oa = (OrderBean) a;
        OrderBean ob = (OrderBean) b;

        return oa.getOrderId().compareTo(ob.getOrderId());

    }
}
```

### 三、MapTask工作机制

![](https://file.buildworld.cn/img/20200530195503.png)

- （1）**Read阶段**：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
- （2）**Map阶段**：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
- （3）**Collect收集阶段**：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
- （4）**Spill阶段**：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

**溢写阶段详情：**

​	步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。

​	步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

​	步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。  



- （**5）Combine阶段：**当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

​	当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。

​	在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。

​	让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

### 四、ReduceTask工作机制

#### 1、工作机制

![](https://file.buildworld.cn/img/20200530204504.png)

- （1）**Copy阶段**：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
- （2）**Merge阶段：**在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。
- （3）**Sort阶段：**按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。
- （4）**Reduce阶段：**reduce()函数将计算结果写到HDFS上。



#### 2、设置ReduceTask并行度（个数）

> educeTask的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置：

```java
// 默认值是1，手动设置为4
job.setNumReduceTasks(4);
```

### 五、OutputFormat数据输出

#### 1、OutputFormat接口实现类

- 文本输出`TextOutputFormat`

 默认的输出格式是TextOutputFormat，它把每条记录写为文本行。它的键和值可以是任意类型，因为TextOutputFormat调用toString()方法把它们转换为字符串。

- `SequenceFileOutputFormat`

 将SequenceFileOutputFormat输出作为后续 MapReduce任务的输入，这便是一种好的输出格式，因为它的格式紧凑，很容易被压缩。

- 自定义`OutputFormat`

根据用户需求，自定义实现输出。

#### 2、自定义OutputFormat使用场景及步骤

- （1）自定义一个类继承FileOutputFormat。

- （2）改写RecordWriter，具体改写输出数据的方法write()。

```java
package cn.buildworld.mapreduce.outputformat;

import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-05-31 16:45
 */
public class MyRecordWriter extends RecordWriter<LongWritable, Text> {

    private FSDataOutputStream atguigu;
    private FSDataOutputStream other;

    /**
     * 初始化方法
     */
    public void initialize(TaskAttemptContext job) throws IOException {
        String dir = job.getConfiguration().get(FileOutputFormat.OUTDIR);
        FileSystem fileSystem = FileSystem.get(job.getConfiguration());
        atguigu = fileSystem.create(new Path(dir+"/my.log"));
        other = fileSystem.create(new Path(dir+"/others.log"));

    }

    /**
     * 将KV写出，每对KV调用一次
     *
     * @param key
     * @param value
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public void write(LongWritable key, Text value) throws IOException, InterruptedException {
        String out = value.toString() + "\n";
        if (out.contains("atguigu")) {
            atguigu.write(out.getBytes());
        } else {
            other.write(out.getBytes());
        }
    }

    @Override
    public void close(TaskAttemptContext context) throws IOException, InterruptedException {
        IOUtils.closeStream(atguigu);
        IOUtils.closeStream(other);
    }
}
```

### 六、Join应用

#### 1、Reduce Join

![](https://file.buildworld.cn/img/20200601111718.png)

#### 2、Map Join

- 1．使用场景

**Map Join适用于一张表十分小、一张表很大的场景。**

- 2．优点

思考：在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？

> 在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。

- 3．具体办法：采用DistributedCache

​	**（1）在Mapper的setup阶段，将文件读取到缓存集合中。**

​	**（2）在驱动函数中加载缓存。**

```java
import java.net.URI;
	
// 缓存普通文件到Task运行节点。
job.addCacheFile(new URI("file:///e:/cache/pd.txt"));
```

  

### 七、MapReduce开发总结

#### 1．输入数据接口：InputFormat

- （1）默认使用的实现类是：`TextInputFormat` 
- （2）`TextInputFormat`的功能逻辑是：一次读一行文本，然后将该行的起始偏移量作为key，行内容作为value返回。
- （3）`KeyValueTextInputFormat`每一行均为一条记录，被分隔符分割为key，value。默认分隔符是tab（\t）。
- （4）`NlineInputFormat`按照指定的行数N来划分切片。
- （5）`CombineTextInputFormat`可以把多个小文件合并成一个切片处理，提高处理效率。
- （6）用户还可以自定义`InputFormat`。

#### 2．逻辑处理接口：Mapper 

**用户根据业务需求实现其中三个方法：map()   setup()   cleanup ()** 

#### 3．Partitioner分区

- （1）有默认实现 HashPartitioner，逻辑是根据key的哈希值和numReduces来返回一个分区号；key.hashCode()&Integer.MAXVALUE % numReduces
- （2）如果业务上有特别的需求，可以自定义分区。

#### 4．Comparable排序

- （1）当我们用自定义的对象作为key来输出时，就必须要实现WritableComparable接口，重写其中的compareTo()方法。
- （2）部分排序：对最终输出的每一个文件进行内部排序。
- （3）全排序：对所有数据进行排序，通常只有一个Reduce。
- （4）二次排序：排序的条件有两个。

#### 5．Combiner合并

`Combiner`合并可以提高程序执行效率，减少IO传输。但是使用时必须不能影响原有的业务处理结果。

#### 6．Reduce端分组：GroupingComparator

在`Reduce`端对key进行分组。应用于：在接收的key为bean对象时，想让一个或几个字段相同（全部字段比较不相同）的key进入到同一个reduce方法时，可以采用分组排序。

#### 7．逻辑处理接口：Reducer

用户根据业务需求实现其中三个方法：`reduce()   setup()   cleanup ()` 

#### 8．输出数据接口：OutputFormat

- （1）默认实现类是TextOutputFormat，功能逻辑是：将每一个KV对，向目标文本文件输出一行。
- （2）将SequenceFileOutputFormat输出作为后续 MapReduce任务的输入，这便是一种好的输出格式，因为它的格式紧凑，很容易被压缩。
- （3）用户还可以自定义OutputFormat。