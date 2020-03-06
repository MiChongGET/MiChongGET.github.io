---
layout: '[layout]'
title: Redis的使用--（二）与Spring的整合
date: 2018-01-31 16:19:10
tags:
- Redis
categories:
- 集群
top_img: https://file.buildworld.cn/img/310d0f7a390f1eadf08997f35c705757_u=3127962649,1328757895&fm=214&gp=0.jpg
cover: https://file.buildworld.cn/img/fe688b5e1c5bef2f800d93736aa3a430_834b4451313d3e9171afcf7841b3a80212eb507a6bee-XzTSuO_fw658.webp
---
##### 主题词：使用Jedis、项目整合Redis
* 项目中整合jedis和spring框架：
    * 设计一个相关接口(把String和Hash类型常用方法进行封装)
    * 完成两个相关实现类(jedisPool的实现和jedisCluster的实现：属性注入)
    * 完成spring-jedis.xml(将jedisPool的实现和jedisCluster的实现进行注入操作)
    * 具体内容参看代码实现

* 需求：在tt-common工程的src/test/java中完成Jedis的简单使用

1. 在tt-common工程引入jedis的依赖


```xml
        <!--redis客户端-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>
```

2. Jedis的三种使用方法


```java
@Test
    public void testJedis1(){
        Jedis jedis = new Jedis("10.31.152.30",6379);
        jedis.set("key","value");
        System.out.println(jedis.get("key"));
        jedis.close();
    }
```


```java
@Test
    public void testJedis2(){
        //获取jedis池
        JedisPool jedisPool = new JedisPool("10.31.152.30",6379);
        //获取Jedis对象
        Jedis jedis = jedisPool.getResource();
        jedis.set("key1","value1");
        System.out.println(jedis.get("key1"));
        //关闭连接
        jedis.close();
        jedisPool.close();
    }
```


```java
@Test
    public void testJedis3(){
        //创建集群节点集合
        Set<HostAndPort> nodes = new HashSet<HostAndPort>();
        //将6个节点加入到集合中
        nodes.add(new HostAndPort("10.31.152.30",7001));
        nodes.add(new HostAndPort("10.31.152.30",7002));
        nodes.add(new HostAndPort("10.31.152.30",7003));
        nodes.add(new HostAndPort("10.31.152.30",7004));
        nodes.add(new HostAndPort("10.31.152.30",7005));
        nodes.add(new HostAndPort("10.31.152.30",7006));
        //创建集群对象
        JedisCluster jedisCluster = new JedisCluster(nodes);
        //存入数据
        jedisCluster.set("key2","value2");
        System.out.println(jedisCluster.get("key2"));
        //关闭连接
        jedisCluster.close();
    }
```

3. 完成上述三种方法的使用后，将三种方法的工具类添加tt-common中

* 在tt-common中添加com.dhc.common.jedis包
* 添加三个工具类

```java
JedisClient.java
JedisClientCluster.java
JedisClientPool.java

JsonUtils.java
```

***

* 需求：Spring项目整合Redis

1. 创建spring-jedis.xml文件


