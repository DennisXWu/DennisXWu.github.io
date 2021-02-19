---
title: SpringCloud学习-Hystrix服务容错保护
date: 2021-1-8 23:29:53
categories:
- Spring
tags:
- Spring
---

## 1、Hystrix容错保护原理

###    1.1、什么是雪崩效应？

​       随着业务的扩展，服务的数量也会随之增多，逻辑会更加复杂，一个服务的某个逻辑需要依赖多个其他服务才能完成。一但一个依赖不能提供服务很可能会产生**雪崩效应**，最后导致整个服务不可访问 。服务超时阻塞，就有可能使得整个主线程池被占满 。

![]({{ site.url }}/assets/img/spring/7.1.png)


  线程池被占满就会导致整个服务不可用，而依赖该服务的其他服务，就又可能会重复产生上述问题。因此整个系统就像雪崩一样逐渐的扩散、坍塌、崩溃了！ 

### 1.2、Hystrix实现原理

hystrix语义为“豪猪”，具有自我保护的能力。hystrix的出现即为解决雪崩效应，它通过四个方面的机制来解决这个问题

- 隔离（线程池隔离和信号量隔离）：限制调用分布式服务的资源使用，某一个调用的服务出现问题不会影响其他服务调用。
- 优雅的降级机制：超时降级、资源不足时(线程或信号量)降级，降级后可以配合降级接口返回托底数据。
- 融断：当失败率达到阀值自动触发降级(如因网络故障/超时造成的失败率高)，熔断器触发的快速失败会进行快速恢复。
- 缓存：提供了请求缓存、请求合并实现。
- 支持实时监控、报警、控制（修改配置）

## 2、Hystrix应用

修改 spring-demo-service-ribbon 的 pom 文件，添加 Hystrix 的依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

 修改启动类，添加 @EnableHystrix 注解开启 Hystrix 功能 

```java
@SpringBootApplication
@EnableEurekaClient
@EnableHystrix
public class ServiceRibbonApplication {
 
	public static void main(String[] args) {
        SpringApplication.run(ServiceRibbonApplication.class, args);
	}
 
	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}
 
	@Bean
	public IRule ribbonRule() {
		return new RandomRule();	//这里配置策略，和配置文件对应
	}
}

```

 修改 SpringDemoRibbonService 

```java
@Service
public class SpringDemoRibbonService {
 
    @Autowired
    RestTemplate restTemplate;
 
    @Autowired
    LoadBalancerClient loadBalancerClient;
 
    @HystrixCommand(fallbackMethod = "portFallback")
    public String port() {
        this.loadBalancerClient.choose("spring-demo-service");  //随机访问策略
        return restTemplate.getForObject("http://SPRING-DEMO-SERVICE/port", String.class);
    }
 
    public String portFallback() {
        return "sorry ribbon, it's error!";
    }
}

```

 在 port 方法上加上 @HystrixCommand 注解，对此方法开启熔断器功能，用 fallbackMethod 属性指定熔断回调方法。 

## 3、参数详解

**Execution相关的属性的配置：**

- hystrix.command.default.execution.isolation.strategy   // 隔离策略，默认值为Thread, 可选Thread｜Semaphore
- hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds   // 命令执行超时时间，默认值为1000ms
- hystrix.command.default.execution.timeout.enabled   // 执行是否启用超时，默认启用true
- hystrix.command.default.execution.isolation.thread.interruptOnTimeout   // 发生超时是是否中断，默认true
- hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests   // 最大并发请求数，默认10，该参数当使用ExecutionIsolationStrategy.SEMAPHORE策略时才有效。如果达到最大并发请 数，请求会被拒绝。理论上选择semaphore size的原则和选择thread size一致，但选用semaphore时每次执行的单元要比较小且执行速度快（ms级别），否则的话应该用thread。semaphore应该占整个容器（tomcat）的线程池的一小部分。

**Fallback相关的属性：**

这些参数可以应用于Hystrix的THREAD和SEMAPHORE策略

- hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests   // 如果并发数达到该设置值，请求会被拒绝和抛出异常并且fallback不会被调用。默认10
- hystrix.command.default.fallback.enabled   // 当执行失败或者请求被拒绝，是否会尝试调用hystrixCommand.getFallback() 。默认true

**Circuit Breaker相关的属性：**

- hystrix.command.default.circuitBreaker.enabled   // 用来跟踪circuit的健康性，如果未达标则让request短路。默认true
- hystrix.command.default.circuitBreaker.requestVolumeThreshold   //  一个rolling window内最小的请求数。如果设为20，那么当一个rolling window的时间内（比如说1个rolling window是10秒）收到19个请求，即使19个请求都失败，也不会触发circuit break。默认20
- hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds   // 触发短路的时间值，当该值设为5000时，则当触发circuit break后的5000毫秒内都会拒绝request，也就是5000毫秒后才会关闭circuit。默认5000
- hystrix.command.default.circuitBreaker.errorThresholdPercentage   // 错误比率阀值，如果错误率>=该值，circuit会被打开，并短路所有请求触发fallback。默认50
- hystrix.command.default.circuitBreaker.forceOpen   // 强制打开熔断器，如果打开这个开关，那么拒绝所有request，默认false
- hystrix.command.default.circuitBreaker.forceClosed   // 强制关闭熔断器 如果这个开关打开，circuit将一直关闭且忽略circuitBreaker.errorThresholdPercentage

