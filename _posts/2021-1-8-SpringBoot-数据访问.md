---
title: SpringBoot学习-数据访问
date: 2021-1-8 23:29:53
categories:
- Spring
tags:
- Spring
---

## 1、Spring Data

​    Spring Data项目是Spring用来解决数据访问问题的一揽子解决方案，Spring Data是一个伞形项目，包含了大量关系型数据库及非关系型数据库的数据访问解决方案。Spring通过依赖Spring Data Commons项目来实现**数据访问操作**，Spring Data Commons让我们使用关系型或非关系型数据访问技术时都使用**基于Spring的统一标准**，**该标准包含CRUD操作**。

## 2、Spring Data JPA

### 2.1、什么是Spring Data JPA？

​    JPA即Java Persistence API。JPA是一个基于**O/R**（即将领域模型和数据库的表进行映射）映射的标准规范。所谓规范即只定义标准规则，不提供实现，软件提供商可以按照标准规范来实现，而使用者只需按照规范中定义的方式来使用。

### 2.2、如何使用JPA?

使用Spring Data JPA建立数据访问层十分简单，只需定义一个继承JpaRepository的接口即可。

```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {
}
```

​    在Spring环境中，使用Spring Data JPA可通过**@EnableJpaRepositories**注解来开发Spring Data JPA的支持。

1、JPA的查询关键字如下所示：

![]({{ site.url }}/assets/img/spring/5.1.png)



2、使用@Query查询

使用参数索引，Spring Data JPA还支持用@Query注解在接口的方法上实现查询。

```java
public interface UserDefineBySelf extends JpaRepository<User, Integer> { 
  /** 
   * 命名参数 
   * 描述：推荐使用这种方法，可以不用管参数的位置 
   */
  @Query("select u from User u where u.name = :name") 
  User findUserByName(@Param("name") String name); 
    
  /** 
   * 索引参数 
   * 描述：使用?占位符 
   */
  @Query("select u from User u where u.email = ?1")// 1表示第一个参数 
  User findUserByEmail(String email); 
    
  /** 
   * 描述：可以通过@Modifying和@Query来实现更新 
   * 注意：Modifying queries的返回值只能为void或者是int/Integer 
   */
  @Modifying
  @Query("update User u set u.name = :name where u.id = :id") 
  int updateUserById(@Param("name") String name, @Param("id") int id); 
} 
```

## 3、声明式事务

   Spring支持声明式事务，即使用注解来选择需要的使用事务的方法，它使用**@Transactional**注解在方法上表明该方法需要事务支持。Spring提供了一个**@EnableTransactionManagement**注解在配置类上来开启声明式事务的支持。

|          参数名称          | 功能描述                                                     |
| :------------------------: | :----------------------------------------------------------- |
|        **readOnly**        | 该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。例如：@Transactional(readOnly=true) |
|      **rollbackFor**       | 该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。 |
|  **rollbackForClassName**  | 该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。 |
|     **noRollbackFor**      | 该属性用于设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。 |
| **noRollbackForClassName** | 该属性用于设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。 |
|      **propagation**       | 该属性用于设置事务的传播行为，具体取值可参考表6-7。          |
|       **isolation**        | 该属性用于设置底层数据库的事务隔离级别，事务隔离级别用于处理多事务并发的情况，通常使用数据库的默认隔离级别即可，基本不需要进行设置。 |
|        **timeout**         | 该属性用于设置事务的超时秒数，默认值为-1表示永不超时         |

## 4、Spring的缓存技

  CacheManager是Spring提供的各种缓存技术抽象类接口，针对不同的缓存技术，需要实现不同的CacheManager。

![]({{ site.url }}/assets/img/spring/5.2.png)


Spring提供了4个注解来声明缓存规则

![]({{ site.url }}/assets/img/spring/5.3.png)


 @Cacheable 、@CacheEvict 、 @CachePut都有**value属性**，指定是要使用的缓存名称。**key属性**指定的数据在缓存中存储的键。 有的时候我们可能并不希望缓存一个方法所有的返回结果。**通过condition属性**可以实现这一功能。 



## 5、参考资料

1、 https://www.cnblogs.com/joe-/p/10340531.html 
2、《JavaEE开发的颠覆者 SpringBoot实战》
