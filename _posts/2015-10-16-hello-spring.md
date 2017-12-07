---
layout:     post
title:      Spring 简介
date:       2015-10-16
categories: Development
---

[Expert One-on-One J2EE Development without EJB](http://www.wrox.com/WileyCDA/WroxTitle/Expert-One-on-One-J2EE-Development-without-EJB.productCd-0764558315.html)  
Rod Johnson

### Spring 模块
![Overview of the Spring Framework](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/spring-overview.png)

#### 依赖注入还是控制反转？
依赖注入(Dependency Injection)  
控制反转(Inversion of Control)  
"The question is, what aspect of control are [they] inverting?" Martin Fowler posed this question about Inversion of Control (IoC) on [his site](http://martinfowler.com/articles/injection.html) in 2004. Fowler suggested renaming the principle to make it more self-explanatory and came up with Dependency Injection.

#### 核心容器
* spring-core
* spring-beans
* spring-context
* spring-context-support
* spring-expression

#### AOP模块
* spring-aop
* spring-aspects

#### 数据访问
* spring-jdbc
* spring-tx
* spring-orm
* spring-oxm
* spring-jms

#### Web
* spring-web
* spring-webmvc
* spring-websocket
* spring-webmvc-portlet

#### 消息
* spring-messaging

#### 测试
* spring-test

### 场景

#### Spring全栈
![Typical full-fledged Spring web application](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/overview-full.png)

#### 集成三方Web框架
![Spring middle-tier using a third-party web framework](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/overview-thirdparty-web.png)

#### 远程调用
![Remoting usage scenario](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/images/overview-remoting.png)