```xml
<!-- 连接池版本 -->
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <!-- 最大连接数 -->
        <property name="maxTotal" value="30"/>
        <!-- 最大空闲连接数 -->
        <property name="maxIdle" value="10"/>
        <!-- 每次释放连接的最大数目 -->
        <property name="numTestsPerEvictionRun" value="1024"/>
        <!-- 释放连接的扫描间隔（毫秒） -->
        <property name="timeBetweenEvictionRunsMillis" value="30000"/>
        <!-- 连接最小空闲时间 -->
        <property name="minEvictableIdleTimeMillis" value="1800000"/>
        <!-- 连接空闲多久后释放, 当空闲时间>该值 且 空闲连接>最大空闲连接数 时直接释放 -->
        <property name="softMinEvictableIdleTimeMillis" value="10000"/>
        <!-- 获取连接时的最大等待毫秒数,小于零:阻塞不确定的时间,默认-1 -->
        <property name="maxWaitMillis" value="1500"/>
        <!-- 在获取连接的时候检查有效性, 默认false -->
        <property name="testOnBorrow" value="true"/>
        <!-- 在空闲时检查有效性, 默认false -->
        <property name="testWhileIdle" value="true"/>
        <!-- 连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true -->
        <property name="blockWhenExhausted" value="false"/>
    </bean>

    <bean id="jedisPool" class="redis.clients.jedis.JedisPool">
        <constructor-arg name="poolConfig" ref="jedisPoolConfig"/>
        <constructor-arg name="host" value="192.168.31.117"/>
        <constructor-arg name="port" value="6379"/>
    </bean>

    <bean id="jedisClientPool" class="com.dhc.common.jedis.JedisClientPool">
        <property name="jedisPool" ref="jedisPool"/>
    </bean>


    <!--集群版本-->
    <bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
        <constructor-arg name="nodes">
            <set>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="192.168.31.117"/>
                    <constructor-arg name="port" value="7001"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="192.168.31.117"/>
                    <constructor-arg name="port" value="7002"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="192.168.31.117"/>
                    <constructor-arg name="port" value="7003"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="192.168.31.117"/>
                    <constructor-arg name="port" value="7004"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="192.168.31.117"/>
                    <constructor-arg name="port" value="7005"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="192.168.31.117"/>
                    <constructor-arg name="port" value="7006"/>
                </bean>
            </set>
        </constructor-arg>
        <constructor-arg name="poolConfig" ref="jedisPoolConfig"/>
    </bean>
    <bean id="jedisClientCluster" class="com.dhc.common.jedis.JedisClientCluster">
        <property name="jedisCluster" ref="jedisCluster"/>
    </bean>
```

1. 使用时将类型注入可以选择其中一个版本
* 单机版的注入时，将spring-jedis.xml中的集群版本注释，注入接口JedisClient.java

* 集群版本注入时，将spring-jedis.xml中的单机版本注释，注入接口JedisClient.java

* 作业：完成首页门户的轮播图效果（redis集群）

***

* 需求：解决**查询缓存**问题与**同步缓存**问题

* 读数据规则（查询缓存问题）：先判断缓存中是否有要的数据
    * 若有，则直接加载
    * 若无，则去DB加载，并存入缓存中
    * 实际代码举例

```java
public List<TbContent> getContentListByCid(Long cid){
    try{
        //查询缓存，如果存在直接加载
        String json = jedisClient.hget("CONTENT_LIST",Long.toString(cid));
        if(StringUtils.isNotBlank(json)){
            List<TbContent> list = JsonUtils.jsonToList(json,TbContent.class);
            return list;
        }
    }catch(Exception e){
        e.printStackTrace();
    }
    //如果缓存中没有
    TbContentExample example = new TbContentExample();
    TbContentExample.Criteria criteria = example.createCriteria();
    criteria.andCategoryIdEqualTo(cid);
    List<TbContent> clist = contentMapper.selectByExample(example);
    
    try{
        //将查询出的数据存放到缓存中
        jedisClient.hset("CONTENT_LIST",Long.toString(cid),JsonUtils.objectToJson(clist));
        return clist;
    }catch(Exception e){
        e.printStackTrace();
    }
}
```


* 改数据规则（缓存同步问题）：
    * 直接失效缓存数据，再修改DB内容
    * 避免突发情况：避免DB修改成功，但由于网络或者其他问题导致缓存数据没有清理，造成了脏数据
    * 实际代码举例

```java
public int addContent(TbContent content){
    try{
        //缓存同步，删除缓存中对应的数据
        jedisClient.hdel("CONTENT_LIST",content.getId().toString());
    }catch(Exception e){
        e.printStackTrace();
    }
    content.setCreated(new Date());
    content.setUpdated(new Date());
    int count = contentMapper.insert(content);
    return count;
}
```