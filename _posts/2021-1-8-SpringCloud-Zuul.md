---
title: SpringCloud学习-Zuul
date: 2021-1-8 23:29:53
categories:
- Spring
tags:
- Spring
---

## 1、什么是Zuul？

​       Zuul是spring cloud中的**微服务网关**。网关：是一个网络整体系统中的前置门户入口。请求首先通过网关，进行路径的路由，定位到具体的服务节点上。

　　**Zuul是一个微服务网关，首先是一个微服务**。也是会在Eureka注册中心中进行服务的注册和发现。也是一个网关，请求应该通过Zuul来进行路由。

　　Zuul网关不是必要的。是推荐使用的。

　　使用Zuul，一般在微服务数量较多（多于10个）的时候推荐使用，对服务的管理有严格要求的时候推荐使用，当微服务权限要求严格的时候推荐使用。

### 1.1、Zuul工作原理

​      zuul的核心是一系列的filters, 其作用类比Servlet框架的Filter，或者AOP。zuul把请求路由到用户处理逻辑的过程中，这些filter参与一些过滤处理，比如Authentication，Load Shedding等

![]({{ site.url }}/assets/img/spring/9.1.png)


**Zuul使用一系列不同类型的过滤器，使我们能够快速灵活地将功能应用于我们的边缘服务。这些过滤器可帮助我们执行以下功能**

- 身份验证和安全性 - 确定每个资源的身份验证要求并拒绝不满足这些要求的请求
- 洞察和监控 - 在边缘跟踪有意义的数据和统计数据，以便为我们提供准确的生产视图
- 动态路由 - 根据需要动态地将请求路由到不同的后端群集
- 压力测试 - 逐渐增加群集的流量以衡量性能。
- Load Shedding - 为每种类型的请求分配容量并删除超过限制的请求。
- 静态响应处理 - 直接在边缘构建一些响应，而不是将它们转发到内部集群。

### 1.2、过滤器生命周期

Zuul中提供了过滤器定义，可以用来过滤代理请求，提供额外功能逻辑。如：权限验证，日志记录等。 

**Zuul提供的过滤器是一个父类。父类是ZuulFilter。通过父类中定义的抽象方法filterType，来决定当前的Filter种类是什么。**有前置过滤、路由后过滤、后置过滤、异常过滤。

- **前置过滤**：是请求进入Zuul之后，立刻执行的过滤逻辑。
- **路由后过滤**：是请求进入Zuul之后，并Zuul实现了请求路由后执行的过滤逻辑，路由后过滤，是在远程服务调用之前过滤的逻辑。
- **后置过滤**：远程服务调用结束后执行的过滤逻辑。
- **异常过滤**：是任意一个过滤器发生异常或远程服务调用无结果反馈的时候执行的过滤逻辑。无结果反馈，就是远程服务调用超时。

![]({{ site.url }}/assets/img/spring/9.2.png)


### 1.3、zuul组件

- **zuul-core--zuul**核心库，包含编译和执行过滤器的核心功能。

-  **zuul-simple-webapp--zuul** Web应用程序示例，展示了如何使用zuul-core构建应用程序。

-  **zuul-netflix--lib**包，将其他NetflixOSS组件添加到Zuul中，例如使用功能区进去路由请求处理。

-  **zuul-netflix-webapp--webapp**，它将zuul-core和zuul-netflix封装成一个简易的webapp工程包。

## 2、如何使用Zuul？

​      zuul网关其**底层使用ribbon来实现请求的路由，并内置Hystrix**，可选择性提供网关fallback逻辑。使用zuul的时候，并不推荐使用Feign作为application client端的开发实现。毕竟Feign技术是对ribbon的再封装，使用Feign本身会提高通讯消耗，降低通讯效率，只在服务相互调用的时候使用Feign来简化代码开发就够了。而且商业开发中，使用Ribbon+RestTemplate来开发的比例更高。 

**1、导入zuul和eureka依赖**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

**2、启动类标注注解**

```java
/**
 * @author Gjing
 */
@SpringBootApplication
@EnableZuulProxy
@EnableEurekaClient
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }
}
```

**3、使用Eureka负载路由方式**

```hxml
server:
  port: 8080
spring:
  application:
    name: zuul
# 配置Eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
# 构建路由地址
zuul:
  routes:
    # 这里可以自定义
    demo2:
      # 匹配的路由规则
      path: /demo/**
      # 路由的目标服务名
      serviceId: demo
```

## 3、参考资料

1、 https://www.jianshu.com/p/863c2a6cd0a9 

2、 https://www.jianshu.com/p/511db36c1b3e 

3、https://www.cnblogs.com/huangjuncong/p/9060984.html
