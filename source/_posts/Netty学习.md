title: Netty学习
author: Wtli
date: 2020-09-17 14:09:09
tags:
---
本文对Netty进行深入学习与实践。

作为当前最流行的NIO框架，Netty在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，一些业界著名的开源组件也基于Netty的NIO框架构建。

<!-- more -->

Netty是一个高性能、异步事件驱动的NIO框架，它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。

不多逼逼，先看代码，然后细细品尝

#### Writing a Discard Server

```
import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
```

1. DiscardServerHandler继承了ChannelInboundHandlerAdapter，ChannelInboundHandlerAdapter是ChannelInboundHandler的实现。ChannelInboundHandler提供了可以覆盖的各种事件处理程序方法。现在，仅扩展ChannelInboundHandlerAdapter而不是自己实现处理程序接口就足够了。
2. 我们在channelRead()这里重写事件处理程序方法。每当从客户端接收到新数据时，channelRead()就会触发。在此示例中，接收到的消息的类型为ByteBuf。
3. 为了实现DISCARD协议，处理程序必须忽略收到的消息。ByteBuf是一个引用计数对象，必须通过该release()方法显式释放。请记住，释放任何传递给处理程序的引用计数对象是处理程序的责任。通常，channelRead()处理程序方法的实现如下：
```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```

4. 当Netty由于I/O错误或处理程序实现由于处理事件时抛出异常而引发异常时，使用Throwable调用exceptionCaught()事件处理程序方法。在大多数情况下，应该记录捕获的异常，并在这里关闭与之关联的通道，尽管此方法的实现可能因您想要处理异常情况而有所不同。例如，您可能想在关闭连接之前发送带有错误代码的响应消息。

到目前为止一切顺利。我们已经实现了DISCARD服务器的前半部分。现在剩下的工作是编写main()方法，该方法使用DiscardServerHandler启动服务器。

```
import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
    
/**
 * Discards any incoming data.
 */
public class DiscardServer {
    
    private int port;
    
    public DiscardServer(int port) {
        this.port = port;
    }
    
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
    
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
    
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port = 8080;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        }

        new DiscardServer(port).run();
    }
}
```

1. NioEventLoopGroup是处理I/O操作的多线程事件循环。Netty提供了多种 EventLoopGroup为了不同类型的传输。在此示例中，我们正在实现服务器端应用程序，因此NioEventLoopGroup将使用两个。第一个通常称为"boss"，接受传入的连接。第二个通常称为"worker"，boss接受连接并向worker注册已接受的连接。使用多少线程以及如何将它们映射到创建的Channels取决于EventLoopGroup实现，甚至可以通过构造函数进行配置。
2. ServerBootstrap是设置辅助类。也可以使用Channel直接设置服务器。但是，请注意，这是一个繁琐的过程，在大多数情况下不需要这样做。
3. 在这里，我们指定使用NioServerSocketChannel用于实例化新类Channel以接受传入连接的类。
4. childHandler是处理新接收的channel，ChannelInitializer是配置辅助类。比如，给新的channel配置ChannelPipeline像增加DiscardServerHandler。随着应用程序变得复杂，可以封装一个通用的类。
5. 我们还可以设置Channel实现的参数。我们正在编写一个TCP/IP服务器，因此我们可以设置socket套接字选项，例如tcpNoDelay和keepAlive。请参考ChannelOption的api和具体ChannelConfig实现有关概述。
6. option()用于NioServerSocketChannel接受传入的连接。childOption()是Channel为父级接受的ServerChannel，在本例中就是指的NioServerSocketChannel。
7. 剩下的就是绑定到端口并启动服务器。在这里，我们绑定到计算机8080中所有NIC（网络接口卡）的端口。现在，您可以bind()根据需要多次调用该方法（使用不同的绑定地址。）

恭喜你！您已经在Netty上完成了第一台服务器。


#### Looking into the Received Data

现在，我们已经编写了第一台服务器，我们需要测试它是否确实有效。测试它的最简单方法是使用telnet命令。例如，您可以telnet localhost 8080在命令行中输入并输入一些内容。

但是，我们可以说服务器工作正常吗？我们真的不知道这是因为它是一个废弃服务器。您根本不会得到任何回应。为了证明它确实有效，让我们修改服务器以打印收到的内容。

我们已经知道，channelRead()每当收到数据时都会调用该方法。让我们将一些代码放入channelRead()方法中DiscardServerHandler：

```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    try {
        while (in.isReadable()) {
            System.out.print((char) in.readByte());
            System.out.flush();
        }
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```
如果您再次运行telnet命令，您将看到服务器打印它接收到的内容。

