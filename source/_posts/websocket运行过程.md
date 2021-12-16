title: websocket运行过程
author: Wtli
tags: []
categories: []
date: 2021-05-11 15:01:00
---
研究在springboot中使用websocket进行通信，websokcet在项目总整个启动过程。

本实验使用的websocket包是tomcat-embed-websocket-9.0.36.jar。

<!--more-->

### 入手点

websocket配置注解。
```
@ServerEndpoint
```
我们可以了解到websocket与endpoint这个概念关系密切。继续看ServerEndpoint注解中使用到的类。

![upload successful](/images/pasted-90.png)

与注解相关的类共有3个：

- WsSci
- WsServerContainer
- ServerEndpointExporter

### 分析注解类

一个一个看，先看WsSci类：
注：ServletContainerInitializers (SCIs)

#### WsSci

```
/**
 * Registers an interest in any class that is annotated with
 * {@link ServerEndpoint} so that Endpoint can be published via the WebSocket
 * server.
 */
@HandlesTypes({ServerEndpoint.class, ServerApplicationConfig.class,
        Endpoint.class})
public class WsSci implements ServletContainerInitializer {

		···

}
```
通过注释，能够看到这个类的作用：

能够把带有@ServerEndpoint注解的类，发布到websocket server中。

可以看出这个类在websocket中还是比较重要的。

#### WsServerContainer

进入第二个类：WsServerContainer
```
/**
 * Provides a per class loader (i.e. per web application) instance of a
 * ServerContainer. Web application wide defaults may be configured by setting
 * the following servlet context initialisation parameters to the desired
 * values.
 * <ul>
 * <li>{@link Constants#BINARY_BUFFER_SIZE_SERVLET_CONTEXT_INIT_PARAM}</li>
 * <li>{@link Constants#TEXT_BUFFER_SIZE_SERVLET_CONTEXT_INIT_PARAM}</li>
 * </ul>
 */
public class WsServerContainer extends WsWebSocketContainer
        implements ServerContainer {
 			
            ···       

} 
```
还是一样，先看注释了解这个类的作用：

提供ServerContainer的每个类装入器（即每个web应用程序）实例。可以通过将以下servlet上下文初始化参数设置为所需的值来配置Web应用程序范围的默认值。

在这个类的addEndpoint方法中，有获取ServerEndpoint注解，并使用注解中decoders、encoders的方法：
```
ServerEndpoint annotation = pojo.getAnnotation(ServerEndpoint.class);
sec = ServerEndpointConfig.Builder.create(pojo, path).
     decoders(Arrays.asList(annotation.decoders())).
     encoders(Arrays.asList(annotation.encoders())).
     subprotocols(Arrays.asList(annotation.subprotocols())).
     configurator(configurator).
     build();
addEndpoint(sec, fromAnnotatedPojo);
```

也就是说添加了websocket注解的类，是在这个类中进行解析的，解析完了调用addEndpoint将类添加到servlet中。

#### ServerEndpointExporter

下面来看第三个类：ServerEndpointExporter
```
/**
 * Detects beans of type {@link javax.websocket.server.ServerEndpointConfig} and registers
 * with the standard Java WebSocket runtime. Also detects beans annotated with
 * {@link ServerEndpoint} and registers them as well. Although not required, it is likely
 * annotated endpoints should have their {@code configurator} property set to
 * {@link SpringConfigurator}.
 *
 * <p>When this class is used, by declaring it in Spring configuration, it should be
 * possible to turn off a Servlet container's scan for WebSocket endpoints. This can be
 * done with the help of the {@code <absolute-ordering>} element in {@code web.xml}.
 *
 * @author Rossen Stoyanchev
 * @author Juergen Hoeller
 * @since 4.0
 * @see ServerEndpointRegistration
 * @see SpringConfigurator
 * @see ServletServerContainerFactoryBean
 */
public class ServerEndpointExporter extends WebApplicationObjectSupport
		implements InitializingBean, SmartInitializingSingleton {
        
        ···
        
}
```

简单的说，就是将ServerEndpointConfig与websokcet联系起来。

### 断点调试

分别在这三个类的初始化和主要方法加断点，进行调试。

WsSci类中一共两个方法，一个是重写ServletContainerInitializer的onStartup，一个是init。

WsServerContainer类构造函数中加断点，在addEndpoint方法也是很重要的。

ServerEndpointExporter类虽然说是关联配置类，但是为了了解整个初始化启动过程，也还是在registerEndpoints方法中加上断点看一下。

### 运行
首先进入断点的是 WsSci.init()方法，Springboot通过线程异步方式，通过FutureTask来启动。

upload successful

简单画个流程图。

![upload successful](/images/pasted-91.png)

### ServerEndpoint注册过程

在springboot的启动过程中说过，refreshContext(context);过程。在AbstractApplicationContext finishBeanFactoryInitialization(beanFactory);方法中进行registerEndpoints()注册Endpoints注解。

画图！

<center>
![upload successful](/images/pasted-92.png)
</center>
最终endpoints和它的配置信息放到了map中，即注册完成。
```
ExactPathMatch newMatch = new ExactPathMatch(sec, fromAnnotatedPojo);
ExactPathMatch oldMatch = configExactMatchMap.put(path, newMatch);
```

### 请求接入过程
ServerEndpoint注册完了，放到map中，怎么就注册完成了呢？接下来看看请求接入，map中的数据怎么使用的。

图文结合！

![upload successful](/images/pasted-93.png)

