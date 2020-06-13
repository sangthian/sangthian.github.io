---
title: Spring Web中的过滤器和拦截器
date: 2020-06-13 20:59:04
categories: Spring
tags: 
	- Spring
---

Java Web有三大器，也就是监听器、过滤器和拦截器，今天我们来对其中的过滤器和拦截器做分析比较

## 过滤器

### 定义

过滤器，顾名思义是起到过滤筛选作用的一种东西，在Spring程序中，过滤器过滤的对象是客户端访问的web资源，也可以理解成一种预处理手段，对资源进行拦截后，将其中我们认为的杂质（用户自定义的）进行过滤，符合条件的放行，不符合的则拦截下来。

<!--more-->

![java过滤器](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/java过滤器.jpg)

过滤器可以拦截request，也可以拦截返回的response，当浏览器发送请求到服务器时，先执行过滤器，然后才访问Web资源。服务器响应Response，从Web资源抵达浏览器之前，也会途径过滤器。

使用过滤器的主要场景是：在过滤器中修改字符编码；在过滤器中修改HttpServletRequest的一些参数，包括：过滤低俗文字、危险字符等。

### 工作原理

- Filter 程序是一个实现了特殊接口的 Java 类，与 Servlet 类似，也是由 Servlet 容器进行调用和执行的。
- 当 Servlet 容器发现已经注册了 Filter 程序，那么容器会**优先调用 Filter 的 doFilter 方法**（可以去激活目标 Servlet 的 service 方法），再由 doFilter 方法决定是否去激活 service 方法。
- 只要**在过滤器的 FilterChain.doFilter 方法的前后**增加某些程序代码，这样就可以在 Servlet 进行响应前后实现某些特殊功能。
- 如果在 Filter.doFilter 方法中没有调用 FilterChain.doFilter 方法，则目标 Servlet 的 service 方法不会被执行，这样通过 Filter 就可以阻止某些非法的访问请求。
- 多个Filter可以组成Filter链，其执行顺序和配置相关

### FIlter接口

过滤器类实现的是`javax.servlet.Filter`接口，看一下`javax.servlet.Filter`接口，定义了三个方法，分别如下：

> 1. public void init(FilterConfig filterConfig)

Filter 的初始化，在 Servlet 容器创建过滤器实例的时候调用，以确保过滤器能够正常工作，该方法在声明周期中只能被执行一次。

> 2. public void doFilter (ServletRequest request, ServletResponse response, FilterChain chain)

Filter 的核心方法，用于对每个拦截到的请求做一些设定好的操作。request和response参数为传递过来的请求和响应对象，chian为当前Filter链的对象。

> 3. public void destroy()

Filter 的销毁，在 Servlet 容器销毁过滤器实例时调用，以释放其占用的资源。只有在 doFilter() 方法中的所有线程退出或超时后，web 容器才会调用此方法。

### 实现自己的过滤器

- 添加pom依赖

`javax.servlet.Filter`无需手动添加pom项目，只要spring boot引入了web组件即可

- 编写controller

```java
package com.mx.demo.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value = "/api")
public class MyController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String first(){
        return "hello";
    }
}
```

- 编写两个过滤器类

```java
package com.mx.demo.filter;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

public class FirstFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void destroy() {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("First,检验接口是否被调用，尝试获取contentType如下： " + response.getContentType());
        chain.doFilter(request, response);//这步使得请求能够继续传导下去，如果没有的话，请求就在此结束
        System.out.println("First,检验接口是否被调用，尝试获取contentType如下： " + response.getContentType());
    }
}
```

```java
package com.mx.demo.filter;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

public class SecondFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void destroy() {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("Second,检验接口是否被调用，尝试获取contentType如下： " + response.getContentType());
        chain.doFilter(request, response);//这步使得请求能够继续传导下去，如果没有的话，请求就在此结束
        System.out.println("Second,检验接口是否被调用，尝试获取contentType如下： " + response.getContentType());
    }
}
```

- 编写配置类

spring早期时候，Filter主要是通过web.xml进行配置的，现在spring boot的配置方法主要有两种，一种是自定义配置类，如下：

