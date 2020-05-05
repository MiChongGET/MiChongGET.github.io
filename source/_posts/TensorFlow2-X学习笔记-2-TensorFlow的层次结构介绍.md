---
title: ' TensorFlow2.X学习笔记(2)--TensorFlow的层次结构介绍'
date: 2020-04-30 18:33:14
tags:
- Python
- TensorFlow
categories:
- 深度学习
description: TensorFlow中5个不同的层次结构：硬件层，内核层，低阶API，中阶API，高阶API。
top_img: https://ae01.alicdn.com/kf/H4ecd8f1e91f6439ba57e5b478ca746c8c.jpg
cover: https://ae01.alicdn.com/kf/H24c9452e63dd4cc08266332d7111b080Z.jpg
---


# TensorFlow的层次结构

TensorFlow中5个不同的层次结构：

`硬件层，内核层，低阶API，中阶API，高阶API`

<img src="https://ae01.alicdn.com/kf/Haf5b341a1ab342d2b56a5ec363e442770.jpg" style="zoom:70%;" />

- 最底层为硬件层，TensorFlow支持CPU、GPU或TPU加入计算资源池。
- 第二层为C++实现的内核，kernel可以跨平台分布运行。
- 第三层为Python实现的操作符，提供了封装`C++内核`的低级API指令，主要包括各种张量操作`算子、计算图、自动微分`. 如`tf.Variable,tf.constant,tf.function,tf.GradientTape,tf.nn.softmax...` 如果把模型比作一个房子，那么第三层API就是【模型之砖】。
- 第四层为Python实现的模型组件，对低级API进行了函数封装，主要包括各种`模型层，损失函数，优化器，数据管道，特征列`等等。 如tf.keras.layers,tf.keras.losses,tf.keras.metrics,tf.keras.optimizers,tf.data.DataSet,tf.feature_column... 如果把模型比作一个房子，那么第四层API就是【模型之墙】。
- 第五层为Python实现的模型成品，一般为按照OOP方式封装的高级API，主要为`tf.keras.models`提供的模型的类接口。 如果把模型比作一个房子，那么第五层API就是模型本身，即【模型之屋】。 


#### 低阶API示范

> 低阶API主要包括张量操作，计算图和自动微分。

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
    
  
#样本数量
n = 400

# 生成测试用数据集
X = tf.random.uniform([n,2],minval=-10,maxval=10) 
w0 = tf.constant([[2.0],[-1.0]])
b0 = tf.constant(3.0)
Y = X@w0 + b0 + tf.random.normal([n,1],mean = 0.0,stddev= 2.0)  # @表示矩阵乘法,增加正态扰动

#使用动态图调试

w = tf.Variable(tf.random.normal(w0.shape))
b = tf.Variable(0.0)

def train(epoches):
    for epoch in tf.range(1,epoches+1):
        with tf.GradientTape() as tape:
            #正向传播求损失
            Y_hat = X@w + b
            loss = tf.squeeze(tf.transpose(Y-Y_hat)@(Y-Y_hat))/(2.0*n)   

        # 反向传播求梯度
        dloss_dw,dloss_db = tape.gradient(loss,[w,b])
        # 梯度下降法更新参数
        w.assign(w - 0.001*dloss_dw)
        b.assign(b - 0.001*dloss_db)
        if epoch%1000 == 0:
            printbar()
            tf.print("epoch =",epoch," loss =",loss,)
            tf.print("w =",w)
            tf.print("b =",b)
            tf.print("")

train(5000)
```

##### 使用autograph机制转换成静态图加速

```python
w = tf.Variable(tf.random.normal(w0.shape))
b = tf.Variable(0.0)

@tf.function
def train(epoches):
    for epoch in tf.range(1,epoches+1):
        with tf.GradientTape() as tape:
            #正向传播求损失
            Y_hat = X@w + b
            loss = tf.squeeze(tf.transpose(Y-Y_hat)@(Y-Y_hat))/(2.0*n)   

        # 反向传播求梯度
        dloss_dw,dloss_db = tape.gradient(loss,[w,b])
        # 梯度下降法更新参数
        w.assign(w - 0.001*dloss_dw)
        b.assign(b - 0.001*dloss_db)
        if epoch%1000 == 0:
            printbar()
            tf.print("epoch =",epoch," loss =",loss,)
            tf.print("w =",w)
            tf.print("b =",b)
            tf.print("")
train(5000)
```



#### 中阶API示范

> TensorFlow的中阶API主要包括各种`模型层`，`损失函数`，`优化器`，`数据管道`，`特征列`等等。

**下面代码在GPU上面测试不通过，有人说可以在CPU上面跑通**

```python
# 要使用CPU则在代码最上面加上下面两行代码
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "-1"
```

```python
import tensorflow as tf
from tensorflow.keras import layers,losses,metrics,optimizers


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
    
    
   #样本数量
n = 800

