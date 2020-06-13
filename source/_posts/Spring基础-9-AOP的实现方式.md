---
title: Spring基础(9)-AOP的实现方式
date: 2020-04-18 18:46:03
categories: Spring
tags:
  - Spring
---

## 关于AOP

AOP（Aspect Oriented Programming），即面向切面编程，可以说是OOP（Object Oriented Programming，面向对象编程）的补充和完善。通过预编译方式和运行时动态代理实现程序功能的统一维护的一种技术，利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合降低。<!--more-->

![Program_Execution](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/Program_Execution.jpg)

AOP需要掌握的几个概念：

- 横切关注点：跨越应用程序的多个模块或功能，比如日志、安全、缓存等
- 切面（Aspect）：横切关注点被模块化的特殊对象，是一个类
- 通知（Advice）：切面完成的工作，是类中的方法
- 目标（Target）：被通知的对象，一般是业务中的接口或者方法
- 代理（Proxy）：向目标对象应用通知之后创建的对象
- 切入点（PointCut）：切面通知执行的位置，在什么类的什么方法执行切入
- 连接点（JointPoint）：与切入点匹配的执行点，是各个业务方法的执行阶段

Spring AOP中，通过Advice定义横切逻辑，支持5种类型的Advice。

| 通知类型     | 连接点               | 实现接口                                        |
| ------------ | -------------------- | ----------------------------------------------- |
| 前置通知     | 方法前               | org.springframework.aop.MethodBeforeAdvice      |
| 后置通知     | 方法后               | org.springframework.aop.AfterReturningAdvice    |
| 环绕通知     | 方法前后             | org.aoppalliance.intercept.MethodInterceptor    |
| 异常抛出通知 | 方法抛出异常         | org.springframework.aop.ThrowsAdvice            |
| 引介通知     | 类中增加新的方法属性 | org.springframework.aop.IntroductionInterceptor |

## 使用Springs实现AOP

使用AOP功能，需要导入如下依赖包

```xml
<dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.1.10.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.5</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>4.3.13.RELEASE</version>
      <scope>compile</scope>
    </dependency>
</dependencies>
```

通过简单案例了解其基础用法。

### 使用Spring的API接口

首先新建业务接口`UserService`及其实现类：

```java
package com.mx.service;

public interface UserService {

    void add();
    void delete();
    void query();
    void update();
}
```

```java
package com.mx.service;

import com.mx.service.UserService;

public class UserServiceImpl implements UserService {
    public void add() {
        System.out.println("this is add");
    }

    public void delete() {
        System.out.println("this is delete");
    }

    public void query() {
        System.out.println("this is query");
    }

    public void update() {
        System.out.println("this is update");
    }
}
```

然后利用AOP的java API新建两个日志类，分别在方法前和方法后执行：

```java
package com.mx.log;

import org.springframework.aop.MethodBeforeAdvice;

import java.lang.reflect.Method;

public class BeforeLog implements MethodBeforeAdvice {
    /**
     *
     * @param method 要执行的目标对象方法
     * @param objects 参数
     * @param o 目标对象
     * @throws Throwable
     */
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println(o.getClass().getName() + "的" + method.getName() + "被执行了");
    }
}
```

```java
package com.mx.log;

import org.springframework.aop.AfterReturningAdvice;

import java.lang.reflect.Method;

public class AfterLog implements AfterReturningAdvice {
    /**
     *
     * @param o 执行之后的返回值
     * @param method
     * @param objects
     * @param o1
     * @throws Throwable
     */
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        System.out.println("执行了"  + method.getName() + "方法，返回结果为：" + o);
    }
}
```

然后使用传统的xml文件进行bean，切点的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd ">

    <!--注册bean-->
    <bean id="userService" class="com.mx.service.UserServiceImpl" />
    <bean id="beforeLog" class="com.mx.log.BeforeLog" />
    <bean id="afterLog" class="com.mx.log.AfterLog" />

    <!--配置AOP-->
    <aop:config>
        <!--切入点，expression填入一个execution表达式，表示要执行的位置-->
        <aop:pointcut id="pointcut" expression="execution(* com.mx.service.UserServiceImpl.*(..))"/>
        <!--执行环绕增加，哪个方法类作用在哪个切点-->
        <aop:advisor advice-ref="beforeLog" pointcut-ref="pointcut" />
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut" />
    </aop:config>
</beans>
```

最后新建测试类查看效果：

```java
import com.mx.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 动态代理的是接口
        UserService userService = context.getBean("userService", UserService.class);
        userService.add();
    }
}
// print "com.mx.service.UserServiceImpl的add被执行了"
// print "this is add"
// print "执行了add方法，返回结果为：null"
```

可以看到，AOP在完全没有改动原来业务逻辑的情况下，通过动态代理实现了日志这种附加逻辑。

### 使用自定义类

首先自定义自己的切面类：

```java
package com.mx.diy;

public class Diy {

    public void before(){
        System.out.println("-----方法执行前------");
    }

    public void after(){
        System.out.println("-----方法执行后------");
    }
}
```

接下来在`beans.xml`进行自定义切面的配置：

```xml
<!--方法二：自定义类-->
<bean id="diy" class="com.mx.diy.Diy" />
	<aop:config>
  	<!--自定义切面，ref为引用的类-->
  	<aop:aspect ref="diy">
    	<!--切入点-->
    	<aop:pointcut id="point" expression="execution(* com.mx.service.UserServiceImpl.*(..))"/>
    <!--通知，也有的翻译为增强-->
    <aop:before method="before" pointcut-ref="point" />
    <aop:after method="after" pointcut-ref="point" />
  </aop:aspect>
</aop:config>
```

执行测试类之后的结果如下：

```java
-----方法执行前------
this is add
-----方法执行后------
```

这种自定义切面的方式比较简单，但是无法获取类中方法的内部信息。

### 使用注解

首先在`beans.xml`开启bean包扫描和aop的注解支持：

```xml
<!--指定注解扫描包-->
<context:component-scan base-package="com.mx.diy"/>
<!--aop的注解支持-->
<aop:aspectj-autoproxy />
```

然后编写自定义切面类：

```java
package com.mx.diy;

import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect // 标注这个类是一个切面
@Component
public class AnnotationPointCut {

    // 配置切入点和连接点
    @Before("execution(* com.mx.service.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("=====方法执行前=====");
    }

    @After("execution(* com.mx.service.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("=====方法执行后======");
    }
}
```

执行测试类之后的结果如下：

```java
=====方法执行前=====
this is add
=====方法执行后======
```

各个连接点的执行顺序从先到后如下：

- 环绕前
- 方法执行前
- 环绕后
- 方法执行后

