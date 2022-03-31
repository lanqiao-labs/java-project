---
show: step
version: 1.0
enable_checker: true
---

# Spring Boot 整合 Java Web Token 实现用户登录认证

## 实验介绍

登录认证是一个系统最基本的功能，本节实现带领大家使用 Java Web Token 来进行登录验证，再此之前我会首先对 session 和 token 做一个详细的讲解，因为这个是一个比较容易混淆的知识点，然后通过代码来实现 Java Web Token 认证。

#### 知识点

- session 与 token 发展历程
- Java Web Token 介绍
- 登录认证流程

#### 开发计划

- 开发内容：
使用 JWT 来完成用户的登录校验

- 开发耗时：实验预计完成时间为 2~4 小时
- 开发难点：
1. 理解 JWT 生成原理。
2. 用户登录校验。

## session 与 token 发展历程

我们都知道，一个普通 Web 应用前后台进行交互使用的是 HTTP 协议，由于 HTTP 协议是无状态的，因此每个请求对于后台服务端来说，都是全新的，但是随着互联网的 Web 应用兴起，像很多购物网站，需要登录的网站就面临一个问题，那就是会话管理，后台需要记住哪些人登录系统，哪些人进行了购物操作，也就是说必须把每个人区分开，但是 HTTP 请求时无状态的，所以就想到一个办法，给每个请求者发一个会话标识，也就是 session，说白了就是一个随机的字符串，保证每个人不重复，这样当大家再次向服务端发送请求的时候，就能区分开来谁是谁了。

随着用户数量的增加，服务器要保存所有用户的 session，这对服务器来说是一个巨大的开销，严重的限制了服务器的扩展能力，比如负载均衡+服务器集群部署方式，往往用户在 A 服务器登录了系统，session 会保存在机器 A 上，但是如果下一次请求被转发到了机器 B 怎么办？通常有两种做法，一种是通过 Sticky 技术将相同用户请求分发到同一个机器上，但是这样就违背了做负载均衡的初衷了，而且万一这个机器挂了，那么还是会请求到另外的机器上。另一种做法就 session 的复制了，将 session 在多个应用之间进行复制，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/f2fa9ecf92b1f56bced1a10bc651ead7-0/wm)

但是集群的数量如果过大，将 session 拷来拷去，却是一件很麻烦的事情，因此，有人支招，将 session 集中存储到一个地方，所有机器都来访问这个地方的数据，这样一来，就不用复制了，如下：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/fa008106d06141c062e48b8cb6506978-0/wm)

但是如果这个 session 存储的机器挂了，所有人都得受到影响，因此也尝试将这个机器也搞一个集群，增加可靠性，但是不管如何，这个小小的 session 对后台来说都是一个沉重的负担。

于是又有人思考，为什么后台要保存这些 session 呢？如果让每个客户端自己去保存该有多好，出于这个起点，人们开始不断的尝试。关键点在于如果服务端不保存 session，如何知道 session 是我生成的。关键点就在于验证，比如客户端 M 登录了系统，我给他发了一个令牌(token),里面包含了该用户的 user id，用来标识客户端身份,下次客户端 M 再次访问我的时候，再把这个 token 通过 Http header 带过来就行了，不过这个本质和 session 没区别，任何人都可以伪造，所以得想点办法，让别人无法伪造。

那就对数据做个签名，将数据和密钥一起作为 token，由于密钥别人不知道，就无法伪造 token 了。当客户端再次进行请求，服务端只需要用密钥再次签名进行验证，如果签名一致，则为有效 token，这时就可以直接取到 user id，如果签名不一致，则说明数据被篡改了，则告诉请求者，认证失败。

## Java Web Token 介绍

上面介绍了 session 与 token 的发展历程，相信大家对 token 已经有了一个初步的认识，我们能够发现，传统的 token 认证有一个缺陷，就是用户的关键信息，比如 id 等信息是通过明文来进行传输的，因此基于安全考虑，Java Web Token 应运而生了。

### Java Web Token 概念

