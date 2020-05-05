---
title: TensorFlow2.X学习笔记(5)--TensorFlow中阶API之数据管道
date: 2020-05-03 16:50:35
tags:
- Python
- TensorFlow
categories:
- 深度学习
description: 使用tf.data可以构建数据输入管道，轻松处理大量的数据，不同的数据格式，以及不同的数据转换。
top_img: https://ae01.alicdn.com/kf/H47b2b50232ee4d2fb4450bb71cdfeac7i.jpg
cover: https://ae01.alicdn.com/kf/H24c9452e63dd4cc08266332d7111b080Z.jpg
---
# TensorFlow的中阶API

## 【模型之墙】

TensorFlow的中阶API主要包括:

- 数据管道(tf.data)
- 特征列(tf.feature_column)
- 激活函数(tf.nn)
- 模型层(tf.keras.layers)
- 损失函数(tf.keras.losses)
- 评估函数(tf.keras.metrics)
- 优化器(tf.keras.optimizers)
- 回调函数(tf.keras.callbacks)

## 一、数据管道Dataset

> 使用 `tf.data` API 可以构建数据输入管道，轻松处理大量的数据，不同的数据格式，以及不同的数据转换。

> 使用`tfrecoreds`文件的优点是压缩后文件较小，便于网络传播，加载速度较快。

### 1、从Numpy array构建数据管道

```python
# 从Numpy array构建数据管道
import tensorflow as tf
import numpy as np 
from sklearn import datasets 
iris = datasets.load_iris()

ds1 = tf.data.Dataset.from_tensor_slices((iris["data"],iris["target"]))
for features,label in ds1.take(5):
    print(features,label)
```

### 2、从 Pandas DataFrame构建数据管道

```python
# 从 Pandas DataFrame构建数据管道
import tensorflow as tf
from sklearn import datasets 
import pandas as pd
iris = datasets.load_iris()
dfiris = pd.DataFrame(iris["data"],columns = iris.feature_names)
ds2 = tf.data.Dataset.from_tensor_slices((dfiris.to_dict("list"),iris["target"]))

for features,label in ds2.take(3):
    print(features,label)
```

###  3、从Python generator构建数据管道

```python
# 从Python generator构建数据管道
import tensorflow as tf
from matplotlib import pyplot as plt 
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# 定义一个从文件中读取图片的generator
image_generator = ImageDataGenerator(rescale=1.0/255).flow_from_directory(
                    "./data/cifar2/test/",
                    target_size=(32, 32),
                    batch_size=20,
                    class_mode='binary')

classdict = image_generator.class_indices
print(classdict)

def generator():
    for features,label in image_generator:
        yield (features,label)

ds3 = tf.data.Dataset.from_generator(generator,output_types=(tf.float32,tf.int32))
```

### 4、从csv文件构建数据管道

```python
# 从csv文件构建数据管道
ds4 = tf.data.experimental.make_csv_dataset(
      file_pattern = ["./data/titanic/train.csv","./data/titanic/test.csv"],
      batch_size=3, 
      label_name="Survived",
      na_value="",
      num_epochs=1,
      ignore_errors=True)

for data,label in ds4.take(2):
    print(data,label)
```

### 5、从文本文件构建数据管道

```python
# 从文本文件构建数据管道
ds5 = tf.data.TextLineDataset(
    filenames = ["./data/titanic/train.csv","./data/titanic/test.csv"]
    ).skip(1) #略去第一行header

for line in ds5.take(5):
    print(line)
```

### 6、从文件路径构建数据管道

```python
ds6 = tf.data.Dataset.list_files("./data/cifar2/train/*/*.jpg")
for file in ds6.take(5):
    print(file)
```

### 7、从tfrecords文件构建数据管道

