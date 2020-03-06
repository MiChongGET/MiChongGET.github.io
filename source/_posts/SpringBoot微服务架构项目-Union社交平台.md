---
title: SpringBoot微服务架构项目--Union社交平台
date: 2019-02-10 18:08:53
tags:
- SpringBoot
- SpringCloud
- SpringDataJpa
- Docker
categories:
- 后端
- 微服务架构
- 服务器
- 系统
- 运维

cover: https://file.buildworld.cn/img/6dc7724665fb0fa34d8d2872f6b40d22_224859_0cf8011f_759177.png
---


![](https://file.buildworld.cn/img/6dc7724665fb0fa34d8d2872f6b40d22_224859_0cf8011f_759177.png)
[Gitee项目地址](https://gitee.com/union_project/union_parent)
#### 前言
> 本项目是采用Spring全家桶的java后端框架，采用目前WEB端比较流行的前后端分离的开发方式，后端采用微服务架构思想，将业务各个拆分出来，通过SpringCloud微服务框架将各个微服务业务连接起来，使得项目业务之间独立运行，在服务部署和运行时不会相互影响。

#### 一、系统概况

##### 1、系统技术栈示意图

![系统技术栈示意图](http://myfile.buildworld.cn/224859_801038ac_759177.jpeg)
##### 2、后端系统架构图
![后端系统架构图](http://myfile.buildworld.cn/微服务架构图新.png)

##### 3、后台微服务系统

微服务系统 | 名称 | 端口
---|---|---
union_config | 配置服务器 | 12000
union_eureka | Eureka服务器 |8110
union-base   | 基础服务器 |9001
union-recruit| 招聘服务器 |9002
union-qa     | 用户问题服务器|9003
union-article|文章服务器|9004
union-gathering|用户活动服务器|9005
union-spit|用户吐槽服务器|9006
union-search|ES搜索服务器|9007
union-user|用户服务器|9008
union-friend|交友服务器|9009
union-manager|管理员管理服务器|9010
union-web|用户管理服务器|9011
union_rabbitmqtest|RM测试服务器|8002
union_ai|Ai人工智能服务器|未开发

#### 二、SpringCloud使用说明
##### 1、主要框架
- 服务发现——Netflix Eureka
- 服务调用——Netflix Feign
- 熔断器——Netflix Hystrix
- 服务网关——Netflix Zuul
- 分布式配置——Spring Cloud Config
- 消息总线 —— Spring Cloud Bus

##### 注意一下Cloud版本

```
Release Train	Boot Version
Greenwich       2.1.x
Finchley        2.0.x
Edgware         1.5.x
Dalston         1.5.x
```

##### 2、服务发现组件--Eureka

```
  Eureka是Netflix开发的服务发现框架，SpringCloud将它集成在自己的子项目
spring-cloud-netflix中，实现SpringCloud的服务发现功能。Eureka包含两个组件：
  Eureka Server和Eureka Client。
    
  Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进行注
册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点
的信息可以在界面中直观的看到。

  Eureka Client是一个java客户端，用于简化与Eureka Server的交互，客户端同时也
就别一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会
向Eureka Server发送心跳,默认周期为30秒，如果Eureka Server在多个心跳周期内没有
接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除(默认90
秒)。

  Eureka Server之间通过复制的方式完成数据的同步，Eureka还提供了客户端缓存机
制，即使所有的Eureka Server都挂掉，客户端依然可以利用缓存中的信息消费其他服务
的API。综上，Eureka通过心跳检查、客户端缓存等机制，确保了系统的高可用性、灵活
性和可伸缩性。
```

> 下面是项目全部跑起来的结果

![Eureka服务器](https://images.gitee.com/uploads/images/2019/0202/224900_36c8b6c3_759177.jpeg)

##### 3、服务间调用组件--Feign

```
  Feign是简化Java HTTP客户端开发的工具（java-to-httpclient-binder），它的灵感
来自于Retrofit、JAXRS-2.0和WebSocket。Feign的初衷是降低统一绑定Denominator到
HTTP API的复杂度，不区分是否为restful。
```
##### 4、熔断器Hystrix

```
  在微服务架构中通常会有多个服务层调用，基础服务的故障可能会导致级联故障，进而
造成整个系统不可用的情况，这种现象被称为服务雪崩效应。服务雪崩效应是一种
因“服务提供者”的不可用导致“服务消费者”的不可用,并将不可用逐渐放大的过程。

  Hystrix 能使你的系统在出现依赖服务失效的时候，通过隔离系统所依赖的服务，防
止服务级联失败，同时提供失败回退机制，更优雅地应对失效，并使你的系统能更快地
从异常中恢复。
```
##### 5、微服务网关Zuul

> Zuul是Netflix开源的微服务网关，他可以和Eureka,Ribbon,Hystrix等组件配合使
用。
Zuul组件的核心是一系列的过滤器，这些过滤器可以完成以下功能：

-  身份认证和安全: 识别每一个资源的验证要求，并拒绝那些不符的请求
-  审查与监控：
-  动态路由：动态将请求路由到不同后端集群
-  压力测试：逐渐增加指向集群的流量，以了解性能
-  负载分配：为每一种负载类型分配对应容量，并弃用超出限定值的请求
-  静态响应处理：边缘位置进行响应，避免转发到内部集群
-  多区域弹性：跨域AWS Region进行请求路由，旨在实现ELB(ElasticLoad Balancing)使用多样化

##### 6、微服务配置文件集中管理Spring Cloud Config
```
  在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所
以需要分布式配置中心组件。在Spring Cloud中，有分布式配置中心组件spring cloud
config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库
中。在spring cloud config 组件中，分两个角色，一是config server，二是config
client。
  Config Server是一个可横向扩展、集中式的配置服务器，它用于集中管理应用程序各个
环境下的配置，默认使用Git存储配置文件内容，也可以使用SVN存储，或者是本地文件
存储。
  Config Client是Config Server的客户端，用于操作存储在Config Server中的配置内容。
  微服务在启动时会请求Config Server获取配置文件的内容，请求到后再启动容器。
详细内容看在线文档： https://springcloud.cc/spring-cloud-config.html
```
##### 7、消息总线组件SpringCloudBus
> 可以用于动态修改各个微服务系统的配置文件，而不要重新启动微服务

#### 三、系统运维
##### 1、新建一个Mysql服务器容器
```
docker run -di --name=union_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root centos/mysql-57-centos7
```
##### 2、新建一个redis服务器
```
docker run -di --name=union_redis -p 6379:6379 redis
```
##### 3、elasticsearch容器
```
docker run -di --name=es -p 5601:5601 -p 9200:9200 nshou/elasticsearch-kibana
带挂载文件的创建方式
docker run -di --name=union_es -p 9200:9200 -p 9300:9300 -v /usr/local/es/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml elasticsearch:5.6.8
```
> head安装
```
docker -di --name=union_eshead -p 9100:9100 mobz/elasticsearch-head:5
```
##### 4、导出某个容器
> 导出某个容器，非常简单，使用docker export命令，语法：docker export $container_id > 容器快照名
##### 5、导入某个容器--docker import命令

> 有了容器快照之后，我们可以在想要的时候随时导入。导入快照使用docker import命令。
例如我们可以使用cat centos.tar | docker import - my/centos:v888 导入容器快照作为镜像

>镜像保存/载入：docker load/docker save；将一个镜像导出为文件，再使用docker load命令将文件导入为一个镜像，会保存该镜像的的所有历史记录。比docker export命令导出的文件大，很好理解，因为会保存镜像的所有历史记录。
容器导入/导出：docker import/docker export；将一个容器导出为文件，再使用docker import命令将容器导入成为一个新的镜像，但是相比docker save命令，容器文件会丢失所有元数据和历史记录，仅保存容器当时的状态，相当于虚拟机快照。


##### 6、rabbitMq创建
```shell
docker run -di --name=union_rm -p 5671:5617 -p 5672:5672 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq:management
```
##### 7、创建私有仓库容器

```
docker run -di --name=registry -p 5000:5000 registry
{"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"],"insecure-registries":["192.168.255.128:5000"]}
刷新配置systemctl daemon-reload
通过Maven插件自动部署。
对于数量众多的微服务，手动部署无疑是非常麻烦的做法，并且容易出错。所以我们这
里学习如何自动部署，这也是企业实际开发中经常使用的方法。
```
##### 8、Maven插件自动部署步骤：
###### （1）修改宿主机的docker配置，让其可以远程访问
```shell
vi /lib/systemd/system/docker.service
```
>其中ExecStart=后添加配置 -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

###### （2）修改pom.xml文件，添加插件
 ```xml
 <build>
        <finalName>config</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- docker的maven插件，官网 https://github.com/spotify/docker-maven-plugin -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.13</version>
                <configuration>
                    <imageName>192.168.255.128:5000/${project.artifactId}:${project.version}</imageName>
                    <baseImage>jdk1.8</baseImage>
                    <entryPoint>["java", "-jar","/${project.build.finalName}.jar"]</entryPoint>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                    <dockerHost>http://192.168.255.128:2375</dockerHost>
                </configuration>
            </plugin>
        </plugins>
    </build>
 ```
##### 9、Gogs安装与配置

```shell
$ docker pull gogs/gogs
$ mkdir -p /var/gogs
$ docker run -di --name=gogs -p 10022:22 -p 3000:3000 -v /var/gogsdata:/data gogs/gogs
```
##### 10、容器管理工具Rancher
>Rancher是一个开源的企业级全栈化容器部署及管理平台。Rancher为容器提供一揽
子基础架构服务：CNI兼容的网络服务、存储服务、主机管理、负载均衡、防护墙……
Rancher让上述服务跨越公有云、私有云、虚拟机、物理机环境运行，真正实现一键式应
用部署和管理。

```shell
docker run -d --name=rancher --restart=always -p 9090:8080 rancher/server
```
##### 11、influxDB监控
> influxDB是一个分布式时间序列数据库。cAdvisor仅仅显示实时信息，但是不存储
监视数据。因此，我们需要提供时序数据库用于存储cAdvisor组件所提供的监控信息，
以便显示除实时信息之外的时序数据。


```shell
docker run -di -p 8083:8083 -p 8086:8086 --expose 8090 --expose 8099 --name influxsrv tutum/influxdb
```
端口概述： 8083端口:web访问端口 8086:数据写入端口

###### 常用操作
- 创建数据库
```
CREATE DATABASE "cadvisor"
```
##### 12、cAdvisor
> Google开源的用于监控基础设施应用的工具，它是一个强大的监控工具，不需要任
何配置就可以通过运行在Docker主机上的容器来监控Docker容器，而且可以监控Docker
主机。更多详细操作和配置选项可以查看Github上的cAdvisor项目文档。


```shell
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:rw --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --publish=8080:8080 --detach=true --link influxsrv:influxsrv --name=cadvisor google/cadvisor -storage_driver=influxdb -storage_driver_db=union-db -storage_driver_host=influxsrv:8086
```
##### 13、Grafana
> Grafana是一个可视化面板（Dashboard），有着非常漂亮的图表和布局展示，功
能齐全的度量仪表盘和图形编辑器。支持Graphite、zabbix、InfluxDB、Prometheus和
OpenTSDB作为数据源。
Grafana主要特性：灵活丰富的图形化选项；可以混合多种风格；支持白天和夜间模式；
多个数据源。

```shell
docker run -d -p 3001:3000 -e INFLUXDB_HOST=influxsrv -e INFLUXDB_PORT=8086 -e INFLUXDB_NAME=cadvisor -e INFLUXDB_USER=root -e INFLUXDB_PASS=root --link influxsrv:influxsrv --name grafana grafana/grafana
```

