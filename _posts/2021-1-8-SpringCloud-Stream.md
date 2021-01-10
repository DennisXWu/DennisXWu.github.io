---
title: SpringCloud学习-Stream
date: 2021-1-8 23:29:53
categories:
- SpringCloud
tags:
- SpringCloud
---

## 1、什么是Stream？

### 1.1、Stream功能

​       Spring Cloud Stream 是一个用来为**微服务应用构建消息驱动能力的框架**。它可以基于 Spring Boot 来创建独立的、可用于生产的 Spring 应用程序。Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动化配置实现，并引入了**发布-订阅、消费组、分区**这三个核心概念。通过使用 Spring Cloud Stream，可以有效**简化开发人员对消息中间件的使用复杂度**，让系统开发人员可以有更多的精力关注于核心业务逻辑的处理。但是目前 Spring Cloud 

### 1.2、为什么要使用Stream？

​       在实际的企业开发中，消息中间件是至关重要的组件之一。消息中间件主要解决应用解耦，异步消
息，流量削锋等问题，实现高性能，高可用，可伸缩和最终一致性架构。不同的中间件其实现方式，内
部结构是不一样的。如常见的RabbitMQ和Kafka，由于这两个消息中间件的架构上的不同，像
RabbitMQ有exchange，kafka有Topic，partitions分区，这些中间件的差异性导致我们实际项目开发
给我们造成了一定的困扰，**我们如果用了两个消息队列的其中一种，后面的业务需求，我想往另外一种**
**消息队列进行迁移，这时候无疑就是一个灾难性的**，一大堆东西都要重新推倒重新做，因为它跟我们的
系统耦合了，这时候 springcloud Stream 给我们提供了一种解耦合的方式。 

### 1.3、Stream原理

​      Spring Cloud Stream由一个中间件中立的核组成。应用通过Spring Cloud Stream插入的input(相当于
消费者consumer，它是从队列中接收消息的)和output(相当于生产者producer，它是从队列中发送消
息的。)通道与外界交流。通道通过指定中间件的Binder实现与外部代理连接。业务开发者不再关注具
体消息中间件，只需关注Binder对应用程序提供的抽象概念来使用消息中间件实现业务即可。 

![]({{ site.url }}/assets/img/spring/12.1.png)


   说明：最底层是消息服务，中间层是绑定层，绑定层和底层的消息服务进行绑定，顶层是消息生产者和
消息消费者，顶层可以向绑定层生产消息和和获取消息消费 

-  **绑定器** ： Binder 绑定器是Spring Cloud Stream中一个非常重要的概念。在没有绑定器这个概念的情况下，我们的Spring Boot应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它
  们的实现细节上会有较大的差异性，这使得我们实现的消息交互逻辑就会非常笨重，因为对具体的中间
  件实现细节有太重的依赖，当中间件有较大的变动升级、或是更换中间件的时候，我们就需要付出非常
  大的代价来实施。通过定义绑定器作为中间层，实现了应用程序与消息中间件(Middleware)细节之间的隔离。通过向应用程序暴露统一的Channel通过，使得应用程序不需要再考虑各种不同的消息中间件的实现。当需要升级消息中间件，或者是更换其他消息中间件产品时，我们需要做的就是更换对应的Binder绑定器而不需要
  修改任何应用逻辑 。甚至可以任意的改变中间件的类型而不需要修改一行代码。 
-  **发布/订阅模型** ： 在Spring Cloud Stream中的消息通信方式遵循了发布-订阅模式，当一条消息被投递到消息中间件之后，它会通过共享的 Topic 主题进行广播，消息消费者在订阅的主题中收到它并触发自身的业务逻辑处理。这里所提到的 Topic 主题是Spring Cloud Stream中的一个抽象概念，用来代表发布共享消息给消费者的地方。在不同的消息中间件中， Topic 可能对应着不同的概念，比如：在RabbitMQ中的它对应了Exchange、而在Kakfa中则对应了Kafka中的Topic。 

![]({{ site.url }}/assets/img/spring/12.2.png)


## 2、如何使用Stream？

### 2.1、消息生产者

 1、创建工程引入依赖 

```xml
   <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        </dependency>
    </dependencies>
```

 2、定义bingding
发送消息时需要定义一个接口，不同的是接口方法的返回对象是 MessageChannel： 

```java
/**
 * 自定义的消息通道
 */
public interface MyProcessor {

    /**
     * 消息生产者的配置
     */
    String MYOUTPUT = "myoutput";

    @Output("myoutput")
    MessageChannel myoutput();

    /**
     * 消息消费者的配置
     */
    String MYINPUT = "myinput";

    @Input("myinput")
    SubscribableChannel myinput();
}
```

```java
/**
 * 负责向中间件发送数据
 */
@Component
@EnableBinding(MyProcessor.class)
public class MessageSender {

    @Autowired
    @Qualifier(value="myoutput")
    private MessageChannel myoutput;

    //发送消息
    public void send(Object obj) {
        myoutput.send(MessageBuilder.withPayload(obj).build());
    }
}
```

3、 yml清单 

```xml
server:
  port: 7001 #服务端口
spring:
  application:
    name: stream_producer #指定服务名
  rabbitmq:
    addresses: 192.168.180.137
    username: guest
    password: guest
  cloud:
    stream:
      bindings:
        output:
          destination: topcheer-default  #指定消息发送的目的地,在rabbitmq中,发送到一个topcheer-default的exchange中
        myoutput:
          destination: topcheer-custom-output
#          producer:
#            partition-key-expression: payload  #分区关键字   对象中的id,对象
#            partition-count: 2  #分区大小
      binders:  #配置绑定器
        defaultRabbit:
          type: rabbit
```

### 2.2、消息消费者

 1、定义bingding
同发送消息一致，在Spring Cloud Stream中接受消息，需要定义一个接口。 

```java
/**
 * 自定义的消息通道
 */
public interface MyProcessor {

    /**
     * 消息生产者的配置
     */
    String MYOUTPUT = "myoutput";

    @Output("myoutput")
    MessageChannel myoutput();

    /**
     * 消息消费者的配置
     */
    String MYINPUT = "myinput";

    @Input("myinput")
    SubscribableChannel myinput();
}
```

```java
@Component
@EnableBinding(MyProcessor.class)
public class MessageListener {

    //监听binding中的消息
    @StreamListener(MyProcessor.MYINPUT)
    public void input(String message) {
        System.out.println("获取到消息: "+message);
    }

}
```

 yml清单 

```xml
server:
  port: 7002 #服务端口
spring:
  application:
    name: rabbitmq-consumer #指定服务名
  rabbitmq:
    addresses: 192.168.180.137
    username: guest
    password: guest
  cloud:
    stream:
#      instanceCount: 2  #消费者总数
#      instanceIndex: 0  #当前消费者的索引
      bindings:
        input: #内置的获取消息的通道 , 从topcheer-default中获取消息
          destination: topcheer-default
        myinput:
          destination: topcheer-custom-output
#          group: group1
#          consumer:
#            partitioned: true  #开启分区支持
      binders:
        defaultRabbit:
          type: rabbit
```

## 3、参考资料

1、 https://fangjian0423.github.io/2019/04/03/spring-cloud-stream-intro/ 

2、 https://www.cnblogs.com/dalianpai/p/12300738.html 



