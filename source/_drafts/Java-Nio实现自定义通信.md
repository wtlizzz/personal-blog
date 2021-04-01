title: Java Pass Socket高速数据转发工具
author: Wtli
tags:
  - Java NIO
categories:
  - 后端
  - ''
date: 2021-01-04 09:42:00
---
Client \--> Server \--> web的通信模式，如下图：
![DtkMi8.png](https://s3.ax1x.com/2020/11/24/DtkMi8.png)

Java Pass Socket高速数据转发通道，高效率的SocketServer通信模式（Socket-SocetServer-WebSocket）。

<!-- more -->
使用Java Nio实现Java服务端通道通信模式，将此模式与传统的WebSocket进行对比（tomcat-embed-websocket-9.0.36-sources.jar） 

## 传统Java Socket

由于做web的实时通信，现在流行的做法就是使用WebSocket来与服务端进行通信，WebSocket目前已经集成在tomcat中，在Java中很方便就能够直接使用WebSocket。

但是tomcat中的WebSocket这种通信方式到底效率是如何的，有的项目是通信数据大、频率低，有的项目是通信数据小、频率高，在此分析一下传统的Java Socket的传输原理，与数据在整个过程中的传输过程。

并且，在后面还会使用Java Nio来根据数据量小，频率高的情景来实现，一种自定义的Java WebSocket，与tomcat集成中的WebSocket进行对比。


### 流程图

Java Server在整个流程中，接收到了Client传输的Socket，然后通过WebSocket发送给Web端，主要功能是进行一个数据的转发。

在目前通信频率的高的情境下，服务端并不需要对数据进行操作，最多使服务端对数据进行存储，现在的边缘计算发展迅速的背景下，数据微小的处理，完全可以放在Client中，使Client发送出来的数据，直接传输到Web端进行使用。

所以，Java Server端将更关注的部分是数据的转发效率，数据的转发效率中重要的部分是对数据的拷贝，现在分析Java IO Server，整个流程中的数据拷贝操作和其他耗时的操作。

![sC4OZF.png](https://s3.ax1x.com/2021/01/04/sC4OZF.png)

### 创建客户端-服务端

使用Java IO创建Client，采用传递数据长度+数据的格式，用于在服务端能够根据数据长度，准确的获取数据包，防止数据在发送接收过程中的粘包问题发生。

```
    private void doSendMessage(Socket socket) throws IOException, SQLException, InterruptedException {
        OutputStreamWriter w = new OutputStreamWriter(socket.getOutputStream(), StandardCharsets.UTF_8);
        Date date = new Date();
        log.info("date:" + date.getTime());
        for (int i = 0; i < 1; i++) {
            AircasNetData aircasNetData = list.get(i);
            String data = aircasNetData.toString().length() + aircasNetData.toString();
            w.write(data);
            w.flush();
        }
        Thread.sleep(1000);
        w.close();
        socket.close();
    }
```

创建Server端,使用普通的Java.net.ServerSocket类，根据获得数据的长度，来准确的获取数据。


```
    public void startServer() throws Exception {
        int port = 8886;
        ServerSocket server = new ServerSocket(port);
        Socket socket = null;
        InputStream inputStream = null;
        while (true) {
            socket = server.accept();
            newGetData(socket);
        }
    }

    private void newGetData(Socket socket) throws Exception {
        InputStream inputStream = socket.getInputStream();
        byte[] bytes = new byte[3];
        for (; ; ) {
            int len = inputStream.read(bytes);
            if (len < 0)
                break;
            String datal = new String(bytes);
            System.out.println(datal);
            int dataLen = Integer.parseInt(datal);
            byte[] bytes1 = new byte[dataLen];
            int len1 = inputStream.read(bytes1);
            if (len1 < 0)
                break;
            sendBytes(bytes);//发送WebSocket给Web端
        }
    }
```

### Java Socket通信原理

Java Socket Server主要涉及两部分
- Socket数据接收
- WebSocket数据发送

#### 程序流程图

程序运行的每一步的流程图，最详细的，没有之一。。

先是Server接收Client发送过来的Socekt数据流程图。

![sCbtW4.png](https://s3.ax1x.com/2021/01/04/sCbtW4.png)


Websocket发送数据方法，实验阶段先只使用Strng类型的数据，使用Object数据的话，Java Pass对比没法控制变量，先透露一下，JPass是使用缓存直接发送。

项目太大了，完整代码有一些不方便透漏的地方，改天整理一下再放到github上，下面是实现方式：
```
	private static CopyOnWriteArraySet<WebSocket> webSockets = new CopyOnWriteArraySet<WebSocket>();
	public void sendMessage(String message) {
        for (WebSocket item : webSockets) {
            try {
                item.session.getBasicRemote().sendText(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

然后是将数据使用tomcat.WebSocket发送给Web的流程图。

![sCvcff.png](https://s3.ax1x.com/2021/01/04/sCvcff.png)

整个发送阶段分成了4部分，首先是
- PreSend：发送前期操作。
- Encode：数据编码操作，例如将类型转换成UTF-8等。
- Deflate：数据压缩操作，将传输的数据进行一次数据的压缩。
- Send：发送部分，最终调用native方法，将数据发送到channel中。

其他部分都好理解，下面着重看一下Deflate部分：

首先讲一下Deflate的作用和使用方式，Deflate主要功能是将数据进行压缩（看网络上使用数据，能将2TB的数据压缩成500GB的大小）。

正常流程是浏览器与服务端建立连接时，需要通过Header发送自己能否对数据进行解压缩，然后服务端将数据进行压缩发送，能够减少数据传输量，然后到了浏览器那边再进行解压缩显示。

> 在握手中使用permessage deflate头来指示连接是否应该使用压缩。  
当客户端发送websocket请求时，如果客户端浏览器支持，它将在websocketextensions头中减少发送的permessage。基于此头的服务器知道是否支持压缩。  
如果服务器决定使用压缩，它将使用与ACK消息类似的相同报头进行响应。客户端在接收到响应后，根据服务器的响应决定是否压缩数据。  
一旦服务器和客户机都决定使用压缩，它们就必须分别使用deflate压缩技术来压缩消息。在创建websocket服务器时，必须使用“perMessageDeflate”选项在服务器上启用压缩。ws-node模块默认启用此功能。ws模块负责头标志，因此您不需要隐式地设置它。

这种压缩传递数据方式，感觉上十分不利于，数据量小，频率高的通信方式，稍后做实验验证一下。

#### 数据流程图

数据流程图主要是对整个通信流程中的，拷贝操作进行记录，分析整个工具的数据传输效率。

原始图：

![sCWE36.png](https://s3.ax1x.com/2021/01/04/sCWE36.png)

优化图：

![sFyeeJ.png](https://s3.ax1x.com/2021/01/05/sFyeeJ.png)


整个流程中的拷贝操作：

1. copy *nativeSocket_buf to buf\[]
2. copy buf\[] to data
3. copy data to CharBuffer part
3.  encode part to encoderBuffer\[8KB]
3.  new MessagePart(encodeBuffer\[8KB])
3.  get MessagePart from messageParts
3.  copy uncompressedPart.payload to uncompressedPayload 
3.  copy uncompressedPayload to deflater
3.  deflater.deflate to compressedPayload
3.  new MessagePart(...compressedPayload)
3.  get mp from allCompressedParts
3.  newOperationState(...buffers)
3.  copy bufs from ByteBuffer\[] srcs

可以看到，一次websocket的传输，需要进行13次的copy操作，并且有的copy操作提前申请buffer的内存大小为8KB，如果是小数据量，高频率的传输模式下，copy操作将是增加传输时延的最大问题所在，同时也是资源浪费的问题所在。


## JPS-Java Pass Socket





![DtAIhT.png](https://s3.ax1x.com/2020/11/24/DtAIhT.png)

#### 通信流程


![DtE4rd.png](https://s3.ax1x.com/2020/11/24/DtE4rd.png)

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