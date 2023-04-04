---
title: apache-log
date: 2022-09-26
banner_img: /img/apache/apache-log4j.png
index_img: /img/apache/apache-log4j.jpg
tags: 
 - apache
categories:
 - basic-component
 - apache
---
# SLF4J、JCL、JUL、log4j、logback的关系

## 名词解释

###   JUL(java.util.logging)

  Java自带的日志系统。

### log4j

`Apache Log4j 2 is an upgrade to Log4j that provides significant improvements over its predecessor, Log4j 1.x, and provides many of the improvements available in Logback while fixing some inherent problems in Logback’s architecture.`

Apache Log4j 2是Log4j的升级版，对Log4j 1.x进行了重大改进，并提供了Logback中可用的许多改进，同时解决了Logback架构中的一些固有问题。

### logback

Logback 继承自 log4j。

Logback 的架构非常的通用，适用不同的使用场景。Logback 被分成三个不同的模块：logback-core，logback-classic，logback-access。

logback-core 是其它两个模块的基础。logback-classic 模块可以看作是 log4j 的一个优化版本，它天然的支持 SLF4J，所以你可以随意的从其它日志框架（例如：log4j 或者 java.util.logging）切回到 logack。

logback-access 可以与 Servlet 容器进行整合，例如：Tomcat、Jetty。它提供了 http 访问日志的功能。

### JCL(Jakarta/Apache Commons logging)

`The Apache Commons Logging (JCL) provides a Log interface that is intended to be both light-weight and an independent abstraction of other logging toolkits. It provides the middleware/tooling developer with a simple logging abstraction, that allows the user (application developer) to plug in a specific logging implementation.`

JCL提供了一个Log接口，它是轻量的且独立于其他日志工具的抽象。它为中间件/工具开发者提供一个简单的日志抽象，允许用户（应用开发者）插入特定日志实现。

`JCL provides thin-wrapper Log implementations for other logging tools, including Log4J, Avalon LogKit (the Avalon Framework’s logging infrastructure), JDK 1.4, and an implementation of JDK 1.4 logging APIs (JSR-47) for pre-1.4 systems. The interface maps closely to Log4J and LogKit.`

JCL为其他日志工具提供轻薄包装的Log实现，包括Log4J，Avalon LogKit(Avalon Framework的日志基础设施)，JDK1.4和用于1.4之前版本系统的JDK1.4日志API(JSR-47）的实现。该接口紧密映射到Log4J和LogKit。

### SLF4J

`The Simple Logging Facade for Java (SLF4J) serves as a simple facade or abstraction for various logging frameworks (e.g. java.util.logging, logback, log4j) allowing the end user to plug in the desired logging framework at deployment time.`

SLF4J是各种日志框架（如java.util.logging， logback, log4j）的一个简单外观或抽象，它允许最终用户在部署时插入需要的日志系统。

`Note that SLF4J-enabling your library implies the addition of only a single mandatory dependency, namely slf4j-api.jar. If no binding is found on the class path, then SLF4J will default to a no-operation implementation.`

```markdown
{% note success %}
请注意，启用SLF4J意味着你的library必须强制添加依赖包slf4j-api.jar。如果在classpath中没有与之绑定的实现，
SLF4J将提供一个默认的无操作实现。
{% endnote %}
```

#### 小结

通过以上名词解释可用得出：

> JUL、log4j、logback都是日志框架，JUL为Java自带日志框架，logback是log4j框架的升级版。目前的主流是logback。
> JCL、SLF4J是日志框架的抽象，他们都致力于提供统一的日志接口，通过插件的方式在运行时注入具体的日志实现。目前看来SLF4J更胜一筹。

### SLF4J：桥接遗留接口

通常，你依赖的一些组件并非使用SLF4J日志API。你也可以假定这些组件在可预见的将来也不会切换到SLF4J。为解决这些问题，SLF4J提供了一些桥接组件来将调用重定向到log4j、JCL和JUL API，使他们看起来像是实现了SLF4J API。下图说明了此方法。

![在这里插入图片描述](https://tva1.sinaimg.cn/large/e6c9d24ely1h6gckhnmrtj216e0u0aiw.jpg)
