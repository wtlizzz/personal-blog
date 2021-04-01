title: SpringBoot集成Swagger图形化显示Api
author: Wtli
tags:
  - SpringBoot
  - Swagger
categories: []
date: 2020-08-24 08:15:00
---
现在有机会重新系统学习一下Swagger。  
Swagger是在Api开发中很好用的工具，能将Api以图形化的形式显示。

<!-- MORE -->

Swagger主要包括2大部分，6小部分，分别是：

**开源部分**
- Swagger Codegen
- Swagger Editor
- Swagger UI

**Pro部分**
- SwaggerHub
- SwaggerHub Enterprise
- Swagger Inspector

### 简单使用

不多逼逼，直接SpringBoot集成，

- maven中添加
```
<!-- swagger -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```
- 添加配置文件
```
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("RestfulApi")
                .genericModelSubstitutes(ResponseEntity.class)
                .useDefaultResponseMessages(true)
                .forCodeGeneration(false)
                .select()
                .build()
                .apiInfo(apiInfo());

    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Aircas 接口文档")
                .version("1.0")
                .description("低空网络信息技术实验室")
                .build();
    }
}
```
- 增加controller配置 
```
@RestController
@RequestMapping("/api/netData")
@Api(tags = "获取网络请求模块")
public class NetDataHandler {

    @Autowired
    private RedisTemplate redisTemplate;

    @Logger
    @PostMapping(value = "/register")
    public ResponseEntity<Object> register(@RequestBody String umsAdminParam) {
        ArrayList<Object> arr = new ArrayList();
        for (int i = 10000; i < 10003; i++) {
            arr.add(SerializeUtil.unserialize((byte[]) redisTemplate.opsForValue().get(String.valueOf(i))));
        }
        return new ResponseEntity<>(arr, HttpStatus.OK);
    }

    @Log
    @ApiOperation(value = "根据id获取信息")
    @GetMapping("/{id}")
    public ResponseEntity<Object> get(@PathVariable String id) {
        return new ResponseEntity<>("I Get id:" + id, HttpStatus.OK);
    }

    @Log("新增机构")
    @PostMapping("/{id}")
    public ResponseEntity<Object> post(@PathVariable String id) {
        return new ResponseEntity<>("I Get id:" + id, HttpStatus.OK);
    }

}
```
- 实现图

![upload successful](/images/pasted-41.png)


### 进一步学习

#### 配置跨域会造成No Mapping
如果在项目中配置了WebMvcConfigurationSupport类，则会造成swagger的error。
解决方式，在WebMvcConfigurationSupport继承类中增加方法：

```
//配置swagger
@Override
protected void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/**").addResourceLocations(
                "classpath:/static/");
    registry.addResourceHandler("swagger-ui.html").addResourceLocations(
                "classpath:/META-INF/resources/");
    registry.addResourceHandler("/webjars/**").addResourceLocations(
                "classpath:/META-INF/resources/webjars/");
    super.addResourceHandlers(registry);
    }
```