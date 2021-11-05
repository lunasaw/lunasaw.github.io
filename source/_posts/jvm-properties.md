---
title: jvm_tuning
date: 2021-8-26 15:45:12
banner_img: /img/basic/java-binner.jpg
index_img: /img/basic/java-index.jpg
tags: 
 - jvm
categories:
 - java
---

# JAVA 启动参数

/opt/vdian/java/bin/java
-Djava.util.logging.config.file=/home/www/fenxiao-core/.server/conf/logging.propertie

-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager 

-Djdk.tls.ephemeralDHKeySize=2048  TLS ephemeral diffie-hellman keys大小


-server 启用-server时新生代默认采用并行收集，其他情况下，默认不启用。-server策略为：新生代使用并行清除，年老代使用单线程Mark-Sweep-Compact的垃圾收集器。
-Xms8g 初始堆大小 默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.
-Xmx8g 最大堆大小 默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制
-XX:PermSize=256m 设置持久代(perm gen)初始值(物理内存的1/64)
-XX:MaxPermSize=256m 设置持久代最大值(物理内存的1/4)
-Xmn4g 年轻代大小 注意：此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小. 增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8
-XX:MaxDirectMemorySize=512m DirectMemory是java nio引入的，直接以native的方式分配内存，不受jvm管理。这种方式是为了提高网络和文件IO的效率，避免多余的内存拷贝而出现的。 DirectMemory占用的大小没有直接的工具或者API可以查看


-XX:SurvivorRatio=8 Eden区与Survivor区的大小比值, 设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10
-XX:+UseConcMarkSweepGC 设置年老代为并发收集(使用CMS内存收集)
-XX:+UseCMSCompactAtFullCollection 年老代使用CMS，默认是不会整理堆碎片的。设置此配置打开对年老代的压缩，即执行FullGC后对内存进行整理压缩，免得产生内存碎片，但有可能会影响性能

-XX:CMSMaxAbortablePrecleanTime=5000 指定CMS-concurrent-abortable-preclean阶段执行的时间，该阶段主要是执行一些预清理，减少应用暂停的时间。但在JDK 5.0+、6.0+的版本中有可能会由于JDK的bug29导致CMS在remark完毕后很久才触发sweeping动作。通过设置-XX: CMSMaxAbortablePrecleanTime=5（单位为ms）来避免，

**注：Concurrent Abortable Preclean(并发可取消的预清理).** 此阶段也不停止应用线程. 本阶段尝试在 STW 的 Final Remark 之前尽可能地多做一些工作。本阶段的具体时间取决于多种因素, 因为它循环做同样的事情,直到满足某个退出条件( 如迭代次数, 有用工作量, 消耗的系统时间,等等)。



-XX:+CMSClassUnloadingEnabled 对永久代区启用类回收
-XX:CMSInitiatingOccupancyFraction=80 使用cms作为垃圾回收,使用80％后开始CMS收集
-XX:+UseCMSInitiatingOccupancyOnly 使用手动定义初始化定义开始CMS收集,禁止hotspot（JVM）自行触发CMS GC

-XX:+ExplicitGCInvokesConcurrent 触发full gc 只不过在CMS在full gc 效率比较高。
http://www.liuinsect.com/2014/05/12/whats-explicitgcinvokesconcurrent-used-for/

```bash
-Dsun.rmi.dgc.server.gcInterval=2592000000 存在rmi调用时gc的时间间隔。
-Dsun.rmi.dgc.client.gcInterval=2592000000
-XX:ParallelGCThreads=4 并行收集器的线程数,此值最好配置与处理器数目相等 同样适用于CMS
-Xloggc:/home/www/fenxiao-core/logs/gc.log 把相关日志信息记录到文件以便分析.与上面几个配合使用
-XX:+PrintGCDetails 开关，可以详细了解GC中的变化。
-XX:+PrintGCDateStamps GC发生的时间信息
-XX:+HeapDumpOnOutOfMemoryError JVM会在遇到OutOfMemoryError时拍摄一个“堆转储快照”，并将其保存在一个文件中。
-XX:HeapDumpPath=/home/www/fenxiao-core/logs/java.hprof dump文件路径
-Djava.awt.headless=true
-Dsun.net.client.defaultConnectTimeout=10000 连接主机的超时时间（单位：毫秒）
-Dsun.net.client.defaultReadTimeout=30000 从主机读取数据的超时时间（单位：毫秒）
-Dfile.encoding=UTF-8
-Ddubbo.application.logger=slf4j
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=8082
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
-Djava.rmi.server.hostname=10.2.125.42
-Dproject.name=fenxiao-core
-Djava.endorsed.dirs=/opt/vdian/tomcat/endorsed
-classpath /opt/vdian/tomcat/bin/bootstrap.[jar:/opt/vdian/tomcat/bin/tomcat-juli.jar](http://jar/opt/vdian/tomcat/bin/tomcat-juli.jar)
-Dcatalina.base=/home/www/fenxiao-core/.server
-Dcatalina.home=/opt/vdian/tomcat
-Djava.io.tmpdir=/home/www/fenxiao-core/.server/temp
org.apache.catalina.startup.Bootstrap
start
```

