---
layout: post
title: 服务发现与注册-Consul
category: springcloud
tags: [springcloud]
excerpt: Consul
keywords: 微服务
---

# 服务发现与注册-Consul

## 一、整合Consul

### 1、加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

### 2、写配置

```properties
spring:
  application:
    # 名称如果存在分隔符，务必用中划线
    name: ms-demo
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        instance-id: ${spring.application.name}-${server.port}-${spring.cloud.client.hostname}
        metadata:
          JIFANG: NJ
```

## 二、Consul的健康检查

![](https://shaeros.github.io/assets/images/2020/springcloud/health.png)

​      如上图所示，consul会定时请求微服务给的一个路径，如果请求正常，就会标记为UP，不正常就标记为DOWN。最简单的就是编写一个端点/test，返回success，不过这种方式过于简陋，只能代表consul和微服务之间的网络是通，不代表整个微服务都是健康的并且被正常调用。

​      正确的健康检查，应该使用springboot actuator，其中的/health端点，就是用来做健康检查的。consul可以整合actuator的健康检查，如下图：

![](https://shaeros.github.io/assets/images/2020/springcloud/consulhealth.png)

