title: SpringBoot AOP集成监听日志
author: Wtli
categories:
  - 后端
  - ''
tags:
  - SpringBoot
  - AOP
  - 日志
date: 2020-08-07 09:55:00
---
### 什么是AOP
    AOP（Aspect Oriented Programming），即面向切面编程。
   <!--more-->
    OOP（面向对象编程）通过的是继承、封装和多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。OOP从纵向上区分出一个个的类来，而AOP则从横向上向对象中加入特定的代码。AOP使OOP由原来的二维变为三维了，由平面变成立体了。
### 添加依赖
```
<!-- aop 依赖 -->
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<!-- 用于日志切面中，以 json 格式打印出入参 -->
<dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.8.5</version>
</dependency>
```
### AOP概念

1. 切面（Aspect）: 切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。比如事务管理是一个切面，权限管理也是一个切面。。
2. 通知（Advice）: 通知定义了切面是什么以及何时使用。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。  
	Spring切面可以应用5种类型的通知：
    - 前置通知（Before）：在目标方法被调用之前调用通知功能
    - 后置通知（After）：在目标方法完成之后调用通知，不关心方法的输出是什么。是“返回通知”和“异常通知”的并集。
    - 返回通知（After-returning）：在目标方法成功执行之后调用通知
    - 异常通知（After-throwing）：在目标方法抛出异常后调用通知   
    - 环绕通知（Around）通知包裹了被通知的方法，可同时定义前置通知和后置通知。
3. 切点（PointCut）: 切点定义了在何处工作，也就是真正被切入的地方，也就是在哪个方法应用通知。切点的定义会匹配通知所有要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。
4. 连接点（join point）: 连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时，抛出异常时，甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。
5. 引入（Introduction）：引入让一个切面可以声明被通知的对象实现了任何他们没有真正实现的额外接口，而且为这些对象提供接口的实现。    
<font color = gray >引入允许我们向现有的类添加新方法或属性。这个新方法和实例变量就可以被引入到现有的类中，从而可以再无需修改这些现有的类的情况下，让它们具有新的行为和状态。</font>
6. 织入（Weaving）: 织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以织入。
    - 编译器：切面在目标类编译时被织入。这种方式需要特殊的编译器。
    - 类加载期：切面在目标类被引入应用之前增强该目标类的字节码。
    - 运行期：切面在应用运行的某个时刻被织入。


### 配置代码

**正常配置**
```
import com.google.gson.Gson;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;

@Aspect
@Component
public class WebLogAspect {

    private final static Logger logger = LoggerFactory.getLogger(WebLogAspect.class);

    /** 切入点 */
    @Pointcut("execution(public * com.aircas.monitor.controller..*.*(..))")
    public void webLog() {}

    /**
     * 在切点之前织入
     * @param joinPoint
     * @throws Throwable
     */
    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
        // 开始打印请求日志
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        // 打印请求相关参数
        logger.info("========================================== Start ==========================================");
        // 打印请求 url
        logger.info("URL            : {}", request.getRequestURL().toString());
        // 打印 Http method
        logger.info("HTTP Method    : {}", request.getMethod());
        // 打印调用 controller 的全路径以及执行方法
        logger.info("Class Method   : {}.{}", joinPoint.getSignature().getDeclaringTypeName(), joinPoint.getSignature().getName());
        // 打印请求的 IP
        logger.info("IP             : {}", request.getRemoteAddr());
        // 打印请求入参
        logger.info("Request Args   : {}", new Gson().toJson(joinPoint.getArgs()));
    }

    /**
     * 在切点之后织入
     * @throws Throwable
     */
    @After("webLog()")
    public void doAfter() throws Throwable {
        logger.info("=========================================== End ===========================================");
        // 每个请求之间空一行
        logger.info("");
    }

    /**
     * 环绕
     * @param proceedingJoinPoint
     * @return
     * @throws Throwable
     */
    @Around("webLog()")
    public Object doAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        Object result = proceedingJoinPoint.proceed();
        // 打印出参
        logger.info("Response Args  : {}", new Gson().toJson(result));
        // 执行耗时
        logger.info("Time-Consuming : {} ms", System.currentTimeMillis() - startTime);
        return result;
    }

}
```

** maven项目模块化配置 **  
通过自定义注解，在其他方法中加入注解即能使用AOP日志。
```
$ 在AOP配置文件中将切点配置为下

/** 以 @Log 注解下定义的所有请求为切入点 */
@Pointcut("@annotation(com.aircas.aspect.aop.log.Log)")

$ 自定义注解@Log文件如下

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface  Log {
	String value() default "";
}
```

### 运行结果
![aWAOX9.png](https://s1.ax1x.com/2020/08/07/aWAOX9.png)