JSON Web Token (JWT) 是一个开放标准（RFC 7519），它定义了一种紧凑且自包含的方式，用于将信息作为 JSON 对象安全地在各方之间传输信息。此信息可以验证和信任，因为它是数字签名。JWT 可以使用密钥（使用 HMAC 算法）或使用 RSA 或 ECDSA 进行公钥/私钥对进行签名。

#### JWT 结构

JSON Web 令牌以紧凑的形式由三个部分组成，由点（.）分隔，它们包括：

- Header
- Payload
- Signature

将上面三部分用（.）拼接起来就形成了JWT，因此，JWT 的格式通常如下所示。

`xxxxx.yyyyy.zzzzz`

#### Header

标头通常由两部分组成：token 的类型（typ）和正在使用的签名算法（alg），如 HMAC SHA256 或 RSA。

例如：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后，此 JSON 编码为 Base64Url，以形成 JWT 的第一部分。

#### Payload

token 的第二部分是有效负载，其中包含 claims。claims 是关于实体（通常为用户）和其他数据的语句。

示例有效负载可能是：

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

然后对有效负载进行 Base64Url 编码，以形成 JSON Web 令牌的第二部分。

请注意，对于已签名的 token，此信息虽然可防止篡改，但任何人都可以阅读。除非对 JWT 进行加密，否则不要将机密信息放在 JWT 的有效负载或标头元素中。

#### Signature

要创建签名部分，您必须使用编码标头、编码有效负载、机密、标头中指定的算法，并签名。

例如，如果要使用 HMAC SHA256 算法，将采用以下方式创建签名：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名用于验证信息在传输过程中不被篡改，对于使用私钥签名的 token，它还可以验证 JWT 的发件人是否为它所说的发件人。

### 放在一起

输出是三个 Base64-URL 字符串，由点分隔，这些点可以在 HTML 和 HTTP 环境中轻松传递，但与基于 XML 的标准（如 SAML）相比，更紧凑。

## 集成 Java Web Token

### 添加依赖

向 `pom.xml` 添加如下依赖：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

### 添加配置文件

上面已经介绍了 JWT 的基本概念了，它由三部分构成，分别是 header, payload 和 signature, 其中前两部分中的参数设计到的一些参数需要用户自定义，因此我们将这些参数放到配置文件中，方便后续修改。

打开 `application.properties` 文件，添加如下配置：

```properties
# 密钥
jwt.secret = 6L6T5LqG5L2g77yM6LWi5LqG5LiW55WM5Y+I6IO95aaC5L2V44CC
# 签名算法：HS256,HS384,HS512,RS256,RS384,RS512,ES256,ES384,ES512,PS256,PS384,PS512
jwt.header.alg = HS256
#jwt签发者
jwt.payload.registerd-claims.iss = qiwen-cms
#jwt过期时间（单位：毫秒）
jwt.payload.registerd-claims.exp = 60 * 60 * 1000 * 24 * 7
#jwt接收者
jwt.payload.registerd-claims.aud = qiwenshare
```

### 添加 JWT 配置属性类

创建包 `com.shiyanlou.file.config.jwt`, 然后新增四个配置类如下：

JwtHeader 属性类用来定义 JWT 第一部分，代码如下：

```java
package com.shiyanlou.file.config.jwt;

import lombok.Data;

@Data
public class JwtHeader {
    private String alg;
    private String typ;
}
```

JwtPayload 属性类用来定义 JWT 第二部分，代码如下：

```java
package com.shiyanlou.file.config.jwt;

import lombok.Data;

@Data
public class JwtPayload {
    private RegisterdClaims registerdClaims;
}
```

```java
package com.shiyanlou.file.config.jwt;

import lombok.Data;

@Data
public class RegisterdClaims {
    private String iss;
    private String exp;
    private String sub;
    private String aud;
}
```

接下来我们将上面的属性类组织起来，形成一个完整的 JWT 配置类 JwtProperties, 该类使用 ConfigurationProperties 注解，其中的 prefix 参数是 jwt，可以读取 `application.properties` 配置文件中以 jwt 开头的配置，代码如下：

```java
package com.shiyanlou.file.config.jwt;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "jwt")
public class JwtProperties {
    private String secret;
    private JwtHeader header;
    private JwtPayload payload;
}
```

