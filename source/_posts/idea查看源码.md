title: idea查看JDK8源码
author: Wtli
tags: []
categories: []
date: 2020-10-28 09:02:00
---
背景：idea运行Java项目，想查看jdk中.class文件的源码。

本文介绍OpenJDK和在idea中配置源码。

<!--more-->

### 配置idea

如果不配置，查看源码会如下图所示：

[![Bljlp6.png](https://s1.ax1x.com/2020/10/28/Bljlp6.png)](https://imgchr.com/i/Bljlp6)

会在顶部显示
```
Decompiled .class file,bytecode version: 52.0(Java 8)
```

因为我们是打开的.class文件，是编译后的文件，所以看不到源码（网上看教程也有说使用工具反编译的）。

设置步骤：
1. 在下方链接中下载自己jdk对应的版本并解压，我是Java8版本。
[下载链接](https://www.injdk.cn/)
2. 打开idea  -\-\>  菜单 \-\-\>  **File** \-\-\> **Project Structure** -\-\> 选择**SDKs**目录 -\-\> **Sourcepath** -\-\>  将下载的openJDK中src.zip添加到目录中。

![jdk24b4d.png](http://www.s3tu.com/images/2020/10/27/jdk24b4d.png)

![Blzt74.png](https://s1.ax1x.com/2020/10/28/Blzt74.png)

添加完成之后就能够显示源代码，.class文件也变成.java文件：

[![B1pNWR.png](https://s1.ax1x.com/2020/10/28/B1pNWR.png)](https://imgchr.com/i/B1pNWR)



### OpenJDK

除了学习之外，不推荐使用openJDK。

历史上的原因是，OpenJDK是官方JDK的开放原始码版本，以GPL协议的形式放出。在JDK7的时候，openJDK已经成为jdk7的主干开 发，Oracle JDK 7是在open JDK 7的基础上发布的，其大部分原始码都相同，只有少部分原始码被替换掉。使用JRL(JavaResearch License，Java研究授权协议)发布。

#### 授权协议的不同

openjdk采用GPL V2协议放出，而JDK则采用JRL放出。两者协议虽然都是开放源代码的，但是在使用上的不同在于GPL V2允许在商业上使用，而JRL只允许个人研究使用。

#### OpenJDK不包含Deployment（部署）功能

部署的功能包括：Browser Plugin、Java Web Start、以及Java控制面板，这些功能在Openjdk中是找不到的。

#### OpenJDK源代码不完整

这个很容易想到，在采用GPL协议的Openjdk中，sun jdk的一部分源代码因为产权的问题无法开放openjdk使用，其中最主要的部份就是JMX中的可选元件SNMP部份的代码。因此这些不能开放的源代码 将它作成plug，以供OpenJDK编译时使用，你也可以选择不要使用plug。而Icedtea则为这些不完整的部分开发了相同功能的源代码 (OpenJDK6)，促使OpenJDK更加完整。

#### 部分源代码用开源代码替换

由于产权的问题，很多产权不是SUN的源代码被替换成一些功能相同的开源代码，比如说字体栅格化引擎，使用Free Type代替。

#### OpenJDK只包含最精简的JDK

OpenJDK不包含其他的软件包，比如Rhino Java DB JAXP……，并且可以分离的软件包也都是尽量的分离，但是这大多数都是自由软件，你可以自己下载加入。

#### 不能使用Java商标

这个很容易理解，在安装openjdk的机器上，输入“java -version”显示的是openjdk，但是如果是使用Icedtea补丁的openjdk，显示的是java。


















