---
layout: post
title: 什么是Spring Cloud？
category: springcloud
tags: [springcloud]
excerpt: springcloud
keywords: 微服务
---
## 什么是Spring Cloud？

```快速开发分布式应用的工具集```

[SpringCloud官网](https://spring.io/projects/spring-cloud)

### Spring Cloud主要功能

![](https://pinapple.gitee.io/assets/images/2020/springcloud/springcloud.png)

### Spring Cloud子项目

![](https://pinapple.gitee.io/assets/images/2020/springcloud/springcloud-child.png)

### Spring Cloud版本命名

- 版本顺序:
Angel -> Brixton -> Camden -> Dalston -> Edgware -> Finchley -> Greenwich -> Hoxton

- 指定版本的发布流程:
SNAPSHOT -> Mx -> RELEASE -> SRx

### Spring Cloud生命周期

- 版本发布规划
 https://github.com/spring-cloud/spring-cloud-release/milestones 
- 版本发布记录
 https://github.com/spring-cloud/spring-cloud-release/releases
- 版本终止声明
 https://spring.io/projects/spring-cloud#overview

### Spring Boot与Spring Cloud兼容性映射关系

![](https://pinapple.gitee.io/assets/images/2020/springcloud/cloud-boot.png)

兼容性查询
[https://spring.io/projects/spring-cloud](https://spring.io/projects/spring-cloud)

### 生产环境如何选择？

- 坚决不用非稳定版本/end-of-life版本
- 尽量用最新一代
SR2之后一般可大规模使用