---
layout: post
title:  "Hello Tomcat!"
date:   2015-08-28 14:00:00
categories: Development
---

### Tomcat 目录结构
- **bin** 执行脚本，启动jar包
- **conf** 配置文件
- **lib** Tomcat 的运行库和 JSP/Servlet API
- **logs** 默认日志目录
- **temp** 临时目录
- **webapps** Web Application 目录
- **work** 运行时目录，比如由 JSP 编译的 class

### Tomcat 启动脚本
catalina.(sh|bat) 是 Tomcat 启动的核心脚本，startup.(sh|bat) 和 shutdown.(sh|bat) 都是调用该脚本启动和停止 Tomcat 的。  
从脚本中可以看到，有一些常用的环境变量可以影响到启动和停止的过程：  

- **CATALINA_HOME** Tomcat 的安装目录，就是下载后解压的文件路径。
- **CATALINA_BASE** 这个算是 Tomcat 的运行实例路径，默认和 **CATALINA_HOME** 一致，但是如果期望运行多个 Tomcat 实例，可以用同一个 **CATALINA_HOME**，但是指定不同的 **CATALINA_BASE** 为实例配置目录。该目录下要包含配置文件 **conf**，一般会把日志目录 **logs**、临时目录 **temp**、Web Application 目录 **webapps**、运行时目录 **work** 也可以配置在 **CATALINA_BASE** 里面，但是脚本目录 **bin** 和运行库 **lib** 是不需要的。
- **CATALINA_OPTS** 优化参数，比如堆大小，GC，JMX等都可以在这里配置，一般这些优化是专门针对Tomcat的
- **CATALINA_TMPDIR** Tomcat 的临时目录，而非 Java 的临时目录，默认就是 CATALINA_BASE/temp
- **JAVA_HOME** 如果需要 debug Tomcat，就要指向 JDK，否则指向 JRE，Tomcat也能运行。
- **JRE_HOME** 这个就是指定 JRE 啦.
- **JAVA_OPTS** Java 优化参数，因为 JAVA\_OPTS 还会被其他的 Java 程序所使用，因此 JAVA_OPTS 算是一个全局的环境变量，如果想配置仅仅针对 Tomcat 的，还是用 CATALINA\_OPTS 吧。
- **JAVA\_ENDORSED\_DIRS** 指定 Java endorsed 目录，Java Endorsed 是 Java 的 API 库覆盖的机制，就是可以用其它的三方实现来代替 JDK 包含的默认实现，详见 [Java Endorsed Standards Override Mechanism](http://docs.oracle.com/javase/7/docs/technotes/guides/standards/)。默认目录在 CATALINA\_HOME/endorsed，默认安装是没有这一目录的，新建一个就行。
- **JPDA_TRANSPORT** [JPDA](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/) 的通讯机制，默认是 dt\_socket。
- **JPDA_ADDRESS** [JPDA](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/) 的通讯地址及端口 ，默认是localhost:8000.
- **JPDA_SUSPEND** [JPDA](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/) 的中断策略，是否在刚刚启动的时候就暂停，默认是 n。
- **JPDA_OPTS** [JPDA](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/) 的全部配置参数,设置之后  JPDA\_TRANSPORT，JPDA\_ADDRESS和JPDA\_SUSPEND全部失效。
- **LOGGING_CONFIG** 覆盖 Tomcat 默认的日志配置文件，比如 LOGGING\_CONFIG="-Djava.util.logging.config.file=%CATALINA\_BASE%\conf\logging.properties"
- **LOGGING_MANAGER** 覆盖 Tomcat 默认的日志管理类，比如 LOGGING\_MANAGER="-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager"

### Tomcat 配置文件
#### catalina.policy
Tomcat 的 Security Policy Permissions 配置文件，具体可参考[Default Policy Implementation and Policy File Syntax](http://docs.oracle.com/javase/8/docs/technotes/guides/security/PolicyFiles.html)
