title: Docker部署SpringBoot服务
author: Wtli
tags:
  - Docker
categories:
  - Docker
date: 2020-09-30 08:30:00
---
本文记录使用Docker来部署SpringBoot服务的实践过程。
<!--more-->

### 拉取镜像

先拉取jdk的镜像。在这里选用的openjdk镜像。

```
(base) LdeMacBook-Pro:~ lli$ docker image ls
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
openjdk              latest              b2324c52d969        2 weeks ago         524MB
```

### 运行容器


```
$ docker run -itd --name javaServer b2324c52d969 bash
```


### 进入容器

```
$ docker exec -it 9dbb4dcfcdc6 bash
```
在根目录，创建java文件夹，用于存放jar包
```
$ mkdir java
```


### 放入jar包

在本主机运行docker cp命令，将jar包拷入docker。

```
$ docker cp /Users/lli/Desktop/monitor-0.1.jar c327d43ac94d:/java
```
命令详解：
```
Copy files/folders between a container and the local filesystem

Usage:	docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
	    docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

到现在为止，使用java -jar命令就能在本地运行了。

### 将容器打包成镜像

一下内容参考上一篇文章，部署mysql。

```
$ docker export c327d43ac94d > javaServer.tar
```

### 将镜像上传到服务器

在主机运行

```
$ scp /Users/lli/Desktop/javaServer.tar root@x.x.x.x:/mysql
```

### 加载镜像

在服务端运行，服务要用到8081和8082端口，将宿主机的对应端口暴露给docker。

```
$ docker import -itd -p 0.0.0.0:8081:8081 -p 0.0.0.0:8082:8082 --name javaServer javaServer.tar bash
```


### 运行

进入docker
```
$ docker exec -it containerId bash
```
找到对应的jar包存放目录，也可以通过将jar包上传到宿主机，然后使用docker cp命令，将宿主机中的jar包拷贝到docker里。
```
$ nohup java -jar aircas-monitor-0.1.jar log.out 2>&1 &
```

运行springboot的jar包，由于springboot是自带tomcat的，所以可以直接运行。

与正常运行命令不同的是，nohup可以让jar包后台运行，不会因为终端关闭而停止运行。
后边的log.outs是日志输出文件   
2>&1 输出所有的日志文件   
& 不影响终端启动，但还是会跟随终端关闭

```
$ java -jar 
```
下一步就是将日志保存到宿主机中，而不是docker中。未完待续～


### 附Redis部署

由于内网服务器，不能使用dockerhub中的镜像docker pull命令。所以需要在本地下载镜像，打包上传到服务器中。

#### 拉取镜像
```
$ docker pull redis
```

#### 运行镜像

由于我本机中已经占用了6379端口，先用6378端口。

1. 运行实例

```
$ docker run --name some-redis -d -p 0.0.0.0:6378:6379 redis
```


2. 持久存储，存储在docker的volume中

```
$ docker run --name redisServer -d -p 0.0.0.0:6378:6379 redis redis-server --appendonly yes
```

If persistence is enabled, data is stored in the VOLUME /data, which can be used with --volumes-from some-volume-container or -v /docker/host/dir:/data

#### 保存镜像

因为redis我在这没有更改配置文件，只是运行了docker，所以在这里直接打包镜像。

**docker save**

```
Usage:	docker save [OPTIONS] IMAGE [IMAGE...]
Save one or more images to a tar archive (streamed to STDOUT by default)
Options:
  -o, --output string   Write to a file, instead of STDOUT
  
$ docker save redis -o redisImage
```

#### 加载镜像

**docker load**

将保存的镜像发送到服务器，然后在服务器中加载镜像文件。

```

Usage:	docker load [OPTIONS]

Load an image from a tar archive or STDIN

Options:
  -i, --input string   Read from tar archive file, instead of STDIN
  -q, --quiet          Suppress the load output

$ docker load -i redisImage
```

### 附

#### 出错

在部署服务的时候突然报错，redis服务也已经部署上。

```
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'transformNetInfoImpl': Unsatisfied dependency expressed through field 'redisTemplate'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'redisTemplate' defined in class path resource [com/aircas/base/redis/config/RedisConfig.class]: Invocation of init method failed; nested exception is java.lang.IllegalStateException: RedisConnectionFactory is required
```

**解决：**由于自己知识不扎实，在打包的时候，不光把项目打包，还要把项目所依赖的项目，所依赖的辅助工程打包，在所有依赖工程中使用maven install一下，解决问题。






