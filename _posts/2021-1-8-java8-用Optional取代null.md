---
title: Java8学习—用Optional取代null
date: 2021-1-8 23:29:53
categories:
- java8
tags:
- java8
---

## 1、Optional类入门

​       Optional类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。*被Optional包装的对象最多只有一个*。

## 2、创建optional对象

###      2.1、声明一个空的Optional

​              可以通过静态工厂方法Optional.empty创建一个**空**的Optioanl对象：


```java
  Optional<Car>  car=Optional.empty();
```

###      2.2、依据一个非空值创建Optional

​               还可以通过静态工厂方法Optional.of，依据一个**非空**值创建一个Optional对象：

```java
      Optional<Car>  car=Optional.of(car);
```

​               如果car是一个null，这段代码将会立即抛出一个**空指针异常**。

###       2.3、可接受null的Optional

​              还可以通过静态工厂方法Optional.ofNullable，可以创建一个允许null值的Optional对象：


```java
      Optional<Car>  car=Optional.ofNullable(car);
```

​              如果car是null，那么得到的Optional对象就是一个空对象。

###       2.4、使用map从Optional对象中提取和转换值

​        工作方式如下：

```java
         Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
         Optional<String> name = optInsurance.map(Insurance::getName); 
```

###      2.5、使用flatMap从Optional对象中提取和转换值


```java
  Optional<Person> optPerson = Optional.of(person);
   Optional<String> name =optPerson.map(Person::getCar)
                                   .map(Car::getInsurance)
                                   .map(Insurance::getName);
```

​      这段代码无法通过编译，optPerson是Optional<Person>类型的变量， 调用map方法应该没有问题。但getCar返回的是一个Optional<Car>类型的对象，这意味着map操作的结果是一个Optional<Optional<Car>>类型的对象。因此，它对getInsurance的调用是非法的，因为最外层的Optional对象包含了另一个Optional
对象的值，而它当然不会支持getInsurance方法 。必须要使用flatMap来进行转换：

```java
 public String getCarInsuranceName(Optional<Person> person) {
      return person.flatMap(Person::getCar)
                   .flatMap(Car::getInsurance)
                   .map(Insurance::getName)
                   .orElse("Unknown");
}
```

###       2.6、常用的Optioanl操作方法

​       （1）get()是这些方法中最简单但又最不安全的方法。如果变量存在，它直接返回封装的变量值，否则就抛出一个NoSuchElementException异常。 

​      （2） orElse(T other)允许你在Optional对象不包含值时提供一个默认值 。

​      （3）orElseGet(Supplier<? extends T> other)是orElse方法的延迟调用版，Supplier方法只有在Optional对象不含值时才执行调用 。

​      （4）ifPresent(Consumer<? super T>)让你能在变量值存在时执行一个作为参数传入的方法，否则就不进行任何操作。

​      （5）isPresent()方法，如果Optional对象包含值，该方法就返回true 。

​        （6）  filter方法接受一个谓词作为参数 ,如果Optional对象的值存在，并且它符合谓词的条件，filter方法就返回其值；否则它就返回一个空的Optional对象 。

​      
