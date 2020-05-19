---
title: TensorFlow2.X学习笔记(7)--TensorFlow中阶API之losses、metrics、optimizers、callbacks
date: 2020-05-18 15:59:29
tags:
- Python
- TensorFlow
categories:
- 深度学习
description: 一般来说，监督学习的目标函数由损失函数和正则化项组成。
top_img: https://ae01.alicdn.com/kf/H97dae6f0ba1246cbaf21c697b8bd3e73y.jpg
cover: https://ae01.alicdn.com/kf/H24c9452e63dd4cc08266332d7111b080Z.jpg
---

## 一、损失函数

**一般来说，监督学习的目标函数由损失函数和正则化项组成。（Objective = Loss + Regularization）**

- 对于二分类模型，通常使用的是二元交叉熵损失函数 binary_crossentropy。
- 对于多分类模型，如果label是类别序号编码的，则使用类别交叉熵损失函数 categorical_crossentropy。如果label进行了one-hot编码，则需要使用稀疏类别交叉熵损失函数 sparse_categorical_crossentropy。



### 1、内置损失函数

**内置的损失函数一般有类的实现和函数的实现两种形式。**

> 常用的内置损失函数

- `mean_squared_error`（平方差误差损失，用于回归，简写为 mse, 类实现形式为 MeanSquaredError 和 MSE）
- `mean_absolute_error` (绝对值误差损失，用于回归，简写为 mae, 类实现形式为 MeanAbsoluteError 和 MAE)
- `mean_absolute_percentage_error` (平均百分比误差损失，用于回归，简写为 mape, 类实现形式为 MeanAbsolutePercentageError 和 MAPE)
- `Huber`(Huber损失，只有类实现形式，用于回归，介于mse和mae之间，对异常值比较鲁棒，相对mse有一定的优势)
- `binary_crossentropy`(二元交叉熵，用于二分类，类实现形式为 BinaryCrossentropy)
- `categorical_crossentropy`(类别交叉熵，用于多分类，要求label为onehot编码，类实现形式为 CategoricalCrossentropy)
- `sparse_categorical_crossentropy`(稀疏类别交叉熵，用于多分类，要求label为序号编码形式，类实现形式为 SparseCategoricalCrossentropy)
- `hinge`(合页损失函数，用于二分类，最著名的应用是作为支持向量机SVM的损失函数，类实现形式为 Hinge)
- kld(相对熵损失，也叫KL散度，常用于最大期望算法EM的损失函数，两个概率分布差异的一种信息度量。`类实现形式为` KLDivergence 或 KLD)
- `cosine_similarity`(余弦相似度，可用于多分类，类实现形式为 CosineSimilarity)

### 2、 自定义损失函数

> 自定义损失函数接收两个张量`y_true`,`y_pred`作为输入参数，并输出一个标量作为损失函数值。

####  函数形式

```python
def focal_loss(gamma=2., alpha=.25):

    def focal_loss_fixed(y_true, y_pred):
        pt_1 = tf.where(tf.equal(y_true, 1), y_pred, tf.ones_like(y_pred))
        pt_0 = tf.where(tf.equal(y_true, 0), y_pred, tf.zeros_like(y_pred))
        loss = -tf.sum(alpha * tf.pow(1. - pt_1, gamma) * tf.log(1e-07+pt_1)) \
           -tf.sum((1-alpha) * tf.pow( pt_0, gamma) * tf.log(1. - pt_0 + 1e-07))
        return loss
    return focal_loss_fixed
```

#### 类形式

```python
class FocalLoss(losses.Loss):

    def __init__(self,gamma=2.0,alpha=0.25):
        self.gamma = gamma
        self.alpha = alpha

    def call(self,y_true,y_pred):

        pt_1 = tf.where(tf.equal(y_true, 1), y_pred, tf.ones_like(y_pred))
        pt_0 = tf.where(tf.equal(y_true, 0), y_pred, tf.zeros_like(y_pred))
        loss = -tf.sum(self.alpha * tf.pow(1. - pt_1, self.gamma) * tf.log(1e-07+pt_1)) \
           -tf.sum((1-self.alpha) * tf.pow( pt_0, self.gamma) * tf.log(1. - pt_0 + 1e-07))
        return loss
```



## 二、评估指标metrics

**损失函数除了作为模型训练时候的优化目标，也能够作为模型好坏的一种评价指标。**

### 1、常用的内置评估指标

