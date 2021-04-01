title: WebSocket源码剖析
author: Wtli
date: 2020-09-10 16:34:59
tags:
---
对Java.WebSocket.Programming这本书进行剖析

<!-- more-->


### WebSocket源码分析

**@ServerEndpoint**
```
The @ServerEndpoint annotation is a class level annotation that is used to tell the Java platform that the class it decorates is actually going to be a WebSocket endpoint. The only mandatory parameter of this annotation is the relative URI (uniform resource identifier) that the developer wants to use to make this endpoint available under. This is a bit like giving someone the phone number where people are able to call them. 
```
类似于@RequestMapping注释，为类声明访问的地址。

**Endpoints**
```
We use the term “endpoint” to mean one end of the WebSocket conversation.
```
**两种WebSocket服务端的实现方法**
```
There are two kinds of Java WebSocket endpoints: annotated endpoints and programmatic endpoints. Annotated endpoints are Java classes that have been transformed into Java WebSocket endpoints using Java annotations from the Java WebSocket API. Programmatic endpoints are Java classes that subclass the Endpoint class from the Java WebSocket API.
```
- 使用注解
- extends Endpoint 

#### 连接原理
**准备阶段（Deployment Phase）**
```
First, the examination will locate any Java classes that are annotated with the @ServerEndpoint and any Java classes that extend the Endpoint class from the Java WebSocket API.It will also locate in the WAR file any classes that implement the ServerApplicationConfig interface; these classes will tell it how to deploy the endpoints.
```
找到@ServerEndpoint注解类，和ServerApplicationConfig配置类，进行配置。


**建立连接（Accepting the First Connection）**

1. 初始化一次HTTP的交互（*an initial HTTP request/response interaction*）。
2. 这种交互又被称为WebSocket的握手（*This interaction is called the WebSocket opening handshake*.）。
3. 当建立连接完成，WebSocket会为每一个Client创建对应的Endpoint实例（*The WebSocket implementation will create a new instance of the endpoint,that will be dedicated to interacting with that single client to which it is now connected*.）。

![upload successful](/images/pasted-47.png)

**特点**

``` 
Unlike HTTP-based technologies, the WebSockets have a lifecycle that is underpinned by the WebSocket protocol itself.
```
与基于http的技术不同，WebSocket有一个由WebSocket协议本身支撑的生命周期。在servlet技术中，底层协议只定义了一个简单的请求/响应交互，它完全独立于下一个交互

WebSocket在客户端和服务器之间定义了一个更长的专用TCP连接(*the WebSocket protocol defines a longer-lived and dedicated TCP connection between a client and a server,*)

此外，WebSocket协议定义了在WebSocket连接上向后和向前传输的单个数据块的格式。(*In addition, the WebSocket protocol defines the format of individual chunks of data that are transmitted backward and forward over the WebSocket connection.*)

在WebSocket协议中有两种主要类型的帧:控制帧和数据帧。（*There are two main types of frames in the WebSocket protocol: control frames and data frames.*），控制帧例如：关闭帧、ping、pong（*Ping and pong frames are transmissions of data that serve as a check on the health of the connection*）

**WebSocket消息（WebSocket Messaging）**

*Now the Transmission Control Protocol (TCP) connection is established.*

WebSocket协议在TCP的基础上用最小的帧数定义了一个消息传递协议

```
The WebSocket protocol defines a messaging protocol on top of TCP with minimal framing.
```
在TCP连接上向后和向前发送的不同WebSocket协议框架定义了WebSocket的生命周期事件，比如打开和关闭连接，同时也定义了应用程序创建的文本和二进制消息如何在连接上传输。

![upload successful](/images/pasted-48.png)

 在图中，我们可以看到，我们在服务器上部署了一个逻辑端点，它有两个instance，由两个Endpoint表示，每个Endpoint处理来自两个独立客户机的消息。