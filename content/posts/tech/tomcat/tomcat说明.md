+++
date = '2024-03-20T22:22:09+08:00'
draft = false
title = 'tomcat说明'
description = "tomcat说明"      
summary = "tomcat说明"                       
categories = ["📒tomcat"]                                      
tags = ["tomcat"]                                                  
+++
```shell
●bin:存放启动和关闭Tomcat的脚本文件，比较常用的是 catalina.sh、startup.sh、shutdown.sh三个文件
●conf：存放Tomcat 服务器的各种配置文件，比较常用的是 server.xml、context.xml、tomcat-users.xml、web.xml 四个文件。
●server.xml: Tomcat的主配置文件，包含Service，Connector，Engine，Realm，Valve,Hosts主组件的相关配置信息;
●context.xml:所有host的默认配置信息;
●tomcat-user.xml:Realm认证时用到的相关角色、用户和密码等信息，Tomcat自带的manager默认情况下会用到此文件，在Tomcat中添加/删除用户，为用户指|定角色等将通过编辑此文件实现;
●web.xml:遵循Servlet规范标准的配置文件，用于配置servlet，并为所有的web应用程序提供包括MIME映射等默认配置信息;
●lib：存放Tomcat运行需要的库文件的jar 包，一般不作任何改动，除非连接第三方服务，比如 redis，那就需要添加相对应的jar 包
●logs：存放 Tomcat 执行时的日志
●temp：存放 Tomcat 运行时产生的文件
●webapps：存放 Tomcat 默认的 Web 应用部署目录
●work：Tomcat工作日录，存放jsp编译后产生的class文件，一般清除Tomcat缓存的时候会使用到
●src:存放Tomcat 的源代码
●doc:存放Tomcat文档
```

```shell
CLASSPATH：编译、运行Java程序时，JRE会去该变量指定的路径中搜索所需的类（.class）文件。
dt.jar：是关于运行环境的类库，主要是可视化的 swing 的包。
tools.jar：主要是一些jdk工具的类库，包括javac、java、javap（jdk自带的一个反编译工具）、javadoc等。
JDK ：java development kit （java开发工具）
JRE ：java runtime environment
（java运行时环境）
JVM ：java virtual machine （java虚拟机），使java程序可以在多种平台上运行class文件。
```
```shell
#### 扩展和优化 Tomcat 的 catalina.sh 文件以调整 JVM 参数

CATALINA_OPTS="-server \
                -Xms2048m -Xmx2048m \
                -Xmn512m \
                -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m \
                -XX:NewRatio=3 \
                -XX:SurvivorRatio=8 \
                -XX:+UseG1GC \
                -XX:MaxGCPauseMillis=200 \
                -XX:InitiatingHeapOccupancyPercent=70 \
                -XX:+PrintGCDetails \
                -XX:+PrintGCDateStamps \
                -XX:+PrintGCCause \
                -Xloggc:/var/log/tomcat/tomcat_gc.log \
                -Djava.awt.headless=true \
                -Dcom.sun.management.jmxremote.port=10086 \
                -Dcom.sun.management.jmxremote.ssl=false \
                -Dcom.sun.management.jmxremote.authenticate=false"


-server: 启用服务器模式的 JVM。
-Xms 和 -Xmx: 分别设置堆的初始大小和最大大小，保持一致可以减少堆大小调整带来的开销。
-Xmn: 设置年轻代（Young Generation）大小。
-XX:MetaspaceSize 和 -XX:MaxMetaspaceSize: 设置元空间（MetaSpace，取代了 PermGen）的初始和最大大小。
-XX:NewRatio: 设置老年代与年轻代的大小比例。
-XX:SurvivorRatio: 设置 Eden 区与 Survivor 区的大小比例。
-XX:+UseG1GC: 使用 G1 垃圾收集器，根据实际需求可选用其他适合的 GC 策略。
-XX:MaxGCPauseMillis: 设置期望的最大 GC 停顿时间。
-XX:InitiatingHeapOccupancyPercent: 设置触发并发标记周期的堆占用百分比。
-XX:+PrintGCDetails 等：开启详细的 GC 日志记录，便于分析和调优。
# 禁用显式gc
-XX:+DisableExplicitGC # 自动将System.gc() 调用转换成一个空操作，即应用中调用
System.gc()会变成一个空操作，避免程序员在代码里进行System.gc()这种危险操作。System.gc()除
非是到了万不得已的情况下使用，都应该交给JVM。
-Xloggc: 设置 GC 日志文件路径。
-Djava.awt.headless=true: 防止在无图形界面环境中出现相关异常。
-Dcom.sun.management.jmxremote.*: 开启 JMX 远程监控，便于通过 JConsole 或其他工具监控 JVM 状态。

```