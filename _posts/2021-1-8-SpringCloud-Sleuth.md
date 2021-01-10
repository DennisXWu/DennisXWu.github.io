---
title: SpringCloud学习-Sleuth
date: 2021-1-8 23:29:53
categories:
- SpringCloud
tags:
- SpringCloud
---

## 1、什么是Sleuth？

### 1.1、Sleuth的功能

​      Spring-Cloud-Sleuth是Spring Cloud的组成部分之一，为SpringCloud应用实现了一种**分布式追踪解决方案**，其兼容了**Zipkin, HTrace和log-based**追踪,追踪微服务rest服务调用链路的问题 。

​    随着分布式系统越来越复杂，你的一个请求发过发过去，各个微服务之间的跳转，有可能某个请求某一天压力太大了，一个请求过去没响应，一个请求下去依赖了三四个服务，但是你去不知道哪一个服务出来问题，这时候就需要对微服务进行追踪 。 监控一个请求的发起，从服务之间传递的过程，记录每一个的耗时多久，一旦出了问题，我们就可以针对性的进行优化 。

### 1.2、Sleuth解决了什么问题？

​    微服务跟踪(sleuth)其实是一个工具,它在整个分布式系统中能**跟踪一个用户请求的过程**(包括数据采集，数据传输，数据存储，数据分析，数据可视化)，捕获这些跟踪数据，就能构建微服务的整个调用链的视图，这是调试和监控微服务的关键工具。 

-  提供链路追踪：通过sleuth可以很清楚的看出一个请求经过了哪些服务，可以方便的理清服务局的调用关系 
-  性能分析: 通过sleuth可以很方便的看出每个采集请求的耗时，分析出哪些服务调用比较耗时，当服务调用的耗时随着请求量的增大而增大时，也可以对服务的扩容提供一定的提醒作用。
-  数据分析优化链路 ： 对于频繁地调用一个服务，或者并行地调用等，可以针对业务做一些优化措施 。
-  可视化： 对于程序未捕获的异常，可以在zipkpin界面上看到 。

### 1.3、Sleuth的原理

**Span**：基本工作单元，发送一个远程调度任务 就会产生一个Span，Span是一个64位ID唯一标识的，Trace是用另一个64位ID唯一标识的，Span还有其他数据信息，比如摘要、时间戳事件、Span的ID、以及进度ID。

**Trace**：一系列Span组成的一个树状结构。请求一个微服务系统的API接口，这个API接口，需要调用多个微服务，调用每个微服务都会产生一个新的Span，所有由这个请求产生的Span组成了这个Trace。

**Annotation**：用来及时记录一个事件的，一些核心注解用来定义一个请求的开始和结束 。这些注解包括以下：

1、cs - Client Sent -客户端发送一个请求，这个注解描述了这个Span的开始。

2、sr - Server Received -服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络传输的时间。

3、ss - Server Sent （服务端发送响应）–该注解表明请求处理的完成(当请求返回客户端)，如果ss的时间戳减去sr时间戳，就可以得到服务器请求的时间。

4、cr - Client Received （客户端接收响应）-此时Span的结束，如果cr的时间戳减去cs时间戳便可以得到整个请求所消耗的时间。

![]({{ site.url }}/assets/img/spring/13.1.png)

## 2、如何实现Sleuth？

1、 在Spring Boot Web应用中增加Sleuth非常简单，只需在pom.xml增加以下的依赖： 

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

2、 首先创建一个服务 

```java
@Service
public class SleuthService {
    public void doSomeWorkSameSpan() throws InterruptedException {
        Thread.sleep(1000L);
        logger.info("Doing some work");
    }
}
```

 然后写一个controller去调用这个服务 

```java
@RestController
public class SleuthController {
    @Autowired
    private final SleuthService sleuthService;
    @GetMapping("/same-span")
    public String helloSleuthSameSpan() throws InterruptedException {
        logger.info("Same Span");
        sleuthService.doSomeWorkSameSpan();
        return "success";
    }
}
```

查看日志：

```xml
2018-04-16 16:35:54.151  INFO [Sleuth Tutorial,0afe3e67168fce4f,0afe3e67168fce4f,false] 5968 --- [nio-8080-exec-1] c.p.s.cloud.sleuth.SleuthController      : Same Span
2018-04-16 16:35:55.161  INFO [Sleuth Tutorial,0afe3e67168fce4f,0afe3e67168fce4f,false] 5968 --- [nio-8080-exec-1] c.p.spring.cloud.sleuth.SleuthService    : Doing some work
```

日志的格式为：[application name, traceId, spanId, export]

- application name — 应用的名称，也就是application.properties中的spring.application.name参数配置的属性。
- traceId — 为一个请求分配的ID号，用来标识一条请求链路。
- spanId — 表示一个基本的工作单元，一个请求可以包含多个步骤，每个步骤都拥有自己的spanId。一个请求包含一个TraceId，多个SpanId
- export — 布尔类型。表示是否要将该信息输出到类似Zipkin这样的聚合器进行收集和展示。

​    可以看到，TraceId和SpanId在两条日志中是相同的，即使消息来源于两个不同的类。这就可以在不同的日志通过寻找traceid来识别一个请求。

## 3、参考资料

1、 https://blog.csdn.net/peterwanghao/article/details/79967634 



