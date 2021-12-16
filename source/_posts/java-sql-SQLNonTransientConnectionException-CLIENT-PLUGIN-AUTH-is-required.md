title: 'java.sql.SQLNonTransientConnectionException: CLIENT_PLUGIN_AUTH is required'
author: Wtli
tags: []
categories: []
date: 2021-04-11 15:11:00
---
启动项目，数据库连接失败。

报错：CLIENT_PLUGIN_AUTH is required。

解决方案：更改mysql-connector-java版本，改为5.1.47（5.1.*）,根据自己的mysql版本找对应的插件版本。

<!--more-->


![upload successful](/images/pasted-95.png)

更改maven pom.xml文件，或者手动修改项目依赖。

启动失败，log内容如下：

```
java.sql.SQLNonTransientConnectionException: CLIENT_PLUGIN_AUTH is required
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:110)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:97)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:89)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:63)
	at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:73)
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:79)
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:836)
	at com.mysql.cj.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:456)
	at com.mysql.cj.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:246)
	at com.mysql.cj.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:197)
	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:156)
	at com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:218)
	at com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:150)
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1560)
	at com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1623)
	at com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2468)
Caused by: com.mysql.cj.exceptions.UnableToConnectException: CLIENT_PLUGIN_AUTH is required
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.cj.exceptions.ExceptionFactory.createException(ExceptionFactory.java:61)
	at com.mysql.cj.exceptions.ExceptionFactory.createException(ExceptionFactory.java:85)
	at com.mysql.cj.protocol.a.NativeAuthenticationProvider.connect(NativeAuthenticationProvider.java:114)
	at com.mysql.cj.protocol.a.NativeProtocol.connect(NativeProtocol.java:1342)
	at com.mysql.cj.NativeSession.connect(NativeSession.java:157)
	at com.mysql.cj.jdbc.ConnectionImpl.connectOneTryOnly(ConnectionImpl.java:956)
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:826)
	... 9 common frames omitted
```

问题：

数据库版本和数据库驱动对应问题。

mysql版本：5.1.73-community

maven驱动版本：mysql-connector-java-8.0.20.jar:8.0.20

切换mysql-connector-java版本到5.1.47。重新maven clean && maven compile，运行。

还是同样问题：

```
Caused by: com.mysql.cj.exceptions.UnableToConnectException: CLIENT_PLUGIN_AUTH is required
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:1.8.0_131]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62) ~[na:1.8.0_131]
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[na:1.8.0_131]
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423) ~[na:1.8.0_131]
	at com.mysql.cj.exceptions.ExceptionFactory.createException(ExceptionFactory.java:61) ~[mysql-connector-java-8.0.20.jar:8.0.20]
```

虽然改了版本，重新编译了，但是还是使用的旧的版本，继续寻找问题。

清理idea缓存（File –> Invalidate Caches / Restart）没用。

目前的问题已经成为修改maven文件（pom.xml），没有作用。

猜测原因：
1、因为项目是模块化的，依赖另外的支撑项目。
2、项目从其他地方copy过来的，保存了idea的配置文件。

尝试：

1. 在File –> Project Structure 重新配置项目的依赖，重新添加本地模块化项目。
2. 删除idea配置的iml文件（不建议使用）。在删除iml文件的项目中运行mvn idea:module，重新生成iml文件。
来来回回折腾半天，最终完美解决方案：

手动配置项目依赖，maven修改了版本号，但是依赖不变，可能是有之前高版本的依赖，再次添加则他会自动选择。

File –> Project Structure –> Modules –> your project –> Dependencies。在目录里手动配置项目的依赖。很烦(╯﹏╰)。

