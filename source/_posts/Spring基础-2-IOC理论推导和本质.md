---
title: Spring基础(2)-IOC理论推导和本质
date: 2020-04-05 11:05:06
categories: Spring
tags:
  - Spring
---

## 案例分析

使用maven新建一个项目

添加一个UserDao接口：

```java
public interface UserDao {
    public void getUser();
}
```

实现这个UserDao接口：<!-- more -->

```java
public class UserDaoImpl implements UserDao {
    @Override
    public void getUser() {
        System.out.println("获取用户数据");
    }
}
```

添加一个UserService接口：

```java
public interface UserService {
    public void getUser();
}
```

实现这个UserService接口：

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao = new UserDaoImpl(); 
    @Override
    public void getUser() {
        userDao.getUser(); // 调用Dao层方法
    }
}
```

编写测试类，模拟controller：

```java
public static void main(String[] args) {
  	UserService service = new UserServiceImpl();
    service.getUser();
}
// print "获取用户数据"
```

现在如果UserDao的实现类增加一个，比如叫`UserDaoMySqlImpl`：

```java
public class UserDaoMySqlImpl implements UserDao {
    @Override
    public void getUser() {
        System.out.println("获取MySql用户数据");
    }
}
```

紧接着我们要去使用MySql的话 , 我们就需要去UserService实现类里面修改对应的实现 :

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao = new UserDaoMySqlImpl();  // 修改实例对象
    @Override
    public void getUser() {
        userDao.getUser(); // 调用Dao层方法
    }
}
```

如果还有更多类似的需求，那么代码改动量将会很大，因此 这种设计的耦合性太高了，牵一发而动全身 ，不适合快速变化的业务系统。

## 方法改进

UserService的实现类不是直接去New一个新的实例，而是留出set接口，让调用者决定选择哪个具体的Dao实现，当然这其中有多态的思想在里面：

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;
    // 利用set方法实现
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    @Override
    public void getUser() {
        userDao.getUser();
    }
}
```

现在再使用测试类：

```java
public static void main(String[] args) {
  	UserService service = new UserServiceImpl();
    service.setUserDao(new UserDaoMySqlImpl());
    service.getUser();
}
// print "获取MySql用户数据"
```

虽然只是一个简单的例子，但其中包含的设计思想确实比较高级的：

- 以前所有东西都是由程序去进行控制创建
- 而现在使用了set注入，主动权交给了调用者
- 程序不用去管怎么创建，而是专注于业务逻辑的实现，耦合性大大降低 . 这也就是IOC的原型 。

## IOC本质

![IOC理论背景](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/IOC理论背景.png)

**控制反转（IOC）是一种设计思想，依赖注入（DI）是实现IOC的一种方式**。在没有IOC的程序中，我们使用面向对象编程，对象的创建和对象之间的依赖完全硬编码在程序中，对象的创建由程序自己控制，控制反转后对象的创建转移给第三方，也就是“**获得依赖对象的方式反转了**”。

## 依赖注入

IOC的另外的名字叫做依赖注入（Dependency Injection），所谓的依赖注入，就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。所以，依赖注入(DI)和控制反转(IOC)是从不同的角度的描述的同一件事情，**就是指通过引入IOC容器，利用依赖关系注入的方式，实现对象之间的解耦**。

![依赖注入类比](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/依赖注入类比.png)

类似上图一样：从外部usb读取文件，主机没必要知道通过啥设备，只要符合连接接口就可以了，外部设备主动权由我们自己控制，这样就解耦到最低。

