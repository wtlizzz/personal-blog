title: Java NIO运行详解
author: Wtli
tags:
  - Java NIO
categories:
  - 后端
date: 2020-10-27 09:42:00
---
本文使用Java NIO实现WebSocket通信，并对Java NIO运行过程进行详细介绍。

<!--more-->

### 服务端实现

创建Server运行主程序,创建WebSocketServer类以及运行的main方法。
```
public class WebSocketServer {

    private static ArrayList<Client> clients = new ArrayList<Client>();

    public static void main(String[] args) {
        try {
            openServer();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

在WebSocketServer类中实现openServer方法，创建ServerSocketChannel，

```
    private static void openServer() throws IOException {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);
        serverSocketChannel.socket().bind(new InetSocketAddress(8888));------------(1)
        Selector selector = Selector.open();---------(2)
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);--------(3)
        while (selector.select() > 0) {---------(4)
            Iterator<SelectionKey> it = selector.selectedKeys().iterator();
            while (it.hasNext()) {
                SelectionKey selectionKey = it.next();
                if (selectionKey.isAcceptable()) {
                    handleAcceptable(selectionKey, selector);
                } else if (selectionKey.isReadable()) {
                    handleReadable(selectionKey);
                }
                it.remove();
            }
        }
    }
```

运行过程

1. 创建了ServerSocketChannel，绑定8888端口。
[![BQEnxA.png](https://s1.ax1x.com/2020/10/27/BQEnxA.png)](https://imgchr.com/i/BQEnxA)

2. 打开Selector，没有将ServerSocketChannel注册到Selector。
[![BQE3a8.png](https://s1.ax1x.com/2020/10/27/BQE3a8.png)](https://imgchr.com/i/BQE3a8)

3. 在Selector中注册ServerSocketChannel。
[![BQEcRJ.png](https://s1.ax1x.com/2020/10/27/BQEcRJ.png)](https://imgchr.com/i/BQEcRJ)

4. 此时的selector.select()并没有数据，进入到等待部分，当请求进入时激活selector.select()方法。


创建handleAcceptable方法，处理请求到达服务端的事件
```
    private static void handleAcceptable(SelectionKey selectionKey, Selector selector) throws IOException {
        ServerSocketChannel channel = (ServerSocketChannel) selectionKey.channel();
        SocketChannel socketChannel = channel.accept();
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_READ);
        System.out.println(String.format("[server] -- client %s connected.", socketChannel.getRemoteAddress().toString()));
    }
```

**说明：**  
因为使用的ServerSocketChannel，会为每个传入连接创建SocketChannel。所以在selectionKey.isAcceptable()状态，将每个传入的请求SocketChannel添加到selector，绑定OP_READ状态。  
即在处理selectionKey.isAcceptable()状态时，都是传入的新链接。在处理selectionKey.isReadable()状态时，都是websocket传输的数据。





创建handleReadable方法，处理请求

```
     private static void readable(SelectionKey selectionKey) throws IOException {
        ClientSession session = (ClientSession) selectionKey.attachment();
        if (session == null) {
            //If the SocketChannel has not been bound by the ClientSession, it is considered to be a new connection and the handshake needs to be completed
            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
            byte[] byteArray = Util.readByteArray(socketChannel);
            System.out.println(new String(byteArray));
            WSProtocol.Header header = WSProtocol.Header.decodeFromString(new String(byteArray));
            String receiveKey = header.getHeader("Sec-WebSocket-Key");
            String response = WSProtocol.getHandShakeResponse(receiveKey);
            socketChannel.write(ByteBuffer.wrap(response.getBytes()));
            ClientSession newSession = new ClientSession(socketChannel);
            selectionKey.attach(newSession);
            socketListener.onOpen(newSession);
        } else {
            socketListener.onMessage(session);
        }
    }
