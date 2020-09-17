title: SpringBoot使用WebSocket
author: Wtli
tags:
  - SpringBoot
  - WebSocket
categories:
  - 后端
  - 通信
date: 2020-09-08 13:50:00
---
本文内容：
- WebSocket介绍
- SpringBoot集成WebSocket
- SpringBoot使用WebSocket
- Vue使用WebSocket
- WebSocket源码分析
<!-- more -->

### WebSocket介绍

**特点：**  
```
（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。

```
**与Http通信比较**  
它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。
![wMofsO.png](https://s1.ax1x.com/2020/09/08/wMofsO.png)

### SpringBoot服务端集成

pom.xml文件引入spring-boot-starter-websocket
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

创建配置类

```
@Configuration
public class WebSocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
```

Controller控制接受请求类
```
@Component
@Slf4j
@ServerEndpoint(value = "/websocket")
public class WebSocket {
        /**
     * 连接成功
     *
     * @param session
     */
    @OnOpen
    public void onOpen(Session session) {
        System.out.println("连接成功");
    }

    /**
     * 连接关闭
     *
     * @param session
     */
    @OnClose
    public void onClose(Session session) {
        System.out.println("连接关闭");
    }

    /**
     * 接收到消息
     *
     * @param text
     */
    @OnMessage
    public String onMsg(String text) throws IOException {
        return "servet 发送：" + text;
    }
}
```
方法介绍：
- @ServerEndpoint：通过这个 spring boot 就可以知道你暴露出去的 ws 应用的路径，有点类似我们经常用的@RequestMapping。比如你的启动端口是8080，而这个注解的值是ws，那我们就可以通过 ws://127.0.0.1:8080/ws 来连接你的应用
- @OnOpen：当websocket 建立连接成功后会触发这个注解修饰的方法
- @OnClose：当 websocket 建立的连接断开后会触发这个注解修饰的方法
- @OnMessage：当客户端发送消息到服务端时，会触发这个注解修饰的方法
- @OnError：当 websocket 建立连接时出现异常会触发这个注解修饰的方法


### SpringBoot使用WebSocket

SpringBoot创建Socket控制类，每次请求都是创建了一个控制类实例。
![wQiFL4.png](https://s1.ax1x.com/2020/09/08/wQiFL4.png)
为了能够保存每次请求对话，在类中需要创建一个static CopyOnWriteArraySet用来保存所有的客户端对应实例对象，每个对象都有保存对应客户端的session对话。
```
//concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
  private static CopyOnWriteArraySet<WebSocket> webSockets = new CopyOnWriteArraySet<WebSocket>();
//与某个客户端的连接会话，需要通过它来给客户端发送数据
private Session session;
```
#### 建立连接
将实例对象的session赋值，用来发送请求。将本次创建的实例添加到CopyOnWriteArraySet中。
```
@OnOpen
public void onOpen(Session session) {
   this.session = session;
   webSockets.add(this); 
}
```

#### 接收消息
使用@OnMessage修饰方法，进行消息的接收触发的回调方法，参数为String
```
    @OnMessage
    public void onMessage(String message) {
        ······
    }
```


#### 发送消息

发送消息需要客户端的session，调用sendText方法。

```
public void sendMessage(String message) 
   for (WebSocket item : webSockets) {
      try {
            item.session.getBasicRemote().sendText(message);
      } catch (IOException e) {
            e.printStackTrace();
      }
   }
}


```
### Vue使用WebSocket

其他客户端可以参考github:[websockets/ws](https://github.com/websockets/ws)，这个链接不适用于web端（Note: This module does not work in the browser. The client in the docs is a reference to a back end with the role of a client in the WebSocket communication. Browser clients must use the native WebSocket object.） 

**直接使用Vue中的的WebSocket。**

```
initWebSocket() {
   this.ws = new WebSocket('ws://localhost:8080/websocket');
   this.ws.onmessage = this.onMessage;
   this.ws.onopen = this.onOpen;
   this.ws.onerror = this.onError;
   this.ws.onclose = this.onClose;
},
```
浏览器关闭，会自动触发onClose方法，服务器那边会收到close的请求。下面是4个不同状态回调的函数。

```
onMessage(message) {//接收到服务器的消息
   console.log('webSocket  onMessage: ' + message.data);
},

onOpen() {//建立连接
   console.log('webSocket  onOpen');
},

onError() {//发生错误
   console.log('webSocket  onError');
},

onClose() {//关闭连接
   console.log('webSocket  onClose');
},
```
Vue发送WebSocket信息，使用send发送消息
```
this.ws.send('i have received:' + message.data);
```


