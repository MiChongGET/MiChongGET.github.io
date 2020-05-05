---
title: TensorFlow2.X学习笔记(4)--TensorFlow低阶API之AutoGraph相关研究
date: 2020-05-02 22:59:57
tags:
- Python
- TensorFlow
categories:
- 深度学习
description: TensorFlow 2.0主要使用的是动态计算图和Autograph。而Autograph机制可以将动态图转换成静态计算图，兼收执行效率和编码效率之利。
top_img: https://ae01.alicdn.com/kf/H2a2c96993f7d4d0ab8fc5e950cdfa9663.jpg
cover: https://ae01.alicdn.com/kf/H24c9452e63dd4cc08266332d7111b080Z.jpg
---
# AutoGraph相关研究

TensorFlow 2.0主要使用的是动态计算图和Autograph。

动态计算图易于调试，编码效率较高，但执行效率偏低。

静态计算图执行效率很高，但较难调试。

而Autograph机制可以将动态图转换成静态计算图，兼收执行效率和编码效率之利。

当然Autograph机制能够转换的代码并不是没有任何约束的，有一些编码规范需要遵循，否则可能会转换失败或者不符合预期。

## 一、Autograph使用规范

### 1、规范总结

- 1，被`@tf.function`修饰的函数应尽可能使用TensorFlow中的函数而不是Python中的其他函数。例如使用tf.print而不是print，使用tf.range而不是range，使用`tf.constant(True)`而不是True.
- 2，避免在@tf.function修饰的函数内部定义tf.Variable.
- 3，被@tf.function修饰的函数不可修改该函数外部的Python列表或字典等数据结构变量。

### 2、规范解析

**被@tf.function修饰的函数应尽量使用TensorFlow中的函数而不是Python中的其他函数。**

```python
import numpy as np
import tensorflow as tf

@tf.function
def np_random():
    a = np.random.randn(3,3)
    tf.print(a)

@tf.function
def tf_random():
    a = tf.random.normal((3,3))
    tf.print(a)
    
# np_random每次执行都是一样的结果
# tf_random每次执行都会有重新生成随机数。
```

**避免在@tf.function修饰的函数内部定义tf.Variable.**

```python
x = tf.Variable(1.0,dtype=tf.float32)
@tf.function
def outer_var():
    x.assign_add(1.0)
    tf.print(x)
```

**被@tf.function修饰的函数不可修改该函数外部的Python列表或字典等结构类型变量。**

```python
tensor_list = []

@tf.function #加上这一行切换成Autograph结果将不符合预期！！！
def append_tensor(x):
    tensor_list.append(x) #测试在此处的tensor_list虽然使用append的方法，但是却起不到作用
    return tensor_list
```



## 二、Autograph机制原理

### 1、@tf.function

**当我们第一次调用这个被@tf.function装饰的函数时，后面到底发生了什么？**

- 第一件事情是创建计算图。即创建一个静态计算图，跟踪执行一遍函数体中的Python代码，确定各个变量的Tensor类型，并根据执行顺序将算子添加到计算图中。 在这个过程中，如果开启了autograph=True(默认开启),会将Python控制流转换成TensorFlow图内控制流。 主要是将if语句转换成 tf.cond算子表达，将while和for循环语句转换成tf.while_loop算子表达，并在必要的时候添加 tf.control_dependencies指定执行顺序依赖关系。

- 第二件事情是执行计算图。

  

### 2、重新理解Autograph的编码规范

- **1，被@tf.function修饰的函数应尽量使用TensorFlow中的函数而不是Python中的其他函数。例如使用tf.print而不是print.**

  解释：Python中的函数仅仅会在跟踪执行函数以创建静态图的阶段使用，普通Python函数是无法嵌入到静态计算图中的，所以 在计算图构建好之后再次调用的时候，这些Python函数并没有被计算，而TensorFlow中的函数则可以嵌入到计算图中。使用普通的Python函数会导致 被@tf.function修饰前【eager执行】和被@tf.function修饰后【静态图执行】的输出不一致。

- **2，避免在@tf.function修饰的函数内部定义tf.Variable.**

  解释：如果函数内部定义了tf.Variable,那么在【eager执行】时，这种创建tf.Variable的行为在每次函数调用时候都会发生。但是在【静态图执行】时，这种创建tf.Variable的行为只会发生在第一步跟踪Python代码逻辑创建计算图时，这会导致被@tf.function修饰前【eager执行】和被@tf.function修饰后【静态图执行】的输出不一致。实际上，TensorFlow在这种情况下一般会报错。

- **3，被@tf.function修饰的函数不可修改该函数外部的Python列表或字典等数据结构变量。**

  解释：静态计算图是被编译成C++代码在TensorFlow内核中执行的。Python中的列表和字典等数据结构变量是无法嵌入到计算图中，它们仅仅能够在创建计算图时被读取，在执行计算图时是无法修改Python中的列表或字典这样的数据结构变量的。

## 三、tf.Module概述

