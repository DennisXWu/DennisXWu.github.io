---
title: Spring学习-IOC
date: 2021-1-8 23:29:53
categories:
- Spring
tags:
- Spring
---

## 1、Ioc容器简介

Spring IoC容器是一个管理Bean的容器，在Spring的定义中，它要求所有的IoC容器都需要实现接口BeanFactory，它是一个顶级容器接口。

![]({{ site.url }}/assets/img/spring/17.1.png)


  首先我看到了多个getBean方法，这也是IoC 容器最重要方法之一 它的意义是从IoC容器中获取Bean而从多个 getBean方法中可看到有按类型（ type ）获取Bean的，也有按名称（ name ）获取Bean的，这就意味着在 pring IoC容器中 ，允许我们按类型或者名称获取Bean。

   **isSingleton**方法则判断Bean是否在Spring IoC中为单例。这里需要记住的是在Spring IoC容器中，默认的情况下， **Bean都是以单例存在的**，也就是使用 getBean方法返回的都是**同一个对象**。与isSingleton方法相反的是 **isPrototype**方法，如果它返回的是true，那么当我们使用getBean方法获取Bean的时候，**Spring IoC容器就会建一个新的Bean回给调用者**。

​    总结， **Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。**

## 2、依赖注入

​    @Autowired是我们使用最多的注解之一，@Autowired提供了这样的规则，**首先它会根据类型找到对应的Bean，如果对应类型的Bean不是唯一的，那么它会根据其属性名称和Bean的名称进行匹配**。如果匹配的上就使用，如果还是无法匹配就抛出异常。

​    如果不能确定其标注属性一定会存在并且允许这个被标注的属性为null，那么你可以**配置@Autowired**

**属性required为false**。

### 2.1、消除歧义性—@Primary和@Quelifier

 @Primary注解是一个修改优先权的注解，这里的@Primary的含义告诉Spring IoC容器，**当发现有多个同样类型的Bean时，请优先使用我进行注入**。

 @Quelifier注解的配置项value需要一个字符串去定义，**它将与@Autowired组合一起，通过类型和名称一起去找到Bean**。

## 3、条件装配Bean

​    有时候某些客观的因素会使 Bean无法进行初始化，例如，在数据库连接池的配置中漏掉一些配置会造成数据源不能连接上在这样的情况下，IoC 容器如果还进行数据源的装配， 则系统将会抛出异常，导致应用无法继续。这时倒是希望IoC 容器不去装配数据源。为了处理这样的场景，Spring提供了**＠Conditional**注解帮助我们，而它需要 配合另外一个**接口Condition**来完成对应的功能。

## 4、Bean的作用域

在 Web容器中，则存在页面（page）、请求（request）、会话（session）和应用（application）4种作用域。

![]({{ site.url }}/assets/img/spring/17.2.png)


## 5、使用@Profile

  在企业开发的过程中，项目往往要面临开发环境、测试环境、准生产环境（用于模拟真实生产环境部署所用〉和生产环境的切换。例如，它会有各自的数据库资源，这样就要求我在不同的数据库之间进行切换。

  在Spring中存在两个参数可以提供给我们配置，以修改启动Profile机制，一个是**spring.profiles.active**，另一个是**spring.profiles.default**。被@Profile标注的Bean将不会被Spring装配到IoC容器中，Spring会先判定是否存在**spring.profiles.active**配置后，再去找**spring.profiles.default**。

## 6、参考文献

1、《深入浅出SpringBoot》





