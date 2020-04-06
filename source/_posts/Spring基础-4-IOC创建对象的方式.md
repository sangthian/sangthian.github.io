---
title: Spring基础(4)-IOC创建对象的方式
date: 2020-04-05 20:27:36
categories: Spring
tags:
  - Spring
---

## 通过无参构造函数

Spring容器中创建对象，虽然没有显式的new，但说到底还是需要利用构造函数。如果xml配置中没有指定，那么默认通过无参构造函数进行创建，证明的示例如下：<!-- more -->

```java
public class Hello {
    private String name;
    Hello(){
        System.out.println("这是无参构造方法");
    }

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

调用之前的测试类main函数：

```java
public class MyTest {
    public static void main(String[] args) {
        // 读取xml配置文件，获取Spring的上下文对象
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 在执行getBean的时候，hello对象已经创建好了，通过无参构造
        Hello hello = (Hello) context.getBean("hello");
        hello.show();
    }
}
// print "这是无参构造方法"
// print "Hello,jesse"
```

## 通过有参构造函数

修改Hello类：

```java
public class Hello {
    private String name;
    Hello(String name){
        this.name = name;
        System.out.println("这是有参构造方法");
    }

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

beans.xml有三种编写方式：

```xml
<!-- 第一种根据index参数下标设置 -->
<bean id="hello" class="com.mx.pojo.Hello">
    <!-- index指构造方法,下标从0开始 -->
    <constructor-arg index="0" value="jesse"/>
</bean>

<!-- 第二种根据参数名字设置，推荐 -->
<bean id="hello" class="com.mx.pojo.Hello">
    <!-- name指参数名 -->
    <constructor-arg name="name" value="jesse"/>
</bean>

<!-- 第三种根据参数类型设置 -->
<bean id="hello" class="com.mx.pojo.Hello">
    <constructor-arg type="java.lang.String" value="jesse"/>
</bean>
```

## Spring配置简单说明

- alias

```xml
<!--别名，如果添加了别名，也可以使用它来获取这个对象-->
<alias name="hello" alias="hello2" />
```

- bean

```xml
<!--id：bean的唯一标识符-->
<!--class：bean对象的全限定名：包名+类型-->
<!--name：也是别名，可以取多个，用空格、逗号或者分好分隔均可-->
<bean id="hello" name="hello2 h2,h3;h4" class="com.mx.pojo.Hello">
  <property name="name" value="Spring"/>
</bean>
```

- import

一般用于团队开发使用，可将多个配置文件导入：

```xml
<import resource="{path}/beans.xml"/>
```

