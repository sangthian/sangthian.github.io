---
title: Spring Boot自动配置原理简单理解
date: 2020-08-16 11:28:28
categories: Spring
tags:
  - Spring
---

本篇文章需要回答的一个问题是：Spring Boot工程中`application.properties`文件中可以写入各种各样的配置，这个事情到底是由什么决定的？

假设我们现在引入了Spring的web对饮的starter，那么当我们试图写一点配置属性的时候，可以看到很多的提示信息：<!--more-->

![properties文件提示](https://cdn.jsdelivr.net/gh/sangthian/CloudPic@master/uPic/properties文件提示.jpg)

这很清楚的表明spring boot工程中一定存在对应的配置类及其属性。那么接下来就开始理解这一机制。

## AutoConfiguration类

spring boot自动装配的文件地址是`/Install_Package/Maven_Repository/org/springframework/boot/spring-boot-autoconfigure/2.2.5.RELEASE/spring-boot-autoconfigure-2.2.5.RELEASE.jar!/META-INF/spring.factories`。

为了方便探究原理，我们打开这个factories文件，找到其中的`HttpEncodingAutoConfiguration`，然后查看其源码如下：

```java
// 表示这是一个配置类
@Configuration(
    proxyBeanMethods = false
)
// 自动配置属性，可以指定配置哪一个类的属性
// 这里指定的HttpProperties类就有若干个属性，我们可以在properties文件编写进行映射，否则就用默认的
@EnableConfigurationProperties({HttpProperties.class})
// @Conditional是spring boot底层注解
// 可根据不同条件来判定当前配置或者类是否生效
@ConditionalOnWebApplication(
    type = Type.SERVLET // 是否为web环境
)
@ConditionalOnClass({CharacterEncodingFilter.class}) // 是否存在这个类
@ConditionalOnProperty(
    prefix = "spring.http.encoding",
    value = {"enabled"},
    matchIfMissing = true // 是否存在指定配置项，如不存在则使用默认值
)
public class HttpEncodingAutoConfiguration {
    private final Encoding properties;
		// 注入这个配置类
    public HttpEncodingAutoConfiguration(HttpProperties properties) {
        this.properties = properties.getEncoding();
    }

    @Bean
    @ConditionalOnMissingBean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
        return filter;
    }

    @Bean
    public HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
        return new HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer(this.properties);
    }

    private static class LocaleCharsetMappingsCustomizer implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {
        private final Encoding properties;

        LocaleCharsetMappingsCustomizer(Encoding properties) {
            this.properties = properties;
        }

        public void customize(ConfigurableServletWebServerFactory factory) {
            if (this.properties.getMapping() != null) {
                factory.setLocaleCharsetMappings(this.properties.getMapping());
            }

        }

        public int getOrder() {
            return 0;
        }
    }
}
```

继续点开`HttpProperties`类进行查看：

```java
// 从配置文件中获取指定的值和bean的属性进行绑定
@ConfigurationProperties(prefix = "spring.http")
public class HttpProperties {
    private boolean logRequestDetails;
    private final HttpProperties.Encoding encoding = new HttpProperties.Encoding();

    public HttpProperties() {
    }

    public boolean isLogRequestDetails() {
        return this.logRequestDetails;
    }

    public void setLogRequestDetails(boolean logRequestDetails) {
        this.logRequestDetails = logRequestDetails;
    }

    public HttpProperties.Encoding getEncoding() {
        return this.encoding;
    }

    public static class Encoding {
        public static final Charset DEFAULT_CHARSET;
        private Charset charset;
        private Boolean force;
        private Boolean forceRequest;
        private Boolean forceResponse;
        private Map<Locale, Charset> mapping;

        public Encoding() {
            this.charset = DEFAULT_CHARSET;
        }

        public Charset getCharset() {
            return this.charset;
        }

        public void setCharset(Charset charset) {
            this.charset = charset;
        }

        public boolean isForce() {
            return Boolean.TRUE.equals(this.force);
        }

        public void setForce(boolean force) {
            this.force = force;
        }
        ......
    }
```

`HttpProperties`类存在一个常见的注解`@ConfigurationProperties(prefix = "spring.http")`，表明该类的属性值是可以去配置文件中取值的，而配置文件中则需要书写前缀为`spring.http`。

总结spring boot自动装配的精髓：

- SpringBoot启动会加载大量的自动配置类
- 看需要的功能有没有在Spring Boot默认写好的自动配置类当中；
- 再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件存在在其中，我们就不需要再手动配置了）
- 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们只需要在配置文件中指定这些属性的值即可；

## @Conditional注解

了解完自动装配的原理后，我们来关注一个细节问题，**自动配置类必须在一定的条件下才能生效**。

@Conditional派生注解：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效。

| @Conditional扩展注解            | 判定规则                                         |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中是否存在指定的bean                         |
| @ConditionalOnMissingBean       | 容器中不存在指定的bean                           |
| @ConditionalOnExpression        | 满足SpEL表达式                                   |
| @ConditionalOnClass             | 系统中有制定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的bean，或者这个bean是首选bean |
| @ConditonalOnproperty           | 系统中指定的属性满足指定的取值                   |
| @ConditonalOnResource           | 类路径下存在指定的资源                           |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定的项                                 |

**那么多的自动配置类，必须在一定的条件下才能生效；也就是说，我们加载了这么多的配置类，但不是所有的都生效了。**

我们怎么知道哪些自动配置类生效？

我们可以通过启用 debug=true属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效；

```properties
# 开启springboot的调试类
debug=true
```

- Positive matches:（自动配置类启用的：正匹配）

- Negative matches:（没有启动，没有匹配成功的自动配置类：负匹配）

- Unconditional classes: （没有条件的类）