```

#### 建立连接

```
            WSProtocol.Header header = WSProtocol.Header.decodeFromString(new String(byteArray));
            String receiveKey = header.getHeader("Sec-WebSocket-Key");
            String response = WSProtocol.getHandShakeResponse(receiveKey);
            socketChannel.write(ByteBuffer.wrap(response.getBytes()));
```

**在服务端解析出通信的header，从header中获取到receiveKey，通过getHandShakeResponse方法，将握手的response写入到channel中，双方就在此时建立连接。**

```
public class WSProtocol {
    static class Header {
        private Map<String, String> headers = new HashMap<>();

        String getHeader(String key) {
            return headers.get(key);
        }

        static Header decodeFromString(String headers) {
            Header header = new Header();

            Map<String, String> headerMap = new HashMap<>();
            String[] headerArray = headers.split("\r\n");
            for (String headerLine : headerArray) {
                if (headerLine.contains(":")) {
                    int splitPos = headerLine.indexOf(":");
                    String key = headerLine.substring(0, splitPos);
                    String value = headerLine.substring(splitPos + 1).trim();
                    headerMap.put(key, value);
                }
            }
            header.headers = headerMap;
            return header;
        }
    }

    static String getHandShakeResponse(String receiveKey) {
        String keyOrigin = receiveKey + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11";
        MessageDigest sha1;
        String accept = null;
        try {
            sha1 = MessageDigest.getInstance("sha1");
            sha1.update(keyOrigin.getBytes());
            accept = new String(Base64.getEncoder().encode(sha1.digest()));
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        String echoHeader = "";
        echoHeader += "HTTP/1.1 101 Switching Protocols\r\n";
        echoHeader += "Upgrade: websocket\r\n";
        echoHeader += "Connection: Upgrade\r\n";
        echoHeader += "Sec-WebSocket-Accept: " + accept + "\r\n";
        echoHeader += "\r\n";

        return echoHeader;
    }
}
```

#### Headere加密

[![BtBdtH.png](https://s1.ax1x.com/2020/10/30/BtBdtH.png)](https://imgchr.com/i/BtBdtH)

```
    static String getHandShakeResponse(String receiveKey) {
        String keyOrigin = receiveKey + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11";
        MessageDigest sha1;
        String accept = null;
        try {
            sha1 = MessageDigest.getInstance("sha1");
            sha1.update(keyOrigin.getBytes());
            accept = new String(Base64.getEncoder().encode(sha1.digest()));
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
		······
    }
```

1. 获取到Sec-WebSocket-Key：MOEX6T+CaLUFGiHqbeNjRQ==

2. 然后将258EAFA5-E914-47DA-95CA-C5AB0DC85B11添加到Sec-WebSocket-Key中：  
String keyOrigin = receiveKey + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11";

3. 下一步使用sha1加密keyOrigin

4. 最后在使用Base64加密。生成accept：2jmj7l5rSw0yVb/vlWAYkK/YBwk=

5. 将对应的accept塞到Header中的Sec-WebSocket-Accept。

```
        String echoHeader = "";
        echoHeader += "HTTP/1.1 101 Switching Protocols\r\n";
        echoHeader += "Upgrade: websocket\r\n";
        echoHeader += "Connection: Upgrade\r\n";
        echoHeader += "Sec-WebSocket-Accept: " + accept + "\r\n";
        echoHeader += "\r\n";
        return echoHeader;
```

#### Buffer
创建ByteBuffer时，Buffer是处于写入状态。

```
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
//        byteBuffer.flip();
        byteBuffer.put("hlloo".getBytes());
```

**position=0   position是写入的位置  
 limit=capacity=1024  分别是数据最大的位置和容器最大的位置**

[![BGUcWT.png](https://s1.ax1x.com/2020/10/29/BGUcWT.png)](https://imgchr.com/i/BGUcWT)

**如果在此时调用flip()再写入就会BufferOverflowException报错**

[![BGU7Y6.png](https://s1.ax1x.com/2020/10/29/BGU7Y6.png)](https://imgchr.com/i/BGU7Y6)