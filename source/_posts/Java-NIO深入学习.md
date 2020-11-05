title: Java NIO深入学习
author: Wtli
tags:
  - Java NIO
categories: []
date: 2020-09-16 17:08:00
---
Java.nio是当前流行框架Netty的底层实现，本文对Java.nio进行学习，这也是Java的很重要的基础包。

<!-- more -->
#### Java NIO 简介
Java NIO(New IO)是Java(来自Java 1.4)的API，代替了Java IO和Java Networking API's。Java NIO提供了一种新的IO工作方式。

**通道和缓存（Channels and Buffers）**

在之前的IO API中，使用字节流和字符流。在NIO中，使用通道和缓存。数据总是从一个通道读入一个缓存，或从缓存写入一个通道。

**非阻塞式IO（Non-blocking IO）**

Java NIO允许执行非阻塞IO。例如，线程可以请求通道将数据读入缓存。当通道将数据读入缓冲区时，线程可以做其他的事情。一旦数据被读入缓冲区，线程就可以继续处理它。将数据写入通道也是如此。

**选择器(Selectors)**

 Java NIO包含了“选择器”的概念。选择器是一个对象，可以监控多个通道的事件(如:连接打开，数据到达等)。因此，单个线程可以监视多个数据通道。
 
#### Java NIO 概述

Java NIO有更多的类和组件，通道、缓冲区和选择器构成了API的核心。其他组件，如Pipe和FileLock，只是与这三个核心组件一起使用的实用程序类。因此，在这个NIO概述中，我将重点关注这三个组件。

##### Channels and Buffers

通常，NIO中的所有IO都从一个通道开始，一个通道是一个bit，类似于stream。通过通道数据可以读入缓冲区，数据也能够从缓冲区中写入通道。

下面是Java NIO中主要通道实现的列表:

❀ FileChannel  
❀ DatagramChannel  
❀ SocketChannel  
❀ ServerSocketChannel  

这些通道包括了UDP+TCP网络IO，文件IO。

下面是Java NIO中核心缓冲区实现的列表:

❀ ByteBuffer  
❀ CharBuffer  
❀ DoubleBuffer  
❀ FloatBuffer  
❀ IntBuffer  
❀ LongBuffer  
❀ ShortBuffer  

这些缓冲区涵盖了你可以通过IO发送的基本数据类型:byte，short，int，long，float，double和char。
除此之外，Java NIO还有一个MappedByteBuffer，它与内存映射文件一起使用。

##### Selectors