### 添加 JWT 工具类

创建包 `com.shiyanlou.file.util` 工具类，然后新增类 `JWTUtil.java`，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/bbec62096d82134ec60a63af9fe7079c-0/wm)

### 初始化

向 `JWTUtil.java` 文件中添加如下代码进行初始化：

```java
package com.shiyanlou.file.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.impl.DefaultClaims;

import org.apache.tomcat.util.codec.binary.Base64;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

import com.shiyanlou.file.config.jwt.JwtProperties;

import java.util.Date;

@Component
public class JwtUtil {

	@Resource
	JwtProperties jwtProperties;

}

```

### 添加生成密钥方法

在 `JWTUtil.java` 中创建 generalSecretKey 方法来生成密钥，生成的过程需要使用 JWT 的方法 SecretKeySpec 来生成密钥，该密钥在创建 JWT 和验证 JWT 的时候都会用到且相同，代码如下：

```java
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

import org.apache.tomcat.util.codec.binary.Base64;

public class JWTUtil {
    ...
    /**
     * 由字符串生成加密key
     * @return
     */
	private SecretKey generalKey() {
		// 本地的密码解码
		byte[] encodedKey = Base64.decodeBase64(jwtProperties.getSecret());
		// 根据给定的字节数组使用AES加密算法构造一个密钥
		SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
		return key;
	}
}
```

### 创建 JWT 方法

继续向 `JWTUtil.java` 文件中补充如下代码：

```java
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;

public class JWTUtil {
    ....

    /**
     * 创建jwt
     * @param subject
     * @return
     * @throws Exception
     */
	public String createJWT(String subject) throws Exception {

		// 生成JWT的时间
		long nowTime = System.currentTimeMillis();
		Date nowDate = new Date(nowTime);
		// 生成签名的时候使用的秘钥secret，切记这个秘钥不能外露，是你服务端的私钥，在任何场景都不应该流露出去，一旦客户端得知这个secret，那就意味着客户端是可以自我签发jwt的
		SecretKey key = generalKey();

        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine se = manager.getEngineByName("js");
        int expireTime = 0;
        try {
            expireTime =(int) se.eval(jwtProperties.getPayload().getRegisterdClaims().getExp());
        } catch (ScriptException e) {
            e.printStackTrace();
        }
		
		// 为payload添加各种标准声明和私有声明
		DefaultClaims defaultClaims = new DefaultClaims();
		defaultClaims.setIssuer(jwtProperties.getPayload().getRegisterdClaims().getIss());
		defaultClaims.setExpiration(new Date(System.currentTimeMillis() + expireTime));
		defaultClaims.setSubject(subject);
		defaultClaims.setAudience(jwtProperties.getPayload().getRegisterdClaims().getAud());

		JwtBuilder builder = Jwts.builder() // 表示new一个JwtBuilder，设置jwt的body
				.setClaims(defaultClaims)
				.setIssuedAt(nowDate) // iat(issuedAt)：jwt的签发时间
				.signWith(SignatureAlgorithm.forName(jwtProperties.getHeader().getAlg()), key); // 设置签名，使用的是签名算法和签名使用的秘钥

		return builder.compact();
	}
}
```

### 解析 JWT 方法

继续向 `JWTUtil.java` 文件中补充如下代码：

```java
import io.jsonwebtoken.Claims;

public class JWTUtil {
    ...
    /**
     * 解密jwt
     * @param jwt
     * @return
     * @throws Exception
     */
	public Claims parseJWT(String jwt) throws Exception {
		SecretKey key = generalKey(); // 签名秘钥，和生成的签名的秘钥一模一样
		Claims claims = Jwts.parser() // 得到DefaultJwtParser
				.setSigningKey(key) // 设置签名的秘钥
				.parseClaimsJws(jwt).getBody(); // 设置需要解析的jwt
		return claims;
	}
}
```

## 注册逻辑实现

