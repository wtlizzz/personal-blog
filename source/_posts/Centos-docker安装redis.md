title: Centos docker安装redis
author: Wtli
date: 2021-11-17 16:49:51
tags:
---
之前服务器配置了服务端，需要redis，正好没有docker，一块记录一下配置docker+redis的过程。

<!-- more -->

#### 安装docker

使用官方安装脚本自动安装
安装命令如下：

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```
curl -sSL https://get.daocloud.io/docker | sh
```

我是按照第一个命令安装，安装过程遇见小问题，就是yum命令不能更新，使用的时候报错：

```
[root@VM-8-14-centos ~]# curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# Executing docker install script, commit: 93d2499759296ac1f9c510605fef85052a2c32be
+ sh -c 'yum install -y -q yum-utils'
错误：rpmdb: BDB0113 Thread/process 21274/140445425371200 failed: BDB1507 Thread died in Berkeley DB library
错误：db5 错误(-30973) 来自 dbenv->failchk：BDB0087 DB_RUNRECOVERY: Fatal error, run database recovery
错误：无法使用 db5 -  (-30973) 打开 Packages 索引
错误：无法从 /var/lib/rpm 打开软件包数据库
CRITICAL:yum.main:
```

解决方式，通过删除yum的数据库，重新build yum的数据库，具体原因估计是版本更新造成的：

```
[root@VM-8-14-centos rpm]# cd /var/lib/rpm
[root@VM-8-14-centos rpm]# rm -rf __db*
[root@VM-8-14-centos rpm]# rpm --rebuilddb
```

然后重新安装docker，这样就成功了

#### 运行docker

虽然安装docker成功了，但是运行的时候却显示：

```
[root@VM-8-14-centos rpm]# docker ps
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

那下一步还需要运行docker，运行下面命令：

```
[root@VM-8-14-centos rpm]# sudo systemctl start docker
```

这样算是安装运行完成！

#### 安装redis，拉取镜像

你可以通过 docker search 命令来查找官方仓库中的镜像，并利用docker pull 命令来将它下载到本地。查找redis的镜像

```
[root@VM-8-14-centos /]# docker search redis
NAME                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
redis                            Redis is an open source key-value store that…   10172     [OK]    
[root@VM-8-14-centos /]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis
7d63c13d9b9b: Pull complete 
a2c3b174c5ad: Pull complete 
283a10257b0f: Pull complete 
7a08c63a873a: Pull complete 
0531663a7f55: Pull complete 
9bf50efb265c: Pull complete 
Digest: sha256:54ee15a0b0d2c661d46b9bfbf55b181f9a4e7ddf8bf693eec5703dac2c0f5546
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

#### 运行镜像

绑定端口，后台运行redis

```
[root@VM-8-14-centos /]# docker run -p 6379:6379 -d redis:latest redis-server
```

结束🔚！














