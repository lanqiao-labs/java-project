---
show: step
version: 1.0
enable_checker: true
---

# Spring Boot 整合 MyBatis 和 MyBatis-Plus

## 实验介绍

在此之前已经完成了 Spring Boot 项目搭建，并且实现了数据库、日志的集成，这节实验将带领大家在此基础之上进行 MyBatis 和 MyBatis-Plus 的集成。

#### 知识点

- MyBatis
- MyBatis-Plus
- JUnit 单元测试

#### 开发计划

- 开发内容：MyBatis 及 MyBatis-Plus的介绍、集成及简单使用。
- 开发耗时：实验预计完成时间为 1~2 小时
- 开发难点：
1. MyBatis 各种标签的熟练使用
2. 集成 MyBatis-Plus 基本配置
3. 熟练掌握 MyBatis 及 MyBatis-Plus 的使用，并提升开发效率。

## 基本概念

#### MyBatis 介绍

MyBatis 是一个 Dao 层框架，它支持普通 SQL 查询，并且支持存储过程和高级映射。MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及对结果集的检索封装。MyBatis 可以使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的 POJO（Plain Old Java Objects，普通的 Java 对象）映射成数据库中的记录。

#### MyBatis-Plus 介绍

MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

#### JUnit 单元测试

JUnit 是一个 Java 编程语言的单元测试框架。它的存在使得我们能够针对性的对项目的关键代码进行测试，每个测试用例称为一个单元，通常我们用它来测试接口或者方法。

## Spring Boot 集成 MyBatis

### 添加依赖

打开 `pom.xml` 文件，添加如下依赖：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```

### 添加配置

打开 `application.properties` 配置文件，添加下面配置，这两行用来指定 mybatis 的配置文件。

```properties
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
```

### 启动类添加 Mapper 扫描包路径

在 Spring Boot 启动类 `Application.java` 中添加 `@MapperScan` 注解，用来指定要扫描的 Mapper 文件夹：

```java
package com.shiyanlou.file;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@MapperScan("com.shiyanlou.file.mapper")
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

### 开发 Mapper 接口类

在上面 `@MapperScan` 注解扫描路径下，创建 Mapper 接口类，Mapper 接口主要是用来生产 Sql 调用接口，供上层调用，一般来说每个实体类都应该对应一个 Mapper 接口，因此下面我将会创建三个 Mapper 接口，并以 UserMapper 接口为例，给出一个使用案例。

在 `com.shiyanlou.file.mapper` 包下创建三个接口，分别为 `FileMapper`，`UserfileMapper` 和 `UserMapper`，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/00f96d0e9dbfab27060f996c4e251ce8-0/wm)

#### FileMapper 接口

```java
package com.shiyanlou.file.mapper;

public interface FileMapper {

}
```

#### UserfileMapper 接口

```java
package com.shiyanlou.file.mapper;

public interface UserfileMapper {

}
```

#### UserMapper 接口

```java
package com.shiyanlou.file.mapper;

import com.shiyanlou.file.model.User;
import java.util.List;

public interface UserMapper {
    void insertUser(User user);
    List<User> selectUser();
}
```

上面创建的 UserMapper 接口中，新建了两个操作，分别是新增用户和查询，下面来创建它们对应的映射文件。

### 添加映射文件

在 `resources` 目录下新增 `mybatis` 目录，并在该目录下创建配置文件 `mybatis-config.xml` ，该配置文件主要用来指定 MyBatis 基础配置文件和实体类映射文件的地址，你也可以根据自己的习惯进行修改，如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
	<typeAliases>
		<typeAlias alias="Integer" type="java.lang.Integer" />
		<typeAlias alias="Long" type="java.lang.Long" />
		<typeAlias alias="HashMap" type="java.util.HashMap" />
		<typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap" />
		<typeAlias alias="ArrayList" type="java.util.ArrayList" />
		<typeAlias alias="LinkedList" type="java.util.LinkedList" />
	</typeAliases>
</configuration>
```

继续在 `mybatis` 目录下创建 `mapper` 目录，并创建 xml 格式的文件，其名称与上面创建的 mapper 接口相对应，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/3dc7f358f6c63d136d227b527c1b64bb-0/wm)

上面 `UserMapper.java` 接口中创建了两个操作，因此这里需要在它对应的 `UserMapper.xml` 中创建其 SQL 实现，初始化内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.shiyanlou.file.mapper.UserMapper">
    <insert id="insertUser" parameterType="com.shiyanlou.file.model.User">
        insert into user (username, password, telephone)
        value (#{username}, #{password}, #{telephone})
    </insert>

    <select id="selectUser" resultType="com.shiyanlou.file.model.User">
        select * from user
    </select>
</mapper>
```

上面这段 xml 脚本中：

- `namespace` 指定了它的 Mapper 接口，因此它的内容就是 Mapper 接口的包名；
- `insert` 标签和 `select` 标签分别对应 Mapper 接口中的插入和查询操作，因此这里的标签 id 与 Mapper 接口的方法名是一致的。

到此为止整个 MyBatis 的代码调用流程就已经实现，接下来我们对代码进行测试。

### 测试

#### 创建测试类

我们采用单元测试代码来进行验证，在工程 `src` 目录下有两个文件目录，分别是 `main` 和 `test`：

- `main` 是主工程存放目录；
- `test` 是测试工程存放路径。

在 `test` 的 `java` 目录下创建包，包名为： `com.shiyanlou.file`，这里需要注意：这个包名必须和主工程的包名相同，否则在执行测试代码的时候会报错，然后在包名下面创建一个测试类，名为 `ApplicationTest.java`，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/32fd2bce4306057fb04d99f0accf7c5d-0/wm)