选择器允许一个线程处理多个通道的线程。如果应用程序打开了许多连接(通道)，但每个连接上的流量都很低，那么这很方便。例如，在聊天服务器中。
![wcyif1.png](https://s1.ax1x.com/2020/09/16/wcyif1.png)

要使用Selectors，你需要注册Channel's。然后调用它的select()方法。此方法将阻塞，直到为其中一个已注册Channel准备好事件为止。方法返回后，线程就可以处理这些事件。事件例如：有传入的连接，接收到的数据等等。

#### Java NIO 通道

Java NIO Channels 与流streams类似，有以下几个区别：
- 可以对通道进行读写操作。流通常是单向的(读或写)。
- 通道可以异步读取和写入。
- 通道总是读取或写入缓冲区。

##### Channel实现

以下是Java NIO中最重要的通道实现:
```
➽ FileChannel：从文件和文件读取数据。
```
```
➽ DatagramChannel：可以通过UDP在网络上读取和写入数据。
```
```
➽ SocketChannel：可以通过TCP在网络上读写数据。
```
```
➽ ServerSocketChannel：允许监听传入的TCP连接，就像web服务器那样。为每个传入连接创建SocketChannel。
```
     
##### Basic Channel Example

下面是一个使用FileChannel读取一些数据到缓冲区的基本例子:

```
    RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
    FileChannel inChannel = aFile.getChannel();

    ByteBuffer buf = ByteBuffer.allocate(48);

    int bytesRead = inChannel.read(buf);
    while (bytesRead != -1) {

      System.out.println("Read " + bytesRead);
      buf.flip();

      while(buf.hasRemaining()){
          System.out.print((char) buf.get());
      }

      buf.clear();
      bytesRead = inChannel.read(buf);
    }
    aFile.close();
```
****注意buf.flip()调用。首先读入一个缓冲区。然后翻转。然后你读出来****

#### Java NIO 缓存

在与NIO通道交互时使用Java NIO缓存。数据是从通道读取到缓冲区，然后从缓冲区写入到通道中。

缓冲区本质上是一个内存块，可以将数据写入其中，然后再读取数据。这个内存块包装在一个NIO缓冲区对象中，该对象提供了一组方法，使使用内存块变得更容易。

##### Basic Buffer Usage

使用缓冲区读取和写入数据通常遵循以下4个步骤:

1. 将数据写入缓冲区
2. 调用buffer.flip()
3. 从缓冲区中读取数据
4. 调用buffer.clear() || buffer.compact()

当将数据写入缓冲区时，缓冲区会跟踪已写入的数据量。一旦需要读取数据，就需要使用flip()方法调用将缓冲区从写入模式切换到读取模式。在读取模式下，缓冲区允许读取写入到缓冲区中的所有数据。

一旦您读取了所有数据，您需要清除缓冲区，使其为再次写入做好准备。有两种方式来进行清除缓存：clear() or calling compact()。clear()方法是清空整个缓存，compact()方法是仅清空已经读取的数据。所有未读数据都被移动到缓冲区的开头，现在数据将在未读数据之后写入缓冲区。

下面是一个简单的缓冲区使用示例:
```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {

  buf.flip();  //make buffer ready for read

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }

  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

##### Buffer Capacity, Position and Limit

缓冲区本质上是一个内存块，您可以将数据写入其中，然后再读取数据。这个内存块包装在一个NIO缓冲区对象中，该对象提供了一组方法，使使用内存块变得更容易。

为了理解缓冲区是如何工作的，缓冲区有三个主要的属性：
1. capacity
2. position
3. limit

position和limit的意义取决于缓冲区是处于读模式还是写模式。capacity总是意味着相同的，无论缓冲模式。
![wcfTG4.png](https://s1.ax1x.com/2020/09/16/wcfTG4.png)

**‡ Capacity（容量）**

作为一个内存块，缓冲区有一定的大小，也称为它的“容量”。您只能将Capacity大小的字节、长度、字符等写入缓冲区。一旦缓冲区满了，就需要清空它(读取数据或清除数据)，然后才能将更多数据写入其中。

**‡ Position（位置）**

当您将数据写入缓冲区时，您是在某个位置写入数据的。初始位置是0。当一个byte，long等已写入缓冲区,Position就变成了的指向下一个单元（cell），即下一次插入数据的位置。位置可以最大限度地变成capacity - 1。

从缓冲区读取数据时，也要从给定位置读取数据。当您将缓冲区从写入模式翻转到读取模式时，position将重置为0。当从缓冲区读取数据时，从position读取数据，并且position指向下一个要读取的位置。

**‡ Limit（限制）**

在写模式下，Limit是指可以写入缓冲区的数据量。在写模式下，Limit等于缓冲区的容量。

当将缓冲区翻转到读模式时，limit表示可以从数据中读取的数据量。因此，当将缓冲区翻转到读模式时，将limit设置为写模式的position。换句话说，读取的Limit就是写入的字节数总量(Limit设置为写入的字节数，由position标记)。


##### Buffer Types
Java NIO提供了以下缓冲区类型:

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

可以看到，这些缓冲区类型表示不同的数据类型。换句话说，它们允许将缓冲区中的bytes改为char、short、int、long、float或double。

此外，MappedByteBuffer有点特殊，将在它自己的文本中介绍。

##### Allocating a Buffer

要获得一个缓冲区对象，您必须首先分配它，每个缓冲区类都有一个执行此操作的allocate()方法。下面是一个例子，显示了字节缓冲区的分配，容量为48字节:
```
ByteBuffer buf = ByteBuffer.allocate(48);
```
下面是一个分配CharBuffer的例子，它有1024个字符的空间:
```
CharBuffer buf = CharBuffer.allocate(1024);
```

##### Writing Data to a Buffer

可以用两种方式将数据写入缓冲区:  
1. 将数据从Channel通道写入缓冲区 
2. 通过缓冲区的put()方法，自己将数据写入缓冲区。

这里是一个例子，显示如何一个通道可以写入数据到缓冲区:

```
int bytesRead = inChannel.read(buf); //read into buffer.
```
下面是一个通过put()方法将数据写入缓冲区的例子:

```
buf.put(127); 
```

put()方法还有许多其他版本，允许您以许多不同的方式将数据写入缓冲区。例如，在特定位置写入，或将字节数组写入缓冲区。有关具体的缓冲区实现的更多细节，请参阅JavaDoc。

##### flip()

flip()方法将缓冲区从写入模式切换到读取模式。调用flip()将position设置回0，并将limit设置为位置原来的位置。

换句话说，position现在标记读取位置，而limit标记有多少字节、字符等写入到缓冲区中——可以读取的字节、字符等的限制。

##### Reading Data from a Buffer

有两种方法可以从缓冲区读取数据。
1. 从缓冲区读取数据到通道中。
2. 自己从缓冲区读取数据，使用get()方法之一。

这里有一个例子，你可以读取数据从缓冲区到一个通道:
```
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```
下面是一个使用get()方法从缓冲区读取数据的示例:
```
byte aByte = buf.get();    
```
get()方法还有许多其他版本，允许您以许多不同的方式从缓冲区读取数据。例如，在特定位置读取，或从缓冲区中读取字节数组。有关具体的缓冲区实现的更多细节，请参阅JavaDoc。

##### rewind()

rewind()将该位置设置为0，这样您就可以重新读取缓冲区中的所有数据。limit保持不变，因此仍然标记了多少元素(字节、字符等)可以从缓冲区中读取。
 
##### clear() and compact()

一旦完成了从缓冲区读取数据的工作，就必须使缓冲区为再次写入做好准备。可以通过调用clear()或compact()来实现

如果调用clear()，position设置为0，limit设置为capacity。换句话说，缓冲区被清除，但是缓冲区中的数据未被清除，只告诉你可以将数据写入缓冲区的位置（如果写入数据的话，之前的数据被覆盖）。

如果在调用clear()时缓冲区中有任何未读数据，那么这些数据将被“遗忘”，这意味着不再有任何标记来说明哪些数据已被读取，哪些数据尚未被读取。

如果缓冲区中仍有未读数据，并且您希望稍后读取它，但需要先进行一些写入操作，请调用compact()而不是clear()。

compact()将所有未读数据复制到缓冲区的开头。然后它将position设置在最后一个未读元素的右侧。limit属性仍然设置为capacity，就像clear()所做的那样。现在缓冲区已经准备好写入，但是您不会覆盖未读数据。

##### mark() and reset()

可以通过调用Buffer.mark()方法标记缓冲区中的给定位置。然后，您可以通过调用Buffer.reset()方法将位置重置回标记的位置。下面是一个例子:
```
buffer.mark();

//call buffer.get() a couple of times, e.g. during parsing.

buffer.reset();  //set position back to mark.    
```
##### equals() and compareTo()

可以使用equals()和compareTo()来比较两个缓冲区.

两个缓冲区满足以下条件，就算是满足equals：

1. 它们是相同类型的(byte、char、int等)。 
2. 它们在剩余的缓冲区中有相同数量的bytes, chars等。 
3. 所有在剩余的缓存中bytes、chars等都是相等的。

可以看到，equals只比较缓冲区的一部分，而不是其中的每个元素。实际上，它只是比较缓冲区中剩余的元素。

compareTo()方法比较两个缓冲区的剩余元素(bytes, chars etc.)，用于排序。一个缓冲区被认为比另一个缓冲区“小”，如果:

1. 第一个元素，与另一个buffer中对应的元素比较，小于另一个缓冲区中的元素。 
2. 所有的元素都是相等的，但是第一个buffer元素更少。

#### Java NIO 分散/收集

Java NIO自带了内置的分散/收集（scatter / gather）支持。分散/聚集是用于从通道读取和写入。

从通道的分散读取是一种读取操作，它将数据读取到一个以上的缓冲区。因此，通道“分散”，把数据从通道转移到多个缓冲区。

对通道的收集写操作是将数据从多个缓冲区写入单个通道的写操作。因此，gather使得通道从多个缓冲区“收集”数据到一个通道。  

在需要分别处理传输数据的各个部分的情况下，分散/收集非常有用。例如，如果消息由消息头和消息体组成，则可以将消息头和消息体保存在单独的缓冲区中。这样做可以使您更容易分别使用标题和主体。

##### Scattering Reads

“分散读取”将数据从一个通道读取到多个缓冲区。下面是一个代码示例，展示了如何执行分散读取:
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```
请注意缓冲区是如何首先插入数组，然后将数组作为参数传递给channel.read()方法的。然后，read()方法按照缓冲区在数组中出现的顺序从通道写入数据。一旦缓冲区满了，通道将继续填充下一个缓冲区。

分散读取在转移到下一个缓冲区之前会填满一个缓冲区，这意味着它不适合动态调整消息部分的大小。换句话说，如果你有一个头和一个体，并且头的大小是固定的(例如128字节)，那么分散读取就可以很好地工作。

##### Gathering Writes
“收集写入”将数据从多个缓冲区写入一个通道。下面是一个代码示例，展示了如何执行一个收集写:
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```
缓冲区数组被传递到write()方法中，该方法按照缓冲区在数组中遇到的顺序写入缓冲区的内容。只有在缓冲区的位置和限制之间的数据被写入。因此，如果缓冲区的容量为128字节，但只包含58字节，则只有58字节从该缓冲区写入到通道。因此，与分散读取不同，收集写入可以很好地处理动态大小的消息部分。


#### Java NIO 信道到信道传输

在Java NIO中，如果其中一个通道是FileChannel，则可以直接将数据从一个通道传输到另一个通道。FileChannel类有一个transferTo()和一个transferFrom()方法，它们为您完成此工作。

##### transferFrom()
transferfrom()方法将数据从源通道传输到**FileChannel**. transferfrom()方法。这里有一个简单的例子:
```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```
参数position和count告诉目标文件中开始写入的位置(position)和最大传输的字节数(count)。如果源通道的字节数少于计数字节，则传输的字节数就更少。

另外，一些SocketChannel实现可能只传输SocketChannel已经在它的内部缓冲区中准备好的数据——即使SocketChannel以后可能有更多可用的数据。因此，它可能不会将所请求的全部数据(计数)从SocketChannel传输到FileChannel。

##### transferTo()

transferTo()方法从一个FileChannel传输到其他通道。简单例子：
```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```

注意，这个示例与前面的示例相似.区别是调用该方法的FileChannel对象。其余的都是一样的。

SocketChannel的问题也出现在transferTo()方法中。SocketChannel实现只能从FileChannel传输字节，直到发送缓冲区满，然后停止。

#### Java NIO 选择器

Java NIO选择器是一个组件，它可以检查一个或多个Java NIO通道实例，并确定哪些通道可以读取或写入。通过这种方式，一个线程可以管理多个通道，从而实现多个网络连接。

##### Why Use a Selector?

使用单个线程处理多个通道的优点是，处理通道所需的线程更少。实际上，您可以使用一个线程来处理所有的通道。对于操作系统来说，线程之间的切换非常昂贵，而且每个线程也会占用操作系统中的一些资源(内存)。因此，使用的线程越少越好。

但是请记住，现代操作系统和CPU在多任务处理方面越来越好，因此多线程的开销会随着时间的推移而减小。实际上，如果一个CPU有多个核心，那么不执行多任务可能会浪费CPU能量。无论如何，设计讨论属于不同的文本。这里只需说明，您可以使用一个选择器，用一个线程处理多个通道。

这里是一个例子，一个线程使用选择器处理3通道的:
![wgJOjs.png](https://s1.ax1x.com/2020/09/16/wgJOjs.png)

##### Creating a Selector
通过调用Selector.open()方法创建一个选择器，如下所示:
```
Selector selector = Selector.open();
```

##### Registering Channels with the Selector
为了使用带有选择器的通道，需要在channel中注册selector。这是通过selectablechanner.register()方法完成的，如下所示:
```
channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```
通道必须处于非阻塞模式才能与选择器一起使用。这意味着你不能使用FileChannel的选择器，因为FileChannel的不能切换到非阻塞模式。套接字通道可以正常工作。

请注意register()方法的第二个参数。这是一个“兴趣集”，意思是您希望通过选择器在通道中侦听的事件。你可以收听四种不同的事件:
1. Connect
2. Accept
3. Read
4. Write

"Ready"代表一个channel触发了一个事件。  
"connect ready"代表channel成功连接到另一个服务。  
"accept"代表一个server socket channel接受一个即将到来的连接。  
"read"代表一个channel已经有读取的数据了。  
"write"代表一个channel准备要往里面写数据。  

这四个事件由四个SelectionKey常量表示:

1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

如果你对多个事件或常量感兴趣，像这样:

```
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;    
```
##### SelectionKey

当使用Selector.register()方法注册一个Channel时，返回一个SelectionKey对象。这个SelectionKey对象包含一些属性:
- The interest set
- The ready set
- The Channel
- The Selector
- An attached object (optional)

**Interest Set**是监听的"selecting"事件集。可以使用如下方法进行读取：
```
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;    
```
可以使用'&'来判断给定的SelectionKey常量是否在兴趣集中。


**Ready Set**是通道准备就绪的操作集。您将主要在选择之后访问准备集，可以像这样访问ready集:
```
int readySet = selectionKey.readyOps();
```
您可以使用与兴趣集相同的方法测试通道准备好了什么事件/操作。但是，你也可以使用这四种方法来代替，它们都是布尔值:
```
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```
**Channel + Selector**从SelectionKey访问通道+选择器非常简单。以下是如何做到的:
```
Channel channel  = selectionKey.channel();

Selector selector = selectionKey.selector();   
```
**Attaching Objects** 可以将对象附加到SelectionKey上，这是识别通道的一种简便方法，也可以将进一步的信息附加到通道上。例如，可以将正在使用的Buffer附加到channel，或将包含更多聚合数据的对象连接。这里是如何附加对象:
```
selectionKey.attach(theObject);

Object attachedObj = selectionKey.attachment();
```
在register()方法中，您还可以在向Selector注册Channel时附加一个对象。这是它的样子:
```
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

##### Selecting Channels via a Selector

一旦您用选择器注册了一个或多个通道，您就可以调用select()方法之一。这些方法返回为您感兴趣的事件(connect, accept, read or write)。换句话说，如果您对准备读取的通道感兴趣，那么您将接收准备从select()方法读取的通道。下面是select()方法:
- int select()
- int select(long timeout)
- int selectNow()

select()一直阻塞，直到至少有一个通道为您注册的事件准备好。

select(long timeout)的操作与select()相同，只是它阻塞的时间最长为超时毫秒(参数)。

selectNow()完全不阻塞。它会立即返回任何准备好的通道。

select()方法返回的int表示有多少通道已经准备好了。也就是说，自上次调用select()以来已经准备好了多少个通道。如果您调用select()，它返回1，因为一个通道已经准备好了，如果您再次调用select()，又有一个通道已经准备好了，它将再次返回1。如果您没有对第一个准备好的通道做任何操作，那么现在有两个准备好的通道，但是在每次select()调用之间只有一个通道准备好了。

**selectedKeys()**

一旦调用了一个select()方法，并且它的返回值表明一个或多个通道已经准备好了，您就可以通过调用选择器selectedKeys()方法，通过“selected key set”访问准备好的通道。类似于这种方式:

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();    
```
当您使用选择器注册一个通道时，channel.register()方法返回一个SelectionKey对象。此键表示使用该选择器注册的通道。您可以通过selectedKeySet()方法访问这些键。从SelectionKey中可以获得这个选定的键集来访问就绪通道:

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();

Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {
    
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
}
```
这个循环迭代所选键集中的键。对于每个键，它测试键以确定键引用的通道已经准备好了。

注意每次迭代结束时的keyIterator.remove()调用。选择器不会从所选键集本身移除SelectionKey实例。当您处理完通道时，您必须这样做。下次通道变为“ready”时，选择器将再次将其添加到选中的键集。

由SelectionKey.channel()方法返回的通道应该被转换为您需要使用的通道，例如ServerSocketChannel或SocketChannel等。

##### wakeUp()
调用了被阻塞的select()方法的线程可以离开select()方法，即使还没有通道准备好。这是通过让另一个线程调用选择器上的Selector.wakeup()方法来实现的，第一个线程在选择器上调用了select()。然后，在select()中等待的线程将立即返回。

如果另一个线程调用了wakeup()，并且当前在select()中没有阻塞线程，那么下一个调用select()的线程将立即“唤醒”。

##### close()
当您完成选择器时，您调用它的close()方法。这将关闭选择器，并使使用此选择器注册的所有SelectionKey实例无效。通道本身并没有关闭。

##### Full Selector Example
下面是一个完整的示例，它打开一个选择器，用它注册一个通道(忽略了通道实例化)，并持续监视选择器对四个事件(accept, connect, read, write)的“准备就绪”。

```
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.selectNow();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```

#### Java NIO 文件信道

Java NIO FileChannel是连接到文件的通道。使用文件通道，您可以从文件读取数据，并将数据写入文件。Java NIO FileChannel类是NIO使用标准Java IO API读取文件的替代方案。

**FileChannel不能设置为非阻塞模式。它总是在阻塞模式下运行。**

##### Opening a FileChannel

在使用FileChannel之前，必须先打开它，但是不能直接打开FileChannel。需要通过InputStream、OutputStream或RandomAccessFile获得FileChannel。以下是如何通过RandomAccessFile打开FileChannel:
```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
```
##### Reading Data from a FileChannel

要从FileChannel读取数据，需要调用read()方法之一。下面是一个例子:
```
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```
首先分配一个缓冲区。从FileChannel读取的数据被读入缓冲区。

然后调用FileChannel.read()方法。此方法将数据从FileChannel读入缓冲区。 read()方法的返回值int告诉缓冲区中容纳了多少字节。如果返回-1，则到达文件结束。

##### Writing Data to a FileChannel

将数据写入FileChannel是使用FileChannel.write()方法，该方法接受缓冲区作为参数。下面是一个例子:
```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```
注意在while循环中是如何调用FileChannel.write()方法的。不能保证write()方法向FileChannel写入了多少字节。因此，我们重复write()调用，直到缓冲区没有字节可写。

##### Closing a FileChannel

当你用完FileChannel时，你必须关闭它。以下是如何做到的:
```
channel.close();   
```

##### FileChannel Position

当读取或写入FileChannel时，在特定位置进行操作的。可以通过调用position()方法获得FileChannel对象的当前位置。还可以通过调用position(long pos)方法来设置FileChannel的位置:

```
long pos = channel.position();

channel.position(pos +123);
```
如果将位置设置在文件结束后，并尝试从通道读取，将得到-1——文件结束标记。

如果将位置设置在文件末尾，并写入通道，则文件将被展开以适应位置和写入数据。这可能会导致“file hole”，即磁盘上的物理文件在写入数据中存在缺口。

##### FileChannel Size

FileChannel对象的size()方法返回通道所连接的文件的大小。这里有一个简单的例子:
```
long fileSize = channel.size();   
```

##### FileChannel Truncate

可以通过调用FileChannel.truncate()方法来截断文件。当你截断一个文件时，你在一个给定的长度切断它。下面是一个例子:
```
channel.truncate(1024);
```
这个示例以1024字节的长度截断文件。

##### FileChannel Force

 force()方法将通道中所有未写入的数据刷新到磁盘。由于性能的原因，操作系统可能会将数据缓存到内存中，因此不能保证写入通道的数据实际上已写入磁盘，直到调用force()方法。

force()方法采用一个布尔值作为参数，它告诉我们是否也应该刷新文件元数据(权限等)。下面是一个同时刷新数据和元数据的例子:
```
channel.force(true);
```
#### Java NIO 套接字信道

Java NIO SocketChannel是连接到TCP网络套接字的通道。它是Java NIO与Java网络的套接字的等效物。有两种方式，SocketChannel可以创建:

1. 您打开一个SocketChannel并连接到internet上某处的服务器。
2. 当传入连接到达ServerSocketChannel时，可以创建SocketChannel。

##### Opening a SocketChannel

如何打开SocketChannel:
```
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```
在使用后，通过调用SocketChannel.close()方法关闭SocketChannel。以下是如何做到的:
```
socketChannel.close();    
```
##### Reading from a SocketChannel

要从SocketChannel读取数据，需要调用一种read()方法。下面是一个例子:
```
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = socketChannel.read(buf);
```
首先分配一个缓冲区。从SocketChannel读取的数据被读入缓冲区。

然后调用SocketChannel.read()方法。这个方法将数据从SocketChannel读取到缓冲区中。read()方法返回的int告诉缓冲区中容纳了多少字节。如果返回-1，则到达流的末端(连接关闭)。

##### Writing to a SocketChannel

向SocketChannel写入数据是使用SocketChannel.write()方法完成的，该方法接受一个缓冲区作为参数。下面是一个例子:
```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```
注意在while循环中是如何调用SocketChannel.write()方法的。不能保证write()方法向SocketChannel写入了多少字节。因此，我们重复write()调用，直到缓冲区没有进一步的字节可写。

##### Non-blocking Mode

可以将SocketChannel设置为非阻塞模式,并可以在异步模式下调用connect()、read()和write()。

**connect()**

如果SocketChannel处于非阻塞模式，并且您调用connect()，该方法可能会在建立连接之前返回。要确定连接是否建立，可以调用finishConnect()方法，如下所示:
```
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...    
}
```

**write()**

在非阻塞模式下，write()方法可能在没有编写任何内容的情况下返回。因此，需要在循环中调用write()方法。但是，由于这已经在前面的write示例中完成了，所以这里不需要做任何不同的操作。

**read()**

在非阻塞模式下，read()方法可能根本没有读取任何数据就返回。因此，您需要注意返回的int，它告诉您读取了多少字节。

**Non-blocking Mode with Selectors**

SocketChannel的非阻塞模式与选择器的工作得更好。通过在选择器中注册一个或多个SocketChannel，您可以向选择器请求已准备好读取、写入等通道。如何使用选择器和SocketChannel将在本教程后面的文本中详细解释。

#### Java NIO ServerSocketChannel

Java NIO ServerSocketChannel是一个可以侦听传入TCP连接的通道，就像标准Java网络中的ServerSocket一样，ServerSocketChannel类位于java.nio.channels包。例如：
```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

##### Opening a ServerSocketChannel

您可以通过调用ServerSocketChannel.open()方法来打开ServerSocketChannel。这是它的样子:
```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```
关闭可以调用
```
serverSocketChannel.close();
```
##### Listening for Incoming Connections

通过调用ServerSocketChannel.accept()方法来监听传入的连接。当accept()方法返回时，它返回带有传入连接的SocketChannel。因此，accept()方法会阻塞，直到传入连接到达。

由于您通常对侦听单个连接不感兴趣，因此可以在while循环中调用accept()。这是它的样子:
```
while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    //do something with socketChannel...
}
```

当然，在while循环中，您可以使用一些其他的停止条件，而不是true。

##### Non-blocking Mode

可以将ServerSocketChannel设置为非阻塞模式。在非阻塞模式下，accept()方法立即返回，如果没有到达传入连接，则可能返回null。因此，您必须检查返回的SocketChannel是否为空。下面是一个例子:
```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel =
            serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
        }
}

