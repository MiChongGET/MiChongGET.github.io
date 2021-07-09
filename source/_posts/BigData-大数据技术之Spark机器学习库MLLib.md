---
title: BigData--大数据技术之Spark机器学习库MLLib
date: 2020-09-09 11:58:07
tags:
- Spark
- MLlib
categories:
- BigData
description: MLlib is Apache Spark's scalable machine learning library.
top_img: https://file.buildworld.cn/img/20200909195914.png
cover: https://file.buildworld.cn/img/20200909195630.png
comments: false
---
>MLlib fits into Spark's APIs and interoperates with NumPy in Python (as of Spark 0.9) and R libraries (as of Spark 1.5). You can use any Hadoop data source (e.g. HDFS, HBase, or local files), making it easy to plug into Hadoop workflows.

### 1、Spark MLib介绍

> - MLlib 是 Spark 的机器学习库，旨在简化机器学习的工程实践工作，并方便扩展到更大规模。
> - MLlib 由一些通用的学习算法和工具组成，包括分类、回归、聚类、协同过滤、降维等，同时还包括底层的优化原语和高层的管道 API。

| 名称           | 说明                                           |
| -------------- | ---------------------------------------------- |
| 数据类型       | 向量、带类别的向量、矩阵等                     |
| 数学统计计算库 | 基本统计量、相关分析、随机数产生器、假设检验等 |
| 算法评测       | AUC、准确率、召回率、F-Measure 等              |
| 机器学习算法   | 分类算法、回归算法、聚类算法、协同过滤等       |

> **Spark 机器学习库从 1.2 版本以后被分为两个包：**
>
> - `spark.mllib `包含基于RDD的原始算法API。Spark MLlib 历史比较长，在1.0 以前的版本即已经包含了，提供的算法实现都是基于原始的 RDD。
> - [`spark.ml`](http://spark.apache.org/docs/latest/ml-guide.html) 则提供了基于[DataFrames](http://spark.apache.org/docs/latest/sql-programming-guide.html#dataframes) 高层次的API，可以用来构建机器学习工作流（`PipeLine`）。`ML Pipeline` 弥补了原始 MLlib 库的不足，向用户提供了一个基于 DataFrame 的机器学习工作流式 API 套件。

	#### 目前MLlib支持的主要的机器学习算法

![](https://file.buildworld.cn/img/20200909141502.png)

### 2、使用

```xml
<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-mllib -->
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-mllib_2.12</artifactId>
    <version>2.4.6</version>
</dependency>
```

> - `DataFrame`：使用Spark SQL中的DataFrame作为数据集，它可以容纳各种数据类型。 较之 RDD，包含了 schema 信息，更类似传统数据库中的二维表格。它被 ML Pipeline 用来存储源数据。例如，DataFrame中的列可以是存储的文本，特征向量，真实标签和预测的标签等。
> - `Transformer`：翻译成转换器，是一种可以将一个DataFrame转换为另一个DataFrame的算法。比如一个模型就是一个 Transformer。它可以把 一个不包含预测标签的测试数据集 DataFrame 打上标签，转化成另一个包含预测标签的 DataFrame。技术上，Transformer实现了一个方法transform（），它通过附加一个或多个列将一个DataFrame转换为另一个DataFrame。
> - `Estimator`：翻译成估计器或评估器，它是学习算法或在训练数据上的训练方法的概念抽象。在 Pipeline 里通常是被用来操作 DataFrame 数据并生产一个 Transformer。从技术上讲，Estimator实现了一个方法fit（），它接受一个DataFrame并产生一个转换器。如一个随机森林算法就是一个 Estimator，它可以调用fit（），通过训练特征数据而得到一个随机森林模型。
> - `Paramete`r：Parameter 被用来设置 Transformer 或者 Estimator 的参数。现在，所有转换器和估计器可共享用于指定参数的公共API。ParamMap是一组（参数，值）对。
> - `PipeLine`：翻译为工作流或者管道。工作流将多个工作流阶段（转换器和估计器）连接在一起，形成机器学习的工作流，并获得结果输出。

```scala 
package cn.buildworld.spark.ml

import org.apache.spark
import org.apache.spark.ml.{Pipeline, PipelineModel}
import org.apache.spark.ml.classification.LogisticRegression
import org.apache.spark.ml.feature.{HashingTF, Tokenizer}
import org.apache.spark.sql.{Row, SparkSession}
import org.apache.spark.ml.linalg.Vector

object SparkMLIB_DEMO {
  def main(args: Array[String]): Unit = {
    val spark: SparkSession = SparkSession.builder().master("local").appName("MLib").getOrCreate()
    import spark.implicits._

    //引入要包含的包并构建训练数据集
    val training = spark.createDataFrame(Seq(
      (0L, "a b c d e spark", 1.0),
      (1L, "b d", 0.0),
      (2L, "spark f g h", 1.0),
      (3L, "hadoop mapreduce", 0.0)
    )).toDF("id", "text", "label")

    val tokenizer: Tokenizer = new Tokenizer()
      .setInputCol("text")
      .setOutputCol("words")

    val hashingTF: HashingTF = new HashingTF()
      .setNumFeatures(1000)
      .setInputCol(tokenizer.getInputCol)
      .setOutputCol("features")

    val lr: LogisticRegression = new LogisticRegression()
      .setMaxIter(10)
      .setRegParam(0.01)

    //照具体的处理逻辑有序的组织PipelineStages 并创建一个Pipeline
    val pipeline: Pipeline = new Pipeline().setStages(Array(tokenizer, hashingTF, lr))

    //现在构建的Pipeline本质上是一个Estimator，在它的fit（）方法运行之后，它将产生一个PipelineModel，它是一个Transformer。
    val model: PipelineModel = pipeline.fit(training)

    //构建测试数据
    val test = spark.createDataFrame(Seq(
      (4L, "spark i j k"),
      (5L, "l m n"),
      (6L, "spark a"),
      (7L, "apache hadoop")
    )).toDF("id", "text")

    //调用我们训练好的PipelineModel的transform（）方法，让测试数据按顺序通过拟合的工作流，生成我们所需要的预测结果。
    model.transform(test).select("id", "text", "probability", "prediction").collect().foreach {
      case Row(id: Long, text: String, prob: Vector, prediction: Double) =>
        println(s"($id, $text) --> prob=$prob, prediction=$prediction")
    }
  }
}
```



------

### 先挖个坑，后面有机会再来学习