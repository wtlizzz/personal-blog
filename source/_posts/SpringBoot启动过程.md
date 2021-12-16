title: SpringBoot启动过程
author: Wtli
tags:
  - 后端
categories: []
date: 2021-04-19 14:45:00
---
上一篇博文是分析websocket的启动过程，在研究启动过程就不得不结合SpringBoot的启动过程来进行分析。

本文是对SpringBoot的启动过程进行分析，代码比较多，一起看代码。

<!--more-->

目录是根据启动过程涉及的类进行编号。

#### MonitorApplication

项目名是MonitorApplication。

首先是MonitorApplication main，项目运行的起点。
```
public static void main(String[] args) throws Exception {
    SpringApplication.run(MonitorApplication.class, args);
    ioServer();
}
```

#### SpringApplication

SpringApplication.run方法，返回了一个new SpringApplication.run的实例。
```
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
	return new SpringApplication(primarySources).run(args);
}
```

#### run()

new SpringApplication(primarySources).run(args)方法创建了一个新的ApplicationContext上下文，换句话说就是加载了配置文件，在自定义配置下创建了一个运行环境。
```
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 */
public ConfigurableApplicationContext run(String... args) {
   	StopWatch stopWatch = new StopWatch();
	stopWatch.start();
	ConfigurableApplicationContext context = null;
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	configureHeadlessProperty();
	SpringApplicationRunListeners listeners = getRunListeners(args);
	listeners.starting();
	try {
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
		configureIgnoreBeanInfo(environment);
		Banner printedBanner = printBanner(environment);
		context = createApplicationContext();
		exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
		prepareContext(context, environment, listeners, applicationArguments, printedBanner);
		refreshContext(context);
		afterRefresh(context, applicationArguments);
		stopWatch.stop();
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
		}
		listeners.started(context);
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
		listeners.running(context);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	return context;
   
   }
```
#### createApplicationContext()

创建上下文Context，包括的主要内容：

创建SERVLET
在方法里出去前几行的启动初始化配置数据结构，配置环境变量。到了
```
context = createApplicationContext()
```
创建上下文的方法。

根据application的类型，来生成对应的contextClass，Spring5新增了一个REACTIVE_WEB，字面意思是响应式web，比较新的，网上也有相关框架–webflux的说明。
```
createApplicationContext():

    switch (this.webApplicationType) {
        case SERVLET:
            contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
            break;
        case REACTIVE:
            contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
            break;
        default:
            contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
    }
    return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
```

#### 创建beanFactory

在GenericApplicationContext类中创建beanFactory。
```
public GenericApplicationContext() {
	this.beanFactory = new DefaultListableBeanFactory();
}
```

#### refresh

上下文生成之后是准备刷新：
```
prepareContext(context, environment, listeners, applicationArguments, printedBanner);
refreshContext(context);
afterRefresh(context, applicationArguments);
```

简单说明一下三个方法的作用。

prepareContext：

- 设置Environment
- 初始化ApplicationContextInitializer
- 绑定SpringApplicationRunListeners
- 获取getBeanFactory

refreshContext：这个方法调用的AbstractApplicationContext类中refresh()。 ServletWebServerApplicationContext是在此方法中创建的。

- Refresh the underlying {@link ApplicationContext}。（refreshContext）

- prepareRefresh();（AbstractApplicationContext以下都是）

- prepareBeanFactory(beanFactory);// Prepare the bean factory for use in this context.

- postProcessBeanFactory(beanFactory);// Allows post-processing of the bean factory in context subclasses.

- invokeBeanFactoryPostProcessors(beanFactory);// Invoke factory processors registered as beans in the context.

- registerBeanPostProcessors(beanFactory);// Register bean processors that intercept bean creation.

- initMessageSource();// Initialize message source for this context.

- initApplicationEventMulticaster();// Initialize event multicaster for this context.

- onRefresh();// Initialize other special beans in specific context subclasses.

- registerListeners();// Check for listener beans and register them.

- finishBeanFactoryInitialization(beanFactory);// Instantiate all remaining (non-lazy-init) singletons.

- finishRefresh();// Last step: publish corresponding event.
```
ServletWebServerApplicationContext:

	@Override
protected void onRefresh() {
	super.onRefresh();
	try {
		createWebServer();
	}
	catch (Throwable ex) {
		throw new ApplicationContextException("Unable to start web server", ex);
	}
}
```

#### AbstractApplicationContext

下一步到了refreshContext的步骤。

调用的方法类是

ServletWebServerApplicationContext.refresh() ==> AbstractApplicationContext.refresh()

在方法中运行顺序如下：
```
// Prepare this context for refreshing.
prepareRefresh();

// Tell the subclass to refresh the internal bean factory.
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

// Prepare the bean factory for use in this context.
prepareBeanFactory(beanFactory);

try {
    // Allows post-processing of the bean factory in context subclasses.
    postProcessBeanFactory(beanFactory);

    // Invoke factory processors registered as beans in the context.
    invokeBeanFactoryPostProcessors(beanFactory);

    // Register bean processors that intercept bean creation.
    registerBeanPostProcessors(beanFactory);

    // Initialize message source for this context.
    initMessageSource();

    // Initialize event multicaster for this context.
    initApplicationEventMulticaster();

    // Initialize other special beans in specific context subclasses.
    onRefresh();

    // Check for listener beans and register them.
    registerListeners();

    // Instantiate all remaining (non-lazy-init) singletons.
    finishBeanFactoryInitialization(beanFactory);

    // Last step: publish corresponding event.
    finishRefresh();
}
```

#### ServletWebServerApplicationContext

AbstractApplicationContext类中refresh方法调用了onRefresh();在子类ServletWebServerApplicationContext中的方法，创建webServer。
```
@Override
protected void onRefresh() {
	super.onRefresh();
	try {
		createWebServer();
	}
	catch (Throwable ex) {
		throw new ApplicationContextException("Unable to start web server", ex);
	}
}
```

#### createWebServer()

创建webServer，主要过程是在BeanFactory中注册。
```
private void createWebServer() {
    WebServer webServer = this.webServer;
    ServletContext servletContext = getServletContext();
    if (webServer == null && servletContext == null) {
        ServletWebServerFactory factory = getWebServerFactory();
        this.webServer = factory.getWebServer(getSelfInitializer());
        getBeanFactory().registerSingleton("webServerGracefulShutdown",
			new WebServerGracefulShutdownLifecycle(this.webServer));
        getBeanFactory().registerSingleton("webServerStartStop",
			new WebServerStartStopLifecycle(this, this.webServer));
    }
    else if (servletContext != null) {
        try {
            getSelfInitializer().onStartup(servletContext);
        }
        catch (ServletException ex) {
            throw new ApplicationContextException("Cannot initialize servlet context", ex);
        }
    }
    initPropertySources();
}
```
启动过程到这里就算结束了，以上内容是简单的了解了一下启动的机制。

#### 附

##### SpringApplicationRunListeners

跑到这主要的逻辑就已经结束，下面是启动listener和runnable。这两个方法在我这个springboot web项目中没有跑到，应该是其他项目能够使用到。

listeners.started(context);
callRunners(context, applicationArguments);
在SpringApplicationRunListeners类中running方法：
```
void running(ConfigurableApplicationContext context) {
    for (SpringApplicationRunListener listener : this.listeners) {
        listener.running(context);
    }
}
```

##### EventPublishingRunListener
running：
```
@Override
public void running(ConfigurableApplicationContext context) {
    context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context));
    AvailabilityChangeEvent.publish(context, ReadinessState.ACCEPTING_TRAFFIC);
}
```