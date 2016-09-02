---
title: 创建Spring-Boot微服务的Hello World
date: 2016-08-21 16:41:57
tags: Spring Boot
---
这是一个以微服务架构为流行的时代，越来越多的人以微服务的形式来开发和部署/架构我们的应用。
我们也看到Spring推出了微架构的一套基础整合框架[Spring Boot](https://projects.spring.io/spring-boot/) ，我在工作中实践这套框架也有一两年的时间了，最近也因为一时起兴倒腾了一下hexo的个人博客，于是重新参与一下现在流行的Geek方式写博客，同时也希望与更多在使用Spring boot的同行ITer进行交流。
最早我应该是从Spring boot 1.2系列的版本开始接触，一直升级追随至当前的1.4.0.RELEASE版本。

## 为什么使用Spring Boot？
Spring Boot利用习惯优于配置的原则，可以很方便快捷的创建一个应用，大多数Spring Boot应用只需要很少的一点配置就能实现，旨在让开发者更快的运行或开发应用，尽可能的专注于业务实现。像不同框架的整合、依赖的维护等这类工作就交给Spring团队的人去维护，我们只要继承相应的Maven或者Gradle父项目的基本配置。
<!--more-->
## Spring Boot的特性
* 创建标准独立的Spring应用
* 内嵌基于Tomcat，Jetty或Undertow容器直接运行，无须再手动部署war文件了
* 提供优化后的'starter' POMs给开发者简单的在Maven进行配置
* 尽可能的自动配置Spring相关配置
* 提供生产运维环境的各项指标数据，健康检查和外部形式的配置方式
* 绝对没有代码生成，不需要XML配置

## 创建我们的第一个Spring Boot应用
### 程序目录结构说明
``` java
pom.xml
src
|-- main
|   |-- java
|   |-- resources
        |-- static (静态资源目录css、js、images...)
	|-- templates (HTML页面模版目录)
	|-- application.properties (默认读取的配置文件)
|-- test
    |-- java
    |-- resources
```
Spring Boot推荐的默认文件目录结构，我们可以看到，没有webapp这一个了，都是把css、js、images等当作资源进行统一管理。
HTML页面模块的模版引擎默认配置的是Thymeleaf，我个人觉的这套模块的好处是可以以HTML的方式去写动态模版，而且开发处理得当的话，静态浏览和动态变量化都可以协作。这对前端和后台人员来说，可以无缝合作。
程序所有的配置信息和参数，统一由application.properties文件进行收纳。

### Maven相关配置
pom.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.haycco.sample</groupId>
  <artifactId>HelloWorld</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>HelloWorld</name>
  <description>Demo project for Spring Boot</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```
### 程序启动类
Application.java
``` java
package org.haycco.sample;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloWorldApplication {

	public static void main(String[] args) {
		SpringApplication.run(HelloWorldApplication.class, args);
	}
}
```
只要这两步，我们一个简单的Java Web程序就基本大功告成了。默认Web容器是tomcat，当然默认端口是8080

### 基础目录结构代码生成器
Spring Boot官网还提供了一个在线的项目代码基础结构生成器，[http://start.spring.io/](http://start.spring.io/)

### 示例源代码
其它详细信息可以自行下载源代码 [请点我下载](/samples/spring_boot_helloworld.zip)