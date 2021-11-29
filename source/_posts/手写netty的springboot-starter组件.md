---
title: 手写netty的springboot-starter组件
date: 2021-09-07 08:57:28
tags:
  - Netty
  - SpringBoot
categories:
  - 后端
description: 从九个细节考虑秒杀系统的设计。
top_img: https://file.buildworld.cn/img/20210910153801.png
cover: https://file.buildworld.cn/img/20210907092140.png
comments: false
---

> 本文的工作分为两个部分，一个是研究 Netty 框架的使用，实现服务端和客户之间的通信，另一个是将服务端和客户端的 Netty 打包成 springboot-starter 组件（两个组件：rpc-server-start,rpc-client-start），方便直接引入 starter 去实现 Netty 的相关功能。
>
> 项目地址：[https://gitee.com/mi_chong/NettyDemo](https://gitee.com/mi_chong/NettyDemo)

## 一、Netty

**1、NIO 是一种非阻塞 IO**

- 单线程可以连接多个客户端。
- 选择器可以实现但线程管理多个 Channel，新建的通道都要向选择器注册。
- 一个 SelectionKey 键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。
- selector 进行 select()操作可能会产生阻塞，但是可以设置阻塞时间，并且可以用 wakeup()唤醒 selector，所以 NIO 是非阻塞 IO。

### 2、Netty 模型 selector 模式

- NIO 采用多线程的方式可以同时使用多个 selector
- 通过绑定多个端口的方式，使得一个 selector 可以同时注册多个 ServerSocketServer
- 单个线程下只能有一个 selector，用来实现 Channel 的匹配及复用

![](https://file.buildworld.cn/img/20210827203719.png)

**半包问题**

> TCP/IP 在发送消息的时候，可能会拆包，这就导致接收端无法知道什么时候收到的数据是一个完整的数据。在传统的 BIO 中在读取不到数据时会发生阻塞，但是 NIO 不会。
>
> 为了解决 NIO 的半包问题，Netty 在 Selector 模型的基础上，提出了**reactor 模式**，从而解决客户端请求在服务端不完整的问题。

### 3、netty 模型 reactor 模式

> 在 selector 的基础上解决了半包问题。
> ![](https://file.buildworld.cn/img/20210827205913.png)

Netty 服务端代码：https://gitee.com/mi_chong/NettyDemo/tree/master/rpc-server-start

Netty 客户端代码：https://gitee.com/mi_chong/NettyDemo/tree/master/rpc-client-start

## 二、自定义 springboot-starter 组件

### 1、介绍

> SpringBoot 中的 starter 是一种非常重要的机制，能够抛弃以前繁杂的配置，将其统一集成进 starter，应用者只需要在 maven 中引入 starter 依赖，SpringBoot 就能自动扫描到要加载的信息并启 动相应的默认配置。starter 让我们摆脱了各种依赖库的处理，需要配置各种信息的困扰。

> SpringBoot 会自动通过 classpath 路径下的类发现需要的 Bean，并注册进 IOC 容器。SpringBoot 提供 了针对日常企业应用研发各种场景的**spring-boot-starter**依赖模块。所有这些依赖模块都遵循着约定成俗的默认配置，并允许我们调整这些配置，即遵循“**约定大于配置**”的理念。

> 以 rpc-server-start 来看，如下图：
> ![](https://file.buildworld.cn/img/20210906210553.png)
>
> 1、属性配置类
>
> 2、业务逻辑执行类
>
> 3、自动配置类
>
> 4、spring.factories 文件（里面指定了自动配置类 3 的位置）

### 2、NettyProperties.java——属性配置类

> 在使用 Spring 官方的 Starter 时通常可以在 application.properties 中来配置参数覆盖掉默认的值

```java
@SuppressWarnings("serial")
@ConfigurationProperties(prefix = "netty.server")
public class NettyProperties implements Serializable {
    // 地址
    private String hostname = "127.0.0.1";
    // 端口
    private Integer host = 8082;
    // 主线程数目
    private Integer bossGroupCounts = 1;
    // 工作线程数目
    private Integer workGroupCounts = 200;

    public NettyProperties() {

    }
    get...
    set...
}
```

### 3、NettyServer.java——业务操作类

> 在这里使用上面的属性配置类

```java
/**
 * 服务启动监听器
 *
 * @author MiChong
 * @date 2021-08-27 21:12
 */

@Slf4j
public class NettyServer {

    private NettyProperties nettyProperties;

    public NettyServer() {
    }

    public NettyServer(NettyProperties nettyProperties) {
        this.nettyProperties = nettyProperties;
    }

    public void start() {
        InetSocketAddress socketAddress = new InetSocketAddress(nettyProperties.getHostname(), nettyProperties.getHost());
        //new 一个主线程组
        EventLoopGroup bossGroup = new NioEventLoopGroup(nettyProperties.getBossGroupCounts());
        //new 一个工作线程组
        EventLoopGroup workGroup = new NioEventLoopGroup(nettyProperties.getWorkGroupCounts());
        ServerBootstrap bootstrap = new ServerBootstrap()
                .group(bossGroup, workGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ServerChannelInitializer())
                .localAddress(socketAddress)
                //设置队列大小
                .option(ChannelOption.SO_BACKLOG, 1024)
                // 两小时内没有数据的通信时,TCP会自动发送一个活动探测数据报文
                .childOption(ChannelOption.SO_KEEPALIVE, true);
        //绑定端口,开始接收进来的连接
        try {
            ChannelFuture future = bootstrap.bind(socketAddress).sync();
            log.info("服务器启动开始监听端口: {}", socketAddress.getPort());
            future.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            log.error("服务器开启失败", e);
        } finally {
            //关闭主线程组
            bossGroup.shutdownGracefully();
            //关闭工作线程组
            workGroup.shutdownGracefully();
        }
    }
}
```

### 4、NettyServerAutoConfiguration.java--自动配置 NettyServer 类

```java
/**
 * @author MiChong
 * @date 2021-09-06 14:24
 */
@Configuration
@EnableConfigurationProperties(NettyProperties.class)
//当类路径下有指定的类为true
@ConditionalOnClass(NettyServer.class)
@ConditionalOnProperty(prefix = "netty.server", value = "enabled", matchIfMissing = true)
public class NettyServerAutoConfiguration {

    @Autowired
    private NettyProperties nettyProperties;

    @Bean
    @ConditionalOnMissingBean(NettyServer.class)
    public NettyServer nettyServer() {
        NettyServer nettyServer = new NettyServer(nettyProperties);
        return nettyServer;
    }

}

```

### 5、spring.factories 文件

```
/META-INF/spring.factories文件放在/src/main/resources目录下
注意：META-INF是自己手动创建的目录，spring.factories也是自己手动创建的文件，在该文件中配置自己的自动配置类。
org.springframework.boot.autoconfigure.EnableAutoConfiguration=cn.buildworld.netty.server.start.config.NettyServerAutoConfiguration
```

### 6、项目打包

**最后，将项目打包 mvn clean install**
**下面链接是引入组件的 springboot 项目**

[https://gitee.com/mi_chong/NettyDemo/tree/master/WebStart](https://gitee.com/mi_chong/NettyDemo/tree/master/WebStart)

```xml
<!--在我们的springboot项目中引入打包好的starter组件-->
<dependency>
    <groupId>cn.buildworld</groupId>
    <artifactId>rpc-server-start</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

**在 springboot 中配置 applicaiton.yml**

```yaml
netty:
  server:
    host: 9000
```

### 7、如何使用 netty server

> 在我们的 springboot 项目中引入 netty-starter 组件，需要在我们项目启动的时候，启动 netty 服务器，需要做一些处理。
>
> 需求就是，启动 springboot 项目--> 执行 netty 组件中的启动服务器方法，目前网上有 5 种方式在 springboot 启动时执行方法。在https://gitee.com/mi_chong/NettyDemo/tree/master/WebStart/src/main/java/cn/buildworld/netty/webstart/start目录下。

**推荐使用@PostConstruct 注解方式**

```java
/**
 * 将要执行的方法所在的类交个spring容器扫描(@Component),并且在要执行的方法上添加@PostConstruct注解或者静态代码块执行
 *
 * @author MiChong
 * @date 2021-09-06 15:38
 */
@Component
public class NettyServerStart {
    @Autowired
    private NettyServer nettyServer;

    @PostConstruct
    public void start() {
        System.out.println(" 使用@PostConstruct注解 ");
        // 开启一个线程去启动Netty服务，防止阻塞springboot启动
        new Thread(() -> {
            nettyServer.start();
        }).start();
    }
}
```

### 8、如何使用 netty client

#### 8.1 引入依赖

```xml
<dependency>
     <groupId>cn.buildworld</groupId>
     <artifactId>rpc-client-start</artifactId>
     <version>1.0-SNAPSHOT</version>
</dependency>
```

#### 8.2 自动导入 Bean

```java
/**
 * @author MiChong
 * @date 2021-09-06 10:01
 */
@RestController
@Slf4j
public class UserController {

    // 注入Bean
    @Autowired
    private NettyClientUtil nettyClientUtil;

    @PostMapping("/helloNetty")
    public ResponseResult helloNetty(@RequestParam String msg) {
        return nettyClientUtil.sendMessage(msg);
    }
}
```

### 9、测试

#### 9.1、请求接口，发送数据

![](https://file.buildworld.cn/img/20210907093445.png)

#### 9.2、客户端发送消息

![](https://file.buildworld.cn/img/20210907093612.png)

#### 9.3、服务端接收消息并返回消息给客户端

![](https://file.buildworld.cn/img/20210907093657.png)
