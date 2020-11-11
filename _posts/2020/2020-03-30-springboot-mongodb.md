---
layout: post
title: springboot整合mongodb
category: springboot
tags: [springboot]
excerpt: NOSQL文档数据库
keywords: springboot、mongodb
---

## 1、MongoDB介绍

##### MongoDB是一个NOSQL文档数据库，这意味着它将数据存储在类似JSON的文档中。这是考虑数据的最自然方法，比传统的行/列模型更具表现力和功能。

官网：[<https://www.mongodb.com/>](https://www.mongodb.com/)

GitHub：[<https://github.com/mongodb/mongo>](https://github.com/mongodb/mongo)

mysql和mongodb概念对比

|         | 传统关系型数据库 | mongodb    |
| ------- | ---------------- | ---------- |
| 数据库  | database         | database   |
| 表/集合 | table            | collection |
| 数据    | record/row       | document   |
| 字段    | column           | field      |

- mongodb 的适合对大量或者无固定格式的数据进行存储

- mongodb适用于不需要事务支持以及复杂join支持的业务场景

## 2、在springboot中使用

### 1、在springboot项目中加入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### 2、添加配置

在`application.properties`中添加

如果没有密码则不需要加`name:pass@`

```
spring.data.mongodb.uri=mongodb://name:pass@127.0.0.1:27017/test
```

如果是集群模式

```
spring.data.mongodb.uri=mongodb://name:pass@127.0.0.1:27017,127.0.0.2:27017/test
```

### 3、代码中使用

创建一个User实体

collection指定表名称，项目启动以后会自动在mongodb中创建表。@Data是lombok中的注解

```java
@Data
@Document(collection = "user")
public class User {
    @Id
    private String id;

    @Field
    private String username;

    @Field
    private String password;
}
```

service层注入template，来操作mongodb

```java
@Autowired
private MongoTemplate mongoTemplate;
```



