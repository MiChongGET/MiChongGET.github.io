---
layout: '[layout]'
title: SpringBoot之API--Swagger2接口文档管理
date: 2018-03-29 17:25:18
tags:
- Java
- SpringBoot
categories:
- 后端
---
##### 1、添加依赖
```
        <!--Swagger2-->
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.2.2</version>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.2.2</version>
		</dependency>
		
		
```
##### 2、创建Swagger2配置类，和application处于同一级

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * API接口文档配置
 *
 * @Author MiChong
 * @Email: 1564666023@qq.com
 * @Create 2018-03-29 16:32
 * @Version: V1.0
 */

@Configuration
@EnableSwagger2
public class Swagger2 {
    
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.buildworld.sbtest.web"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .contact("MiChong")
                .version("1.0")
                .build();
    }

}

```
> 如上代码所示，通过@Configuration注解，让Spring来加载该类配置。再通过@EnableSwagger2注解来启用Swagger2。
再通过createRestApi函数创建Docket的Bean之后，apiInfo()用来创建该Api的基本信息（这些基本信息会展现在文档页面中）。select()函数返回一个ApiSelectorBuilder实例用来控制哪些接口暴露给Swagger来展现，本例采用指定扫描的包路径来定义，Swagger会扫描该包下所有Controller定义的API，并产生文档内容（除了被@ApiIgnore指定的请求）。


##### 3、添加文档内容
> 在完成了上述配置后，其实已经可以生产文档内容，但是这样的文档主要针对请求本身，而描述主要来源于函数等命名产生，对用户并不友好，我们通常需要自己增加一些说明来丰富文档内容。如下所示，我们通过@ApiOperation注解来给API增加说明、通过@ApiImplicitParams、@ApiImplicitParam注解来给参数增加说明。


```java
@Autowired
    private HseCustomerService hseCustomerService;

    @ApiOperation(value = "客户列表")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "fromId",value = "起始位置",required = false,dataType = "Integer"),
            @ApiImplicitParam(name = "limit",value = "每页显示条数",required = false,dataType = "Integer")
    })
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



#####  4、最后显示效果
本地的访问地址：
http://localhost:9091/swagger-ui.html
![](https://ws1.sinaimg.cn/large/005EneYkgy1fptt65qlusj30so0piq4o.jpg)




