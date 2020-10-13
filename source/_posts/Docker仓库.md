title: Docker仓库
author: Wtli
tags:
  - Docker
categories:
  - Docker
date: 2020-09-28 09:07:00
---
仓库（ Repository ）是集中存放镜像的地方。
类似于gihthub，本文记录如何使用dockerhub。

<!--more-->

### 登录

可以通过执行docker login命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。

你可以通过docker logout退出登录。

### 拉取镜像

你可以通过 docker search 命令来查找官方仓库中的镜像，并利用docker pull 命令来将它下载到本地。

例如以centos 为关键词进行搜索：
```
(base) LdeMacBook-Pro:~ lli$ docker search centos
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
centos                             The official build of CentOS.                   6207                [OK]                
ansible/centos7-ansible            Ansible on Centos7                              132                                     [OK]
consol/centos-xfce-vnc             Centos container with "headless" VNC session…   122                                     [OK]
jdeathe/centos-ssh                 OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   115                                     [OK]
centos/systemd                     systemd enabled base container.                 86                                      [OK]                                   
```

可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、收藏数（表示 该镜像的受关注程度）、是否官方创建、是否自动创建。

官方的镜像说明是官方项目组创建和维护的，automated 资源允许用户验证镜像的 来源和内容。

根据是否是官方提供，可将镜像资源分为两类。

一种是类似 centos 这样的镜像，被称为基础镜像或根镜像。这些基础镜像由 Docker 公司创建、验证、支持、提供。这样的镜像往往使用单个单词作为名字。

还有一种类型，比如 tianon/centos 镜像，它是由 Docker 的用户创建并维护 的，往往带有用户名称前缀。可以通过前缀 username/ 来指定使用某个用户提 供的镜像，比如 tianon 用户。

下载官方centos 镜像到本地。

```
$ docker pull centos 
Pulling repository centos 
0b443ba03958: Download complete 
539c0211cd76: Download complete 
511136ea3c5a: Download complete 
7064731afe90: Download complete
```

### 推送镜像

用户也可以在登录后通过docker push命令来将自己的镜像推送到 Docker Hub。以下命令中的username 请替换为你的 Docker 账号用户名。

```
$ docker push username/ubuntu:18.04
```

### 私有仓库

有时候使用 Docker Hub 这样的公共仓库可能不方便，用户可以创建一个本地仓库 供私人使用。

本节介绍如何使用本地仓库。

docker-registry 是官方提供的工具，可以用于构建私有的镜像仓库。本文内容 基于 docker-registry v2.x 版本。

#### 安装运行 docker-registry

容器运行

你可以通过获取官方registry 镜像来运行。

```
$ docker run -d -p 5000:5000 --restart=always --name registry re gistry
```

这将使用官方的 registry 镜像来启动私有仓库。默认情况下，仓库会被创建在 容器的 /var/lib/registry 目录下。你可以通过 -v 参数来将镜像文件存放在 本地的指定路径。例如下面的例子将上传的镜像放到本地的/opt/data/registry 目录。

```
$ docker run -d \

  -p 5000:5000 \

  -v /opt/data/registry:/var/lib/registry \ registry
```

#### 在私有仓库上传、搜索、下载镜像

创建好私有仓库之后，就可以使用 docker tag 来标记一个镜像，然后推送它到 仓库。例如私有仓库地址为 127.0.0.1:5000 。

使用docker tag 将 ubuntu:latest 这个镜像标记为
127.0.0.1:5000/ubuntu:latest 。

格式为
```
$ docker tag IMAGE\[:TAG\] \[REGISTRY_HOST\[:REGISTRY_PORT]/]REPOSITORY\[:TAG]
```

使用
```
$ docker push     上传标记的镜像。
```

用curl查看仓库中的镜像。

```
$ curl 127.0.0.1:5000/v2/_catalog 
{"repositories":["ubuntu"]}
```

这里可以看到{"repositories":\["ubuntu"]} ，表明镜像已经被成功上传了。

这样就可以下载了：
```
$ docker pull 127.0.0.1:5000/ubuntu:latest 
Pulling repository 127.0.0.1:5000/ubuntu:latest 
ba5877dc9bec: Download complete 
511136ea3c5a: Download complete 
9bad880da3d2: Download complete 
25f11f5fb0cb: Download complete 
ebc34468f71d: Download complete 
2318d26665ef: Download complete
```

**注意事项：**

如果你不想使用 127.0.0.1:5000 作为仓库地址，比如想让本网段的其他主机也 能把镜像推送到私有仓库。你就得把例如 192.168.199.100:5000 这样的内网地 址作为私有仓库地址，这时你会发现无法成功推送镜像。

这是因为 Docker 默认不允许非 HTTPS 方式推送镜像。我们可以通过 Docker 的 配置选项来取消这个限制，或者查看下一节配置能够通过 HTTPS 访问的私有仓 库。



















