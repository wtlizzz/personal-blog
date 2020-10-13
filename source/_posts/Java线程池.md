title: Java线程池
author: Wtli
tags:
  - Java
categories:
  - 后端
date: 2020-09-25 08:48:00
---
Java中线程池对于多线程的处理，能够提高很大的效率，最近在看Java NIO、Netty在此重新学习一下Java中的线程池的使用。
<!--more-->

java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类。  
简单先看一下使用,采用了上一篇文章中的Netty中的伪异步线程池代码：
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

ThreadPoolExecutor类中提供了四个构造方法：

```
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```
构造方法的参数分别是：corePoolSize、maximumPoolSize、keepAliveTime、unit、workQueue、threadFactory、RejectedExecutionHandler

corePoolSize:核心线程数量，除非超时数量为0，其他情况是最小存活线程的数量。
```
    /**
     * Core pool size is the minimum number of workers to keep alive
     * (and not allow to time out etc) unless allowCoreThreadTimeOut
     * is set, in which case the minimum is zero.
     */
    private volatile int corePoolSize;
```

maximumPoolSize：线程池最大线程数，表示在线程池中最多能创建多少个线程；
```
    /**
     * Maximum pool size. Note that the actual maximum is internally
     * bounded by CAPACITY.
     */
    private volatile int maximumPoolSize;
```
workQueue:一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响。
```
    /**
     * The queue used for holding tasks and handing off to worker
     * threads.  We do not require that workQueue.poll() returning
     * null necessarily means that workQueue.isEmpty(), so rely
     * solely on isEmpty to see if the queue is empty (which we must
     * do for example when deciding whether to transition from
     * SHUTDOWN to TIDYING).  This accommodates special-purpose
     * queues such as DelayQueues for which poll() is allowed to
     * return null even if it may later return non-null when delays
     * expire.
     */
    private final BlockingQueue<Runnable> workQueue;
```

threadFactory：线程工厂，主要用来创建线程；


