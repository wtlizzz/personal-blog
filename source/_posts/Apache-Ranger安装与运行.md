title: Apache Ranger安装与运行
author: Wtli
tags:
  - 大数据
categories: []
date: 2022-03-24 09:50:00
---

本文是关于Apache Ranger的安装教程，与ranger-usersync系统用户同步的配置，与针对HDFS的策略配置，以及Ldap用户同步。

<font color="grey"> 直接使用源码编译安装，不使用Ambari </font>

![](/images/pasted-110.png)

<!-- more -->

##### Ranger编译

附：[git地址](https://github.com/apache/ranger)

从git中下载源码，直接git clone或者下载压缩包。

然后在根目录中运行下面命令：

```
$ mvn clean compile package install
```

编译时间很长，由于是国内，下载maven依赖很慢，我是用了一天时间。。。

如果能成功，就OK；

编译可能遇见的报错：

1. 如果报错是因为有的jar包依赖下载不下来，则直接去maven仓库网站中，下载相应的jar包依赖到本地的maven仓库中。
2. 如果是因为PhantomJS报错，则在运行命令后面加上-DskipJSTests，这个参数跳过测试。PhantomJS是用来测试的，也是因为网络问题可能会报错。

编译完成后，在target文件夹中就会生成多个tar.gz文件：

![upload successful](/images/pasted-122.png)


##### 配置Mysql

需要提前安装mysql，用于Ranger的数据存储
在mysql中创建ranger数据库，并且创建ranger用户，赋予权限。
```
$ mysql -uroot -p000000
$ mysql> create database ranger;
$ mysql> grant all privileges on ranger.* to ranger@'%'  identified by 'ranger';
```

这里我遇见的报错信息：

1. root用户授权失败，这里需要注意root用户是否有赋予其他用户权限的权限。
2. Ranger运行的sql，创建表失败，字段太长，Mysql最大支持的长度不够。

针对问题1，需要先查询root用户的权限，直接用navicat查看mysql.user表，或者跑下边的命令。

```
$ select Grant_priv,Super_priv from mysql.user where user = 'root' and host = '%';
```

如果两个有N的话，就需要更新root用户权限，然后在进行授权。

```
$ update mysql.user set Grant_priv='Y',Super_priv='Y' where user = 'root' and host = '%';

$ flush privileges;
```

针对问题2，有一个初始化的sql文件报错，报错原因是有一个表，两个字段的长度初始化创建的是4000，我是将字段长度改成了1000，就能成功安装。

##### 安装Ranger

创建ranger文件夹，将**ranger-3.0.0-SNAPSHOT-admin.tar.gz**文件解压缩：

```
$ mkdir /opt/module/ranger
$ tar -zxvf ranger-3.0.0-SNAPSHOT-admin.tar.gz -C /opt/module/ranger
```

进入Ranger文件夹，修改**install.properties**配置文件：

```
$ cd /opt/module/ranger
$ vim install.properties
```

修改内容如下：

```
#mysql驱动
SQL_CONNECTOR_JAR=/opt/software/mysql-connector-java-5.1.48.jar

#mysql的主机名和root用户的用户名密码
db_root_user=root
db_root_password=000000
db_host=hadoop102

#ranger需要的数据库名和用户信息，和2.2.1创建的信息要一一对应
db_name=ranger
db_user=ranger
db_password=ranger

#Ranger各组件的admin用户密码，这里必须是大于8位的，不然后面运行报错
rangerAdmin_password=ranger123
rangerTagsync_password=ranger123
rangerUsersync_password=ranger123
keyadmin_password=ranger123

#ranger存储审计日志的路径，默认为solr，这里为了方便暂不设置
audit_store=

#策略管理器的url,rangeradmin安装在哪台机器，主机名就为对应的主机名
policymgr_external_url=http://hadoop102:6080

#启动ranger admin进程的linux用户信息
unix_user=ranger
unix_user_pwd=ranger
unix_group=ranger
```

改完运行安装脚本：

```
$ ./setup.sh
```

出现以下信息，说明安装完成
```
[I] Ranger all admins default password change request processed successfully..
Installation of Ranger PolicyManager Web Application is completed.
```

##### 运行配置

安装完成，再进行配置的修改：

```
$ vi conf/ranger-admin-site.xml
```
增加如下参数，ranger.service.host这个参数是hadoop的namenode所在的主机。
```
<property>
    <name>ranger.jpa.jdbc.password</name>
    <value>ranger</value>
</property>

<property>
    <name>ranger.service.host</name>
    <value>hadoop102</value>
</property>
```

##### 启动Ranger

这里环境变量已经配置好了，直接用ranger-admin启动就行，并且ranger-admin在安装时已经配设置为开机自启动，因此之后无需再手动启动。

```
$ ranger-admin start
```

查看启动后的进程：
```
$ jps
7058 EmbeddedServer
8132 Jps
```

访问Ranger的WebUI，地址为：http://hadoop102:6080

![upload successful](/images/pasted-124.png)

登陆用户名是admin，密码是之前设置的ranger123。登录成功以后界面如下，我是内网配置运行的，用手机拍照，后期配置了一个hdfs的策略。

![upload successful](/images/pasted-126.png)

##### 安装RangerUsersync

RangerUsersync作为Ranger提供的一个管理模块，可以将Linux机器上的用户和组信息同步到RangerAdmin的数据库中进行管理。

首先，解压软件：

```
$ tar -zxvf ranger-3.0.0-SNAPSHOT-usersync.tar.gz -C /opt/module/ranger/
```

修改安装配置文件：

```
$ cd /opt/module/ranger/
$ vim install.properties
```

修改内容：
```
#rangeradmin的url
POLICY_MGR_URL =http://hadoop102:6080

#同步间隔时间，单位(分钟)
SYNC_INTERVAL = 1

#运行此进程的linux用户
unix_user=ranger
unix_group=ranger

#rangerUserSync用户的密码，参考rangeradmin中install.properties的配置
rangerUsersync_password=ranger123
```

运行安装脚本：

```
$ ./setup.sh
```

出现以下信息，说明安装完成
```
ranger.usersync.policymgr.password has been successfully created.
Provider jceks://file/etc/ranger/usersync/conf/rangerusersync.jceks was updated.
[I] Successfully updated password of rangerusersync user
```

修改运行配置文件：

```
$ vi conf/ranger-ugsync-site.xml
```

```
<property>
      <name>ranger.usersync.enabled</name>
      <value>true</value>
</property>
```

然后，在Ranger-admin界面的Setting-Users/Groups/Roles菜单中就能够，看到一个新增的External用户，都是Linux中原先的用户。

##### 安装Ranger-HDFS插件

与上文中usersync类似，首先将压缩包解压：

```
$ tar -zxvf ranger-3.0.0-SNAPSHOT-hdfs.tar.gz -C /opt/module/ranger/
```
然后修改安装配置文件install.properties

```
# Ranger admin地址
POLIVY_MGR_URL=HTTP://hadoop102:6080

# 策略配置名称
REPOSITORY_NAME=hdfs_demo

# hadoop地址
COMPONENT_INSTALL_DIR_NAME=/aircas/hadoop

#hdfs组件使用的user
CUSTOM_USER=ranger
CUSTOM_GROUP=ranger
```

然后运行**enable-hdfs-plugin.sh**文件。

我们能够在hadoop中的hdfs-site.xml文件中看到，添加了Ranger相关的权限认证。

##### Ranger-HDFS权限配置

由于在内网中配置，运行，测试。在此就简单介绍一下流程，不配图了。

首先在Ranger-admin界面中，在Access Manager-HDFS栏，点击➕，新增服务。

新增服务完成后，然后在点击服务的名称，就会跳转到新增Policies中，点击右侧的Add New Policy按钮，进入策略编辑界面。

我们就可以添加相应的控制策略，能够针对某一个文件夹，递归的进行细粒度的控制。

HDFS策略测试，使用到的命令如下：

```
# 使用root用户，在hdfs中创建文件夹，并上传文件到文件夹中
$ hadoop fs -mkdir /hdfstest
$ hadoop fs -put lwt.properties /hdfstest/sub

# 切换用户ranger，这里使用su -，命令切换用户的同时切换shell，防止ranger用户中没有hadoop命令
$ su - ranger
$ hadoop fs -delete /hdfstest/sub/lwt.properties

# 使用ranger用户删除文件，将会报权限不允许，这时在ranger-admin中配置策略，在此删除，就能成功删除
```

遇到的问题：

创建ranger用户，需要在home文件夹中创建ranger文件夹，并且需要赋予ragner用户权限。













