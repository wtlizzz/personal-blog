title: Java NIO实践
author: Wtli
tags:
  - Java NIO
categories:
  - 后端
  - ''
date: 2020-09-24 14:48:00
---
本文主要对Java NIO进行实践学习介绍，并对代码进化史简单说明：  
传统BIO通信 --> 伪异步通信编程 --> Java NIO通信编程。  
并使用Java NIO实现rpc、socket。

<!--more-->

#### 传统BIO编程

Server启动端：
```
public class TimeServer {
    public static void main(String[] args) throws IOException {
        int port = 8080;
        ServerSocket server = null;
        try {
            server = new ServerSocket(port);
            System.out.println("The time server is start in port : " + port);
            Socket socket = null;
            while (true) {
                socket = server.accept();
                new Thread(new TimeServerHandler(socket)).start();
            }
        } finally {
            if (server != null) {
                System.out.println("The time server close");
                server.close();
                server = null;
            }
        }
    }
}
```
启动了一个ServerSocket对端口进行监听。使用while死循环获取server.accept的信息，每获取一个信息，就新建一个HandlerThread线程去进行处理。

Server处理端：
```
public class TimeServerHandler implements Runnable {

    private Socket socket;

    public TimeServerHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        BufferedReader in = null;
        PrintWriter out = null;
        try {
            in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
            out = new PrintWriter(this.socket.getOutputStream(), true);
            String currentTime = null;
            String body = null;
            while (true) {
                body = in.readLine();
                if (body == null) {
                    break;
                }
                System.out.println("The time server receive order : " + body);
                currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new Date(System.currentTimeMillis()).toString() : "BAD ORDER";
                out.println(currentTime);
            }
        } catch (Exception e) {
            if (in != null) {
                try {
                    in.close();
                } catch (IOException el) {
                    el.printStackTrace();
                }
            }
            if (out != null) {
                out.close();
                out = null;
            }
            if (this.socket != null) {
                try {
                    this.socket.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
                this.socket = null;
            }
        }

    }
}
```

Client端：
```
public class TimeClient {
    public static void main(String[] args) {
        int port = 8080;
        Socket socket = null;
        BufferedReader in = null;
        PrintWriter out = null;
        try {
            socket = new Socket("127.0.0.1", port);
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream(), true);
            out.println("QUERY TIME ORDER");
            System.out.println("Send order 2 server succeed");
            String resp = in.readLine();
            System.out.println("Now is : " + resp);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(out != null){
                out.close();
                out = null;
            }
            if (in != null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                in = null;
            }
            if(socket != null){
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                socket = null;
            }
        }
    }
}
```

**整个流程分析**

1. Server端：运行，进入到while循环中，在socket = server.accept();处等待。 
2. Client端：运行socket = new Socket("127.0.0.1", port);
2. Server端：新建线程new Thread(new TimeServerHandler(socket)).start();
4. Server端：负责接收请求的线程进入到body = in.readLine();等待客户端输入流；这时已经建立好连接。
5. Client端：调用out.println("QUERY TIME ORDER");发送消息给服务端。进入到String resp = in.readLine();等待返回的数据。
6. Server端：通过body = in.readLine();接收到数据，通过out.println(currentTime);返回数据。
7. Client端：接收到返回的数据，关闭连接。
8. Server端：关闭连接。

到此为止，BIO主要的问题在于每当有一个新的客户端请求接入时，服务端必须创建一个新的线程处理新的接入请求的客户端链路，一个线程只能处理一个客户端连接。在高性能服务器应用领域，往往需要面向成千上万个客户端的并发连接，这种模型显然无法满足高性能、高并发接入的场景。

为了改进一线程一连接的模型，后来又演进出了一种通过线程池或者消息队列实现1个或者多个线程处理N个客户端的模型，由于他的底层通信机制依然使用同步阻塞你IO，所以被称为“伪异步”。

#### 伪异步IO编程
实现原理：通过线程池可以灵活调配线程资源，设置线程的最大值，防止由于海量并发接入导致线程耗尽。

伪异步IO的TimeServer代码
```
public class TimeServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = null;
        try {
            serverSocket = new ServerSocket(8080);
            System.out.println("The time server is start in port : " + 8080);
            Socket socket = null;
            TimeServerHandlerExecutePool singleExecutor = new TimeServerHandlerExecutePool(50, 1000);
            while (true) {
                socket = serverSocket.accept();
                singleExecutor.execute(new TimeServerHandler(socket));
            }
        } ...
    }
}
```

