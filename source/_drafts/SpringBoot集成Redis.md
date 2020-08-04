title: SpringBoot集成Redis
author: Wtli
date: 2020-08-04 09:21:35
tags:
---
SpringBoot对Redis的集成，包括在gRPC双向流传输中使用Redis存储。

### 在Pom.xml文件中导入Redis包

<!-- more -->
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
   <version>2.3.1.RELEASE</version>
</dependency>
```

***注意版本的对应，这里使用redis2.3.1和spring-boot2.3.1版本***