#### Writing an Echo Server

到目前为止，我们一直在使用数据而没有任何响应。但是，通常应假定服务器对请求作出响应。让我们学习如何通过实现ECHO协议将响应消息写入客户端，在此将所有接收到的数据发送回去。

与我们在上一节中实现的丢弃服务器的唯一区别在于，它会将接收到的数据发回，而不是将接收到的数据打印到控制台。因此，再次修改该channelRead()方法就足够了：

```
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ctx.write(msg); // (1)
        ctx.flush(); // (2)
    }
```

1. 一个ChannelHandlerContext对象提供的各种操作，使您可以触发各种I/O的事件和操作。在这里，我们调用write(Object)将接收到的消息写入进去。请注意，我们没有像DISCARD示例中那样释放收到的消息。这是因为Netty在将其写到网络时会为您释放它。
2. ctx.write(Object)不会将消息写到网络中。它在内部进行缓冲，然后通过ctx.flush()传输到网络。或者，可以使用ctx.writeAndFlush(msg)更简洁。

如果再次运行telnet命令，您将看到服务器发回发送给它的任何内容。

#### Writing a Time Server
它与前面的示例不同之处在于，它发送包含32位整数的消息，不接收任何请求，并在发送消息后关闭连接。

因为我们将忽略任何接收到的数据，而是在建立连接后立即发送消息，所以channelRead()这次我们不能使用该方法。相反，我们应该重写该channelActive()方法。以下是实现：

```
package io.netty.example.time;

public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(final ChannelHandlerContext ctx) { // (1)
        final ByteBuf time = ctx.alloc().buffer(4); // (2)
        time.writeInt((int) (System.currentTimeMillis() / 1000L + 2208988800L));
        
        final ChannelFuture f = ctx.writeAndFlush(time); // (3)
        f.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        }); // (4)
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```
1. channelActive()当建立连接时，将调用该方法。让我们写一个代表该方法当前时间的32位整数。

2. 要发送新消息，我们需要分配一个包含消息的新缓冲区。我们将要写入一个32位整数，因此我们需要一个ByteBuf容量至少为4个字节的字节。通过ChannelHandlerContext.alloc()获得当前的ByteBufAllocator并分配一个新的缓冲区。

3. 我们不是曾经调用java.nio.ByteBuffer.flip()方法在NIO中发送消息之前吗？ByteBuf没有这样的方法，因为它有两个指针；一个用于读取操作，另一个用于写入操作。当您向某处写入内容时，写入指针会增加，读取指针不会改变。读取指针和写入指针分别表示消息的开始和结束位置。  
相反，NIO缓冲区不提供一种干净的方法来确定消息内容的开始和结束位置，而无需调用flip方法。当您忘记翻转缓冲区时会遇到麻烦，因为将不会发送任何内容或发送不正确的数据。在Netty中不会发生这样的错误，因为我们对不同的操作类型有不同的指针。您会发现，适应它会使您的生活变得更加轻松-无需flipping的生活！  
要注意的另一点是ChannelHandlerContext.write()（和writeAndFlush()）方法返回一个ChannelFuture。A ChannelFuture表示尚未发生的I/O操作。这意味着，由于Netty中的所有操作都是异步的，因此可能尚未执行任何请求的操作。例如，以下代码甚至在发送消息之前就可能关闭连接：
```
Channel ch = ...;
ch.writeAndFlush(message);
ch.close();
```
因此，您需要在close()方法ChannelFuture完成后调用该方法，该write()方法由该方法返回，并在完成写操作后通知其侦听器。请注意，close()也可能不会立即关闭连接，并且返回ChannelFuture。

4. 当写请求完成时，我们如何得到通知？这就像将添加ChannelFutureListener到return 一样简单ChannelFuture。在这里，我们创建了一个匿名操作ChannelFutureListener将在操作完成时关闭Channel。  
另外，您可以使用预定义的侦听器简化代码：
```
f.addListener(ChannelFutureListener.CLOSE);
```
要测试我们的时间服务器是否按预期工作，可以使用UNIX rdate命令：
```
$ rdate -o <port> -p <host>
```
其中<port>是您在main()方法中指定的端口号，<host>通常为localhost。

#### Writing a Time Client

与DISCARD和ECHO服务器不同，我们需要用于时间协议的客户机，因为人类无法将32位二进制数据转换为日历上的日期。在本节中，我们将讨论如何确保服务器正确工作，并学习如何使用Netty编写客户端。

