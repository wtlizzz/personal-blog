title: Linux配置服务开机启动
author: Wtli
tags:
  - linux
categories:
  - 后端
date: 2021-01-28 15:03:00
---
本文章讲述如何把自己的服务添加到系统服务中（可以直接使用systemctl或者service），并设置开机启动。
<!--more-->

CentOS7的服务systemctl脚本存放在:/usr/lib/systemd/,有系统（system）和用户（user）之分,
需要开机不登陆就能运行的程序，存在系统服务里，即：/usr/lib/systemd/system目录下.

需要自己写一个脚本，添加到目录中，就能够直接使用systemctl命令。

CentOS7的每一个服务以.service结尾，一般会分为3部分：\[Unit]、\[Service]和\[Install]

以tomcat为例：

```
#vim /usr/lib/systemd/system/tomcat.service
 
[Unit]
Description=java tomcat project
After=tomcat.service

[Service]
Type=forking
User=users
Group=users
PIDFile=/usr/local/tomcat/tomcat.pid
ExecStart=/usr/local/tomcat/bin/startup.sh
ExecReload=
ExecStop=/usr/local/tomcat/bin/shutdown.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```





**\[Unit]**

部分主要是对这个服务的说明，内容包括Description和After，Description 用于描述服务，After用于描述服务类别

**\[Service]**

部分是服务的关键，是服务的一些具体运行参数的设置.
Type=forking是后台运行的形式，
User=users是设置服务运行的用户,
Group=users是设置服务运行的用户组,
PIDFile为存放PID的文件路径，
ExecStart为服务的具体运行命令,
ExecReload为重启命令，
ExecStop为停止命令，
PrivateTmp=True表示给服务分配独立的临时空间
Environment设置环境变量
\[Service]
注意：\[Service]部分的启动、重启、停止命令全部要求使用绝对路径，使用相对路径则会报错！


**配置SpringBoot项目（直接运行jar包）**
```
#vim /usr/lib/systemd/system/your_project.service
 

[Unit]
Description=aircas-monitor project
After=aircas.service

[Service]
Type=forking
User=root
ExecStart=/bin/sh -c 'java -jar /aircas/aircas-monitor-0.1.jar >/dev/null 2>&1 &'
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID

[Install]
WantedBy=multi-user.target
```

- /bin/sh -c
- MAINPID是当前程序的线程id

添加开机启动，直接运行
```
systemctl enable your_project.service
```

～～～～～～～以上亲测可行～～～～～～～

亦可以通过springboot进行配置


**SpringBoot项目配置运行**

增加\<executable\>true\<\/executable\>配置。

Please note that since Spring Boot 1.3.0.M1, you are able to build fully executable jars using Maven

在SpringBoot1.3.0之后，能够打包整个运行程序。

The fully executable jar contains an extra script at the front of the file, which allows you to just symlink your Spring Boot jar to init.d or use a systemd script.

能够通过配置service来在服务器中运行。


```
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <executable>true</executable>
                </configuration>
            </plugin>
```

init.d 配置:

```
命令：
$ ln -s /var/yourapp/yourapp.jar /etc/init.d/yourapp
例子：
$ ln -s aircas-monitor-0.1.jar /etc/init.d/aircas
```
运行的时候就能够直接这样运行：
```
$ /etc/init.d/yourapp start|stop|restart

或

$ service aircas-monitor start

```