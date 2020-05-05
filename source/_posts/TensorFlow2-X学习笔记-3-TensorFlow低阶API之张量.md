---
title: TensorFlow2.X学习笔记(3)--TensorFlow低阶API之张量
date: 2020-05-02 01:18:43
tags:
- Python
- TensorFlow
categories:
- 深度学习
description: 张量的操作主要包括张量的结构操作和张量的数学运算。张量结构操作诸如：张量创建，索引切片，维度变换，合并分割。张量数学运算主要有：标量运算，向量运算，矩阵运算。另外我们会介绍张量运算的广播机制。
top_img: https://ae01.alicdn.com/kf/H3ffb1faadb934fd8a85760e1a11e1418p.jpg
cover: https://ae01.alicdn.com/kf/H24c9452e63dd4cc08266332d7111b080Z.jpg
---
TensorFlow的低阶API主要包括`张量操作`，`计算图`和`自动微分`。

如果把模型比作一个房子，那么低阶API就是**【模型之砖】**。

在低阶API层次上，可以把TensorFlow当做一个增强版的numpy来使用。

TensorFlow提供的方法比numpy更全面，运算速度更快，如果需要的话，还可以使用GPU进行加速。



#### 一、张量的结构操作

张量的操作主要包括张量的结构操作和张量的数学运算。

**张量结构操作**诸如：`张量创建，索引切片，维度变换，合并分割`。

**张量数学运算**主要有：`标量运算，向量运算，矩阵运算`。另外我们会介绍张量运算的广播机制。

Autograph计算图我们将介绍使用Autograph的规范建议，Autograph的机制原理，Autograph和tf.Module.

##### 1、创建张量

```python
import tensorflow as tf
import numpy as np

# 1、创建一个一维张量
a = tf.constant([1,2,3],dtype = tf.float32)
tf.print(a)

# 2、创建范围1-10，间隔为2的张量
tf.print(tf.range(1,10,delta=2))

# 3、0.0-6.18之间分成100份
tf.print(tf.linspace(0.0,2*3.14,100))

# 4、创建3*3的零向量
tf.print(tf.zeros([3,3]))

# 5、创建3*2,用5填充的张量
tf.print( tf.fill([3,2],5))

# 6、均匀分布随机
tf.random.set_seed(1.0) #Sets the global random seed.
a = tf.random.uniform([5],minval=0,maxval=10)
tf.print(a)

# 7、正态随机分布
tf.print(tf.random.normal([3,3],mean=0.0,stddev=1.0))

# 8、正态分布随机，剔除2倍方差以外数据重新生成
c = tf.random.truncated_normal((5,5), mean=0.0, stddev=1.0, dtype=tf.float32)
tf.print(c)

# 打印结果
1、[1 2 3]
2、[1 3 5 7 9]
3、[0 0.0634343475 0.126868695 ... 6.15313148 6.21656609 6.28]
4、
[[0 0 0]
 [0 0 0]
 [0 0 0]]
5、
[[5 5]
 [5 5]
 [5 5]]

6、[2.91975141 2.06566453 5.35390759 5.61257458 4.16674519]

7、[[1.06688023 0.194549292 -0.530828953]
   [0.0919008255 -0.177537084 -0.919308841]
   [-2.07775569 2.03919029 0.802899718]]

8、
[[-1.80412865 -0.111534528 -0.845551133 0.848961473 0.181714371]
 [0.0783366337 -0.772812247 0.510512829 1.09207666 -0.685003579]
 [-0.0209237766 -0.870738804 0.00304621807 0.29193154 -0.484454393]
 [1.13532615 -0.133236796 -0.620660245 1.43435645 -0.0828505158]
 [0.762984335 0.0506231971 -0.368702501 -0.46321547 -0.0791869536]]
```



##### 2、索引切片

张量的索引切片方式和numpy几乎是一样的。切片时支持缺省参数和省略号。

对于`tf.Variable`,可以通过索引和切片对部分元素进行修改。

对于提取张量的连续子区域，也可以使用`tf.slice`.

此外，对于不规则的切片提取,可以使用`tf.gather,tf.gather_nd,tf.boolean_mask`。

