title: Shell脚本基本写法
author: Wtli
tags:
  - Shell
categories:
  - Docker
  - ''
date: 2020-09-29 16:55:00
---
学习Shell脚本，用来写Dockerfile。
<!--more-->

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。

Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

### echo

**echo 命令用于向窗口输出文本。**

新建一个test.sh文件,并进行添加 echo "Hello World !"，

```
# touch test.sh
```

更改mac权限，否则不能正常运行

```
# chmod 777 test.sh
```

运行脚本文件
```
(base) LdeMacBook-Pro:docker lli$ ./test.sh
Hello World!
```