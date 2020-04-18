---
title: Spring基础(7)-使用注解开发
date: 2020-04-14 20:52:34
categories: Spring
tags:
  - Spring
---

## Bean的注入

随着Spring技术的发展，目前已经越来越多使用注解开发，而Spring Boot更是可以做到全注解。

在Spring4之后，要使用注解进行开发，必须导入aop的包，且在配置文件中需要context的约束。

我们之前都是使用xml配置文件来定义bean，现在可以使用注解了。首先是配置扫描哪些包下面的注解，配置如下：<!-- more -->

```xml
<!--指定注解扫描包-->
<context:component-scan base-package="com.mx.pojo"/>
```

然后在指定的包下编写类，增加一个`@Component`注解：

```java
@Component
// 相当于配置文件中 <bean id="user" class="当前注解的类"/>
public class User {
    public String name = "jesse";
}
```

`@Component`：关于组件的注解，放在类上，说明这个类被Spring管理了，变成了bean。

## 属性的注入

我们使用`@Value`注解来对属性进行注入。

方式1：不提供set方法，直接在属性定义上添加注解：

```java
@Component
// 相当于配置文件中 <bean id="user" class="当前注解的类"/>
public class User {
    @Value("jesse")
    // 相当于配置文件中 <property name="name" value="jesse"/>
    public String name;
}
```

方式2：如果提供了set方法，那么可以在set方法上添加注解：

```java
@Component
public class User {
    public String name;
  
    @Value("jesse")
    public void setName(String name) {
        this.name = name;
    }
}
```

## 衍生的注解

`@Component`有几个衍生的注解，在微博开发中会按照mvc三层架构进行分级，于是产生了下列的衍生注解：

- `@Controller`：web层
- `@Service`：service层
- `@Repository`：dao层

这几个注解在功能上本质是一样的，都是将类交给Spring来管理，注册成为bean。分开只是为了分层管理。

## 自动装配的注解

也就是前面讲到的`@Autowired`，补充说下`@Autowired`的三种应用场景，分别是字段（变量）注入，构造器注入和set注入，举例如下：

```java
// 变量（filed）注入：
@Autowired
private LoginService loginService;

// 构造器注入
private LoginService loginService;
@Autowired
public LoginController (LoginService loginService) {
	this.loginService = loginService;
}

// set方法注入
private LoginService loginService;
@Autowired
public void setLoginService (LoginService loginService) {
	this.loginService = loginService;
}
```

三者的关系和区别：

- 变量注入：过于依赖注入容器了，当没有启动整个依赖容器时，这个类就不能运转，在反射时无法提供这个类需要的依赖。不推荐使用，在Idea中会提示“Field injection is not recommended”的警告
- 构造器注入：如果注入的属性是必选的属性，则通过构造器注入。某些强制规范的类，都采用此方式。
- set注入：如果注入的属性是可选的属性，则通过setter方法注入。set注入的好处在于可有可无，即使没有注入这个依赖，那么也不会影响整个类的运行。

## 作用域注解

使用`@scope`注解：

- singleton：默认，Spring会采用单例模式创建这个对象。关闭工厂 ，所有的对象都会销毁。
- prototype：多例模式。关闭工厂 ，所有的对象不会销毁。内部的垃圾回收机制会回收。

```java
@Controller
@Scope("prototype")
public class User {
    @Value("jesse")
    public String name;
}
```

## 使用Java类进行配置

现在完全不使用Spring的xml来进行配置，全部交给Java来做。利用Spring的子项目JavaConfig来做。

新建pojo类`com.mx.pojo.User`：

```java
// 说明这个类被Spring接管了，注册到了容器中
@Component
public class User {
    private String name;

    public String getName() {
        return name;
    }

    // 属性注入值
    @Value("jesse")
    public void setName(String name) {
        this.name = name;
    }
}
```

新建配置类`com.mx.config.AppConfig`：

```java
// 这个也会被Spring容器托管，注册到容器中
// 本质上是一个@Component，@Configuration代表这个是配置类，和beans.xml起的作用是一致的
@Configuration
@ComponentScan("com.mx.pojo")
public class AppConfig {

    // 注册一个Bean，相当于xml中的bean标签
    // 方法的名字，相当于bean标签中的id属性
    // 方法的返回值，相当于bean标签的class属性
    @Bean
    public User getUser(){
        return new User();  // 返回注入的bean
    }
}
```

编写测试类：

```java
public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        User user = context.getBean("getUser", User.class);
        System.out.println(user.getName());
    }
}
// print "jesse"
```

再补充下，导入其他配置的做法：再次编写一个配置类：

```java
@Configuration  
public class AppConfig2 {
  ... 
}
```

导入这个配置类

```java
@Configuration
@Import(AppConfig2.class)  //导入合并其他配置类，类似于配置文件中的inculde标签
public class AppConfig {
    @Bean
    public Dog dog(){
        return new Dog();
    }
}
```

此处例子比较简单，更多的内容需要去官方文档查看。

全注解开发是Spring Boot的精髓之一，之后的学习需要更多了解这方面内容。