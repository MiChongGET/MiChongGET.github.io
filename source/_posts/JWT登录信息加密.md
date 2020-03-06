---
layout: '[layout]'
title: JWT登录信息加密
date: 2018-04-11 17:58:59
tags:
- Java
categories:
- 后端
---
#### 1、背景
对于传统的单点登录系统，使用cookie和session的方式存储用户登录信息，但是对于安全性要求较高的企业--金融企业，就需要对用户的信息进行加密存储，防止客户信息泄露。

![](https://ws1.sinaimg.cn/large/005EneYkgy1fq8uglu1ekj30yl0cijst.jpg)

#### 2、JWT构成
###### JWT----JSON Web Token
![](https://ws1.sinaimg.cn/large/005EneYkgy1fq8uj6ep36j30vv0gpwgw.jpg)



> 第一部分我们称它为头部（header)

```
完整的头部就像下面这样的JSON：

{

"typ": "JWT",  //声明类型，这里是jwt

"alg": "HS256" //声明加密的算法 通常直接使用 HMAC SHA256

}
```


> 第二部分我们称其为载荷（payload)



```
载荷就是存放有效信息的地方。
这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分

标准中注册的声明

公共的声明

私有的声明
```

```

标准中注册的声明 (建议但不强制使用) ：

iss: jwt签发者

sub: jwt所面向的用户

aud: 接收jwt的一方

exp: jwt的过期时间，这个过期时间必须要大于签发时间

nbf: 定义在什么时间之前，该jwt都是不可用的.

iat: jwt的签发时间

jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
```
```
公共的声明 ：

公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.

```
```
私有的声明 ：

私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个payload：

{

"name":"MiChong",

"age":"23",

"org":"天王盖地虎"

} 
```


> 第三部分是签证（signature)

```
jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

header (base64后的)

payload (base64后的)

secret

```

#### 3、Java实现

###### 添加依赖

```
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.3.0</version>
        </dependency>

```

###### 加密解密实现

```java

package cn.buildworld.daliy;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.Claim;
import com.auth0.jwt.interfaces.DecodedJWT;

import java.io.UnsupportedEncodingException;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * jwt加密解密工具
 *
 * @Author MiChong
 * @Email: 1564666023@qq.com
 * @Create 2018-04-11 16:16
 * @Version: V1.0
 */

public class JwtToken {


    /**
     * 公共密钥
     */
    public static final String SECRET = "michong";


    /**
     * 创建token
     * @return
     * @throws UnsupportedEncodingException
     */
    public static String createToken() throws UnsupportedEncodingException {

        //签发时间
        Date date = new Date();

        //过期时间- 1分钟过期
        Calendar nowTime = Calendar.getInstance();
        nowTime.add(Calendar.MINUTE,1);
        Date expiresDate = nowTime.getTime();

        Map<String,Object> map = new HashMap<>();
        map.put("alg","HS256");
        map.put("typ","JWT");
        String token = JWT.create()
                .withClaim("name", "michong")
                //设置过期时间
                .withExpiresAt(expiresDate)
                //设置签发时间
                .withIssuedAt(date)
                .sign(Algorithm.HMAC256(SECRET));

        return token;
    }


    /**
     * 解密
     * @param token
     * @return
     * @throws UnsupportedEncodingException
     */
    public static Map<String,Claim> verifyToken(String token) throws UnsupportedEncodingException {

        JWTVerifier verifier = JWT.require(Algorithm.HMAC256(SECRET))
        .build();

        DecodedJWT jwt = null;

        try{

            //解密
            jwt = verifier.verify(token);

        }catch (Exception e){
            throw new RuntimeException("token已经失效");
        }

        return jwt.getClaims();

    }

}


```

#### 四、测试

```java
public class Auth0WithJWT {
    public static void main(String[] args) throws UnsupportedEncodingException {

        String token = JwtToken.createToken();
        System.out.println(token);
        
        Map<String, Claim> stringClaimMap = JwtToken.verifyToken(token);
        System.out.println(stringClaimMap.get("name").asString());
    }
}
```
###### 结果
![](https://ws1.sinaimg.cn/large/005EneYkgy1fq8vbms62jj30jf052mxe.jpg)




