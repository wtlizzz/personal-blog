title: Java基础
author: Wtli
tags:
  - Java
categories:
  - 后端
  - ''
date: 2020-11-20 09:08:00
---
Java常用的知识：

- 分层领域模型规约（PO、DAO、DTO）
- 云计算中服务类别（IaaS、Paas、SaaS、DaaS）
- 单例模式
- 定时器
- 多线程

<!--more-->

#### 分层领域模型规约

**ALibaba开发手册中记录应用分层中的分层领域模型规约，共分为6种对象，分别为：DO、DTO、BO、AO、VO、Query**

- DO（Data Object）：与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
- DTO（Data Transfer Object）：数据传输对象，Service 或 Manager 向外传输的对象。
- BO（Business Object）：业务对象。由 Service 层输出的封装业务逻辑的对象。
- AO（Application Object）：应用对象。在 Web 层与 Service 层之间抽象的复用对象模型，极为贴近展示层，复用度不高。 
- VO（View Object）：显示层对象，通常是 Web 向模板渲染引擎层传输的对象。 
- Query：数据查询对象，各层接收上层的查询请求。注意超过 2 个参数的查询封装，禁止使用 Map 类来传输。


另外附上应用分层结构：

<img style="margin: auto;" src="/images/pasted-69.png" width="500" height="500" />

- 开放接口层：可直接封装 Service 方法暴露成 RPC 接口；通过 Web 封装成 http 接口；进行 网关安全控制、流量控制等。 
- 终端显示层：各个端的模板渲染并执行显示的层。当前主要是 velocity 渲染，JS 渲染， JSP 渲染，移动端展示等。 
- Web 层：主要是对访问控制进行转发，各类基本参数校验，或者不复用的业务简单处理等。 
- Service 层：相对具体的业务逻辑服务层。 
- Manager 层：通用业务处理层，它有如下特征：  
1） 对第三方平台封装的层，预处理返回结果及转化异常信息；  
2） 对 Service 层通用能力的下沉，如缓存方案、中间件通用处理；  
3） 与 DAO 层交互，对多个 DAO 的组合复用。 
- DAO 层：数据访问层，与底层 MySQL、Oracle、Hbase 等进行数据交互。 
- 外部接口或第三方平台：包括其它部门 RPC 开放接口，基础平台，其它公司的 HTTP 接口。

POJO是个统称

- POJO（Plain Ordinary Java Object）: 在本手册中，POJO 专指只有 setter / getter / toString 的简单类，包括 DO/DTO/BO/VO 等。

**总结：**在开发过程中，主要是区分DO和DTO对象，一个是DAO中使用绑定数据库的对象，一个是在数据传输过程中使用的对象。DO无法用于命名文件，所以尽量使用DAO（Data Access Object）来标示数据持久化对象。

#### 云计算中服务类别

最近和阿里云做交流，发现了他们的配图，如下：

<img style="margin: auto;" src="/images/pasted-71.png"/>

他们是底层向上顺序是   

**Iaas &nbsp;&nbsp; ===> &nbsp;&nbsp; Daas &nbsp;&nbsp; ===> &nbsp;&nbsp; Paas**

我们不管他上下顺序对不对，先来了解一下Iaas、Paas、Daas，还有一个是Saas在这个图中没有显示。  
微软的 Azure 云服务有一张图，解释这三种模式的差异。
<img style="margin: auto;" src="/images/pasted-72.png"/>

在上图结合阿里云介绍图，

**Iaas（Infrastructure-as-a-service）**包括网络（Networking）、存储（Storage）、虚拟化（Virtualization）、服务（Servers），另外好一点的云服务提供商（阿里云）会提供计算服务。

**Paas（Platform-as-a-service）**包括操作系统（OS）、中间件（Middleware）、进行时（Runtime）。阿里云中着重介绍了目前云服务器中集成的主流中间件。

**Daas（Data-as-a-service）**数据即服务，主要是提供了数据库（Data）的支持。

就目前如果根据上图来看，Daas应该是在Paas上层的。


下面是详细的概念：
- IaaS: Infrastructure-as-a-Service(基础设施即服务)有了IaaS，你可以将硬件外包到别的地方去。IaaS公司会提供场外服务器，存储和网络硬件，你可以租用。节省了维护成本和办公场地，公司可以在任何时候利用这些硬件来运行其应用。一些大的IaaS公司包括Amazon, Microsoft, VMWare, Rackspace和Red Hat.不过这些公司又都有自己的专长，比如Amazon和微软给你提供的不只是IaaS，他们还会将其计算能力出租给你来host你的网站。
- PaaS: Platform-as-a-Service(平台即服务)第二层就是所谓的PaaS，某些时候也叫做中间件。你公司所有的开发都可以在这一层进行，节省了时间和资源。PaaS公司在网上提供各种开发和分发应用的解决方案，比如虚拟服务器和操作系统。这节省了你在硬件上的费用，也让分散的工作室之间的合作变得更加容易。网页应用管理，应用设计，应用虚拟主机，存储，安全以及应用开发协作工具等。一些大的PaaS提供者有Google App Engine,Microsoft Azure，Force.com,Heroku，Engine Yard。最近兴起的公司有AppFog,Mendix和Standing Cloud.
- SaaS: Software-as-a-Service(软件即服务)第三层也就是所谓SaaS。这一层是和你的生活每天接触的一层，大多是通过网页浏览器来接入。任何一个远程服务器上的应用都可以通过网络来运行，就是SaaS了。你消费的服务完全是从网页如Netflix,MOG,Google Apps,Box.net,Dropbox或者苹果的iCloud那里进入这些分类。尽管这些网页服务是用作商务和娱乐或者两者都有，但这也算是云技术的一部分。一些用作商务的SaaS应用包括Citrix的Go To Meeting，Cisco的WebEx，Salesforce的CRM，ADP，Workday和SuccessFactors。


#### 线程同步



#### 单例模式

##### Holder模式

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


##### 饿汉模式
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

#### 定时器

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

#### 多线程

##### 实现Runnable接口

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

##### 继承Thread类

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