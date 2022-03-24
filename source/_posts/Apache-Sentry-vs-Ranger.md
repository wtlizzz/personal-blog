title: Apache Sentry 对比 Ranger
author: Wtli
date: 2022-03-17 14:11:24
tags:
---
Apache Ranger 与 Apache Sentry 功能重叠，都是处理授权和策略管理。

<div style="display: flex;flex-direction: row;align-items: center">
	<img src="/images/pasted-111.png" width=""/>
	<font size=100px>VS</font>
	<img src="/images/pasted-110.png" width=""/> 
</div>

当要做大数据生态的角色授权和用户细粒度划分时，不免会被这两个组件困惑，在大数据生态持续更新的今天，再来详细的分析两个组件的优劣，以及目前更应该使用哪一种。


<!-- more -->

##### 背景

在大数据生态中，一直都是三分天下，不收费的Hadoop版本主要有三个，分别是：
- Apache（最原始的版本，所有发行版均基于这个版本进行改进）
- Cloudera版本（Cloudera’s Distribution Including Apache Hadoop，简称CDH）
- Hortonworks版本(Hortonworks Data Platform，简称HDP）

但是在2018年10月初，Cloudera收购了Hortonworks公司，所以两者合并，在产品上将CDH与HDP合并成为CDP（Cloudera Data Platform），并将它们两者之间的技术进行了淘汰+融合+互补+共享。

![](/images/pasted-112.png)

<font color=grey>
另外：CDH免费版最高版本提供到了6.3.2，从6.3.3开始不提供免费版
</font>


##### Sentry 和 Ranger

Sentry是一个基于角色的粒度授权模块，提供了对Hadoop集群上经过身份验证的用户提供了控制和强制访问数据或数据特权的能力。它可以和Hive/Hcatalog、Apache Solr 和Cloudera Impala等集成，未来可以扩展到其他Hadoop生态系统组件，如HDFS和HBase。


Ranger的中文释义是“园林管理员”。正如其名，Apache Ranger很好的承担了Hadoop这个大园林的管理员职责。Ranger提供了一个集中式的安全管理框架，用户可以通过操作Ranger控制台来配置各种策略，从而实现对Hadoop生态组件如HDFS、Hive、HBase、Yarn等进行细粒度的数据访问控制。

##### 架构比较

Sentry架构：

通过Sentry Client，来收集生态中的各个组件信息，并传递到Sentry Server中，再与Sentry Store进行交互。

![upload successful](/images/pasted-113.png)

下图是Sentry在namenode中的架构


<div style="display: flex;flex-direction: row;align-items: center">
	<img src="/images/pasted-115.png" width="50%" height="300px"/>
	<img src="/images/pasted-114.png" width="50%" height="300px"/> 
</div>

Sentry HA(High Availablity)高可用性架构：

与HMS（Hive Metastore）更新同步：指定一个节点处理HMS通知日志。它还提供了一种通过引用配置单元对象名称将哨兵权限状态与特定HMS通知ID更改关联的方法。  
使用ZooKeeper进行Leader-Worker模式。  
使用Curator框架实现，第一个声明节点的守护进程将成为Leader。

![upload successful](/images/pasted-116.png)


Renger架构：

首先看核心，和Sentry一致，都是有一个Client，如下图所示指的是Ranger Plugin，相当于在Hadoop生态组件中的插件。

然后Ranger的核心主要包括Ranger Audit Server，Ranger Policy Server， Ranger Administration Protal。

架构列表：
- Ranger Admin Server/Portal
- Ranger Policy Server
- Ranger Plugins
- Ranger User/Group Sync
- Ranger Tag Sync
- Ranger Audit Server

![upload successful](/images/pasted-118.png)

Ranger Admin Server/Portal

- 安全管理的中央界面
- 管理员用户可以
	- 定义存储库
	- 创建和更新策略 
	- 管理 Ranger 用户/组
	- 定义审计策略
	- 查看审计活动 
- 它运行嵌入式 Tomcat 服务器
- 提供 Ranger API 

Ranger Policy Server

- 允许管理员用户定义/更新策略详细信息
- 允许管理员用户指定哪些用户是委托管理员，他们有权修改策略
- 策略可以划分为不同的安全区域
	- 一种资源只能分配给一个安全区域
	- 如果资源匹配，则仅检查定义区域中的策略
	- 如果没有资源匹配，将使用默认区域下的策略（没有名称） 
- 支持允许和拒绝策略
	- 拒绝将在津贴前检查 
- 策略可以应用于用户或组级别 

Ranger User/Group Sync

- 用于拉取用户和组的同步实用程序，它支持来自以下的用户/组源：
	- Unix
	- LDAP
	- AD
- 用户/组信息存储在 Ranger 管理策略数据库中并用于策略定义 

Ranger Plugins

- 要安装在 Hadoop 组件中的轻量级 Java 程序，例如 HDFS 或 Hive
- 定期从管理服务器拉入策略并在本地缓存
- 充当授权模块并根据安全策略评估用户请求
	- 如果未找到策略，将回退到 HDFS 请求的 HDFS ACL，所有其他组件的访问将被拒绝 
- 触发审计数据存储请求（对 HDFS 和 Solr） 

Ranger Audit Server

- 通过策略配置审核（如果此策略适用，用户指定是否需要启用审核）
- 审计默认存储在 HDFS 和 Solr 中
	- Solr 中的数据将用于在 Ranger 管理 UI 中显示审计数据
	- HDFS中的数据作为备份，不会被使用（据我了解）
	- 自 0.5 起不再支持 DB 中的审计 
- 支持审计日志汇总
	- 由于 Apache Ranger 0.5 
	- 定义时间段内仅时间戳不同的类似日志将汇总到单个审计条目，以避免大量审计日志
	- 默认为 5 秒 

Ranger Tag Sync

- 自 Apache Ranger 0.6
- 它将资源分类与访问授权分开
- 可以将一个标签策略应用于多个组件，只要资源附加了相同的标签
	- 有助于减少 Ranger 中所需的策略数量 
- 需要 Apache Atlas 来管理元数据（Hive DB/表、HDFS 路径、Kafka 主题和标签/分类等）
- 基于事件
	- Hive 等中的任何更改都会将事件发送到 Kafka 主题（ATLAS_HOOK），然后 Atlas 将获取更改
	- Atlas 中的任何更改都会将事件发送到 Kafka 主题（ATLAS_ENTITIES），然后 Ranger Tag Sync 将获取更改 
- 标签策略将在基于资源的策略之前进行评估 

策略运行机制

![upload successful](/images/pasted-119.png)


##### 选择Sentry还是Ranger

选择Ranger。


原因一：

Sentry目前在Apache中已经不再更新和维护了，已经放到Apache阁楼中。

Ranger目前还在Apache中正常更新维护。

原因二：

根据对Hadoop组件生态支持情况，选择Ranger：
![upload successful](/images/pasted-120.png)

原因三：

在Cloudera和Hortonworks合并之后，决定对Ranger标准化，并且在新Hadoop云环境CDP中使用Ranger。

![upload successful](/images/pasted-123.png)

原因四：

Sentry并不是很成熟。
![upload successful](/images/pasted-121.png)