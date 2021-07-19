title: Java多线程4-JVM线程
author: Wtli
date: 2021-07-13 15:27:51
tags:
---
Java多线程的使用

- JVM可以建立多少线程
- Java内存模型
- 内存配置与并发

<!--more-->

一台电脑运行Java项目，使用线程池或者新建线程，具体数量设置为多少能够正常运行？是几个还是几十个还是多少？

首先要知道Java新建一个线程，主要是占用了哪部分内存。先看一下Java的内存模型。

#### Java内存模型

![upload successful](/images/pasted-82.png)

在这里不详细的对每一部分进行解释，仅了解一下堆内存和栈内存。

<font color='red'>Java虚拟机栈</font>
与程序计数器一样，Java 虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。  
虚拟机栈描述的是 Java 方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧（Stack Frame，是方法运行时的基础数据结构）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。  
在活动线程中，只有位于栈顶的帧才是有效的，称为当前栈帧。正在执行的方法称为当前方法，栈帧是方法运行的基本结构。在执行引擎运行时，所有指令都只能针对当前栈帧进行操作。
![upload successful](/images/pasted-83.png)

<font color='red'>Java堆</font>
对于大多数应用来说，Java 堆（Java Heap）是 Java 虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。  
堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC堆”（Garbage Collected Heap）。从内存回收的角度来看，由于现在收集器基本都采用分代收集算法，所以 Java 堆中还可以细分为：新生代和老年代；再细致一点的有 Eden 空间、From Survivor 空间、To Survivor 空间等。从内存分配的角度来看，线程共享的 Java 堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer,TLAB）。  
Java 堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，当前主流的虚拟机都是按照可扩展来实现的（通过 -Xmx 和 -Xms 控制）。如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出 OutOfMemoryError 异常。

#### 线程所使用的JVM内存

线程常见的创建方式主要有三种，分别是
1. new Thread()
2. Executors
3. new ThreadPoolExecutor()

栈内存是线程私有的，生命周期与线程相同，一个线程都有自己的一个虚拟机栈的单独内存。



#### JVM可以建立多少个线程？

影响线程数量的因素有下面几个：

|配置|配置信息|
|---|---|
|-Xms| intial java heap size|
|-Xmx |maximum java heap size|
|-Xss|the stack size for each thread|
|系统限制|系统最大可开线程数|

***

|测试环境|配置|
|--|--|
|系统|Ubuntu 10.04 Linux Kernel 2.6 （32位）|
|内存|2G|
|JDK|1.7|

***

测试结果：

|-Xms|-Xmx|-Xss|结果|
|--|--|--|--|
|1024m|1024m|1024k|1737|
|1024m|1024m|64k|26077|
|512m|512m|64k|31842|
|256m|256m|64k|31842|

在创建的线程数量达到31842个时，系统中无法创建任何线程。



#### 堆内存（Heap）越大限制并发量

首先要知道几个点：

1. 不同的机器上，每个进程有自己最大可使用的内存（可以设置）。比如：x86的机器上的进程最多可以使用2048mb的内存， 
2. 每个JVM线程都有一个私有JVM栈内存，与线程同时创建。

new Thread()方法，内存分配过程应该是，先是new方法，通过堆内存进行分配，然后Thread创建成功后，会有其对应的栈内存。（[Java specification ](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-2.html#jvms-2.5.2)）

**结果：**
如果分配的堆内存太大，会导致栈内存小，则新建线程无法分配栈内存。

<font color = 'grey'>从技术上讲，Java规范允许堆栈内存存储在堆上。但至少JRockit JVM使用了不同的内存部分。[参考网站](https://stackoverflow.com/questions/36898701/how-does-java-jvm-allocate-stack-for-each-thread)<font color = 'grey'>