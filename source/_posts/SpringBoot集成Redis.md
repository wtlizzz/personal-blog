title: SpringBoot集成Redis
author: Wtli
tags:
  - SpringBoot
  - Redis
categories:
  - 集成
date: 2020-08-04 09:21:00
---
SpringBoot对Redis的集成，包括在gRPC双向流传输中使用Redis存储。
<!-- more -->

### Redis启动命令

进入到redis根目录，运行
```
$ redis-server
```
新建终端在redis根目录进入redis，运行
```
$ redis-cli
```
基本使用命令
```
$ get key
$ set key (value)
$ exists key
$ expire key  #设置时效
$ persist key	#取消设置的时效
```


### 在Pom.xml文件中导入Redis包

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
   <version>2.3.1.RELEASE</version>
</dependency>
```

***注意版本的对应，这里使用redis2.3.1和spring-boot2.3.1版本***

### 导入lettuce连接工具包

```
 <dependency>
     <groupId>io.lettuce</groupId>
     <artifactId>lettuce-core</artifactId>
     <version>5.3.1.RELEASE</version>
 </dependency>
```
### 导入lettuce连接池配置包

```
<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-pool2</artifactId>
</dependency>
```

### 在项目中添加Redis配置
```
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    password:
    lettuce:
      pool:
        max-active: 8 #连接池最大连接数量
        max-wait: -1  #
        max-idle: 8   #连接池中最大空闲连接，默认8
        min-idle: 0   #连接池最小空闲连接，默认0
```
***注意是在spring的配置内***

### 简单连接运行

```
public static void main(String[] args) throws Exception {
    RedisClient client = RedisClient.create(RedisURI.create("127.0.0.1", 6379));
    GenericObjectPool<StatefulRedisConnection<String, String>> pool = ConnectionPoolSupport.createGenericObjectPool(() -> client.connect(), new GenericObjectPoolConfig<>());
    StatefulRedisConnection<String, String> connection = pool.borrowObject();
    RedisCommands<String, String> commands = connection.sync();
    commands.multi();
    commands.set("name", "lewwt");
    commands.exec();
    pool.close();
    client.shutdown();
}
```

