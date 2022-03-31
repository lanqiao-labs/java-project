---
show: step
version: 1.0
enable_checker: true
---

# Spring Boot 项目的搭建

## 实验介绍

本实验主要是介绍 Spring Boot 的一些基础知识和搭建流程，并集成常用的 MySQL 和日志，在项目搭建的过程中，我会尽可能的给大家讲清楚每个步骤、每行代码的基本含义。如果你对 Spring Boot 项目已经有一定的了解，可以在学习的过程中适当跳过一些简单的步骤。

#### 知识点

- 项目介绍
- Spring Boot 基本概念介绍
- 开发环境搭建
- 项目搭建流程
- 项目的启动和停止
- 集成日志和数据库

#### 开发计划

- 开发内容：本次实验带领大家从零开始，完成 Spring Boot 项目的搭建，并完成 MySQL 和日志的集成。
- 开发耗时：实验预计完成时间为 1~2 小时
- 开发难点：无

## 项目介绍

网盘项目一直以来都是一个比较热门的话题，可能每个人都希望能够拥有自己的网盘，那么本实验课程，就手把手教大家如何搭建一个网盘项目，另外如果你想要快速提高自己的开发水平，那么这个项目就非常适合你。

本课程我们将使用 Spring Boot 和 Vue 这两个热门开源框架来进行开发。在学习完本次课程之后，你不仅能够达到熟练使用 Spring Boot 和 Vue ，而且将会具备独立开发一个完整项目的能力，包括前端、后台、运维、数据库建模、系统设计等。因此，项目讲解最终目的并不是简单的教大家完成如何开发网盘，而是要教会大家学会如何思考问题，如何设计才能提高系统的可扩展性，面对一个新需求如何进行分析、建模。往往一个好的底层设计，不管需求如何改变，都是能满足系统要求，而一旦设计出了问题，那么随着后面需求复杂度的提升，其结果无外乎是代码冗余越来越多，最终推翻重来。

项目的开发模式为前后端分离模式，前后台分离的好处就是我们可以对前后台独立开发和部署，这种模式是以后发展的趋势，在前面实验课程主要以 Spring Boot 为重点展开讲解，后面实验课程教大家用 Vue-Cli3 来制作前台页面并请求后台接口。接下来我们就会从最基本的概念讲起，教大家一步一步的去完成项目的开发。

## Spring Boot 基本概念介绍

Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化新 Spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。也就是说 Spring Boot 集成了很多第三方库，并且都给了一套默认的配置，这样我们在使用 Spring Boot 时仅仅只需要很少的配置即可。Spring Boot 的迭代升级是很快的，到目前为止最新的发布版本是 2.4.1，本课程也将会基于 2.4.1 版本展开讲解。

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/c4ed98dcf8e2b87e878d0c6752f46e9a-0/wm)

#### SpringBoot 所具备的特征有：

1. 可以创建独立的 Spring 应用程序，并且基于其 Maven 或 Gradle 插件，可以创建可执行的 JAR 和 WAR。
2. 内嵌 Tomcat 或 Jetty 等 Servlet 容器。
3. 提供自动配置的“starter”项目对象模型（POMS）以简化 Maven 配置。
4. 尽可能自动配置 Spring 容器。
5. 提供准备好的特性，如指标、健康检查和外部化配置。
6. 不需要 XML 配置。

## Spring Boot 项目搭建

为了快速开始，我们直接使用课程开发环境，所需要的软件在环境里面已经默认为我们安装好了，可以通过以下命令来检测环境：

#### JDK 环境检测

使用 `java -version` 命令可以查看当前环境是否安装 JDK，如果已经安装 JDK，那么使用该命令可以显示 JDK 版本，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3472/600404/22d5838d13914bc778319d76667fe78d-0/wm)

从上图可以看出，环境所使用的 JDK 版本是 11.0.9.1。

#### maven 环境检测

maven 是一个项目构建和管理的工具，使用 `mvn -version` 命令可以检查当前环境是否安装，如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/1f14c7f0d5194c383e584a89deb91deb-0/wm)

从上图可以看出，当前 maven 工具版本是 3.6.3，因为 maven 工具在编译的过程中是需要用到 JDK 的，所以这里也给出了 Java 版本相关信息。

#### MySQL 环境检测

使用 `sudo service mysql start` 命令启动 MySQL，启动成功之后可以使用 `mysql -u root` 命令进行连接，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/600404/4d6723a10d25f709fec50d9b0ba05e6f-0/wm)

从上图连接信息可以看出，环境使用的是 MariaDB 数据库，MariaDB 是 MySQL 的一个分支，语法、API 和命令行完全兼容 MySQL，它们的区别就是 MariaDB 是由开源社区在维护。熟悉完环境之后，下面开始演示具体项目的创建。

### 创建项目

一般来说，我们创建 Spring Boot 项目有好几种可选的方式，如下：

