title: Java多线程3-线程池使用
author: Wtli
date: 2021-07-12 15:59:21
tags:
---
Java线程池的使用（JDK1.8）。

- ThreadPoolExecutor
- Executors

<!-- more -->

Alibaba开发手册：  

1. <font color='red'> 【强制】 </font>线程池不允许使用 Executors 去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。  
说明：Executors 返回的线程池对象的弊端如下：  
1）FixedThreadPool和SingleThreadPool允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。   
2）CachedThreadPool允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。

#### 使用ThreadPoolExecutor创建线程池

![upload successful](/images/pasted-81.png)

ThreadPoolExecutor有四种创建方式，参数分别是如图所示。

|序号|	名称	|类型|	含义|
|---|---|---|---|
|1	|corePoolSize	|int|	核心线程池大小|
|2	|maximumPoolSize	|int	|最大线程池大小|
|3	|keepAliveTime|	long	|线程最大空闲时间|
|4	|unit	|TimeUnit	|时间单位|
|5	|workQueue	|BlockingQueue\<Runnable>|	线程等待队列|
|6	|threadFactory	|ThreadFactory	|线程创建工厂|
|7	|handler	|RejectedExecutionHandler|	拒绝策略|

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

  
##### corePoolSize和maximumPoolSize区别

官方注释

corePoolSize：除非设置了allowCoreThreadTimeOut，corePoolSize数量的thread会一直保留在线程池中。

maximumPoolSize：线程池中最大的线程数量。

