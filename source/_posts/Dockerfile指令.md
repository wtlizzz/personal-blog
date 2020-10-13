title: Dockerfile指令
author: Wtli
tags:
  - Docker
categories:
  - Docker
  - ''
date: 2020-09-27 16:55:00
---
本文主要对Dockerfile的指令进行记录。
<!--more-->

### FROM

**指定基础镜像**

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行 了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并 且必须是第一条指令。

### RUN

**执行命令**

RUN 指令是用来执行命令行命令的。由于命令行的强大能力， RUN 指令在定制 镜像时是最常用的指令之一。其格式有两种：

- shell 格式： RUN <命令> ，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。

```
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index .html
```

- exec 格式： RUN \["可执行文件", "参数1", "参数2"] ，这更像是函数调用中的格式。

既然 RUN 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本 一样把每个命令对应一个 RUN 呢？比如这样：
```
FROM debian:stretch

RUN apt-get update 
RUN apt-get install -y gcc libc6-dev make wget 
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redi s-5.0.3.tar.gz" 
RUN mkdir -p /usr/src/redis 
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 
RUN make -C /usr/src/redis 
RUN make -C /usr/src/redis install
```
之前说过，Dockerfile 中每一个指令都会建立一层， RUN 也不例外。每一个 RUN 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行 这些命令，执行结束后， commit 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需 要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生 非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是 很多初学 Docker 的人常犯的一个错误。

Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是 不得超过 127 层。


上面的Dockerfile 正确的写法应该是这样：

```
FROM debian:stretch

RUN buildDeps='gcc libc6-dev make wget' \ 
	&& apt-get update \ 
	&& apt-get install -y $buildDeps \ 
    && wget -O redis.tar.gz "http://download.redis.io/releases/r edis-5.0.3.tar.gz" \ 
    && mkdir -p /usr/src/redis \ 
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-component s=1 \ 
    && make -C /usr/src/redis \ 
    && make -C /usr/src/redis install \ 
    && rm -rf /var/lib/apt/lists/* \ 
    && rm redis.tar.gz \ 
    && rm -r /usr/src/redis \ 
    && apt-get purge -y --auto-remove $buildDeps
```

首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没 有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 RUN 对一一 对应不同的命令，而是仅仅使用一个 RUN 指令，并使用 && 将各个所需命令串 联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 \ 的 命令换行方式，以及行首 # 进行注释的格式。良好的格式，比如换行、缩进、注 释等，会让维护、排障更为容易，这是一个比较好的习惯。

此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建 所需要的软件，清理了所有下载、展开的文件，并且还清理了 apt 缓存文件。这 是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层 被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要 添加的东西，任何无关的东西都应该清理掉。


### COPY 
格式：
```
- COPY [--chown=<user>:<group>] <源路径>... <目标路径>

- COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```
和RUN 指令一样，也有两种格式，一种类似于命令行，一种类似于函数调用。

COPY 指令将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。比如：
```
COPY package.json /usr/src/app/
```
<源路径> 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 filepath.Match 规则，如：
```
COPY hom* /mydir/ 
COPY hom?.txt /mydir/
```
<目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工 作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存 在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 COPY 指令，源文件的各种元数据都会保留。比如 读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建 相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 属用户及所属组。--chown=\<user>:\<group>选项来改变文件的所属用户及所属组。

```
COPY --chown=55:mygroup files* /mydir/ 
COPY --chown=bin files* /mydir/ 
COPY --chown=1 files* /mydir/ 
COPY --chown=10:11 files* /mydir/
```

### ADD 
**更高级的复制文件**

ADD 指令和COPY 的格式和性质基本一致。但是在COPY 基础上增加了一些功能。

比如<源路径>可以是一个URL，这种情况下，Docker 引擎会试图去下载这个链接的文件放到<目标路径>去。下载后的文件权限自动设置为600，如果这并不是想要的权限，那么还需要增加额外的一层 RUN 进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 RUN 指令进行解压缩。所以不如直接使用 RUN 指令，然后使用wget或者curl工具下载，处理权限、解压缩、然后清理无用文件更合理。因此，这个功能其实并不实用，而且不推荐使用。

如果<源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip , bzip2 以及 xz 的情况下， ADD 指令将会自动解压缩这个压缩文件到<目标路径> 去。

在某些情况下，这个自动解压缩的功能非常有用，比如官方镜像ubuntu 中：
```
FROM scratch 
ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz / 
...
```
但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用 ADD 命令了。