1. 通过 [官网](http://start.spring.io/) 提供的 Spring initializr 去创建。
2. 借助 IDEA 开发工具提供的 Spring initializr 进行创建，创建流程与第一种类似。
3. 创建一个普通的 maven 工程，然后自己手动引入 Spring Boot 的依赖包。

在本实验中，为了便于讲解，我们选择第三种，通过手动方式一步一步搭建一个网盘项目。

打开环境，会看到一个编辑器界面，左侧 `PROJECT` 是环境所在的目录，我们可以直接在该目录下开始搭建网盘项目，接下来我将通过两种方式创建项目，大家可以根据喜好来选择其中一种来开始网盘项目搭建。

#### 第一种方式

在 `PROJECT` 目录下创建一个新的目录 `qiwen-file`，创建完目录，敲击回车，目录就创建好了，可以将该目录作为我们项目的根目录，开始创建 Spring Boot 项目。

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/7b2b6143bec0c3876cd6adb66e0f3160-0/wm)

因为我们所使用的 Spring Boot 项目是基于 maven 构建的，因此第一步需要创建一个 maven 工程，maven 工程标准目录结构如下：

```txt
qiwen-file
    |---src
    |    |---main
    |    |    |---java
    |    |    |---resources
    |    |
    |    |---test
    |         |---java
    |         |---resources
    |
    |---target
    |---pom.xml
```

手动创建上面的 maven 目录结构，创建完成的效果图如下：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/b0be587145ad5912765ab6f12dc8ed5c-0/wm)

#### 第二种方式

如果觉得第一种方式手动创建这么多目录过于麻烦，可以在命令行输入 maven 命令自动生成，执行命令后，maven 就会帮我们创建好一个 maven 工程，代码如下：

```bash
mvn archetype:generate -DgroupId=com.shiyanlou.file -DartifactId=qiwen-file -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeCatalog=local -DinteractiveMode=false
```

执行完上述命令后，一个普通的 maven 工程就创建好了，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/9a8643a96e9bdbed8f0047f16ba7676f-0/wm)

从上图可以看出，maven 为我们默认生成了 `App.java` 类和 `AppTest.java` 类，其中 `App.java` 是一个普通的 Java 类，里面仅包含一个 main 方法，`AppTest.java` 是一个 Java 测试类，它的运行依赖于 junit 包，而这个 junit 包的依赖 maven 已经为我们生成在 pom.xml 中，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/02734685f694959b5b3e847644245c21-0/wm)

由于我们现在要搭建的是一个 Spring Boot 项目，而非一个普通的 maven 工程，这种直接引入 junit 包的方式并不适合，Spring Boot 已经为我们管理了这些包的依赖，所以这里将 `App.java` 和 `AppTest.java` 这两个文件删除。

另外，我们发现目录结构中缺失了 `resources` 目录，按照上面的目录结构规划我们需要在 `/src/main/` 和 `/src/test/` 目录下创建 `resources` 目录，`resources` 目录在 Spring Boot 中是用来存放配置文件和静态资源的，然后一个 Spring Boot 的目录结构的创建就完成了。

> 注意 Spring Boot 目录下存放静态资源的目录是 `resources` 而不是 `resource`，检查一下是否拼写正确，避免报错。

### 修改 pom.xml

在项目的根路径下有一个名为 `pom.xml` 的文件，pom 文件是 maven 构建的根本，它约定了项目构建所需要的相关信息和依赖，打开 `pom.xml` 文件，初始化如下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.shiyanlou.file</groupId>
	<artifactId>qiwen-file</artifactId>
	<version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

</project>
```

上面的 xml 文件定义了项目的最基本的信息，此时一个最简单 maven 工程就已经搭建完毕。我们可以试着执行一下 maven 构建，看是否能够成功。在命令行界面，通过 cd 命令切换目录到项目根路径，命令如下：

```bash
cd qiwen-file
```

在命令行目录下，输入 `mvn install` 尝试构建，如果构建成功，会出现 `BUILD SUCCESS` 字样，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/600404/4cafbdd3d7b0eedcb4838c9e058830c4-0/wm)

## 添加 Spring Boot 依赖

#### 增加父 pom 依赖

在 `pom.xml` 中的 `project` 节点下，增加如下依赖。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
```

`spring-boot-starter-parent` 是 Spring Boot 项目的父 pom，它里面定义了 Spring Boot 所使用的依赖库版本等信息，但它本身并没有任何依赖。

#### 项目依赖

在 `pom.xml` 中的 `project` 节点下，继续增加如下依赖。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

上面这段 xml 脚本增加了两个依赖，其中：

`spring-boot-starter-web` 依赖项是 Spring Boot 的核心，引入该依赖之后 Spring Boot 将会使用 SpringMVC 构建一个 Web 应用程序，且使用 Tomcat 作为默认的嵌入式容器。

`spring-boot-starter-test` 包含了很多测试依赖项的类库，比如 JUnit Jupiter，Hamcrest 和 Mockito，引入这个依赖之后就可以直接进行单元测试类的编写。

