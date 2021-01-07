## 1、阻塞双端队列BlockingDeque

   BlockingDeque 类是一个双端队列，**BlockingDeque接口继承自BlockingQueue接口**，在不能够插入元素时，它将阻塞住试图插入元素的线程；在不能够抽取元素时，它将阻塞住试图抽取的线程。**双端队列是一个你可以从任意一端插入或者抽取元素的队列**。

​     在线程既是一个队列的生产者又是这个队列的消费者的时候可以使用到 BlockingDeque。如果生产者线程需要在队列的两端都可以插入数据，消费者线程需要在队列的两端都可以移除数据，这个时候也可以使用BlockingDeque。

![](../../img/多线程/3.1.png)


BlockingDeque 具有 4 组不同的方法用于插入、移除以及对双端队列中的元素进行检查。 

|          | 抛异常         | 特定值        | 阻塞         | 超时                           |
| -------- | -------------- | ------------- | ------------ | ------------------------------ |
| **插入** | addFirst(o)    | offerFirst(o) | putFirst(o)  | offerFirst(o,timeout,timeunit) |
| **移除** | removeFirst(o) | pollFirst(o)  | takeFirst(o) | pollFirst(timeout,timeunit)    |
| **检查** | getFirst(o)    | peekFirst(o)  |              |                                |

|          | 抛异常        | 特定值       | 阻塞        | 超时                          |
| -------- | ------------- | ------------ | ----------- | ----------------------------- |
| **插入** | addLast(o)    | offerLast(o) | putLast(o)  | offerLast(o,timeout,timeunit) |
| **移除** | removeLast(o) | pollLast(o)  | takeLast(o) | pollLast(timeout,timeunit)    |
| **检查** | getLast(o)    | peekLast(o)  |             |                               |

## 2、链阻塞双端队列LinkedBlockingDeque

LinkedBlockingDeque 是一个双端队列，在它为空的时候，一个试图从中抽取数据的线程将会阻塞，无论该线程是试图从哪一端抽取数据。

```java
BlockingDeque<String> deque = new LinkedBlockingDeque<String>();
deque.addFirst("1");
deque.addLast("2");
String two = deque.takeLast();
String one = deque.takeFirst();
```