- `MeanSquaredError`（平方差误差，用于回归，可以简写为MSE，函数形式为mse）
- `MeanAbsoluteError` (绝对值误差，用于回归，可以简写为MAE，函数形式为mae)
- `MeanAbsolutePercentageError` (平均百分比误差，用于回归，可以简写为MAPE，函数形式为mape)
- `RootMeanSquaredError` (均方根误差，用于回归)
- `Accuracy` (准确率，用于分类，可以用字符串"Accuracy"表示，Accuracy=(TP+TN)/(TP+TN+FP+FN)，要求y_true和y_pred都为类别序号编码)
- `Precision` (精确率，用于二分类，Precision = TP/(TP+FP))
- `Recall` (召回率，用于二分类，Recall = TP/(TP+FN))
- `TruePositives` (真正例，用于二分类)
- `TrueNegatives` (真负例，用于二分类)
- `FalsePositives` (假正例，用于二分类)
- `FalseNegatives` (假负例，用于二分类)
- `AUC`(ROC曲线(TPR vs FPR)下的面积，用于二分类，直观解释为随机抽取一个正样本和一个负样本，正样本的预测值大于负样本的概率)
- `CategoricalAccuracy`（分类准确率，与Accuracy含义相同，要求y_true(label)为onehot编码形式）
- `SparseCategoricalAccuracy` (稀疏分类准确率，与Accuracy含义相同，要求y_true(label)为序号编码形式)
- `MeanIoU` (Intersection-Over-Union，常用于图像分割)
- `TopKCategoricalAccuracy` (多分类TopK准确率，要求y_true(label)为onehot编码形式)
- `SparseTopKCategoricalAccuracy` (稀疏多分类TopK准确率，要求y_true(label)为序号编码形式)
- `Mean` (平均值)
- `Sum` (求和)

### 2、自定义评估指标

**KS指标适合二分类问题，其计算方式为 `KS=max(TPR-FPR).`**

> TPR=TP/(TP+FN) , FPR = FP/(FP+TN)
>
> TPR曲线实际上就是正样本的累积分布曲线(CDF)，FPR曲线实际上就是负样本的累积分布曲线(CDF)。

**KS指标就是正样本和负样本累积分布曲线差值的最大值。**

#### 函数形式的自定义评估指标

```python
#函数形式的自定义评估指标
@tf.function
def ks(y_true,y_pred):
    y_true = tf.reshape(y_true,(-1,))
    y_pred = tf.reshape(y_pred,(-1,))
    length = tf.shape(y_true)[0]
    t = tf.math.top_k(y_pred,k = length,sorted = False)
    y_pred_sorted = tf.gather(y_pred,t.indices)
    y_true_sorted = tf.gather(y_true,t.indices)
    cum_positive_ratio = tf.truediv(
        tf.cumsum(y_true_sorted),tf.reduce_sum(y_true_sorted))
    cum_negative_ratio = tf.truediv(
        tf.cumsum(1 - y_true_sorted),tf.reduce_sum(1 - y_true_sorted))
    ks_value = tf.reduce_max(tf.abs(cum_positive_ratio - cum_negative_ratio)) 
    return ks_value

y_true = tf.constant([[1],[1],[1],[0],[1],[1],[1],[0],[0],[0],[1],[0],[1],[0]])
y_pred = tf.constant([[0.6],[0.1],[0.4],[0.5],[0.7],[0.7],[0.7],
                      [0.4],[0.4],[0.5],[0.8],[0.3],[0.5],[0.3]])
tf.print(ks(y_true,y_pred))
```

#### 类形式的自定义评估指标

```python
#类形式的自定义评估指标
class KS(metrics.Metric):

    def __init__(self, name = "ks", **kwargs):
        super(KS,self).__init__(name=name,**kwargs)
        self.true_positives = self.add_weight(
            name = "tp",shape = (101,), initializer = "zeros")
        self.false_positives = self.add_weight(
            name = "fp",shape = (101,), initializer = "zeros")

    @tf.function
    def update_state(self,y_true,y_pred):
        y_true = tf.cast(tf.reshape(y_true,(-1,)),tf.bool)
        y_pred = tf.cast(100*tf.reshape(y_pred,(-1,)),tf.int32)

        for i in tf.range(0,tf.shape(y_true)[0]):
            if y_true[i]:
                self.true_positives[y_pred[i]].assign(
                    self.true_positives[y_pred[i]]+1.0)
            else:
                self.false_positives[y_pred[i]].assign(
                    self.false_positives[y_pred[i]]+1.0)
        return (self.true_positives,self.false_positives)

    @tf.function
    def result(self):
        cum_positive_ratio = tf.truediv(
            tf.cumsum(self.true_positives),tf.reduce_sum(self.true_positives))
        cum_negative_ratio = tf.truediv(
            tf.cumsum(self.false_positives),tf.reduce_sum(self.false_positives))
        ks_value = tf.reduce_max(tf.abs(cum_positive_ratio - cum_negative_ratio)) 
        return ks_value
    
    
y_true = tf.constant([[1],[1],[1],[0],[1],[1],[1],[0],[0],[0],[1],[0],[1],[0]])
y_pred = tf.constant([[0.6],[0.1],[0.4],[0.5],[0.7],[0.7],
                      [0.7],[0.4],[0.4],[0.5],[0.8],[0.3],[0.5],[0.3]])

myks = KS()
myks.update_state(y_true,y_pred)
tf.print(myks.result())
```



