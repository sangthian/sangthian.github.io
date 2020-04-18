---
title: Spring基础(6)-Bean的自动装配
date: 2020-04-11 23:29:35
categories: Spring
tags:
  - Spring
---

## XML自动装配

之前在`beans.xml`文件中手动为对象赋初值的操作，就是手动装配，显然这是比较麻烦的，书写量比较大，那么我们就需要一种自动的机制来做这件事。

自动装配是Spring满足bean依赖的一种方式。Spring会在上下文中自动寻找，并自动给bean装配属性。<!-- more -->

Spring中有三种装配的方式：

- 在xml中显式的配置
- 在java中显式的配置
- 隐式的自动装配

通过简单例子XML自动装配，来了解这一原理

新建传统三件套，`pojo`，`beans.xml`和测试类，首先是pojo类：

```java
public class Cat {
    public void bark(){
        System.out.println("miao");
    }
}

public class Dog {
    public void bark(){
        System.out.println("wang");
    }
}

public class People {
    private String name;
    private Cat cat;
    private Dog dog;
}
```

如上所示，我们想对People类自动注入，它有两个引用对象dog和cat，正常写法是使用ref，但我们也可以使用autowired来自动装配：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cat" class="com.mx.pojo.Cat" />
    <bean id="dog" class="com.mx.pojo.Dog" />

    <!--byName，会自动在容器上下文中查找，和自己对象set方法后面值对应的bean的id-->
    <bean id="people" class="com.mx.pojo.People" autowire="byName">
        <property name="name" value="jesse" />
    </bean>
    
</beans>
```

autowire=“byName”这个配置项允许Spring容器根据name来自动对bean进行装配（赋值），这里列举autowired常用的方式：

- byName

  会自动在容器上下文中查找，和自己对象set方法对应的name相同的bean，使用id来查找，需要保证所有bean的id唯一，并且这个bean需要和自动注入的属性的set方法的值一致

- byType

  会自动在容器上下文中查找，和自己对象属性类型相同的bean，需要保证所有bean的class唯一，并且这个bean需要和自动注入的属性的类型一致

## 使用注解实现自动装配

使用注解的自动装配比XML要方便，是官方推荐的方式，注解支持的最低jdk版本是1.5.

要使用注解，需要如下支持：

- 导入约束：context约束
- 配置注解的支持：`<context:annotation-config/>`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context" 
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
  
    <bean id="cat" class="com.mx.pojo.Cat" />
    <bean id="dog" class="com.mx.pojo.Dog" />
    <bean id="people" class="com.mx.pojo.People" />

    <context:annotation-config/>
</beans>
```

然后直接在属性上使用`@Autowired`，就实现了属性的自动注入。也可以在set方法上使用，一般而言，使用`@Autowired`之后，我们就可以不写set方法了，前提是这个这个自动装配的属性在IoC（Spring）容器中存在，且按照先类型再名字的方式匹配。

`@Autowired`存在一个属性设置`required`，来看一下它的用法：

```java
// 如果定义了Autowired的required属性为false，说明这个对象可以为null，否则不允许为空
@Autowired(required = true)
private Cat cat；
```

如果属性的name无法匹配，且类型无法唯一对应，比如出现多个不同id的Dog类型的bean，还想通过`@Autowired`自动装配的话，就要借助于另一个注解：`@Qualifier`:

```java
@Autowired
@Qualifier(value = "dog2") // 指定一个id值进行装配，但是也要符合相同类型才行
private Dog dog;
```

总结：如果自动装配的环境比较复杂，无法使用一个注解完成装配，可以使用`@Qualifier`去配合`@Autowired`的使用，制定一个唯一的bean对象进行注入。

java还有一个原生注解`@Resource`注解，类似于上述二者的结合体，首先通过名字匹配，然后会通过类型匹配，这个功能强大，但是用的相对较少。

```java
@Resource(name = "cat2")
private Cat cat;
@Resource
private Dog dog;
```

总结下`@Autowired`和`@Resource`的关系和区别：

- 都可以用来自动装配，可以放在属性字段上

- `@Autowired`，属于Spring规范，默认通过byType方式实现，如果要允许null 值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用

- `@Resource`，属于J2EE规范，默认通过byName方式实现，如果找不到name，则通过byType实现，如果都找不到会提示报错。同时，如果name属性一旦指定，就只会按照名称进行装配。

  

