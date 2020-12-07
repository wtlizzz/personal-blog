title: Java NIO实现自定义通信
author: Wtli
tags:
  - Java NIO
  - ''
categories:
  - Java NIO
  - ''
date: 2020-11-24 09:18:00
---
使用Java Nio实现Java服务端通道通信模式  
Client \--> Server \--> web的通信模式，如下图：
[![DtkMi8.png](https://s3.ax1x.com/2020/11/24/DtkMi8.png)](https://imgchr.com/i/DtkMi8)

<!-- more -->

### 背景

监控平台项目

[![DtAIhT.png](https://s3.ax1x.com/2020/11/24/DtAIhT.png)](https://imgchr.com/i/DtAIhT)


#### 通信流程


[![DtE4rd.png](https://s3.ax1x.com/2020/11/24/DtE4rd.png)](https://imgchr.com/i/DtE4rd)

由客户端Client发出，然后经过Server端记录，通过WebSocket发送给Web端显示。


#### 可利用条件

在工程中，基于现阶段能够利用的条件，有如下4条：

1. 监控平台中Client数据格式基本一致。可以设置固定的buffer或者byte等，节约资源，提高通信效率。
2. Client与Server的通信有多种通信方式，例如gRPC双向流通信、Java io中的Socket、netty等。
3. 平台有Web端显示，Web端现在成熟高效的通信模式基本就只有WebSocket。Web端配置gRPC比较麻烦。
4. 既要能够接收发送Web端的WebSocket，并且能够实现Client通过Server快速传输。

#### Java Nio实现目标

使用Java Nio来进行实现Server端通道模式的通行方式，实现的愿景有：

1. 能够快速传输，使Server端作为一个通道，通道在快速传输过程中可以使用多线程方式来进行数据的记录。
2. 使用一个端口，通过Header中字段来进行区分Client和Web，即把Server端多种通信融合成一种通信模式。
3. Client传输的数据，格式一致，通过限制数据格式结合Java Nio中的Channel操作来进一步提高通信速度。

### 实现

#### 实现自定义握手

WebSocket是通过在Header中发送Upgrade: websocket和Sec-WebSocket-Key等给服务器，使得建立连接。

在本服务中使用同一端口进行实现，故也需要先发送Header数据，服务器收到之后根据具体的字段进行区分，是客户端发送的Socket通信还是Web发送的WebSocket通信。






































