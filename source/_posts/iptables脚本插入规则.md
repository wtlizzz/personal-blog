title: iptables脚本插入规则
author: Wtli
tags: []
categories: []
date: 2021-12-15 10:06:00
---
- iptables规则介绍
- sh脚本简单实现规则写入
<!--more-->

### iptables

#### iptables简单介绍

首先理解一个概念：iptables是什么。

iptables其实不是真正的防火墙，我们可以把它理解成一个客户端代理，用户通过iptables这个代理，将用户的安全设定执行到对应的”安全框架”中，这个”安全框架”才是真正的防火墙，这个框架的名字叫netfilter

netfilter才是防火墙真正的安全框架（framework），netfilter位于内核空间。

iptables其实是一个命令行工具，位于用户空间，我们用这个工具操作netfilter真正的框架。

然而，iptables效率很低，并且查找替换插入规则耗时长，iptables生效并且有先后顺序，只会生效前端的规则，所以规则的写入一般都是在头部插入。

#### 五表五链
在RedHat Linux 安全文档中找到了此图：
![upload successful](/images/pasted-103.png)

先说一下表和链的作用：

链：iptables通过使用链来控制网络通信规则，有
- PREROUTING
- INPUT
- FORWARD
- POSTROUTING
- OUTPUT

表：根据相同的规则形式或描述组成的一个组织，有

**filter:** *The default table, which is used when we're deciding if packets should be allowed to traverse specific chains (INPUT, FORWARD, OUTPUT).*

**nat:** *Used with packets that require a source or destination address/port translation. The table operates on the following chains: PREROUTING, INPUT, OUTPUT, and POSTROUTING.*

**mangle:** *Used with specialized packet alterations involving IP headers (such as MSS = Maximum Segment Size or TTL = Time to Live). The table supports the following chains: PREROUTING, INPUT, FORWARD, OUTPUT, and POSTROUTING.*

**raw:** *Used when we're disabling connection tracking (NOTRACK) on specific packets, mainly for stateless processing and performance optimization purposes. The table relates to the PREROUTING and OUTPUT chains.*

**security:** *Used for MAC when packets are subject to SELinux policy constraints. The table interacts with the INPUT, FORWARD, and OUTPUT chains.*

![upload successful](/images/pasted-104.png)

<font color="yellow">参考文献在本文最后哦</font>


#### iptables规则介绍

iptables规则插入使用，比较好的方式是创建自己专用链，把链插入到input，在本文中规则都是基于JUNSENINPUT链来操作。

在这里介绍的仅仅是脚本中使用到的，用作了解。

##### 规则查询

查询整个iptables规则命令：
```
$ iptables -nvL
```

查询某个链中规则：
```
$ iptables -S JUNSENINPUT 
```

清理规则，可选清理某个链：

```
$ iptables -F JUNSENINPUT
```
##### 链操作

定义一个新链

```
$ iptables -N JUSENINPUT
```

将新创建的链插入到基础链input中

```
$ iptables -I INPUT -j JUSENINPUT
```

##### 规则操作

插入指定ip

```
$ iptables -I JUSENINPUT -p tcp -s 192.168.1.1 -j ACCEPT
```

插入范围ip
```
$ iptables -I JUSENINPUT -p tcp -m iprange -s 192.168.1.1 --dst-range 192.168.2.1-192.168.2.10 -j ACCEPT
```

开放指定端口

```
$ iptables -I JUSENINPUT -p tcp --dport 2000:2100 -j ACCEPT
$ iptables -I JUSENINPUT -p tcp --sport 2000:2100 -j ACCEPT
```

### shell脚本操作iptables

需要注意一些空格，例如变量声明，=中间就不能有空格

```
#!/bin/bash

chain=`iptables -nvL | grep JUSENINPUT`

if ["$chain" = "" ];then
  iptables -N JUSENINPUT
  iptables -I INPUT -j JUSENINPUT
fi

function ex(){
  str=$1
  if [ "$1" == "exit" ];then
    exit 0
  fi
}

for i in {1..100}
do
  insertStr="iptables -I JUSENINPUT"
  for a in {1..6}
  do
    if [ $a -eq 1];then
      echo -e "请输入通信类型: \c"
      read p
      ex $p
      if [ "$p" != "*" -a "$p" != ""];then
        insertStr="$insertStr -p $p"
      fi
	fi
    ······省略其他参数获取
  done
  echo $insertStr
  echo ""
  $insertStr
done  
```

脚本中间用到了一些功能，例如创建方法，变量声明，for循环，if条件，等待用户输入，获取用户输入参数，在此记录一下。












附：[RedHat has a great doc about iptables](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/pdf/security_guide/red_hat_enterprise_linux-6-security_guide-en-us.pdf)