```python
import os
import numpy as np

# inpath：原始数据路径 outpath:TFRecord文件输出路径
def create_tfrecords(inpath,outpath): 
    writer = tf.io.TFRecordWriter(outpath)
    dirs = os.listdir(inpath)
    for index, name in enumerate(dirs):
        class_path = inpath +"/"+ name+"/"
        for img_name in os.listdir(class_path):
            img_path = class_path + img_name
            img = tf.io.read_file(img_path)
            #img = tf.image.decode_image(img)
            #img = tf.image.encode_jpeg(img) #统一成jpeg格式压缩
            example = tf.train.Example(
               features=tf.train.Features(feature={
                    'label': tf.train.Feature(int64_list=tf.train.Int64List(value=[index])),
                    'img_raw': tf.train.Feature(bytes_list=tf.train.BytesList(value=[img.numpy()]))
               }))
            writer.write(example.SerializeToString())
    writer.close()
# 将数据打包成tfrecord文件
create_tfrecords("./data/cifar2/test/","./data/cifar2_test.tfrecords/")


def parse_example(proto):
    description ={ 'img_raw' : tf.io.FixedLenFeature([], tf.string),
                   'label': tf.io.FixedLenFeature([], tf.int64)} 
    example = tf.io.parse_single_example(proto, description)
    img = tf.image.decode_jpeg(example["img_raw"])   #注意此处为jpeg格式
    img = tf.image.resize(img, (32,32))
    label = example["label"]
    return(img,label)

# 读取tfrecord文件
ds7 = tf.data.TFRecordDataset("./data/cifar2_test.tfrecords").map(parse_example).shuffle(3000)

```



## 二、应用数据转换

**Dataset**数据结构应用非常灵活，因为它本质上是一个Sequece序列，其每个元素可以是各种类型，例如可以是**张量，列表，字典，也可以是Dataset。**

Dataset包含了非常丰富的数据转换功能。

- `map`: 将转换函数映射到数据集每一个元素。

  ```python
  #map:将转换函数映射到数据集每一个元素
  ds = tf.data.Dataset.from_tensor_slices(["hello world","hello China","hello Beijing"])
  ds_map = ds.map(lambda x:tf.strings.split(x," "))
  for x in ds_map:
      print(x)
  ```

- `flat_map`: 将转换函数映射到数据集的每一个元素，并将嵌套的Dataset压平。

  ```python
  ds = tf.data.Dataset.from_tensor_slices(["hello world","hello China","hello Beijing"])
  ds_flatmap = ds.flat_map(lambda x:tf.data.Dataset
                           .from_tensor_slices(tf.strings.split(x," ")))
  for x in ds_flatmap:
      tf.print(x)
  
  # 打印结果：
      hello
      world
      hello
      China
      hello
      Beijing
  ```

- `interleave`: 效果类似flat_map,但可以将不同来源的数据夹在一起。

  ```python
  ds = tf.data.Dataset.from_tensor_slices(["hello world","hello China","hello Beijing"])
  ds_interleave = ds.interleave(lambda x:tf.data.Dataset.from_tensor_slices(tf.strings.split(x," ")))
  for x in ds_interleave:
      print(x)
  
  # 打印结果
      tf.Tensor(b'hello', shape=(), dtype=string)
      tf.Tensor(b'hello', shape=(), dtype=string)
      tf.Tensor(b'hello', shape=(), dtype=string)
      tf.Tensor(b'world', shape=(), dtype=string)
      tf.Tensor(b'China', shape=(), dtype=string)
      tf.Tensor(b'Beijing', shape=(), dtype=string)
  ```

- `filter`: 过滤掉某些元素。

  ```python
  #找出含有字母a或B的元素
  ds_filter = ds.filter(lambda x: tf.strings.regex_full_match(x, ".*[a|B].*"))
  ```

- `zip`: 将三个长度相同的Dataset横向铰合。

  ```python
  #zip:将三个长度相同的Dataset横向铰合。
  
  ds1 = tf.data.Dataset.range(0,3)
  ds2 = tf.data.Dataset.range(3,6)
  ds3 = tf.data.Dataset.range(6,9)
  ds_zip = tf.data.Dataset.zip((ds1,ds2,ds3))
  for x,y,z in ds_zip:
      print(x.numpy(),y.numpy(),z.numpy())
      
  # 打印结果
      0 3 6
      1 4 7
      2 5 8
  ```

