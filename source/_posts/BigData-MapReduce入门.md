---
title: BigData--MapReduce入门
date: 2020-05-24 21:51:03
tags:
- Hadoop
- MapReduce
categories:
- BigData
description: MapReduce是一个分布式运算程序的编程框架，是用户开发“基于Hadoop的数据分析应用”的核心框架。
top_img: https://ae01.alicdn.com/kf/H28c822f52f6f4fe2ad3d71f17d1f7a5bN.jpg
cover: https://ae01.alicdn.com/kf/H675a31a08d4b4015ab2bd5886ce81979C.jpg
---
## MapReduce入门

### 一、MapReduce概述

**MapReduce是一个分布式运算程序的编程框架，是用户开发“基于Hadoop的数据分析应用”的核心框架**。

**MapReduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个Hadoop集群上。**

#### 1、优点

- MapReduce易于编程
- 良好的扩展性
- 高容错性
- 适合海量数据的离线处理

#### 2、缺点

- 不擅长实时计算，无法像MySQL一样，在毫秒或者秒级内返回结果。
- 不擅长流式计算，MapReduce的输入数据是静态。
- 不擅长DAG(有向图)计算，如果每个MapReduce作业的输出结果都写入到磁盘，会造成大量的磁盘IO，导致性能非常的低下。

#### 3、MapReduce核心编程思想

![](https://file.buildworld.cn/img/20200524104738.png)

- 1）分布式的运算程序往往需要分成至少2个阶段。
- 2）第一个阶段的`MapTask`并发实例，完全并行运行，互不相干。
- 3）第二个阶段的`ReduceTask`并发实例互不相干，但是他们的数据依赖于上一个阶段的所有`MapTask`并发实例的输出。
- 4）MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行。

#### 4、MapReduce进程

**一个完整的MapReduce程序在分布式运行时有三类实例进程：**

- 1）`MrAppMaster`：负责整个程序的过程调度及状态协调。

- 2）`MapTask`：负责Map阶段的整个数据处理流程

- 3）`ReduceTask`：负责Reduce阶段的整个数据处理流程。

  

#### 5、MapReduce编程规范

##### 1) Mapper阶段

  ![](https://file.buildworld.cn/img/20200524111628.png)

##### 2）Reducer阶段

![](https://file.buildworld.cn/img/20200524111651.png)

##### 3）Driver阶段

**用于提交封装了MapReduce程序相关运行参数的job对象。**



### 二、WordCount案例实操

**主要实现的是对文件中单词出现频率的分析，统计出单词出现的次数，这也是官方的示例教程**

#### 1、WcMapper ，负责数据的切分

```java
package cn.buildworld.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-05-24 11:32
 */
public class WcMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    private Text word = new Text();
    private IntWritable one = new IntWritable(1);


    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 拿到这行数据
        String line = value.toString();
        //按照空格切分数据
        String[] words = line.split(" ");
        //遍历数组
        for (String word : words) {
            this.word.set(word);
            context.write(this.word, this.one);
        }

    }
}
```

#### 2、WcReducer,负责数据的统计

```java
package cn.buildworld.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-05-24 11:33
 */
public class WcReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    private IntWritable total = new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;

        //累加
        for (IntWritable value : values) {
            sum += value.get();
        }

        //包装结果并输出
        total.set(sum);
        context.write(key, this.total);
    }
}
```

#### 3、WcDriver，代码相对固定，负责提交我们的Job

```java
package cn.buildworld.mapreduce.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-05-24 11:33
 */
public class WcDriver {

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        // 1、获取一个job实例
        Job job = Job.getInstance(new Configuration());
        // 2、设置类路径
        job.setJarByClass(WcDriver.class);
        // 3、设置Mapper和Reducer
        job.setMapperClass(WcMapper.class);
        job.setReducerClass(WcReducer.class);

        // 4、设置Mapper和Reducer输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // 5 设置最终输出kv类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //6、设置输入输出数据
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 7、提交我们的job
        boolean b = job.waitForCompletion(true);
        System.exit(b ? 0 : 1);
    }
}
```

### 三、Hadoop序列化

**不可以使用Java自带的序列化，要使用自定义bean对象实现序列化接口（Writable）**

#### 示例代码

```java
package cn.buildworld.mapreduce.flow;

import org.apache.hadoop.io.Writable;

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
public class FlowBean implements Writable {
    private long upFlow;
    private long downFlow;
    private long sumFlow;

    /**
     * 反序列化时，需要反射调用空参构造函数，所以必须有空参构造
     */
    public FlowBean() {
    }

    public void set(long upFlow, long downFlow) {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.sumFlow = upFlow + downFlow;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(long downFlow) {
        this.downFlow = downFlow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }

    /**
     * 最后会按照下面的格式显示在输出文件中
     *
     * 要想把结果显示在文件中，需要重写toString()，可用”\t”分开，方便后续用。
     * @return
     */
    @Override
    public String toString() {
        return "FlowBean{" +
                "upFlow=" + upFlow +
                ", downFlow=" + downFlow +
                ", sumFlow=" + sumFlow +
                '}';
    }

    /**
     * 重写序列化方法
     *
     *注意反序列化的顺序和序列化的顺序完全一致
     *
     * @param out 框架给我们提供的数据出口
     * @throws IOException
     */
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(upFlow);
        out.writeLong(downFlow);
        out.writeLong(sumFlow);
    }

    /**
     * 重写反序列化方法
     *
     * 注意反序列化的顺序和序列化的顺序完全一致
     *
     * @param in 框架给我们提供的数据来源
     * @throws IOException
     */
    @Override
    public void readFields(DataInput in) throws IOException {
        upFlow = in.readLong();
        downFlow = in.readLong();
        sumFlow = in.readLong();
    }
}
```

