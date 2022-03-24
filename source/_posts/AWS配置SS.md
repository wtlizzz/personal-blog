title: AWS配置SS
author: Wtli
tags: []
categories: []
date: 2020-12-10 10:49:00
---
AWS搞活动，12个月免费，正好用来学习。

<!--more-->

使用密钥连接服务器

```
$ sudo ssh -i "aws.pem" ec2-user@ip
```

命令行直接配置

**查看python版本**

```
$ python -V
```

**安装pip**

```
$ yum install python-pip
```
其他linux版本也可以使用apt-get

**安装ss**

```
$ pip install shadowsocks
```

**创建配置文件**

```
$ sudo vim /etc/shadowsocks.json
```
最好是使用root权限，不然有的服务器无法修改

内容如下：

```
{
    "server":"0.0.0.0",
    "server_port":8089,
    "local_port":1089,
    "password":"******",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

**启动服务**

```
$ ssserver -c /etc/shadowsocks.json -d start
```

以上就可以了。