然后输入下面的代码进行测试：

```java
package com.shiyanlou.file;

import com.shiyanlou.file.mapper.UserMapper;
import com.shiyanlou.file.model.User;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
public class ApplicationTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void test1() {
        User user = new User();
        user.setUsername("用户名1");
        user.setPassword("密码1");
        user.setTelephone("手机号1");
        userMapper.insertUser(user);
        System.out.println("数据库字段查询结果显示");
        List<User> list = userMapper.selectUser();
        list.forEach(System.out::println);
    }

}
```

以上代码通过调用 MyBatis 接口，给用户数据库插入了一条数据，然后查询并打印结果。

> 如果以上代码存在属性爆红的情况，应该是你的 IDE 未安装 Lombok 插件，VSCode 可通过如下方式安装 Lombok 插件。
![图片描述](https://doc.shiyanlou.com/courses/8842/1557563/7d332a3e73f72118ff2cec888b895300-0/wm)
安装完成之后点击右下角重启按钮即可生效。

#### 验证测试类

记得启动环境中的 MySQL 数据库：

```bash
sudo service mysql start
```

上面的代码首先在 `user` 表中插入一条用户信息，然后查询，如果能查询到插入的信息，说明插入成功。

点击测试类方法旁边的运行按钮，就可以开始执行测试方法，执行完成之后便可查看结果，如下图：

![图片描述](https://doc.shiyanlou.com/courses/8842/1557563/0d243928912b4cc89ccdb8d8ca5edbb8-0/wm)

## Spring Boot 集成 MyBatis-Plus

### 添加依赖

向 `pom.xml` 文件中继续添加如下依赖：

```xml
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.4.1</version>
</dependency>
```

上面这段 xml 脚本引入了 1 个依赖，其中 `mybatis-plus-boot-starter` 是 `MyBatis-Plus` 的依赖包，引入之后就可以使用 MyBatis-Plus 的所有功能。

### 添加配置

如果我们不设置 MyBatis-Plus 默认的驼峰式编码，在 MyBatis-Plus 则会默认把驼峰式编码映射为下划线格式，比如实体类种属性 userId 映射到数据库字段 user_id, 这种下划线格式的字段会导致代码报错，所以我们需要关闭驼峰命名规则映射，在 `application.properties` 配置文件中添加如下配置：

```properties
mybatis-plus.mapper-locations=classpath:mybatis/mapper/*.xml

mybatis-plus.configuration.map-underscore-to-camel-case=false
```

### 添加注解

在之前已经创建的三个实体类中，需要添加 MyBatis-Plus 相关注解，这里需要用到的注解有两个，分别是 `@TableName` 和 `@TableId`。

#### 注解说明
|注解|描述|
|-|-|
|@TableName(“表名”)|实体类添加，如果不添加，会按照默认规则进行表明的映射，比如UserTable->user_table|
|@TableId(type = IdType.AUTO)|用来标注实体类主键|

#### 实体类添加注解

向 `UserFile.java` 文件中添加如下代码：

```java
...
import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.TableId;

...
@TableName("userfile")
public class UserFile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @TableId(type = IdType.AUTO)
    @Column(columnDefinition = "bigint(20) comment '用户文件id'")
    private Long userFileId;

    ...
```

向 `User.java` 文件中添加如下代码：

```java
...
import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.TableId;

...
@TableName("user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @TableId(type = IdType.AUTO)
    @Column(columnDefinition = "bigint(20) comment '用户id'")
    private Long userId;

    ...
```

向 `File.java` 文件中添加如下代码：

```java
...
import com.baomidou.mybatisplus.annotation.TableName;
import com.baomidou.mybatisplus.annotation.TableId;

...
@TableName("file")
public class File {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @TableId(type = IdType.AUTO)
    @Column(columnDefinition="bigint(20) comment '文件id'")
    private Long fileId;

    ...
```



### Mapper 接口继承 BaseMapper<T>

这里我们以 `UserMapper.java` 为例，进行讲解。打开 `UserMapper.java` 并继承 `BaseMapper`，代码如下：

```java
package com.shiyanlou.file.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.shiyanlou.file.model.User;
import java.util.List;

public interface UserMapper extends BaseMapper<User>{
    void insertUser(User user);
    List<User> selectUser();
}
```

### 测试

下面来验证集成结果，首先在 `user` 表中插入一条用户信息，然后查询，查看查询结果是否插入成功。

#### 新增测试方法

向 `ApplicationTest.java` 文件中补充如下代码：

```java
@Test
public void test2() {
    User user = new User();
    user.setUsername("用户名2");
    user.setPassword("密码2");
    user.setTelephone("手机号2");
    userMapper.insert(user);
    List<User> list = userMapper.selectList(null);
    System.out.println("数据库字段查询结果显示");
    list.forEach(System.out::println);
}
```

#### 查看测试结果

如下图我们使用 MyBatis-Plus 提供的方法可以很容易的进行插入和查询，而且不用写一行 SQL 及其各种映射文件，点击执行，查看结果如下图：

![图片描述](https://doc.shiyanlou.com/courses/8842/1557563/26a7241a1132a36fc8341ecc010f356c-0/wm)

## 实验总结

本节实验带领大家完成了 MyBatis 和 MyBatis-Plus 的集成，并且通过一个简单的例子完成了 Java 代码对数据库的操作，我们可以发现使用 MyBatis-Plus 能够大大提高工作效率，这便是它存在的意义。

本次实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/8842/code3.zip
```
