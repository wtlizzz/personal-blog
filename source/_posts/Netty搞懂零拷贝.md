title: Netty搞懂零拷贝
author: Wtli
tags:
  - Netty
categories:
  - 后端
date: 2020-12-09 11:00:00
---
最近弄Java Nio通信，发现Netty有零拷贝技术，号称性能更高，今天来学习一下。
<!--more-->

<center>
𝕵~~大橘护体,文章必过~~𝕵
</center>

<img style="margin: auto;" src="https://s3.ax1x.com/2020/12/09/rCiZ5D.png" width="300" height="300" />


Netty的“零拷贝”主要体现在如下三个方面：

#### 缓冲区内存拷贝

Netty的接收和发送ByteBuffer采用DIRECT BUFFERS，使用堆外直接内存进行Socket读写，不需要进行字节缓冲区的二次拷贝。

如果使用传统的堆内存（HEAP BUFFERS）进行Socket读写，JVM会将堆内存Buffer拷贝一份到直接内存中，然后才写入Socket中。相比堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。

自己去翻了翻write的源码，意思大概是：

**普通的ByteBuffer用的时候比DirectBuffer多拷贝一次。**
- 在write源码中①显示，如果src是DirectBuffer的实现类就能够直接调用writeFromNativeBuffer方法，写入到Native中。

- 如果src不是DirectBuffer的实现类的话，这说明ByteBuffer是在堆内存中的（后面会说堆内存和直接内存的buffer）会先定义一个直接内存的ByteBuffer bb = Util.getTemporaryDirectBuffer(rem)，进行一次堆内存的拷贝 bb.put(src)，bb.flip()，拷贝到直接内存的buffer中，然后在写入到native中，这样就硬生生的多了一次拷贝。

```
static int write(FileDescriptor fd, ByteBuffer src, long position,
                     NativeDispatcher nd)
        throws IOException
    {
        if (src instanceof DirectBuffer) //  ①
            return writeFromNativeBuffer(fd, src, position, nd);

        // Substitute a native buffer
        int pos = src.position();
        int lim = src.limit();
        assert (pos <= lim);
        int rem = (pos <= lim ? lim - pos : 0);
        ByteBuffer bb = Util.getTemporaryDirectBuffer(rem);
        try {
            bb.put(src);
            bb.flip();
            // Do not update src until we see how many bytes were written
            src.position(pos);

            int n = writeFromNativeBuffer(fd, bb, position, nd);
            if (n > 0) {
                // now update src
                src.position(pos + n);
            }
            return n;
        } finally {
            Util.offerFirstTemporaryDirectBuffer(bb);
        }
    }
```

**再看Netty：**

Netty刚看了一下，给ByteBuf分配资源，和Java Nio一样，也是两种分配方式,一种是分配的堆内存，一种是直接内存，但是《权威指南》上说是默认的是directBuffer直接内存，那就这样去理解吧，Netty为了提高效率默认是使用的零拷贝的直接内存：

```
        ByteBuf byteBuf = Unpooled.buffer(4);
        ByteBuf byteBuf1 = Unpooled.directBuffer(4);
```


附：

DirectBuffer是一个接口类型interface，一个类只能有一个实现的接口，但能够多个继承的父类。

ByteBuffer是一个抽象类abstract。

也就是说并不是Netty的产生，诞生了零拷贝的概念。是在Java Nio中，本身就两种ByteBuffer。在ByteBuffer源码中有两种allocate方式：

```
   //①
   public static ByteBuffer allocate(int capacity) {
        if (capacity < 0)
            throw new IllegalArgumentException();
        return new HeapByteBuffer(capacity, capacity);
   }
   //②
   public static ByteBuffer allocateDirect(int capacity) {
        return new DirectByteBuffer(capacity);
   }
```

- ①的内存分配方式就是返回了一个HeapByteBuffer，堆内存的buffer。
- ②的内存分配方式是返回了一个DirectByteBuffer，直接内存buffer。
- 如果DirectByteBuffer真的这么好，那为什么还有HeapByteBuffer的产生呢？网上有的说是直接内存读取快，但是资源分配慢，这个没有测试，应该对于数据量小的话还是使用直接内存更好。

#### CompositeByteBuf

下面看第二种“零拷贝”的实现CompositeByteBuf，它对外将多个ByteBuf封装成一个ByteBuf，对外提供统一封装后的ByteBuf接口。

![rCi8Vf.png](https://s3.ax1x.com/2020/12/09/rCi8Vf.png)

实际上是一个ByteBuf的装饰器，将多个ByteBuf组成一个集合components，然后对外提供统一的接口。

```
    private static final ByteBuffer EMPTY_NIO_BUFFER = Unpooled.EMPTY_BUFFER.nioBuffer();
    private static final Iterator<ByteBuf> EMPTY_ITERATOR = Collections.<ByteBuf>emptyList().iterator();

    private final ByteBufAllocator alloc;
    private final boolean direct;
    private final int maxNumComponents;

    private int componentCount;
    private Component[] components;
```

#### 文件传输零拷贝

第三种“零拷贝”就是文件传输，Netty文件传输类DefaultFileRegion通过transferTo方法将文件发送到目标Channel中。

直接使用的Java Nio方法，Netty应该是基于Java Nio与其他不是基于Java Nio的通信进行比较的。






*部分摘自Netty的权威指南*