---
title: BCI--Python-EEG工具库MNE
date: 2020-06-12 20:52:14
top: True
tags:
- BCI
- Python
- MNE
categories:
- 脑机交互
description: Python-EEG工具库MNE的使用
top_img: https://ae01.alicdn.com/kf/H32f1a2189cf54b20a201315213d6c44fa.jpg
cover: https://ae01.alicdn.com/kf/H32f1a2189cf54b20a201315213d6c44fa.jpg
comments: false
---
## Python-EEG工具库MNE

### 一、环境配置

#### 安装MNE-python

```shell
pip install -U mne
```

#### 测试

```python
import mne
from mne.datasets import sample
import matplotlib.pyplot as plt

# sample的存放地址，下面语法是从网络中获取数据集
# data_path = sample.data_path()
# 该fif文件存放地址
fname = 'F:/data/MNE-sample-data/MEG/sample/sample_audvis_raw.fif'

"""
如果上述给定的地址中存在该文件，则直接加载本地文件，
如果不存在则在网上下载改数据
"""
raw = mne.io.read_raw_fif(fname)

"""
案例：
获取10-20秒内的良好的MEG数据

# 根据type来选择 那些良好的MEG信号(良好的MEG信号，通过设置exclude="bads") channel,
结果为 channels所对应的的索引
"""

picks = mne.pick_types(raw.info, meg=True, exclude='bads')
t_idx = raw.time_as_index([10., 20.])
data, times = raw[picks, t_idx[0]:t_idx[1]]
plt.plot(times,data.T)
plt.title("Sample channels")
plt.show()
```

- 出现下面的图片，说明环境搭建成功了