`tf.boolean_mask`功能最为强大，它可以实现`tf.gather,tf.gather_nd`的功能，并且`tf.boolean_mask`还可以实现布尔索引。

如果要通过修改张量的某些元素得到新的张量，可以使用`tf.where，tf.scatter_nd`。

```python
tf.random.set_seed(3)
t = tf.random.uniform([5,5],minval=0,maxval=10,dtype=tf.int32)
tf.print(t)

# 结果
[[4 7 4 2 9]
 [9 1 2 4 7]
 [7 2 7 4 0]
 [9 6 9 7 2]
 [3 7 0 0 3]]

#第0行
tf.print(t[0]) #[4 7 4 2 9]
#倒数第一行
tf.print(t[-1]) #[3 7 0 0 3]
#第1行第3列
tf.print(t[1,3])
tf.print(t[1][3]) 

#第1行至第3行
tf.print(t[1:4,:])
tf.print(tf.slice(t,[1,0],[3,5])) #tf.slice(input,begin_vector,size_vector) 从[1,0]位置开始，寻找三行五列的数据

#第1行至最后一行，第0列到最后一列每隔两列取一列
tf.print(t[1:4,:4:2]) #或者写成这样的：tf.print(t[1:4,0:4:2])

#对变量来说，还可以使用索引和切片修改部分元素
x = tf.Variable([[1,2],[3,4]],dtype = tf.float32)
x[1,:].assign(tf.constant([0.0,0.0]))

#省略号可以表示多个冒号
a=tf.random.uniform([3,3,3],minval=0,maxval=10,dtype=tf.int32)
# a的值：
		[[[2, 2, 6],
        [5, 7, 6],
        [4, 8, 6]],

       [[4, 6, 3],
        [8, 1, 7],
        [3, 1, 3]],

       [[2, 1, 6],
        [3, 1, 8],
        [9, 3, 7]]]
 tf.print(a[...,1]) #等价于tf.print(a[:,:,1])
# 结果：
		[[2 7 8]
 		[6 1 1]
 		[1 1 3]]
    
    
    
# 考虑班级成绩册的例子，有4个班级，每个班级10个学生，每个学生7门科目成绩。可以用一个4107的张量来表示。
scores = tf.random.uniform((4,10,7),minval=0,maxval=100,dtype=tf.int32)

# 抽取每个班级第0个学生，第5个学生，第9个学生的全部成绩
p = tf.gather(scores,[0,5,9],axis=1)

# 抽取每个班级第0个学生，第5个学生，第9个学生的第1门课程，第3门课程，第6门课程成绩
q = tf.gather(tf.gather(scores,[0,5,9],axis=1),[1,3,6],axis=2)

# 抽取第0个班级第0个学生，第2个班级的第4个学生，第3个班级的第6个学生的全部成绩
# indices的长度为采样样本的个数，每个元素为采样位置的坐标
s = tf.gather_nd(scores,indices = [(0,0),(2,4),(3,6)])

# 抽取每个班级第0个学生，第5个学生，第9个学生的全部成绩（等价位：tf.gather(scores,[0,5,9],axis=1)）
p = tf.boolean_mask(scores,[True,False,False,False,False,
                            True,False,False,False,True],axis=1)


#找到矩阵中小于0的元素
c = tf.constant([[-1,1,-1],[2,2,-2],[3,-3,3]],dtype=tf.float32)
tf.print(tf.boolean_mask(c,c<0),"\n") 
tf.print(c[c<0]) #布尔索引，为boolean_mask的语法糖形式


#找到张量中小于0的元素,将其换成np.nan得到新的张量
#tf.where和np.where作用类似，可以理解为if的张量版本
c = tf.constant([[-1,1,-1],[2,2,-2],[3,-3,3]],dtype=tf.float32)
d = tf.where(c<0,tf.fill(c.shape,np.nan),c) 

#如果where只有一个参数，将返回所有满足条件的位置坐标
indices = tf.where(c<0)
#将张量的第[0,0]和[2,1]两个位置元素替换为0得到新的张量
d = c - tf.scatter_nd([[0,0],[2,1]],[c[0,0],c[2,1]],c.shape)

#scatter_nd的作用和gather_nd有些相反
#可以将某些值插入到一个给定shape的全0的张量的指定位置处。
indices = tf.where(c<0)
tf.scatter_nd(indices,tf.gather_nd(c,indices),c.shape)


```

