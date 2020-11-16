---
layout: post
title: 客户端负载均衡器-Ribbon
category: springcloud
tags: [springcloud]
excerpt: 负载均衡
keywords: 微服务
---

## 一、什么是客户端负载均衡器

### 1、负载均衡

![image-20201116110027484](https://pinapple.gitee.io/assets/images/2020/springcloud/image-20201116110027484.png)

我们现在要实现的如上图一样的功能，部署了多个用户微服务实例，课程微服务想要请求的时候，由负载均衡器帮我们选择一个实例来响应。

早期的比较优秀的负载均衡器如Nginx，这是服务器端负载均衡器，负载均衡算法是由Nginx来提供，用户请求过来由Nginx来选择一个后端服务来响应。

![image-20201116110249811](https://pinapple.gitee.io/assets/images/2020/springcloud/image-20201116110249811.png)

目前Spring Cloud比较流行的客户端负载均衡组件是Ribbon，它提供了很多算法来选择微服务实例，如果算出来要请求实例3，然后就把它交给RestTemplate去请求。

![image-20201116110631334](https://pinapple.gitee.io/assets/images/2020/springcloud/image-20201116110631334.png)

### 2、Ribbon & Spring Cloud Loadbalancer对比与选择

目前Cloud生态中，有Ribbon、Loadbalancer两种负载均衡器，他们实现原理都大同小异，现在来做个比较。

| 负载均衡器   | 优点                                       | 缺点                           |
| ------------ | ------------------------------------------ | ------------------------------ |
| Ribbon       | 成熟、流行、稳定                           | 维护模式，不再升级             |
| Loadbalancer | 作为下一代客户端侧负载均衡器用来替代Ribbon | 过于简陋，暂时没有找到成功案例 |

## 二、负载均衡器的本质

- 给它一个List，让它给你选择1个元素

## 三、使用Ribbon实现负载均衡

### 1、什么是Ribbon

Ribbon是Netflix开源的客户端侧负载均衡器

![image-20201116135529402](https://pinapple.gitee.io/assets/images/2020/springcloud/image-20201116135529402.png)

### 2、如何使用？

#### 1、加依赖

不需要加，因为```consul-discovery```这个包里面已经加了ribbon依赖。

#### 2、写注解

加上```@LoadBalanced```即可

```java
@Bean
@LoadBalanced
RestTemplate restTemplate() { return new RestTemplate(); }
```

#### 3、写代码

url里面直接使用微服务的名称即可

```java
restTemplate.getForObject("http://ms-user/users/{userId}", UserDto.class, userId);
```