初始化结束位置是到了Http11Processor的构造函数，对request、buffer、filter都进行的初始化。
```
public Http11Processor(AbstractHttp11Protocol<?> protocol, Adapter adapter) {
       super(adapter);
       this.protocol = protocol;

       httpParser = new HttpParser(protocol.getRelaxedPathChars(),
               protocol.getRelaxedQueryChars());

       inputBuffer = new Http11InputBuffer(request, protocol.getMaxHttpHeaderSize(),
               protocol.getRejectIllegalHeader(), httpParser);
       request.setInputBuffer(inputBuffer);

       outputBuffer = new Http11OutputBuffer(response, protocol.getMaxHttpHeaderSize());
       response.setOutputBuffer(outputBuffer);

       // Create and add the identity filters.
       inputBuffer.addFilter(new IdentityInputFilter(protocol.getMaxSwallowSize()));
       outputBuffer.addFilter(new IdentityOutputFilter());

       // Create and add the chunked filters.
       inputBuffer.addFilter(new ChunkedInputFilter(protocol.getMaxTrailerSize(),
               protocol.getAllowedTrailerHeadersInternal(), protocol.getMaxExtensionSize(),
               protocol.getMaxSwallowSize()));
       outputBuffer.addFilter(new ChunkedOutputFilter());

       // Create and add the void filters.
       inputBuffer.addFilter(new VoidInputFilter());
       outputBuffer.addFilter(new VoidOutputFilter());

       // Create and add buffered input filter
       inputBuffer.addFilter(new BufferedInputFilter());

       // Create and add the gzip filters.
       //inputBuffer.addFilter(new GzipInputFilter());
       outputBuffer.addFilter(new GzipOutputFilter());

       pluggableFilterIndex = inputBuffer.getFilters().length;
   }
```

请求接收开始也是从Http11Processor类，从service()方法开始，主要的内容有如下几部分：

#### 判断http版本

判断http是0.9或者1.0或者1.1版本。另外也判断是否支持ssl（https）通信。
```
// Process the Protocol component of the request line
// Need to know if this is an HTTP 0.9 request before trying to
// parse headers.
prepareRequestProtocol();
                
    private void prepareRequestProtocol() {
        MessageBytes protocolMB = request.protocol();
        if (protocolMB.equals(Constants.HTTP_11)) {
            http09 = false;
            http11 = true;
            protocolMB.setString(Constants.HTTP_11);
        } else if (protocolMB.equals(Constants.HTTP_10)) {
            http09 = false;
            http11 = false;
            keepAlive = false;
            protocolMB.setString(Constants.HTTP_10);
        } else if (protocolMB.equals("")) {
            // HTTP/0.9
            http09 = true;
            http11 = false;
            keepAlive = false;
        } else {
            // Unsupported protocol
            http09 = false;
            http11 = false;
            // Send 505; Unsupported HTTP version
            response.setStatus(505);
            setErrorState(ErrorState.CLOSE_CLEAN, null);
            if (log.isDebugEnabled()) {
                log.debug(sm.getString("http11processor.request.prepare")+
                          " Unsupported HTTP version \""+protocolMB+"\"");
            }
        }
    }
```

#### 返回状态码

根据服务器状态，在此类中返回状态码，例如500、400.
```
if (protocol.isPaused()) {
	// 503 - Service unavailable
	response.setStatus(503);
	setErrorState(ErrorState.CLOSE_CLEAN, null);
}
               
// 400 - Bad Request
response.setStatus(400);
// 500 - Internal Server Error
response.setStatus(500);
```

#### 判断websocket

websocket的http-header中携带了Upgrade标识位，当携带了标识位，在此类中进行操作。
```
// Has an upgrade been requested?
if (isConnectionToken(request.getMimeHeaders(), "upgrade")) {
	// Check the protocol
	String requestedProtocol = request.getHeader("Upgrade");
}
```

#### 过滤器WsFilter

endpoints的加载过程中，最后是将encoder、decoder等配置放到了configExactMatchMap中，在过滤器WsFilter中调用。

![upload successful](/images/pasted-94.png)


#### UpgradeUtil处理websocket
UpgradeUtil类是专门处理websocekt-header数据的工具类。
两个主要功能：接收websocket请求头中的数据和添加response头部数据。

UpgradeUtil获取websokcet-header中的多个数据，数据类型如下。
```
// HTTP upgrade header names and values
public static final String HOST_HEADER_NAME = "Host";
public static final String UPGRADE_HEADER_NAME = "Upgrade";
public static final String UPGRADE_HEADER_VALUE = "websocket";
public static final String ORIGIN_HEADER_NAME = "Origin";
public static final String CONNECTION_HEADER_NAME = "Connection";
public static final String CONNECTION_HEADER_VALUE = "upgrade";
public static final String LOCATION_HEADER_NAME = "Location";
public static final String AUTHORIZATION_HEADER_NAME = "Authorization";
public static final String WWW_AUTHENTICATE_HEADER_NAME = "WWW-Authenticate";
public static final String WS_VERSION_HEADER_NAME = "Sec-WebSocket-Version";
public static final String WS_VERSION_HEADER_VALUE = "13";
public static final String WS_KEY_HEADER_NAME = "Sec-WebSocket-Key";
public static final String WS_PROTOCOL_HEADER_NAME = "Sec-WebSocket-Protocol";
public static final String WS_EXTENSIONS_HEADER_NAME = "Sec-WebSocket-Extensions";
```
添加response头部数据。
```
// If we got this far, all is good. Accept the connection.
resp.setHeader(Constants.UPGRADE_HEADER_NAME,
        Constants.UPGRADE_HEADER_VALUE);
resp.setHeader(Constants.CONNECTION_HEADER_NAME,
        Constants.CONNECTION_HEADER_VALUE);
resp.setHeader(HandshakeResponse.SEC_WEBSOCKET_ACCEPT,
        getWebSocketAccept(key));
```