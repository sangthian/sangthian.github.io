---
title: Spring基础(3)-第一个Spring Demo
date: 2020-04-05 19:39:06
categories: Spring
tags:
  - Spring
---

## Spring Demo

利用maven新建一个空的项目，然后逐步添加Spring相关代码，以最简单的Demo理解Spring的IOC原理。

添加maven依赖：<!-- more -->

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.1.10.RELEASE</version>
  </dependency>
</dependencies>
```

创建`com.mx.pojo.Hello`类：

```java
public class Hello {
    private String name;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void show(){
        System.out.println("Hello," + name );
    }
}
```

编写`resources.beans.xml`配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--使用Spring来创建对象，在Spring中这些对象都称之为Bean-->
    <bean id="hello" class="com.mx.pojo.Hello">
        <!--类中的属性成员，可以赋初值，通过setter方法-->
        <property name="name" value="jesse"/>
    </bean>
</beans>
```

编写测试类`test.MyTest`：

```java
public class MyTest {
    public static void main(String[] args) {
        // 读取xml配置文件，获取Spring的上下文对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 根据bean的id选取，获得对象实例
        Hello hello = (Hello) context.getBean("hello");
        hello.show();
    }
}
// print "Hello,jesse"
```

`ClassPathXmlApplicationContext`和`ApplicationContext`存在N次继承关系，是利用多态的向上转型，从类名中可以探知，获取上下文对象的方式除了xml配置之外，还有其他的，比如注解、文件等。

## 问题思考

- hello对象是谁创建的？
  - hello对象是由Spring创建的，程序中没有new

- hello对象的属性是怎么设置的？   
  - hello对象的属性是由Spring容器设置的

这个过程就叫做控制反转：

- 控制：谁来控制对象的创建，在使用Spring之后，对象是由Spring来创建
- 反转：程序本身不创建对象，变成被动的接受对象
- 依赖注入：利用set方法进行注入

因此对于初级的Spring程序，要实现不同的操作，只需要在xml配置文件中进行修改，IoC的也就成了：对象由Spring来创建、管理和装配！

## 案例进阶

利用ref，引用Spring中创建好的bean对象：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="mySqlImpl" class="com.mx.dao.UserDaoMySqlImpl" />
    
    <bean id="UserServiceImpl" class="com.mx.service.UserServiceImpl">
        <property name="userDao" ref="mySqlImpl"/>
    </bean>
</beans>
```