创建包 `com.shiyanlou.file.service` ，在该包下面新增 `UserService.java` 类，该类为 Service 层的接口，然后继续创建包 `com.shiyanlou.service.impl`，在该包下新增 `UserServiceImpl.java` 类，用来作为 `UserService.java` 的实现类，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/99a3cc825fd4c2ec35f982471ec5f2a3-0/wm)

### 初始化 UserService.java

```java
package com.shiyanlou.file.service;

import com.shiyanlou.file.common.RestResult;
import com.shiyanlou.file.model.User;

public interface UserService {
    RestResult<String> registerUser(User user);
}
```

### 初始化 UserServiceImpl.java

```java
package com.shiyanlou.file.service.impl;

import java.util.List;
import java.util.UUID;

import javax.annotation.Resource;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.shiyanlou.file.common.RestResult;
import com.shiyanlou.file.mapper.UserMapper;
import com.shiyanlou.file.model.User;
import com.shiyanlou.file.service.UserService;
import com.shiyanlou.file.util.DateUtil;

import org.springframework.stereotype.Service;
import org.springframework.util.DigestUtils;
import org.springframework.util.StringUtils;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService{

    @Resource
    UserMapper userMapper;

    @Override
    public RestResult<String> registerUser(User user) {
        //判断验证码
        String telephone = user.getTelephone();
        String password = user.getPassword();

        if (!StringUtils.hasLength(telephone) || !StringUtils.hasLength(password)){
            return RestResult.fail().message("手机号或密码不能为空！");
        }
        if (isTelePhoneExit(telephone)){
            return RestResult.fail().message("手机号已存在！");
        }


        String salt = UUID.randomUUID().toString().replace("-", "").substring(15);
        String passwordAndSalt = password + salt;
        String newPassword = DigestUtils.md5DigestAsHex(passwordAndSalt.getBytes());

        user.setSalt(salt);

        user.setPassword(newPassword);
        user.setRegisterTime(DateUtil.getCurrentTime());
        int result = userMapper.insert(user);

        if (result == 1) {
            return RestResult.success();
        } else {
            return RestResult.fail().message("注册用户失败，请检查输入信息！");
        }
    }

    private boolean isTelePhoneExit(String telePhone) {
        LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
        lambdaQueryWrapper.eq(User::getTelephone, telePhone);
        List<User> list = userMapper.selectList(lambdaQueryWrapper);
        if (list != null && !list.isEmpty()) {
            return true;
        } else {
            return false;
        }

    }

}
```

向 `com.shiyanlou.file.util` 模块中添加 `DateUtil.java` 文件，并向其中写入如下代码：

```java
package com.shiyanlou.file.util;

import java.util.Date;

public class DateUtil {
    /**
     * 获取系统当前时间
     *
     * @return 系统当前时间
     */
    public static String getCurrentTime() {
        Date date = new Date();
        String stringDate = String.format("%tF %<tT", date);
        return stringDate;
    }
    
}
```

## 登录逻辑实现

### 登录接口方法

打开 `UserService.java` 类，新增如下接口方法：

```java
RestResult<User> login(User user);
```

### 登录接口实现

打开 `UserServiceImpl.java` 类，新增 `login` 接口方法实现：

```java
@Override
public RestResult<User> login(User user) {
    String telephone = user.getTelephone();
    String password = user.getPassword();

    LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
    lambdaQueryWrapper.eq(User::getTelephone, telephone);
    User saveUser = userMapper.selectOne(lambdaQueryWrapper);
    String salt = saveUser.getSalt();
    String passwordAndSalt = password + salt;
    String newPassword = DigestUtils.md5DigestAsHex(passwordAndSalt.getBytes());
    if (newPassword.equals(saveUser.getPassword())) {
        saveUser.setPassword("");
        saveUser.setSalt("");
        return RestResult.success().data(saveUser);
    } else {
        return RestResult.fail().message("手机号或密码错误！");
    }

}
```

## 使用 Java Web Token 实现用户登录认证

首先我们需要在 `UserController.java` 中创建两个接口，分别是登录和注册，前面我们已经将 `UserService.java` 接口和实现类完成了，这里我们只需要注入就可以使用了。