```

#### Java NIO: Non-blocking Server

即使您了解Java NIO非阻塞特性(选择器、通道、缓冲区等)是如何工作的，设计一个非阻塞服务器仍然是困难的。非阻塞IO比阻塞IO包含几个难点的挑战。本非阻塞服务器教程将讨论非阻塞服务器的主要挑战，并描述一些可能的解决方案。

很难找到设计非阻塞服务器的好信息。因此，本教程中提供的解决方案是基于我自己的工作和想法。本教程中描述的思想是围绕Java NIO设计的。但是，我相信这些思想可以在其他语言中重用，只要它们具有某种类似选择器的构造。据我所知，底层操作系统提供了这样的结构，所以您很有可能也可以用其他语言访问它。

##### Non-blocking Server - GitHub Repository

I have created a simple proof-of-concept of the ideas presented in this tutorial and put it in a GitHub repository for you to look at. Here is the GitHub repository:

[https://github.com/jjenkov/java-nio-server]

##### Non-blocking IO Pipelines

非阻塞IO管道是处理非阻塞IO的组件链。这包括以非阻塞方式读取和写入IO。这里是一个简化的非阻塞IO管道的例子:

![wRAK5q.png](https://s1.ax1x.com/2020/09/17/wRAK5q.png)

组件使用选择器检查通道是否有要读取的数据。然后组件读取输入数据并根据输入生成一些输出。输出将再次写入一个通道。

非阻塞IO管道不需要同时读取和写入数据。有些管道可能只读取数据，有些管道可能只写入数据。

上面的图表只显示了单个组件。非阻塞IO管道可以有多个组件处理传入数据。非阻塞IO管道的长度取决于管道需要做什么。

非阻塞IO管道也可以同时从多个通道读取数据。例如，从多个socketchannel读取数据。

上图中的控制流也得到了简化。组件通过选择器从通道开始读取数据。并不是通道将数据推送到选择器中，然后再从那里推送到组件中，即使这是上面的图表所显示的。

##### Non-blocking vs. Blocking IO Pipelines

非阻塞IO管道和阻塞IO管道之间的最大区别是如何从底层通道(套接字或文件)读取数据。

IO管道通常从一些流(从套接字或文件)读取数据，并将数据分割成一致的消息。这类似于将数据流分解为记号，以便使用记号赋予器进行解析。相反，您将数据流分解为更大的消息。我将调用将流分解为消息读取器的消息的组件。下面是一个将消息流分解为消息的例子:
![wRAlGV.png](https://s1.ax1x.com/2020/09/17/wRAlGV.png)

阻塞的IO管道可以使用类似于inputstream的接口，其中可以从底层通道每次读取一个字节，并且类似于inputstream的接口会阻塞，直到有数据可供读取。这将导致阻塞消息读取器实现。

对流使用阻塞IO接口可以大大简化消息阅读器的实现。阻塞消息读取器永远不需要处理没有从流中读取数据的情况，或者只从流中读取了部分消息并且稍后需要恢复消息解析的情况。

类似地，阻塞消息写入器(将消息写入流的组件)永远不必处理只写入了消息的一部分，以及稍后必须恢复消息写入的情况。

##### Blocking IO Pipeline Drawbacks

虽然阻塞消息读取器更容易实现，但它有一个令人遗憾的缺点，即需要为需要拆分为消息的每个流使用单独的线程。这是必要的原因是，每个流的IO接口会一直阻塞，直到有一些数据可以从它读取。这意味着单个线程不能尝试从一个流读取数据，如果没有数据，就从另一个流读取数据。当线程试图从流中读取数据时，线程就会阻塞，直到确实有一些数据需要读取。

如果IO管道是服务器的一部分，它必须处理大量的并发连接，那么服务器将需要每个活动的导入连接一个线程。如果服务器在任何时候只有几百个并发连接，这可能不是问题。但是，如果服务器有数百万个并发连接，这种类型的设计就不能很好地扩展。每个线程将为其堆栈占用320K(32位JVM)和1024K(64位JVM)之间的内存。因此，1.000.000个线程将占用1 TB的内存!这是在服务器使用任何内存来处理传入消息之前(例如，分配给消息处理期间使用的对象的内存)。

为了减少线程的数量，许多服务器使用这样的设计:服务器保留一个线程池(例如100个)，每次读取一个入站连接的消息。入站连接保存在一个队列中，线程按照入站连接放入队列的顺序处理来自每个入站连接的消息。此设计在此说明:
![wRANZ9.png](https://s1.ax1x.com/2020/09/17/wRANZ9.png)

但是，这种设计要求入站连接合理地经常发送数据。如果入站连接处于非活动状态的时间更长，那么大量的非活动连接实际上可能阻塞线程池中的所有线程。这意味着服务器响应速度变慢，甚至没有响应。

有些服务器设计试图通过线程池中线程数量的弹性来缓解这个问题。例如，如果线程池耗尽了线程，线程池可能会启动更多线程来处理负载。这种解决方案意味着需要更多的低速连接才能使服务器失去响应。但是请记住，对于运行的线程数量仍然有一个上限。因此，这将不能很好地扩展1.000.000个慢速连接。

##### Basic Non-blocking IO Pipeline Design

非阻塞IO管道可以使用一个线程从多个流读取消息。这要求流可以切换到非阻塞模式。在非阻塞模式下，当您试图从流中读取数据时，流可能返回0或更多字节。如果流没有要读取的数据，则返回0字节。当流实际有一些数据要读取时，将返回1+字节。

为了避免检查需要读取0字节的流，我们使用Java NIO选择器。可以向选择器注册一个或多个SelectableChannel实例。当您在选择器上调用select()或selectNow()时，它只给您SelectableChannel实例，该实例实际上有要读取的数据。此设计在此说明:
![wRA4Rf.png](https://s1.ax1x.com/2020/09/17/wRA4Rf.png)

##### Reading Partial Messages

当我们从一个可选内存读取一个数据块时，我们不知道该数据块包含的消息是少是多。数据块可能包含部分消息(少于消息)、完整消息或多于消息，例如1.5或2.5条消息。 各种部分消息的可能性在这里说明:
![wRALon.png](https://s1.ax1x.com/2020/09/17/wRALon.png)

处理部分消息有两个挑战:
1. 检测数据块中是否有完整的消息。
2. 在消息的其余部分到达之前，如何处理部分消息。

检测完整消息要求消息读取器查看数据块中的数据，以确定数据是否包含至少一条完整消息。如果数据块包含一个或多个完整消息，则可以通过管道发送这些消息进行处理。寻找完整信息的过程将会重复很多次，所以这个过程必须尽可能快。

只要数据块中有部分消息，无论是单独的消息，还是在一条或多条完整消息之后，都需要存储该部分消息，直到该消息的其余部分从通道到达。

检测完整消息和存储部分消息都是消息读取器的职责。为了避免混合来自不同通道实例的消息数据，我们将为每个通道使用一个消息阅读器。设计是这样的:
![wRAxzT.png](https://s1.ax1x.com/2020/09/17/wRAxzT.png)

在检索到有数据要从选择器读取的通道实例之后，与该通道关联的消息读取器将读取数据并试图将其分解为消息。如果这导致读取任何完整消息，则可以通过read管道将这些消息传递到需要处理它们的任何组件。

消息阅读器当然是特定于协议的。消息阅读器需要知道它试图读取的消息的消息格式。如果我们的服务器实现要跨协议重用，它需要能够插入消息读取器实现——可能通过接受消息读取器工厂作为配置参数。

##### Storing Partial Messages

既然我们已经确定了在接收到完整消息之前存储部分消息是消息读取器的责任，那么我们需要确定应该如何实现该部分消息存储。

我们在设计时应该考虑两个方面:
1. 我们希望尽可能少地复制消息数据。复制越多，性能越低。
2. 我们希望将完整的消息存储在连续的字节序列中，以使解析消息更加容易。

**A Buffer Per Message Reader**

显然，部分消息需要存储在某种缓冲区中。直接的实现是在每个消息阅读器内部简单地具有一个缓冲区。但是，该缓冲区应该有多大？它必须足够大才能存储最大允许的消息。因此，如果最大允许的消息为1MB，则每个消息阅读器中的内部缓冲区至少需要为1MB。

当我们达到数百万个连接时，每个连接使用1MB并不能真正起作用。1.000.000 x 1MB仍然是1TB内存！如果最大邮件大小为16MB，该怎么办？还是128MB？

**Resizable Buffers**

另一个选择是实现可调整大小的缓冲区，以供在每个Message Reader中使用。可调整大小的缓冲区将从小处开始，如果消息对于该缓冲区而言太大，则该缓冲区将被扩展。这样，每个连接将不一定需要例如1MB的缓冲区。每个连接仅占用它们容纳下一条消息所需的内存。

**Resize by Copy**

实现可调整大小的缓冲区的第一种方法是从一个小缓冲区开始，例如4KB。如果消息无法放入4KB缓冲区中，则可以分配更大的缓冲区（例如8KB），并将来自4KB缓冲区的数据复制到更大的缓冲区中。

"Resize by Copy"缓冲区实现的优点是，一条消息的所有数据都保存在一个连续的字节数组中。这使得解析消息更加容易。

"Resize by Copy"缓冲区实现的缺点是，它将导致针对较大消息复制大量数据。

为了减少数据复制，您可以分析流经系统的消息的大小，以找到可以减少复制数量的某些缓冲区大小。例如，您可能会看到大多数消息都小于4KB，因为它们仅包含很小的请求/响应。这意味着第一个缓冲区大小应为4KB。

然后，您可能会看到，如果一条消息大于4KB，通常是因为它包含一个文件。然后，您可能会注意到，流经系统的大多数文件都小于128KB。然后，使第二缓冲区大小为128KB有意义。

最终，您可能会看到，一旦消息超过128KB，就没有真正的消息大小模式，因此，最终的缓冲区大小应该仅仅是最大消息大小。

使用这3种缓冲区大小（基于流经系统的消息大小），可以减少数据复制。4KB以下的消息将永远不会被复制。对于1.000.000并发连接，导致1.000.000 x 4KB = 4GB，这在今天（2015年）的大多数服务器中都是可能的。4KB和128KB之间的消息将被复制一次，并且仅4KB数据将需要复制到128KB缓冲区中。128KB和最大消息大小之间的消息将被复制两次。第一次将复制4KB，第二次将复制128KB，因此对于最大消息，总共将复制132KB。假设没有太多消息超过128KB，这是可以接受的。

一旦消息已被完全处理，分配的内存应再次释放。这样，从同一连接接收到的下一条消息将再次以最小的缓冲区大小开始。必须确保在连接之间可以更有效地共享内存。很有可能并非所有连接都同时需要大缓冲区。

**Resize by Append**

调整缓冲区大小的另一种方法是使缓冲区包含多个数组。当需要调整缓冲区大小时，您只需分配另一个字节数组，然后将数据写入该字节数组即可。

有两种方法可以增加这种缓冲区。一种方法是分配单独的字节数组，并保留这些字节数组的列表。另一种方法是分配更大的共享字节数组的切片，然后保留分配给缓冲区的切片的列表。就个人而言，我认为切片方法稍好一些，但差异很小。

通过将单独的数组或切片附加到缓冲区中来增加缓冲区的优点是，在写入过程中无需复制任何数据。可以将所有数据直接从套接字（Channel）直接复制到数组或切片中。

以这种方式增长缓冲区的缺点是数据不会存储在单个连续的数组中。这使消息解析更加困难，因为解析器需要同时查找每个单个数组的结尾和所有数组的结尾。由于您需要在书面数据中查找消息的结尾，因此该模型不太容易使用。

**TLV Encoded Messages**

某些协议消息格式使用TLV格式（类型，长度，值）进行编码。这意味着，当消息到达时，消息的总长度存储在消息的开头。这样，您立即知道要为整个消息分配多少内存。

TLV编码使内存管理更加容易。您立即知道要为该消息分配多少内存。在仅部分使用的缓冲区的末尾不会浪费任何内存。

TLV编码的一个缺点是，您必须在消息的所有数据到达之前为消息分配所有内存。因此，一些发送大消息的慢速连接可以分配所有可用的内存，从而使服务器无响应。

解决此问题的方法是使用一种消息格式，其中包含多个TLV字段。因此，为每个字段分配内存，而不是为整个消息分配内存，并且仅在字段到达时才分配内存。但是，大字段对大容量消息的影响与您的内存管理相同。

另一个解决方法是使在10到15秒内未收到的消息超时。这可以使您的服务器从同时出现的许多大消息中恢复过来，但仍会使服务器在一段时间内无响应。此外，故意的DoS（拒绝服务）攻击仍可能导致为服务器完全分配内存。

TLV编码存在不同的变体。确切地使用了多少字节，因此指定字段的类型和长度取决于每个单独的TLV编码。也有TLV编码将字段的长度放在首位，然后是类型，然后是值（LTV编码）。尽管字段的顺序不同，但这仍然是TLV的变体。

TLV编码使内存管理更容易的事实是HTTP 1.1如此糟糕的协议的原因之一。这是他们试图在HTTP 2.0中解决的问题之一，在HTTP 2.0中，数据以LTV编码的帧进行传输。这也是为什么我们为 使用TLV编码的VStack.co项目设计了自己的网络协议的原因。

##### Writing Partial Messages

在无阻塞的IO管道中，写入数据也是一个挑战。在 非阻塞模式下调用write(ByteBuffer)a时Channel，无法保证ByteBuffer其中写入了多少字节。该write(ByteBuffer)方法返回已写入的字节数，因此可以跟踪已写入的字节数。这就是挑战：跟踪部分写入的消息，以便最后发送一条消息的所有字节。

要管理将部分消息写入A，Channel我们将创建一个消息编写器。就像使用消息阅读器一样，每次Channel向其写入消息时，我们都需要一个消息编写器。在每个Message Writer内，我们都准确跟踪当前正在写入的消息的多少字节。

如果到达消息编写器的消息超出了直接写入的数量Channel，则需要在消息编写器内部将消息排队。然后，消息编写器将消息尽快写入Channel。

这是设计部分消息的图：
![wREVW6.png](https://s1.ax1x.com/2020/09/17/wREVW6.png)

为了使Message Writer能够发送仅部分发送的较早消息，需要不时调用Message Writer，以便可以发送更多数据。

如果您有很多连接，则将有很多Message Writer实例。检查例如一百万个Message Writer实例以查看它们是否可以写入任何数据很慢。首先，许多Message Writer实例很多没有任何要发送的消息。我们不想检查那些Message Writer实例。其次，并非所有Channel 实例都准备好向其中写入数据。我们不想浪费时间尝试将数据写入Channel 仍然无法接受任何数据的。

要检查是否Channel已准备好写入，您可以使用来注册频道Selector。但是，我们不希望在中注册所有Channel实例Selector。想象一下，如果您有1.000.000个连接，其中大多数都是空闲的，并且所有1.000.000个连接都已在上注册Selector。然后，当您调用select()这些Channel 实例中的大多数时，它们将准备好进行写操作（它们大多处于空闲状态，还记得吗？）。然后，您必须检查所有这些连接的消息编写器，以查看它们是否有任何数据要写入。

为了避免检查所有Message Writer实例中的消息，以及始终Channel没有任何消息要发送给它们的所有实例，我们使用以下两步方法：

当一个消息被写入到一个消息编写器，消息编写器相关联的注册其Channel 与Selector（如果它尚未注册）。

当服务器有时间时，它将检查Selector以查看哪些已注册Channel 实例可以进行写入。对于每个准备就绪Channel的写入，都要求其关联的消息编写器将数据写入Channel。如果消息编写器将其所有消息都写入 Channel，Channel则会从Selector再次注销。
这种分两步走的小方法可确保仅Channel将具有要写入消息的实例实际注册到Selector。

##### Putting it All Together
如您所见，非阻塞服务器需要不时检查传入数据，以查看是否接收到任何新的完整消息。服务器可能需要多次检查，直到收到一个或多个完整的消息。仅检查一次是不够的。

同样，非阻塞服务器需要不时检查是否有任何数据要写入。如果是，则服务器需要检查是否有任何相应的连接已准备好将数据写入其中。仅在消息第一次排队时进行检查是不够的，因为消息可能会被部分写入。

总而言之，一个无阻塞服务器最终需要定期执行的三个“管道”：

- 读取管道，用于检查来自打开的连接的新传入数据。
- 处理接收到的所有完整消息的处理管道。
- 用于检查是否可以将任何传出消息写入任何打开的连接的写管道。

这三个管道在一个循环中重复执行。您也许可以在某种程度上优化执行。例如，如果没有消息排队，则可以跳过写管道。或者，如果没有收到新的完整消息，则可以跳过流程管道。

整个服务器循环的图：
![wRE7p6.png](https://s1.ax1x.com/2020/09/17/wRE7p6.png)

资源地址
[https://github.com/jjenkov/java-nio-server]

##### Server Thread Model

GitHub存储库中的非阻塞服务器实现使用具有2个线程的线程模型。第一个线程接受来自的传入连接ServerSocketChannel。第二个线程处理接受的连接，这意味着读取消息，处理消息并将响应写回到连接。这两个线程模型如下所示：

![wREX0H.png](https://s1.ax1x.com/2020/09/17/wREX0H.png)

#### Java NIO 数据报通道

Java NIO DatagramChannel是可以发送和接收UDP数据包的通道。由于UDP是无连接的网络协议，因此默认情况下，不能用DatagramChannel读写内容。但是可以发送和接收数据包。

##### Opening a DatagramChannel

这是打开方式DatagramChannel：
```
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```
本示例打开一个DatagramChannel，它可以在UDP端口9999上接收数据包。

##### Receiving Data

通过调用它的receive()方法，可以从DatagramChannel接收数据，如下所示:
```
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();

