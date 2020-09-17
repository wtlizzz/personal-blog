title: SpringBoot集成Redis+Lettuce
author: Wtli
tags:
  - SpringBoot
  - Redis
categories:
  - 后端
  - ''
date: 2020-08-04 09:21:00
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
1. 创建RedisClient实例并提供一个指向本地主机端口6379（默认端口）的Redis URI。  
2. 打开Redis Standalone连接。从初始化使用端点RedisClient  
3. 获取用于同步执行的命令API。Lettuce也支持异步和反应式执行模型。  
4. 发出GET命令以获取密钥foo。  
5. 完成后关闭连接。这通常发生在应用程序的最后。连接被设计为长寿命的。  
6. 关闭客户端实例以释放线程和资源。这通常发生在应用程序的最后。
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

在配置文件中配置
```

@Configuration
@Slf4j
public class RedisConfig {
	//获取配置参数
    @Value("${spring.redis.host}")
    private String redisHostName;
    @Value("${spring.redis.password}")
    private String redisPwd;
    @Value("${spring.redis.port}")
    private int redisPort;
    @Value("${spring.redis.lettuce.pool.max-idle}")
    private int maxIdle;
    @Value("${spring.redis.lettuce.pool.min-idle}")
    private int minIdle;
    @Value("${spring.redis.lettuce.pool.max-active}")
    private int maxActive;
    @Value("${spring.redis.lettuce.pool.max-wait}")
    private Long maxWait;
    @Value("${spring.redis.timeout}")
    private Long timeOut;
    @Value("${spring.redis.lettuce.shutdown-timeout}")
    private Long shutdownTimeOut;
    ···

```

redis数据库中有16个分块，分别是db0-db15，相当于16个内部的数据库，内部的key和value不会互相干扰。~~通过setDatabase方法来选择数据库,详细配置RedisConnectionFactory连接池如下：
@Bean
public RedisConnectionFactory redisConnectionFactory() {
    //redis配置
    RedisStandaloneConfiguration rsc = new RedisStandaloneConfiguration();
    rsc.setDatabase(DB_INDEX);
    rsc.setHostName(redisHostName);
    rsc.setPort(redisPort);
    rsc.setPassword(redisPwd);
    //连接池配置
    GenericObjectPoolConfig<RedisConfig> genericObjectPoolConfig = new GenericObjectPoolConfig<RedisConfig>();
    genericObjectPoolConfig.setMaxIdle(maxIdle);
    genericObjectPoolConfig.setMinIdle(minIdle);
    genericObjectPoolConfig.setMaxWaitMillis(maxWait);
    genericObjectPoolConfig.setMaxTotal(maxActive);
    //redis客户端配置  
    LettucePoolingClientConfiguration.LettucePoolingClientConfigurationBuilder
                builder = LettucePoolingClientConfiguration.builder().
                commandTimeout(Duration.ofMillis(timeOut));
    builder.shutdownTimeout(Duration.ofMillis(shutdownTimeOut));
    builder.poolConfig(genericObjectPoolConfig);
    LettuceClientConfiguration lettuceClientConfiguration = builder.build();
        //根据配置和客户端配置创建连接
    LettuceConnectionFactory lettuceConnectionFactory = new
            LettuceConnectionFactory(rsc, lettuceClientConfiguration);
    lettuceConnectionFactory.afterPropertiesSet();
    return lettuceConnectionFactory;
}
通过连接池的配置~~，
  
  **目前SpringBoot能自动读取application配置文件中的参数，去进行设置，不再去自行配置。**
  
  来对RedisTemplate实现
```
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setValueSerializer(fastJsonRedisSerializer());
        template.setHashValueSerializer(fastJsonRedisSerializer());
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setConnectionFactory(factory);
        return template;
    }
```

之后在使用的时候就可以直接@Autowired
```
@Autowired
private RedisTemplate redisTemplate;
```
### gRPC集成使用Redis存储

````
@GrpcService
public class TransformNetInfoImpl extends NetInfoServiceGrpc.NetInfoServiceImplBase {

    private Logger logger = Logger.getLogger(TransformNetInfoImpl.class.getName() + ".GRPC");

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public void transformNetInfoSimple(NetInfoRequest request, StreamObserver<NetInfoResponse> responseObserver) {

        String netInfo = "id:" + request.getId() + "\t"
                + "isTcp:" + request.getIsTcp() + "\t"
                + "src:" + request.getSrc() + "\t"
                + "dst:" + request.getDst() + "\t"
                + "sport:" + request.getSport() + "\t"
                + "dport:" + request.getDport() + "\t"
                + "mark:" + request.getMark();
        logger.info("Receive Net Info:" + netInfo);
        redisTemplate.opsForValue().set(request.getId(), netInfo);
        NetInfoResponse reply = NetInfoResponse.newBuilder().setAcceptInfo(("I got id:" + request.getId())).build();
        responseObserver.onNext(reply);
        responseObserver.onCompleted();
        logger.info("Message from gRPC-Client:" + request.getId());
    }

    @Override
    public StreamObserver<NetInfoRequest> transformNetInfo(StreamObserver<NetInfoResponse> responseObserver) {
        return new StreamObserver<NetInfoRequest>() {

            @Override
            public void onNext(NetInfoRequest request) {
                String netInfo = "id:" + request.getId() + "\t"
                        + "isTcp:" + request.getIsTcp() + "\t"
                        + "src:" + request.getSrc() + "\t"
                        + "dst:" + request.getDst() + "\t"
                        + "sport:" + request.getSport() + "\t"
                        + "dport:" + request.getDport() + "\t"
                        + "mark:" + request.getMark();
                logger.info("Receive Net Info:" + netInfo);
                redisTemplate.opsForValue().set(request.getId(), netInfo);
                NetInfoResponse response = NetInfoResponse.newBuilder().setAcceptInfo("I Get Id : " + request.getId()).build();
                responseObserver.onNext(response);
            }

            @Override
            public void onError(Throwable t) {
                logger.warning("Stupid TransformNetInfoImpl.transformNetInfo error : " + t.getMessage());
            }

            @Override
            public void onCompleted() {
                SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                responseObserver.onCompleted();
                logger.info("Receive TransformNetInfoImpl.transformNetInfo onCompleted Time is : " + df.format(new Date()));
            }
        };
    }
}
````