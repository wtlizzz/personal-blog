title: Websocket IllegalStateException
author: Wtli
date: 2021-03-12 17:11:05
tags:
---
Websocket在高并发情景下，会报IllegalStateException错误。

<!-- more -->

**报错：**
<img style="margin: auto;" src="/images/pasted-73.png"/>

报错位置：

```
package org.apache.tomcat.websocket.WsRemoteEndpointImplBase类
```

![upload successful](/images/pasted-75.png)

**情景：**

高并发调用websocket.sendMessage

sendMessage方法：
![upload successful](/images/pasted-74.png)


**初步修复：**

```
                try {
                    ws.sendMessage(JSON.toJSONString(netData));
                } catch (IllegalStateException e) {
                    log.warn("gRPC to WebSocket IllegalStateException");
                }
```
在使用websocket.sendMessage时增加异常检测。

**进一步修复：**

```
    public void sendMessage(String message) {
        for (WebSocket item : webSockets) {
            synchronized (item){
                try {
                    item.session.getBasicRemote().sendText(message);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```
增加线程同步，在websocket.sendMessage内，这样就能够保持session的状态。

**原因：**

websocket在发送数据时，一直会有一个状态跟随在整个数据发送过程中。
当session发送完上一条才能发送下一条数据（如果不这么做，websocket在高并发的状态下，浏览器接收的数据格式不对会直接报错）。但是在高并发的情景下，会造成还没发送完上一条数据，下一条数据就进入了sendMessage方法，获取了session，发现状态不在待发送就会抛出异常。

目前我了解的websocket只有一个session，应该是与浏览器保持的连接。在高并发状态下，性能并不是很高，我正在做一个高性能的websokcet。




















