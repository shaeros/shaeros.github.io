---
layout: post
title: Ribbon配置自定义
category: springcloud
tags: [springcloud]
excerpt: Config
keywords: 微服务
---

> TIPS
>
> Ribbon可以进行细粒度的配置，可以为每一个微服务实例配置不同的负载均衡规则

## 一、Java代码方式

```java
@Configuration
@RibbonClient(name = "ms-user", configuration = RibbonConfiguration.class)
public class MsUserRibbonConfig {
}
```
官方解释：<https://docs.spring.io/spring-cloud-netflix/docs/2.2.5.RELEASE/reference/html/#spring-cloud-ribbon>

> The `CustomConfiguration` class must be a `@Configuration` class, but take care that it is not in a `@ComponentScan` for the main application context. Otherwise, it is shared by all the `@RibbonClients`. If you use `@ComponentScan` (or `@SpringBootApplication`), you need to take steps to avoid it being included (for instance, you can put it in a separate, non-overlapping package or specify the packages to scan explicitly in the `@ComponentScan`).

```java
@Configuration
public class RibbonConfiguration {
    @Bean
    public IRule iRule() { return new MyRibbonRuleV2(); }
}
```

## 二、配置属性方式

- <clientName>.ribbon.NFLoadBalancerRuleClassName=负载均
  衡规则的全路径

只要在配置文件中加上上面的配置就可以达到细粒度配置，这样代表的是用户微服务使用随机规则

```yaml
#配置属性配置方式
ms-user:
  ribbon:
    NFLoadbalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

## 三、两种方式对比

| 配置方式 | 优点                                                         | 缺点                                             |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ |
| 代码配置 | 基于代码，更加灵活                                           | 有小坑(父子上下文)<br />线上修改得重新打包、发布 |
| 属性配置 | 易上手<br />配置更加直观<br />线上无需重新打包、发布<br />**优先级更高** | 极端场景下没有代码配置方式灵活                   |

## 四、Ribbon配置最佳实践总结

- 尽量使用属性配置，属性方式实现不了的情况下再考虑用代码配置 

- 在同一个微服务内尽量保持单一性
   • 比如统一使用属性配置，不建议两种方式混用，增加定位代码的复杂性 

## 五、Ribbon全局配置

```java
@Configuration
@RibbonClients(defaultConfiguration = RibbonConfiguration.class)
public class RibbonGlobalConfig {
}
```

目前全局配置只支持代码方式配置，没有配置属性方式。

## 六、Ribbon支持的配置项

Java代码的方式用```@Bean```方式修改接口实现即可

配置属性方式用如下方式：

<clientName>.ribbon.如下属性：

- NFLoadBalancerClassName：ILoadBalancer实现类
- NFLoadBalancerRuleClassName：IRule实现类
- NFLoadBalancerPingClassName：IPing实现类
- NIWSServerListClassName：ServerList实现类
- NIWSServerListFilterClassName：ServerListFilter实现类

## 七、Ribbon饥饿加载

默认情况下Ribbon是懒加载的，这样导致第一次访问接口会比较慢，后面才会很快，可以配置饥饿加载，让第一次也很快。

```yaml
ribbon:
  eager-load:
    enabled: true
    #如果有多个，以逗号分隔，不支持*通配
    clients: ms-user
```

