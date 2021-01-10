---
title: Concurrent并发包学习-BlockingQueue
date: 2021-1-8 23:29:53
categories:
- 多线程
tags:
- 多线程
---

## 1、BlockingQueue用法

  BlockingQueue 通常用于一个线程生产对象，而另外一个线程消费这些对象的场景 。

![]({{ site.url }}/assets/img/多线程/1.1.png)

​    一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。
也就是说，它是**有限的**。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插
入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。
负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中
提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。  

|      | 抛异常     | 特定值   | 阻塞    | 超时                        |
| ---- | ---------- | -------- | ------- | --------------------------- |
| 插入 | add(o)     | offer(o) | put(o)  | offer(o, timeout, timeunit) |
| 移除 | remove(0)  | poll(o)  | take(o) | poll(timeout, timeunit)     |
| 检查 | element(o) | peek(o)  |         |                             |

 四组不同的行为方式解释：

- 抛异常：如果试图的操作无法立即执行，抛一个异常。

- 特定值：如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。

- 阻塞：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。

- 超时：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等
  待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)  。

  **无法向一个 BlockingQueue 中插入 null。如果你试图插入 null， BlockingQueue 将会抛出**
**一个 NullPointerException**。  
  
## 2、BlockingQueue的实现

   BlockingQueue是个接口，你需要使用它的实现之一来使用BlockingQueue。

  - ArrayBlockingQueue
  - DelayQueue 
  - LinkedBlockingQueue
  - PriorityBlockingQueue 
  - SynchronousQueue 
  
     首先，BlockingQueueExample 类分别在两个独立的线程中启动了一个 Producer 和 一个 Consumer。Producer 向一个共享的 BlockingQueue 中注入字符串，而 Consumer 则会从中把它们拿出来。 

  ```java
  package blockingqueue;
  
  import java.util.concurrent.ArrayBlockingQueue;
  import java.util.concurrent.BlockingQueue;
  
  public class BlockingQueueExample {
      public static void main(String[] args) throws Exception {
          BlockingQueue queue = new ArrayBlockingQueue(1024);
          Producer producer = new Producer(queue);
          Consumer consumer = new Consumer(queue);
          new Thread(producer).start();
          new Thread(consumer).start();
          Thread.sleep(4000);
      }
  }
  ```

  以下是 Producer 类。注意它在每次 put() 调用时是如何休眠一秒钟的。这将导致 Consumer  

  在等待队列中对象的时候发生阻塞。 

  ```java
  package blockingqueue;
  
  import java.util.concurrent.BlockingQueue;
  
  public class Producer implements Runnable {
      protected BlockingQueue queue = null;
  
      public Producer(BlockingQueue queue) {
          this.queue = queue;
      }
  
      public void run() {
          try {
              queue.put("1");
              Thread.sleep(1000);
              queue.put("2");
              Thread.sleep(1000);
              queue.put("3");
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  ```

  以下是 Consumer 类。它只是把对象从队列中抽取出来，然后将它们打印到 System.out。

  ```java
  package blockingqueue;
  
  import java.util.concurrent.BlockingQueue;
  
  public class Consumer implements Runnable {
      protected BlockingQueue queue = null;
  
      public Consumer(BlockingQueue queue) {
          this.queue = queue;
      }
  
      public void run() {
          try {
              System.out.println(queue.take());
              System.out.println(queue.take());
              System.out.println(queue.take());
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  ```

  
