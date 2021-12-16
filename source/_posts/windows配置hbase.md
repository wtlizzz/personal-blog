title: windows配置hbase
author: Wtli
tags: []
categories: []
date: 2021-05-17 14:29:00
---
本文介绍如何在windows系统中配置hbase。

<!--more-->

hbase是针对unix系统开发的分布式、面向列的存储、参照Google的BigTable实现的，构建在Hadoop之上，用于MapReduce和分布式文件系统。

网上教程也很多，现在记录一下我配置成功的方式，还有遇到的问题及解决方式。

HBase是一个开源的非关系型分布式数据库（NoSQL），它参考了谷歌的BigTable建模，实现的编程语言为 Java。它是Apache软件基金会的Hadoop项目的一部分，运行于HDFS文件系统之上，为 Hadoop 提供类似于BigTable 规模的服务。因此，它可以对稀疏文件提供极高的容错率。

HBase在列上实现了BigTable论文提到的压缩算法、内存操作和布隆过滤器。HBase的表能够作为MapReduce任务的输入和输出，可以通过Java API （页面存档备份，存于互联网档案馆）来访问数据，也可以通过REST、Avro或者Thrift的API来访问。

虽然最近性能有了显著的提升，HBase 还不能直接取代SQL数据库。如今，它已经应用于多个数据驱动型网站，包括 Facebook的消息平台。
<font color='gray'>From 维基百科</font>

#### 前期准备

我们在windows的机器上设置HBase，需要：

Java JDK 1.8 - Download from this link JAVA JDK。 Install and set JAVA_HOME in environment variable.
HBase - Apache HBase can be downloaded from this link。

#### Step 1:

解压HBase压缩包。

Unzip the downloaded Hbase and place it in some common path, say C:/Document/hbase-2.2.5

#### Step 2:

创建两个文件夹，分别是data和zookeeper文件夹。

Create a folders as shown below inside root folder for HBase data and zookeeper
```
-> C:/Document/hbase-2.2.5/hbase
-> C:/Document/hbase-2.2.5/zookeeper
```
#### Step 3:

修改配置文件/bin/hbase.cmd。

Open C:/Document/hbase-2.2.5/bin/hbase.cmd in notepad++. Search for below given lines and remove %HEAP_SETTINGS% from that line as dictated in the video embedded with this blog

set java_arguments= ~~%HEAP_SETTINGS%~~ %HBASE_OPTS% -classpath “%CLASSPATH%” %CLASS% %hbase-command-arguments%

#### Step 4:

修改conf/hbase-env.cmd文件,在文件中添加一段变量。

Open C:/Document/hbase-2.2.5/conf/hbase-env.cmd n notepad++. Add the below lines to the file after the comment session.
```
set JAVA_HOME=%JAVA_HOME%
set HBASE_CLASSPATH=%HBASE_HOME%\lib\client-facing-thirdparty\*
set HBASE_HEAPSIZE=8000
set HBASE_OPTS="-XX:+UseConcMarkSweepGC" "-Djava.net.preferIPv4Stack=true"
set SERVER_GC_OPTS="-verbose:gc" "-XX:+PrintGCDetails" "-XX:+PrintGCDateStamps" %HBASE_GC_OPTS%
set HBASE_USE_GC_LOGFILE=true

set HBASE_JMX_BASE="-Dcom.sun.management.jmxremote.ssl=false" "-Dcom.sun.management.jmxremote.authenticate=false"
::优化配置，暂时不配置
::set HBASE_MASTER_OPTS=%HBASE_JMX_BASE% "-Dcom.sun.management.jmxremote.port=10101"
::set HBASE_REGIONSERVER_OPTS=%HBASE_JMX_BASE% "-Dcom.sun.management.jmxremote.port=10102"
::set HBASE_THRIFT_OPTS=%HBASE_JMX_BASE% "-Dcom.sun.management.jmxremote.port=10103"
::set HBASE_ZOOKEEPER_OPTS=%HBASE_JMX_BASE% -Dcom.sun.management.jmxremote.port=10104"
set HBASE_REGIONSERVERS=%HBASE_HOME%\conf\regionservers
set HBASE_LOG_DIR=%HBASE_HOME%\logs
set HBASE_IDENT_STRING=%USERNAME%
set HBASE_MANAGES_ZK=true
```
#### Step 5:

修改conf/hbase-site.xml配置文件。

Open C:/Document/hbase-2.2.5/conf/hbase-site.xml notepad++. Add the below lines inside \<configuration> tag.
```
<property>
    <name>hbase.rootdir</name>
    <value>file:///C:/Documents/hbase-2.2.5/hbase</value>
 </property>
 <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/C:/Documents/hbase-2.2.5/zookeeper</value>
 </property>
 <property>
     <name> hbase.zookeeper.quorum</name>
    <value>localhost</value>
 </property>
 ```
#### Step 6:

在环境变量中配置HBASE_HOME

Setup the Environment variable for HBASE_HOME and add bin to the path variable as shown in the below image.
```
HBASE_HOME

C:/Document/hbase-2.2.5
```
#### Step 7:

运行，cmd直接进入到bin文件夹下运行.\start-hbase.cmd文件。

遇到的问题
我这里报错找不到HADOOP_HOME环境变量。下载winutils.exe，在github中可以选择下载整个项目，然后解压。

设置HADOOP_HOME环境变量到winutils.exe文件夹。

网上也有教程说需要下载hadoop，先安装hadoop，没有尝试，也可以使用hadoop中的winutils。

附：[参考网站](https://www.learntospark.com/2020/08/setup-hbase-in-windows.html)