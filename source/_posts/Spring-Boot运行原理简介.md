---
title: Spring Boot运行原理简介
date: 2020-08-15 15:39:25
categories: spring
tags: -spring
---

## Maven依赖

一个最基本的Spring Boot项目一般是从`pom.xml`文件开始研究的，打开项目的`pom.xml`文件，一项项看着走：

1.父依赖<!--more-->

```xml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.5.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
</parent>
```

点进去后发现，还能找到下一个父依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

这里才是真正管理SpringBoot应用里面所有依赖版本的地方，SpringBoot的版本控制中心；我们引入的依赖如果不在版本控制中，那么才需要手动写上版本号。

2.starter

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

starter就是spring boot的场景启动器，我们可以理解为一个可插拔式的插件，starter和普通jar包的区别在于，它能够实现自动配置，和Spring Boot无缝衔接，从而节省我们大量开发时间。

## 主启动类

我们可以很轻松找到主启动类，里面代码也很简单：

```java
//@SpringBootApplication 来标注一个主程序类
//说明这是一个Spring Boot应用
@SpringBootApplication
public class SpringbootApplication {
   public static void main(String[] args) {
     //以为是启动了一个方法，没想到启动了一个服务
      SpringApplication.run(SpringbootApplication.class, args);
   }
}
```

可以看到起作用的其实就是`@SpringBootApplication`这个注解，作用：标注在某个类上说明这个类是SpringBoot的主配置类 。这是一个复合注解，点进去可以发现其源码如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    .....
}

```

看似包含了一堆注解，其实最重要的也就只有三个：

- `@SpringBootConfiguration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

我们现在一个一个看：

### @SpringBootConfiguration

`@SpringBootConfiguration`也是一个复合注解，可以继续往下点，结果如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}
```

发现有个熟悉的注解：`@Configuration`，其实`@SpringBootConfiguration`继承自`@Configuration`。二者功能也一致，标注当前类是配置类（配置类就是对应Spring的xml 配置文件）。

启动类标注了`@Configuration`之后，本身其实也是一个IoC容器的配置类，意味着将启动类中声明的一个或多个以@Bean注解标记的方法的实例纳入到spring容器中。

### @EnableAutoConfiguration

先说结论，`@EnableAutoConfiguration`是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到Ioc容器。它也是一个复合注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

其中最重要的是：`@Import({AutoConfigurationImportSelector.class})`，借助引入`AutoConfigurationImportSelector`这个类，`@EnableAutoConfiguration`可以帮助Springboot应用将所有符合条件的@configuration都加载到当前的SpringBoot创建并使用的Ioc容器。

AutoConfigurationImportSelector ：顾名思义，就是自动配置导入选择器，那么它会导入哪些组件的选择器呢？继续点开源码，注意到类中有如下方法：

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```

从`loadFactoryNames()`方法往下一路追踪，可以发现`loadFactoryNames()`读取了ClassPath下面的 META-INF/spring.factories 文件，这就是spring自动配置的幕后英雄。

spring.factories是一个典型的java properties文件，配置格式为key-value形式，只不过key和value都是java类型的完整类名。摘抄一段如下：

![spring_factories](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/spring_factories.jpg)

现在找一个EnableAutoConfiguration的配置类来看看，比如`org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration`：

![webMvcConfig](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/webMvcConfig.jpg)

这是一个典型的JavaConfig配置类，因此，自动配置真正实现是从classpath中搜寻所有的META-INF/spring.factories配置文件 ，并将其中对应的 org.springframework.boot.autoconfigure包下的配置项，通过反射实例化为对应标注了 @Configuration的JavaConfig形式的IOC容器配置类 ， 然后将这些都汇总成为一个实例并加载到IOC容器中。

### @ComponentScan

扫描当前包及其子包下被`@Component`，`@Controller`，`@Service`，`@Repository`注解标记的类并纳入到spring容器中进行管理。相当于以前的`<context:component-scan>`（以前使用在xml中使用的标签，用来扫描包配置的平行支持）。

我们可以通过basePackages等属性指定@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现从声明@ComponentScan所在类的package进行扫描，默认情况下是不指定的，所以SpringBoot的启动类最好放在root package下。