- `concatenate`: 将两个Dataset纵向连接。

  ```python
  ds1 = tf.data.Dataset.range(0,3)
  ds2 = tf.data.Dataset.range(3,6)
  ds_concat = tf.data.Dataset.concatenate(ds1,ds2)
  for x in ds_concat:
      tf.print(x)
  
  # 打印结果
      0
      1
      2
      3
      4
      5
  ```

- `reduce`: 执行归并操作。

  ```python
  ds = tf.data.Dataset.from_tensor_slices([1,2,3,4,5.0])
  result = ds.reduce(0.0,lambda x,y:tf.add(x,y))
  tf.print(result)
  
  # 打印结果
  	15
  ```

- `batch` : 构建批次，每次放一个批次。比原始数据增加一个维度。 其逆操作为unbatch。

  ```python
  ds = tf.data.Dataset.range(12)
  ds_batch = ds.batch(3)
  for x in ds_batch:
      tf.print(x)
      
  # 打印结果
      [0 1 2]
      [3 4 5]
      [6 7 8]
      [9 10 11]
  ```

- `padded_batch`: 构建批次，类似batch, 但可以填充到相同的形状。

  ```python
  elements = [[1, 2],[3, 4, 5],[6, 7],[8]]
  ds = tf.data.Dataset.from_generator(lambda: iter(elements), tf.int32)
  
  ds_padded_batch = ds.padded_batch(2,padded_shapes = [4,])
  for x in ds_padded_batch:
      tf.print(x)
      
      
  # 打印结果：
  tf.Tensor(
  [[1 2 0 0]
   [3 4 5 0]], shape=(2, 4), dtype=int32)
  tf.Tensor(
  [[6 7 0 0]
   [8 0 0 0]], shape=(2, 4), dtype=int32)
  ```

- `window` :构建滑动窗口，返回Dataset of Dataset.

  ```python
  ds = tf.data.Dataset.range(12)
  #window返回的是Dataset of Dataset,可以用flat_map压平
  ds_window = ds.window(3, shift=1).flat_map(lambda x: x.batch(3,drop_remainder=True)) 
  for x in ds_window:
      print(x)
      
  # 打印结果：
      tf.Tensor([0 1 2], shape=(3,), dtype=int64)
      tf.Tensor([1 2 3], shape=(3,), dtype=int64)
      tf.Tensor([2 3 4], shape=(3,), dtype=int64)
      tf.Tensor([3 4 5], shape=(3,), dtype=int64)
      tf.Tensor([4 5 6], shape=(3,), dtype=int64)
  ```

- `shuffle`: 数据顺序洗牌。

  ```python
  ds = tf.data.Dataset.range(12)
  ds_shuffle = ds.shuffle(buffer_size = 5)
  for x in ds_shuffle:
      print(x)
  ```

- `repeat`: 重复数据若干次，不带参数时，重复无数次。

  ```python
  ds = tf.data.Dataset.range(3)
  ds_repeat = ds.repeat(3)
  for x in ds_repeat:
      print(x)
      
  # 打印结果：
  tf.Tensor(0, shape=(), dtype=int64)
  tf.Tensor(1, shape=(), dtype=int64)
  tf.Tensor(2, shape=(), dtype=int64)
  tf.Tensor(0, shape=(), dtype=int64)
  tf.Tensor(1, shape=(), dtype=int64)
  tf.Tensor(2, shape=(), dtype=int64)
  tf.Tensor(0, shape=(), dtype=int64)
  tf.Tensor(1, shape=(), dtype=int64)
  tf.Tensor(2, shape=(), dtype=int64)
  ```

- `shard`: 采样，从某个位置开始隔固定距离采样一个元素。

  ```python
  ds = tf.data.Dataset.range(12)
  ds_shard = ds.shard(3,index = 1)
  
  for x in ds_shard:
      print(x)
  
  # 打印结果：
  tf.Tensor(1, shape=(), dtype=int64)
  tf.Tensor(4, shape=(), dtype=int64)
  tf.Tensor(7, shape=(), dtype=int64)
  tf.Tensor(10, shape=(), dtype=int64)
  ```