​	TensorFlow提供了一个基类`tf.Module`，通过继承它构建子类，我们不仅可以获得以上的自然而然，而且可以非常方便地管理变量，还可以非常方便地管理它引用的其它Module，而且我们能够利用`tf.saved_model`保存模型并实现跨平台部署使用。``

​	实际上，`tf.keras.models.Model,tf.keras.layers.Layer` 都是继承自`tf.Module`的，提供了方便的变量管理和所引用的子模块管理的功能。

### 1、tf.Module

**因此，利用tf.Module提供的封装，再结合TensoFlow丰富的低阶API，实际上我们能够基于TensorFlow开发任意机器学习模型(而非仅仅是神经网络模型)，并实现跨平台部署使用。**

> 定义一个简单的function.

```python
import tensorflow as tf 
x = tf.Variable(1.0,dtype=tf.float32)

#在tf.function中用input_signature限定输入张量的签名类型：shape和dtype,当a不符合这个标准，就会报错
@tf.function(input_signature=[tf.TensorSpec(shape = [], dtype = tf.float32)])    
def add_print(a):
    x.assign_add(a)
    tf.print(x)
    return(x)
```

> 自定义module

```python
class DemoModule(tf.Module):
    def __init__(self,init_value = tf.constant(0.0),name=None):
        super(DemoModule, self).__init__(name=name)
        with self.name_scope:  #相当于with tf.name_scope("demo_module")
            self.x = tf.Variable(init_value,dtype = tf.float32,trainable=True)


    @tf.function(input_signature=[tf.TensorSpec(shape = [], dtype = tf.float32)])  
    def addprint(self,a):
        with self.name_scope:
            self.x.assign_add(a)
            tf.print(self.x)
            return(self.x)
        
#执行
demo = DemoModule(init_value = tf.constant(1.0))
result = demo.addprint(tf.constant(5.0))

#查看模块中的全部变量和全部可训练变量
print(demo.variables)
print(demo.trainable_variables)
#查看模块中的全部子模块
demo.submodules

#使用tf.saved_model 保存模型，并指定需要跨平台部署的方法
tf.saved_model.save(demo,"./data/demo/1",signatures = {"serving_default":demo.addprint})
#加载模型
demo2 = tf.saved_model.load("./data/demo/1")
demo2.addprint(tf.constant(5.0))

# 查看模型文件相关信息，红框标出来的输出信息在模型部署和跨平台使用时有可能会用到
!saved_model_cli show --dir ./data/demo/1 --all
```

> 在tensorboard中查看计算图，模块会被添加模块名demo_module,方便层次化呈现计算图结构。

```python
import datetime

# 创建日志
stamp = datetime.datetime.now().strftime("%Y%m%d-%H%M%S")
logdir = './data/demomodule/%s' % stamp
writer = tf.summary.create_file_writer(logdir)

#开启autograph跟踪
tf.summary.trace_on(graph=True, profiler=True) 

#执行autograph
demo = DemoModule(init_value = tf.constant(0.0))
result = demo.addprint(tf.constant(5.0))

#将计算图信息写入日志
with writer.as_default():
    tf.summary.trace_export(
        name="demomodule",
        step=0,
        profiler_outdir=logdir)
```

> 使用tensorboard

```python
#启动 tensorboard在jupyter中的魔法命令
from tensorboard import notebook
notebook.list()
notebook.start("--logdir ./data/demomodule/")
```

![](https://ae01.alicdn.com/kf/H1c4b8f2321884270b7b33ebc987197d6U.jpg)

> 除了利用tf.Module的子类化实现封装，我们也可以通过给`tf.Module`添加属性的方法进行封装。

```python
mymodule = tf.Module()
mymodule.x = tf.Variable(0.0)

@tf.function(input_signature=[tf.TensorSpec(shape = [], dtype = tf.float32)])  
def addprint(a):
    mymodule.x.assign_add(a)
    tf.print(mymodule.x)
    return (mymodule.x)

mymodule.addprint = addprint
```

### 2、tf.Module和tf.keras.Model，tf.keras.layers.Layer

> tf.keras中的模型和层都是继承tf.Module实现的，也具有变量管理和子模块管理功能。

```python
import tensorflow as tf
from tensorflow.keras import models,layers,losses,metrics

tf.keras.backend.clear_session() 

model = models.Sequential()

model.add(layers.Dense(4,input_shape = (10,)))
model.add(layers.Dense(2))
model.add(layers.Dense(1))
model.summary()

# 打印结果：
Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
dense (Dense)                (None, 4)                 44        
_________________________________________________________________
dense_1 (Dense)              (None, 2)                 10        
_________________________________________________________________
dense_2 (Dense)              (None, 1)                 3         
=================================================================
Total params: 57
Trainable params: 57
Non-trainable params: 0
    

# 获得model中的变量
model.variables
# 获得model中的可训练变量
model.trainable_variables
model.layers[0].trainable = False #冻结第0层的变量,使其不可训练

model.submodules #获得每层model的情况
## Sequence of all sub-modules.
## Submodules are modules which are properties of this module, or found as
## properties of modules which are properties of this module (and so on).

model.layers

```