channel.receive(buf);
```
receive()方法将接收到的数据包的内容复制到给定的缓冲区中。如果接收到的数据包所包含的数据超过了缓冲区所能包含的数据，那么剩余的数据将被静默地丢弃。

##### Sending Data

您可以通过DatagramChannel通过调用其send()方法发送数据，如下所示:
```
String newData = "New String to write to file..."
                    + System.currentTimeMillis();
    
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```
这个示例将字符串发送到UDP端口80上的“jenkov.com”服务器。没有任何东西在监听那个端口，所以什么也不会发生。您将不会被通知发送数据包是否被接收，因为UDP没有对数据的传递做出任何保证。

##### Connecting to a Specific Address

可以“连接” DatagramChannel到网络上的特定地址。由于UDP是无连接的，因此这种连接地址的方法不会像TCP通道那样建立真正的连接。而是，它锁定了您的身份，DatagramChannel因此您只能从一个特定地址发送和接收数据包。例如：
```
channel.connect(new InetSocketAddress("jenkov.com", 80));    
```
连接后，还可以使用read()和write()方法，就像使用传统通道一样。您无法保证发送的数据的交付。下面是一些例子:
```
int bytesRead = channel.read(buf);  
```
```
int bytesWritten = channel.write(buf);
```
#### Java NIO Pipe
Java NIO管道是两个线程之间的单向数据连接。管道具有源通道和接收通道。将数据写入接收通道。然后可以从源通道读取该数据。

下面是管道原理的说明:
![wRVYNR.png](https://s1.ax1x.com/2020/09/17/wRVYNR.png)

##### Creating a Pipe

您可以通过调用Pipe.open()方法来打开管道。这是它的样子:
```
Pipe pipe = Pipe.open();
```

##### Writing to a Pipe
要写入管道，您需要访问接收通道。以下是如何做到的:
```
Pipe.SinkChannel sinkChannel = pipe.sink();
```
你通过调用一个SinkChannel的write()方法来写入一个SinkChannel，就像这样:
```
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```
##### Reading from a Pipe
要从管道读取数据，您需要访问源通道。以下是如何做到的:
```
Pipe.SourceChannel sourceChannel = pipe.source();
```
要从源通道读取，可以像这样调用它的read()方法:
```
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```
read()方法返回的int表示有多少字节被读入缓冲区。

#### Java NIO vs. IO

Java NIO和IO之间的区别

##### Main Differences Betwen Java NIO and IO
下表总结了Java NIO和IO之间的主要区别。
![wRVO2V.png](https://s1.ax1x.com/2020/09/17/wRVO2V.png)

##### Stream Oriented vs. Buffer Oriented

面向流的Java IO意味着您一次从流中读取一个或多个字节。您对读取字节的处理取决于您自己。它们不会在任何地方缓存。此外，您无法在流中的数据中来回移动。如果需要来回移动从流中读取的数据，则需要先将其缓存在缓冲区中。

Java NIO的面向缓冲区的方法略有不同。数据被读入缓冲区，以后再从缓冲区中进行处理。您可以根据需要在缓冲区中来回移动。这使您在处理过程中更具灵活性。但是，您还需要检查缓冲区是否包含您需要的所有数据，以便对其进行完全处理。并且，您需要确保在将更多数据读入缓冲区时，不要覆盖尚未处理的缓冲区中的数据。

##### Blocking vs. Non-blocking IO

Java IO的各种流正在阻塞。这意味着，当线程调用一次read()或write()方法，该线程将被阻塞，直到有一些数据要读取或数据被完全写入为止。在此期间，线程无法执行其他任何操作。

Java NIO的非阻塞模式使线程可以请求从通道读取数据，并且仅获取当前可用的数据，或者如果当前没有可用数据，则什么都不获取。线程可以继续进行其他操作，而不是在数据可供读取之前保持阻塞状态。

非阻塞写入也是如此。线程可以请求将某些数据写入通道，但不等待将其完全写入。然后线程可以继续运行，同时执行其他操作。

当没有阻塞IO调用时，哪些线程会花费空闲时间，通常在此期间在其他通道上执行IO。也就是说，单个线程现在可以管理输入和输出的多个通道。

##### Selectors

Java NIO的选择器允许单个线程监视多个输入通道。您可以使用选择器注册多个通道，然后使用单个线程"select"具有可用于处理输入的通道，或选择准备好写入的通道。这种选择器机制使单个线程可以轻松管理多个通道。

##### How NIO and IO Influences Application Design

您选择NIO还是IO作为IO工具包可能会影响应用程序设计的以下方面：
1. API调用NIO或IO类。
2. 数据处理。
3. 用于处理数据的线程数。

**The API Calls**

当然，使用NIO时的API调用看起来与使用IO时不同。这并不奇怪。数据必须首先被读入缓冲区，然后从那里进行处理，而不是从例如InputStream一个字节一个字节地读取数据。

**The Processing of Data**

与使用IO设计相比，使用纯NIO设计时数据处理也会受到影响。

在IO设计中，你从一个输入流或读取器一个字节一个字节地读取数据。假设您正在处理基于行的文本数据流。例如:
```
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```
这个文本行流可以这样处理:
```
InputStream input = ... ; // get the InputStream from the client socket

BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```
注意，如何通过程序执行的距离来确定处理状态。换句话说，一旦第一个reader.readLine()方法返回，就可以确定已经读取了整行文本。这readLine()就是为什么直到读取整行为止的块。您还知道此行包含名称。同样，当第二个readLine()有返回时，您知道此行包含年龄等。

如您所见，该程序仅在有新数据要读取时才继续运行，并且对于每个步骤，您都知道该数据是什么。一旦执行线程的进度超过了读取代码中的特定数据段，该线程就不会在数据中向后移动（大多数情况下不会）。此原理也在此图中说明：
![wRZ4Rx.png](https://s1.ax1x.com/2020/09/17/wRZ4Rx.png)

NIO实现看起来会有所不同。这是一个简化的示例：
```
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);
```
请注意第二行，该行从通道读取字节到中ByteBuffer。当该方法调用返回时，您不知道所需的所有数据是否都在缓冲区内。您所知道的是缓冲区包含一些字节。这使处理有些困难。

试想一下，如果在第一次read(buffer)调用之后，读入缓冲区的所有内容都是半行。例如:"Name: An"。您可以处理这些数据吗？并不是的。您需要等到至少一整行数据都已放入缓冲区，然后才可以处理所有数据。

那么如何知道缓冲区是否包含足够的数据以使其有意义呢？好吧，你没有。找出的唯一方法是查看缓冲区中的数据。结果是，您可能必须多次检查缓冲区中的数据，然后才能知道是否所有数据都在其中。这既效率低下，又可能使程序设计变得混乱。例如：
```
ByteBuffer buffer = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buffer);

