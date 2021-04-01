title: Mysql冷知识
author: Wtli
tags:
  - mysql
categories:
  - 后端
date: 2021-02-26 10:35:00
---
目录：
1. date类型字段在mysql中自动填充。

<!--more-->

#### date在mysql中自动填充

在mybatis中添加数据，让date字段和id一样，没有的话会自动填充当前时间。

- 需要将字段date字段类型设置为timestamp类型
- 设置不是null
- 设置默认值为CURRENT_TIMESTAMP


![upload successful](/images/pasted-68.png)


如果设置了CURRENT_TIMESTAMP为默认值，勾选了根据当前时间更新,表示每次更新这条数据的时候，该字段都会更新成当前时间，**其他时间字段在更新的时候此字段也会更新。**
