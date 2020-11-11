---
layout: post
title: springboot整合Swagger2
category: springboot
tags: [springboot]
excerpt: API Tools
keywords: springboot、swagger2
---

## 1、Swagger2介绍

借助Swagger开源和专业工具集，为用户，团队和企业简化API开发。提供给前端接口测试，方便前后端对接。

## 2、在springboot中使用

### 1、在springboot项目中加入依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```

### 2、添加配置类

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

  //swagger2的配置文件，这里可以配置swagger2的一些基本的内容，比如扫描的包等等
  @Bean
  public Docket createRestApi() {

      List<Parameter> parameters = Lists.newArrayList();

      ParameterBuilder authParamter = new ParameterBuilder();
      authParamter.name("Authorization").description("JWT Token")
              .modelRef(new ModelRef("string")).parameterType("header")
              .required(false).build();

      parameters.add(authParamter.build());

      return new Docket(DocumentationType.SWAGGER_2)
              .apiInfo(apiInfo())
              .select()
              //为当前包路径
              .apis(RequestHandlerSelectors.basePackage("com.saber.film.controller"))
              .paths(PathSelectors.any())
              .build()
              .globalOperationParameters(parameters);
  }

  //构建 api文档的详细信息函数,注意这里的注解引用的是哪个
  private ApiInfo apiInfo() {
      return new ApiInfoBuilder()
              //页面标题
              .title("Spring Boot 测试使用 Swagger2 构建RESTful API")
              //创建人
              .contact(new Contact("Saber", "http://ke.qq.com", "617697655@qq.com"))
              //版本号
              .version("1.0")
              //描述
              .description("API 描述")
              .build();
  }
}
```

### 3、代码中使用

Controller类上使用`@Api`注解

```java
@RestController
@RequestMapping(value = "film/user")
@Api(tags = "用户模块")
public class UserController {

    @Autowired
    UserServiceAPI userServiceAPI;

    @ApiOperation(value = "用户名重复检查", notes = "用户名重复检查")
    @ApiImplicitParam(name = "username", value = "待验证的用户名称",
            paramType = "query", required = true, dataType = "string")
    @PostMapping(value = "check")
    public BaseResponseVO checkUserName(String username) throws CommonServiceException, FilmException {
        if (StringUtils.isEmpty(username)) {
            throw new FilmException(1, "用户名不能为空");
        }
        boolean hasUserName = userServiceAPI.checkUserName(username);
        if (hasUserName) {
            return BaseResponseVO.fail("用户名已存在");
        }
        return BaseResponseVO.ok();
    }

    @ApiOperation(value = "用户注册", notes = "用户注册")
    @ApiImplicitParam(name = "enrollUserVO", value = "用户注册信息",
            required = true, dataType = "EnrollUserVO")
    @PostMapping(value = "register")
    public BaseResponseVO register(@RequestBody EnrollUserVO enrollUserVO) throws CommonServiceException, FilmException, ParamErrorException {
        //贫血模型 说的是一个VO中如果只有get set 比较浪费
        enrollUserVO.checkParam();
        userServiceAPI.userEnroll(enrollUserVO);
        return BaseResponseVO.ok();
    }
```

### 4、界面上使用

打开浏览器，地址栏输入`http://localhost:8080/swagger-ui.html`，选择对应接口输入对应参数，就可以请求接口进行测试。

