---
title: SpringCloud学习-Ribbon
date: 2021-1-8 23:29:53
categories:
- Spring
tags:
- Spring
---

## 1、什么是Ribbon？

### 1.1、Ribbon的作用？

​         Ribbon是一个基于**HTTP**和**TCP**客户端的**负载均衡器**，默认使用**轮询**的方式 。通过SpringCloud的封装，可以让我们轻松的将面向服务的**REST模板请求自动转换成客户端负载均衡的服务调用**。 Feign是集成、封装了Ribbonn这个组件而来的，让服务间的调用更方便。

   Spring Cloud Ribbon虽然只是一个工具类框架，**它不像服务注册中心、配置中心、API网关那样需要独立部署，但是它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。因为微服务间的调用，API网关的请求转发等内容，实际上都是通过Ribbon来实现的**，包括后续我们将要介绍的Feign，它也是基于Ribbon实现的工具。

​        Spring Cloud Ribbon 在单独使用时，可以通过在客户端中配置 ribbonServerList 来指定服务实例列表，通过轮训访问的方式起到负载均衡的作用。 

​         在与 Eureka 联合使用时，ribbonServerList 会被重写，改为通过 Eureka 服务注册中心获取服务实例列表，可以通过简单的几行配置完成 Spring Cloud 中服务调用的负载均衡。 

### 1.2、如何开启Ribbon？

1、添加相关依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```

2、启用服务发现客户端，声明负载均衡的restTemplate

```java
@EnableDiscoveryClient
@SpringBootApplication
public class RibbonConsumerApplication {

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(RibbonConsumerApplication.class, args);
    }
}
```

3、通过rest请求发起服务消费请求

```java
@RestController
public class ConsumerController {

    @Autowired
    RestTemplate restTemplate;

    private static final String HELLO_SERVICE = "HTTP://hello-service/";

    @GetMapping("hello")
    public String hello() {
        // return restTemplate.getForObject(HELLO_SERVICE + "hello", String.class);
        return restTemplate.getForEntity(HELLO_SERVICE + "/hello", String.class).getBody();
    }
}
```

## 2、RestTemplat详解

  RestTemplate是一个HTTP客户端，使用它我们可以方便的调用HTTP接口，支持GET、POST、PUT、DELETE等方法。 

### 2.1、GET方法

 **getForObject**方法返回对象为响应体中数据转化成的对象 ：

```java
@GetMapping("/{id}")
public CommonResult getUser(@PathVariable Long id) {
    return restTemplate.getForObject(userServiceUrl + "/user/{1}", CommonResult.class, id);
}
```
**getForEntity**方法返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等 ：

```java
@GetMapping("/getEntityByUsername")
public CommonResult getEntityByUsername(@RequestParam String username) {
    ResponseEntity<CommonResult> entity = restTemplate.getForEntity(userServiceUrl + "/user/getByUsername?username={1}", CommonResult.class, username);
    if (entity.getStatusCode().is2xxSuccessful()) {
        return entity.getBody();
    } else {
        return new CommonResult("操作失败", 500);
    }
}
```

### 2.2、POST方法

postForObject示例

```java
@PostMapping("/create")
public CommonResult create(@RequestBody User user) {
    return restTemplate.postForObject(userServiceUrl + "/user/create", user, CommonResult.class);
}
```

postForEntity示例

```java
@PostMapping("/create")
public CommonResult create(@RequestBody User user) {
    return restTemplate.postForEntity(userServiceUrl + "/user/create", user, CommonResult.class).getBody();
}
```

### 2.3、PUT请求

```java
@PutMapping("/update")
public CommonResult update(@RequestBody User user) {
    restTemplate.put(userServiceUrl + "/user/update", user);
    return new CommonResult("操作成功",200);
}
```

### 2.4、DELETE请求

```java
@DeleteMapping("/delete/{id}")
public CommonResult delete(@PathVariable Long id) {
   restTemplate.delete(userServiceUrl + "/user/delete/{1}", null, id);
   return new CommonResult("操作成功",200);
}
```

## 3、负载均衡策略

### 3.1，轮询策略（RoundRobinRule）

轮询策略理解起来比较简单，就是拿到所有的server集合，然后根据id进行遍历。这里的id是ip+端口，Server实体类中定义的id属性如下：

this.id = host + ":" + port

这里还有一点需要注意，轮询策略有一个上限，当轮询了10个服务端节点还没有找到可用服务的话，轮询结束。

### 3.2、 随机策略（RandomRule） 

 随机策略：使用jdk自带的随机数生成工具，生成一个随机数，然后去可用服务列表中拉取服务节点Server。如果当前节点不可用，则进入下一轮随机策略，直到选到可用服务节点为止。 

### 3.3、 可用过滤策略（AvailabilityFilteringRule） 

 策略描述：过滤掉连接失败的服务节点，并且过滤掉高并发的服务节点，然后从健康的服务节点中，使用轮询策略选出一个节点返回。 

### 3.4、响应时间权重策略（WeightedResponseTimeRule）

 策略描述：根据响应时间，分配一个权重weight，响应时间越长，weight越小，被选中的可能性越低。 

### 3.5、轮询失败重试策略（RetryRule） 

 轮询失败重试策略（RetryRule）是这样工作的，首先使用轮询策略进行负载均衡，如果轮询失败，则再使用轮询策略进行一次重试，相当于重试下一个节点，看下一个节点是否可用，如果再失败，则直接返回失败。

### 3.6、 并发量最小可用策略（BestAvailableRule） 

策略描述：选择一个并发量最小的server返回。如何判断并发量最小呢？ServerStats有个属性activeRequestCount，这个属性记录的就是server的并发量。轮询所有的server，选择其中activeRequestCount最小的那个server，就是并发量最小的服务节点。 

## 4、常用配置

|                           配置名称                           |             作用             |
| :----------------------------------------------------------: | :--------------------------: |
|                     ConnectTimeout: 1000                     | 服务请求连接超时时间（毫秒） |
|                      ReadTimeout: 3000                       | 服务请求处理超时时间（毫秒） |
|                OkToRetryOnAllOperations: true                |    对超时请求启用重试机制    |
|                 MaxAutoRetriesNextServer: 1                  |    切换重试实例的最大个数    |
|                      MaxAutoRetries: 1                       |    切换实例后重试最大次数    |
| NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule |       修改负载均衡算法       |

## 5、参考资料

1、 https://juejin.im/post/5adee863f265da0b7527c26e 

2、 https://www.phpsong.com/3713.html 

3、 https://juejin.im/post/5d7f9006f265da03951a260c 

4、 https://www.jianshu.com/p/d4e4de2ad3aa 

5、https://www.jianshu.com/p/1bd66db5dc46