这里有个简单的[例子](http://www.bigsoft.co.uk/blog/2009/11/27/rules-of-a-threadpoolexecutor-pool-size)：
```
Take this example. Starting thread pool size is 1, core pool size is 5, max pool size is 10 and the queue is 100.

Here are Sun's rules for thread creation in simple terms:

    1.If the number of threads is less than the corePoolSize, create a new Thread to run a new task.
    2.If the number of threads is equal (or greater than) the corePoolSize, put the task into the queue.
    3.If the queue is full, and the number of threads is less than the maxPoolSize, create a new thread to run tasks in.
    4.If the queue is full, and the number of threads is greater than or equal to maxPoolSize, reject the task.
```
解释：

设置corePoolSize为5，maximumPoolSize为10，queue为100.

当

1. 线程数 < corePoolSize，创建新的线程执行task。
2. corePoolSize <= 线程数 <= maximumPoolSize，queue没满，task放进queue。
3. corePoolSize <= 线程数 <= maximumPoolSize，queue满了，创建新线程执行task。
3. 线程数 >= maximumPoolSize，queue满了，reject task。

##### 测试执行顺序

测试：场景1.2 -- queue不满

代码：

Task：延时操作，加了Thread.sleep(3000)
```
    static class MyTask implements Runnable {
        private String name;

        public MyTask(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            try {
                System.out.println(this.getName() + " *** " + this.toString() + " is running!");
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        public String getName() {
            return name;
        }

        @Override
        public String toString() {
            return "MyTask [name=" + name + "]";
        }
    }
```


ThreadPoolExecutor：
```
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 4, 10, 
        	TimeUnit.SECONDS,
        	new ArrayBlockingQueue<>(10), 
        	new NameTreadFactory(), //自定义输出名称Factory
        	new MyIgnorePolicy()); //自定义打印reject
```

结果：

根据设置的corePoolSize大小，只创建了两个线程：my-thread-1、my-thread-2.

```
my-thread-1 has been created
my-thread-2 has been created
1 *** MyTask [name=1] is running!
2 *** MyTask [name=2] is running!
3 *** MyTask [name=3] is running!
4 *** MyTask [name=4] is running!
5 *** MyTask [name=5] is running!
6 *** MyTask [name=6] is running!
7 *** MyTask [name=7] is running!
8 *** MyTask [name=8] is running!
9 *** MyTask [name=9] is running!
10 *** MyTask [name=10] is running!
```
测试：场景3.4 -- queue满了

将ArrayBlockingQueue大小改成4.

代码：
ThreadPoolExecutor：
```
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 4, 10, 
        	TimeUnit.SECONDS,
        	new ArrayBlockingQueue<>(4), 
        	new NameTreadFactory(), //自定义输出名称Factory
        	new MyIgnorePolicy()); //自定义打印reject
```

结果：

1. 根据设置的corePoolSize大小，第一个任务进入时，创建线程my-thread-1，第二个任务进入时，创建线程my-thread-2。
2. 第3，4，5，6任务进入时，将任务压入队列中，此时队列已满。
3. 当前线程数currentThread(2) < maximumPoolSize(4)。
4. 任务7，8进入，还能够再创建两个线程，此时创建my-thread-3和my-thread-4来执行任务7，8.
5. 任务9，10进入，此时线程数已经currentThread(4) = maximumPoolSize(4)，则直接拒绝任务9，10.


```
my-thread-1 has been created
my-thread-2 has been created
1 *** MyTask [name=1] is running!
my-thread-3 has been created
2 *** MyTask [name=2] is running!
my-thread-4 has been created
7 *** MyTask [name=7] is running!
8 *** MyTask [name=8] is running!
MyTask [name=9] rejected
MyTask [name=10] rejected
3 *** MyTask [name=3] is running!
4 *** MyTask [name=4] is running!
6 *** MyTask [name=6] is running!
5 *** MyTask [name=5] is running!
```

说明：

任务执行顺序，先满足corePoolSize数量的任务，然后剩下的任务都放到等待队列中，队列满了之后创建新的线程（直到任务数量到maximumPoolSize）来执行新的任务，而不是先执行队列中未执行的任务。

#### 其他方法

prestartAllCoreThreads: 提前启动corePoolSize数量的线程。该方法运行则创建线程，等待Task进入。

说明：
```
   /**
     * Starts all core threads, causing them to idly wait for work. This
     * overrides the default policy of starting core threads only when
     * new tasks are executed.
     *
     * @return the number of threads started
     */
    public int prestartAllCoreThreads() {
        int n = 0;
        while (addWorker(null, true))
            ++n;
        return n;
    }
```

allowsCoreThreadTimeOut: 

具体没有碰到使用场景。。先在此处光进行翻译。。

```
设置控制核心线程是否可以超时并在保持活动状态时间内没有任务到达时终止的策略，如果需要，在新任务到达时替换。如果为false，则不会由于缺少传入任务而终止核心线程。如果为true，则应用于非核心线程的保持活动状态策略也适用于核心线程。为了避免连续的线程替换，在设置{@code true}时，保持活动时间必须大于零。通常，应该在主动使用池之前调用此方法。
```

#### 等待队列任务执行顺序

首先：多次实验ArrayBlockingQueue，得到运行等待队列中几次结果分别是：

3，5，4，6  
3，6，4，5  
4，3，5，6

由此可以说明等待队列中的Task并不能保证顺序运行。

既然使用ArrayBlockingQueue不能保证任务执行顺序，那如果是因为任务的插入顺序和取出出现的异常的话，那使用链表队列LinkedBlockingQueue能否避免这种问题。

比较ArrayBlockingQueue和LinkedBlockingQueue

结论：ArrayBlockingQueue和LinkedBlockingQueue结果相同，也没有固定的执行顺序。

使用LinkedBlockingQueue结果：
```
my-thread-1 has been created
my-thread-2 has been created
1 *** MyTask [name=1] is running!
my-thread-3 has been created
2 *** MyTask [name=2] is running!
my-thread-4 has been created
7 *** MyTask [name=7] is running!
8 *** MyTask [name=8] is running!
MyTask [name=9] rejected
MyTask [name=10] rejected
3 *** MyTask [name=3] is running!
6 *** MyTask [name=6] is running!
4 *** MyTask [name=4] is running!
5 *** MyTask [name=5] is running!
```

运行数次结果：  
3，6，5，4  
3，6，5，4  
3，6，4，5


##### 所调用的阻塞队列方法

重写等待队列，查看是入队的顺序有问题还是出队的顺序有问题，查看是否在队列中的顺序发生变化。


#### 使用Executors创建线程池

虽然不推荐使用Executors，但保不准什么时候优化好了，没有bug了，用起来就方便了，现在也简单用一下。

创建ExecutorService：
```
        ExecutorService executorService1 = Executors.newSingleThreadExecutor();
        ExecutorService executorService2 = Executors.newCachedThreadPool();
        ExecutorService executorService3 = Executors.newFixedThreadPool(4);
        ExecutorService executorService4 = Executors.newScheduledThreadPool(4);
        ExecutorService executorService5 = Executors.newWorkStealingPool();
```

运行Task：
```
MyTask task = new MyTask(String.valueOf(i));
            executorService1.execute(task);
```

<font color='grey'>用起来貌似更方便，不用配置一些参数。</font>

结果：
```
1 *** MyTask [name=1] is running!
2 *** MyTask [name=2] is running!
3 *** MyTask [name=3] is running!
4 *** MyTask [name=4] is running!
5 *** MyTask [name=5] is running!
6 *** MyTask [name=6] is running!
7 *** MyTask [name=7] is running!
8 *** MyTask [name=8] is running!
9 *** MyTask [name=9] is running!
10 *** MyTask [name=10] is running!
```
