---
layout: post
title: Spring Boot Actuator
category: springboot
tags: [springboot]
excerpt: 监控
keywords: springboot
---

## Spring Boot Actuator

### 1、介绍

springboot actuator是官方出品的一个监控组件，提供开箱即用的健康检查功能，并且还提供其他的监控，并且把各种监控指标以端点形式暴露出来。

### 2、常用端点

图中所示就是常用的一些端点，可以常用来排查问题。

![](https://pinapple.gitee.io/assets/images/2020/springboot/actuator.png)

### 3、整合Actuator

加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

写配置

```yaml
management:
  endpoints:
    web:
      exposure:
      # 星号代表暴露所有端点
        include: '*'
  endpoint:
    health:
      show-details: always
  metrics:
    tags:
      application: ${spring.application.name}
```

启动微服务，<http://localhost:8080/actuator>，即可看到Actuator的所有端点

### 4、health端点

访问<http://localhost:8080/actuator/health>，返回结果如下

![](https://pinapple.gitee.io/assets/images/2020/springboot/healthendpoint.png)

不难发现，健康检查，不仅仅检查网络，也检查了微服务使用的资源，有磁盘、数据库、中间件等等。最终的UP是一个聚合的结果。这些检查都是自动的。

### 5、健康检查原理

![](https://pinapple.gitee.io/assets/images/2020/springboot/healthindicator.png)

可以看到实现很多的检查，以DataSourceHealthIndicator为例，获取数据库产品名，并执行验证sql。

```java
protected void doHealthCheck(Builder builder) throws Exception {
    if (this.dataSource == null) {
        builder.up().withDetail("database", "unknown");
    } else {
        this.doDataSourceHealthCheck(builder);
    }

}

private void doDataSourceHealthCheck(Builder builder) throws Exception {
    builder.up().withDetail("database", this.getProduct()); //获取数据库产品名
    String validationQuery = this.query;
    if (StringUtils.hasText(validationQuery)) {
        builder.withDetail("validationQuery", validationQuery); //验证sql
        List<Object> results = this.jdbcTemplate.query(validationQuery, new DataSourceHealthIndicator.SingleColumnRowMapper()); //执行sql
        Object result = DataAccessUtils.requiredSingleResult(results);
        builder.withDetail("result", result);
    } else {
        builder.withDetail("validationQuery", "isValid()");
        boolean valid = this.isConnectionValid();
        builder.status(valid ? Status.UP : Status.DOWN);
    }

}
```

### 6、实现自己的健康检查

如果Actuator没有实现你想要的资源检查，可以自己实现，继承AbstractHealthIndicator这个类，把具体的检查方法实现就可以了。