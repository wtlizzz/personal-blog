title: IDEA maven compile太慢
date: 2020-08
categories:
    - 编译器
tags:
    - maven  
    - IDEA
---

添加Ali镜像解决，IDEA maven compile太慢
<!--more-->

在此记录一下解决方案：

有的人是单纯配置的maven，在    ~/.m2/setting.xml修改，找到<mirrors>标签处增加
```
<mirror>

      <id>alimaven</id>

      <name>aliyun maven</name>

                  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>

      <mirrorOf>central</mirrorOf>       

</mirror>
```

**********************切记不要在注释里面添加，直接在<miorrs>标签下添加***********************

但是我是直接在IDEA里面配置的maven，安装的maven插件，这样的话在～/.m2目录中没有setting文件。查了半天原来是在

/Applications/IntelliJ IDEA.app/Contents/plugins/maven/lib/maven3/conf/settings.xml

这个目录中，这样就能够使在IDEA中的maven插件，加速了。
