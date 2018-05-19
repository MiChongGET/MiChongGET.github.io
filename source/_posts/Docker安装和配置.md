---
title: 'Docker安装和配置'
date: 2018-05-19 16:43:46
tags: 
- Docker
- Linux
categories:
- 运维
- 服务器
---

#### 前言：

Docker这个名称大家估计已经不陌生了吧，在我大三的时候就听说过这个，也研究过一阵子，也就知道点皮毛，现在由于线上服务器资源紧张，本地搭建虚拟机也过于浪费，所以这段时间好好研究一下Docker技术，也为后面的后端开发提供各种环境保障，比如服务器集群，es，Nginx等等。

![](https://ws1.sinaimg.cn/large/005EneYkgy1frgtavax6jj30cr079mxp.jpg)

从上面的图像就可以看到，一个大鲸鱼上面有好多集装箱，大鲸鱼就好像是我们的产品（大容器），集装箱就是一个个服务器。

下面看看传统虚拟机和Docker架构的区别：

![](https://ws1.sinaimg.cn/large/005EneYkgy1frgto9v4xaj30ok0dtwen.jpg)

从上面的两张图片可以看出，传统的虚拟机架构和Docker体系架构的区别是，在传统的虚拟机中有一层虚拟机操作系统，而Docker却没有。所以Docker的启动速度和存储空间远远优于传统的虚拟机。

下面大家看看Docker官方的架构图：

![](https://ws1.sinaimg.cn/large/005EneYkgy1frgyjprvcbj30qm0e640h.jpg)

#### windows安装

##### 1、使用的软件

http://mirrors.aliyun.com/docker-toolbox/windows/

![](https://ws1.sinaimg.cn/large/005EneYkgy1frh21puistj30hg03uq2v.jpg)

docker-for-windows适合于在win10上面使用，默认使用的是win10自带的虚拟机软件Hyper，直接安装即可，提供官方的账号登录。

docker-toolbox加载boot2docker.iso镜像到virtualbox中去，通过xshell等连接软件即可连接到docker系统。使用这种方式注意，主要是这个镜像是放在github上面的，国内通常下载不下来，因为软件第一次打开的时候回去联网查询最新的版本并且下载下来，国内网通常导致连接超时，无法下载镜像，进而导致软件无法使用，也就无法使用docker了。

`Docker Quickstart Terminal`启动后会复制`C:\Users\Administrator\.docker\machine\cache`下的镜像`boot2docker.iso`到`C:\Users\Administrator\.docker\machine\machines\default`下面。

检测到默认的镜像不是最新版本的，需要到<https://github.com/boot2docker/boot2docker/releases>下载最新的，并复制到`C:\Users\Administrator\.docker\machine\cache`目录下。

![](https://ws1.sinaimg.cn/large/005EneYkgy1frh28u5bwcj30ra0dtwf3.jpg)

##### 2、安装步骤

我的是在windows上面安装的，大家如果有需要去看看这这两篇博客，下面都是基于docker-toolbox来安装的：

https://blog.csdn.net/tina_ttl/article/details/51372604

https://blog.csdn.net/zistxym/article/details/42918339

#### Linux安装

##### 1、Ubuntu 14.04 16.04 (使用apt-get进行安装)

```
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

##### 2、CentOS 7 (使用yum进行安装)

```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ee.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

##### 安装校验

```
root@iZbp12adskpuoxodbkqzjfZ:$ docker version
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

