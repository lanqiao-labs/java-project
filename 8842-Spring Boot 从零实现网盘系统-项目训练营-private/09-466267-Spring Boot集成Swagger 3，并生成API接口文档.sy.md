---
show: step
version: 1.0
enable_checker: true
---

# Spring Boot 集成 Swagger 3,并生成 API 接口文档

## 实验介绍

前后端分离的项目，接口文档的存在十分重要。与手动编写接口文档不同，swagger 是一个自动生成接口文档的工具，在需求不断变更的环境下，手动编写文档的效率实在太低。与 swagger2 相比新版的 swagger3 配置更少，使用更加方便，本实验通过介绍 swagger 3，可以很方便的生成 api 文档。

#### 知识点

- springdoc-openapi 集成
- knife4j 集成
- 登录接口调试

#### 开发计划

- 开发内容：
集成 springdoc-openapi 实现接口文档的生成，并使用 knife4j 对接口文档进行页面优化及功能增强，使用 knife4j 完成接口测试。
- 开发耗时：实验预计完成时间为 2~4 小时
- 开发难点：
1. 运用 springdoc-openapi 注解，生成 api 接口文档。
2. 使用 knife4j 进行接口调试。

## Swagger 基本概念

Swagger 是一个 API 文档维护组织，后来成为了 Open API 标准的主要定义者，现在最新的版本为 17 年发布的 Swagger3。

Swagger 是一个规范和完整的框架，用于生成可视化 RESTful 风格的 Web 服务。

如果想要将 Swagger 文档集成到 Spring 中，目前有两个开源项目可供开发者选择，一个是 SpringFox，另一个是 SpringDoc，这两个项目都是由 Spring 社区来维护的，在 Swagger 2.0 时代，SpringFox 是主流，但是随着 Swagger 3.0 版本发布之后，SpringDoc 对最新版本的兼容性更好，而且 SpringDoc 支持 Swagger 页面 Oauth2 登录，因此使用 SpringDoc 是更好的选择，下面我将主要介绍 SpringDoc 集成。

## Spring Boot 集成 springdoc-openapi

### 添加依赖

向 `pom.xml` 文件中添加如下依赖：

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.5.4</version>
</dependency>
```

### 添加配置类

在 `com.shiyanlou.file.config` 包下创建 `OpenApiConfig.java` 文件，写入如下代码：

```java
package com.shiyanlou.file.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import io.swagger.v3.oas.models.ExternalDocumentation;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;

@Configuration
public class OpenApiConfig {
    @Bean
    public OpenAPI qiwenFileOpenAPI() {
        return new OpenAPI()
                .info(new Info().title("网盘项目 API")
                        .description("基于springboot + vue 框架开发的Web文件系统，旨在为用户提供一个简单、方便的文件存储方案，能够以完善的目录结构体系，对文件进行管理 。")
                        .version("v1.0.0")
                        .license(new License().name("MIT").url("http://springdoc.org")))
                .externalDocs(new ExternalDocumentation()
                        .description("网盘gitee地址")
                        .url("https://www.gitee.com/qiwen-cloud/qiwen-file"));
    }
}
```

### 注解介绍

#### @Tag 注解

该注解可以用在类或方法上，当作用在方法是用来定义单个操作，当作用在类上代表所有操作。

| 属性         | 描述                     |
| ------------ | ------------------------ |
| name         | 标签名                   |
| description  | 这里可以做一个简短的描述 |
| externalDocs | 添加一个扩展文档         |
| extensions   | 可选的扩展列表           |

#### @Operation 注解

该注解可用于将资源方法定义为 OpenAPI 操作，在该注解中也可以定义该操作的其他属性。

| 属性        | 描述                           |
| ----------- | ------------------------------ |
| method      | HTTP 请求方法                  |
| tags        | 按照资源对操作进行逻辑分组     |
| summary     | 提供此操作的简要说明。         |
| description | 对操作的详细描述               |
| requestBody | 与操作关联的请求报文           |
| parameters  | 一个可选的参数数组             |
| responses   | 执行此操作返回的可能响应的列表 |
| deprecated  | 允许将操作标记为已弃用         |
| security    | 可用于此操作的安全机制的声明。 |

#### @Schema 注解

该注解用来定义模型，主要用来定义模型类及模型的属性，请求和响应的内容、报文头等。

| 属性        | 描述                             |
| ----------- | -------------------------------- |
| not         | 提供用于禁止匹配属性的 java 类。 |
| name        | 用于描述模型类或属性的名称       |
| title       | 用于描述模型类的标题             |
| maximum     | 设置属性的最大数值。             |
| minimum     | 设置属性的最小数值。             |
| maxLength   | 设置字符串值的最大长度。         |
| minLength   | 设置字符串值的最大小度。         |
| pattern     | 值必须满足的模式。               |
| required    | 是否必输                         |
| description | 描述                             |
| nullable    | 如果为 true 则可能为 null。      |
| example     | 使用示例                         |

### 代码实现

#### 添加 Controller 接口信息

打开 `UserController.java` 类，在类上添加 `@Tag` 标签用来描述一组操作的信息，在方法上面添加 `@Operation` 注解，用来描述接口信息，代码如下：

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;

@Tag(name = "user", description = "该接口为用户接口，主要做用户登录，注册和校验token")
@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {

    ...
    @Operation(summary = "用户注册", description = "注册账号", tags = {"user"})
    @PostMapping(value = "/register")
    @ResponseBody
    public RestResult<String> addUser(@RequestBody RegisterDTO registerDTO) {
        ...
    }

    @Operation(summary = "用户登录", description = "用户登录认证后才能进入系统", tags = {"user"})
    @GetMapping("/login")
    @ResponseBody
    public RestResult<LoginVO> userLogin(String telephone, String password) {
        ...
    }

    @Operation(summary = "检查用户登录信息", description = "验证token的有效性", tags = {"user"})
    @GetMapping("/checkuserlogininfo")
    @ResponseBody
    public RestResult<User> checkToken(@RequestHeader("token") String token) {

    }

}

```