**Metrics相关参数：**

- hystrix.command.default.metrics.rollingStats.timeInMilliseconds   // 设置统计的时间窗口值的毫秒值，circuit break 的打开会根据1个rolling window的统计来计算。若rolling window被设为10000毫秒，则rolling window会被分成n个buckets，每个bucket包含success，failure，timeout，rejection的次数的统计信息。默认10000
- hystrix.command.default.metrics.rollingStats.numBuckets   // 设置一个rolling window被划分的数量，若numBuckets＝10，rolling window＝10000，那么一个bucket的时间即1秒。必须符合rolling window % numberBuckets == 0。默认10
- hystrix.command.default.metrics.rollingPercentile.enabled   // 执行时是否enable指标的计算和跟踪，默认true
- hystrix.command.default.metrics.rollingPercentile.timeInMilliseconds   // 设置rolling percentile window的时间，默认60000
- hystrix.command.default.metrics.rollingPercentile.numBuckets   // 设置rolling percentile window的numberBuckets。逻辑同上。默认6
- hystrix.command.default.metrics.rollingPercentile.bucketSize   // 如果bucket size＝100，window＝10s，若这10s里有500次执行，只有最后100次执行会被统计到bucket里去。增加该值会增加内存开销以及排序的开销。默认100
- hystrix.command.default.metrics.healthSnapshot.intervalInMilliseconds   // 记录health 快照（用来统计成功和错误绿）的间隔，默认500ms

**Request Context 相关参数：**

- hystrix.command.default.requestCache.enabled 默认true，需要重载getCacheKey()，返回null时不缓存
- hystrix.command.default.requestLog.enabled 记录日志到HystrixRequestLog，默认true

**Collapser Properties 相关参数 :**

- hystrix.command.default.requestCache.enabled 默认true，需要重载getCacheKey()，返回null时不缓存
- hystrix.command.default.requestLog.enabled 记录日志到HystrixRequestLog，默认true
- hystrix.collapser.default.maxRequestsInBatch 单次批处理的最大请求数，达到该数量触发批处理，默认Integer.MAX_VALUE
- hystrix.collapser.default.timerDelayInMilliseconds 触发批处理的延迟，也可以为创建批处理的时间＋该值，默认10
- hystrix.collapser.default.requestCache.enabled 是否对HystrixCollapser.execute() and HystrixCollapser.queue()的cache，默认true

**ThreadPool 相关参数：**

线程数默认值10适用于大部分情况（有时可以设置得更小），如果需要设置得更大，那有个基本的公式可供参考：

requests per second at peak when healthy × 99th percentile latency in seconds + some breathing room

每秒最大支撑的请求数* (99%平均响应时间 + 缓存值)

比如：每秒能处理1000个请求，99%的请求响应时间是60ms，那么公式是：

threadsNums = 1000 *（0.060+0.012）

基本的原则是保持线程池尽可能小，他主要是为了释放压力，防止资源被阻塞。当一切都是正常的时候，线程池一般仅会有1到2个线程激活来提供服务

- hystrix.threadpool.default.coreSize   // 默认值为10，并发执行的最大线程数
- hystrix.threadpool.default.maxQueueSize   // 默认值为-1，线程等待的队列最大值，BlockingQueue的最大队列数，当设为－1，会使用SynchronousQueue，值为正时使用LinkedBlcokingQueue。该设置只会在初始化时有效，之后不能修改threadpool的queue size，除非reinitialising thread executor。默认－1。
- hystrix.threadpool.default.queueSizeRejectionThreshold   // 默认值为5，即使maxQueueSize没有达到，达到queueSizeRejectionThreshold该值后，请求也会被拒绝。因为maxQueueSize不能被动态修改，这个参数将允许我们动态设置该值。if maxQueueSize == -1，该字段将不起作用
- hystrix.threadpool.default.allowMaximumSizeToDivergeFromCoreSize   // 默认值为false，是否允许队列满之后新建线程到达到最大线程数，如果为false，当队列满后就执行降级方法
- hystrix.threadpool.default.keepAliveTimeMinutes   // 如果corePoolSize和maxPoolSize设成一样（默认实现）该设置无效。如果通过plugin使用自定义实现，该设置才有用，默认1.
- hystrix.threadpool.default.metrics.rollingStats.timeInMilliseconds   // 线程池统计指标的时间，默认10000
- hystrix.threadpool.default.metrics.rollingStats.numBuckets   // 将rolling window划分为n个buckets，默认10

## 4、参考资料

1、 https://blog.csdn.net/loushuiyifan/article/details/82702522 

2、 https://www.jianshu.com/p/e07661b9bae8 

3、 https://blog.csdn.net/tongtong_use/article/details/78611225 

4、 https://www.cnblogs.com/rickiyang/p/11973218.html 
