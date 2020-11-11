---
layout: post
title: springboot简介和历史以及基本使用
category: springboot
tags: [springboot]
excerpt: springboot初窥
keywords: springboot
---

# springboot简介和历史以及基本使用

## 1、why springboot

springboot是一个敏捷开发框架，大大减少了开发人员对于配置方面的精力和时间。

## 2、springboot和spring对比

最初spring来开发web，需要配置大量的xml文件来解决依赖注入；以及开启事务、开启AOP等等都需要手动配置；部署时还需要放在tomcat中并启动。

而springboot解决以上问题，零配置注解来开启功能，并且内嵌tomcat、jetty、undertow等容器，直接使用main方法启动。

## 3、springboot环境搭建

Quickstart Your Project

直接使用官网提供的网址https://start.spring.io来初始化工程，或者使用idea内置的初始化。

![](https://pinapple.gitee.io/assets/images/2020/springboot/image-springinit.png)

引入两个maven依赖，就完成了最简单的环境

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

这两个依赖会导入以下jar包

![](https://pinapple.gitee.io/assets/images/2020/springboot/image-jar.png)

可以看到有spring的核心包，也有springboot相关的包，就是这样代替spring。

## 4、springboot项目开发

只需要一个@SpringBootApplication，就可以愉快地进行开发了。

```java
@SpringBootApplication
@RestController
public class DemoApplication {

public static void main(String[] args) {
  SpringApplication.run(DemoApplication.class, args);
}

@GetMapping("/hello")
public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
  return String.format("Hello %s!", name);
 }
}
```

