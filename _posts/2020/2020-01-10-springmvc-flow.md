---
layout: post
title: springmvc请求响应核心调用流程和过滤器
category: spring
tags: [springmvc]
excerpt: 流程解析
keywords: springmvc
---

# springmvc请求响应核心调用流程和过滤器

## 1、取代springmvc.xml

其实用一个@EnableWebMvc就可以完全取代xml，其实两者完成的工作是一样的， 都是为了创建必要组件的实例

```java
@Configuration
@EnableWebMvc
public class AppConfig extends WebMvcConfigurerAdapter {

  @Autowired
  private UserInterceptor userInterceptor;
  @Override
  public void configureViewResolvers(ViewResolverRegistry registry) {
      registry.jsp("/WEB-INF/views/", ".jsp");
  }
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
      configurer.enable();
  }
  @Override
  public void addInterceptors(InterceptorRegistry registry) {  registry.addInterceptor(userInterceptor).addPathPatterns("/user/**").excludePathPatterns("");
      super.addInterceptors(registry);
  }
}
```

比如配置视图解析器、配置解析器等等

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

这里面用@Import注解导入DelegatingWebMvcConfiguration类，并继承WebMvcConfigurationSupport，这个类里面会完成很多组件的实例化，比如HandlerMapping、HandlerAdapter

## 2、HandlerMapping工作原理及作用

## 3、HandlerAdapter工作原理及作用

## 4、springmvc过滤器和HandlerInterceptor