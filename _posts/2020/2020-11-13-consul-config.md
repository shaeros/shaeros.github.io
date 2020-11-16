---
layout: post
title: 配置管理-Consul
category: springcloud
tags: [springcloud]
excerpt: 配置服务器
keywords: 微服务
---

# 配置管理-Consul

## 一、Spring Boot配置管理与优先级

<https://docs.spring.io/spring-boot/docs/2.2.0.M5/reference/html/spring-boot-features.html#boot-features-external-config>

spring官网提供了17种配置方式，优先级依次递减。我们以第10种```OS environment variables```为例

```yaml
server:
  port: ${SERVER_PORT:8081}
```

大写的```SERVER_PORT```就是取环境变量的值，如果没有这个环境变量，就取```8081```这个值

以这种```java -jar demo.jar --server.port=8888```或者```SERVER_PORT=8888 java -jar demo.jar```两种方式启动，就是读取环境变量，再者可以在/etc/profile或者~/.bash_profile中设置环境变量。

这里有一个小技巧，可以访问<http://localhost:8888/actuator/env>，这里面可以看到所有的环境变量，包括系统环境变量和应用环境变量，还有一个链接<http://localhost:8081/actuator/configprops>，这个里面可以看到配置的所有属性，对于排查问题有很大作用。

## 二、Profile

```java -jar demo.jar --spring.profiles.active=prod```手动指定启动哪个配置文件

或者配置文件里面指定启动哪个配置文件

```yaml
spring:
  profiles:
    active: prod
```

## 三、为什么要使用配置服务器

### 1、配置共享与重用

多个微服务可以共用一段配置，而不用重复写

### 2、配置动态刷新

生产环境动态修改数据库连接池配置

## 四、使用Consul管理配置

### 1、如何配置

在配置之前，如果引入一个概念，叫作引导上下文(Bootstrap Context)，它是Application Context的父上下文，由Spring Cloud引入的。引导上下文是Spring Cloud用来连接配置服务器，获取远程配置的。所以想读取consul里面的配置，配置需要写在bootstrap.yml里面，配置如下

![](https://pinapple.gitee.io/assets/images/2020/springcloud/configyaml.png)

另外，配置在consul的文件夹名需要和bootstrap.yml里面的配置一致才能够读取到consul里面的配置，比如consul里面需要创建```config/service-name,dev/data```这样的格式，如果没有profiles，则为```config/service-name/data```。

### 2、配置属性动态刷新

- 刷新前提:类上有@RefreshScope注解

  ```java
  @RefreshScope
  @RestController
  public class TestController {
  
      @Value("${first.config}")
      private String config;
  
      @GetMapping("/test-config")
      public String testConfig() {
          return this.config;
      }
  }
  ```

- 自动刷新：默认情况，会有一定的时延

- 手动刷新：/refresh端点

  发送一个post请求，让应用强制刷新

  ```shell
  curl -X POST http://localhost:8080/actuator/refresh
  ```

## 五、Spring Cloud Consul支持的配置格式

consul支持四种格式

- YAML
- Properties
- KEY_VALUE
- files

不过需要遵循配置约定，yaml和properties一样，keyvalue和files如下

![](https://pinapple.gitee.io/assets/images/2020/springcloud/configkeyvalue.png)

![](https://pinapple.gitee.io/assets/images/2020/springcloud/configfiles.png)

## 六、使用git2consul管理配置

git2consul是一款基于node.js开发的小工具，它的作用是将git仓库中的数据，同步到consul中。

启动**git2consul**

git2consul --config-file /Users/shaeros/Desktop/git2consul.json -e localhost -p 8500

```json
{
  "version": "1.0",
  "repos": [
    {
      "name": "config",
      "url": "https://gitee.com/pinapple/ms-user-config-repo",
      "branches": [
        "master"
      ],
      "include_branch_name": false,
      "hooks": [
        {
          "type": "polling",
          "interval": "1"
        }
      ]
    }
  ]
}
```

在```ms-user-config-repo```仓库中创建一个文件```ms-user-dev.yaml```，内容如下：

```yaml
first.config: bob10
```

git2consul会定时地把url指定的仓库地址中的文件同步到consul中。

使用git2consul的好处有两点

1. 可以在git的提交记录里面看到每次都改动了什么，也可以提交回滚
2. 可以持久化配置，每次启动consul不会丢失，开发环境下，每次停掉consul后，创建的配置会丢失

## 七、配置管理最佳实践

- 能放本地，不放远程
- 尽量规避优先级
- 制定配置规范
- 利用/actuator/configprops、/actuator/env、启动日志快速排错