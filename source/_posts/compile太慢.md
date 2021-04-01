title: maven入门
categories:
  - 后端
  - ''
tags:
  - maven
  - IDEA
date: 2020-08-03 12:51:00
---
今天做maven项目模块化，依赖其他项目需要install一下重新打jar包，学识太浅耽误了几分钟，在此学习下maven常用命令。  

<!--more-->

Idea中maven的功能大部分就这几个：  
❀ clean  
❀ validate  
❀ compile  
❀ test  
❀ package  
❀ verify  
❀ install  
❀ site  
❀ deploy  

### 常用命令介绍

```
$ maven clean ：清除以前编译的代码，删除target目录和相关内容删除
```
![upload successful](/images/pasted-31.png)

```
$ maven validate 
```
```
$ maven compile ：编译项目主目录下边的代码（main下的代码）–下载main相关代码依赖的外部资源
```
```
$ maven test ：编译项目主目录下边的test代码（编译test之前一定编译main代码，保证main正常编译成功）–下载test依赖的外部资源 前提需要执行mvn compile（若不主动执行，命令会自动执行mvn compile）
```
```
$ maven package 
```
```
$ maven verify 
```
```
$ maven install ：把编译好的class文件和下载的jar都打成一个完整的*.jar文件，直接使用jar包可以进行部署
```
```
$ maven site : 生成项目报告，站点，发布站点。
```
![upload successful](/images/pasted-32.png)

```
$ maven deploy 
```
![upload successful](/images/pasted-33.png)


### IDEA maven compile太慢
添加Ali镜像解决，IDEA maven compile太慢
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