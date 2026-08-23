---
title: WebSocket在SpringBoot下的实践
date: 2020-06-10 22:10:45
tags:
- WebSocket
- Java
- Spring Boot
categories:
- Java
---
项目中有消息推送的需求，就用到了WebSocket。

先在[菜鸟教程](https://www.runoob.com/html/html5-websocket.html)上抄一段介绍：

> WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。
>
> WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。
<!-- more -->
>
> 在 WebSocket API 中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。
>
> 现在，很多网站为了实现推送技术，所用的技术都是 Ajax 轮询。轮询是在特定的的时间间隔（如每1秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。
>
> HTML5 定义的 WebSocket 协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。

后台是用的Spring Boot，Java我不会，网上的文章、例子也看不懂，最后找了个视频教程[千锋微信公众号和微信支付入门视频](https://ke.qq.com/course/315308)，边看边照着打，实现得功能。

把带注释的示例代码记录下来，避免下次使用。

## 依赖：pom.xml

在Spring Boot框架下使用WebSocket实现消息推送，首先在pom.xml中加入依赖。

~~~ xml
<!-- webSocket 开始-->
    <dependency>
        <groupId>javax.websocket</groupId>
        <artifactId>javax.websocket-api</artifactId>
        <version>1.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-websocket</artifactId>
        <version>5.0.8.RELEASE</version>
    </dependency>
<!-- webSocket 结束-->
~~~

## 配置类：WebSocketConfig.java

接着我们新建一个配置类，去注册我们的WebSocket。

~~~ java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSockectConfig implements WebSocketConfigurer {
    /**
     * 注册websocket
     * @param webSocketHandlerRegistry
     */
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
        /**
         * webSocketHandler:处理器
         * stings:websocket访问路径
         * addInterceptors:拦截器,给session加标记
         * setAllowedOrigins:设置允许访问的来源，url
         */
        webSocketHandlerRegistry.addHandler(new MsgHandler(),"/websocket/*").addInterceptors(new MsgInterceptor()).setAllowedOrigins("*");
    }
}
~~~

## 拦截器类：MsgInterceptor.java

思路是将前端传过来的ws://websocket/{username}先拦截下来，通过username来确定与谁建立连接。

~~~ java
import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;

import java.util.Map;

public class MsgInterceptor extends HttpSessionHandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        String url = request.getURI().toString();
        String username =  url.substring(url.lastIndexOf("/") + 1);
        attributes.put("username", username);

        return super.beforeHandshake(request, response, wsHandler, attributes);
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception ex) {
        super.afterHandshake(request, response, wsHandler, ex);
    }
}
~~~

## 处理器类：MsgHandler.java

处理器中主要应为与业务相关的代码，此处简单作为示例，将收到的消息传给接收者

~~~ java
import net.sf.json.JSONObject;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class MsgHandler extends TextWebSocketHandler {
    // 创建key是username，value是session的哈希表，用来查找username对应的session
    private Map<String,WebSocketSession> allClient = new HashMap<>();

    /**
     * 建立连接触发
     * @param session
     * @throws Exception
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        String username = (String) session.getAttributes().get("username");
        if (username != null) {
            allClient.put(username, session);
        }
    }

    /**
     * 收到消息触发
     * @param session
     * @param message
     * @throws Exception
     */
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        JSONObject jsonObject = JSONObject.fromObject(new String(message.asBytes()));
        String to = jsonObject.getString("toUser");
        String toMessage = jsonObject.getString("toMessage");
        String formUser = (String) session.getAttributes().get("username");
        String content = "来自"+formUser+"的内容："+toMessage;
        TextMessage toTextMessage = new TextMessage(content);
        sendMsg(to, toTextMessage);
    }

    /**
     * 断开连接触发
     * @param session
     * @param status
     * @throws Exception
     */
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        super.afterConnectionClosed(session, status);
    }

    /**
     * 封装的发送消息方法
     * @param toUser
     * @param message
     */
    public void sendMsg(String toUser, TextMessage message) {
        WebSocketSession session = allClient.get(toUser);
        if (session != null && session.isOpen()) {
            try {
                session.sendMessage(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
~~~

## 测试页面：Msg.html

我们需要一个简单的页面去测试是否能推送消息。

~~~ html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>msg</title>
    <script type="text/javascript">
    var websocket = null;
    function connection() {
        var username = document.getElementById("name").value;
        if('WebSocket' in window) {
            websocket = new WebSocket("ws://localhost/websocket/" + username);
        } else {
            alert("不支持websocket")
        }
        websocket.onopen = function() {
            document.getElementById("message").innerHTML = '连接建立';
        };
        
        websocket.onmessage = function(event) {
            var data = event.data;
            document.getElementById("message").innerHTML = data;
        };
    
        websocket.onerror = function() {
            document.getElementById("message").innerHTML = '异常';
        };
        websocket.onclose = function() {
            document.getElementById("message").innerHTML = '连接关闭';
        };

        // 当浏览器关闭窗口时，也应该关闭连接，否者服务器端websocket会抛出异常
        window.onbeforeunload = function() {
            if (websocket != null) {
                websocket.close();
            }
        }
    }
    function send() {
        var toUser = document.getElementById("toUser").value;
        var toMessage = document.getElementById("toMessage").value;
        var json = '{"toUser":"' + toUser +'","toMessage":"'+toMessage+'"}';
        if (websocket != null) {
            websocket.send(json);
        }
    }
    </script>
</head>
<body>
    <input type="text" id="name"><button  onclick="connection()">连接</button>
    接收人：<input type="text" id="toUser"><br>
    内容：<input type="text" id="toMessage">
    <button onclick="send()">发送</button><br><br><br><br>
    <span id="message"></span>
</body>
</html>
~~~

## 配置wss

因为项目后台使用https，所以WebSocket也要从ws://改为wss://

是Spring Boot框架，所以在项目入口文件

`SpringApplication.run(DemoApplication.class, args);`的启动方法下增加`bean`配置。

~~~ java
	/**
   * 创建wss协议接口
   * @return
   */
  @Bean
  public TomcatContextCustomizer tomcatContextCustomizer() {
      return new TomcatContextCustomizer() {
          @Override
          public void customize(Context context) {
              context.addServletContainerInitializer(new WsSci(), null);
          }
      };
  }
~~~
