title: Hadoop单机搭建
author: Wtli
tags:
  - Hadoop
categories:
  - 数据
date: 2020-10-10 09:31:00
---
Apache Hadoop是一个框架，用于在由商用硬件构建的大型集群上运行应用程序。Hadoop框架透明地为可靠性和数据移动提供了应用程序。Hadoop实现了一个名为Map/Reduce的计算范式，其中应用程序被划分为许多小的工作片段，每个片段都可以在集群中的任何节点上执行或重新执行。此外，它还提供了一个分布式文件系统(HDFS)，用于在计算节点上存储数据，从而在整个集群中提供了非常高的聚合带宽。MapReduce和Hadoop分布式文件系统的设计都是为了让框架自动处理节点故障。

<!--more-->

### 所需软件
Linux和Windows所需软件包括:

JavaTM1.5.x，必须安装，建议选择Sun公司发行的Java版本。
ssh 必须安装并且保证 sshd一直运行，以便用Hadoop 脚本管理远端Hadoop守护进程。
Windows下的附加软件需求

Cygwin - 提供上述软件之外的shell支持。

### 安装软件
如果你的集群尚未安装所需软件，你得首先安装它们。

以Ubuntu Linux为例:
```
$ sudo apt-get install ssh
$ sudo apt-get install rsync
```
在Windows平台上，如果安装cygwin时未安装全部所需软件，则需启动cyqwin安装管理器安装如下软件包：

- openssh - Net 类

### 下载

为了获取Hadoop的发行版，从Apache的某个镜像服务器上下载最近的[稳定发行版](https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.10.1/hadoop-2.10.1.tar.gz)。

### 准备

解压所下载的Hadoop发行版。编辑 etc/hadoop/hadoop-env.sh文件，至少需要将JAVA_HOME设置为Java安装根路径。

在bin父目录中尝试如下命令：
$ bin/hadoop
将会显示hadoop 脚本的使用文档。

现在你可以用以下三种支持的模式中的一种启动Hadoop集群：

- 单机模式
- 伪分布式模式
- 完全分布式模式


### 单机模式的操作方法

默认情况下，Hadoop被配置成以非分布式模式运行的一个独立Java进程。这对调试非常有帮助。

下面的实例将已解压的 conf 目录拷贝作为输入，查找并显示匹配给定正则表达式的条目。输出写入到指定的output目录。

```
  $ mkdir input
  $ cp etc/hadoop/*.xml input
  $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar grep input output 'dfs[a-z.]+'
  $ cat output/*
```

### 伪分布式模式的操作方法

Hadoop可以在单节点上以所谓的伪分布式模式运行，此时每一个Hadoop守护进程都作为一个独立的Java进程运行。


#### 配置

使用如下的 etc/hadoop/core-site.xml:
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

etc/hadoop/hdfs-site.xml:
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

#### 免密码ssh设置
现在确认能否不输入口令就用ssh登录localhost:

```
 $ ssh localhost
```

如果不输入口令就无法用ssh登陆localhost，执行下面的命令：
```
  $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  $ chmod 0600 ~/.ssh/authorized_keys
```

如果是mac报错Connection refused：
```
$ sudo systemsetup -f -setremotelogin on
```

#### 执行
下面的说明用于在本地运行MapReduce作业。

1. 格式文件系统:

```
  $ bin/hdfs namenode -format
```

2. 启动NameNode守护进程和DataNode守护进程:
```
  $ sbin/start-dfs.sh
```

3.  浏览web界面找到NameNode;默认情况下可在:
> NameNode - http://localhost:50070/

4. 创建执行MapReduce作业所需的HDFS目录:
```
  $ bin/hdfs dfs -mkdir /user
  $ bin/hdfs dfs -mkdir /user/<username>
```

5. 将输入文件复制到分布式文件系统:
```
  $ bin/hdfs dfs -put etc/hadoop input
```

6. 运行提供的一些示例:
```
  $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.1.jar grep input output 'dfs[a-z.]+'
```

7. 检查输出文件:将分布式文件系统的输出文件复制到本地文件系统，检查:
```
  $ bin/hdfs dfs -get output output
  $ cat output/*
```

 或查看分布式文件系统的输出文件:
```
  $ bin/hdfs dfs -cat output/*
```
 
8. 当你完成时，停止守护进程:
```
  $ sbin/stop-dfs.sh
```

#### 在单个节点上使用YARN

通过设置一些参数并另外运行ResourceManager守护进程和NodeManager守护进程，您可以在YARN上以伪分布式模式运行MapReduce作业。

1. 配置参数如下:etc/hadoop/map -site.xml:

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

etc/hadoop/yarn-site.xml:

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

2. 启动ResourceManager守护进程和NodeManager守护进程:

```
  $ sbin/start-yarn.sh
```

3. 浏览ResourceManager的网页界面;默认情况下可在:
> ResourceManager - http://localhost:8088/

4. 运行一个MapReduce作业。

5. 当你完成时，停止守护进程:
```
  $ sbin/stop-yarn.sh
```




















