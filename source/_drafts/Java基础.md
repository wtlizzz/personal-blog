title: Java基础
author: Wtli
tags:
  - Java
categories:
  - Java
date: 2020-11-20 09:08:00
---
Java常用的知识：
- 单例模式
- 定时器
- 多线程
<!--more-->

### 单例模式

#### Holder模式

```
public class Singleton {
    /**
     * 类级的内部类，也就是静态的成员式内部类，该内部类的实例与外部类的实例
     * 没有绑定关系，而且只有被调用到才会装载，从而实现了延迟加载
     */
    private static class SingletonHolder{
        /**
         * 静态初始化器，由JVM来保证线程安全
         */
        private static Singleton instance = new Singleton();
    }
    /**
     * 私有化构造方法
     */
    private Singleton(){
    }
    public static  Singleton getInstance(){
        return SingletonHolder.instance;
    }
}
```
 优点：将懒加载和线程安全完美结合的一种方式（无锁）。（推荐）


#### 饿汉模式
```
public class Singleton {

//4：定义一个静态变量来存储创建好的类实例

//直接在这里创建类实例，只会创建一次

    private static Singleton instance = new Singleton();

//1：私有化构造方法，好在内部控制创建实例的数目

    private Singleton(){

    }

//2：定义一个方法来为客户端提供类实例

//3：这个方法需要定义成类方法，也就是要加static

//这个方法里面就不需要控制代码了

    public static Singleton getInstance(){

//5：直接使用已经创建好的实例

        return instance;

    }

}
```

 优点：饿汉模式天生是线程安全的，使用时没有延迟。

 缺点：启动时即创建实例，启动慢，有可能造成资源浪费。

### 定时器

使用Timer的schedule，schedule有3个参数：

schedule(TimerTask task, long delay, long period)
第一个为定时任务，根据业务需要重写TimerTask的run方法即可；

第二个为延时启动，单位毫秒；

第三个位多久运行一次，单位毫秒；

```
new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                try {
                    //do Something
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        },0,5L * 60 * 1000);
```

### 多线程

#### 实现Runnable接口

```
package ljz;
class MyThread implements Runnable{ // 实现Runnable接口，作为线程的实现类
    private String name ;       // 表示线程的名称
    public MyThread(String name){
        this.name = name ;      // 通过构造方法配置name属性
    }
    public void run(){  // 覆写run()方法，作为线程 的操作主体
        for(int i=0;i<10;i++){
            System.out.println(name + "运行，i = " + i) ;
        }
    }
};
public class RunnableDemo01{
    public static void main(String args[]){
        MyThread mt1 = new MyThread("线程A ") ;    // 实例化对象
        MyThread mt2 = new MyThread("线程B ") ;    // 实例化对象
        Thread t1 = new Thread(mt1) ;       // 实例化Thread类对象
        Thread t2 = new Thread(mt2) ;       // 实例化Thread类对象
        t1.start() ;    // 启动多线程
        t2.start() ;    // 启动多线程
    }
};
```

#### 继承Thread类

```
class MyThread extends Thread{  // 继承Thread类，作为线程的实现类
    private String name ;       // 表示线程的名称
    public MyThread(String name){
        this.name = name ;      // 通过构造方法配置name属性
    }
    public void run(){  // 覆写run()方法，作为线程 的操作主体
        for(int i=0;i<10;i++){
            System.out.println(name + "运行，i = " + i) ;
        }
    }
};
public class ThreadDemo02{
    public static void main(String args[]){
        MyThread mt1 = new MyThread("线程A ") ;    // 实例化对象
        MyThread mt2 = new MyThread("线程B ") ;    // 实例化对象
        mt1.start() ;   // 调用线程主体
        mt2.start() ;   // 调用线程主体
    }
};
```
















