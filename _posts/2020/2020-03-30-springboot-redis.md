---
layout: post
title: springboot整合redis
category: springboot
tags: [springboot]
excerpt: 缓存利器
keywords: springboot、redis
---

## 1、Redis介绍

redis是一个高性能key-value、nosql内存数据库，支持string、hashmap、list、set、zset五种格式数据。

## 2、在springboot中使用

### 1、在springboot项目中加入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 2、添加配置

springboot2.0版本使用了lettuce客户端

`Lettuce`是一个高性能基于`Java`编写的`Redis`驱动框架，底层集成了`Project Reactor`提供天然的反应式编程，通信框架集成了`Netty`使用了非阻塞`IO`，`5.x`版本之后融合了`JDK1.8`的异步编程特性。

```
###redis配置###
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=
# 连接超时时间(毫秒)
spring.redis.timeout=5000
# 连接池最大连接数(使用负值表示没有限制)
spring.redis.lettuce.pool.max-active=10
# 连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=10
# 连接池最大阻塞等待时间(使用负值表示没有限制)
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=1
```

### 3、添加cache配置类

注入一个key生产器，以便我们使用@Cacheable自动生成key

重新定义RedisTemplate，设置序列化，这样存入或者读取的数据不会有乱码。

最后加入`@EnableCaching`开启缓存

```java
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

  @Override
  @Bean
  public KeyGenerator keyGenerator() {
      return new KeyGenerator() {
          @Override
          public Object generate(Object target, Method method, Object... params) {
              StringBuilder sb = new StringBuilder();
              sb.append(target.getClass().getName());
              sb.append(method.getName());
              for (Object obj : params) {
                  sb.append(obj.toString());
              }
              return sb.toString();
          }
      };
  }

  @Bean
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
      RedisTemplate<String, Object> template = new RedisTemplate<>();
      template.setConnectionFactory(factory);

      Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);
      ObjectMapper mapper = new ObjectMapper();
      mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
      serializer.setObjectMapper(mapper);
      template.setValueSerializer(serializer);
      //使用StringRedisSerializer来序列化和反序列化redis的key值
      template.setKeySerializer(new StringRedisSerializer());
      template.afterPropertiesSet();
      return template;
  }
}
```

### 4、代码中使用

1、项目中我们可以使用`RedisTemplate`来操作redis，注入即可。

```java
@Autowired
private RedisTemplate<String, String> redisTemplate;
```

2、第二种可以直接使用注解，这种方式侵入性比较低

1.value属性是必须指定的，表示当前方法使用哪个缓存器，如果该缓存器不存在，则创建一个

2.key属性是可有可无的，如果没有指定，则使用`RedisConfig`中的key生成器来自动生成

如请求<http://localhost:8080/test?id=3>，则redis生成的key为`test-key::com.neo.web.UserControllertest3`

如果知道key值，则为`test-key::saber3`

```java
@RequestMapping("/test")
@Cacheable(value="test-key",key="'saber'+#id")
public String test(Integer id) {
   return "test";
}
```