#### 添加实体类信息

在此之前我们在接口层使用了两个实体类，分别是 `RegisterDTO.java` 和 `LoginVO.java`，然后现在我们添加该实体类注解：

```java
package com.shiyanlou.file.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

@Schema(description="注册DTO")
@Data
public class RegisterDTO {
    @Schema(description="用户名")
    private String username;
    @Schema(description="手机号")
    private String telephone;
    @Schema(description="密码")
    private String password;
}
```

```java
package com.shiyanlou.file.vo;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

@Schema(description="登录VO")
@Data
public class LoginVO {
    @Schema(description="用户名")
    private String username;
    @Schema(description="token")
    private String token;
}
```

### 启动项目

启动 MySQL 数据库：

```bash
sudo service mysql start
```

点击 run 启动项目，在控制台查看项目启动成功之后，点击右侧 Web 服务进行访问，在浏览器页面地址后面追加 `/swagger-ui.html` 查看效果，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/87c813893f75f5c22d4cfea7762e07e0-0/wm)

点击其中的一个接口，查看信息：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/58e71d633523360752240a62ff58067d-0/wm)

在 Request body 项点击 Schema 可以查看该模型类信息：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/edd0d0d4e530582ca6d1d1e69487ebe2-0/wm)

从上图可以看出该模型的具体信息及其字段含义，整个文档在开发过程已经满足日常使用的要求，但是整体文档风格及排版不太符合大多数人的视觉体验，甚至有人觉得它丑，这个因人而异，因此下面我将介绍另一款工具 knife4j，来美化该文档。

### 项目集成 knife4j

knife4j 在最新版本已经基本完全兼容了 springdoc-openapi-ui，因此在替换的时候将依赖替换掉就可以了，打开 `pom.xml` 文件，删除掉之前的 `springdoc-openapi-ui` 依赖，添加如下依赖：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>3.0.2</version>
</dependency>
```

替换完成之后，重新启动项目，在地址栏之后添加 `/doc.html` 即可访问文档，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/dde375aa04e3741fbe814c3c64f24422-0/wm)

上面这种文档接口比较符合我们平时使用的 API 文档，而且各种信息也显示的比较清晰，然后在 `OpenApiConfig.java` 配置文件中添加如下配置：

```java
package com.shiyanlou.file.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import io.swagger.v3.oas.models.ExternalDocumentation;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

@Configuration
public class OpenApiConfig {
//     @Bean
//     public OpenAPI qiwenFileOpenAPI() {
//         return new OpenAPI()
//                 .info(new Info().title("网盘项目 API")
//                         .description("基于springboot + vue 框架开发的Web文件系统，旨在为用户提供一个简单、方便的文件存储方案，能够以完善的目录结构体系，对文件进行管理 。")
//                         .version("v1.0.0")
//                         .license(new License().name("MIT").url("http://springdoc.org")))
//                 .externalDocs(new ExternalDocumentation()
//                         .description("网盘gitee地址")
//                         .url("https://www.gitee.com/qiwen-cloud/qiwen-file"));
//     }

    @Bean(value = "indexApi")
    public Docket indexApi() {
            return new Docket(DocumentationType.OAS_30)
                            .groupName("网站前端接口分组").apiInfo(apiInfo())
                            .select()
                            .apis(RequestHandlerSelectors.basePackage("com.shiyanlou.file.controller"))
                            .paths(PathSelectors.any())
                            .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                        .title("网盘项目 API")
                        .description("基于springboot + vue 框架开发的Web文件系统，旨在为用户提供一个简单、方便的文件存储方案，能够以完善的目录结构体系，对文件进行管理 。")
                        .version("1.0")
                        .build();
        }
}
```

重新启动后，查看效果，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/89c90526e48209f3f5fc87090f01c37f-0/wm)

## 接口测试

目前已经实现了三个接口，分别是用户登录，用户注册，检查用户登录信息。现在我们借助 knife4j 对接口进行测试。

### 用户注册接口测试

点击左侧用户注册接口入口，点击调试页签，此时进入到了接口调试界面，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/f1ee2bd180722b531cc2199c781a6749-0/wm)

可以看到，POST 接口请求参数是使用 JSON 报文作为请求参数，输入如下请求报文点击发送:

```json
{
  "password": "test",
  "telephone": "12345678900",
  "username": "test"
}
```

请求结果如下图:

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/34944db9649a93154696979ffa14f49d-0/wm)

从上图返回成功可以看出，注册已经成功，接着我们使用该信息进行登录。

### 用户登录接口测试

点击左侧用户登录接口入口，点击调试页签，此时进入到了接口调试界面，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/11ac7d3f2a65fda1d07d3a7cc4fd2697-0/wm)

从上图可以看出，GET 接口请求是使用类似表单作为请求参数，输入注册用户信息，进行登录，结果如下图:

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/012a67c28d773ece24c6e39aee1b6353-0/wm)

### token 校验

登录成功之后，接口会返回给我们 token，现在我们调用校验接口对该 token 进行校验，来判断我们是否处于登录状态，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/fe84424124ccee28f2ccb3168abb2aa5-0/wm)

从上图可以看出该接口返回成功，说明目前处于登录状态。

## 实验总结

本节实验需要掌握 springdoc-openapi 的集成，并熟练掌握各种注解的含义及其用法，这样才能给前台显示一份完美的 api 文档，同时也可以借助该工具进行接口测试，减少后续沟通成本。

本次实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/8842/code6.zip
```
