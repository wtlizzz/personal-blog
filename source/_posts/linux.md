title: linux修改ip修改root密码
author: Wtli
tags: []
categories: []
date: 2021-08-06 10:01:00
---
<font color = 'red'>笔记：</font>
使用镜像安装linux系统，使用命令行配置ip地址，并命令行修改root密码。

<!--more-->

背景：使用VMware ESXi，挂载ISO镜像系统。

安装完成，配置系统ip。

修改ip：

第一步：查看系统的网卡，
```
$ ifconfig

docker0   Link encap:以太网
ens192    Link encap:以太网
lo        Link encap:本地环回
virbr0    Link encap:以太网

```

```
$ sudo vi /etc/network/interfaces

interfaces:

# The loopback network interface
auto lo
iface lo inet loopback



```