```java
package com.mx.demo.config;

import com.mx.demo.filter.FirstFilter;
import com.mx.demo.filter.SecondFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class WebConfig {
    @Bean
    public FilterRegistrationBean<FirstFilter> firstFilter() {
        FilterRegistrationBean<FirstFilter> filterRegBean = new FilterRegistrationBean<>();
        filterRegBean.setFilter(new FirstFilter());
        filterRegBean.addUrlPatterns("/api/*"); // 匹配模式
        filterRegBean.setOrder(1); // 执行顺序，数字小的先执行
        return filterRegBean;
    }

    @Bean
    public FilterRegistrationBean<SecondFilter> secondFilter() {
        FilterRegistrationBean<SecondFilter> filterRegBean = new FilterRegistrationBean<>();
        filterRegBean.setFilter(new SecondFilter());
        filterRegBean.addUrlPatterns("/api/*"); // 匹配模式
        filterRegBean.setOrder(2); // 执行顺序
        return filterRegBean;
    }
}
```

这种方式的好处在于可以集中配置。

另一种则是使用`@WebFilter`和`@Order`的注解，能够在每个Filter之上进行配置，减少代码量

```java
package com.mx.demo.filter;

import org.springframework.core.annotation.Order;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@Order(10) // 设置优先级
@WebFilter(filterName = "firstFilter", urlPatterns = "/api/*")  // 设置匹配模式
public class FirstFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void destroy() {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("First,检验接口是否被调用，尝试获取contentType如下： " + response.getContentType());
        chain.doFilter(request, response);//这步使得请求能够继续传导下去，如果没有的话，请求就在此结束
        System.out.println("First,检验接口是否被调用，尝试获取contentType如下： " + response.getContentType());
    }
}

```

然后还需要在启动类中增加注解`@ServletComponentScan`。

- 执行效果，控制台会打印如下信息：

```shell
First,检验接口是否被调用，尝试获取contentType如下： null
Second,检验接口是否被调用，尝试获取contentType如下： null
Second,检验接口是否被调用，尝试获取contentType如下： text/plain;charset=UTF-8
First,检验接口是否被调用，尝试获取contentType如下： text/plain;charset=UTF-8
```

### 链式调用

当我们配置了多个 filter，且一个请求能够被多次拦截时，该请求将沿着 `客户端 -> 过滤器1 -> 过滤器2 -> servlet -> 过滤器2 -> 过滤器1 -> 客户端` 链式流转，如下图所示 ：

![Filter链式调用](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/Filter链式调用.jpg)

## 拦截器

### 定义

拦截器是 AOP 的一种实现策略，用于在某个方法或字段被访问前对它进行拦截，然后在其之前或之后加上某些操作。同filter一样，interceptor也是链式调用。每个interceptor的调用会依据它的声明顺序依次执行。

拦截器的主要使用场景

- 日志记录：记录请求信息的日志，以便进行信息监控、信息统计等

- 权限检查：如登录检测，进入处理器检测检测是否登录

- 性能监控：通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间


### HandlerInterceptor接口

拦截器实现的是HandlerInterceptor接口，该接口定义了三个方法，分别如下：

> 1. preHandler(HttpServletRequest request, HttpServletResponse response, Object handler)

方法将在**请求处理之前**进行调用。SpringMVC中的`Interceptor`同Filter一样都是**链式调用**。每个Interceptor的调用会依据它的声明顺序依次执行，而且最先执行的都是Interceptor中的preHandle方法.

> 2. postHandler(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)

在当前**请求进行处理之后**，也就是Controller 方法调用之后执行，但是它会在DispatcherServlet 进行视图返回渲染之前被调用，所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作。

> 3. afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handle, Exception ex)

该方法也是需要当前对应的Interceptor的preHandle方法的返回值为true时才会执行。顾名思义，该方法将在整个请求结束之后，也就是在DispatcherServlet **渲染了对应的视图之后执行**。这个方法的主要作用是用于进行资源清理工作的。

### 实现自己的拦截器

- 添加配置类，同上
- 编写controller

