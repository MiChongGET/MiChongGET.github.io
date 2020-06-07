---
title: 'BigData--MapReduce进阶(一)之框架原理'
date: 2020-05-26 17:44:22
tags:
- Hadoop
- MapReduce
categories:
- BigData
description: MapReduce是一个分布式运算程序的编程框架，是用户开发“基于Hadoop的数据分析应用”的核心框架。
top_img: https://ae01.alicdn.com/kf/Hb56005a7327e4b878181a92c494b4983y.jpg
cover: https://ae01.alicdn.com/kf/H675a31a08d4b4015ab2bd5886ce81979C.jpg
---
## MapReduce进阶(一)--框架原理

#### 1、InputFormat

**MapReduce数据流**![MapReduce数据流](https://file.buildworld.cn/img/20200525102001.png)

#### 2、MapTask并行度决定机制

**数据块：**Block是HDFS物理上把数据分成一块一块。

**数据切片：**数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。

- 1）一个Job的Map阶段并行度由客户端在提交Job时的切片数决定
- 2）每一个Split切片分配一个MapTask并行实例处理
- 3）默认情况下，切片大小=BlockSize
- 4）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片                              

#### 3、Job提交流程源码解析

![](https://file.buildworld.cn/img/20200525144602.png)



#### 4、FileInputFormat切片源码解析(input.getSplits(job))

##### 1）源码解析

![](https://file.buildworld.cn/img/20200525145028.png)   

##### 2）切片机制

- （1）简单地按照文件的内容长度进行切片
- （2）切片大小，默认等于Block大小
- （3）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

##### 3）切片大小的参数配置

![](https://file.buildworld.cn/img/20200525152428.png)



#### 5、小文件切片--CombineTextInputFormat切片机制

**生成切片过程包括：虚拟存储过程和切片过程二部分。**

![](https://file.buildworld.cn/img/20200525152901.png)

##### （1）虚拟存储过程：

> 将输入目录下所有文件大小，依次和设置的`setMaxInputSplitSize`值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值2倍，此时将文件均分成2个虚拟存储块（防止出现太小切片）。

##### （2）切片过程：

```
（a）判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。

（b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。

（c）测试举例：有4个小文件大小分别为1.7M、5.1M、3.4M以及6.8M这四个小文件，则虚拟存储之后形成6个文件块，大小分别为：1.7M，（2.55M、2.55M），3.4M以及（3.4M、3.4M）
	最终会形成3个切片，大小分别为：（1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M
```

#### 6、自定义InputFormat

![](https://file.buildworld.cn/img/20200526173913.png)

##### 1) WholeFileInputFormat 继承FileInputFormat

```java
package cn.buildworld.mapreduce.inputformat;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-05-25 16:12
 */
public class WholeFileInputFormat extends FileInputFormat<Text, BytesWritable> {

    @Override
    protected boolean isSplitable(JobContext context, Path filename) {
        return false;
    }

    @Override
    public RecordReader<Text, BytesWritable> createRecordReader(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
        return new WholeFileRecordReader();
    }
}
```



##### 2）自定义RecordReader--WholeFileRecordReader

```java
package cn.buildworld.mapreduce.inputformat;

import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.FileSplit;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

import java.io.IOException;

/**
 * @author MiChong
 * @date 2020-05-25 16:15
 * <p>
 * 自定义RecordReader，处理一个文件，把这个文件直接读成 一个KV值
 */
public class WholeFileRecordReader extends RecordReader<Text, BytesWritable> {

    private boolean notRead = true;
    private Text key = new Text();
    private BytesWritable value = new BytesWritable();
    private FSDataInputStream inputStream;
    private Path path;
    private FileSplit fs;

    /**
     * 初始化方法，框架会在开始的时候调用一次
     *
     * @param split
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
        //转换切片类型到文件切片
        fs = (FileSplit) split;
        //通过切片获取路径
        path = fs.getPath();
        //通过路径获取文件系统
        FileSystem fileSystem = path.getFileSystem(context.getConfiguration());
        //开流
        inputStream = fileSystem.open(path);
    }

    /**
     * 读取下一组KV值
     *
     * @return 如果读到了，返回true，读完了，返回False
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {

        if (notRead) {
            // 具体读文件的过程
            //读key
            key.set(fs.getPath().toString());

            //读value
            byte[] buf = new byte[(int) fs.getLength()];
            inputStream.read(buf);
            value.set(buf, 0, buf.length);

            notRead = false;
            return true;
        } else {

            return false;
        }
    }

    /**
     * 获取到当前的key
     *
     * @return 当前的key
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public Text getCurrentKey() throws IOException, InterruptedException {
        return key;
    }

    /**
     * 获取当前读到的Value
     *
     * @return 当前Value
     * @throws IOException
     * @throws InterruptedException
     */

    @Override
    public BytesWritable getCurrentValue() throws IOException, InterruptedException {
        return value;
    }

    /**
     * 当前数据读取的进度
     *
     * @return 当前进度
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public float getProgress() throws IOException, InterruptedException {
        return notRead ? 0 : 1;
    }

    /**
     * 关闭资源
     *
     * @throws IOException
     */
    @Override
    public void close() throws IOException {
        IOUtils.closeStream(inputStream);
    }
}
```