TimeServerHandlerExecutePool自定义线程池

```
public class TimeServerHandlerExecutePool {

    private ExecutorService executorService;

    public TimeServerHandlerExecutePool(int maxPoolSize, int queueSize) {

        executorService = new ThreadPoolExecutor(Runtime.getRuntime().availableProcessors(), maxPoolSize, 120L, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(queueSize));

    }

    public void execute(Runnable task) {
        executorService.execute(task);
    }

}
```

**伪异步弊端**
当对Socket的输入流进行读取操作的时候，他会一直阻塞下去，直到发生如下三种事件。
- 有数据可读；
- 可用数据已经读取完毕；
- 发生空指针或者IO异常。

这意味着当对方发送请求或者应答消息比较缓慢，或者网络传输较慢时，读取输入流一方的通信线程将被长时间阻塞，如果对方要60s才能够将数据发送完成，读取一方的IO线程也将会被通读阻塞60s，在此期间，其他接入消息只能在消息队列中排队。

当调用OutputStream的write方法写输出流的时候，他将会被阻塞，直到所有要发送的字节全部写入完毕，或者发生异常。当消息的接收方处理缓慢的时候，将不能及时的从TCP缓冲区读取数据，这将会导致发送放的TCP window size不断减少，直到为0，双方处于Keep-Alive状态，消息发送方将不能再向TCP缓冲区写入消息，这时如果采用的是同步阻塞IO，write操作将会被无限期阻塞，直到TCP window size大于0或者发生IO异常。

伪异步IO实际上仅仅是对之前IO线程模型的一个简单优化，它无法从根本上解决同步IO导致的通信线程阻塞问题。下面我们就简单分析下通信对方返回应答时间过长会引起的级联故障。
1. 服务端处理缓慢，返回应答消息耗费60s，平时只需要10ms。
2. 采用伪异步IO的线程正在读取故障服务节点的响应，由于读取输入流是阻塞的，它将会被同步阻塞60s。
3. 加入所有的可用线程都被故障服务器阻塞，那后续所有的IO消息豆浆在队列中排队。
4. 由于线程池采用阻塞队列实现，当队列挤满之后，后续入队列的操作将被阻塞。
5. 由于前端只有一个Accptor线程接收客户端接入，他被阻塞在线程池的同步阻塞队列之后，新的客户端请求消息将被拒绝，客户端会发生大量的连接超时。
6. 由于几乎所有的连接都超时，调用者会认为系统已经崩溃，无法接受新的请求消息。

#### NIO编程

与Socket类和ServerSocket类相对应，NIO也提供了SocketChannel和ServerSocketChannel两种不同的套接字通道实现，这两种新增的通道都支持阻塞和非阻塞两种模式。

- 缓冲区（Buffer）  
在NIO库中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读取到缓冲区中的；在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

缓冲区实质上是一个数组。通常他是一个字节数组（ByteBuffer），也可以使用其他种类的数组，但是一个缓冲区不仅仅是一个数组，缓冲区提供了对数据的结构化访问以及维护读写位置（limit）等信息。

- 通道（Channel）

- 多路复用器（Selector）  
它是JavaNIO的基础，简单来讲，Selector会不断的轮询注册在其上的Channel，如果某个Channel上面发生读或者写事件，这个Channel就处于就绪状态，会被Selector轮询出来，然后通过SelectionKey可以获取就绪Channel的集合，进行后续的IO操作。  
只需要一个线程负责Selector的轮询，就可以接入成千上万的客户端。

服务端创建的基本步骤：
1. 打开ServerSocketChannel,用于监听客户端的连接，它是所有客户端连接的父管道：
```
ServerSocketChannel acceptorSvr = ServerSocketChannel.open();
```
2. 绑定监听端口，设置连接为非阻塞模式：
```
acceptorSvr.socket().bind(new InetSocketAddress(InetAddress.getByName("IP"), port));
acceptorSvr.configureBlocking(false);
```

3. 创建Reactor线程，创建多路复用器并启动线程：
```
Selector selector = Selector.open();
new Thread(new ReactorTask()).start();
```

4. 将ServerSocketChannel注册到Reactor线程的多路复用器Selector中，监听ACCEPT事件：
```
SelectionKey key = acceptorSvr.register(selector, SelectionKey.OP_ACCEPT, ioHandler);
```

5. 多路复用器在线程run方法的无限循环体内轮询准备就绪的Key：
```
    selector.select(1000);
    Set<SelectionKey> selectionKeySet = selector.selectedKeys();
    Iterator<SelectionKey> it = selectionKeySet.iterator();
    SelectionKey key = null;
    while (it.hasNext()) {
         key = it.next();
         it.remove();
         handleInput(key);
    }
```

