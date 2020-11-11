---
layout: post
title: springboot整合mybatis
category: springboot
tags: [springboot]
excerpt: 优秀的ORM框架
keywords: springboot、mybatis
---

## 1、Mybatis介绍

mybatis是一款优秀的半自动化ORM框架，完美融合springboot，使用非常方便。

GitHub：[<https://github.com/mybatis/spring-boot-starter>](<https://github.com/mybatis/spring-boot-starter>)

## 2、在springboot中使用

### 1、在springboot项目中加入依赖

```xml
<!-- 数据源驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.13</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.2</version>
</dependency>
```

### 2、添加配置

```
mybatis.type-aliases-package=
mybatis.mapper-locations=
```

### 3、代码配置

加入`@MapperScan("com.demo.mapper")`开启扫描

```java
@SpringBootApplication
@MapperScan("com.demo.mapper")
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

### 4、代码中使用

配置完以上的内容，就可以愉快地使用了。

service层注入mapper，来操作数据库

```java
@Autowired
private DemoMapper demoMapper;
```



