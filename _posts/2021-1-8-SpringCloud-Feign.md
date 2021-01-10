---
title: SpringCloud学习-Feign
date: 2021-1-8 23:29:53
categories:
- SpringCloud
tags:
- SpringCloud
---

## 1、什么是Feign？

​     Feign 是一个**声明式的 Web Service 客户端**。 使用 Feign 能让编写 Web Service 客户端更加简单，我们只需要像调用方法一样调用它就可以完成服务请求及相关处理 ，同时支持与Hystrix、Ribbon 组合使用以支持负载均衡。

### 1.1、Feign的优点？

1. 发Rest请求时不需要自己拼接url，拼接参数等硬编码操作。
2.  Feign能让编写WebService客户端更加简单。
3.  Feign也支持可插拔式的编码器和解码器。
4.  SpringCloud对Feign进行了封装，使其支持SpringMVC标准注解和HttpMessageConverters 。
5.  Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

### 1.2、Feign原理

Feign的基本用法：

```java
// 1. Feign 动态代理
GitHub github = Feign.builder()
    .decoder(new GsonDecoder())
    .target(GitHub.class, "https://api.github.com");
// 2. Feign 执行
List<Contributor> contributors = github.contributors("OpenFeign", "feign");
```

 **总结：** `Feign` 使用时分成两步：一是生成 Feign 的动态代理；二是 Feign 执行。 

![]({{ site.url }}/assets/img/spring/8.1.png)


**总结：**

1. 前两步是生成动态对象：将 Method 方法的注解解析成 MethodMetadata，并最终生成 Feign 动态代理对象。
2. 后几步是调用过程：根据解析的 MethodMetadata 对象，将 Method 方法的参数转换成 Request，最后调用 Client 发送请求。

## 2、Feign配置

1、引入pom.xml

```xml
<dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2、 SpringbootApplication启动类加上@EnableFeignClients注解 。

3、编写Feign客户端

```java
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {

    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam("name") String name);
}
```

4、编写服务端接口

```java
@RestController
public class HiController {

    @Autowired
    SchedualServiceHi schedualServiceHi;

    @GetMapping(value = "/hi")
    public String sayHi(@RequestParam String name){
        return schedualServiceHi.sayHiFromClientOne(name);

    }
}
```

## 3、Ribbin配置

​     由于 Spring Cloud Feign 的客户端负载均衡是通过 Spring Cloud Ribbon 实现的，所以我们可以直接通过配置 Ribbon 的客户端的方式来自定义各个服务客户端调用的参数。 

### 3.1、全局配置

​    全局配置的方法非常简单，我们可以i直接使用 ribbon.<key>=<value>的方式来设置 ribbon 的各项默认参数，比如，修改默认的客户端调用超时时间示例如下，使用 yml 格式配置：

ribbon:

  ConnectionTimeout: **500**

  ReadTimeout: **500**

### 3.2、指定服务配置

  大多数情况下，我们对于服务调用的超时时间等可能会根据实际服务的特性做一些调整，所以仅仅依靠默认配置是不行的，在使用 Spring Cloud Feign 的时候，针对各个服务客户端进行个性化的配置方式与使用 Spring Cloud Ribbon 时的配置方式是一样的，都采用 <client>.ribbon.<key>=<value> 的格式设置，<client> 为使用 @FeignClient 注解的 name 属性或者 value属性指定的服务名，需要区分大小写，示例如下：

ORG.LIXUE.HELLOWORLD:

  ribbon:

​      ConnectionTimeout: **50**

​      ReadTimeout: **50**

### 3.3、常用配置

| 配置名称                   | 描述                         |
| -------------------------- | ---------------------------- |
| ConnectionTimeout          | 连接超时时间                 |
| ReadTimeout                | 读取超时时间                 |
| OkToRetryOnAllOperatotions | **对所有操作请求都进行重试** |
| MaxAutoRetriesNextServer   | 切换服务器实例的重试次数     |
| MaxAutoRetries             | 对当前实例的重试次数         |
| ServerListRefreshInterval  | 刷新服务列表源的间隔时间     |

## 4、Hystrix配置

1、 在application.properties 中增加熔断开关 

 默认的feign功能中，熔断开关是关闭的，所以，熔断器hystrix的开关需要手动打开 。

```java
spring.application.name=eureka-client
eureka.client.service-url.defaultZone=http://localhost:5430/eureka/
server.port=5433
feign.hystrix.enabled=true
```

2、 实现熔断回调接口 

 熔断回调接口需要继承远程调用接口HelloService，整个类用@Component组件修饰，将该类交给spring容器统一管理。 

```java
package com.example.consumer.hystrix;
 
import org.springframework.stereotype.Component;
 
import com.example.consumer.service.HelloService;
 
@Component
public class HiHystrix implements HelloService{
 
	@Override
	public String hello() {
		return "hi, sorry, service is not used.";
	}
}
```

 3、远程接口配置熔断回调接口 

```java
package com.example.consumer.service;
 
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
 
import com.example.consumer.hystrix.HiHystrix;
 
/**
 * 远程调用接口
 * 调用远程服务eureka-server，并增加熔断方案HiHystrix
 * @author PC
 */
@FeignClient(value="eureka-server", fallback = HiHystrix.class)
public interface HelloService {
 
	@GetMapping("/hi")
	String hello();
}
```

## 5、参考资料

1、 https://www.jianshu.com/p/8c7b92b4396c 

2、 https://www.cnblogs.com/binarylei/p/11563023.html 