![](https://file.buildworld.cn/img/20200611205438.png)

### 二、MNE中数据结构Raw及其用法简介

**Raw对象主要用来存储连续型数据，核心数据为n_channels和times,也包含Info对象。**

#### 1、打开raw数据

```python
# 引入python库
import mne
from mne.datasets import sample
import matplotlib.pyplot as plt

# sample的存放地址，下面为从网络地址下载
#data_path = sample.data_path()

# 打开本地的数据，该fif文件存放地址
fname = data_path + '/MEG/sample/sample_audvis_raw.fif'

"""
如果上述给定的地址中存在该文件，则直接加载本地文件，
如果不存在则在网上下载改数据
"""
raw = mne.io.read_raw_fif(fname)
```



```python
# 通常raw的数据访问方式如下：
data, times = raw[picks, time_slice]

# picks:是根据条件挑选出来的索引；
# time_slice:时间切片

# 想要获取raw中所有数据，以下两种方式均可：
data,times=raw[:]
data,times=raw[:,:]
```

#### 2、sfreq：采样频率

```python
"""
sfreq：采样频率

raw返回所选信道以及时间段内的数据和时间点，
分别赋值给data以及times（即raw对象返回的是两个array）
"""
sfreq=raw.info['sfreq']
data,times=raw[:5,int(sfreq*1):int(sfreq*3)]
plt.plot(times,data.T)
plt.title("Sample channels")
```

![](https://file.buildworld.cn/img/20200612101529.png)

#### 3、绘制各通道的功率谱密度

```python
raw.plot_psd()
plt.show()
```

![](https://file.buildworld.cn/img/20200612101711.png)

#### 4、绘制SSP矢量图

```python
raw.plot_projs_topomap()
plt.show()
```

![](https://file.buildworld.cn/img/20200612101820.png)



#### 5、绘制通道频谱图作为topography

```python
raw.plot_psd_topo()
plt.show()
```

![](https://file.buildworld.cn/img/20200612102005.png)

#### 6、绘制电极位置

```python
raw.plot_sensors()
plt.show()
```

![](https://file.buildworld.cn/img/20200612102107.png)

### 三、MNE从头创建Raw对象

#### 1、简单介绍

**在实际过程中，有时需要从头构建数据来创建Raw对象。**
**方式：通过`mne.io.RawArray`类来手动创建Raw**

> 注：使用mne.io.RawArray创建Raw对象时，其构造函数只接受矩阵和info对象。

数据对应的单位：

- `V: eeg, eog, seeg, emg, ecg, bio, ecog`
- `T: mag`
- `T/m: grad`
- `M: hbo, hbr`
- `Am: dipole`
- `AU: misc`

构建一个Raw对象时，需要准备两种数据，一种是**data**数据，一种是**Info**数据，

data数据是一个二维数据，形状为`(n_channels,n_times)`

#### 2、包装数据

```python
"""
生成一个大小为5x1000的二维随机数据
其中5代表5个通道，1000代表times
"""
data = np.random.randn(5,1000)

"""
创建info结构,
内容包括：通道名称和通道类型
设置采样频率为:sfreq=100
"""
info = mne.create_info(
    ch_names=['MEG1','MEG2','EEG1','EEG2','EOG'],
    ch_types=['grad','grad','eeg','eeg','eog'],
    sfreq=100
)

custom_raw = mne.io.RawArray(data,info)
```

#### 3、对图形进行缩放

```python
"""
对图形进行缩放

对于实际的EEG / MEG数据，应使用不同的比例因子。
对通道eeg、grad，eog的数据进行2倍缩小
"""

scalings = {'eeg':2,'grad':2,'eog':2}

#scalings = 'auto' 设置自动缩放

custom_raw.plot(n_channels=5,
                scalings=scalings,
                title='Data from arrays',
                show=True,
                block=True)
plt.show()
```

![](https://file.buildworld.cn/img/20200612110956.png)

### 四、MNE中数据结构Epoch

**从连续的脑电图信号中提取一些特定时间窗口的信号，这些时间窗口可以称作为`epochs`.**

####  1、创建Epochs对象方式有三种：

- (1)通过Raw对象和事件事件点(event times)-
- (2)通过读取.fif文件数据生成Epoch对象
- (3)通过mne.EpochsArray从头创建Epoch对象

#### 2、读取fif文件创建Epoch对象

```python 
步骤：
1）读取fif文件，构建raw对象;
2）创建event对象；
3）创建epoch对象；
4）对epoch进行叠加平均得到evoked对象；
5）绘制evoked。


import mne
from mne import io
from mne.datasets import sample

# 从网络中下载
data_path = sample.data_path()

raw_fname = data_path + '/MEG/sample/sample_audvis_filt-0-40_raw.fif'
event_fname = data_path + '/MEG/sample/sample_audvis_filt-0-40_raw-eve.fif'
event_id, tmin, tmax = 1, -0.2, 0.5

# 读取fif文件,创建raw对象
raw = io.read_raw_fif(raw_fname)
# 读取包含event的fif文件，创建event对象
events = mne.read_events(event_fname)

"""
 挑选通道:EEG + MEG - bad channels 
"""
raw.info['bads'] += ['MEG 2443', 'EEG 053']  # bads + 2 more
picks = mne.pick_types(raw.info, meg=True, eeg=False, stim=True, eog=True,
                       exclude='bads')

# 读取Epoch数据
epochs = mne.Epochs(raw, events, event_id, tmin, tmax, proj=True,
                    picks=picks, baseline=(None, 0), preload=True,
                    reject=dict(grad=4000e-13, mag=4e-12, eog=150e-6))
"""
对epochs数据进行求平均获取诱发响应
"""
evoked = epochs.average()

evoked.plot(time_unit='s')
plt.show()
```

![](https://file.buildworld.cn/img/20200612134440.png)                             

#### 3、从头创建Epoch对象

**Epochs对象是一种将连续数据表示为时间段集合的方法**

> 方式：利用`mne.EpochsArray`创建Epochs对象，创建时直接构建numpy数组即可，数组的形状必须是`(n_epochs, n_chans, n_times)`

**数据对应的单位：**

- V: eeg, eog, seeg, emg, ecg, bio, ecog
- T: mag
- T/m: grad
- M: hbo, hbr
- Am: dipole
- AU: misc

##### 第一步：构建数据

```python
第一步：构建数据

构建一个大小为10x5x200的三维数组，数组中数据是随机数；
第一维数据表示：10 epochs
第二维数据表示：5 channels
第三维数据表示：2 seconds per epoch


# 采样频率
sfreq = 100
data = np.random.randn(10, 5, sfreq * 2)

# 创建一个info结构
info = mne.create_info(
    ch_names=['MEG1', 'MEG2', 'EEG1', 'EEG2', 'EOG'],
    ch_types=['grad', 'grad', 'eeg', 'eeg', 'eog'],
    sfreq=sfreq
)
```

##### 第二步：构建events

```python
在创建Epochs对象时，必须提供一个"events"数组，

事件(event)描述的是某一种波形(症状)的起始点，其为一个三元组，形状为(n_events,3):
第一列元素以整数来描述的事件起始采样点；
第二列元素对应的是当前事件来源的刺激通道(stimulus channel)的先前值(previous value),该值大多数情况是0；
第三列元素表示的是该event的id。

events = np.array([
    [0, 0, 1],
    [1, 0, 2],
    [2, 0, 1],
    [3, 0, 2],
    [4, 0, 1],
    [5, 0, 2],
    [6, 0, 1],
    [7, 0, 2],
    [8, 0, 1],
    [9, 0, 2],
])
```

```python
设置事件的id

如果是dict，则以后可以使用这些键访问关联的事件。示例：dict（听觉=1，视觉=3）
如果是int，将创建一个id为string的dict。
如果是列表，则使用列表中指定ID的所有事件。
如果没有，则所有事件都将与一起使用，并使用与事件id整数对应的字符串整数名称创建dict。

# 创建event id，受试者或者微笑或者皱眉
event_id = dict(smiling=1, frowning=2)
"""
tmin:event开始前的时间，如果未指定，则默认为0
"""
# 设置事件开始前时间为-0.1s
tmin = -0.1
```

##### 第三步：创建epochs对象

```python
"""
利用mne.EpochsArray创建epochs对象
"""
custom_epochs = mne.EpochsArray(data, info, events, tmin, event_id)
print(custom_epochs)
# 绘制
_ = custom_epochs['smiling'].average().plot(time_unit='s')
```

![](https://file.buildworld.cn/img/20200612153814.png)

#### 4、查看epoch对象

**epochs对象类似于mne.io.Raw对象，也具有info属性和event属性。**

**可以通过下面两种方式来查看epoch内的event相关信息**

```python
print(epochs.events[:3])
print(epochs.event_id)

print(epochs[1:5])
print(epochs['Auditory/Right'])

# 搜索
print(epochs['Right'])
print(epochs['Right', 'Left'])
```

#### 5、epoch平均叠加

**通过调用`mne.Epochs.average()`方法可返回Evoked对象，average()方法可以通过参数指定需要的信道。**

```python
ev_right = epochs['Right'].average()
ev_left = epochs['Left'].average()

f, axs = plt.subplots(3, 2, figsize=(10, 5))
_ = f.suptitle('Left / Right auditory', fontsize=20)
_ = ev_left.plot(axes=axs[:, 0], show=False, time_unit='s')
_ = ev_right.plot(axes=axs[:, 1], show=False, time_unit='s')
plt.tight_layout()
```

![](https://file.buildworld.cn/img/20200612162428.png)

#### 6、Epoch对象中的元数据(metadata)

有时候使用`mne`的`metadata`属性来存储相关数据特别有用，metadata使用`pandas.DataFrame`来封装数据。其中每一行对应一个epoch，每一列对应一个epoch的元数据属性。列必须包含字符串、整数或浮点数。



#### 7、Epochs数据可视化

`mne.Epochs.plot()`提供了一个交互式浏览器，当与关键字`block = True`结合使用时，允许手动拒绝。这将阻止脚本执行，直到关闭浏览器窗口。



```python
"""
加载数据，如果本地无该数据,
则从网络中下载
"""
data_path = op.join(mne.datasets.sample.data_path(), 'MEG', 'sample')

raw = mne.io.read_raw_fif(
    op.join(data_path, 'sample_audvis_filt-0-40_raw.fif'), preload=True)
# 设置event ID
event_id = {'auditory/left': 1, 'auditory/right': 2, 'visual/left': 3,
            'visual/right': 4, 'smiley': 5, 'button': 32}
events = mne.find_events(raw)
epochs = mne.Epochs(raw, events, event_id=event_id, tmin=-0.2, tmax=.5,
                    preload=True)
```

```python
# 绘制epochs
epochs.plot(block=True)
plt.show()
```

![](https://ae01.alicdn.com/kf/Hbd1728d2d0894d548c1a0ca2da6ed73fr.jpg)

**顶部的数字表示epoch的事件id。底部的数字是各个epoch的运行编号。**





### 五、Evoked结构

**Evoked potential(EP**)诱发电位或诱发反应是指在出现诸如闪光或纯音之类的刺激后，从人类或其他动物的神经系统，特别是大脑的特定部分记录的特定模式的电位。不同形式和类型的刺激会产生不同类型的电位。

诱发电位振幅往往较低，从小于1微伏到数微伏不等，而脑电图为数十微伏，肌电图为毫伏，心电图通常接近20毫伏。为了在EEG、ECG、EMG等生物信号和环境噪声的背景下解决这些低幅度电位，通常需要对信号进行平均。**信号被时间锁定在刺激上，大部分噪声是随机产生的，这样就可以通过对重复响应来平均掉噪声。**

**诱发电位(Evoked)结构主要用于存储实验期间的平均数据**，在MNE中，创建Evoked对象通常使用`mne.Epochs.average()`来平均epochs数据来实现。