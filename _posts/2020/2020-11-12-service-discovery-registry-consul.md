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

## 三、服务发现API

```java
@Autowired
DiscoveryClient discoveryClient;

@GetMapping("test-discovery")
public List<ServiceInstance> testDiscovery() {
    return discoveryClient.getInstances("ms-demo");
}
```

这行代码的意思是到consul上查询指定微服务的所有实例。

启动应用，访问<http://localhost:8080/test-discovery>，得到以下信息

![](https://shaeros.github.io/assets/images/2020/springcloud/discovery.png)

可以看到，能获取实例的详细信息，这样就可以调用服务提供者了。

## 四、springcloud元数据

### 1、什么是元数据

元数据是springcloud的一个扩展点，是一组map属性

### 2、元数据的设置

元数据配置如下：

```yaml
spring:
  cloud:
    consul:
      discovery:
        metadata:
          JIFANG: NJ
        tags: JIFANG=NJ
```

springcloud中的元数据为metadata，低版本的consul的元数据为tags，两者之间做了适配。新版本的consul又增加一个属性叫metadata，不过没有做适配，但是代码中也是可以使用的。

下图就是最新版本的consul中服务的配置项

![](https://shaeros.github.io/assets/images/2020/springcloud/tagmeta.png)

ConsulDiscoveryProperties中有两个变量，一个是tags，一个metadata，分别对应consulUI里面的值

![](https://shaeros.github.io/assets/images/2020/springcloud/codemeta.png)

不过如果想获取服务提供者的元数据，必须要配置tags属性，只配置metadata属性是获取不到的。

如下图，获取本身的元数据可以使用consulDiscoveryProperties.getMetadata()或者consulDiscoveryProperties.getTags()，获取服务提供者的元数据要通过ServiceInstance中的getMetadata()，虽然字面上是metadata，但其实对应的是tags的值，这是springcloud做的适配而已。

![](https://shaeros.github.io/assets/images/2020/springcloud/coderobbin.png)

### 3、元数据的作用

- 为微服务添加描述信息

- 分配元数据的调用，比如可以控制同机房优先调用

## 五、多网卡环境的IP选择

### 1、如何注册IP到consul

只要设置图中属性就可以把IP注册到consul上去

```yaml
spring:
  cloud:
    consul:
      discovery:
        prefer-ip-address: true
```

![](https://shaeros.github.io/assets/images/2020/springcloud/consulip.png)

### 2、如何指定注册到consul上的IP

现实中有这样一种情况，一台服务器有多个网卡，想把想要的IP注册到consul上，可以设置如下配置

```yaml
spring:
  cloud:
    inetutils:
      preferred-networks:
        - 192.168
        - 10.0
```

指定想要的网段就行了

## 六、自定义InstanceId

consul把instanceId作为唯一标识

![](https://shaeros.github.io/assets/images/2020/springcloud/consulip.png)

如图中的```ms-user-8081-192-168-1-41```就是instanceId，默认的instanceId是应用名称加端口的形式，这种在单机情况下是没问题的，如果在多机情况下，两个微服务实例端口一样的话，导致consul只能保留一个实例，所以要让不同实例有不同的instanceId

```yaml
spring:
  cloud:
    consul:
      discovery:
        instance-id: ${spring.application.name}-${server.port}-${spring.cloud.client.ip-address}
或者
spring:
  cloud:
    consul:
      discovery:
        instance-id: ${spring.application.name}-${server.port}-${spring.cloud.client.hostname}
```

> TIPS
>
> 这里，`${spring.cloud.client.hostname}` 以及 `${spring.cloud.client.ip-address}` ，是利用了Spring Boot配置文件可以读取环境变量的特点。
>
> 你的应用只要集成`Spring Boot Actuator` ，就可以通过 `/actuator/env` 查看所有环境变量啦！环境变量的Key值，都可以写到配置文件中。