while(! bufferFull(bytesRead) ) {
    bytesRead = inChannel.read(buffer);
}
```
bufferFull()方法跟踪有多少数据被读入缓冲区，并根据缓冲区是否满返回true或false。换句话说，如果缓冲区已准备好进行处理，则认为它已满。

bufferFull()方法扫描缓冲区，但必须使缓冲区保持与调用bufferFull()方法之前相同的状态。否则，下一个读入缓冲区的数据可能不会被读入正确的位置。这并非不可能，但这又是一个需要注意的问题。

##### Summary

NIO允许您仅使用一个（或几个）线程来管理多个通道（网络连接或文件），但是代价是解析数据可能比从阻塞流中读取数据更为复杂。

如果您需要同时管理数千个打开的连接（每个连接仅发送少量数据），例如聊天服务器，则在NIO中实现该服务器可能是一个优势。同样，如果您需要保持与其他计算机的大量开放连接，例如在P2P网络中，则使用单个线程来管理所有出站连接可能是一个优势。

如果只有很少的连接具有很高的带宽，一次发送大量数据，那么经典的IO服务器实现也许是最合适的选择。下图说明了经典的IO服务器设计：

![wRl4KI.png](https://s1.ax1x.com/2020/09/17/wRl4KI.png)

#### Java NIO Path

Java Path接口是Java NIO2更新的一部分，在Java 6和Java 7中。Java Path接口是在Java 7中添加到Java NIO中的，Path接口位于java.nio.file中。因此Java Path接口的完全限定名是Java.nio.file.Path。

Java Path实例表示文件系统中的一个路径。路径既可以指向文件，也可以指向目录。路径可以是绝对的，也可以是相对的。绝对路径包含从文件系统的根到它所指向的文件或目录的完整路径。相对路径包含文件或目录相对于其他路径的路径。相对路径可能听起来有点混乱。

在某些操作系统中，不要混淆文件系统路径和path环境变量。java.nio.file.Path接口与Path环境变量无关。

在很多方面，java.nio.file.Path接口类似于java.io.File类，但是有一些小的区别。但是在许多情况下，可以用Path接口替换File类的使用。

##### Creating a Path Instance
为了使用java.nio.file.Path实例，需要创建一个Path实例。使用Paths类(java.nio.file.Paths)中的名为Paths.get()的静态方法创建一个Path实例。下面是一个Java Paths.get()的例子:
```
import java.nio.file.Path;
import java.nio.file.Paths;

public class PathExample {

    public static void main(String[] args) {

        Path path = Paths.get("c:\\data\\myfile.txt");

    }
}
```
请注意示例顶部的两个import语句。要使用Path接口和Paths类，我们必须首先导入它们。

 其次，注意Path.get("c:\\data\\myfile.txt")方法调用。正是对Path .get()方法的调用创建了Path实例。换句话说，Path.get()方法是路径实例的工厂方法。

##### Creating an Absolute Path

通过使用绝对文件作为参数调用Paths.get()工厂方法来创建绝对路径。下面是一个创建代表绝对路径的路径实例的例子:
```
Path path = Paths.get("c:\\data\\myfile.txt");
```
绝对路径是c:\data\myfile.txt。双\字符在Java字符串中是必需的，因为\是一个转义字符，这意味着下面的字符将告诉字符串中的这个位置实际上是什么字符。通过编写\\，可以告诉Java编译器在字符串中写入一个\字符。

上面的路径是Windows文件系统路径。在Unix系统(Linux, MacOS, FreeBSD等)上，上面的绝对路径看起来像这样:

```
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