在 Docker 官方的 Dockerfile 最佳实践文档 中要求，尽可能的使用 COPY ，因为 COPY 的语义很明确，就是复制文件而已，而 ADD 则包含了更复杂的功能，其 行为也不一定很清晰。最适合使用 ADD 的场合，就是所提及的需要自动解压缩的 场合。

另外需要注意的是， ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变 得比较缓慢。

### CMD 

**容器启动命令**

CMD指令的格式和RUN相似，也是两种格式：

- shell 格式： CMD <命令> 
- exec 格式： CMD \["可执行文件", "参数1", "参数2"...] 
- 参数列表格式： CMD \["参数1", "参数2"...] 。在指定了ENTRYPOINT指令后，用 CMD 指定具体的参数。

之前介绍容器的时候曾经说过，Docker 不是虚拟机，容器就是进程。既然是进 程，那么在启动容器的时候，需要指定所运行的程序及参数。 CMD 指令就是用于指定默认的容器主进程的启动命令的。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如， ubuntu 镜像默认的 CMD 是 /bin/bash ，如果我们直接 docker run -it ubuntu 的
话，会直接进入bash 。我们也可以在运行时指定运行别的命令，如 docker run -it ubuntu cat /etc/os-release 。这就是用 cat /etc/os-release 命令替换了默认的 /bin/bash 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号" ，而不要使用单引号。

如果使用shell格式的话，实际的命令会被包装为sh -c 的参数的形式进行执行。比如：
```
CMD echo $HOME
```
在实际执行中，会将其变更为：
```
CMD [ "sh", "-c", "echo $HOME" ]
```
这就是为什么我们可以使用环境变量的原因，因为这些环境变量会被 shell 进行解 析处理。

提到 CMD 就不得不提容器中应用在前台执行和后台执行的问题。这是初学者常出 现的一个混淆。

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机 里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。

一些初学者将CMD写为：
```
CMD service nginx start
```
然后发现容器执行后就立即退出了。甚至在容器内去使用 systemctl 命令结果 却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和 虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主 进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的 东西。