# 生成测试用数据集
X = tf.random.uniform([n,2],minval=-10,maxval=10) 
w0 = tf.constant([[2.0],[-1.0]])
b0 = tf.constant(3.0)
Y = X@w0 + b0 + tf.random.normal([n,1],mean = 0.0,stddev= 2.0)  # @表示矩阵乘法,增加正态扰动

#构建输入数据管道
ds = tf.data.Dataset.from_tensor_slices((X,Y)) \
     .shuffle(buffer_size = 1000).batch(100) \
     .prefetch(tf.data.experimental.AUTOTUNE)  

#定义优化器
optimizer = optimizers.SGD(learning_rate=0.001)

linear = layers.Dense(units = 1)
linear.build(input_shape = (2,)) 

@tf.function
def train(epoches):
    for epoch in tf.range(1,epoches+1):
        L = tf.constant(0.0) #使用L记录loss值
        for X_batch,Y_batch in ds:
            with tf.GradientTape() as tape:
                Y_hat = linear(X_batch)
                loss = losses.mean_squared_error(tf.reshape(Y_hat,[-1]),tf.reshape(Y_batch,[-1]))
            grads = tape.gradient(loss,linear.variables)
            optimizer.apply_gradients(zip(grads,linear.variables))
            L = loss

        if(epoch%100==0):
            printbar()
            tf.print("epoch =",epoch,"loss =",L)
            tf.print("w =",linear.kernel)
            tf.print("b =",linear.bias)
            tf.print("")

train(500)

# 运行结果
================================================================================11:54:07
epoch = 100 loss = 4.13672256
w = [[1.9903785]
 [-0.987648785]]
b = [2.40720725]

================================================================================11:54:10
epoch = 200 loss = 4.51364088
w = [[1.99168313]
 [-0.987588942]]
b = [2.90266657]

================================================================================11:54:12
epoch = 300 loss = 4.33221674
w = [[1.99366784]
 [-0.98791343]]
b = [3.00248265]

================================================================================11:54:15
epoch = 400 loss = 3.12717295
w = [[1.98944652]
 [-0.986787856]]
b = [3.02270603]

================================================================================11:54:17
epoch = 500 loss = 3.830863
w = [[1.99245548]
 [-0.987303793]]
b = [3.02663541]

```

#### 高阶API示范

> TensorFlow的高阶API主要为tf.keras.models提供的模型的类接口。
>
> 使用Keras接口有以下3种方式构建模型：使用**Sequential**按层顺序构建模型，使用**函数式API**构建任意结构模型，**继承Model基类**构建自定义模型。



##### (1)、使用Sequential按层顺序构建模型

```python
import tensorflow as tf
from tensorflow.keras import models,layers,optimizers

#样本数量
n = 800

# 生成测试用数据集
X = tf.random.uniform([n,2],minval=-10,maxval=10) 
w0 = tf.constant([[2.0],[-1.0]])
b0 = tf.constant(3.0)

Y = X@w0 + b0 + tf.random.normal([n,1],mean = 0.0,stddev= 2.0)  # @表示矩阵乘法,增加正态扰动

tf.keras.backend.clear_session()

linear = models.Sequential()
linear.add(layers.Dense(1,input_shape =(2,)))
linear.summary()

### 使用fit方法进行训练
linear.compile(optimizer="adam",loss="mse",metrics=["mae"])
linear.fit(X,Y,batch_size = 20,epochs = 200)  

tf.print("w = ",linear.layers[0].kernel)
tf.print("b = ",linear.layers[0].bias)

### 运行结果
Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
dense (Dense)                (None, 1)                 3         
=================================================================
Total params: 3
Trainable params: 3
Non-trainable params: 0
_________________________________________________________________
Train on 800 samples

............
。。。略过。。。
............

Epoch 197/200

 20/800 [..............................] - ETA: 0s - loss: 4.1910 - mae: 1.5875
360/800 [============>.................] - ETA: 0s - loss: 3.7929 - mae: 1.5485
660/800 [=======================>......] - ETA: 0s - loss: 4.1543 - mae: 1.6328
800/800 [==============================] - 0s 171us/sample - loss: 4.0170 - mae: 1.6049
Epoch 198/200

 20/800 [..............................] - ETA: 0s - loss: 5.7438 - mae: 1.8000
340/800 [===========>..................] - ETA: 0s - loss: 4.1317 - mae: 1.5987
700/800 [=========================>....] - ETA: 0s - loss: 4.1257 - mae: 1.6281
800/800 [==============================] - 0s 151us/sample - loss: 4.0162 - mae: 1.6045
Epoch 199/200

 20/800 [..............................] - ETA: 0s - loss: 5.3425 - mae: 1.6237
340/800 [===========>..................] - ETA: 0s - loss: 3.9504 - mae: 1.5925
660/800 [=======================>......] - ETA: 0s - loss: 4.1211 - mae: 1.6202
800/800 [==============================] - 0s 162us/sample - loss: 4.0166 - mae: 1.6045
Epoch 200/200

 20/800 [..............................] - ETA: 0s - loss: 4.0931 - mae: 1.7218
