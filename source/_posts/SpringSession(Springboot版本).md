---
layout: '[layout]'
title: SpringSession(Springboot版本)
date: 2018-04-19 14:16:24
tags:
- SpringBoot
- SpringSession
- Java
categories:
- 后端
- 
---
#### 特性：

- 使用GemFire来构建C/S架构的httpSession（不关注）
- 使用第三方仓储来实现集群session管理，也就是常说的分布式session容器，替换应用容器（如tomcat的session容器）。仓储的实现，Spring Session提供了三个实现（redis，mongodb，jdbc），其中redis使我们最常用的。程序的实现，使用AOP技术，几乎可以做到透明化地替换。（核心）
- 可以非常方便的扩展Cookie和自定义Session相关的Listener，Filter。
- 可以很方便的与Spring Security集成，增加诸如findSessionsByUserName，rememberMe，限制同一个账号可以同时在线的Session数（如设置成1，即可达到把前一次登录顶掉的效果）等等

>本文的例子使用springsession结合redis实现session的缓存，解决单点登录的分布式session存储问题

#### 1、添加依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>

```

#### 2、添加一个配置
> 配置类开启Redis Http Session，基本是0配置，只需要让主配置扫描到@EnableRedisHttpSession即可

```java
@Configuration
@EnableRedisHttpSession
public class HttpSessionConfig {

}
```
#### 3、配置文件

> 配置文件application.yml，配置连接的redis信息

```
spring:
  redis:
    host: localhost
    port: 6379
    database: 0
```
***注意：测试使用redis3会报异常，建议使用4及4以上***


#### 4、编写测试controller

```java
@Autowired
    private FindByIndexNameSessionRepository<? extends ExpiringSession> sessionRepository;
    
    @ResponseBody
    @RequestMapping("cookie/{browser}")
    public String cookie(@PathVariable("browser") String browser, HttpServletRequest request, HttpSession session) {
        //取出session中的browser
        Object sessionBrowser = session.getAttribute("browser");
        if (sessionBrowser == null) {
            System.out.println("不存在session，设置browser=" + browser);
            session.setAttribute("browser", browser);
        } else {
            System.out.println("存在session，browser=" + sessionBrowser.toString());
        }
        Cookie[] cookies = request.getCookies();
        if (cookies != null && cookies.length > 0) {
            for (Cookie cookie : cookies) {
                System.out.println(cookie.getName() + " : " + cookie.getValue());
            }
        }
        return "index";
    }

```

#### 5、结果
![](https://ws1.sinaimg.cn/large/005EneYkgy1fqhsfzkf9oj30j603ot8u.jpg)

```
​1 spring:session是默认的Redis HttpSession前缀（redis中，我们常用’:’作为分割符）。

2 每一个session都会有三个相关的key，第三个key最为重要，它是一个HASH数据结构，
将内存中的session信息序列化到了redis中。如上文的browser，就被记录为sessionAttr:browser=chrome,
还有一些meta信息，如创建时间，最后访问时间等。

3 另外两个key，expirations:1504446540000和sessions:expires:7079…我发现大多数的文章都没有对其分析，
前者是一个SET类型，后者是一个STRING类型，可能会有读者发出这样的疑问，redis自身就有过期时间的设置方式TTL，
为什么要额外添加两个key来维持session过期的特性呢？这需要对redis有一定深入的了解才能想到这层设计。
当然这不是本节的重点，简单提一下：redis清除过期key的行为是一个异步行为且是一个低优先级的行为，
用文档中的原话来说便是，可能会导致session不被清除。于是引入了专门的expiresKey，来专门负责session的清除，
包括我们自己在使用redis时也需要关注这一点。在开发层面，我们仅仅需要关注第三个key就行了。
```

#### 6、使用自定义CookieSerializer

```java
@Bean
public CookieSerializer cookieSerializer() {
    DefaultCookieSerializer serializer = new DefaultCookieSerializer();
    serializer.setCookieName("JSESSIONID");
    serializer.setCookiePath("/");
    serializer.setDomainNamePattern("^.+?\\.(\\w+\\.[a-z]+)$");
    return serializer;
}
```
> 使用上述配置后，我们可以将Spring Session默认的Cookie Key从SESSION替换为原生的JSESSIONID。而CookiePath设置为根路径且配置了相关的正则表达式，可以达到同父域下的单点登录的效果，在未涉及跨域的单点登录系统中，这是一个非常优雅的解决方案。如果我们的当前域名是moe.cnkirito.moe，该正则会将Cookie设置在父域cnkirito.moe中，如果有另一个相同父域的子域名blog.cnkirito.moe也会识别这个Cookie，便可以很方便的实现同父域下的单点登录。

#### 7、根据用户名查找用户归属的SESSION
> 这个特性听起来非常有意思，你可以在一些有趣的场景下使用它，如知道用户名后即可删除其SESSION。一直以来我们都是通过线程绑定的方式，让用户操作自己的SESSION，包括获取用户名等操作。但如今它提供了一个反向的操作，根据用户名获取SESSION，恰巧，在一些项目中真的可以使用到这个特性，最起码，当别人问起你，或者讨论到和SESSION相关的知识时，你可以明晰一点，这是可以做到的。

```java
@Controller
public class CookieController {
    @Autowired
    FindByIndexNameSessionRepository<? extends ExpiringSession> sessionRepository;

    @RequestMapping("/test/findByUsername")
    @ResponseBody
    public Map findByUsername(@RequestParam String username) {
        Map<String, ? extends ExpiringSession> usersSessions = sessionRepository.findByIndexNameAndIndexValue(FindByIndexNameSessionRepository.PRINCIPAL_NAME_INDEX_NAME, username);
        return usersSessions;
    }
}
```
