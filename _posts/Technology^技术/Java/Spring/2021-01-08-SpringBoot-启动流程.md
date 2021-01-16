---
title: SpringBoot-启动流程
author: Narule
date: 2021-01-08 20:10:00 +0800
categories: [Technology^技术, Java, Spring]
tags: [writing, Java, Spring]
---



# SpringBoot-启动流程

平时开发springboot项目的时候，一个SpringBootApplication注解加一个main方法就可以启动服务器运行起来（默认tomcat），看了下源码，这里讲下认为主要的流程

**主要流程如下**

0.启动main方法开始

1.**初始化配置**：通过类加载器，（loadFactories）读取classpath下所有的spring.factories配置文件，创建一些初始配置对象；通知监听者应用程序启动开始，创建环境对象environment，用于读取环境配置 如 application.yml

2.**创建应用程序上下文**-createApplicationContext，创建 bean工厂对象

3.**刷新上下文（启动核心）**
	3.1 配置工厂对象，包括上下文类加载器，对象发布处理器，beanFactoryPostProcessor
	3.2 注册并实例化bean工厂发布处理器，并且调用这些处理器，对包扫描解析(主要是class文件)
	3.3 注册并实例化bean发布处理器 beanPostProcessor
	3.4 初始化一些与上下文有特别关系的bean对象（创建tomcat服务器）
	3.5 实例化所有bean工厂缓存的bean对象（剩下的）
	3.6 发布通知-通知上下文刷新完成（启动tomcat服务器）

4.**通知监听者-启动程序完成**



启动中，大部分对象都是BeanFactory对象通过反射创建

SpringBoot的启动解析代码过多，下文是整体流程的部分主要代码

### 启动

启动程序：

```java
import org.springframework.boot.SpringApplication;//启动类
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication //启动必要注解
public class YourApplication {
	//运行main方法启动springboot
	public static void main(String[] args) {
		SpringApplication.run(YourApplication.class, args);//启动类静态run方法
	}
    
}
```

### 启动类

`org.springframework.boot.SpringApplication` 包含主流程方法

启动类在运行静态run方法的时候，是先创建一个SpringApplication对象，再运行对象的run方法，工厂初始配置在构造函数中完成，run方法定义运行总体流程

```java
// 静态方法 org.springframework.boot.SpringApplication.run(Class<?>[], String[])
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return new SpringApplication(primarySources).run(args);
}

// 构造方法
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    //.......... 
    //// 1.(loadFactories)读取classpath下所有的spring.factories配置文件 ////
    // 配置应用程序启动前的初始化对象
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class)); 
    // 配置应用程序启动前的监听器
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}

// 对象run方法 开始启动程序
public ConfigurableApplicationContext run(String... args) {
    //......
    // 通知监听者启动开始
    listeners.starting(); 
    try {
        // 创建应用程序环境 配置文件在此处读取(application.properties application.yml)
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        //// 2.创建应用程序上下文...此处创建了beanfactory ////
        context = createApplicationContext();
        //// 3.刷新上下文（spring启动核心） ////
        refreshContext(context);

        //// 4.启动完成通知...... ////
        listeners.started(context);
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



## 初始化配置

**springboot启动应用程序之前，会创建一些初始化对象和监听器**

这个操作在构造方法中完成，根据配置文件，创建`ApplicationContextInitializer.class`,`ApplicationListener.class`两个接口的实现类，至于具体创建那些类对象，根据下面的方法逻辑去做

`org.springframework.boot.SpringApplication.getSpringFactoriesInstances()`  ->
`org.springframework.core.io.support.SpringFactoriesLoader.loadFactoryNames()` ->
`org.springframework.core.io.support.SpringFactoriesLoader.loadSpringFactories()`->
`createSpringFactoriesInstances()`

```java
//构造方法中的初始化对象创建
setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class)); 
setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));

//看一下getSpringFactoriesInstances方法
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = getClassLoader();
    // 获取初始化类的类名
    Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 通过这些类名实例化对象
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}

// 读取配置方法
// 更详深层的代码在org.springframework.core.io.support.SpringFactoriesLoader.loadSpringFactories(ClassLoader)
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    String factoryTypeName = factoryType.getName();
    return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());   
}
// loadSpringFactories(classLoader)读取运行环境中所有META-INF/spring.factories配置
```

通过上面的方法，

spring-boot-2.2.8.RELEASE.jar/META-INF/spring.factories的文件中是这样，

```F#
# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.rsocket.context.RSocketPortInfoApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer
```

如果只读取这一个文件，`loadFactoryNames(ApplicationContextInitializer.class,classLoader)`读取返回的就是下面的数组:

```java
[org.springframework.context.ApplicationContextInitializer,
 org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,
 org.springframework.boot.context.ContextIdApplicationContextInitializer,
 org.springframework.boot.context.config.DelegatingApplicationContextInitializer,
 org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer]