##### 3、维度变换

维度变换相关函数主要有 tf.reshape, tf.squeeze, tf.expand_dims, tf.transpose.

- `tf.reshape` 可以改变张量的形状。
- `tf.squeeze` 可以减少维度。如果张量在某个维度上只有一个元素，利用`tf.squeeze`可以消除这个维度。
- `tf.expand_dims` 可以增加维度。
- `tf.transpose` 可以交换维度，它会改变张量元素的存储顺序。`tf.transpose`常用于图片存储格式的变换上。

tf.reshape可以改变张量的形状，但是其本质上不会改变张量元素的存储顺序，所以，该操作实际上非常迅速，并且是可逆的。

```python
tf.expand_dims(s,axis=0) #在第0维插入长度为1的一个维度

# Batch,Height,Width,Channel
a = tf.random.uniform(shape=[100,600,600,4],minval=0,maxval=255,dtype=tf.int32)
tf.print(a.shape)

# 转换成 Channel,Height,Width,Batch
s= tf.transpose(a,perm=[3,1,2,0])
tf.print(s.shape)

# 结果：perm中定义了维度顺序
TensorShape([100, 600, 600, 4])
TensorShape([4, 600, 600, 100])
```

##### 4、合并分割

`tf.concat`和`tf.stack`有略微的区别，tf.concat是**连接**，不会增加维度，而tf.stack是**堆叠**，会增加维度。

```python
a = tf.constant([[1.0,2.0],[3.0,4.0]])
b = tf.constant([[5.0,6.0],[7.0,8.0]])
c = tf.constant([[9.0,10.0],[11.0,12.0]])

tf.concat([a,b,c],axis = 0)
<tf.Tensor: shape=(6, 2), dtype=float32, numpy=
array([[ 1.,  2.],
       [ 3.,  4.],
       [ 5.,  6.],
       [ 7.,  8.],
       [ 9., 10.],
       [11., 12.]], dtype=float32)>

tf.stack([a,b,c])
<tf.Tensor: shape=(3, 2, 2), dtype=float32, numpy=
array([[[ 1.,  2.],
        [ 3.,  4.]],

       [[ 5.,  6.],
        [ 7.,  8.]],

       [[ 9., 10.],
        [11., 12.]]], dtype=float32)>
```

> `tf.split`是tf.concat的逆运算，可以指定分割份数平均分割，也可以通过指定每份的记录数量进行分割。

```python
#tf.split(value,num_or_size_splits,axis)
tf.split(c,3,axis = 0)  #指定分割份数，平均分割

tf.split(c,[2,2,2],axis = 0) #指定每份的记录数量 
```

#### 二、张量的数学运算

##### 1、标量运算

```python
a = tf.constant([[1,2],[3,4]])
a%3 #mod的运算符重载，等价于m = tf.math.mod(a,3)
a//3  #地板除法
a==4 #tf.equal(a,4)
# 结果
<tf.Tensor: shape=(2, 2), dtype=bool, numpy=
array([[False, False],
       [False,  True]])>

# 三个张量相加
a = tf.constant([1.0,8.0])
b = tf.constant([5.0,6.0])
c = tf.constant([6.0,7.0])
tf.add_n([a,b,c])
tf.maximum(a,b)  #[5 8]
tf.minimum(a,b)  #[1 6]
```

##### 2、向量运算

> 向量运算符只在一个特定轴上运算，将一个向量映射到一个标量或者另外一个向量。 许多向量运算符都以reduce开头。

