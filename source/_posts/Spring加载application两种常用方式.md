title: Spring加载application两种常用方式
author: Wtli
tags:
  - Spring
categories:
  - 后端
  - ''
date: 2020-08-25 09:04:00
---
常用的两种方式：  
```
@Value("${}") 
```
```
@ConfigurationProperties(prefix = "")
```
<!-- more -->

第一种实例：

```
@Value("${spring.redis.host}")
private String host;
```

第二种实例：
```
@Configuration
@Slf4j
@ConfigurationProperties(prefix = "spring.redis")
public class RedisConfig {

    private String host;
    private String port;
    private String database;
    
    public String gethost() {
        return host;
    }

    public void sethost(String host) {
        this.host = host;
    }
    public String getPort() {
        return port;
    }

    public void setPort(String port) {
        this.port = port;
    }

    public String getDatabase() {
        return database;
    }

    public void setDatabase(String database) {
        this.database = database;
    }
}
```
**在属性处需要加上get/set才有效**

使用lombok则能更方便使用,仅需要在类前加上@Data注解。
```
@Configuration
@Slf4j
@Data
@ConfigurationProperties(prefix = "spring.redis")
public class RedisConfig {
    
    private String host;
    private String port;
    private String database;
}
```