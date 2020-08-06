title: SpringBoot集成Redis+Lettuce
author: Wtli
tags:
  - SpringBoot
  - Redis
categories:
  - 集成
date: 2020-08
---
SpringBoot对Redis的集成，包括在gRPC双向流传输中使用Redis存储。
<!-- more -->

### Redis启动命令

redis运行
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
### 项目配置

#### 在Pom.xml文件中导入Redis包

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
   <version>2.3.1.RELEASE</version>
</dependency>
```

<font color = gray>注意版本的对应，这里使用redis2.3.1和spring-boot2.3.1版本</font>
#### 导入lettuce连接工具包

```
 <dependency>
     <groupId>io.lettuce</groupId>
     <artifactId>lettuce-core</artifactId>
     <version>5.3.1.RELEASE</version>
 </dependency>
```
#### 导入lettuce连接池配置包

连接池是用来管理redis连接的，防止后期连接过多造成资源浪费。
```
<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-pool2</artifactId>
</dependency>
```

#### 在项目中添加Redis配置
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
<font color = gray>注意application.yml文件中spring配置内的子配置</font>
### 简单连接运行

```
RedisClient client = RedisClient.create("redis://localhost");          (1)
StatefulRedisConnection<String, String> connection = client.connect(); (2)
RedisCommands<String, String> commands = connection.sync();            (3)
String value = commands.get("foo");                                    (4)
...
connection.close();                                                    (5)
client.shutdown();
```
1、创建RedisClient实例并提供一个指向本地主机端口6379（默认端口）的Redis URI。  
2、打开Redis Standalone连接。从初始化使用端点RedisClient  
3、获取用于同步执行的命令API。Lettuce也支持异步和反应式执行模型。  
4、发出GET命令以获取密钥foo。  
5、完成后关闭连接。这通常发生在应用程序的最后。连接被设计为长寿命的。  
6、关闭客户端实例以释放线程和资源。这通常发生在应用程序的最后。
<font color = gray >Redis连接被设计为长期存在并且是线程安全的，如果连接丢失，将重新连接直到close()被调用。成功重新连接后，将（重新）发送尚未超时的未决命令。</font>
#### RedisURI

RedisURI包含主机/端口，并且可以携带身份验证/数据库详细信息。连接成功后，您将获得身份验证，然后选择数据库。这在连接断开后重新建立连接之后也适用。

```
还可以从URI字符串创建Redis URI。支持的格式有：
- redis://[password@]host[:port][/databaseNumber] 纯文本Redis连接
- rediss://[password@]host[:port][/databaseNumber] SSL Redis连接
- redis-sentinel://[password@]host[:port][,host2[:port2]][/databaseNumber]#sentinelMasterId 使用Redis Sentinel
- redis-socket:///path/to/socket Unix域套接字连接到Redis
```
### 深入配置Redis-Lettuce
#### 配置Lettuce连接器
以下示例显示了如何创建新的Lettuce连接工厂：
```
@Configuration
class AppConfig {

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
      return new LettuceConnectionFactory(new RedisStandaloneConfiguration("server", 6379));
    }
}
```
默认情况下，LettuceConnection由LettuceConnectionFactory共享创建的所有实例对于所有非阻塞和非事务性操作共享相同的线程安全本机连接。  
**要每次使用专用连接，请将设置shareNativeConnection为false。**

#### 通过RedisTemplate处理对象
配置后，该模板是线程安全的，并且可以在多个实例之间重用。

