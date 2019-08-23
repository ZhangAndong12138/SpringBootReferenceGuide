---
title: SpringBoot参考指南-13.构建系统
date: 2019-08-22 03:59:06
tags: SpringBoot
---

# 第三部分 使用 Spring Boot

这一节深入细节介绍了你应该怎样使用`SpringBoot`。包括几个主题，比如构建系统，自动配置，以及如何运行你的应用。我们也提供了一些优秀的`SpringBoot`练习。虽然`SpringBoot`没什么特殊的（它只是又一个你花时间去学习的库罢了），不过有一些建议，如果你能接受的话，它们可以让你的开发过程更轻松一点。

如果你刚开始使用`SpringBoot`，在看这一节之前，也许你应该看一下[Getting Started](https://www.malaji.xyz/2019/08/22/SpringBoot%E5%8F%82%E8%80%83%E6%8C%87%E5%8D%97-8-%E4%BB%8B%E7%BB%8DSpringBoot/)向导. 

# 13. 构建系统

我们强烈推荐你选择一个支持`依赖管理`，并且可以生成能发布到"Maven中心仓库"的产品的构建系统。我们推荐你选择`Maven`或者`Gradle`。其他的构建系统（比如`Ant`）可能也可以使用`SpringBoot`，但它们可能不会被完全支持。

## 13.1 依赖管理

每一个`SpringBoot`版本都会提供一个它支持的依赖列表。在使用时，你不需要在你的构建配置中提供这个列表中的依赖，因为`SpringBoot`已经帮你管理好了。而当你升级`SpringBoot`的时候，这些依赖就都被一起升级好了。

> **笔记**
>
> 如果你需要的话，你仍然可以指定某个特定的版本来覆盖`SpringBoot`推荐的版本。

这个依赖列表包括你能在`SpringBoot`中用到的全部的`spring`模块，也包括精选的第三方类库。这个列表作为一个标准的组件清单，在`Maven`和`Gradle`上都可以使用。

> **警告**
>
> 每个`SpringBoot`的版本都关联着一个`Spring Framework`的版本。我们**强烈**建议你**不要**自行指定它。


## 13.2 Maven

Maven使用者可以继承`spring-boot-starter-parent`项目来获取默认配置。这个父项目提供了以下特性：

- 默认使用`Java1.8`编译
- `UTF-8`源编码
- 一个从`spring-boot-dependencies`pom中继承得到的，用来管理通用依赖的版本的依赖管理部分。当你在你的pom中使用这些依赖时，这个依赖管理可以允许你省略这些依赖`<version>`标签。
- 合理的资源筛选
- 合理的插件配置（exec 插件，Git提交Id和shade）
- 合理的`application.properties`和`application.yml`资源筛选，包括特定的配置文件（比如`application-dev.properties`和`application-dev.yml`）

请注意，由于`application.properties`和`application.yml`接受Spring风格的占位符`${...}`，`Maven`的占位符改为`@...@`。（你可以通过修改`Maven`的`resource.delimiter`参数来设置）

### 继承 Starter 父项目

让你的项目继承`spring-boot-starter-parent`父项目，需要如下配置：
``` xml
<!-- Inherit defaults from Spring Boot -->
<parent>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-parent</artifactId>
 <version>2.1.7.RELEASE</version>
</parent>
```

> **笔记**
>
> 你应该只需要指定`SpringBoot`的版本号。如果你引入了额外的starter，你可以放心的省略它们的版本号。

在设置的同时，你也可以通过覆盖你自己项目中的参数来覆盖特定的依赖。举个例子，要升级到另一个`Spring Data`版本，你的`pom.xml`应该这么写：

```xml
<properties>
 <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

> **Tip**
>
> 查看pom支持哪些参数：[spring-boot-dependencies pom](https://github.com/spring-projects/spring-boot/blob/v2.1.7.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)

### 不通过 Parent POM来使用Spring Boot

并不是所有人都喜欢继承`spring-boot-starter-parent`POM。你有可能要用你自己公司的标准父项目，或者你可能更喜欢自己明确声明所有的`Maven`配置。

如果如果你不想用`spring-boot-starter-parent`，你仍然可以通过使用一个`socpe=import`的依赖来享受到依赖管理的优势（但是插件就管不了了），像下面这样

```xml
<dependencyManagement>
 <dependencies>
  <dependency>
   <!-- Import dependency management from Spring Boot -->
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-dependencies</artifactId>
   <version>2.1.7.RELEASE</version>
   <type>pom</type>
   <scope>import</scope>
  </dependency>
 </dependencies>
</dependencyManagement>
```

正如前面所说的那样，上述的设置并不能允许你指定某个特定依赖的版本。为了达到同样的效果，你需要在你项目的`dependencyManagement`中的`spring-boot-dependencies` **之前**增加一条设置。举个例子，要升级到另一个`Spring Data`版本，你的`pom.xml`应该这么写：

```xml
<dependencyManagement>
 <dependencies>
  <!-- Override Spring Data release train provided by Spring Boot -->
  <dependency>
   <groupId>org.springframework.data</groupId>
   <artifactId>spring-data-releasetrain</artifactId>
   <version>Fowler-SR2</version>
   <type>pom</type>
   <scope>import</scope>
  </dependency>
  <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-dependencies</artifactId>
   <version>2.1.7.RELEASE</version>
   <type>pom</type>
   <scope>import</scope>
  </dependency>
 </dependencies>
</dependencyManagement>
```

> **笔记**
>
> 在前面的例子中，我们只特别指定了一个*BOM*，任何类型的依赖都可以通过同样的方式来覆盖。

### 使用Spring Boot Maven插件

`Spring Boot`包括了一个`Maven`插件，用这个插件，我们可以把应用打包成一个可执行的jar包。如果你想用它的话，你可以像下面的例子这样把它增加到你的`<plugins>`中：

```xml
<build>
 <plugins>
  <plugin>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-maven-plugin</artifactId>
  </plugin>
 </plugins>
</build>
```

> **笔记**
>
> 如果你的项目使用了Spring Boot starter 的父pom，你只需要增加这个插件就可以，不需要对它做任何的配置，除非你想要改变父pom中默认的配置。

## 13.3 Gradle

学习使用`Gradle`来创建`SpringBoot`，请参考`SpringBoot`的`Gradle`插件：
- 参考文档（[HTML](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/gradle-plugin/reference/html/) | [PDF](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf
)）
- [API](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/gradle-plugin/api/
)

## 13.4 Ant

使用`Apache Ant`+`Ivy`创建`SpringBoot`项目也是有可能的。`spring-boot-antlib` “AntLib”模块也可以帮助`Ant`生成可执行的jar包。

要声明依赖，一个典型的`ivy.xml`应该像下面的这个例子一样：

```xml
<ivy-module version="2.0">
 <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
 <configurations>
  <conf name="compile" description="everything needed to compile this module" />
  <conf name="runtime" extends="compile" description="everything needed to run this module" />
 </configurations>
 <dependencies>
  <dependency org="org.springframework.boot" name="spring-boot-starter"
   rev="${spring-boot.version}" conf="compile" />
 </dependencies>
</ivy-module>
```

一个典型的`build.xml`应该像这样：
```xml
<project
 xmlns:ivy="antlib:org.apache.ivy.ant"
 xmlns:spring-boot="antlib:org.springframework.boot.ant"
 name="myapp" default="build">
 <property name="spring-boot.version" value="2.1.7.RELEASE" />
 <target name="resolve" description="--> retrieve dependencies with ivy">
  <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
 </target>
 <target name="classpaths" depends="resolve">
  <path id="compile.classpath">
   <fileset dir="lib/compile" includes="*.jar" />
  </path>
 </target>
 <target name="init" depends="classpaths">
  <mkdir dir="build/classes" />
 </target>
 <target name="compile" depends="init" description="compile">
  <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
 </target>
 <target name="build" depends="compile">
  <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
   <spring-boot:lib>
    <fileset dir="lib/runtime" />
   </spring-boot:lib>
  </spring-boot:exejar>
 </target>
</project>
```

> **Tip**
>
> 如果你不想用`spring-boot-antlib`模块，可以查看这个链接中的“How-to”部分：[Section 91.9, “Build an Executable Archive from Ant without Using spring-boot-antlib”]()

## 13.5 Starters

`Starters`是一个你可以包括到你的应用中的，便捷的依赖描述的集合。有了它，你不需要再通过样板代码和复制粘贴的方式加载依赖描述，可以一站式获取到你所需的`Spring`相关技术。举个例子，如果你想要开始使用`Spring JPA`来访问数据库，只需要在项目中引入`spring-boot-starter-data-jpa`就可以了。

`Starters`包括能让一个项目快速运行起来的依赖，这些依赖是一致的，支持管理的，可传递的一个集合。

> **Starter的名称里包括哪些信息**
>
> 一个官方的`starter`名称应该像这样：`spring-boot-starter-*`，其中`*`是特定的应用类型。这个命名结构旨在帮你找到合适的`starter`。许多IDE中整合的`Maven`都会允许你通过名称搜索依赖项。比如，安装了正确的`Eclipse`或`STS`插件，你可以在`POM`编辑器中按下`Ctrl + 空格`组合键然后输入`spring-boot-starter`来获取到完整的列表。
>
> 正如在[49.5 Create Your Own Starter]()介绍的，第三方的`starter`不应该以`spring-boot`开头，这个开头是保留给官方的。第三方的`starter`应该以项目名称作为开头。比如，一个叫做`thirdpartyproject`的第三方`starter`项目应该取名` thirdpartyproject-spring-bootstarter`。

`SpringBoot`下的`org.springframework.boot`中提供了下列的应用`starter`：

---
*表13-1 SpringBoot应用starter*

名称 | 描述 |  POM文件  
-|-|-
`spring-boot-starter` | 核心`starter`，包括自动配置支持，日志和YAML | [POM]() |
`spring-boot-starter-activemq` |使用`Apache ActiveMQ`，为`JMS`消息系统提供的`starter` | [POM]() |
`spring-boot-starter-amqp` | 使用`Spring AMQP`和`Rabbit MQ`的`starter` | [POM]() |
`spring-boot-starter-aop` | 使用`Spring AOP` 和 `AspectJ`，为`面向切面编程`提供的`starter` | [POM]() |
`spring-boot-starter-artemis` | 使用`Apache Artemis`为`JMS`消息系统提供的`starter` | [POM]() |
`spring-boot-starter-batch` | `Spring`批处理的`starter`| [POM]() |
`spring-boot-starter-cache` | 使用`Spring`框架的缓存支持的`starter` | [POM]() |
`spring-boot-starter-cloud-connectors` | 使用` Spring Cloud Connectors`以连接像`Cloud Foundry`和`Heroku`这样的云平台服务 | [POM]() |
`spring-boot-starter-data-cassandra` | 使用`Cassandra`分布式数据库和`Spring Data Cassandra`的`starter` | [POM]() |
`spring-boot-starter-data-cassandra-reactive` | 使用`Cassandra`分布式数据库和`Spring Data Cassandra Reactive`的`starter` | [POM]() |
`spring-boot-starter-data-couchbase` | 使用`Couchbase`数据库和`Spring Data Couchbase`的`starter` | [POM]() |
`spring-boot-starter-data-couchbase-reactive` | 使用`Couchbase`数据库和`Spring Data Couchbase Reactive`的`starter` | [POM]() |
`spring-boot-starter-data-elasticsearch` | 使用`Elasticsearch`搜索和分析引擎和`Spring Data Elasticsearch`的`starter`| [POM]() |
`spring-boot-starter-data-jpa` | 使用`Hibernate`的`Spring Data JPA`的`starter` | [POM]() |
`spring-boot-starter-data-ldap` | 使用`Spring Data LDAP`的`stater` | [POM]() |
`spring-boot-starter-data-mongodb` | 使用`MongoDB`数据库和`Spring Data MongoDB`的`starter` | [POM]() |
`spring-boot-starter-data-mongodb-reactive` | 使用`MongoDB`数据库和` Spring Data MongoDB Reactive`的`starter` | [POM]() |
`spring-boot-starter-data-neo4j` | 使用`Neo4j`图表数据库和`Spring Data Neo4j`的`starter` | [POM]() |
`spring-boot-starter-data-redis` | 使用`Redis``key-value`数据库和`Spring Data Redis`和`Lettuce`客户端的`starter` | [POM]() |
`spring-boot-starter-data-redis-reactive` | 使用`Redis``key-value`数据库和`Spring Data Redis reactive`和`Lettuce`客户端的`starter` | [POM]() |
`spring-boot-starter-data-rest` | 使用`Spring Data REST`以`REST`方式暴露`Spring Data`仓库的`starter` | [POM]() |
`spring-boot-starter-data-solr` | 使用`Spring Data Solr`和`Apache Solr`搜索平台的`starter` | [POM]() |
`spring-boot-starter-freemarker` | 使用`FreeMarker`视图构建`MVC`web应用的`starter`| [POM]() |
`spring-boot-starter-groovy-templates` | 使用`Groovy`模板构建`MVC`web应用的`starter`| [POM]() |
`spring-boot-starter-hateoas` | 使用`Spring MVC`和`Spring HATEOAS`构建基于多媒体的`RESTful`web应用的`starter`| [POM]() |
`spring-boot-starter-integration` | 使用`Spring Integration`的`starter`| [POM]() |
`spring-boot-starter-jdbc` | 使用`JDBC`和`HikariCP`连接池的的`starter`| [POM]() |
`spring-boot-starter-jersey` | 使用`JAX-RS`和`Jersey`构建`RESTful`web应用的`starter`，可以用来替代`spring-boot-starter-web`| [POM]() |
`spring-boot-starter-jooq` | 使用`jOOQ`来访问SQL数据库的`starter`，可以用来替代`spring-bootstarter-data-jpa`或`spring-boot-starter-jdbc`| [POM]() |
`spring-boot-starter-json` | 用来读写`json`的`starter`| [POM]() |
`spring-boot-starter-jta-atomikos`|使用`Atomikos`来执行`JTA`事务的`starter`| [POM]() |
`spring-boot-starter-jta-bitronix`|使用`Bitronix`来执行`JTA`事务的`starter`| [POM]() |
`spring-boot-starter-mail`|使用`Java Mail`和`Spring Framework`的电子邮件发送支持的`starter`| [POM]() |
`spring-boot-starter-mustache`|使用`Mustache`构建web应用的`starter`| [POM]() |
`spring-boot-starter-oauth2-client`|使用`Spring Security’s OAuth2/OpenID Connect`客户端的`starter`| [POM]() |
`spring-boot-starter-oauth2-resource-server`|使用`Spring Security’s OAuth2/OpenID`服务端的`starter`| [POM]() |
`spring-boot-starter-quartz`|使用`Quartz`任务调度的`starter`| [POM]() |
`spring-boot-starter-security`|使用`Spring Security`的`starter`| [POM]() |
`spring-boot-starter-test`|使用`JUnit`、`Hamcrest`和`Mockito`对`SpringBoot`项目进行测试的`starter`| [POM]() |
`spring-boot-starter-thymeleaf`|使用`Thymeleaf`构建`MVC`web应用的`starter`| [POM]() |
`spring-boot-starter-validation`|使用`Hibernate Validator`的`Java Bean Validation`的`starter`| [POM]() |
`spring-boot-starter-web`|使用`Spring MVC`构建web应用，包括`RESTful`应用，使用`Tomcat`作为默认内置服务器的`starter`| [POM]() |
`spring-boot-starter-web-services`|使用`Spring Web Services`的`starter`| [POM]() |
`spring-boot-starter-webflux`|使用`Spring Framework`的`Reactive Web`支持来构建`WebFlux`应用的`starter`| [POM]() |
`spring-boot-starter-websocket`|使用`Spring Framework`的`WebSocket`支持来构建`WebSocket`应用的`starter`| [POM]() |

---

除了这些应用`starter`，下面的一些`starter`添加了一些[production ready]()的特性，可以在生产环境中使用：

---

*表13-2 SpringBoot生产环境starter*

名称 | 描述 |  POM文件  
-|-|-
`spring-boot-starter-actuator`|使用`SpringBoot`的`Actuator`，提供`production ready`特性来帮助监控和管理你的应用的`starter`|[POM]()|

---
最后，`Spring Boot`还包含一些用于排除或替代某些特定技术的`starter`：

---

*表13-3 SpringBoot技术starter*

名称 | 描述 |  POM文件  
-|-|-
`spring-boot-starter-jetty`|使用`Jetty`作为默认内置servlet容器的`starter`，可以用来替代`spring-bootstarter-tomcat`|[POM]()|
`spring-boot-starter-log4j2`|使用`Log4j2`记录日志的`starter`，可以用来替代`spring-boot-starter-logging`|[POM]()|
`spring-boot-starter-logging`|使用`Logback`记录日志的`starter`，是默认的日志记录`starter`|[POM]()|
`spring-boot-starter-reactor-netty`|使用`Reactor Netty `作为默认内置动态`HTTP`服务器的`starter`|[POM]()|
`spring-boot-starter-tomcat`|使用`Tomcat`作为默认内置servlet容器的`starter`，是`spring-boot-starter-web`的默认`starter`|[POM]()|
`spring-boot-starter-undertow`|使用`Undertow`作为默认内置servlet容器的`starter`，可以用来替代`spring-bootstarter-tomcat`|[POM]()|

---

> **Tip**
>
>查看GitHub上位于spring-boot-starters模块内的[README](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-starters/README.adoc)文件，可以获取到一个社区贡献的其他`starter`列表。
