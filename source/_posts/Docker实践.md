title: Docker实践
author: Wtli
tags:
  - Docker
categories:
  - Docker
  - ''
  - ''
date: 2020-09-27 15:36:00
---
Docker的简单实践一个webserver\--nginx。
<!--more-->

**首先明确镜像和容器的定位。**

镜像是容器的基础，每次执行 docker run 的时候都会指定哪个镜像作为容器运 行的基础。


### Web服务器

定制一个简单的web服务器，使用nginx镜像。

1. 使用命令下载下nginx镜像
```
$ docker pull nginx
```
2. 启动容器，以刚下载的nginx为镜像。
```
$ docker run --name webserver -d -p 80:80 nginx
```
这条命令会用 nginx 镜像启动一个容器，命名为webserver,并且映射了80端口，这样我们可以用浏览器去访问这个nginx服务器。

如果是在Linux 本机运行的 Docker，或者如果使用的是 Docker for Mac、Docker for Windows，那么可以直接访问：http://localhost   如果使用的是 Docker Toolbox，或者是在虚拟机、云服务器上安装的Docker，则需要将 localhost 换为虚拟机地址或者实际云服务器地址。

然后我们可以看到nginx欢迎界面：

![upload successful](/images/pasted-13.png)

现在，假设我们非常不喜欢这个欢迎页面，我们希望改成欢迎 Docker 的文字，我 们可以使用 docker exec 命令进入容器，修改其内容。

```
$ docker exec -it webserver bash
```

我们以交互式终端方式进入 webserver容器，并执行了bash命令，也就是获得一个可操作的Shell。
运行命令，然后我们刷新浏览器就能够看到浏览器发生了变化。
```
$ echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

我们修改了容器的文件，也就是改动了容器的存储层。我们可以通过命令看到具体的改动。
```
$ docker diff
```
运行之后我们可以看到webserver容器的改动

![upload successful](/images/pasted-14.png)

- A	创建了文件或目录  
- D	删除了文件或目录  
- C	修改了文件或目录

#### 将容器保存为镜像

类似于git命令，\--author "Wtli"  \--message "修改了默认网页" 这两个是作为\[OPTIONS\]，container是webserver，保存的\[REPOSITORY\[:TAG\]\]为nginx:0927。
```
$ docker commit  --author "Wtli"  --message "修改了默认网页" webserver nginx:0927
```

**使用 docker commit 命令虽然可以比较直观的帮助理解镜像分层存储的概念， 但是实际环境中并不会这样使用。**

首先，如果仔细观察之前的 docker diff webserver 的结果，你会发现除了真
正想要修改的 /usr/share/nginx/html/index.html文件外，由于命令的执行，还有很多文件被改动或添加了。这还仅仅是最简单的操作，如果是安装软件 包、编译构建，那会有大量的无关内容被添加进来，如果不小心清理，将会导致镜 像极为臃肿。


### Dockerfile

从刚才的 docker commit 的学习中，我们可以了解到，镜像的定制实际上就是 定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作 的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复 的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令 构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

在一个空白目录中，建立一个文本文件，并命名为Dockerfile：
```
FROM nginx RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index .html
```

#### FROM
FROM 指定基础镜像.所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个nginx镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

#### RUN
RUN 指令是用来执行命令行命令的。由于命令行的强大能力， RUN 指令在定制 镜像时是最常用的指令之一。其格式有两种：

- shell 格式： RUN <命令> ，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。
```
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index .html
```

- exec 格式： RUN \["可执行文件", "参数1", "参数2"\] ，这更像是函数调用中的格式。

#### 使用Dockerfile


```
Build an image from a Dockerfile

$ docker build [OPTIONS] PATH | URL | -
```
在Dockerfile 文件所在目录执行：
```
$ docker build -t nginx:v3 .
```

![upload successful](/images/pasted-15.png)


#### 镜像构建上下文（Context）

如果注意，会看到 docker build 命令最后有一个 '.' 。 '.' 表示当前目录，而 Dockerfile 就在当前目录，因此不少初学者以为这个路径是在指定 Dockerfile 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定上下文路径。

首先我们要理解 docker build 的工作原理。Docker 在运行时分为 Docker 引擎 （也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API， 被称为 Docker Remote API，而如 docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在 本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端 （Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。

当我们进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要 将一些本地文件复制进镜像，比如通过 COPY 指令、 ADD 指令等。而 docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎 中构建的。

这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路 径， docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上 传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜 像所需的一切文件。

如果在Dockerfile 中这么写：

```
COPY ./package.json /app/
```
这并不是要复制执行 docker build 命令所在的目录下的 package.json ，也 不是复制 Dockerfile 所在目录下的 package.json ，而是复制 上下文 （context） 目录下的 package.json 。

因此， COPY 这类指令中的源文件的路径都是相对路径。这也是初学者经常会问 的为什么 COPY ../package.json /app 或者 COPY /opt/xxxx /app 无法工
作的原因，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位 置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

现在就可以理解刚才的命令 docker build -t nginx:v3 '.' 中的这个 '.' ，实际上是在指定上下文的目录， docker build 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。

一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果 该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西 确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一 个 .dockerignore ，该文件是用于剔除不需要作为上下文传递给 Docker 引擎 的。

#### 其它docker build的用法

##### 直接用 Git repo 进行构建

这行命令指定了构建所需的 Git repo，并且指定默认的 master 分支，然后 Docker 就会自己去 git clone 这个项目、切换到指定分支、并进入到指定目录后开始构建。
```
$ docker build https://github.com/twang2218/gitlab-ce-zh.git
```

##### 用给定的 tar 压缩包构建

```
$ docker build http://server/context.tar.gz
```
如果所给出的 URL 不是个 Git repo，而是个 tar 压缩包，那么 Docker 引擎会下 载这个包，并自动解压缩，以其作为上下文，开始构建。