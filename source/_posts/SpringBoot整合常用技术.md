---
layout: '[layout]'
title: SpringBoot整合常用技术
date: 2018-05-17 10:56:03
description: springboot和一些常用的框架进行整合
tags: 
- SpringBoot
- MyBatis
- Redis
categories:
- 后端
cover: https://file.buildworld.cn/img/u=2551084964,3275034231&fm=26&gp=0.jpg
---



[模板地址](https://gitee.com/mi_chong/spring-boot-model)

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526542764168&di=307b509c8c315a84383917119b23eba5&imgtype=0&src=http%3A%2F%2Ftherealdanvega.com%2Fwp-content%2Fuploads%2F2015%2F11%2Fspring-boot-logo.png)

#### 前言

​	`Spring Boot是目前大火的Java后端框架，遵循着“约定大于配置”的规则，可以快速开发后台，摆脱SSM框架的各种xml配置，开箱即用，快速部署。依靠着spring的强大社区，框架中集成了各种优秀的第三方框架。`

#### 一、逆向生成model、mapper
##### maven的配置
##### 1、首先引入依赖

```
<!--整合mybatis-->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.1</version>
		</dependency>

		<!--mapper-->
		<dependency>
			<groupId>tk.mybatis</groupId>
			<artifactId>mapper-spring-boot-starter</artifactId>
			<version>2.0.0</version>
		</dependency>
		<!--pagehelper-->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.2.3</version>
			<exclusions>
				<exclusion>
					<groupId>org.mybatis.spring.boot</groupId>
					<artifactId>mybatis-spring-boot-starter</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
			<version>1.1.0</version>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
```
##### 2、build中添加插件

```
            <!--mybatis逆向生成插件-->
			<plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.2</version>
				<configuration>
				
			     	<configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
					<overwrite>true</overwrite>
					<verbose>true</verbose>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>mysql</groupId>
						<artifactId>mysql-connector-java</artifactId>
						<version>${mysql.version}</version>
					</dependency>
					<dependency>
						<groupId>tk.mybatis</groupId>
						<artifactId>mapper-generator</artifactId>
						<version>1.0.0</version>
					</dependency>

				</dependencies>
			</plugin>
```
##### 3、自定义一个MyMapper

```
import tk.mybatis.mapper.common.Mapper;
import tk.mybatis.mapper.common.MySqlMapper;
import tk.mybatis.mapper.common.base.delete.DeleteByPrimaryKeyMapper;
import tk.mybatis.mapper.common.condition.SelectByConditionMapper;
import tk.mybatis.mapper.common.ids.DeleteByIdsMapper;
import tk.mybatis.mapper.common.ids.SelectByIdsMapper;

/**
 * 继承自己的MyMapper
 *
 */
public interface MyMapper<T> extends
        Mapper<T>,
        MySqlMapper<T>,
        SelectByIdsMapper<T>,
        SelectByConditionMapper<T>,
        DeleteByIdsMapper<T>,
        DeleteByPrimaryKeyMapper<T> {
    //TODO
    //FIXME 特别注意，该接口不能被扫描到，否则会出错
}


```


##### 4、编写generator配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <!--配置环境的文件，数据库的信息-->
    <properties resource="application-dev.properties"/>

    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <!--继承自己的MyMapper-->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="cn.buildworld.sbtest.util.MyMapper"/>
        </plugin>

        <!--数据库的信息-->
        <jdbcConnection driverClass="${spring.datasource.driver-class-name}"
                        connectionURL="${spring.datasource.url}"
                        userId="${spring.datasource.username}"
                        password="${spring.datasource.password}">
        </jdbcConnection>

        <!--实体类生成的位置-->
        <javaModelGenerator targetPackage="cn.buildworld.sbtest.model" targetProject="src/main/java"/>

        <!--dao接口生成的位置-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources"/>

        <!--XML文件生成的位置-->
        <javaClientGenerator targetPackage="cn.buildworld.sbtest.mapper" targetProject="src/main/java"
                             type="XMLMAPPER"/>

        <!--数据库表的信息，%代表生成数据库中所有的表-->
        <table tableName="%">
            <!--mysql 配置-->
            <generatedKey column="id" sqlStatement="Mysql" identity="true"/>
            <!--oracle 配置-->
            <!--<generatedKey column="id" sqlStatement="select SEQ_{1}.nextval from dual" identity="false" type="pre"/>-->
        </table>
    </context>
</generatorConfiguration>
```

#### 二、基于mybatis的CRUD

##### 1、在Application启动文件中添加注解

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

@SpringBootApplication

/**
 * 扫描mapper包路径
 */
@MapperScan(basePackages = "cn.buildworld.sbtest.mapper")
public class SbtestApplication {
	public static void main(String[] args) {
		SpringApplication.run(SbtestApplication.class, args);
	}
}

```

