---
title: SpringBoot2.0 新特性
category:
  - Spring
tags:
  - springboot
abbrlink: aaed1c03
date: 2018-03-09 00:00:00
---
## SpringBoot 2.0 新特性
在 2018 年 3 月 1 日早上，Spring Boot 2.0 发布，在Spring Boot的官网中，2.0.0已经是最新的Spring Boot推荐版本，并提供了 Maven 中央仓库地址。

官方表示，这个版本经历了 17 个月的开发，有 215 个不同的使用者提供了超过 6800 次的提交。该版本是自 4 年前发布 Spring Boot 1.0 以来的第一次重大修订，也是首个提供对 Spring Framework 5.0 支持的 GA 稳定版本。

Spring Boot 2.0 主要有以下特性（详见：[Spring Boot 2.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes)）。
## 支持Java 8和Java 9
SpringBooot 2.0最低支持Java 8 版本，许多现有的API更新，以利用Java 8 的特性，例如：接口的默认方法，函数回调和新的APIs,如：javax.time。如果你当前使用的是Java 7或者更早的版本，在你开发Springboot 2.0程序之前，你需要升级你的Java版本

<br />Spring Boot 2.0也可以很好地工作，并且已经通过JDK 9进行了测试。

## 第三方库的升级
Spring Boot 2.0建立在Spring Framework 5之上，并且需要Spring Framework 5

Spring Boot 2.0已尽可能升级到其他第三方游戏机的最新稳定版本。本版本中一些值得注意的依赖性升级包括
<br />Tomcat 8.5
<br />Flyway 5
<br />Hibernate 5.2
<br />Thymeleaf 3
## 更好的响应式支持
SpringBoot 2.0通过auto-configuration 和 starter-POMs更好的支持响应式应用。Spring Boot的内部本身也在必要时进行了更新，以提供反应性的反应（最明显的是嵌入式服务器支持）

使用 Spring WebFlux/WebFlux.fn 提供响应式 Web 编程支持
Spring Data还为响应式应用程序提供支持。目前Cassandra，MongoDB，Couchbase和Redis都有反应式API支持

## 支持 HTTP/2 
为Tomcat，Undertow和Jetty提供HTTP / 2支持。是否支持取决于所选的Web服务器和应用程序环境（因为JDK 8不支持该协议）
## Gradle 插件
Spring Boot的Gradle插件在很大程度上已被重写，以实现许多重大改进。需要注意：SpringBoot 2.0 现在需要Gradle 4.x 的版本
## Kotlin
SpringBoot 2.0对Kotlin 1.2.x 版本的支持，并提供了一个runApplication函数，该函数提供了一种使用惯用Kotlin运行Spring Boot应用程序的方法。
## 对Quartz调度支持
SpringBoot 2.0 对Quartz的支持，我们只需要加入 `spring-boot-starter-quartz` starter POM.

支持JobStores或者基于JDBC的存储。 Spring应用程序上下文中的所有JobDetail，Calendar和Trigger bean将自动注册到Scheduler中
## Testing 测试
新版本对测试做了一些改变。
* 一个新的`@WebFluxTest`注解，以支持WebFlux应用程序的“slice”测试。
* 现在使用`@WebMvcTest`和`@WebFluxTest`自动扫描Converter和GenericConverter bean
## 动画ASCII艺术
最后，为了好玩，Spring Boot 2.0现在支持动画GIF横幅。
[![](http://myluffy.com/wp-content/uploads/2018/03/animated-ascii-art-1.gif)](http://myluffy.com/wp-content/uploads/2018/03/animated-ascii-art-1.gif)