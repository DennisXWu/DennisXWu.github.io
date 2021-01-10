---
title: SpringCloud学习-Bus
date: 2021-1-8 23:29:53
categories:
- SpringCloud
tags:
- SpringCloud
---

## 1、什么是Bus?

### 1.1、Bus的功能

​       Spring cloud bus通过轻量消息代理连接各个分布的节点。这会用在广播状态的变化（例如配置变化）或者其他的消息指令。Spring bus的一个**核心思想是通过分布式的启动器对spring boot应用进行扩展，也可以用来建立一个多个应用之间的通信频道**。目前唯一实现的方式是用**AMQP**消息代理作为通道，同样特性的设置（有些取决于通道的设置）在更多通道的文档中。

​    大家可以将它理解为管理和传播所有分布式项目中的消息既可，其实本质是利用了MQ的广播机制在分布式的系统中传播消息，目前常用的有**Kafka和RabbitMQ**。

### 1.2、Bus的原理

​    利用bus的机制可以做很多的事情，其中配置中心客户端刷新就是典型的应用场景之一，我们用一张图来描述bus在配置中心使用的机制。

![]({{ site.url }}/assets/img/spring/11.1.jpg)


根据此图我们可以看出利用Spring Cloud Bus做配置更新的步骤:

1. 提交代码触发post给客户端A发送bus/refresh
2. 客户端A接收到请求从Server端更新配置并且发送给Spring Cloud Bus
3. Spring Cloud bus接到消息并通知给其它客户端
4. 其它客户端接收到通知，请求Server端获取最新配置
5. 全部客户端均获取到最新的配置

## 2、Bus的分类

### 2.1、RabbitMQ

1、首先我们来创建一个普通的Spring Boot工程，然后添加如下依赖： 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

2、接下来在application.properties中配置RabbitMQ的连接信息，如下： 

```xml
spring.application.name=rabbitmq-hello
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=sang
spring.rabbitmq.password=123456
server.port=2009
```

 这里我们分别配置了RabbitMQ的地址为localhost，端口为5672（注意这里没写错，web管理端端口是15672），用户名和密码则是我们刚刚创建出来的（也可以使用默认的guest）。 

3、创建消息生产者

 发送消息我们有一个现成的封装好的对象AmqpTemplate，通过AmqpTemplate我们可以直接向某一个消息队列发送消息，消息生产者的定义方式如下： 

```java
@Component
public class Sender {
    @Autowired
    private AmqpTemplate rabbitTemplate;
    public void send() {
        String msg = "hello rabbitmq:"+new Date();
        System.out.println("Sender:"+msg);
        this.rabbitTemplate.convertAndSend("hello", msg);
    }
}
```

 注入AmqpTemplate，然后利用AmqpTemplate向一个名为hello的消息队列中发送消息。 

4、创建消息消费者

```java
@Component
@RabbitListener(queues = "hello")
public class Receiver {
    @RabbitHandler
    public void process(String msg) {
        System.out.println("Receiver:"+msg);
    }
}
```

@RabbitListener(queues = “hello”)注解表示该消息消费者监听hello这个消息队列。

@RabbitHandler注解则表示process方法是用来处理接收到的消息的，我们这里收到消息后直接打印即可。 

5、配置消息队列Bean

```java
@Configuration
public class RabbitConfig {
    @Bean
    public Queue helloQueue() {
        return new Queue("hello");
    }
}
```

 创建一个名为hello的消息队列。 

6、测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = RabbitmqHelloApplication.class)
public class RabbitmqHelloApplicationTests {

    @Autowired
    private Sender sender;

    @Test
    public void contextLoads() {
        sender.send();
    }
}
```

### 2.2、Kafka

1、添加依赖

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>

  <dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>1.1.1.RELEASE</version>
  </dependency>

  <dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.2</version>
  </dependency>
</dependencies>
```

2、消息实体类

```java
@Data
public class Message {
    private Long id;//id
    private String msg; //消息
    private Date sendTime; //发送时间
}
```

3、消息生产者

 在该类中创建一个消息发送的方法，使用KafkaTemplate.send()发送消息，`wqh`是Kafka里的Topic。 

```java
@Component
@Slf4j
public class KafkaSender {

    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;

    private Gson gson = new GsonBuilder().create();

    public void send(Long i){
        Message message = new Message();
        message.setId(i);
        message.setMsg(UUID.randomUUID().toString());
        message.setSendTime(new Date());
        log.info("========发送消息  "+i+" >>>>{}<<<<<==========",gson.toJson(message));
        kafkaTemplate.send("wqh",gson.toJson(message));
    }
}
```

4、消息接收类

 在这个类中，创建consumer方法，并使用@KafkaListener注解监听指定的topic，如这里是监听wanqh和wqh两个topic。 

```java
@Component
@Slf4j
public class KafkaConsumer {

    @KafkaListener(topics = {"wanqh","wqh"})
    public void consumer(ConsumerRecord<?,?> consumerRecord){
        //判断是否为null
        Optional<?> kafkaMessage = Optional.ofNullable(consumerRecord.value());
        log.info(">>>>>>>>>> record =" + kafkaMessage);
        if(kafkaMessage.isPresent()){
            //得到Optional实例中的值
            Object message = kafkaMessage.get();
            log.info(">>>>>>>>接收消息message =" + message);
        }
    }
}
```

5、配置文件

```xml
spring.application.name=kafka-hello
server.port=8080
#============== kafka ===================
# 指定kafka 代理地址，可以多个
spring.kafka.bootstrap-servers=192.168.18.136:9092

#=============== provider  =======================
spring.kafka.producer.retries=0
# 每次批量发送消息的数量
spring.kafka.producer.batch-size=16384
spring.kafka.producer.buffer-memory=33554432

# 指定消息key和消息体的编解码方式
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

#=============== consumer  =======================
# 指定默认消费者group id
spring.kafka.consumer.group-id=test-consumer-group

spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.enable-auto-commit=true
spring.kafka.consumer.auto-commit-interval=100

# 指定消息key和消息体的编解码方式
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

## 3、参考资料

1、 https://fangjian0423.github.io/2019/04/09/spring-cloud-bus-intro/ 

2、 https://www.cnblogs.com/babycomeon/p/11141160.html 

3、 https://mp.weixin.qq.com/s/vZdN35MB6PdrOf__CRa92g 

4、 https://segmentfault.com/a/1190000013031488 

