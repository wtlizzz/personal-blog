title: redis常用命令
author: Wtli
tags:
  - redis
categories:
  - 后端
date: 2020-08-26 09:19:00
---
记录常用的redis命令，方便直接对数据进行操作。
<!-- more -->

启动：
```
redis-server
```
终端进入：
```
redis-cli
```
存储
```
$ SET key value
```
获取
```
$ GET key
```
删除
```
$ DEL key
```
查询当前数据库key的数量
```
$ DBSIZE
```
获取多个
```
$ MGET key1 key2
```
选择数据库db
```
$ SELECT INDEX
```
删除当前db
```
$ FLUSHDB
```
删除所有数据
```
$ FLUSHALL
```