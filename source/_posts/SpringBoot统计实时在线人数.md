---
layout: '[layout]'
title: SpringBoot统计实时在线人数
date: 2018-03-06 18:19:10
tags:
- Java
- SpringBoot
categories:
- 后端
---
#### 1、配置pom文件依赖

```
		<!--统计实时人数-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-websocket</artifactId>
			<version>1.5.10.RELEASE</version>
		</dependency>
```

#### 2、新建一个WebSocketConfig

```java
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```
#### 3、在控制器的包下新建一个MySocket

```java
package cn.buildworld.faceweb.controller;

import org.springframework.stereotype.Component;

import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.CopyOnWriteArraySet;

/**
 * 检测实时在线人数
 *
 * @Author MiChong
 * @Email: 1564666023@qq.com
 * @Create 2018-03-06 16:43
 * @Version: V1.0
 */

@Component
@ServerEndpoint("/websocket")  //该注解表示该类被声明为一个webSocket终端
public class MySocket {
    /**
     * 初始在线人数
     */
    private static int online_num = 0;
    /**
     * 线程安全的socket集合
     */
    private static CopyOnWriteArraySet<MySocket> webSocketSet = new CopyOnWriteArraySet<MySocket>();
    /**
     * 会话
     */
    private Session session;

    @OnOpen
    public void onOpen(Session session){
        this.session = session;
        webSocketSet.add(this);
        addOnlineCount();
        System.out.println("有链接加入，当前人数为:"+getOnline_num());
        this.session.getAsyncRemote().sendText("有链接加入，当前人数为:"+getOnline_num());

    }

    @OnClose
    public void onClose(){
        webSocketSet.remove(this);
        subOnlineCount();
        System.out.println("有链接关闭,当前人数为:"+getOnline_num());
    }

    @OnMessage
    public void onMessage(String message,Session session) throws IOException {
        System.out.println("来自客户端的消息:"+message);
        for(MySocket item:webSocketSet){
            this.session.getAsyncRemote().sendText(message);

        }
    }
    public synchronized int getOnline_num(){
        return MySocket.online_num;
    }
    public synchronized int subOnlineCount(){
        return MySocket.online_num--;
    }
    public synchronized int addOnlineCount(){
        return MySocket.online_num++;
    }
}
```

#### 4、前端显示结果

```html
<!DOCTYPE html>
<html>
<head>
    <title>My WebSocket</title>
</head>

<body>
Welcome<br/>
<input id="text" type="text" /><button onclick="send()">Send</button>    <button onclick="closeWebSocket()">Close</button>
<div id="message">
</div>
</body>

<script type="text/javascript">
    var websocket = null;

    //判断当前浏览器是否支持WebSocket
    if('WebSocket' in window){
        onlinenum = new WebSocket("ws://localhost:9090/face/websocket");
    }
    else{
        alert('Not support websocket')
    }

    //连接发生错误的回调方法
    onlinenum.onerror = function(){
        setMessageInnerHTML("error");
    };

    //连接成功建立的回调方法
    onlinenum.onopen = function(event){
        setMessageInnerHTML("open");
    }

    //接收到消息的回调方法
    onlinenum.onmessage = function(event){
        setMessageInnerHTML(event.data);
    }

    //连接关闭的回调方法
    onlinenum.onclose = function(){
        setMessageInnerHTML("close");
    }

    //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
    window.onbeforeunload = function(){
        onlinenum.close();
    }

    //将消息显示在网页上
    function setMessageInnerHTML(innerHTML){
        document.getElementById('message').innerHTML += innerHTML + '<br/>';
    }

    //关闭连接
    function closeWebSocket(){
        onlinenum.close();
    }

    //发送消息
    function send(){
        var message = document.getElementById('text').value;
        onlinenum.send(message);
    }
</script>
</html>
```
##### 测试结果
![这里写图片描述](http://img.blog.csdn.net/20180306181809300?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzE2NzM2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)