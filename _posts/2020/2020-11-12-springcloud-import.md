---
layout: post
title: 整合Spring Cloud
category: springcloud
tags: [springcloud]
excerpt: springcloud整合
keywords: 微服务
---

## 整合Spring Cloud

### 加依赖

选择最新版本Hoxton.SR8，type为pom，表示导入springcloud的依赖，scope为import时，仅用于dependencyManagement标签下。

```xml
<dependencyManagement>
  <dependencies>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-dependencies</artifactId>
          <version>Hoxton.SR8</version>
          <type>pom</type>
          <scope>import</scope>
      </dependency>
  </dependencies>
</dependencyManagement>
```



对应的Spring Boot依赖，选择2.3.4版本

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