6. 多路复用器监听到有新的客户端接入，处理新的接入请求，完成TCP三次握手，建立物理链路：
```
SocketChannel socketChannel = serverSocketChannel.accept();
```

7. 设置客户端链路为非阻塞模式：
```
socketChannel.configureBlocking(false);
socketChannel.socket().setReuseAddress(true);
```

8. 将新接入的客户端连接注册到Reactor线程的多路复用器上，监听读操作，读取客户端发送的网络消息：
```
SelectionKey key = serverSocketChannel.register(selector, SelectionKey.OP_READ, ioHandler);
```
9. 异步读取客户端请求消息到缓冲区：
```
int readBytes = sc.read(byteBuffer);
```

10. 对ByteBuffer进行编解码，如果有半包消息指针reset，继续读取后续报文，将解码成功的消息封装成Task，投递到业务线程池中，进行业务逻辑编排：
```
        while (buffer.hasRemaining()) {
            byteBuffer.mark();
            message = decode(byteBuffer);
            if (message == null) {
                byteBuffer.reset();
                break;
            }
            messageList.add(message);
        }
        if (!byteBuffer.hasRemaining()) {
            byteBuffer.clear();
        } else {
            byteBuffer.compact();
        }
        if (messageList != null && !messageList.isEmpty()) {
            for (Object messageE : messageList)
                handlerTask(messageE);
        }
```

11. 将POJO对象encode成ByteBuffer，调用SocketChannel的异步write接口，将消息异步发送给客户端：
```
socketChannel.write(buffer)；
```

** Note: ** 如果发送区TCP缓冲区满，会导致写半包，此时，需要注册监听写操作为，循环写，直到整包消息写入TCP缓冲区。

##### NIO创建TimeServer源码

TimeServer:
```
    public static void main(String[] args) throws IOException {
        int port = 8080;
        new Thread(new ReactorTask(port)).start();
    }
```
新建一个Thread来进行端口的监听。独立的线程，负责轮询多路复用器Selector，可以处理多个客户端的并发接入。

ReactorTask：
```
public class ReactorTask implements Runnable {

    private Selector selector;//Java Nio多路复用器

    private ServerSocketChannel serverSocketChannel;

    private volatile boolean stop;

    public ReactorTask(int port) throws IOException {
        selector = Selector.open();
        serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);
        serverSocketChannel.socket().bind(new InetSocketAddress(port), 1024);
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("The time server is start in port : " + port);
    }

    public void stop() {
        this.stop = true;
    }

    @Override
    public void run() {
        while (!stop) {
            try {
                selector.select(1000);
                Set<SelectionKey> selectionKeySet = selector.selectedKeys();
                Iterator<SelectionKey> it = selectionKeySet.iterator();
//                SelectionKey key = serverSocketChannel.register(selector, SelectionKey.OP_READ, ioHandler);
                SelectionKey key = null;
                while (it.hasNext()) {
                    key = it.next();
                    it.remove();
                    handleInput(key);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }

    public void handleInput(SelectionKey key) throws IOException {
        if (key.isValid()) {
            if (key.isAcceptable()) {
                ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
                SocketChannel socketChannel = serverSocketChannel.accept();
                socketChannel.configureBlocking(false);
                socketChannel.socket().setReuseAddress(true);
                socketChannel.register(selector, SelectionKey.OP_READ);
            }
            if (key.isReadable()) {
                SocketChannel sc = (SocketChannel) key.channel();
                ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                int readBytes = sc.read(byteBuffer);
                if (readBytes > 0) {
                    byteBuffer.flip();
                    byte[] bytes = new byte[byteBuffer.remaining()];
                    byteBuffer.get(bytes);
                    String body = new String(bytes, "UTF-8");
                    System.out.println("The time server receive order : " + body);
                    String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new Date(System.currentTimeMillis()).toString() : "BAD ORDER";
                    doWhite(sc, currentTime);
                } else if (readBytes < 0) {
                    key.cancel();
                    sc.close();
                } else {
                    ;//读到0字节
                }
            }
        }
    }

    public void doWhite(SocketChannel channel, String response) throws IOException {
        if (response != null && response.trim().length() > 0) {
            byte[] bytes = response.getBytes();
            ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
            writeBuffer.put(bytes);
            writeBuffer.flip();
            channel.write(writeBuffer);
        }
    }

}
```