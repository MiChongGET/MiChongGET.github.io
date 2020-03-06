---
layout: '[layout]'
title: SpringBoot整合Dubbo
date: 2018-03-29 17:23:02
tags:
- Java
- SpringBoot
categories:
- 后端
cover: https://file.buildworld.cn/img/u=2551084964,3275034231&fm=26&gp=0.jpg
---
[官方Github地址](https://github.com/alibaba/dubbo-spring-boot-starter)

#### 一、服务端开发
###### 1、添加依赖

```
  <dependency>
      <groupId>com.alibaba.spring.boot</groupId>
      <artifactId>dubbo-spring-boot-starter</artifactId>
      <version>2.0.0</version>
  </dependency>
    
```

###### 2、在application.properties添加dubbo的相关配置信息

```
# Spring boot application
spring.application.name = dubbo-provider-demo
server.port = 9090
management.port = 9091

# Base packages to scan Dubbo Components (e.g., @Service, @Reference)
dubbo.scan.basePackages  = com.alibaba.boot.dubbo.demo.provider.service

# Dubbo Config properties
## ApplicationConfig Bean
dubbo.application.id = dubbo-provider-demo
dubbo.application.name = dubbo-provider-demo

## ProtocolConfig Bean
dubbo.protocol.id = dubbo
dubbo.protocol.name = dubbo
dubbo.protocol.port = 12345

## RegistryConfig Bean
dubbo.registry.id = my-registry
dubbo.registry.address = N/A

```
###### 3、接下来在Spring Boot Application的上添加@EnableDubboConfiguration，表示要开启dubbo功能. (dubbo provider服务可以使用或者不使用web容器)

```java
@SpringBootApplication
@EnableDubboConfiguration
public class DubboProviderLauncher {
  //...
}
```
###### 4、编写你的dubbo服务，只需要添加要发布的服务实现上添加@Service（import com.alibaba.dubbo.config.annotation.Service）注解，其中interfaceClass是要发布服务的接口.

```java
@Service(interfaceClass = IHelloService.class)
@Component
public class HelloServiceImpl implements IHelloService {
  //...
}
```
==注意：实体类必须实现Serializable接口==


#### 二、消费端的消费服务
###### 1、添加依赖

> 同上

###### 2、配置文件

```
# Spring boot application
spring.application.name = dubbo-provider-demo

# Dubbo Config properties
## ApplicationConfig Bean
dubbo.application.id = dubbo-consumer-demo
dubbo.application.name = dubbo-consumer-demo

## ProtocolConfig Bean
dubbo.protocol.id = dubbo
dubbo.protocol.name = dubbo
dubbo.protocol.port = 12345
```
###### 3、add @EnableDubboConfiguration on Spring Boot Application
```java
@SpringBootApplication
@EnableDubboConfiguration
public class DubboConsumerLauncher {
  //...
}
```
###### 4、使用

```java
@RestController
public class DubboConsumer {

    @Reference(url = "dubbo://127.0.0.1:20880")
    private HseCustomerService customerService;

    @GetMapping("list2")
    public Object getList(@RequestParam(defaultValue = "0")Integer fromId,
                          @RequestParam(defaultValue = "2")Integer limit ){

        //初始化page插件，传入分页参数
        PageHelper.startPage(fromId,limit);
        List<HseCustomer> list = customerService.getList();

        //包装想要返回的结果，包含多种信息
        PageInfo pageInfo = new PageInfo(list);

        return pageInfo;
    }
}
```
###### 5、成功效果

![](https://ws1.sinaimg.cn/large/005EneYkgy1fptt8cwrzxj30ol09bt90.jpg)