```python
#向量reduce
a = tf.range(1,10)
tf.print(tf.reduce_sum(a))
tf.print(tf.reduce_mean(a))
tf.print(tf.reduce_max(a))
tf.print(tf.reduce_min(a))
tf.print(tf.reduce_prod(a))

#张量指定维度进行reduce
b = tf.reshape(a,(3,3))
tf.print(tf.reduce_sum(b, axis=1, keepdims=True))
tf.print(tf.reduce_sum(b, axis=0, keepdims=True))

#bool类型的reduce
p = tf.constant([True,False,False])
q = tf.constant([False,False,True])
tf.print(tf.reduce_all(p)) #结果为0，计算一个张量在维度上元素的“逻辑和”
tf.print(tf.reduce_any(q)) #结果为1，在张量的维度上计算元素的 "逻辑或"

#cum扫描累积
a = tf.range(1,10)
tf.print(tf.math.cumsum(a))
tf.print(tf.math.cumprod(a))
# 结果：
[1 3 6 ... 28 36 45]
[1 2 6 ... 5040 40320 362880]

#arg最大最小值索引
a = tf.range(1,10)
tf.print(tf.argmax(a))
tf.print(tf.argmin(a))

#tf.math.top_k可以用于对张量排序
a = tf.constant([1,3,7,5,4,8])
values,indices = tf.math.top_k(a,3,sorted=True) #将a中的元素按照从大到小排序，然后取前三位
tf.print(values) 
tf.print(indices)
#结果：
[8 7 5]
[5 2 3]

```

##### 3、矩阵运算

> 矩阵运算包括**：矩阵乘法，矩阵转置，矩阵逆，矩阵求迹，矩阵范数，矩阵行列式，矩阵求特征值，矩阵分解**等运算。大部分和矩阵有关的运算都在`tf.linalg`子包中。

```python
#矩阵乘法
a = tf.constant([[1,2],[3,4]])
b = tf.constant([[2,0],[0,2]])
a@b  #等价于tf.matmul(a,b)

#矩阵转置
a = tf.constant([[1.0,2],[3,4]])
tf.transpose(a)
#结果
<tf.Tensor: shape=(2, 2), dtype=float32, numpy=
array([[1., 3.],
       [2., 4.]], dtype=float32)>

#矩阵逆，必须为tf.float32或tf.double类型
a = tf.constant([[1.0,2],[3.0,4]],dtype = tf.float32)
tf.linalg.inv(a)

#矩阵求trace
a = tf.constant([[1.0,2],[3,4]])
tf.linalg.trace(a)

#矩阵求范数
a = tf.constant([[1.0,2],[3,4]])
tf.linalg.norm(a)

#矩阵行列式
a = tf.constant([[1.0,2],[3,4]])
tf.linalg.det(a)

#矩阵特征值
tf.linalg.eigvalsh(a)

#矩阵qr分解
a  = tf.constant([[1.0,2.0],[3.0,4.0]],dtype = tf.float32)
q,r = tf.linalg.qr(a)

#矩阵svd分解
a  = tf.constant([[1.0,2.0],[3.0,4.0]],dtype = tf.float32)
v,s,d = tf.linalg.svd(a)
tf.matmul(tf.matmul(s,tf.linalg.diag(v)),d)
#利用svd分解可以在TensorFlow中实现主成分分析降维
```

##### 4、广播机制

- 1、如果张量的维度不同，将维度较小的张量进行扩展，直到两个张量的维度都一样。
- 2、如果两个张量在某个维度上的长度是相同的，或者其中一个张量在该维度上的长度为1，那么我们就说这两个张量在该维度上是相容的。
- 3、如果两个张量在所有维度上都是相容的，它们就能使用广播。
- 4、广播之后，每个维度的长度将取两个张量在该维度长度的较大值。
- 5、在任何一个维度上，如果一个张量的长度为1，另一个张量长度大于1，那么在该维度上，就好像是对第一个张量进行了复制。

> `tf.broadcast_to` 以显式的方式按照广播机制扩展张量的维度。

```python
a = tf.constant([1,2,3])
b = tf.constant([[0,0,0],[1,1,1],[2,2,2]])
b + a  #等价于 b + tf.broadcast_to(a,b.shape)

#计算广播后计算结果的形状，静态形状，TensorShape类型参数
tf.broadcast_static_shape(a.shape,b.shape)

#计算广播后计算结果的形状，动态形状，Tensor类型参数
c = tf.constant([1,2,3])
d = tf.constant([[1],[2],[3]])
tf.broadcast_dynamic_shape(tf.shape(c),tf.shape(d))
#广播效果
c+d #等价于 tf.broadcast_to(c,[3,3]) + tf.broadcast_to(d,[3,3])
```