---
title: 编写自己的Spring Boot Starter
date: 2020-08-16 11:28:57
categories: Spring
tags:
  - Spring
---

Spring Boot 对比 Spring MVC 最大的优点就是使用简单，约定大于配置。开发Spring Boot应用的时候，只需要引入各种类型的starter，就能实现对应的功能。Spring Boot 的 starters有官方提供的，也有第三方开源出来。

使用 Spring Boot 的功能组件（例如 spring-boot-starter-actuator、 spring-boot-starter-data-redis 等）的步骤非常简单，用著名的把大象放冰箱的方法来概括的话，有以下三步就可以完成组件功能的使用：<!--more-->

- pom文件引入对应的starter和其他包
- 在应用配置文件中加入相应的配置，配置都是组件约定好的，需要查看官方文档或者相关说明
- 使用组件提供的相关接口来开发业务功能

那么如果我们想要自己实现自己的Starter，需要做些什么呢？下面内容将会回答这个问题，同时通过该学习还能更好的理解Spring Boot的自动装配思想。

## Starter基本原理

首先介绍一下Starter的基本原理，可分为以下几个步骤：

- SpringBoot 在启动时会去依赖的starter包中寻找 `resources/META-INF/spring.factories `文件，然后根据文件中配置的jar包去扫描项目所依赖的Jar包。
- 根据 `spring.factories`配置加载对应的`AutoConfigure`类
- 根据`AutoConfigure`类中`@Conditional`注解的条件，进行自动配置并将Bean注入Spring Context上下文当中，同时可根据`@EnableConfigurationProperties`注解，决定配置属性的字段和默认值

下面，我们就根据以上原理，进行一个简单的Starter工程的开发，实现一个读取配置并打印的简单service。

## Starter工程编写

### 代码结构

首先来看一下这个工程的代码结构：

```bash
.
├── pom.xml
├── spring-boot-starter.iml
└── src
    └── main
        ├── java
        │   └── com
        │       └── mx
        │           └── demo
        │               ├── DemoApplication.java
        │               ├── config
        │               │   └── MxAutoConfigure.java
        │               ├── properties
        │               │   └── MxProperties.java
        │               └── service
        │                   └── MxService.java
        └── resources
            ├── META-INF
            │   └── spring.factories
            └── application.properties
```

### 添加maven依赖

首先看`pom.xml`文件的写法：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.mx</groupId>
    <artifactId>mx-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
    </dependencies>

</project>

```

首先Spring Boot的父依赖是需要的，保持原有写法即可。

然后是本项目的配置：groupId可以自己任取；artifactId则是Starter的名字，需要遵循Spring Boot的规范，Spring 官方 Starter通常命名为`spring-boot-starter-{name}` 如 `spring-boot-starter-web`，非官方Starter命名应遵循`{name}-spring-boot-starter`的格式。版本号也是自己写一个就行，后面要用到。

实现 Starter 主要依赖自动配置注解，所以要在pom中引入自动配置相关的两个jar包，如上所示。

**踩坑**：大多数通过构建工具新建的Spring Boot项目，会存在build依赖，配置如下：

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

这一段的主要作用是将工程编译为可执行的jar包，作为Starter则用不到，应该删去。反之会出错，导致引用该Starter的工程无法找到对应的service类等问题。

### 创建`spring.factories`文件

在 resource/META-INF 目录下创建名称为`spring.factories`的文件，为什么在这里？当Spring Boot启动的时候，会在classpath下寻找所有名称为spring.factories 的文件，然后运行里面的配置指定的自动加载类，将指定类中的相关bean初始化。

```bash
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.mx.demo.config.MxAutoConfigure # 这里是自动配置类的路径
```

如果有多个自动配置类，用逗号分隔换行即可。

### 实现属性配置类

```java
// 绑定配置文件中以“example.mx”为前缀的配置项
@ConfigurationProperties("example.mx")
public class MxProperties {
    private String host;
    private String port;

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public String getPort() {
        return port;
    }

    public void setPort(String port) {
        this.port = port;
    }
}
```

配置类很简单，只有两个属性，一个host，一个port。配置参数以example.mx作为前缀。

### 实现业务Service类

```java
public class MxService {
    private String host;
    private String port;

    public MxService(MxProperties mxProperties) {
        this.host = mxProperties.getHost();
        this.port = mxProperties.getPort();
    }

    public void print() {
        System.out.println(this.host + ":" + this.port);
    }
}
```

严格来讲，这个业务功能类不应该放在Starter中，应该放在单独的jar包里，但是此处的demo非常简单，也就在这里写了。

### 编写自动配置类

现在开始编写上面配置中提到的`MxAutoConfigure`类，它是实现starter最核心的功能，主要用来初始化Starter中的相关bean，代码如下：

```java
// 表示这是一个配置类
@Configuration
// 只有在classpath中找到MxService类的情况下，才会解析此自动配置类，否则不解析。
@ConditionalOnClass(MxService.class)
// 启用名为MxProperties的配置类，将其注入到容器
@EnableConfigurationProperties(MxProperties.class)

public class MxAutoConfigure {
    @Autowired
    private MxProperties mxProperties;

    @Bean
    // 只有当上下文中不存在MxService实例的时候，才会执行注入，确保是单例模式
    @ConditionalOnMissingBean(MxService.class)
    // 当应用配置文件中有相关的配置才会执行其所注解的代码块。
    @ConditionalOnProperty(prefix = "example.mx", value = "enabled", havingValue = "true")
    MxService mxService() {
        return new MxService(mxProperties);
    }
}
```

用到的几个注解的含义已经通过注释给出了，容易看出该类的作用：把配置文件的属性值装配到MxProperties类中，然后又将这些属性提供给MxService类使用，最后把MxService作为bean注入到容器中，仅此而已。

### 打包

通过maven命令将这个Starter安装到本地maven仓库，后续就能直接引用。

```bash
maven install
```

## 测试

现在新建一个Spring Boot工程来使用刚发布的Starter。

首先是pom文件（仅列出依赖部分）：

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--引入自己的starter，因为是自己写的，因此需要填写版本号-->
        <dependency>
            <groupId>com.mx</groupId>
            <artifactId>mx-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
</dependencies>
```

接下来填写application.properties配置项：

```properties
server.port=8080

example.mx.enabled=true  # 开启才能生效
example.mx.host=127.0.0.1
example.mx.port=8888
```

最后编写controller类，调用引入的service方法：

```java
@RestController
public class MyController {
    @Autowired
    private MxService mxService;

    @RequestMapping(value = "/health", method = RequestMethod.GET)
    public String health() {
        return "ok";
    }

    @RequestMapping(value = "/print", method = RequestMethod.GET)
    public void print() {
        mxService.print();
    }
}
```

启动服务，访问`http://localhost:8080/print`接口，控制台打印如下内容：

```
127.0.0.1:8888
```

至此，整个Starter的功能就被完整实现了。