###### 2、PageHelper分页插件的使用

```
 @Autowired
    private HseCustomerService hseCustomerService;
    @GetMapping("list")
    public Object getList(@RequestParam(defaultValue = "0")Integer fromId,
                          @RequestParam(defaultValue = "2")Integer limit ){

        //初始化page插件，传入分页参数
        PageHelper.startPage(fromId,limit);
        List<HseCustomer> list = hseCustomerService.getList();

        //包装想要返回的结果，包含多种信息
        PageInfo pageInfo = new PageInfo(list);

        return pageInfo;
    }
```
###### 3、在service实现类中使用事务

```
 @Override
    //事务--查询
    @Transactional(propagation = Propagation.SUPPORTS)
    public List<StudentInfo> getList() {

        return studentInfoMapper.selectByIds("123");

    }
    
    //事务--修改
    @Transactional(propagation = Propagation.REQUIRED)
    
```

#### 三、整合缓存Redis
##### 1、添加依赖

```
        <!--引入redis依赖-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```
##### 2、配置文件中添加配置（application.properties）

```
###########################
#                         #
#         Redis配置        #
#                         #
###########################
# Redis数据库索引（默认为0）
spring.redis.database=1
# Redis服务器地址
spring.redis.host=119.29.181.95
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=1000
# 连接池最大阻塞时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=10
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=2
# 连接超时时间
spring.redis.timeout=0
```
##### 3、使用redis

```
    /**
     * redis测试
     */

    @Autowired
    //spring-boot自带的模板
    private StringRedisTemplate redisTemplate;
    @GetMapping("redis")
    public Object getRedis(){

        //往redis数据库存入数据
        redisTemplate.opsForValue().set("name","michong");

        //读出redis中的数据
        return redisTemplate.opsForValue().get("name");

    }
```

#### 四、整合定时器
##### 1、在启动项加上注解

```
/**
 * 开启定时任务
 */
@EnableScheduling
public class SbtestApplication {
	public static void main(String[] args) {
		SpringApplication.run(SbtestApplication.class, args);
	}
}
```
##### 2、简单使用定时器

> 注意加上@Component和@Scheduled注解

```
/**
 * 定时任务
 *
 * @Author MiChong
 * @Email: 1564666023@qq.com
 * @Create 2018-03-28 9:02
 * @Version: V1.0
 */
@Component
public class TimeTesk {

    private static final SimpleDateFormat DATE_FORMAT = new SimpleDateFormat("HH:mm:ss");

    /**
     *    定义三秒执行的任务
     */
    @Scheduled(fixedRate = 3000)
    public void ShowTime(){
        System.out.println("现在时间："+DATE_FORMAT.format(new Date()));
    }


}
```

##### 3、使用cron表达式执行定时任务
[表达式地址](http://cron.qqe2.com/)

```
@Component
public class TimeTesk {

    private static final SimpleDateFormat DATE_FORMAT = new SimpleDateFormat("HH:mm:ss");

    /**
     * 从0开始，每隔三秒执行一次操作
     */
    @Scheduled(cron = "0/3 * * * * ? ")
    public void ShowTime(){
        System.out.println("现在时间："+DATE_FORMAT.format(new Date()));
    }


}

```

#### 五、整合异步任务
##### 1、启动项添加注解


```
/**
 * 开启异步任务
 */
@EnableAsync
public class SbtestApplication {
	public static void main(String[] args) {
		SpringApplication.run(SbtestApplication.class, args);
	}
}

```

##### 2、使用

> 在使用类上面加上@Component，在类中的方法上面加上@Async


#### 六、拦截器的使用
##### 1、首先创建一个configer类

```
@Configuration
public class WebMvcConfiger extends WebMvcConfigurerAdapter {

    /**
     * 重写父类的方法，在此处对拦截器进行设置
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        /**
         * 拦截器按照顺序执行,如果将one改成 * 则表示拦截所有的请求
         */
        registry.addInterceptor(new TwoInterCeptor()).addPathPatterns("/one/**")
                .addPathPatterns("/two/**");
        registry.addInterceptor(new OneInterCeptor()).addPathPatterns("/two/**");
        super.addInterceptors(registry);
    }
}

```
##### 2、新建拦截器

```
public class OneInterCeptor implements HandlerInterceptor {

    /**
     * 在请求处理之前进行调用（Controller方法调用之前）
     */
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        return true;
    }


    /**
     * 请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
     */
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }


    /**
     * 在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行
     * （主要是用于进行资源清理工作）
     */
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }


    public void returnErrorResponse(HttpServletResponse response)
            throws IOException, UnsupportedEncodingException {
        OutputStream out=null;
        try{
            response.setCharacterEncoding("utf-8");
            response.setContentType("text/json");
            out = response.getOutputStream();
            out.flush();
        } finally{
            if(out!=null){
                out.close();
            }
        }
    }
}

```