而使用 service nginx start 命令，则是希望 upstart 来以后台守护进程形式启动 nginx 服务。而刚才说了 CMD service nginx start 会被理解为 CMD \["sh", "-c", "service nginx start"] ，因此主进程实际上是 sh 。那么当 service nginx start 命令结束后， sh 也就结束了， sh 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行nginx 可执行文件，并且要求以前台形式运行。比如：
```
CMD ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT

**入口点**

ENTRYPOINT的格式和RUN 指令格式一样，分为exec格式和shell格式。

ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参 数。 ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 --entrypoint 来指定。

当指定了 ENTRYPOINT 后， CMD 的含义就发生了改变，不再是直接的运行其命 令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令，换句话说实际执行 时，将变为：
```
<ENTRYPOINT> "<CMD>"
```
那么有了CMD后，为什么还要有 ENTRYPOINT 呢？这种 \<CMD>" 有什么好处么？让我们来看几个场景。

#### 场景一：让镜像变成像命令一样使用

假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用CMD来实现：
```
FROM ubuntu:18.04 
RUN apt-get update \
	&& apt-get install -y curl \
	&& rm -rf /var/lib/apt/lists/* 
CMD [ "curl", "-s", "https://ip.cn" ]
```
假如我们使用 docker build -t myip .来构建镜像的话，如果我们需要查询当前公网 IP，只需要执行：

```
$ docker run myip 
当前 IP：61.148.226.66 来自：北京市 联通
```
嗯，这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我 们希望加参数呢？比如从上面的 CMD 中可以看到实质的命令是 curl ，那么如 果我们希望显示 HTTP 头信息，就需要加上 -i 参数。那么我们可以直接加 -i 参数给 docker run myip 么？

```
$ docker run myip -i 
docker: Error response from daemon: invalid header field value " oci runtime error: container_linux.go:247: starting container pr ocess caused \"exec: \\\"-i\\\": executable file not found in $P ATH\"\n".
```
我们可以看到可执行文件找不到的报错，executable file not found 。之前 我们说过，跟在镜像名后面的是 command ，运行时会替换 CMD 的默认值。因此 这里的 -i 替换了原来的 CMD ，而不是添加在原来的 curl -s https://ip.cn 后面。而 -i 根本不是命令，所以自然找不到。

那么如果我们希望加入-i 这参数，我们就必须重新完整的输入这个命令：

```
$ docker run myip curl -s https://ip.cn -i
```

这显然不是很好的解决方案，而使用 ENTRYPOINT 就可以解决这个问题。现在我 们重新用 ENTRYPOINT 来实现这个镜像：

```
FROM ubuntu:18.04 RUN apt-get update \

&& apt-get install -y curl \

&& rm -rf /var/lib/apt/lists/* ENTRYPOINT [ "curl", "-s", "https://ip.cn" ]
```
这次我们再来尝试直接使用docker run myip -i ：

```
$ docker run myip -i 
HTTP/1.1 200 OK 
Server: nginx/1.8.0 
Date: Tue, 22 Nov 2016 05:12:40 GMT 
Content-Type: text/html; charset=UTF-8 
Vary: Accept-Encoding 
X-Powered-By: PHP/5.6.24-1~dotdeb+7.1 
X-Cache: MISS from cache-2 
X-Cache-Lookup: MISS from cache-2:80 
X-Cache: MISS from proxy-2_6 
Transfer-Encoding: chunked 
Via: 1.1 cache-2:80, 1.1 proxy-2_6:8006 
Connection: keep-alive
```
可以看到，这次成功了。这是因为当存在ENTRYPOINT后，CMD 的内容将会作为参数传给 ENTRYPOINT ，而这里 -i就是新的 CMD ，因此会作为参数传给curl ，从而达到了我们预期的效果。

#### 场景二：应用运行前的准备工作

启动容器就是启动主进程，但有些时候，启动主进程前，需要一些准备工作。

比如 mysql 类的数据库，可能需要一些数据库配置、初始化的工作，这些工作要 在最终的 mysql 服务器运行之前解决。

此外，可能希望避免使用 root 用户去启动服务，从而提高安全性，而在启动服 务前还需要以 root 身份执行一些必要的准备工作，最后切换到服务用户身份启 动服务。或者除了服务外，其它命令依旧可以使用 root 身份执行，方便调试 等。

这些准备工作是和容器 CMD 无关的，无论 CMD 为什么，都需要事先进行一个 预处理的工作。这种情况下，可以写一个脚本，然后放入 ENTRYPOINT 中去执 行，而这个脚本会将接到的参数（也就是 \<CMD> ）作为命令，在脚本最后执行。 比如官方镜像 redis 中就是这么做的：

```
FROM alpine:3.4 
...
RUN addgroup -S redis && adduser -S -G redis redis 
...
ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 6379 
CMD [ "redis-server" ]
```

可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了ENTRYPOINT为 docker-entrypoint.sh 脚本。
```
#!/bin/sh ...

# allow the container to be started with `--user`

ENTRYPOINT

if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then

	chown -R redis .

	exec su-exec redis "$0" "$@"
fi

exec "$@"
```

该脚本的内容就是根据 CMD 的内容来判断，如果是 redis-server 的话，则切 换到 redis 用户身份启动服务器，否则依旧使用 root 身份执行。比如：

```
$ docker run -it redis id uid=0(root) gid=0(root) groups=0(root)
```
### ENV
**设置环境变量**

格式有两种：

- ENV \<key> \<value>

- ENV \<key1>=\<value1> \<key2>=\<value2>...

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如RUN ，还是运行时的应用，都可以直接使用这里定义的环境变量。

```
ENV VERSION=1.0 DEBUG=on \
	NAME="Happy Feet"
```

这个例子中演示了如何换行，以及对含有空格的值用双引号括起来的办法，这和 Shell 下的行为是一致的。

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方 node 镜像 Dockerfile 中，就有类似这样的代码：

```
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NOD E_VERSION-linux-x64.tar.xz" \

&& curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS25 6.txt.asc" \

&& gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.tx t.asc \

&& grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.t xt | sha256sum -c - \

&& tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/loc

al --strip-components=1 \

&& rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.as c SHASUMS256.txt \ && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

在这里先定义了环境变量 NODE_VERSION ，其后的 RUN 这层里，多次使用 $NODE_VERSION 来进行操作定制。可以看到，将来升级镜像构建版本的时候，只 需要更新 7.2.0 即可， Dockerfile 构建维护变得更轻松了。

下列指令可以支持环境变量展开：
ADD 、 COPY 、 ENV 、 EXPOSE 、 LABEL 、 USER 、 WORKDIR 、 VOLUME 、 STOPSIGNAL 、 ONBUILD 。

可以从这个指令列表里感觉到，环境变量可以使用的地方很多，很强大。通过环境 变量，我们可以让一份 Dockerfile 制作更多的镜像，只需使用不同的环境变量 即可。

### ARG
**构建参数**


格式： ARG \<参数名>\[=<默认值>]

构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是， ARG 所设置的 构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因 此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有 值的。

Dockerfile中的ARG指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 docker build 中用 \--build-arg <参数名>=<值> 来覆盖。

在 1.13 之前的版本，要求 --build-arg 中的参数名，必须在 Dockerfile 中 用 ARG 定义过了，换句话说，就是 --build-arg 指定的参数，必须在 Dockerfile 中使用了。如果对应参数没有被使用，则会报错退出构建。从 1.13 开始，这种严格的限制被放开，不再报错退出，而是显示警告信息，并继续构建。 这对于使用 CI 系统，用同样的构建流程构建不同的 Dockerfile 的时候比较有 帮助，避免构建命令必须根据每个 Dockerfile 的内容修改。

### VOLUME
**定义匿名卷**

格式为：
- VOLUME \["<路径1>", "<路径2>"...] 
- VOLUME <路径>

之前我们说过，容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类 需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中，为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```
VOLUME /data
```

这里的 /data 目录就会在运行时自动挂载为匿名卷，任何向 /data 中写入的 信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行时 可以覆盖这个挂载设置。比如：

```
docker run -d -v mydata:/data xxxx
```

在这行命令中，就使用了 mydata 这个命名卷挂载到了/data 这个位置，替代了 Dockerfile 中定义的匿名卷的挂载配置。

### EXPOSE

**声明端口**

格式为EXPOSE \<端口1> \[<端口2>...] 。

EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不 会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有 两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映 射；另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

要将 EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。 -p ，是 映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访 问，而 EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行 端口映射。

### WORKDIR

**指定工作目录** 

格式为WORKDIR <工作目录路径> 。

使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前 目录就被改为指定的目录，如该目录不存在， WORKDIR 会帮你建立目录。

之前提到一些初学者常犯的错误是把 Dockerfile 等同于 Shell 脚本来书写，这 种错误的理解还可能会导致出现下面这样的错误：
```
RUN cd /app 
RUN echo "hello" > world.txt
```

如果将这个Dockerfile进行构建镜像运行后，会发现找不到 /app/world.txt 文件，或者其内容不是 hello。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；而在 Dockerfile 中，这两行 RUN 命令的执行环境根本不同，是两个完全不同的容器。这就是对 Dockerfile 构建分层存储的概念不了解所导致的错误。

之前说过每一个 RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。 第一层 RUN cd /app 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的 容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用WORKDIR 指令。

### USER

**指定当前用户**

格式： USER <用户名>\[:<用户组>]

USER指令和WORKDIR相似，都是改变环境状态并影响以后的层。 WORKDIR是改变工作目录， USER 则是改变之后层的执行 RUN , CMD 以及ENTRYPOINT 这类命令的身份。

当然，和 WORKDIR 一样， USER 只是帮助你切换到指定用户而已，这个用户必 须是事先建立好的，否则无法切换。
```
RUN groupadd -r redis && useradd -r -g redis redis 
USER redis 
RUN [ "redis-server" ]
```
如果以 root 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立 好的用户来运行某个服务进程，不要使用 su 或者 sudo ，这些都需要比较麻烦 的配置，而且在 TTY 缺失的环境下经常出错。建议使用 gosu 。

```
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令 
RUN groupadd -r redis && useradd -r -g redis redis 
# 下载 gosu 
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/ 
releases/download/1.7/gosu-amd64" \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true 
    # 设置 CMD，并以另外的用户执行 
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

### HEALTHCHECK

**健康检查**

格式：

- HEALTHCHECK \[选项] CMD <命令> ：设置检查容器健康状况的命令 
- HEALTHCHECK NONE ：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是Docker 1.12 引入的新指令。

### ONBUILD

**为他人做嫁衣裳**

格式： ONBUILD <其它指令> 。

ONBUILD 是一个特殊的指令，它后面跟的是其它指令，比如 RUN , COPY 等， 而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去 构建下一级镜像的时候才会被执行。

Dockerfile中的其它指令都是为了定制当前镜像而准备的，唯有ONBUILD是为了帮助别人定制自己而准备的。









