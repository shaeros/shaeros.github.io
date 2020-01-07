---
layout: post
title: 零xml配置整合springmvc和嵌入式tomcat
category: spring
tags: [springmvc]
excerpt: 零配置揭秘
keywords: spring、tomcat
---

# 零xml配置整合springmvc和嵌入式tomcat

## 1、怎么替代web.xml

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

通过spi查找META-INF/services/javax.servlet.ServletContainerInitializer 实现这个接口的类并加载

```java
@HandlesTypes(LoadServlet.class)
public class MyServletContainerInitializer implements ServletContainerInitializer {
  @Override
  public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
      Iterator var4;
      if (set != null) {
          var4 = set.iterator();
          while (var4.hasNext()) {
              Class<?> clazz = (Class<?>) var4.next();
              if (!clazz.isInterface() && !Modifier.isAbstract(clazz.getModifiers()) && LoadServlet.class.isAssignableFrom(clazz)) {
                  try {
                      ((LoadServlet) clazz.newInstance()).loadOnStartup(servletContext);
                  } catch (Exception e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  }
}
```

这个注解@HandlesTypes会找到所有实现LoadServlet接口的类并放入Set集合，接着实例化并调用loadOnStartup方法

```java
public interface LoadServlet {
   void loadOnStartup(ServletContext servletContext);
}
```

```java
public class LoadServletImpl implements LoadServlet {
  @Override
  public void loadOnStartup(ServletContext servletContext) {
      ServletRegistration.Dynamic initServlet = servletContext.addServlet("initServlet", "com.xiangxue.jack.servlet.InitServlet");
      initServlet.setLoadOnStartup(1);
      initServlet.addMapping("/init");
  }
}
```

这个实现类里面同样定义了servlet、loadOnStartup、mapping，和web.xml如出一辙

```java
public class InitServlet extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
      System.out.println("=====doget===");
      PrintWriter writer = resp.getWriter();
      writer.print("<h1>Jack</h1>");
  }

  @Override
  protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
      super.doPost(req, resp);
  }

  @Override
  public void init(ServletConfig config) throws ServletException {
      super.init(config);
  }

  @Override
  protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
      super.service(req, resp);
  }
}
```

同样，springmvc也同样是这样处理的。

## 2、嵌入式tomcat加载springmvc

同样也会在spring中spring-web-5.1.3.RELEASE.jar这个包中的META-INF/serivces/javax.servlet.ServletContainerInitializer文件中找到org.springframework.web.SpringServletContainerInitializer这个类

```java
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer
```

```java
public interface WebApplicationInitializer {
	void onStartup(ServletContext servletContext) throws ServletException;
}
```

![](https://shaeros.github.io/assets/images/2020/spring/webappinitializer.png)

我们写一个类来实现这个接口

```java
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
  //父容器
  @Override
  protected Class<?>[] getRootConfigClasses() {
      return new Class<?>[]{SpringContainer.class};
  }

  //SpringMVC配置子容器
  @Override
  protected Class<?>[] getServletConfigClasses() {
      return new Class<?>[]{MvcContainer.class};
  }

  //获取DispatcherServlet的映射信息
  @Override
  protected String[] getServletMappings() {
      return new String[]{"/"};
  }

  @Override
  protected Filter[] getServletFilters() {
      return super.getServletFilters();
  }
}
```

## 3、springmvc和spring的关系

## 4、请求响应核心流程图解