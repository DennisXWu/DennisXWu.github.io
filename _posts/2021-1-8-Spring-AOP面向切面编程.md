---
title: spring学习-AOP面向切面编程
date: 2021-1-8 23:29:53
categories:
- spring
tags:
- spring
---

## 1、什么是AOP？AOP的优点？

​       在软件开发中，散步于应用中的多处功能被称为**横切关注点**，通常来讲这些横切关注点从概念上是与应用的业务逻辑相分离的。把这些**横切关注点与业务逻辑相分离正是面向切面编程（AOP）所要解决的问题。**

​      如图所示展现一个被划分为模块的典型应用，每个模块的核心功能都是为特定业务领域提供服务，但是这些模块都需要类似的辅助功能，比如安全、事务、日志等。这些辅助功能就可以被模块化为横切关注点，横切关注点可以被模块化为特殊的类，这些类被称为**切面**。切面有两个好处：1、每个关注点都集中在一个地方，而不是分散到多处代码中。2、服务模块更加简洁，因为它们只包含主要关注点的代码，而次要关注点的代码被转移到切面中了。

![]({{ site.url }}/assets/img/spring/14.1.png)


## 2、AOP的术语

​    描述切面的常用术语有通知（advice）、切点（pointcut）和连接点（join point）。

###   2.1、通知

​     在AOP术语中，切面的工作被称为通知。通知定义了**切面是什么以及何时使用**。  除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。

![]({{ site.url }}/assets/img/spring/14.2.png)


###   2.2、连接点

​       我们的应用可能有数以千计的时机应用通知。这些时机被称为**连接点**。连接点是在应用执行过程中能够插入切面的一个点，这个点可以是**调用方法时、抛出异常时、甚至是修改一个字段**。

### 2.3、切点

​        一个切面并不需要通知应用的所有连接点，**切点**的作用就是通知连接点的范围。切点和连接点的区别：具体举个例子：比如开车经过一条高速公路，这条高速公路上有很多个出口（连接点），但是我们不会每个出口都会出去，只会选择我们需要的那个出口（切点）开出去。简单可以理解为，每个出口都是**连接点**，但是我们使用的那个出口才是**切点**。每个应用有多个位置适合织入通知，这些位置都是**连接点**。但是只有我们选择的那个具体的位置才是**切点**。

## 3、Spring的自定义注解

###     3.1、什么是注解？

​       注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。而我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象$Proxy1。通过代理对象调用自定义注解（接口）的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池。

###     3.2、常用的注解

```java

@Target –注解用于什么地方，默认值为任何元素，表示该注解用于什么地方。可用的ElementType指定参数  
  ● ElementType.CONSTRUCTOR:用于描述构造器
  ● ElementType.FIELD:成员变量、对象、属性（包括enum实例）
  ● ElementType.LOCAL_VARIABLE:用于描述局部变量
  ● ElementType.METHOD:用于描述方法
  ● ElementType.PACKAGE:用于描述包
  ● ElementType.PARAMETER:用于描述参数
  ● ElementType.TYPE:用于描述类、接口(包括注解类型) 或enum声明
 
 
@Retention –什么时候使用该注解，即注解的生命周期，使用RetentionPolicy来指定
  ●   RetentionPolicy.SOURCE : 在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们        不会写入字节码。@Override, @SuppressWarnings都属于这类注解。
  ●   RetentionPolicy.CLASS : 在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式
  ●   RetentionPolicy.RUNTIME : 始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。
    
 
@Documented –注解是否将包含在JavaDoc中
 
@Inherited – 是否允许子类继承该注解
    @Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。
```

## 4、用注解创建切面

1、我们写一个自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Perform {

}
```

2、将这个自定义注解打在sing方法上

```java
@Component
public class SingPerform {

    @Perform
    public void sing() {
        System.out.println("sing a song....");
    }
}
```

3、此时我们再写一个切面，切点指向被这个注解标记的任何方法，当执行这个方法是切面将发出通知。

```java
@Component
@Aspect
public class Fans {

    @Pointcut("@annotation(com.example.learning.aop.Perform)")
    public void perform() {
    }

    @Before("perform()")
    public void silenceCellPhones() {
        System.out.println("silence cell phones");
    }

    @AfterReturning("perform()")
    public void applause() {
        System.out.println("Good Good Good");
    }

    @AfterThrowing("perform()")
    public void demandRefund() {
        System.out.println("Bad Bad Bad");
    }
}
```

4、编写单元测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AppConfig.class)
public class AopTest {
    
    @Autowired
    private SingPerform singPerform;

    @Test
    public void testAop() throws Exception {
        singPerform.sing();
    }
}
```

5、结果发现切面确实起了作用
![]({{ site.url }}/assets/img/spring/14.3.png)



## 5、环绕通知

   环绕通知是最强大的通知类型。它能够让你编写的逻辑被通知的目标方法完全包装起来，实际上就像一个通知方法中同时编写前置通知和后置通知。举个例子

```java
   @Around("perform()")
    public void watchPerform(ProceedingJoinPoint point) {
        try {
            System.out.println("silence cell phones");
            point.proceed();
            System.out.println("Good Good Good");
        } catch (Throwable throwable) {
            System.out.println("Bad Bad Bad");
        }
    }
```

结果也和普通的通知一样

![]({{ site.url }}/assets/img/spring/14.4.png)
