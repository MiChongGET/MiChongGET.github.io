---
title: TensorFlow2.X学习笔记(1)--TensorFlow核心概念
date: 2020-04-29 23:18:34
tags:
- Python
- TensorFlow
categories:
- 深度学习
description: TensorFlow™ 是一个采用 数据流图（data flow graphs），用于数值计算的开源软件库。节点（Nodes）在图中表示数学操作，图中的线（edges）则表示在节点间相互联系的多维数据数组，即张量（tensor）。它灵活的架构让你可以在多种平台上展开计算，例如台式计算机中的一个或多个CPU（或GPU），服务器，移动设备等等。
top_img: https://ae01.alicdn.com/kf/Hf295045b24cc4e59aa501320d265cc4bE.jpg
cover: https://ae01.alicdn.com/kf/H24c9452e63dd4cc08266332d7111b080Z.jpg
---
> 该系列笔记来自于对https://lyhue1991.github.io/eat_tensorflow2_in_30_days 文档的学习，感谢大神的文档！

  `TensorFlow`™ 是一个采用 **数据流图**（data flow graphs），用于数值计算的开源软件库。节点（Nodes）在图中表示数学操作，图中的线（edges）则表示在节点间相互联系的多维数据数组，即张量（tensor）。它灵活的架构让你可以**在多种平台上展开计算**，例如台式计算机中的一个或多个CPU（或GPU），服务器，移动设备等等。TensorFlow 最初由Google大脑小组（隶属于Google机器智能研究机构）的研究员和工程师们开发出来，**用于机器学习和深度神经网络**方面的研究，但这个系统的通用性使其也可**广泛用于其他计算领域**。

TensorFlow的`主要优点`：
- 灵活性：支持底层数值计算，C++自定义操作符
- 可移植性：从服务器到PC到手机，从CPU到GPU到TPU
- 分布式计算：分布式并行计算，可指定操作符对应计算设备

#### 1、张量数据结构

`TensorFlow程序` = `张量数据结构` + `计算图算法语言`

张量和计算图是 TensorFlow的核心概念。

Tensorflow的基本数据结构是张量Tensor。张量即多维数组。Tensorflow的张量和numpy中的array很类似。

从行为特性来看，有两种类型的张量，常量constant和变量Variable.

`常量`的值在计算图中不可以被重新赋值，`变量`可以在计算图中用`assign`等算子重新赋值。



##### （1）常量张量

```python
i = tf.constant(1) # tf.int32 类型常量
l = tf.constant(1,dtype = tf.int64) # tf.int64 类型常量
f = tf.constant(1.23) #tf.float32 类型常量
d = tf.constant(3.14,dtype = tf.double) # tf.double 类型常量
s = tf.constant("hello world") # tf.string类型常量
b = tf.constant(True) #tf.bool类型常量
```

`标量`为`0维`张量，`向量`为`1维`张量，`矩阵`为`2维`张量。

`彩色图像`有rgb三个通道，可以表示为`3维`张量。

`视频`还有时间维，可以表示为`4维`张量。

```python
scalar = tf.constant(True)  #标量，0维张量
vector = tf.constant([1.0,2.0,3.0,4.0]) #向量，1维张量
matrix = tf.constant([[1.0,2.0],[3.0,4.0]]) #矩阵, 2维张量
tensor3 = tf.constant([[[1.0,2.0],[3.0,4.0]],[[5.0,6.0],[7.0,8.0]]])  # 3维张量
```

```python
#可以用tf.cast改变张量的数据类型。
h = tf.constant([123,456],dtype = tf.int32)
f = tf.cast(h,tf.float32)

#可以用numpy方法将tensorflow中的张量转化成numpy中的张量。
y = tf.constant([[1.0,2.0],[3.0,4.0]])
y.numpy() #转换成np.array

#可以用shape方法查看张量的尺寸。
y.shape	
```

##### （2）变量张量

```python
# 常量值不可以改变，常量的重新赋值相当于创造新的内存空间
c = tf.constant([1.0,2.0])

# 变量的值可以改变，可以通过assign, assign_add等方法给变量重新赋值
v = tf.Variable([1.0,2.0],name = "v")
v.assign_add([1.0,1.0])
```



#### 2、三种计算图