绝对路径现在是/home/jakobjenkov/myfile.txt。

如果在Windows机器上使用这种路径(以/开头的路径)，则该路径将被解释为相对于当前驱动器的路径。例如，路径
```
/home/jakobjenkov/myfile.txt
```
可以被解释为位于C驱动器上。然后路径对应于完整路径:
```
C:/home/jakobjenkov/myfile.txt
```
##### Creating a Relative Path

相对路径是指从一个路径(基本路径)指向一个目录或文件的路径。相对路径的完整路径(绝对路径)是通过结合基本路径和相对路径得到的。

Java NIO Path类还可以用于处理相对路径。您可以使用这些路径创建一个相对路径。得到(basePath relativePath)方法。下面是两个Java中相对路径的例子:

```
Path projects = Paths.get("d:\\data", "projects");

Path file = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

第一个示例创建了一个Java Path实例，它指向路径(目录)d:\data\项目。第二个示例创建了一个路径实例，它指向路径(文件)d:\data\projects\a-project\myfile.txt。

在使用相对路径时，可以在路径字符串中使用两个特殊代码。这些代码是:
- .
- ..

'.'的意思是“当前目录”。例如，如果你像这样创建一个相对路径:

```
Path currentDir = Paths.get(".");
System.out.println(currentDir.toAbsolutePath());
```

那么Java path实例所对应的绝对路径就是执行上述代码的应用程序所在的目录。

如果.在路径字符串的中间使用，它只是表示路径在该点所指向的同一目录。这里有一个路径的例子说明:
```
Path currentDir = Paths.get("d:\\data\\projects\.\a-project");
```
此路径对应于以下路径:
```
d:\data\projects\a-project
```
'..'代码的意思是“父目录”或“一个目录”。这里有一个Path Java的例子说明:

```
Path parentDir = Paths.get("..");
```
此示例创建的Path实例将对应于启动运行此代码的应用程序所在目录的父目录。

如果你使用'..'路径字符串中间的代码，它对应于在路径字符串的那个点上改变一个目录。例如:
```
String path = "d:\\data\\projects\\a-project\\..\\another-project";
Path parentDir2 = Paths.get(path);
```
这个例子创建的Java路径实例将对应于这个绝对路径:
```
d:\data\projects\another-project
```
'..'在a-project目录之后的代码更改了父目录项目的目录，然后路径从那里向下引用到另一个-project目录。

'.'和'..'代码还可以与两字符串路径.get()方法结合使用。下面是两个Java path .get()示例，展示了简单的例子:
```
Path path1 = Paths.get("d:\\data\\projects", ".\\a-project");

Path path2 = Paths.get("d:\\data\\projects\\a-project",
                       "..\\another-project");
```
可以使用Java NIO Path类处理相对路径的方法还有很多。您将在本教程的后面部分了解更多相关内容。

##### Path.normalize()
Path接口的normalize()方法可以规范化路径。标准化意味着它消除了。和. .路径字符串中间的代码，并解析路径字符串引用的路径。下面是一个Java路径。normalize()示例:
```
String originalPath =
        "d:\\data\\projects\\a-project\\..\\another-project";

Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);

Path path2 = path1.normalize();
System.out.println("path2 = " + path2);
```
这个路径示例首先创建一个带有..在中间编码。然后，示例从这个路径字符串创建一个路径实例，并打印出该路径实例(实际上它打印的是Path. tostring())。

然后，该示例对创建的Path实例调用normalize()，它将返回一个新的Path实例。然后还会打印出这个新的规范化的Path实例。

下面是上面例子输出的结果:
```
path1 = d:\data\projects\a-project\..\another-project
path2 = d:\data\projects\another-project
```
正如您所看到的，规范化路径不包含a-project\..部分，因为这是多余的。被删除的部分没有添加到最终的绝对路径。
#### Java NIO Files

Java NIO文件类(java.nio.file.Files) 提供了几种在文件系统中操作文件的方法。本Java NIO文件教程将介绍这些方法中最常用的方法。Files类包含许多方法，所以如果您需要这里没有描述的方法，也请检查JavaDoc.file类可能仍然有一个方法。

java.nio.file.Files类与java.nio.file.Path实例一起工作。因此，在使用Files类之前，您需要了解Path类。

##### Files.exists()

该Files.exists()方法检查给定Path文件系统中是否存在给定文件。

可以创建Path文件系统中不存在的实例。例如，如果您打算创建一个新目录，则首先要创建相应的Path实例，然后再创建目录。

由于Path实例可能指向文件系统中存在的路径，也可能不指向文件系统中存在的路径，因此可以使用该Files.exists()方法确定它们是否存在（以防需要检查）。

Java Files.exists()示例：
```
Path path = Paths.get("data/logging.properties");

boolean pathExists =
        Files.exists(path,
            new LinkOption[]{ LinkOption.NOFOLLOW_LINKS});
```
这个例子首先创建一个Path实例，指向我们想要检查的路径是否存在。其次，该示例使用Path实例作为第一个参数调用Files.exists()方法。

注意Files.exists()方法的第二个参数。此参数是一个选项数组，它影响file .exists()如何确定路径是否存在。在上面的这个例子中，数组包含了LinkOption.NOFOLLOW_LINKS，这意味着Files.exists()方法不应该遵循文件系统中的符号链接来确定路径是否存在。

##### Files.createDirectory()
createdirectory()方法从一个Path实例创建一个新目录。下面是一个Java Files.createDirectory()示例:
```
Path path = Paths.get("data/subdir");

try {
    Path newDir = Files.createDirectory(path);
} catch(FileAlreadyExistsException e){
    // the directory already exists.
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```
第一行创建了表示要创建的目录的Path实例。在try-catch块中，以路径作为参数调用Files.createDirectory()方法。如果创建目录成功，则返回一个Path实例，该实例指向新创建的路径。

如果该目录已经存在，将抛出java.nio.file.FileAlreadyExistsException。如果出现其他错误，可能会抛出IOException。例如，如果所需的新目录的父目录不存在，则可能会抛出IOException。父目录是您要在其中创建新目录的目录。因此，它意味着新目录的父目录。

##### Files.copy()

copy()方法将文件从一条路径复制到另一条路径。下面是一个Java NIO Files.copy()例子：

```
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath);
} catch(FileAlreadyExistsException e) {
    //destination file already exists
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```

首先，示例创建了一个源和目标路径实例。然后，示例调用Files.copy()，将两个路径实例作为参数传递。这将导致源路径引用的文件被复制到目标路径引用的文件中。

如果目标文件已经存在，则使用java.nio.file.FileAlreadyExistsException抛出。如果出现其他错误，就会抛出IOException。例如，如果要将文件复制到的目录不存在，就会抛出IOException。

##### Overwriting Existing Files
可以强制file. copy()覆盖现有文件。下面的例子展示了如何使用file .copy()覆盖现有文件:

```
Path sourcePath      = Paths.get("data/logging.properties");
Path destinationPath = Paths.get("data/logging-copy.properties");

try {
    Files.copy(sourcePath, destinationPath,
            StandardCopyOption.REPLACE_EXISTING);
} catch(FileAlreadyExistsException e) {
    //destination file already exists
} catch (IOException e) {
    //something else went wrong
    e.printStackTrace();
}
```
请注意Files.copy()方法的第三个参数。此参数指示copy()方法在目标文件已经存在的情况下覆盖现有文件。

##### Files.move()

Java NIO Files类还包含一个函数，用于将文件从一个路径移动到另一个路径。移动文件和重命名文件类似，在一个操作中，移动文件可以同时将其移动到不同的目录并重命名。java.io.File类也可以通过它的renameTo()方法来实现这一点，现在在java.nio.file.Files中也有了这个功能。

下面是一个Java Files.move()的例子:
```
Path sourcePath      = Paths.get("data/logging-copy.properties");
Path destinationPath = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.move(sourcePath, destinationPath,
            StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
    //moving file failed.
    e.printStackTrace();
}
```
首先创建源路径和目标路径。源路径指向要移动的文件，而目标路径指向应该移动文件的位置。然后调用Files.move()方法。这将导致文件被移动。

请注意传递给Files.move()的第三个参数。此参数告诉Files.move()方法覆盖目标路径上的任何现有文件。这个参数实际上是可选的。

move()方法可能会在移动文件失败时抛出IOException。例如，如果目标路径上已经存在一个文件，并且您忽略了StandardCopyOption.REPLACE_EXISTING选项，或者如果要移动的文件不存在等。

##### Files.delete()

delete()方法可以删除文件或目录。下面是一个Java Files.delete()的例子:

```
Path path = Paths.get("data/subdir/logging-moved.properties");

try {
    Files.delete(path);
} catch (IOException e) {
    //deleting file failed
    e.printStackTrace();
}
```
首先创建指向要删除的文件的路径。然后调用Files.delete()方法。如果file .delete()由于某些原因(例如，该文件或目录不存在)未能删除该文件，则抛出IOException。

##### Files.walkFileTree()

walkfiletree()方法包含递归遍历目录树的功能。walkFileTree()方法接受一个路径实例和一个FileVisitor作为参数。Path实例指向您想要遍历的目录。在反转期间调用FileVisitor。

在我解释遍历如何工作之前，这里首先是FileVisitor接口:
```
public interface FileVisitor {

    public FileVisitResult preVisitDirectory(
        Path dir, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFile(
        Path file, BasicFileAttributes attrs) throws IOException;

    public FileVisitResult visitFileFailed(
        Path file, IOException exc) throws IOException;

    public FileVisitResult postVisitDirectory(
        Path dir, IOException exc) throws IOException {

}
```
您必须自己实现FileVisitor接口，并将实现的实例传递给walkFileTree()方法。在遍历目录期间，FileVisitor实现的每个方法将在不同的时间被调用。如果您不需要连接到所有这些方法，那么您可以扩展SimpleFileVisitor类，它包含FileVisitor接口中所有方法的默认实现。

下面是一个walkFileTree()的例子:
```
Files.walkFileTree(path, new FileVisitor<Path>() {
  @Override
  public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
    System.out.println("pre visit dir:" + dir);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
    System.out.println("visit file: " + file);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
    System.out.println("visit file failed: " + file);
    return FileVisitResult.CONTINUE;
  }

  @Override
  public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
    System.out.println("post visit directory: " + dir);
    return FileVisitResult.CONTINUE;
  }
});
```

在遍历过程中，FileVisitor实现中的每个方法都会在不同的时间被调用:

在访问任何目录之前调用preVisitDirectory()方法。在访问一个目录之后调用postVisitDirectory()方法。

在遍历文件期间，对访问的每个文件调用visitFile() mehtod。它不用于目录—只用于文件。在访问文件失败时调用visitFileFailed()方法。例如，如果您没有正确的权限，或者出现了其他错误。

这四个方法中的每一个都返回一个FileVisitResult枚举实例。FileVisitResult枚举包含以下四个选项:

- CONTINUE
- TERMINATE
- SKIP_SIBLINGS
- SKIP_SUBTREE

通过返回这些值中的一个，被调用的方法可以决定文件遍历应该如何继续。

CONTINUE表示文件遍历应该正常继续。  

TERMINATE意味着文件遍历现在应该终止。

SKIP_SIBLINGS表示继续执行文件遍历，但不访问此文件或目录的任何兄弟文件。

SKIP_SUBTREE表示继续执行文件遍历，但不访问此目录中的条目。这个值只有在从preVisitDirectory()返回时才有一个函数。如果从任何其他方法返回，它将被解释为一个CONTINUE。

##### Searching For Files
下面是一个walkFileTree()，它扩展了SimpleFileVisitor来寻找一个名为README.txt的文件:
```
Path rootPath = Paths.get("data");
String fileToFind = File.separator + "README.txt";

try {
  Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
    
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      String fileString = file.toAbsolutePath().toString();
      //System.out.println("pathString = " + fileString);

      if(fileString.endsWith(fileToFind)){
        System.out.println("file found at path: " + file.toAbsolutePath());
        return FileVisitResult.TERMINATE;
      }
      return FileVisitResult.CONTINUE;
    }
  });
} catch(IOException e){
    e.printStackTrace();
}
```

##### Deleting Directories Recursively

walkfiletree()还可以用于删除包含所有文件和子目录的目录。delete()方法只在目录为空时删除该目录。通过遍历所有目录并删除每个目录中的所有文件(在visitFile()中)，然后删除目录本身(在postVisitDirectory()中)，您可以删除包含所有子目录和文件的目录。下面是一个递归目录删除示例:
```
Path rootPath = Paths.get("data/to-delete");