上面这两个依赖项是 Spring Boot 最基本的依赖，可以看到在引用的时候并没有版本号定义，其实 Spring Boot 的父 pom 的三方件管理库已经为我们定义好了，这里我们只需要直接使用即可。一般来说，我们在构建项目的时候，也应该这样做，将所有引用的第三方 jar 版本统一定义到最上层的父 pom，做到统一管理和配置，比如 Spring Boot 这种，我们在后期升级 Spring Boot 版本时，只需要更改 parent 这一处引入版本，整个项目的所有引用依赖都会被升级 。

## 创建 Spring Boot 启动类

#### 创建包

在 `src/main/java` 下，创建一个包，一般是根据项目命名，这里我创建一个名为 `com.shiyanlou.file` 的包，以后所有的代码就都放到这个包下面。

#### 创建启动类

在 `com.shiyanlou.file` 包下创建一个名为 Application 的 Java 类，创建完成的示例图如下：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/503ae3588db55a9db0ab0d01de8572ef-0/wm)

#### 编写启动类内容

在 `Application.java` 的类中添加内容如下：

```java
package com.shiyanlou.file;

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}

```

- `@SpringBootApplication` 是 Spring Boot 的核心注解，顾名思义，它用来声明该项目是一个 Spring Boot 应用。
- `main` 方法，Java 应用程序一般使用 main 方法来作为整个程序的入口，Spring Boot 也遵循了这一标准，我们的主方法通过调用 run 来委托 Spring Boot 的 SpringApplication 类。SpringApplication 引导我们的应用程序，启动 Spring，而 Spring 又启动自动配置的 Tomcat web 服务器。我们需要传递 `Application.class` 作为 run 方法的参数来告诉 SpringApplication 哪个是主要的 Spring 组件。args 数组也可以接收任何命令行参数。

## 项目启动和停止

#### 启动项目

在命令行执行如下命令就可以启动项目:

```bash
mvn spring-boot:run
```

> 也可以直接运行 `Application.java` 类来启动 Spring Boot 项目。

#### 访问项目

点击右侧 `Web 服务` 按钮，在浏览器出现如下内容，就说明项目启动成功：

![图片描述](https://doc.shiyanlou.com/courses/3472/600404/c9f6ce461201925f1bc1ba8b8bf78bd4-0/wm)

> 这里需要说明的是，项目在启动的时候是没有指定端口号的，所以 Spring Boot 会默认使用 `8080` 端口启动服务，而 Web 服务也指向的是 `8080` 端口，因此这里没有配置就可以直接进行访问了。

#### 停止项目

在命令行按下 `Ctrl + c` 停止项目，到此为止，一个最简单的 Spring Boot 项目就已经开发完成了。

## 配置数据库连接

#### 添加依赖

打开 `pom.xml` 文件，在 `dependencies` 节点下添加如下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 添加配置

在 `src/main/resources` 目录下创建 `application.properties` 文件，该文件是 Spring Boot 默认读取的配置文件，创建完成之后打开 `application.properties` 文件，添加 MySQL 连接的相关配置，具体代码如下：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/file
spring.datasource.username=root
spring.datasource.password=
```

其中：

- `spring.datasource.url` 配置了数据库连接的 url，Spring Boot 会根据它的属性值来判断数据源并进行连接；
- `spring.datasource.username` 指定数据库的用户名；
- `spring.datasource.password` 配置数据库的密码。

### 创建数据库

使用 `sudo mysql -u root` 连接数据库环境，会进入到数据库命令行模式，执行创建数据库脚本，命令如下：

```sql
create database file default charset utf8 collate utf8_general_ci;
```

这样一个数据库就创建好了，可以使用 `show databases` 命令就可以看到创建的数据库了，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/d4d94bd4f4d3b6fdfbbdcabe538cdcda-0/wm)

## 配置日志

因为 Spring Boot 内部已经集成了 Logback 日志模块，所以，我们只在 `application.properties` 配置文件中添加 log 日志的相关信息即可。

#### 添加配置

```properties
# 配置日志保存路径
logging.file.name=/home/project/log/web.log
# 配置日志级别
logging.level.root=info
```

为了便于查看日志，我们将日志路径设置到环境工程路径下：`/home/project`，在实际开发过程中，可以根据需要调整日志级别，配置完成之后，重新启动项目，就可以在对应目录下看到生成日志文件了，如下图：

![图片描述](https://doc.shiyanlou.com/courses/3472/1557563/6f6651321979bfa684a038bd08797570-0/wm)

在大型项目中，如果需要更加精细的控制每个包的日志级别，并且将不同级别的日志打印到不同的日志文件，可以使用 `logback.xml` 进行配置，这里就不过多介绍，感兴趣的可以自行学习。

## 实验总结

本实验向大家演示了 Spring Boot 搭建流程，希望大家能够在搭建的过程中，了解到它每一步的原理。这样在以后的开发过程中才能够得心应手。

本次实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/8842/code1.zip
```
