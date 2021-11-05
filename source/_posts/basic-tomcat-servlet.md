---
title: tomcat servlet
date: 2021-04-26 18:55:54
banner_img: /img/basic/tomcat-binner.jpg
index_img:  /img/basic/tomcat-index.jpg
tags: 
 - tomcat
categories:
 - basic-component
 - tomcat
---
# **概念**

```java
Servlet（Server Applet），全称Java Servlet，未有中文译文。是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态Web内容。狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。

Servlet运行于支持Java的应用服务器中。从实现上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。
```

**Java Web**，是基于Java语言实现web服务的技术总和。介于现在Java在web客户端应用的比较少，我把学习重点放在了JavaWeb服务端应用。虽然用Springboot就可以很快地搭建一个web项目了，但是如果想要深入了解JavaWeb的实现原理，就不得不先学习Servlet和Servlet容器的相关知识。

首先什么是Servlet？

**Servlet**从广义上讲是Sun公司提供的一门用于开发动态Web资源的技术，而狭义上指的是实现了javax.servlet.Servlet接口的类的统称。Servlet接口很简单，只有init、getServletConfig、service、getServletInfo和destroy这5个方法，它们构成了实现Servlet功能的规范。像Spring的DispatcherServlet，都是一种具体的Servlet。

当然了，光有Servlet这一个类可没什么用，它没有main方法，不能独立运行，就像光有子弹没有枪，子弹的价值就发挥不出来。要想实现Servlet的功能，就必须有一个Servlet容器。

**Servlet容器**，也叫做Servlet引擎，是web服务器的一部分，用于接收网络请求，把请求转发给对应的Servlet，并把Servlet处理的结果返回给网络。

它是web服务器和Servlet之间的媒介。

它建立服务端socket、监听端口、创建流。

它管理着Servlet的生命周期，如加载部署Servlet，实例化初始化Servlet，调用Servlet方法（处理业务），以及销毁Servlet。

**Tomcat**就是一个独立运行的Servlet容器。

# Tomcat组织结构

下面就以Tomcat8为例，看看一个具体的Servlet容器是如何实现上述功能的。

先来一张Tomcat的结构图：



