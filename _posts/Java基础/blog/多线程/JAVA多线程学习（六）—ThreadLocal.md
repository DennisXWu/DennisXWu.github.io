## 1、什么是ThreadLocal

  ThreadLocal是JDK包提供的，它提供了线程本地变量，也就是如果你创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个**本地副本**。当多个线程操作这个变量时，实际操作的是自己本地内存里面的变量，从而避免了线程安全问题。

![](../../img/多线程/8.1.jpeg)


## 2、如何使用ThreadLocal？

```java
public class ThreadLocalTest {

    static ThreadLocal<String> localVariable = new ThreadLocal<>();

    public static void main(String[] args) {
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    localVariable.set("threadOne local variable");
                    Thread.sleep(1000);
                    System.out.println("threadOne: " + localVariable.get());
                    localVariable.remove();
                    System.out.println("threadOne remove after " + ":" + localVariable.get());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        Thread threadTwo = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    localVariable.set("threadTwo local variable");
                    Thread.sleep(2000);
                    System.out.println("threadTwo: " + localVariable.get());
                    localVariable.remove();
                    System.out.println("threadTwo remove after " + ":" + localVariable.get());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        threadOne.start();
        threadTwo.start();
    }
}
```

结果如下：

```java
threadOne: threadOne local variable
threadOne remove after :null
threadTwo: threadTwo local variable
threadTwo remove after :null
```

## 3、ThreadLocal的实现原理

![](../../img/多线程/8.2.jpeg)


Thread类中有一个ThreadLocalMap类型的变量，ThreadLocal类通过set方法将变量存放在ThreadLocalMap变量中，当调用get方法时，再从当前线程的ThreadLocalMap中将其拿出来使用。

## 4、ThreadLocal的继承性

**ThreadLocal本身不具有继承性**，也就是说同一个ThreadLocal变量在父线程中被设置值后，在子线程是获取不到的。为了解决这个问题，**InheritableThreadLocal**应运而生，其提供一个特性，就是让子线程可以访问父线程中设置的本地变量。



## 5、参考文献

1、《Java并发编程之美》