在Netty中，服务器和客户机之间最大也是唯一的区别是使用了不同的Bootstrap和Channel实现。请查看以下代码:
```
public class TimeClient {
    public static void main(String[] args) throws Exception {
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            Bootstrap b = new Bootstrap(); // (1)
            b.group(workerGroup); // (2)
            b.channel(NioSocketChannel.class); // (3)
            b.option(ChannelOption.SO_KEEPALIVE, true); // (4)
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new TimeClientHandler());
                }
            });
            
            // Start the client.
            ChannelFuture f = b.connect(host, port).sync(); // (5)

            // Wait until the connection is closed.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```
1. BootstrapServerBootstrap除了用于非服务器通道（例如客户端通道或无连接通道）之外，其他方法与之相似。
2. 如果仅指定一个EventLoopGroup，它将同时用作boss和worker。但是客户端不区分。
3. NioSocketChannel代替NioServerSocketChannel被用来创建一个客户端Channel。
4. 请注意，childOption()此处不使用与此处不同的方法，ServerBootstrap因为客户端SocketChannel没有父级。
5. 我们应该调用connect()方法而不是bind()方法。

与服务器端代码实际上没有什么不同。它应该从服务器接收一个32位的整数，将其转换为人类可读的格式，打印转换后的时间，然后关闭连接:
```
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg; // (1)
        try {
            long currentTimeMillis = (m.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        } finally {
            m.release();
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```
在TCP/IP中，Netty读取从对等点发送的数据到ByteBuf。它看起来非常简单，与服务器端示例没有任何不同。但是，这个处理程序有时会抛出IndexOutOfBoundsException。我们将在下一节中讨论为什么会发生这种情况。

#### Dealing with a Stream-based Transport

##### One Small Caveat of Socket Buffer

