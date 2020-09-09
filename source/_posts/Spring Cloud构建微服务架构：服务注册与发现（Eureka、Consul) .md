---
layout: '[layout]'
title: Spring Cloud构建微服务架构：服务注册与发现（Eureka、Consul)
date: 2018-04-21 13:43:50
tags:
- SpringBoot
- SpringCloud
- Java
- 微服务架构
categories:
- 后端
---

#### 一、Spring Cloud Eureka
> Spring Cloud Eureka是Spring Cloud Netflix项目下的服务治理模块。而Spring Cloud Netflix项目是Spring Cloud的子项目之一，主要内容是对Netflix公司一系列开源产品的包装，它为Spring Boot应用提供了自配置的Netflix OSS整合。通过一些简单的注解，开发者就可以快速的在应用中配置一下常用模块并构建庞大的分布式系统。它主要提供的模块包括：服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）等。

##### 1、创建“服务注册中心”-
添加依赖
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```
> 比较繁琐，可以直接通过idea添加
![](https://ws1.sinaimg.cn/large/005EneYkgy1fqi3remmeej30rj07vjry.jpg)

##### 2、通过注解开启服务

```
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}
}
```

##### 3、修改配置文件

```
server.port=110

eureka.instance.hostname=localhost
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
```
##### 4、创建“服务提供方”（client）

###### 添加依赖

```
<parent> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
###### 添加配置

```
spring.application.name=eureka-client
server.port=2001
eureka.client.serviceUrl.defaultZone=http://localhost:110/eureka/

```


###### 在application中添加注解

```
@SpringBootApplication
@EnableDiscoveryClient
public class ServiceConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceConsumerApplication.class, args);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		// Do any additional configuration here
		return builder.build();
	}
}
```

###### 在控制器中使用

```
@RestController
public class DcController {

    @Autowired
    DiscoveryClient discoveryClient;

    @GetMapping("/dc")
    public String dc() {
        String services = "Services: " + discoveryClient.getServices();
        System.out.println(services);
        return services;
    }

}
```
打开连接http://localhost:110

![](https://ws1.sinaimg.cn/large/005EneYkgy1fqi7jqltmzj31gl0okgnp.jpg)

##### 5、创建eureka消费者
> eureka消费者和提供者结构配置一样

```
<!--消费提供者提供的接口-->
@RestController
public class IndexController {

    @Autowired
    LoadBalancerClient loadBalancerClient;
    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/consumer")
    public String dc() {
        ServiceInstance serviceInstance = loadBalancerClient.choose("eureka-client");
        String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/dc";
        System.out.println(url);

        ServiceInstance instance = loadBalancerClient.choose("eureka-consumer");
        System.out.println(instance.getHost()+":"+instance.getPort());
        return restTemplate.getForObject(url, String.class);
    }
}
```



#### 二、Spring Cloud Consul


```
Spring Cloud Consul项目是针对Consul的服务治理实现。Consul是一个分布式高可用的系统
，它包含多个组件，但是作为一个整体，在微服务架构中为我们的基础设施提供服务发现和服务配置的工具。
它包含了下面几个特性：

服务发现
健康检查
Key/Value存储
多数据中心
```
##### 1、添加依赖

```
<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-consul-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-consul-discovery</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```
##### 2、properties添加配置

```
spring.cloud.consul.host=localhost
spring.cloud.consul.port=8500
```

##### 3、开启服务
> 需要本地开启consul服务，去官网下载服务端软件

[https://www.consul.io/](https://www.consul.io/)

![](https://ws1.sinaimg.cn/large/005EneYkgy1fqiw3huac5j317l0gfn08.jpg)

> 从官网下载对应版本的服务端软件，Windows系统在当前的软件的目录下面打开cmd，并且输入consul agent -dev,即可开启服务

##### 4、开启spring boot项目

[输入http://localhost://8500](http://localhost://8500)即可打开UI界面

![image](https://ws1.sinaimg.cn/large/005EneYkgy1fqi9an22ntj315c0l40u3.jpg)