## 三、优化器optimizers

### 1、优化器

**优化器主要使用`apply_gradients`方法传入变量和对应梯度从而来对给定变量进行迭代，或者直接使用minimize方法对目标函数进行迭代优化。**

```python
# 使用optimizer.apply_gradients
optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)  
while tf.constant(True): 
        with tf.GradientTape() as tape:
            y = a*tf.pow(x,2) + b*x + c
        dy_dx = tape.gradient(y,x)
        optimizer.apply_gradients(grads_and_vars=[(dy_dx,x)])
```

```python
# 使用model.fit
model.compile(optimizer = 
              tf.keras.optimizers.SGD(learning_rate=0.01),loss = myloss)
history = model.fit(tf.zeros((100,2)),
                    tf.ones(100),batch_size = 1,epochs = 10)  #迭代1000次
```

```python
# 使用optimizer.minimize
def train(epoch = 1000):  
    for _ in tf.range(epoch):  
        optimizer.minimize(f,[x])
    tf.print("epoch = ",optimizer.iterations)
    return(f())
```

###  2、内置优化器

深度学习优化算法大概经历了 SGD -> SGDM -> NAG ->Adagrad -> Adadelta(RMSprop) -> Adam -> Nadam 这样的发展历程。

在keras.optimizers子模块中，它们基本上都有对应的类的实现。

- `SGD`, 默认参数为纯SGD, 设置momentum参数不为0实际上变成SGDM, 考虑了一阶动量, 设置 nesterov为True后变成NAG，即 Nesterov Acceleration Gradient，在计算梯度时计算的是向前走一步所在位置的梯度。
- `Adagrad`, 考虑了二阶动量，对于不同的参数有不同的学习率，即自适应学习率。缺点是学习率单调下降，可能后期学习速率过慢乃至提前停止学习。
- `RMSprop`, 考虑了二阶动量，对于不同的参数有不同的学习率，即自适应学习率，对Adagrad进行了优化，通过指数平滑只考虑一定窗口内的二阶动量。
- `Adadelta`, 考虑了二阶动量，与RMSprop类似，但是更加复杂一些，自适应性更强。
- `Adam`, 同时考虑了一阶动量和二阶动量，可以看成RMSprop上进一步考虑了Momentum。
- `Nadam`, 在Adam基础上进一步考虑了 Nesterov Acceleration。



## 四、回调函数callbacks                       

tf.keras的回调函数实际上是一个类，一般是在model.fit时作为参数指定，用于控制在训练过程开始或者在训练过程结束，在每个epoch训练开始或者训练结束，在每个batch训练开始或者训练结束时执行一些操作，例如收集一些日志信息，改变学习率等超参数，提前终止训练过程等等。

### 1、内置回调函数

- `BaseLogger`： 收集每个epoch上metrics在各个batch上的平均值，对stateful_metrics参数中的带中间状态的指标直接拿最终值无需对各个batch平均，指标均值结果将添加到logs变量中。该回调函数被所有模型默认添加，且是第一个被添加的。
- `History`： 将BaseLogger计算的各个epoch的metrics结果记录到history这个dict变量中，并作为model.fit的返回值。该回调函数被所有模型默认添加，在BaseLogger之后被添加。
- `EarlyStopping`： 当被监控指标在设定的若干个epoch后没有提升，则提前终止训练。
- `TensorBoard`： 为Tensorboard可视化保存日志信息。支持评估指标，计算图，模型参数等的可视化。
- `ModelCheckpoint`： 在每个epoch后保存模型。
- `ReduceLROnPlateau`：如果监控指标在设定的若干个epoch后没有提升，则以一定的因子减少学习率。
- `TerminateOnNaN`：如果遇到loss为NaN，提前终止训练。
- `LearningRateScheduler`：学习率控制器。给定学习率lr和epoch的函数关系，根据该函数关系在每个epoch前调整学习率。
- `CSVLogger`：将每个epoch后的logs结果记录到CSV文件中。
- `ProgbarLogger`：将每个epoch后的logs结果打印到标准输出流中。

### 2、自定义回调函数

**可以使用`callbacks.LambdaCallback`编写较为简单的回调函数，也可以通过对`callbacks.Callback`子类化编写更加复杂的回调函数逻辑。**

