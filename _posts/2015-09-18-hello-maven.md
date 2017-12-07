---
layout:     post
title:      Maven 简介
date:       2015-09-18
categories: Development
---

[Apache Maven](https://maven.apache.org/)  
[Maven Users Centre](https://maven.apache.org/users/index.html)  
[MVN Repository](http://mvnrepository.com/)  
[Maven Central Repository](http://search.maven.org/)  
[Available Plugins](https://maven.apache.org/plugins/)

### 常用命令
+ **mvn –version** 
+ **export MAVEN\_OPTS="-Xmx1024m -XX:MaxPermSize=128m"**
+ **set MAVEN\_OPTS=-Xmx1024m -XX:MaxPermSize=128m**  
+ **mvn help:system** 环境变量和系统属性
+ **mvn help:effective-pom** 查看有效POM文件
+ **mvn dependency:tree** 查看依赖树  
+ **mvn dependency:build-classpath** 查看依赖CLASSPATH
+ **mvn dependency:resolve** 下载依赖包  
+ **mvn dependency:sources** 下载依赖包源代码

### Project Object Model
{% highlight xml %}
<project>
  <parent>...</parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>
  <name>...</name>
  <description>...</description>
  <url>...</url>
  <inceptionYear>...</inceptionYear>
  <licenses>...</licenses>
  <organization>...</organization>
  <developers>...</developers>
  <contributors>...</contributors>
  <dependencies>...</dependencies>
  <dependencyManagement>...</dependencyManagement>
  <modules>...</modules>
  <properties>...</properties>
  <build>...</build>
  <reporting>...</reporting>
  <issueManagement>...</issueManagement>
  <ciManagement>...</ciManagement>
  <mailingLists>...</mailingLists>
  <scm>...</scm>
  <prerequisites>...</prerequisites>
  <repositories>...</repositories>
  <pluginRepositories>...</pluginRepositories>
  <distributionManagement>...</distributionManagement>
  <profiles>...</profiles>
</project>
{% endhighlight %}

### Super POM
MAVEN\_HOME/lib/maven-model-builder-3.X.X.jar - org/apache/maven/model/pom-4.0.0.xml

### 代理设置
{% highlight xml %}
<proxy>
  <id>internal_proxy</id>
  <active>true</active>
  <protocol>http</protocol>
  <username>proxyuser</username>
  <password>proxypass</password>
  <host>127.0.0.1</host>
  <port>8087</port>
  <nonProxyHosts>repo.halo9pan.cn|repo.hinkstack.info</nonProxyHosts>
</proxy>
{% endhighlight %}

### 服务器认证
{% highlight xml %}
<server>
  <id>central</id>
  <username>your_username</username>
  <password>your_password</password>
</server>
{% endhighlight %}
[Password Encryption](https://maven.apache.org/guides/mini/guide-encryption.html)

### Maven + SVN
{% highlight xml %}
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>cn.halo9pan</groupId>
  <artifactId>hello-maven</artifactId>
  <version>1.0.0</version>
  <scm>
    <connection>scm:svn:http://localhost/svn/nc</connection>
    <developerConnection>scm:svn:https://localhost/svn/nc</developerConnection>
    <url>
    https://localhost/svn/nc
    </url>
  </scm>
  <build>
    <plugins>
      <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-scm-plugin</artifactId>
      <version>1.9</version>
      <configuration>
        <connectionType>
        connection
        </connectionType>
      </configuration>
      </plugin>
    </plugins>
  </build>
</project>
{% endhighlight %}
+ scm:add
+ scm:bootstrap
+ scm:branch
+ scm:checkin
+ scm:checkout
+ scm:diff
+ scm:edit
+ scm:list
+ scm:remove
+ scm:status
+ scm:tag
+ scm:update

### 本地库
USER\_HOME/.m2/repository
MAVEN\_HOME/conf/settings: <localRepository>/path/to/local/repo</localRepository>

### 镜像
{% highlight xml %}
<mirror>
  <id>Central</id>
  <url>http://repo1.maven.org/maven2</url>
  <mirrorOf>central</mirrorOf>
</mirror>
{% endhighlight %}
{% highlight xml %}
<mirror>
  <id>internal.mirror.halo9pan.cn</id>
  <url>http://internal.mirror.halo9pan.cn/maven/</url>
  <mirrorOf>*</mirrorOf>
</mirror>
{% endhighlight %}

### 发布
{% highlight xml %}
<distributionManagement>
  <repository>
  <id>local-file-repository</id>
  <name>Local File Repository</name>
  <url>file:///home/halo9pan/maven/deploy</url>
  </repository>
</distributionManagement>
{% endhighlight %}

### 生命周期
MAVEN\_HOME/lib/maven-core-3.2.3.jar - META-INF/plex/components.xml  
**mvn help:describe -Dcmd=deploy**  
#### Clean
pre-clean -> clean -> post-clean
#### Default
validate -> initialize -> generate-sources -> process-sources -> generate-resources -> process-resources -> compile -> process-classes -> generate-test-sources -> process-test-sources -> generate-test-resources -> process-test-resources -> test-compile -> process-test-classes -> test -> prepare-package -> package -> pre-integration-test -> integration-test -> post-integration-test -> verify -> install -> deploy
#### Site
pre-site -> site -> post-site -> site-deploy

###  阶段、目标
Packaging  
process-resources -- resources:resources  
compile -- compiler:compile  
process-test-resources -- resources:testResources  
test-compile -- compiler:testCompile  
test -- surefire:test  
package -- jar:jar  
install -- install:install  
deploy -- deploy:deploy  

### 最佳实践
#### 依赖管理 dependencyManagement
#### 定义 parent module
#### 使用 properties
#### 不要重复 parent module 的 groupId 和 version
#### 仔细遵循命名规范
#### 使用 profiles
#### 重用已有的插件
#### release 插件
#### enforcer 插件
#### 不要同时使用 release 和 snapshot 版本的库
#### 尽可能少的库
#### 使用镜像而不是修改库 URL
#### 保留默认目录结构
#### 开发时使用 SNAPSHOT 版本
#### 去除无用的依赖
#### 使用 archetypes 新建项目
#### DIY 避免重复工作