---
layout: post
title: Ribbon的核心组件
category: springcloud
tags: [springcloud]
excerpt: RibbonCore
keywords: 微服务
---

## 一、Ribbon核心组件

![image-20201116140538351](https://pinapple.gitee.io/assets/images/2020/springcloud/image-20201116140538351.png)

了解ribbon的核心组件，可以更好地清楚ribbon实现原理，并且方便进行扩展。

其中```IRule```接口是最核心的一个组件，ribbon提供了默认值，在实际使用中，我们可以更改默认值。

## 二、Ribbon内置的负载均衡规则

![image-20201116141105620](https://pinapple.gitee.io/assets/images/2020/springcloud/image-20201116141105620.png)