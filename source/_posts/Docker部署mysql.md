title: Docker部署mysql
author: Wtli
tags:
  - Docker
categories:
  - Docker
date: 2020-09-28 16:54:00
---
在服务器中部署并配置Mysql。
<!--more-->
### 拉取镜像

之前使用的mysql的镜像，不是很好配置，改成mysql-server镜像。

```
docker pull mysql/mysql-server
```

### 运行容器

```
$ docker run -p 0.0.0.0:3305:3306 --name=mysql1 -d mysql/mysql-server
```
```
-i, --interactive                    Keep STDIN open even if not attached  
-t, --tty                            Allocate a pseudo-TTY
```
显示容器docker ps ==docker container ls  
-a :显示所有的容器，包括未运行的。

```
$ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                             PORTS                 NAMES
70543eea61bb        mysql/mysql-server   "/entrypoint.sh mysq…"   23 seconds ago      Up 22 seconds (health: starting)   3306/tcp, 33060/tcp   mysqlD
```


### 查看日志密码

```
$ docker logs mysqlD
```

在显示的日志里，找到初始密码，可以使用

docker logs mysql1 2>&1 | grep GENERATED

这个快捷命令。
```
[Entrypoint] GENERATED ROOT PASSWORD: m3fxIm]eh@dYStoRwExin#Aw@z9
```

### 进入mysql

复制之前在log中的初始密码
```
$ docker exec -it mysql1 mysql -uroot -p
```

修改用户登陆密码
```
$ ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';
```

修改访问host地址,刷新权限，否则本地访问不到
```
$ use mysql;
$ update user set host = '%' where user = 'root';
$ FLUSH PRIVILEGES;
```

### 导出容器

```
$ docker export 8301c1e81dac > mysql.tar
```

### 传输到服务器

使用scp命令。
```
$ scp /Users/lli/Desktop/mysql.tar root@172.24.3.19:/mysql
```
将容器快照导入为镜像
```
$ cat mysql.tar | docker import - mysql/mysql-server:0929 
```
运行导入的镜像必须带command。可以在导出容器的时候查看，
```
(base) LdeMacBook-Pro:Desktop lli$ docker ps --no-trunc
CONTAINER ID                                                       IMAGE                COMMAND                   CREATED             STATUS                       PORTS                               NAMES
8301c1e81dac33be77b14d0d58af56e40204d80060d9beeca17c74d6a7b5fb06   mysql/mysql-server   "/entrypoint.sh mysqld"   About an hour ago   Up About an hour (healthy)   33060/tcp, 0.0.0.0:3305->3306/tcp   mysql4
```
在服务器中运行就要带着"/entrypoint.sh mysqld"
```
$  docker run -p 0.0.0.0:3305:3306 --name=mysql1 -d mysql/mysql-server:0929 /entrypoint.sh mysqld
```
**注意**
如果不带的话就会报错：
```
[root@k8s-master mysql]# docker run -p 0.0.0.0:3306:3306 --name=mysql1 -d mysql/mysql-server:0929
docker: Error response from daemon: No command specified.
See 'docker run --help'.
```