![img](https:////upload-images.jianshu.io/upload_images/9705246-088f353b84544269.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Server：Tomcat顶层容器，代表着整个服务器。包含一个或多个Service组件；

Service：存活在Server内部的中间组件，包含Connector和Container这两个核心组件，负责将一个或多个Connector组件绑定到一个Container上；

Connector：监听端口，处理与客户端基于某种协议的通信，提供Socket与request和response的转换；

Container：封装和管理Servlet，负责对请求进行处理，并生成响应。

（先介绍这几个大的组件，小组件在后面会细讲）

这样展示可能比较抽象，我们可以打开我们安装的Tomcat目录下的conf/server.xml，看下Tomcat是如何配置这些组件的：

```xml
<Server port="8005" shutdown="SHUTDOWN"> 
  <Service name="Catalina"> 
    <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8"/> 
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost"> 
      <Host name="localhost" appBase="webapps"> 
        ...
      </Host> 
   <Host...
    </Engine> 

  </Service> 

</Server> 
```

先看**Connector**，可以看到一个Service里是可以配置多个Connector的，这里我们主要关心HTTP协议的Connector。

Connector使用持有的ProtocolHandler类型对象来处理请求，它包含的三个部件：

Endpoint：绑定端口、监听请求；

Processor：将Endpoint接收到的Socket封装成Request；

Adapter：将Request交给Container进行具体的处理。

再看**Container**，它内部包含了4个子容器

Engine：Servlet引擎，Container最上层，每个Service只能包含一个，表示一个特定的Service的请求处理流水线，从Connector接收处理所有的请求并返回响应；

Host：虚拟主机，一个引擎可以包含多个Host；一个Host可以包含多个Context；

Context：一个Context表示了一个Web应用程序（Web工程）。Context直接管理Servlet在容器中的包装类；

Wrapper：每一个Servlet在容器中的包装类。

我们可以通过Tomcat文件夹里的文件结构来帮助理解，上面提到的conf/server.xml里配置Host时有个appBase的属性是webapps，是不是很眼熟？我们在Tomcat安装目录下总是有一个webapp文件夹，整个webapps就是一个Host站点，里面放着的每个文件夹目录就对应一个Context，其中ROOT目录中存放着主应用，其他目录存放着子应用。

# Tomcat类加载

先看Tomcat是如何加载类的。

Tomcat启动时创建的类加载器有

> BootstrapClassLoader：加载JVM提供的基本运行类，和$JAVA_HOME/jre/lib/ext里的jar包；
>
> SystemClassLoader：加载tomcat启动的类，即Catalina.bat中指定位置的类；
>
> CommonClassLoader：加载tomcat以及应用通用的类，位于CATALINA_HOME/lib下。父加载器是AppClassLoader；
>
> WebAppClassLoader：每个应用在部署后都创建一个唯一的类加载器，加载位于WEB-INF/lib中的jar包和WEB-INF/classes下的class文件。父加载器是CommonClassLoader。
>

Tomcat默认类加载逻辑：

1、先在本地缓存中查找，如果已经加载即返回，否则继续下一步

2、尝试Bootstrap加载，如果加载到即返回，否则

3、WebApp自行加载，先/WEB-INF/classes，再/WEB-INF/lib/*.jar，如果加载到即返回，否则

4、委托WebApp父类加载器(Common ClassLoader)去加载。。。

注意：第3、4两步违反了双亲委托机制，但也只是Tomcat自定义的ClassLoader加载顺序违反了，顶层还是相同的。

Tomcat的这种加载逻辑保证了每个应用程序的同名类库是独立的，同时可以共享共有类库。

每一个JSP文件对应一个Jsp类加载器，当一个jsp文件修改了，就直接卸载这个jsp类加载器，重新创建类加载器，重新加载jsp文件。

# Tomcat启动流程

![img](https://tva1.sinaimg.cn/large/008i3skNly1gvo3i678zgg613v0me4qp02.gif)

下面开始分析Tomcat大致启动流程，建议配合源码食用。

Tomcat传统的启动入口通过startup.bat和catalina.bat脚本调用org.apache.catalina.startup.Bootstrap.main()，分为两部分：

一、init()：初始化main线程的daemon（一个Bootstrap对象）。初始化Tomcat类加载器，通过反射来实例化Catalina对象；

二、daemon执行三个方法setAwait(true)、load(args)和start()：

1、setAwait：通过反射调用catalina的setAwait方法设置await属性，后面会用到；

2、load(args)：通过反射调用catalina的load方法，创建xml解析器，解析conf/server.xml创建出了StandardServer对象并init，继而调用内部包含的service的int，以此逐层初始化所有组件；

3、start()：通过反射调用catalina的start()方法，和init方法一样逐层start所有组件；最后利用前面设置的await属性调用await方法，继而调用server的await方法，保证主线程运行并持续监听8005端口的SHUTDOWN指令，接收到后调用stop方法关闭Tomcat。

上面各个组件的init和start都是一笔带过，那么他们实际完成了什么样的工作呢？

Server.init()：调用包含的Service的init；

Service.init()：初始化Engine，初始化Executor（所有Connector共享的线程池），初始化mapperListener（用来保存容器映射），调用Connector.init；

Connector.init()：初始化ProtocolHandler、Adapter，Endpoint创建ServerSocket并绑定监听端口

Server.start()：调用包含的Services的start;

Service.start()：与初始化对应，调用Engine.start，启动Executor，启动mapperListener（作为监听者加到容器和它们的子容器中），调用Connector.start

Connector.start()：Endpoint创建acceptor线程来接收客户端的连接以及poller线程来处理连接中的读写请求

Engine.start()：逐一启动Host、Context、Wrapper

Context.start()：步骤很多，这里列举几个重要的：

*）创建读取资源文件的对象

*）创建ClassLoader对象，就是上面提到过的每个应用唯一的WebAppClassLoader

*）设置应用的工作目录

*）启动相关辅助对象，如Logger、realm、resources等

*）通知监听者ContextConfig读取和解析Web应用web.xml和注解 

*）启动web.xml解析到的子容器（解析时将Servlet包装成StandardWrapper）

*）启动Pipeline（一种责任链设计模式后面会讲）

*）获取或创建ServletContext，并设置必要的参数

*）创建Context中配置的Listener；

*）创建和初始化配置的Filter；

*）创建和初始化loadOnStartup大于等于0的Servlet

# Tomcat处理请求过程

现在我们知道Tomcat是如何启动的，那么启动之后Tomcat如何处理一次请求的呢？

下面以一次Http请求为例来说明，请求URL=http://hostname:port/contextpath/servletpath。

在Connector组件树中：

{% note success %}

前面启动的过程中提到过`Connector的Endpoint的acceptor线程负责接收Socket连接`，acceptor接收请求之后调用processSocket方法，把socket包装成SocketWrapper，创建一个SocketProcessor任务，从线程池中获取一个线程处理该任务。run方法中调用AbstractEndpoint.Handler.process方法，根据请求的协议类型（con/server.xml中connector元素的protocol属性值）创建相应的类型处理类Processor，对SocketWrapper的输入流和输出流进行包装，**根据SocketWrapper创建轻量级的coyote.Request和coyote.Response**，`解析http请求的请求头和请求行`，最后Adapter.service(Request, Response)，将coyote.Request和coyote.Response转化成Connector.Request和Connector.Response，调用connector.getService().getMapper().map()，根据hostname、contextpath和servletpath找到对应的host、context和Wapper（前面利用mapperListener保存的容器完整关系），设置到Request中去；再调用connector.getService().getContainer().getPipeline().getFirst().invoke(request, response)将请求传递给与Connector关联的Container逐级传递下去（Engine->Host->Context->Wapper）。

{% endnote %}

在Container组件树中：

`Container容器按照责任链的设计模式`，使用管道`Pipeline和Value`的方式来传递请求。

第一层是Engine，先通过conf/server.xml中配置的value，最后总会流到StandardEngineValue，调用host.getPipeline().getFirst().invoke(request, response)将请求传递给request中保存的Host；

第二层是Host，同样先通过配置的value，最后流到StandardHostValue，再传递给request中保存的Context；

第三层是Context，流到StandardContextValue传递给request中保存的Wapper；

最后是Wapper，流到StandardWapperValue，获取Servlet单例（双检查锁机制），获取FilterChain执行Filter链，也是一种责任链模式，执行完所有配置的Filter后执行Servlet.service，即我们希望其完成的业务逻辑。

（在进入Filter的时候，传入的是Connector.Request的门面类RequestFacade，和Request一样都是HttpServletRequest和HttpServletResponse的实现类）

返回过程略。

## 专注HTTP请求的Servlet

1　写一个专门处理HTTP请求的Servlet

因为现在我们的请求都是基于HTTP协议的，所以我们应该专门为HTTP请求写一个Servlet做为通用父类。

对于专注于HTTP的Servlet，我们需要处理以下几个问题：

service()方法的参数ServletRequest和ServletResponse，但因为所有的请求都是HTTP请求，所以传递给service()就去的参数其实是HttpServletRequest和HttpServletResponse对象。如果子类希望使用的是HttpServletRequest，而不是ServletRequest，那么它需要自己去做强转。

 

Servlet：SUN公司 设计之初 以后不仅仅依赖HTTP协议

       GenericServlet  --- 通用Servlet
       service(ServletRequest req, ServletResponse res)
       HttpServlet    --- 与协议相关的Servlet
       service(HttpServletRequest req, HttpServletResponse resp)

 以后再写Servlet  继承  HttpServlet

       Servlet           --- 一个标准
              |
       GenericServlet      ---  是Servlet接口子类
              |
       HttpServlet    --- 是GenericServlet子类，一个专门处理Http请求的Servlet

HttpServlet

       两个Service方法
              * 父类service 调用子类service   使用子类service方法就可以
              * 子类中service 根据请求方式不同 调用不同的方法
              只需要重写doGet和doPost就行.             

       写一个Servlet 继承HttpServlet
    
       重写doGet和doPost 方法.

没错，Java中已经存在了javax.servlet.http.HttpServlet类。打开源代码看看这个类！

2　HTTP请求方法
HTTP请求方法不只是GET和POST，还有其他的方法，但基本上用不上。这里只是简单介绍一下。你自己心里有个数，HTTP请求除了GET和POST之外还有别的就行了。

> GET通过请求URI得到资源
> POST用于添加新的内容
> PUT用于修改某个内容
> DELETE,删除某个内容
> CONNECT用于代理进行传输，如使用SSL
> OPTIONS询问可以执行哪些方法
> PATCH部分文档更改
> RACE用于远程诊断服务器
> HEAD类似于GET, 但是不返回body信息，用于检查对象是否存在，以及得到对象的元数据
> TRACE用于远程诊断服务器

#### 参考

[^1]: https://blog.csdn.net/qq_19782019/article/details/80292110