在基于流的传输中（例如TCP/IP），将接收到的数据存储到套接字接收缓冲区中。不幸的是，基于流的传输的缓冲区不是数据包队列而是字节队列。这意味着，即使您将两个消息作为两个独立的数据包发送，操作系统也不会将它们视为两个消息，而只是一堆字节。因此，不能保证您所阅读的内容与远程对等方写的内容完全相同。例如，让我们假设操作系统的TCP / IP堆栈已收到三个数据包：
![w7OTAA.png](https://s1.ax1x.com/2020/09/21/w7OTAA.png)

由于基于流的协议具有此一般属性，因此很有可能在您的应用程序中以以下分段形式读取它们：
![w7OHht.png](https://s1.ax1x.com/2020/09/21/w7OHht.png)

因此，无论是服务器端还是客户端，接收方都应将接收到的数据整理到一个或多个有意义的帧中，以使应用程序逻辑易于理解。在上面的示例中，接收到的数据应采用以下格式：
![w7OTAA.png](https://s1.ax1x.com/2020/09/21/w7OTAA.png)

##### 第一个解决方案

现在让我们回到TIME客户示例。我们在这里有同样的问题。32位整数是非常少量的数据，并且不太可能经常被分段。但是，问题在于它可以被碎片化，并且碎片化的可能性会随着流量的增加而增加。

简单的解决方案是创建一个内部累积缓冲区，然后等待直到所有4个字节都被接收到内部缓冲区中为止。以下是修改后的TimeClientHandler实现，可以解决此问题：

```
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    private ByteBuf buf;
    
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        buf = ctx.alloc().buffer(4); // (1)
    }
    
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        buf.release(); // (1)
        buf = null;
    }
    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        buf.writeBytes(m); // (2)
        m.release();
        
        if (buf.readableBytes() >= 4) { // (3)
            long currentTimeMillis = (buf.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        }
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```
1. 一个ChannelHandler有两个生命周期侦听器方法：handlerAdded()和handlerRemoved()。您可以执行任意（取消）初始化任务，只要它不会长时间阻塞即可。
2. 首先，应将所有接收到的数据累加到中buf。
3. 然后，处理程序必须检查是否buf有足够的数据（在此示例中为4个字节），然后继续进行实际的业务逻辑。否则，channelRead()当有更多数据到达时，Netty将再次调用该方法，最终将累加所有4个字节。

##### 第二种解决方案
尽管第一个解决方案已经解决了TIME客户端的问题，但是修改后的处理程序看起来并不干净。想象一个更复杂的协议，它由多个字段（例如可变长度字段）组成。您的ChannelInboundHandler实施将很快变得难以维护。

正如你可能已经注意到，您可以添加多个ChannelHandler到ChannelPipeline，因此，您可以拆分一个单片ChannelHandler成多个模块化的人减少了应用程序的复杂性。例如，您可以分为TimeClientHandler两个处理程序：

1. TimeDecoder处理碎片问题
2. 初始简单版本TimeClientHandler。

幸运的是，Netty提供了一个可扩展的类，帮助您编写第一个开箱即用的:
```
public class TimeDecoder extends ByteToMessageDecoder { // (1)
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) { // (2)
        if (in.readableBytes() < 4) {
            return; // (3)
        }
        
        out.add(in.readBytes(4)); // (4)
    }
}
```
1. ByteToMessageDecoder是ChannelInboundHandler的实现，可以轻松处理碎片问题。
2. ByteToMessageDecoderdecode()每当接收到新数据时，都使用内部维护的累积缓冲区调用该方法。
3. decode()当累积缓冲区中没有足够的数据时，可以决定不向out添加任何内容。收到更多数据时ByteToMessageDecoder将decode()再次调用。
4. 如果decode()将对象添加到out，则表示解码器成功解码了一条消息。
5. 如果decode()向out添加了一个对象，则意味着解码器成功地解码了一条消息。ByteToMessageDecoder将丢弃缓冲区中读取部分。请记住，您不需要解码多条消息。ByteToMessageDecoder会一直调用该decode()方法，直到该方法不添加任何内容out。

现在我们有了另一个要插入到ChannelPipeline的处理程序，我们应该在TimeClient中修改ChannelInitializer实现:
```
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```
如果你是一个喜欢挑战的人，你可能会想要尝试ReplayingDecoder，它简化了解码器甚至更多。不过，您需要参考API参考以获得更多信息。
```
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
            ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}
```
此外，Netty提供了开箱即用的解码器，它使您能够非常容易地实现大多数协议，并帮助您避免以一个无法维护的单块处理器实现告终

#### 使用POJO代替ByteBuf

到目前为止，我们回顾的所有示例都使用ByteBuf作为协议消息的主要数据结构。在本节中，我们将改进时间协议客户机和服务器示例，使其使用POJO而不是ByteBuf。

在ChannelHandlers中使用POJO的优势是显而易见的;通过将从处理程序中提取ByteBuf信息的代码分离出来，处理程序将变得更加可维护和可重用。在时间客户端和服务器示例中，我们只读取一个32位整数，直接使用ByteBuf并不是一个大问题。但是，您会发现在实现实际协议时进行分离是必要的。

 首先，让我们定义一个名为UnixTime的新类型
 ```
 public class UnixTime {

    private final long value;
    
    public UnixTime() {
        this(System.currentTimeMillis() / 1000L + 2208988800L);
    }
    
    public UnixTime(long value) {
        this.value = value;
    }
        
    public long value() {
        return value;
    }
        
    @Override
    public String toString() {
        return new Date((value() - 2208988800L) * 1000L).toString();
    }
}
 ```
我们现在可以修改TimeDecoder来生成UnixTime而不是ByteBuf。
```
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readUnsignedInt()));
}
```
与更新的解码器，时间lienthandler不再使用ByteBuf:
```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```
更简单，更优雅，对吧?同样的技术也可以应用于服务器端。这次让我们先更新TimeServerHandler:
```
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```
现在，唯一缺少的部分是编码器，它是ChannelOutboundHandler的实现，它将UnixTime转换回ByteBuf。这比编写解码器要简单得多，因为在编码消息时不需要处理包碎片和程序集。
```
public class TimeEncoder extends ChannelOutboundHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        UnixTime m = (UnixTime) msg;
        ByteBuf encoded = ctx.alloc().buffer(4);
        encoded.writeInt((int)m.value());
        ctx.write(encoded, promise); // (1)
    }
}
```
1. 首先，我们按原样传递原始ChannelPromise，以便当编码的数据实际写入到网络时，Netty将其标记为成功或失败。
2. 其次，我们没有调用ctx.flush()。有一个单独的处理程序方法void flush(ChannelHandlerContext ctx)，它用于覆盖flush()操作。

为了进一步简化，你可以使用MessageToByteEncoder:
```
public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt((int)msg.value());
    }
}
```
剩下的最后一个任务是在TimeServerHandler之前在服务器端的ChannelPipeline中插入一个TimeEncoder，这只是一个简单的练习。

#### Shutting Down Your Application
关闭一个网络应用程序通常就像关闭通过shutdownGracefully()的关闭创建的所有EventLoopGroups一样简单。它返回一个Future，当EventLoopGroup被完全终止并且属于该组的所有通道被关闭时，它会通知您。




附上参考文献：[https://netty.io/wiki/user-guide-for-4.x.html]