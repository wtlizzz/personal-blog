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
![BQEnxA.png](https://s1.ax1x.com/2020/10/27/BQEnxA.png)

2. 打开Selector，没有将ServerSocketChannel注册到Selector。
![BQE3a8.png](https://s1.ax1x.com/2020/10/27/BQE3a8.png)

3. 在Selector中注册ServerSocketChannel。
![BQEcRJ.png](https://s1.ax1x.com/2020/10/27/BQEcRJ.png)

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

#### Header加密

![BtBdtH.png](https://s1.ax1x.com/2020/10/30/BtBdtH.png)

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

#### 接收数据

读取SocketChannel中数据，使用bytebuffer将数据从channle中读取到缓存（内存块）中，然后将缓存中数据转换成byte\[]类型数据。

``` 
int num = s.read(buffer);
buffer.flip();
int limit = buffer.limit();
byte[] bytes = new byte[limit];
while (num != -1 && buffer.hasRemaining()) {
     buffer.get(bytes);
}
return bytes;
```
判断连接断开，通过byteData\[0]来进行判断。也可以根据第二位来获取数据的长度，通过bytesData\[1] & 0x7f方法。
```
        if ((bytesData[0] & 0xf) == 8) {
            this.onClose(session);
            return;
        }
        byte payloadLength = (byte) (bytesData[1] & 0x7f);
```

下面是获取数据
```
        byte[] mask = Arrays.copyOfRange(bytesData, 2, 6);
        byte[] payloadData = Arrays.copyOfRange(bytesData, 6, bytesData.length);
```

### WebSocket包解析

对WebSocket进行抓包查看，

![Dml0pR.png](https://s3.ax1x.com/2020/11/18/Dml0pR.png)

建立一个WebSocket连接，首先会通过TCP请求，来进行握手，然后发送HTTP请求建立连接，再使用WebSocket Protocol来进行数据的传输。通过TCP连接来进行心跳保活（大约几秒一次，会发送TCP请求）。

报文包解析：

![DmJLSH.png](https://s3.ax1x.com/2020/11/18/DmJLSH.png)

WebSocket前几位分别是：Fin、Reserved、Opcode、Mask、Payload length。之后是Masking-key、Masked payload，下面的Payload就是这次请求携带的Data。

![DmYSTf.png](https://s3.ax1x.com/2020/11/18/DmYSTf.png)

**附：**  
**FIN：**标识是否为此消息的最后一个数据包，占 1 bit  
**RSV1, RSV2, RSV3:** 用于扩展协议，一般为0，各占1bit  
**Opcode：**数据包类型（frame type），占4bits  
0x0：标识一个中间数据包  
0x1：标识一个text类型数据包  
0x2：标识一个binary类型数据包  
0x3-7：保留  
0x8：标识一个断开连接类型数据包  
0x9：标识一个ping类型数据包  
0xA：表示一个pong类型数据包  
0xB-F：保留   
**MASK：**占1bits，用于标识PayloadData是否经过掩码处理。  
**Payload length：**Payload data的长度。  
1、如果其值在0-125，则是payload的真实长度。  
2、如果值是126，则后面2个字节形成的16bits无符号整型数的值是payload的真实长度。  
3、如果值是127，则后面8个字节形成的64bits无符号整型数的值是payload的真实长度。  




### Buffer
创建ByteBuffer时，Buffer是处于写入状态。

```
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
//        byteBuffer.flip();
        byteBuffer.put("hlloo".getBytes());
```

**position=0   position是写入的位置  
 limit=capacity=1024  分别是数据最大的位置和容器最大的位置**

![BGUcWT.png](https://s1.ax1x.com/2020/10/29/BGUcWT.png)

**如果在此时调用flip()再写入就会BufferOverflowException报错**

![BGU7Y6.png](https://s1.ax1x.com/2020/10/29/BGU7Y6.png)

看着源码，在解释一遍buffer的属性：
```
 *   <p> A buffer's <i>capacity</i> is the number of elements it contains.  The
 *   capacity of a buffer is never negative and never changes.  </p>
 *
 *   <p> A buffer's <i>limit</i> is the index of the first element that should
 *   not be read or written.  A buffer's limit is never negative and is never
 *   greater than its capacity.  </p>
 *
 *   <p> A buffer's <i>position</i> is the index of the next element to be
 *   read or written.  A buffer's position is never negative and is never
 *   greater than its limit.  </p>
 
 * <p> A buffer's <i>mark</i> is the index to which its position will be reset
 * when the {@link #reset reset} method is invoked.  The mark is not always
 * defined, but when it is defined it is never negative and is never greater
 * than the position.  If the mark is defined then it is discarded when the
 * position or the limit is adjusted to a value smaller than the mark.  If the
 * mark is not defined then invoking the {@link #reset reset} method causes an
 * {@link InvalidMarkException} to be thrown.
 
```

- capacity： 永远不会变化的值，代表他能包含的数量。
- limit： 不应该读或写的第一个索引。不会是负数，不会比capacity大。不太明白，先看position的定义。初始化生成一个buffer时处于写状态，limit=capaticy，代表最大容量值，调用flip()方法时，处于读状态时limit=position，position=0，代表数据的容量。。
- position： 代表读或者写的下一个位置。不能为负数，不能比limit大。

也就是说：

  <blockquote>
      <tt>0</tt> <tt>&lt;=</tt>
      <i>mark</i> <tt>&lt;=</tt>
      <i>position</i> <tt>&lt;=</tt>
      <i>limit</i> <tt>&lt;=</tt>
      <i>capacity</i>
  </blockquote>

```
    public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }
```

buffer的flip()方法，将limit设置为position，position设置为0。将读取或者写入的位置设置为0，将数据的最大索引设置为limit。

也就是说flip()方法不是说完全意义上的将buffer的状态从写状态转换成读状态。

```
        byte b = 'b';
        byte a = 'a';
        ByteBuffer buffer2 = ByteBuffer.allocate(1024);
        buffer2.put(b);
        buffer2.flip();
        buffer2.put(a);
```
如上代码，在生成buffer之后，将'b'写入，然后调用flip()方法，再次将'a'写入，在最后时buffer中的数据内容只有一个a。

它只有在写入的时候，会根据position >= limit来判断能不能写入。根据设定的position必须小于limit，才能写入，这与之前部分的position<=limit冲突，可能是在某些情况下position可以<=limit，但是在写入的时候position必须<limit,并且mark的值最开始是-1,之前的源码中注释写的是mark>=0的。
```
    final int nextPutIndex() {                          
        if (position >= limit)
            throw new BufferOverflowException();
        return position++;
    }
```

**综上所述：**
buffer.flip()方法，功能不是读写状态的切换，它的功能是将limit设置为position的位置，将position设置为0，将mark设置为-1。读和写操作都是在position的位置，都必须position<limit，否则就会报错BufferUnderflowException。只要是满足position<limit这个条件，就能够进行读或写操作。


buffer写数据与读数据方法，学习一下源码，来理解这些变量的用法。

**写数据：**

调用socketChannel.read(buffer)方法，会往buffer中写数据。

```
源码实现类HeapByteBuffer:
    public ByteBuffer put(byte x) {
        hb[ix(nextPutIndex())] = x;
        return this;
    }
```
调用了Buffer的nextPutIndex()方法，该方法返回的是position的下一位，也就是说如果是写数据的时候，是从position的下一位开始写，ix()方法是在定义offest的情况下索引的偏移量。
```
    final int nextPutIndex() {                          
        if (position >= limit)
            throw new BufferOverflowException();
        return position++;
    }
```

**读数据：**
```
源码实现类HeapByteBuffer:
    public byte get() {
        return hb[ix(nextGetIndex())];
    }
```
直接返回的是hb\[]，hb是定义的final byte\[] hb;缓存的实质是一个byte的数组。
```
    final int nextGetIndex() {                          
        if (position >= limit)
            throw new BufferUnderflowException();
        return position++;
    }
```