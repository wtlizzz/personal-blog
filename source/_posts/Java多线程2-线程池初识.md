title: Java多线程2-线程池初识
author: Wtli
date: 2021-07-06 14:57:47
tags:
---
初识Java线程池（JDK1.8）。

<!-- more -->

Alibaba开发手册：  
1. <font color='red'> 【强制】 </font>线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。  
说明：线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。 如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。



### Executors

使用Executors创建线程池，Executors类中共有4种线程池的创建方法。

Executors
- newCachedThreadPool
- newFixedThreadPool
- newScheduledThreadPool
- newSingleThreadExecutor
- newWorkStealingPool

使用Executors创建方式：

```
        /**
         * 1、单线程线程池
         * 2、缓存线程池
         * 3、固定大小线程池
         * 4、定时线程池
         */
        ExecutorService executorService1 = Executors.newSingleThreadExecutor();
        ExecutorService executorService2 = Executors.newCachedThreadPool();
        ExecutorService executorService3 = Executors.newFixedThreadPool(4);
        ExecutorService executorService4 = Executors.newScheduledThreadPool(4);
        ExecutorService executorService5 = Executors.newWorkStealingPool();
```

### ThreadPoolExecutor

基础类

newSingleThreadExecutor、newFixedThreadPool都是返回的此类。

```
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

参数含义：

corePoolSize：线程池的大小。
```
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
```
maximumPoolSize：线程池最大容量。
```
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
```
keepAliveTime：当线程数大于核心时，这是多余空闲线程的最长时间。

```
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
```

unit：{@code keepAliveTime}参数的时间单位。
```
	* @param unit the time unit for the {@code keepAliveTime} argument
```
workQueue：用于在执行任务之前保存任务的队列。这个队列将只保存由{@code execute}方法提交的{@code Runnable}任务。
```
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
```

threadFactory：执行器创建新线程时要使用的工厂。
```
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
```

handler：由于达到线程边界和队列容量而阻止执行时要使用的处理程序。
```
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
```

**实例：**

```
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```

*newSingleThreadExecutor*

```
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

*newFixedThreadPool*

```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```




### newSingleThreadExecutor

说明：

```
    /**
     * Creates an Executor that uses a single worker thread operating
     * off an unbounded queue. (Note however that if this single
     * thread terminates due to a failure during execution prior to
     * shutdown, a new one will take its place if needed to execute
     * subsequent tasks.)  Tasks are guaranteed to execute
     * sequentially, and no more than one task will be active at any
     * given time. Unlike the otherwise equivalent
     * {@code newFixedThreadPool(1)} the returned executor is
     * guaranteed not to be reconfigurable to use additional threads.
     **/
```

说明：

创建一个执行器，该执行器使用一个工作线程在无边界队列中进行操作(但是请注意，如果此单个线程在关机之前的执行过程中由于故障而终止，则在需要执行后续任务时，将替换一个新线程。）任务保证按顺序执行，并且在任何给定时间都不会有多个任务处于活动状态。与其他等价的{@code newFixedThreadPool（1）}不同，返回的执行器保证不可重新配置以使用其他线程。


解释：

使用new LinkedBlockingQueue\<Runnable>()，一种不限制大小的queue来创建。

```
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

**运行过程：**


只运行一个Thread来执行任务，一个Thread结束时，会建立新的Thread来执行下一个任务，保证了任务执行的顺序。

这是与newFixedThreadPool(1)不同的地方，newFixedThreadPool(1)执行任务时，有可能被其他操作进行重新配置，配置成多线程。

### newCachedThreadPool

说明：

```
    /**
     * Creates a thread pool that creates new threads as needed, but
     * will reuse previously constructed threads when they are
     * available.  These pools will typically improve the performance
     * of programs that execute many short-lived asynchronous tasks.
     * Calls to {@code execute} will reuse previously constructed
     * threads if available. If no existing thread is available, a new
     * thread will be created and added to the pool. Threads that have
     * not been used for sixty seconds are terminated and removed from
     * the cache. Thus, a pool that remains idle for long enough will
     * not consume any resources. Note that pools with similar
     * properties but different details (for example, timeout parameters)
     * may be created using {@link ThreadPoolExecutor} constructors.
     *
     * @return the newly created thread pool
     */
```
翻译：

创建一个线程池，该线程池根据需要创建新线程，但在以前构造的线程可用时将重用它们。这些池通常会提高执行许多短期异步任务的程序的性能。对{@code execute}的调用将重用先前构造的线程（如果可用）。如果没有可用的现有线程，将创建一个新线程并将其添加到池中。六十秒未使用的线程将被终止并从缓存中删除。因此，空闲时间足够长的池不会消耗任何资源。请注意，可以使用{@link ThreadPoolExecutor}构造函数创建属性相似但细节不同的池（例如，超时参数）。


初始化：

```
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```


### newFixedThreadPool

说明：

```
    /**
     * Creates a thread pool that reuses a fixed number of threads
     * operating off a shared unbounded queue.  At any point, at most
     * {@code nThreads} threads will be active processing tasks.
     * If additional tasks are submitted when all threads are active,
     * they will wait in the queue until a thread is available.
     * If any thread terminates due to a failure during execution
     * prior to shutdown, a new one will take its place if needed to
     * execute subsequent tasks.  The threads in the pool will exist
     * until it is explicitly {@link ExecutorService#shutdown shutdown}.
     *
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * @throws IllegalArgumentException if {@code nThreads <= 0}
     */
```

翻译：

创建一个线程池，该线程池重用在共享无边界队列上运行的固定数量的线程。在任何时候，至多{@code nThreads}个线程都是活动的处理任务。如果在所有线程都处于活动状态时提交其他任务，它们将在队列中等待，直到有线程可用。如果任何线程在关机前的执行过程中由于失败而终止，那么如果需要执行后续任务，将有一个新线程代替它。池中的线程将一直存在，直到它显式地{@link ExecutorService#shutdown}。

初始化：

```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

**运行过程：**

和newSingleThreadExecutor一样，使用无容量限制的LinkedBlockingQueue进行初始化。


### newScheduledThreadPool

说明：

```
    /**
     * Creates a thread pool that can schedule commands to run after a
     * given delay, or to execute periodically.
     * @param corePoolSize the number of threads to keep in the pool,
     * even if they are idle
     * @return a newly created scheduled thread pool
     * @throws IllegalArgumentException if {@code corePoolSize < 0}
     */
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```

翻译：

创建一个线程池，该线程池可以安排命令在给定延迟后运行，或定期执行。

运行过程：

![upload successful](/images/pasted-80.png)

可以看出就是使用了不同的queue来实现，在这里使用的是DelayedWorkQueue。


### newWorkStealingPool

说明：

```
    /**
     * Creates a work-stealing thread pool using all
     * {@link Runtime#availableProcessors available processors}
     * as its target parallelism level.
     * @return the newly created thread pool
     * @see #newWorkStealingPool(int)
     * @since 1.8
     */
    public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```
翻译：

创建一个线程池，该线程池维护足够的线程以支持给定的并行级别，并且可以使用多个队列来减少争用。并行级别对应于主动参与或可用于参与任务处理的最大线程数。线程的实际数量可能会动态地增长和收缩。偷工池不能保证提交任务的执行顺序。