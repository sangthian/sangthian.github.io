---
title: Spring基础(5)-依赖注入
date: 2020-04-11 12:27:24
categories: Spring
tags: 
  - Spring
---

## 关于依赖注入

依赖注入（Dependency Injection，DI），是指程序运行过程中，如果需要调用另一个java对象协助时，无须在代码中创建被调用者，而是依赖于外部的注入。

- 依赖：Bean对象的创建依赖于容器 . Bean对象的依赖资源。
- 注入：Bean对象所依赖的资源 , 由容器来设置和装配。<!-- more -->

主要分为三种方式：构造器注入，set注入（重点）和扩展方式注入，其中构造器注入的方式在前文中有提及，重点关注set注入。

## Set方式注入

根据简单示例代码来学习了解这一部分内容

### 简单set注入

首先新建`com.mx.pojo.Student`类，包含各种类型的数据：

```java
package com.mx.pojo;

import java.util.List;
import java.util.Map;
import java.util.Properties;

public class Student {
    private String name;
    private Address address;
    private String[] books;
    private List<String> hobbies;
    private Map<String, String> card;
    private Properties info;
}

```

然后新建配置文件`beans.xml`，顺便说下，只要在pom中配置了spring-mvc的依赖，那么就能在idea中看到新建spring xml配置的选项，可以直接生成模板：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--添加spring-mvc依赖之后可以看到该模板-->
    <bean id="student" class="com.mx.pojo.Student">
        <!--第一种，普通value赋值-->
        <property name="name" value="jesse" />
    </bean>

</beans>a
```

最后新建测试类`test.java.MyTest`:

```java
import com.mx.pojo.Student;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context =  new ClassPathXmlApplicationContext("beans.xml");
        Student student = (Student) context.getBean("student");
        System.out.println(student.getName());
    }
}
// print "jesse"
```

完成以上三步，就能实现最简单的属性set注入

### 其他类型的注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="address" class="com.mx.pojo.Address" />

    <bean id="student" class="com.mx.pojo.Student">
        <!--第一种，普通value赋值-->
        <property name="name" value="jesse" />
        <!--第二种，bean引用注入-->
        <property name="address" ref="address" />
        <!--第三种，数组注入-->
        <property name="books">
            <array>
                <value>西游记</value>
                <value>三国演义</value>
            </array>
        </property>
        <!--第四种，list注入-->
        <property name="hobbies">
            <list>
                <value>篮球</value>
                <value>足球</value>
            </list>
        </property>
        <!--第五种，map注入-->
        <property name="card">
            <map>
                <entry key="id" value="123456" />
                <entry key="vip" value="123456" />
            </map>
        </property>
        <!--第六种，配置类注入-->
        <property name="info">
            <props>
                <prop key="year">2020</prop>
                <prop key="month">04</prop>
            </props>
        </property>
    </bean>
</beans>
```

然后这个bean的各个属性初值就被注入，然后只需要使用就可以了。

## Bean的作用域

Spring IOC容器创建一个Bean实例时，可以为Bean指定实例的作用域，作用域包括singleton（单例模式）、prototype（原型模式）、request（HTTP请求）、session（会话）、global-session（全局会话）。

![bean的作用域](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/bean的作用域.png)

- 单例模式（Spring默认机制）

当一个bean的作用域为Singleton，那么Spring IoC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。Singleton是单例类型，就是在创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象。注意，Singleton作用域是Spring中的缺省作用域。

```xml
<bean id="user1" class="com.mx.pojo.User" scope="singleton" />
```

- 原型模式，每次从容器中get的时候，都会产生一个新的对象

当一个bean的作用域为Prototype，表示一个bean定义对应多个对象实例。Prototype作用域的bean会导致在每次对该bean请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）时都会创建一个新的bean实例。Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。

```xml
<bean id="user2" calss="com.mx.pojo.User" scope="prototype" />
```

- 其余的request，session，application作用域

仅在基于web的应用中使用（不必关心你所采用的是什么web应用框架）



