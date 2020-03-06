---
title: Python版本的OpenCV安装
date: 2020-03-05 12:51:20
tags: 
- Python
- OpenCV
categories:
- 人工智能
- 编程语言
description: 📢 一般正统的CV开发方式都是C++开发的但是那种方式环境搭建比较复杂
cover: https://file.buildworld.cn/img/opencv.png

---
###  第一步、安装Anaconda
https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/

### 第二步、 创建虚拟环境
```shell
conda create --name opencv-env python=3.7
```
###  第三步、激活虚拟环境，也就是进入到虚拟环境中去
```shell
activate opencv-env
```
###  第四步、安装opencv+contrib
```shell
pip install numpy scipy matplotlib scikit-learn jupyter
pip install opencv-contrib-python
```
###  第五步、测试
```python
import cv2
cv2.__version__
```