```java
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import javax.annotation.Resource;
import com.shiyanlou.file.service.UserService;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
    @Resource
    UserService userService;
    ...
}    
```

### 前台注册接口

创建包 `com.shiyanlou.file.dto` 用来存放接口请求参数，在该包下面创建 `RegisterDTO.java`，并初始化内容如下：

```java
package com.shiyanlou.file.dto;

import lombok.Data;

@Data
public class RegisterDTO {
    private String username;
    private String telephone;
    private String password;
}
```

接下来在 `UserController.java` 中，添加注册方法，代码如下：

```java
import com.shiyanlou.file.dto.RegisterDTO;
import com.shiyanlou.file.model.User;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
    ...
    @PostMapping(value = "/register")
    @ResponseBody
    public RestResult<String> register(@RequestBody RegisterDTO registerDTO) {
        RestResult<String> restResult = null;
        User user = new User();
        user.setUsername(registerDTO.getUsername());
        user.setTelephone(registerDTO.getTelephone());
        user.setPassword(registerDTO.getPassword());

        restResult = userService.registerUser(user);

        return restResult;
    }
}
```

### 前台登录接口

创建包 `com.shiyanlou.file.vo` 用来存放接口响应参数，在该包下面创建 `LoginVO.java`，并初始化内容如下：

```java
package com.shiyanlou.file.vo;

import lombok.Data;

@Data
public class LoginVO {
    private String username;
    private String token;
}
```

接下来在 `UserController.java` 中，添加登录方法，代码如下：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.shiyanlou.file.util.JWTUtil;
import com.shiyanlou.file.vo.LoginVO;

@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {

    @Resource
    JWTUtil jwtUtil;

    ...
    @GetMapping(value = "/login")
    @ResponseBody
    public RestResult<LoginVO> userLogin(String telephone, String password) {
        RestResult<LoginVO> restResult = new RestResult<LoginVO>();
        LoginVO loginVO = new LoginVO();
        User user = new User();
        user.setTelephone(telephone);
        user.setPassword(password);
        RestResult<User> loginResult = userService.login(user);

        if (!loginResult.getSuccess()) {
            return RestResult.fail().message("登录失败！");
        }

        loginVO.setUsername(loginResult.getData().getUsername());
        String jwt = "";
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            jwt = jwtUtil.createJWT(objectMapper.writeValueAsString(loginResult.getData()));
        } catch (Exception e) {
            return RestResult.fail().message("登录失败！");
        }
        loginVO.setToken(jwt);

        return RestResult.success().data(loginVO);
    }
}
```

### token 校验接口

用户登录成功后，可以调用该接口来获取登录状态，判断 token 是否失效，保证前后台登录状态一致。

继续向 `UserController.java` 文件中添加如下代码：

```java
import org.springframework.web.bind.annotation.RequestHeader;
import io.jsonwebtoken.Claims;

@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
    ...
    @GetMapping("/checkuserlogininfo")
    @ResponseBody
    public RestResult<User> checkToken(@RequestHeader("token") String token) {
        RestResult<User> restResult = new RestResult<User>();
        User tokenUserInfo = null;
        try {

            Claims c = jwtUtil.parseJWT(token);
            String subject = c.getSubject();
            ObjectMapper objectMapper = new ObjectMapper();
            tokenUserInfo = objectMapper.readValue(subject, User.class);

        } catch (Exception e) {
            log.error("解码异常");
            return RestResult.fail().message("认证失败");

        }

        if (tokenUserInfo != null) {

            return RestResult.success().data(tokenUserInfo);

        } else {
            return RestResult.fail().message("用户暂未登录");
        }
    }
}
```

上面这段代码，如果 token 不正确，或者 token 过期，就会导致解码失败，返回认证失败，如果能够正确解析，那么就会返回成功。

## 实验总结

通过本节实验，我们需要掌握登录方式的基本思路和原理，并能够熟练运用 JWT 来进行登录认证，目前已经完成了用户登录和注册接口的实现，而在下一节实验中我将带领大家完成 Swagger 3 的搭建，生成接口文档，并对本节实验接口做一个验证。

本次实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/8842/code5.zip
```
