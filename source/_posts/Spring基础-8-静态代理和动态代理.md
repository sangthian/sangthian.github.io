---
title: Spring基础(8)-静态代理和动态代理
date: 2020-04-18 17:25:44
categories: Spring
tags:
  - Spring
---

## 代理模式

为什么我们要接触学习代理模式，因为这是Spring AOP的底层原理。

代理模式的分类：

- 静态代理
- 动态代理<!-- more -->

![代理模式](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/代理模式.png)

## 静态代理

角色分析：

- 抽象角色：一般使用接口或者抽象类解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，代理真实角色后，可以做一些附属操作
- 客户：访问代理

典型的静态代理步骤简单如下：

- 为现有的每一个类都编写一个**对应的**代理类，并且让它实现和目标类相同的接口（假设都有）
- 在创建代理对象时，通过构造器或者setter函数塞入一个目标对象，然后在代理对象的方法内部调用目标对象同名方法，并在调用前后增加附属操作（比如打印日志）
- **代理对象 = 增强代码 + 目标对象（原对象）**，有了代理对象后，就不用原对象了

实例代码如下，首先是抽象的接口类：

```java
public interface UserService {

    void add();
    void delete();
    void query();
    void update();
}
```

然后真实角色实现这个接口：

```java
public class UserServiceImpl implements UserService{
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

接下来编写静态代理类：

```java
public class UserServiceProxy implements UserService{
    private UserServiceImpl userService;
    
    UserServiceProxy(){
        
    }
    
    UserServiceProxy(UserServiceImpl userService){
        this.userService = userService;
    }

    public void add() {
        log();
        userService.add();
    }

    public void delete() {
        log();
        userService.delete();
    }

    public void query() {
        log();
        userService.query();
    }

    public void update() {
        log();
        userService.update();
    }
    
    private void log(){
        System.out.println("before use");
    }
}
```

编写测试类：

```java
public class MyTest {
    public static void main(String[] args) {
        // 真实角色
        UserServiceImpl userService = new UserServiceImpl();
        // 代理角色
        UserServiceProxy proxy = new UserServiceProxy(userService);
        // 调用代理类方法
        proxy.add();
    }
}

// print "before use"
// print "this is add"
```

代理模式的好处：

- 可以使得真实角色更加纯粹，不用去关注一些公共业务
- 公共业务交给代理角色，实现了业务分工
- 公共业务发生扩展，方便修改

静态代理缺点：

- 一个真实角色就会产生一个代理角色，代码量翻倍

## 动态代理

动态代理利用java反射的原理，可以自动生成对应的的代理类。

- 动态代理和静态代理的角色是一致的
- 动态代理的类是动态生成的，不是直接编码的
- 动态代理实现分为：基于接口的动态代理，基于类的动态代理
  - 基于接口-JDK动态代理【主要方法】
  - 基于类-cglib
  - java字节码实现-Javassist

JDK动态代理需要了解两个类：

- `java.lang.reflect.Proxy`，代理类
  - 提供创建动态代理类和实例的静态方法
  - `newProxyInstance`是主要生成代理实例的方法，可以返回指定接口的代理类实例，并将方法调用分派给指定的调用处理程序，也就是InvocationHandler
- `java.lang.reflect.InvocationHandler`，调用处理程序
  - 是由代理实例的调用处理程序实现的接口。每个代理实例都有一个关联的调用处理程序，**当代理实例调用方法的时候，方法调用将被编码并分配到其调用处理程序的invoke方法**
  - 包含唯一的`invoke`方法，需实现它

实例代码如下，抽象的接口类和真实角色和上面一样，就先省略了，下面直接看动态代理类：

```java
// 使用这个类自动生成代理类
public class ProxyInvocationHandler implements InvocationHandler {

    // 被代理的接口类
    private Object target;

    // setter方法传值
    public void setTarget(Object target) {
        this.target = target;
    }

    // 通过Proxy的静态方法，生成得到代理类
    public Object getProxy(){
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }

    // 实现InvocationHandler接口的唯一方法
    // 处理代理实例的方法，并返回结果
    // proxy参数看起来并未使用，其作用不是显式的，看资料说，只有proxy实例在InvocationHandler实现类里加载才能产生第二个参数method
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
        log(method.getName());
        return method.invoke(target, args);
    }

    // 增加的操作
    public void log(String msg){
        System.out.println("before " + msg);
    }
}
```

编写测试类：

```java
public class MyTest {
    public static void main(String[] args) {
        // 真实角色
        UserServiceImpl userService = new UserServiceImpl();
        // 代理角色
        ProxyInvocationHandler pih = new ProxyInvocationHandler();
        // 设置需要代理的对象
        pih.setTarget(userService);
        //动态生成代理类
        UserService proxy = (UserService) pih.getProxy();
        proxy.add();
    }
}
// print "before add"
// print "this is add"
```

动态代理的好处：

- 一个动态代理类代理的是一个接口，一般就是对应的一类业务
- 一个动态代理类可以代理多个类，只要是实现了同一个接口即可