- `take`: 采样，从开始位置取前几个元素。

  ```python
  ds = tf.data.Dataset.range(12)
  ds_take = ds.take(3)
  
  list(ds_take.as_numpy_iterator())
  
  # 打印结果
  [0, 1, 2]
  ```

## 三、提升管道性能

模型训练的耗时主要来自于两个部分，一部分来自**数据准备**，另一部分来自**参数迭代**。

参数迭代过程的耗时通常依赖于GPU来提升。

而数据准备过程的耗时则可以通过**构建高效的数据管道**进行提升。

以下是一些构建高效数据管道的建议。

- 1，使用 `prefetch` 方法让数据准备和参数迭代两个过程相互并行。

  ```python
  import tensorflow as tf
  
  #打印时间分割线
  @tf.function
  def printbar():
      ts = tf.timestamp()
      today_ts = ts%(24*60*60)
  
      hour = tf.cast(today_ts//3600+8,tf.int32)%tf.constant(24)
      minite = tf.cast((today_ts%3600)//60,tf.int32)
      second = tf.cast(tf.floor(today_ts%60),tf.int32)
  
      def timeformat(m):
          if tf.strings.length(tf.strings.format("{}",m))==1:
              return(tf.strings.format("0{}",m))
          else:
              return(tf.strings.format("{}",m))
  
      timestring = tf.strings.join([timeformat(hour),timeformat(minite),
                  timeformat(second)],separator = ":")
      tf.print("=========="*8,end = "")
      tf.print(timestring)
      
  # tf.data.experimental.AUTOTUNE 可以让程序自动选择合适的参数
  for x in ds.prefetch(buffer_size = tf.data.experimental.AUTOTUNE):
      train_step() 
  ```

- 2，使用 `interleave` 方法可以让数据读取过程多进程执行,并将不同来源数据夹在一起。

  ```python
  ds_files = tf.data.Dataset.list_files("./data/titanic/*.csv")
  # ds = ds_files.flat_map(lambda x:tf.data.TextLineDataset(x).skip(1))
  # 使用interleave()方法代替flat_map()方法
  ds = ds_files.interleave(lambda x:tf.data.TextLineDataset(x).skip(1))
  for line in ds.take(8):
      print(line)
  ```

- 3，使用 `map` 时设置`num_parallel_calls` 让数据转换过程多进行执行。

  ```python
  ds = tf.data.Dataset.list_files("./data/cifar2/train/*/*.jpg")
  def load_image(img_path,size = (32,32)):
      label = 1 if tf.strings.regex_full_match(img_path,".*/automobile/.*") else 0
      img = tf.io.read_file(img_path)
      img = tf.image.decode_jpeg(img) #注意此处为jpeg格式
      img = tf.image.resize(img,size)
      return(img,label)
  
  
  #多进程转换
  printbar()
  tf.print(tf.constant("start parallel transformation..."))
  
  ds_map_parallel = ds.map(load_image,num_parallel_calls = tf.data.experimental.AUTOTUNE)
  for _ in ds_map_parallel:
      pass
  ```

- 4，使用 `cache` 方法让数据在第一个`epoch`后缓存到内存中，仅限于数据集不大情形。

  ```python
  import time
  
  # 模拟数据准备
  def generator():
      for i in range(5):
          #假设每次准备数据需要2s
          time.sleep(2) 
          yield i 
          
  # 使用 cache 方法让数据在第一个epoch后缓存到内存中，仅限于数据集不大情形。
  ds = tf.data.Dataset.from_generator(generator,output_types = (tf.int32)).cache()
  
  # 模拟参数迭代
  def train_step():
      #假设每一步训练需要0s
      time.sleep(0) 
  ```

- 5，使用 `map`转换时，先`batch`, 然后采用向量化的转换方法对每个batch进行转换。

  ```python
  #先batch后map
  ds = tf.data.Dataset.range(100000)
  ds_batch_map = ds.batch(20).map(lambda x:x**2)
  
  printbar()
  tf.print(tf.constant("start vector transformation..."))
  for x in ds_batch_map:
      pass
  printbar()
  tf.print(tf.constant("end vector transformation..."))
  ```