380/800 [=============>................] - ETA: 0s - loss: 3.9471 - mae: 1.6048
700/800 [=========================>....] - ETA: 0s - loss: 3.8557 - mae: 1.5794
800/800 [==============================] - 0s 160us/sample - loss: 4.0185 - mae: 1.6047
w= [[1.99929214]
 [-0.96500361]]
b= [3.04800749]

```

##### (2)、继承Model基类构建自定义模型

```python
# 要使用CPU则在代码最上面加上下面两行代码
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "-1"
```



```python
import tensorflow as tf
from tensorflow.keras import models,layers,optimizers,losses,metrics


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
    
#样本数量
n = 800

# 生成测试用数据集
X = tf.random.uniform([n,2],minval=-10,maxval=10) 
w0 = tf.constant([[2.0],[-1.0]])
b0 = tf.constant(3.0)

Y = X@w0 + b0 + tf.random.normal([n,1],mean = 0.0,stddev= 2.0)  # @表示矩阵乘法,增加正态扰动

ds_train = tf.data.Dataset.from_tensor_slices((X[0:n*3//4,:],Y[0:n*3//4,:])) \
     .shuffle(buffer_size = 1000).batch(20) \
     .prefetch(tf.data.experimental.AUTOTUNE) \
     .cache()

ds_valid = tf.data.Dataset.from_tensor_slices((X[n*3//4:,:],Y[n*3//4:,:])) \
     .shuffle(buffer_size = 1000).batch(20) \
     .prefetch(tf.data.experimental.AUTOTUNE) \
     .cache()
    
    
tf.keras.backend.clear_session()
# 自定义模型
class MyModel(models.Model):
    def __init__(self):
        super(MyModel, self).__init__()

    def build(self,input_shape):
        self.dense1 = layers.Dense(1)   
        super(MyModel,self).build(input_shape)

    def call(self, x):
        y = self.dense1(x)
        return(y)

model = MyModel()
model.build(input_shape =(None,2))
model.summary()


### 自定义训练循环(专家教程)
optimizer = optimizers.Adam()
loss_func = losses.MeanSquaredError()

train_loss = tf.keras.metrics.Mean(name='train_loss')
train_metric = tf.keras.metrics.MeanAbsoluteError(name='train_mae')

valid_loss = tf.keras.metrics.Mean(name='valid_loss')
valid_metric = tf.keras.metrics.MeanAbsoluteError(name='valid_mae')


@tf.function
def train_step(model, features, labels):
    with tf.GradientTape() as tape:
        predictions = model(features)
        loss = loss_func(labels, predictions)
    gradients = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))

    train_loss.update_state(loss)
    train_metric.update_state(labels, predictions)

@tf.function
def valid_step(model, features, labels):
    predictions = model(features)
    batch_loss = loss_func(labels, predictions)
    valid_loss.update_state(batch_loss)
    valid_metric.update_state(labels, predictions)


@tf.function
def train_model(model,ds_train,ds_valid,epochs):
    for epoch in tf.range(1,epochs+1):
        for features, labels in ds_train:
            train_step(model,features,labels)

        for features, labels in ds_valid:
            valid_step(model,features,labels)

        logs = 'Epoch={},Loss:{},MAE:{},Valid Loss:{},Valid MAE:{}'

        if  epoch%100 ==0:
            printbar()
            tf.print(tf.strings.format(logs,
            (epoch,train_loss.result(),train_metric.result(),valid_loss.result(),valid_metric.result())))
            tf.print("w=",model.layers[0].kernel)
            tf.print("b=",model.layers[0].bias)
            tf.print("")

        train_loss.reset_states()
        valid_loss.reset_states()
        train_metric.reset_states()
        valid_metric.reset_states()

train_model(model,ds_train,ds_valid,400)


#### 运行结果
Model: "my_model"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
dense (Dense)                multiple                  3         
=================================================================
Total params: 3
Trainable params: 3
Non-trainable params: 0
_________________________________________________________________
================================================================================18:17:08
Epoch=100,Loss:67.9662247,MAE:6.36445856,Valid Loss:65.3885117,Valid MAE:6.17629
w= [[1.65662384]
 [-1.01629746]]
b= [1.92026019]

================================================================================18:17:16
Epoch=200,Loss:36.144165,MAE:4.0302186,Valid Loss:35.4477425,Valid MAE:4.01481533
w= [[1.99435031]
 [-1.00531375]]
b= [3.02523756]

================================================================================18:17:25
Epoch=300,Loss:25.3425236,MAE:3.20795441,Valid Loss:25.308445,Valid MAE:3.25203133
w= [[1.99592912]
 [-1.00504756]]
b= [3.08958364]

================================================================================18:17:34
Epoch=400,Loss:19.9554043,MAE:2.79780984,Valid Loss:20.2524834,Valid MAE:2.87172508
w= [[1.99595356]
 [-1.00504148]]
b= [3.08971953]

```