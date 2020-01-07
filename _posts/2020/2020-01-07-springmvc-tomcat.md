---
layout: post
title: 零xml配置整合springmvc和嵌入式tomcat
category: spring
tags: [springmvc]
excerpt: 零配置揭秘
keywords: spring、tomcat
---

# 零xml配置整合springmvc和嵌入式tomcat

1、怎么替代web.xml

web.xml和spring.xml都不需要，xml配置的方式，都是通过读取xml里面的配置封装成对象

原始的web.xml配置会加载两个spring容器

一个通过listener

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>
    <!--加载spring配置-->
    classpath:spring.xml
  </param-value>
</context-param>
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

一个通过servlet

```xml
<servlet>
  <servlet-name>spring-dispatcher</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <!--springmvc的配置文件-->
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring-dispatcher.xml</param-value>
  </init-param>
  <load-on-startup>0</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>spring-dispatcher</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

通过spi查找META-INF/services/javax.servlet.ServletContainerInitializer这个文件下的类并加载

![image-20200107150106265](/Users/shaeros/Library/Application Support/typora-user-images/image-20200107150106265.png)



2、嵌入式tomcat加载springmvc

3、springmvc和spring的关系

4、请求响应核心流程图解