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

替代的方法是通过spi查找META-INF/services/javax.servlet.ServletContainerInitializer 实现这个接口的类并加载

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

![](https://pinapple.gitee.io/assets/images/2020/spring/webappinitializer.png)

AbstractContextLoaderInitializer类的onStartup方法创建listener

```java
@Override
public void onStartup(ServletContext servletContext) throws ServletException {
  registerContextLoaderListener(servletContext);
}
```

```java
protected void registerContextLoaderListener(ServletContext servletContext) {

  //创建spring上下文，注册了SpringContainer
  WebApplicationContext rootAppContext = createRootApplicationContext();
  if (rootAppContext != null) {
    //创建监听器
    /*
      形如这种配置
    * <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      <!--<listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>-->
      </listener>
    *
    * */
    ContextLoaderListener listener = new ContextLoaderListener(rootAppContext);
    listener.setContextInitializers(getRootApplicationContextInitializers());
    servletContext.addListener(listener);
  }
  else {
    logger.debug("No ContextLoaderListener registered, as " +
        "createRootApplicationContext() did not return an application context");
  }
}
```

AbstractDispatcherServletInitializer类的onStartup创建servlet

```java
@Override
public void onStartup(ServletContext servletContext) throws ServletException {
  super.onStartup(servletContext);
  //注册DispatcherServlet
  registerDispatcherServlet(servletContext);
}
```

```java
protected void registerDispatcherServlet(ServletContext servletContext) {
  String servletName = getServletName();
  Assert.hasLength(servletName, "getServletName() must not return null or empty");

  //创建springmvc的上下文，注册了MvcContainer类
  WebApplicationContext servletAppContext = createServletApplicationContext();
  Assert.notNull(servletAppContext, "createServletApplicationContext() must not return null");

  //创建DispatcherServlet
  FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);
  Assert.notNull(dispatcherServlet, "createDispatcherServlet(WebApplicationContext) must not return null");
  dispatcherServlet.setContextInitializers(getServletApplicationContextInitializers());

  ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
  if (registration == null) {
    throw new IllegalStateException("Failed to register servlet with name '" + servletName + "'. " +
        "Check if there is another servlet registered under the same name.");
  }

  /*
  * 如果该元素的值为负数或者没有设置，则容器会当Servlet被请求时再加载。
    如果值为正整数或者0时，表示容器在应用启动时就加载并初始化这个servlet，
    值越小，servlet的优先级越高，就越先被加载
  * */
  registration.setLoadOnStartup(1);
  registration.addMapping(getServletMappings());
  registration.setAsyncSupported(isAsyncSupported());

  Filter[] filters = getServletFilters();
  if (!ObjectUtils.isEmpty(filters)) {
    for (Filter filter : filters) {
      registerServletFilter(servletContext, filter);
    }
  }

  customizeRegistration(registration);
}
```

以上就是springmvc替代web.xml的过程

## 3、springmvc和spring的关系

创建listener和servlet过程中，会创建spring容器，分别对应方法中的createRootApplicationContext()和createServletApplicationContext()

```java
protected WebApplicationContext createRootApplicationContext() {
  Class<?>[] configClasses = getRootConfigClasses();
  if (!ObjectUtils.isEmpty(configClasses)) {
    AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
    context.register(configClasses);
    return context;
  }
  else {
    return null;
  }
}
```

```java
protected WebApplicationContext createServletApplicationContext() {
  AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
  Class<?>[] configClasses = getServletConfigClasses();
  if (!ObjectUtils.isEmpty(configClasses)) {
    context.register(configClasses);
  }
  return context;
}
```

getRootConfigClasses()和getServletConfigClasses()分别来自于自定义的子类

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

```java
//不扫描有@Controller注解的类
@ComponentScan(value = "com.xiangxue.jack",excludeFilters = {
      @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
})
public class SpringContainer {
}
```

```java
@ComponentScan(value = "com.xiangxue.jack",includeFilters = {
      @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
},useDefaultFilters = false)
public class MvcContainer {
}
```

通过AnnotationConfigWebApplicationContext类注册扫描的类来创建spring容器

此时，只是初始化容器，但并没有走核心流程refresh来创建bean实例

1、listener中是通过把容器上下文放入到ContextLoaderListener中，从而调用initWebApplicationContext方法来调用refresh()，由listener创建的容器是父容器。

2、servlet中是创建DispatcherServlet放入容器上下文，然后在servlet的init方法中调用initServletBean()方法，initServletBean()方法中调用initWebApplicationContext()来调用refresh()，由servlet创建的容器是子容器。

## 4、请求响应核心流程图解

![](https://pinapple.gitee.io/assets/images/2020/spring/request.png)

一个请求过来，会先走HttpServlet的service方法，然后判断get或者post请求，从而走doGet或者doPost方法。

springmvc中也一样，FrameworkServlet继承了HttpServlet的service、doGet和doPost方法。

doGet和doPost中会调用processRequest方法，processRequest方法中会调用DispatcherServlet的doService方法（实现了FrameworkServlet），最终会走到核心流程doDispatch方法。

```java
protected void service(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

  HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
  if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
    processRequest(request, response);
  }
  else {
    super.service(request, response);
  }
}
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException {

processRequest(request, response);
}
protected final void doPost(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException {

processRequest(request, response);
}
```

