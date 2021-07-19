title: Java多线程1-多线程方法
author: Wtli
date: 2021-06-30 16:39:47
tags:
---
java多线程。

- 线程的创建
- Thread类和Object类  
	- start()和run()
	- sleep()和wait()



<!-- more -->

最近遇到一种场景：例如  
 ```
 List<String> dataList = new ArrayList<String>();  
 ```
数据列表dataList，有一个线程专门往list中写数据，另外如果有数据时新建线程读取数据。


先看张图：详细的记录了Thread的几个状态和触发方法

![upload successful](/images/pasted-78.png)

简单说，Thread有三种状态：Timed Waiting、Waiting、Blocked。

#### Thread类和Object类

在多线程的使用中主要涉及两个类，Thread和Object类。

Thread类中有多种操作线程的方法包括但不限于：start()、stop()、run()、join()。

Object类中也有类似于Thread类中操作线程的方法：notify()和wait()方法。

什么是Thread类，一个Runnable的实现类，有两种启动新线程的方法如下所示：
```
 * There are two ways to create a new thread of execution. One is to
 * declare a class to be a subclass of <code>Thread</code>. This
 * subclass should override the <code>run</code> method of class
 * <code>Thread</code>. An instance of the subclass can then be
 * allocated and started. For example, a thread that computes primes
 * larger than a stated value could be written as follows:
 * <hr><blockquote><pre>
 *     class PrimeThread extends Thread {
 *         long minPrime;
 *         PrimeThread(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote><hr>
 * <p>
 * The following code would then create a thread and start it running:
 * <blockquote><pre>
 *     PrimeThread p = new PrimeThread(143);
 *     p.start();
 * </pre></blockquote>
 * <p>
 * The other way to create a thread is to declare a class that
 * implements the <code>Runnable</code> interface. That class then
 * implements the <code>run</code> method. An instance of the class can
 * then be allocated, passed as an argument when creating
 * <code>Thread</code>, and started. The same example in this other
 * style looks like the following:
 * <hr><blockquote><pre>
 *     class PrimeRun implements Runnable {
 *         long minPrime;
 *         PrimeRun(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote><hr>
 * <p>
 * The following code would then create a thread and start it running:
 * <blockquote><pre>
 *     PrimeRun p = new PrimeRun(143);
 *     new Thread(p).start();
 * </pre></blockquote>
 * <p>
public class Thread implements Runnable {

}
```


什么是Object类，看一下注释：

```
/**
 * Class {@code Object} is the root of the class hierarchy.
 * Every class has {@code Object} as a superclass. All objects,
 * including arrays, implement the methods of this class.
 */
```

理解成一个基础类就行了。

#### start()和run()

在Thread类中有两个方法，start()和run()方法。

![upload successful](/images/pasted-79.png)

虽然run()方法现在不推荐使用，在这里也简单说明一下区别：

start()方法：
```
     * The result is that two threads are running concurrently: the
     * current thread (which returns from the call to the
     * <code>start</code> method) and the other thread (which executes its
     * <code>run</code> method).
```
调用start()方法之后会有两个运行的线程：  
Thread1：调用了Thread2.start()的父线程；
Thread2：被调用了start()，执行了自身的run()方法。

run()方法：
调用了之后还是只有一个线程，除非使用了一个独立的Runnable。
```
    /**
     * If this thread was constructed using a separate
     * <code>Runnable</code> run object, then that
     * <code>Runnable</code> object's <code>run</code> method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of <code>Thread</code> should override this method.
     *
     * @see     #start()
     * @see     #stop()
     * @see     #Thread(ThreadGroup, Runnable, String)
     */
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```



##### sleep()和wait()

sleep()和wait()方法都有sleep(long millis)和wait(long timeout)方法，大致的含义都是延时timeout时间再执行。

首先说一下区别：
1. sleep()是Thread类中，wait()是Object类中的方法。
2. wait()方法是和notify()方法对应的，都是Object类中。Object.wait()可以通过Object.notify()方法唤醒。Thread.sleep()不能通过notify()唤醒。
3. wait()方法是放弃当前线程的monitor，sleep()不会释放，可以简单理解为资源和CPU的释放，wait()被激活后需要进入block，直至重新获取资源，才能够继续运行。


sleep()：
```
The thread does not lose ownership of any monitors.
```

wait()：     
```
	 * The current thread must own this object's monitor. The thread
     * releases ownership of this monitor and waits until another thread
     * notifies threads waiting on this object's monitor to wake up
     * either through a call to the {@code notify} method or the
     * {@code notifyAll} method. The thread then waits until it can
     * re-obtain ownership of the monitor and resumes execution.
```

#### wait()和notify()使用

尝试将一个正在运行的线程，调用wait()方法进入等待，然后调用notify()方法唤醒。

新建线程thread，持续计数：

```
        thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (start) {
                    System.out.println(count++);
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
```

然后用户输入运行：

```

```