`静态计算图`，`动态计算图`，以及`Autograph`.

> 在TensorFlow2.0时代，采用的是动态计算图，即每使用一个算子后，该算子会被动态加入到隐含的默认计算图中立即执行得到结果，而无需开启Session。

使用动态计算图即Eager Excution的好处是方便调试程序，它会让TensorFlow代码的表现和Python原生代码的表现一样，写起来就像写numpy一样，各种日志打印，控制流全部都是可以使用的。

使用动态计算图的缺点是运行效率相对会低一些。因为使用动态图会有许多次Python进程和TensorFlow的C++进程之间的通信。而静态计算图构建完成之后几乎全部在TensorFlow内核上使用C++代码执行，效率更高。此外静态图会对计算步骤进行一定的优化，剪去和结果无关的计算步骤。

如果需要在TensorFlow2.0中使用静态图，可以使用@tf.function装饰器将普通Python函数转换成对应的TensorFlow计算图构建代码。运行该函数就相当于在TensorFlow1.0中用Session执行代码。使用tf.function构建静态图的方式叫做 Autograph.

##### (1)静态计算图

```python
#在TensorFlow1.0中，使用静态计算图分两步，第一步定义计算图，第二步在会话中执行计算图。

import tensorflow as tf

#定义计算图
g = tf.Graph()
with g.as_default():
    #placeholder为占位符，执行会话时候指定填充对象
    x = tf.placeholder(name='x', shape=[], dtype=tf.string)  
    y = tf.placeholder(name='y', shape=[], dtype=tf.string)
    z = tf.string_join([x,y],name = 'join',separator=' ')

#执行计算图
with tf.Session(graph = g) as sess:
    print(sess.run(fetches = z,feed_dict = {x:"hello",y:"world"}))
```

**TensorFlow2.0 怀旧版静态计算图**

```python
#TensorFlow2.0为了确保对老版本tensorflow项目的兼容性，在tf.compat.v1子模块中保留了对TensorFlow1.0那种静态计算图构建风格的支持。

import tensorflow as tf
g = tf.compat.v1.Graph()
with g.as_default():
    x = tf.compat.v1.placeholder(name='x', shape=[], dtype=tf.string)
    y = tf.compat.v1.placeholder(name='y', shape=[], dtype=tf.string)
    z = tf.strings.join([x,y],name = "join",separator = " ")

with tf.compat.v1.Session(graph = g) as sess:
    # fetches的结果非常像一个函数的返回值，而feed_dict中的占位符相当于函数的参数序列。
    result = sess.run(fetches = z,feed_dict = {x:"hello",y:"world"})
    print(result)
```



##### (2)动态计算图

> 动态计算图已经不区分计算图的定义和执行了，而是定义后立即执行。因此称之为 `Eager Excution`.Eager这个英文单词的原意是"迫不及待的"，也就是立即执行的意思。(但是动态计算图运行效率比较低！)

```python
# 动态计算图在每个算子处都进行构建，构建后立即执行

x = tf.constant("hello")
y = tf.constant("world")
z = tf.strings.join([x,y],separator=" ")
```



##### (3)TensorFlow2.0的Autograph

> 在TensorFlow2.0中，如果采用Autograph的方式使用计算图，第一步定义计算图变成了定义函数，第二步执行计算图变成了调用函数。



> 实践中，我们一般会先用动态计算图调试代码，然后在需要提高性能的的地方利用`@tf.function`切换成Autograph获得更高的效率。



```python
import tensorflow as tf

# 可以用@tf.function装饰器将普通Python函数转换成和TensorFlow1.0对应的静态计算图构建代码。
# 使用autograph构建静态图

@tf.function
def strjoin(x,y):
    z =  tf.strings.join([x,y],separator = " ")
    tf.print(z)
    return z

result = strjoin(tf.constant("hello"),tf.constant("world"))

print(result)
```



#### 3、自动微分机制

> 神经网络通常依赖反向传播求梯度来更新网络参数，求梯度过程通常是一件非常复杂而容易出错的事情。而深度学习框架可以帮助我们自动地完成这种求梯度运算。Tensorflow一般使用梯度磁带`tf.GradientTape`来记录正向运算过程，然后反播磁带自动得到梯度值。这种利用tf.GradientTape求微分的方法叫做Tensorflow的自动微分机制。