```java
package com.mx.demo.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value = "/api")
public class MyController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String hello(){
        return "hello";
    }

    @RequestMapping(value = "/world", method = RequestMethod.GET)
    public String world(){
        return "world";
    }

    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public String login(){
        return "login";
    }
}

```

- 编写2个拦截器，其中第一个的功能是打印请求的执行时间

```java
package com.mx.demo.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class FirstInterceptor implements HandlerInterceptor {

    long start = System.currentTimeMillis();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("这是第1个拦截器");
        start = System.currentTimeMillis();
        return true;
    }

    // postHandler是请求结束执行的，但只有preHandle方法返回true的时候才会执行
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("Interceptor cost="+(System.currentTimeMillis()-start));
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}

```

第二个拦截器的功能是为登录用户跳转login

```java
package com.mx.demo.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

@Component
public class SecondInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("这是第2个拦截器");
        HttpSession session = request.getSession();
        //用户已登录
        if (session.getAttribute("user") != null) {
            return true;
        } else {//用户未登录，直接跳转登录页面
            response.sendRedirect("/api/login");
            return false;
        }
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}

```

- 编写配置

```java
package com.mx.demo.config;

import com.mx.demo.interceptor.FirstInterceptor;
import com.mx.demo.interceptor.SecondInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.AutoConfigureOrder;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Autowired
    private FirstInterceptor firstInterceptor;

    @Autowired
    private SecondInterceptor secondInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 1.加入的顺序就是拦截器执行的顺序，
        // 2.按顺序执行所有拦截器的preHandle
        // 3.所有的preHandle 执行完再执行全部postHandle 最后是postHandle
        registry.addInterceptor(firstInterceptor).addPathPatterns("/api/**");
        registry.addInterceptor(secondInterceptor).addPathPatterns("/api/**").excludePathPatterns("/api/login");
    }
}


```

- 调用接口，得到执行结果

如果请求的是`/api/hello`，那么会被重定向到`/api/login`里头：

```shell
这是第1个拦截器
这是第2个拦截器
这是第1个拦截器
Interceptor cost=23
```

如果请求的是`/api/hello`，则只会打印如下结果：

```shell
这是第1个拦截器
Interceptor cost=2
```

### 链式调用

拦截器的链式调用方式和过滤器十分相似

![](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/拦截器链式调用.png)

### 混合后的调用顺序

如果一个spring工程中既有过滤器，又有拦截器，那么他们的执行顺序如下图，可见拦截器的执行是在过滤器的包围之中的。

![](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/过滤器和拦截器的执行顺序.jpg)



## 二者比较

如果我们去网上查找过滤器和拦截器的区别，多半会得到以下答案：

1. 拦截器是基于java的反射机制的，而过滤器是基于函数回调。
2. 拦截器不依赖与servlet容器，过滤器依赖与servlet容器。
3. 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
4. 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
5. 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。

以上基于技术实现的角度，如果我们想更加直观的理解二者的区别和联系，可以从设计模式出发：

- 过滤器（Filter）：当你有一堆东西的时候，你只希望选择符合你要求的某一些东西。定义这些要求的工具，就是过滤器。

过滤器换一种表达就是预处理（pre processing）或者后处理（post processing），举个例子，煮饭之前用水先对米进行清洗，这就是预处理；榨完果汁后用筛子将果渣过滤掉，这就是后处理。**对数据进行预处理或者后处理就是过滤器要做的工作**，常见的应用有将请求中的数据进行转码，日志系统，系统缓存这些都是要依赖过滤器来实现，当有多个过滤器时就形成了过滤器链，也就是要依次经过过滤器链中的过滤器才能最终到达实际目标.

- 拦截器（Interceptor）：在一个流程正在进行的时候，你希望干预它的进展，甚至终止它进行，这是拦截器做的事情。

拦截器顾名思义，就是对数据进行拦截，从这个定义上看，似乎和过滤器很像，**但是拦截器要做的工作更多是安全方面**，比如用户验证、判断是否登陆。和过滤器的一个区别就是拦截器不一定会到达目标，也就是可以拒绝你的请求，但是过滤器写出来后一般是会到达目标的，只是在到达目标前或者后要进行一些操作。