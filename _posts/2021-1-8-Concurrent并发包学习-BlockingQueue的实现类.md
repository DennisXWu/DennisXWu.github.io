---
title: Concurrent并发包学习-BlockingQueue的实现类
date: 2021-1-8 23:29:53
categories:
- 多线程
tags:
- 多线程
---

## 1、ArrayBlockingQueue

​     ArrayBlockingQueue 类实现了 BlockingQueue 接口。 ArrayBlockingQueue 是一个**有界**的阻塞队列，其内部实现是将对象放到一个**数组**里。以下是在使用 ArrayBlockingQueue 的时候对其初始化的一个示例： 

```java
BlockingQueue queue = new ArrayBlockingQueue(1024);

queue.put("1");

Object object = queue.take();
```

## 2、延迟队列DelayQueue

   DelayQueue 实现了 BlockingQueue 接口。 DelayQueue 对元素进行持有直到一个特定的延迟到期。

注入其中的元素必须实现 java.util.concurrent.Delayed 接口，该接口定义： 

```java
public interface Delayed extends Comparable<Delayed< {
    public long getDelay(TimeUnit timeUnit);
}
```

   DelayQueue 将会在每个元素的 getDelay() 方法返回的值的时间段之后才**释放掉该元素**。如果返回的是0或者负值，延迟将被认为**过期**，该元素将会在DelayQueue的**下一次take被调用的时候被释放掉**。

## 3、链阻塞队列LinkedBlockingQueue

​    LinkedBlockingQueue 内部以一个链式结构(链接节点)对其元素进行存储。如果需要的话，这一链式结构可以选择一个上限。如果没有定义上限，将使用Integer.MAX_VALUE作为上限。 

```java
BlockingQueue<String> unbounded = new LinkedBlockingQueue<String>();
BlockingQueue<String> bounded = new LinkedBlockingQueue<String>(1024);
bounded.put("Value");
String value = bounded.take();
```

## 4、具有优先级的阻塞队列PriorityBlockingQueue

PriorityBlockingQueue类实现了BlockingQueue接口。PriorityBlockingQueue是一个**无界的并发队列**。它使用和类 java.util.PriorityQueue一样的排序规则。你无法向这个队列中插入null值。**所有插入到PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口**。因此该队 列中元素的排序就取决于你自己的 Comparable 实现。 注意 PriorityBlockingQueue 对于具有相等优先级(compare() == 0)的元素并不强制任何特定行为。 同时注意，如果你从一个 PriorityBlockingQueue 获得一个 Iterator 的话，该 Iterator 并不能保证它对元素的遍历是以优先级为序的。

```java
BlockingQueue queue = new PriorityBlockingQueue();
 //String implements java.lang.Comparable
 queue.put("Value");
 String value = queue.take();
```

## 5、同步队列SynchronousQueue

   SynchronousQueue 类实现了 BlockingQueue 接口。SynchronousQueue 是一个特殊的队列，**它的内部同时只能够容纳单个元素**。如果该队列已 **有一元素**的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该**队列为空**，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。