##### （1）利用梯度磁带求导数

```python
import tensorflow as tf
import numpy as np 

# f(x) = a*x**2 + b*x + c的导数

x = tf.Variable(0.0,name = "x",dtype = tf.float32)
a = tf.constant(1.0)
b = tf.constant(-2.0)
c = tf.constant(1.0)

with tf.GradientTape() as tape:
    y = a*tf.pow(x,2) + b*x + c

dy_dx = tape.gradient(y,x)


# 对常量张量也可以求导，需要增加watch
with tf.GradientTape() as tape:
    tape.watch([a,b,c])
    y = a*tf.pow(x,2) + b*x + c
dy_dx,dy_da,dy_db,dy_dc = tape.gradient(y,[x,a,b,c])

# 可以求二阶导数
with tf.GradientTape() as tape2:
    with tf.GradientTape() as tape1:   
        y = a*tf.pow(x,2) + b*x + c
    dy_dx = tape1.gradient(y,x)   
dy2_dx2 = tape2.gradient(dy_dx,x)

# 可以在autograph中使用
@tf.function
def f(x):   
    a = tf.constant(1.0)
    b = tf.constant(-2.0)
    c = tf.constant(1.0)

    # 自变量转换成tf.float32
    x = tf.cast(x,tf.float32)
    with tf.GradientTape() as tape:
        tape.watch(x)
        y = a*tf.pow(x,2)+b*x+c
    dy_dx = tape.gradient(y,x) 

    return((dy_dx,y))
```

##### （2）利用梯度磁带和优化器求最小值

###### 方法一

```python
# 求f(x) = a*x**2 + b*x + c的最小值
# 使用optimizer.apply_gradients

x = tf.Variable(0.0,name = "x",dtype = tf.float32)
a = tf.constant(1.0)
b = tf.constant(-2.0)
c = tf.constant(1.0)

optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)
for _ in range(1000):
    with tf.GradientTape() as tape:
        y = a*tf.pow(x,2) + b*x + c
    dy_dx = tape.gradient(y,x)
    optimizer.apply_gradients(grads_and_vars=[(dy_dx,x)])

tf.print("y =",y,"; x =",x)


y = 0 ; x = 0.999998569
```

###### 方法二

```python
# 求f(x) = a*x**2 + b*x + c的最小值
# 使用optimizer.minimize
# optimizer.minimize相当于先用tape求gradient,再apply_gradient

x = tf.Variable(0.0,name = "x",dtype = tf.float32)

#注意f()无参数
def f():   
    a = tf.constant(1.0)
    b = tf.constant(-2.0)
    c = tf.constant(1.0)
    y = a*tf.pow(x,2)+b*x+c
    return(y)

optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)   
for _ in range(1000):
    optimizer.minimize(f,[x])   

tf.print("y =",f(),"; x =",x)

y = 0 ; x = 0.999998569
```

###### 方式三

```python
# 在autograph中完成最小值求解
# 使用optimizer.apply_gradients

x = tf.Variable(0.0,name = "x",dtype = tf.float32)
optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)

@tf.function
def minimizef():
    a = tf.constant(1.0)
    b = tf.constant(-2.0)
    c = tf.constant(1.0)

    for _ in tf.range(1000): #注意autograph时使用tf.range(1000)而不是range(1000)
        with tf.GradientTape() as tape:
            y = a*tf.pow(x,2) + b*x + c
        dy_dx = tape.gradient(y,x)
        optimizer.apply_gradients(grads_and_vars=[(dy_dx,x)])

    y = a*tf.pow(x,2) + b*x + c
    return y

tf.print(minimizef())
tf.print(x)
```

###### 方式四

```python
# 在autograph中完成最小值求解
# 使用optimizer.minimize

x = tf.Variable(0.0,name = "x",dtype = tf.float32)
optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)   

@tf.function
def f():   
    a = tf.constant(1.0)
    b = tf.constant(-2.0)
    c = tf.constant(1.0)
    y = a*tf.pow(x,2)+b*x+c
    return(y)

@tf.function
def train(epoch):  
    for _ in tf.range(epoch):  
        optimizer.minimize(f,[x])
    return(f())


tf.print(train(1000))
tf.print(x)
```