```

通过` Class.forName(className)`获取这些类的Class，最后反射`newInstance`创建这些对象

**创建好这些对象后，启动监听器**

```java
listeners.starting();  // 这里也是一些调用操作
```

**读取application配置**

监听器启动之后，会读取application.properties 或者 application.yml文件

```java
ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments); //此处application.properties配置文件会被读取
```



## 创建应用上下文


初始化和配置好后，开始创建应用程序上下文，createApplicationContext ，关键的工厂BeanFactory就是此处创建，具体逻辑如下

```java
// 创建应用程序上下文
context = createApplicationContext();

protected ConfigurableApplicationContext createApplicationContext() {
    // 上下文创建的判断逻辑
    Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
        try {
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
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                "Unable create a default ApplicationContext, please specify an ApplicationContextClass", ex);
        }
    }
    return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}

public static final String DEFAULT_SERVLET_WEB_CONTEXT_CLASS = "org.springframework.boot."
			+ "web.servlet.context.AnnotationConfigServletWebServerApplicationContext";
// 默认是创建这个类
```

这里通过this.webApplicationType判断创建具体的应用上下文，也是反射创建对象，默认创建的是`org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext`对象，看一下这个类的基本信息

```java
public class AnnotationConfigServletWebServerApplicationContext extends ServletWebServerApplicationContext
		implements AnnotationConfigRegistry {
    // 构造方法
	public AnnotationConfigServletWebServerApplicationContext() {
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
}

```

**创建工厂对象**

此类继承了很多类，其中一个父类是`org.springframework.context.support.GenericApplicationContext`
jvm机制，创建对象的时候，先运行父类的构造方法，所以创建了beanFactory

```java
// 超级父类 GenericApplicationContext的构造方法
public GenericApplicationContext() {
    this.beanFactory = new DefaultListableBeanFactory();//创建工厂对象
}
```



## 刷新应用上下文

创建好上下文之后，开始刷新上下文，这里做了很多

工厂配置，bean处理器配置，类的扫描，解析，bean定义，bean类信息缓存，服务器创建，bean实例化，动态代理对象的创建等，

**spring中注册bean信息和实例化bean是两件事情。**

注册bean信息不是创建bean对象，是解析bean类获取详细信息，会创建BeanDefinition对象，携带bean类的字节码和方法等信息，把BeanDefinition对象注册保存到工厂BeanDefinitionMap中。

工厂实例化bean时直接BeanDefinitionMap.get(beanName) 获取bean的字节码信息，通过反射创建对象，然后将bean对象保存到singletonObjects中。

```java
refreshContext(context); //刷新上下文
```

默认实际对应的是`org.springframework.context.support.AbstractApplicationContext`类的`refresh()`方法

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        //......
        // 3.1配置工厂对象
        prepareBeanFactory(beanFactory);
        try {       
            postProcessBeanFactory(beanFactory);
            
            // 3.2注册并实例化bean工厂处理器,并调用他们
            invokeBeanFactoryPostProcessors(beanFactory);
            
            // 3.3注册并实例化bean处理器
            registerBeanPostProcessors(beanFactory);
            
            // 3.4 初始化一些与上下文有特别关系的bean对象（创建tomcat）
            onRefresh();
            
            // 3.5 实例化所有bean工厂缓存的bean对象（剩下的）.
            finishBeanFactoryInitialization(beanFactory);
            
            // 3.6 发布通知-通知上下文刷新完成（包括启动tomcat）
            finishRefresh();
        }
        catch (BeansException ex) {// ......Propagate exception to caller.
            throw ex;
        }
        finally {// ......
            resetCommonCaches();
        }
    }
}
```



### 配置工厂对象，包括上下文类加载器，bean工厂发布处理器

工厂创建好后，首先配置的是类加载器，然后是一些对象发布处理器（拦截器）

```java
//// 3.1配置工厂对象 
prepareBeanFactory(beanFactory);

protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 设置类加载器
    beanFactory.setBeanClassLoader(getClassLoader());
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // 添加BeanPostProcessor
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
	// ......
    }
}
```



### 注册并实例化bean工厂发布处理器,并调用他们

过程主要是工厂发布处理器的创建和调用，逻辑较多

```java
//// 3.2注册并实例化bean工厂处理器,并调用他们
invokeBeanFactoryPostProcessors(beanFactory);

protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
    // ......
}

// PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors
public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
        postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
                // 创建BeanDefinitionRegistryPostProcessor处理器
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
            }
        }
    	// 调用这些处理器
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
    	// ...

}

// 循环调用
private static void invokeBeanDefinitionRegistryPostProcessors(
			Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry) {
    for (BeanDefinitionRegistryPostProcessor postProcessor : postProcessors) {
        postProcessor.postProcessBeanDefinitionRegistry(registry);
    }
}
```

BeanDefinitionRegistryPostProcessor的子类对象在此处创建并调`postProcessBeanDefinitionRegistry`方法。

其中`org.springframework.context.annotation.ConfigurationClassPostProcessor`就是BeanDefinitionRegistryPostProcessor的子类，是一个spring的类解析器，扫描包下所有的类，解析出bean类，注册到bean工厂由此类主要参与，其中有不少递归

### 注册并实例化bean发布处理器

```java
//// 3.3注册并实例化bean处理器
registerBeanPostProcessors(beanFactory);
```

BeanFactoryPostProcessors 和 BeanPostProcessors是有区别的

BeanFactoryPostProcessors 是工厂发布处理器，定义什么是bean，知道哪些是bean类，解析class文件，包括注解解析，成员对象依赖解析等；BeanPostProcessors主要在BeanFactoryPostProcessors调用完之后工作

一般在bean对像的创建之前或之后，BeanFactory调用这些bean处理器拦截处理，Spring代理对象的创建也是通过beanPostProcessor处理器来实现

#### bean发布处理器生产AOP代理对象

AnnotationAwareAspectJAutoProxyCreator实现了BeanPostProcessors，在bean被工厂创建之后，BeanFactory调用拦截器的postProcessAfterInitialization做拦截处理。此拦截器处理器实际执行的是父类`org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator`的方法

比如一个UserServiceImp类有@service注解，并且有切点Aspectj注解增强方法，bean工厂创建userServiceImp后，代理拦截器检测到AOP相关注解，会创建动态代理对象userServiceImp$$EnhancerBySpringCGLIB并返代理对象，而不是返回userServiceImp

Spring工厂部分bean创建拦截代码逻辑

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(String, Object, RootBeanDefinition)
// bean初始化
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    invokeAwareMethods(beanName, bean);
    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 初始化之前，拦截
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    invokeInitMethods(beanName, wrappedBean, mbd);
    if (mbd == null || !mbd.isSynthetic()) {
        // 初始化之后拦截
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}

@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {
    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        // 循环bean发布处理器调用postProcessAfterInitialization方法  
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

AbstractAutoProxyCreator在此循环中被调用，比如在userServiceImp服务类上有事务注解@Transactional，一般就会被拦截生成代理对象，添加额外的处理事务的功能代码，返回增强的代理对象

### 初始化一些与上下文有特别关系的bean对象

默认tomcat服务器的创建就是此方法完成，此处定义特别的bean创建，一般是服务器有关或个性化对象，

```java
//// 3.4 初始化一些与上下文有特别关系的bean对象
onRefresh();

// org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext
// 子类context重写
@Override
protected void onRefresh() {
    super.onRefresh();
    try {
        createWebServer(); //创建服务器
    }
    catch (Throwable ex) {
        throw new ApplicationContextException("Unable to start web server", ex);
    }
}

private void createWebServer() {
    WebServer webServer = this.webServer;
    ServletContext servletContext = getServletContext();
    if (webServer == null && servletContext == null) {
        ServletWebServerFactory factory = getWebServerFactory();
        this.webServer = factory.getWebServer(getSelfInitializer()); 
        // 默认创建tomcat服务器
        // org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory.getWebServer()
    }
    // ......
    initPropertySources();
}
```



### 实例化所有bean工厂缓存的bean对象

服务器启动后，创建spring工厂里面缓存的bean信息（没有被创建的单例）

```java
//// 3.5 实例化所有bean工厂缓存的bean对象（剩下的）.
finishBeanFactoryInitialization(beanFactory);

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // ......
    // Instantiate all remaining (non-lazy-init) singletons.
    beanFactory.preInstantiateSingletons();
}

// 子类org.springframework.beans.factory.support.DefaultListableBeanFactory实现方法，完成剩下的单例bean对象创建
@Override
public void preInstantiateSingletons() throws BeansException {
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                getBean(beanName); //创建还没有实例化的bean对象
        }
    }
}
```



### 发布通知-通知上下文刷新完成 

上下文初始化完成之后，启动tomcat服务器

```java
finishRefresh();

// super.finishRefresh
protected void finishRefresh() {
    // ...... 发布刷行完成事件
    // Publish the final event.
    publishEvent(new ContextRefreshedEvent(this));
}

// org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.finishRefresh()
@Override
protected void finishRefresh() {
    super.finishRefresh();
    WebServer webServer = startWebServer();// 启动服务器
    if (webServer != null) {
        publishEvent(new ServletWebServerInitializedEvent(webServer, this));
    }
}

```





## **通知监听者-启动程序完成**

发布通知监听器启动完成，监听器会根据事件类型做个性化操作

```java
listeners.started(context);
listeners.running(context);

void started(ConfigurableApplicationContext context) {
    for (SpringApplicationRunListener listener : this.listeners) {
        listener.started(context);
    }
}

void running(ConfigurableApplicationContext context) {
    for (SpringApplicationRunListener listener : this.listeners) {
        listener.running(context);
    }
}

@Override
public void started(ConfigurableApplicationContext context) {
    context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context));
}

@Override
public void running(ConfigurableApplicationContext context) {
    context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context));
}
```



不定期更新...