try {
  Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      System.out.println("delete file: " + file.toString());
      Files.delete(file);
      return FileVisitResult.CONTINUE;
    }

    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
      Files.delete(dir);
      System.out.println("delete dir: " + dir.toString());
      return FileVisitResult.CONTINUE;
    }
  });
} catch(IOException e){
  e.printStackTrace();
}
```

##### Additional Methods in the Files Class

java.nio.file.Files类包含许多其他有用的函数，比如创建符号链接的函数、确定文件大小的函数、设置文件权限的函数等。在JavaDoc中查找，java.nio.file.Files有关这些方法的更多信息。


#### Java NIO 异步文件通道

在Java 7中，AsynchronousFileChannel被添加到Java NIO。AsynchronousFileChannel使得可以异步地从文件读取数据和向文件写入数据。

##### Creating an AsynchronousFileChannel

可以通过其静态方法open()创建一个AsynchronousFileChannel。下面是一个创建AsynchronousFileChannel的例子:

```
Path path = Paths.get("data/test.xml");

AsynchronousFileChannel fileChannel =
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);
```
open()方法的第一个参数是一个路径实例，它指向要与AsynchronousFileChannel关联的文件。

第二个参数是一个或多个open选项，它告诉AsynchronousFileChannel要对底层文件执行哪些操作。在本例中，我们使用StandardOpenOption.READ表示文件将被打开进行读取。

##### Reading Data

AsynchronousFileChannel可以通过两种方式读取数据。每一种读取数据的方法都调用AsynchronousFileChannel的read()方法之一。下面几节将介绍这两种读取数据的方法。

**Reading Data Via a Future**

AsynchronousFileChannel读取数据的第一种方法是调用read()方法，该方法返回Future。下面是调用read()方法的样子:

```
Future<Integer> operation = fileChannel.read(buffer, 0);
```

这个版本的read()方法将ByteBuffer作为第一个参数。AsynchronousFileChannel读取的数据被读入这个ByteBuffer。第二个参数是文件中开始读取的字节位置。

即使读操作还没有完成，read()方法也会立即返回。您可以通过调用read()方法返回的将来实例的isDone()方法来检查读操作何时完成。

下面是一个完整的示例，展示了如何使用read()方法:
```
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.READ);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

Future<Integer> operation = fileChannel.read(buffer, position);

while(!operation.isDone());

buffer.flip();
byte[] data = new byte[buffer.limit()];
buffer.get(data);
System.out.println(new String(data));
buffer.clear();
```
这个示例创建了一个AsynchronousFileChannel，然后创建了一个ByteBuffer，它作为参数传递给read()方法，并且位置为0。在调用read()之后，示例循环直到返回的Future的isDone()方法返回true。当然，这不是一个非常有效的CPU使用-但您需要等待，直到读取操作已经完成。

读取操作完成后，数据将读入ByteBuffer，然后读入字符串并打印System.out。

**Reading Data Via a CompletionHandler**

AsynchronousFileChannel读取数据的第二种方法是调用read()方法版本，该版本采用CompletionHandler作为参数。下面是调用read()方法的方法:
```
fileChannel.read(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {
    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("result = " + result);

        attachment.flip();
        byte[] data = new byte[attachment.limit()];
        attachment.get(data);
        System.out.println(new String(data));
        attachment.clear();
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {

    }
});
```
读取操作完成后，将调用CompletionHandler的completed()方法。作为completed()方法的参数，会传递一个整数，告诉读取了多少字节，以及传递给read()方法的"attachment"。"attachment"是read()方法的第三个参数。在这种情况下，数据也被读取到ByteBuffer中。您可以自由选择要附加的对象。

如果读操作失败，则会调用CompletionHandler的failed()方法。

##### Writing Data

就像读取一样，您可以通过两种方式将数据写入一个AsynchronousFileChannel。每一种写入数据的方法都调用异步filechannel的write()方法之一。下面几节将介绍这两种写入数据的方法。

**Writing Data Via a Future**
AsynchronousFileChannel允许异步写入数据。这是一个完整的Java异步filechannel写的例子:
```
Path path = Paths.get("data/test-write.txt");
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

Future<Integer> operation = fileChannel.write(buffer, position);
buffer.clear();

while(!operation.isDone());

System.out.println("Write done");
```
首先在写模式下打开一个AsynchronousFileChannel。然后创建一个ByteBuffer，并将一些数据写入其中。然后将ByteBuffer中的数据写入文件。最后，示例检查返回的Future以查看写操作何时完成。

注意，在此代码工作之前，该文件必须已经存在。如果该文件不存在，那么write()方法将抛出一个java.nio.file.NoSuchFileException。

您可以通过以下代码确保路径指向的文件存在:

```
if(!Files.exists(path)){
    Files.createFile(path);
}
```
**Writing Data Via a CompletionHandler**

您还可以使用CompletionHandler向AsynchronousFileChannel写入数据，以告诉您写入何时完成，而不是将来。下面是一个使用CompletionHandler向AsynchronousFileChannel写入数据的例子:

```
Path path = Paths.get("data/test-write.txt");
if(!Files.exists(path)){
    Files.createFile(path);
}
AsynchronousFileChannel fileChannel = 
    AsynchronousFileChannel.open(path, StandardOpenOption.WRITE);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

buffer.put("test data".getBytes());
buffer.flip();

fileChannel.write(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {

    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("bytes written: " + result);
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
        System.out.println("Write failed");
        exc.printStackTrace();
    }
});
```

当写操作完成时，CompletionHandler的completed()方法将被调用。如果写入操作由于某种原因失败，则会调用failed()方法。

请注意ByteBuffer是如何作为附件使用的——它是传递给CompletionHandler方法的对象。

--- 完结 ---

另附上[参考网站](http://tutorials.jenkov.com/java-nio/index.html)。