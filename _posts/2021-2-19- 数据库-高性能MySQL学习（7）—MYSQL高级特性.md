---
title: 高性能MySQL学习（7）—MYSQL高级特性
date: 2021-2-19 23:29:53
categories:
- 数据库
tags:
- 数据库
---

## 1、分区表

### 1.1、分区表介绍

>   分区表是一个独立的逻辑表，但是底层由多个物理字表组成。实现分区的代码实际上是对一组底层表的句柄对象的封装，对于分区表的请求，都会通过句柄对象转化成存储引擎的接口调用。所以分区对于SQL层来说是一个完全封装底层实现的黑盒子，对应用是透明的，但是从底层来看每个分区表都有一个使用#分隔命名的表文件。

MySQL实现分区表的方式——对底层表的封装——意味着索引也是按照分区的字表定义，没有全局索引。

MySQL在创建表时用**PARTITION BY 子句定义每个分区存放的数据**。在执行查询时，优化器根据分区定义**过滤那些没有我们需要数据的分区**，这样查询无须扫描所有分区，只需要找到包含需要数据的分区就可以了。

使用场景和优点：

1、 表非常大以至于无法全部存放在内存中，获取的数据比较少

2、 数据更容易维护，删除、更新方便

3、 分区表的数据可以存放在不同的物理设备上。

4、 备份和恢复独立的分区。

限制：

1、 一个表最多分区1024个

2、 分区字段中有主键或者唯一索引的列，所有主键列和唯一索引必须包含进来

3、 分区表无法使用外键约束

### 1.2、分区表原理

  分区表由多个相关的底层表实现，这些底层表是由句柄对象来表示，可以直接访问分区，分区表的索引只是在各个底层表上各自加上一个完全相同的索引，从存储引擎上看，分区表和普通表没有什么不同。

**SELECT查询**：

打开并锁住所有的底层表，优化器判断是否会过滤部分分区，然后调用对应的存储引擎接口访问各个分区的数据。

**INSERT操作：**

打开并锁住所有底层表，确定哪个分区接收记录，写入底层表

**DELETE操作：**

打开并锁住所有底层表，确定数据对应的分区，删除操作

**UPDATE操作：**

打开并锁住所有的底层表，确定更新的分区，取出更新，判断更新后的数据应该存放的分区位置，写入操作。

### 1.3、分区类型

目前MySQL支持**范围分区**（RANGE），**列表分区**（LIST），**哈希分区**（HASH）以及**KEY分区**四种。下面我们逐一介绍每种分区：

- **RANGE分区**

基于属于一个给定连续区间的列值，把多行分配给分区。最常见的是基于**时间字段**. 基于**分区的列最好是整型**，如果日期型的可以使用函数转换为整型。本例中使用to_days函数

```sql
CREATE TABLE my_range_datetime(
    id INT,
    hiredate DATETIME
) 
PARTITION BY RANGE (TO_DAYS(hiredate) ) (
    PARTITION p1 VALUES LESS THAN ( TO_DAYS('20171202') ),
    PARTITION p2 VALUES LESS THAN ( TO_DAYS('20171203') ),
    PARTITION p3 VALUES LESS THAN ( TO_DAYS('20171204') ),
    PARTITION p4 VALUES LESS THAN ( TO_DAYS('20171205') ),
    PARTITION p5 VALUES LESS THAN ( TO_DAYS('20171206') ),
    PARTITION p6 VALUES LESS THAN ( TO_DAYS('20171207') ),
    PARTITION p7 VALUES LESS THAN ( TO_DAYS('20171208') ),
    PARTITION p8 VALUES LESS THAN ( TO_DAYS('20171209') ),
    PARTITION p9 VALUES LESS THAN ( TO_DAYS('20171210') ),
    PARTITION p10 VALUES LESS THAN ( TO_DAYS('20171211') )，
    PARTITION p11 VALUES LESS THAN (MAXVALUE) 
);
```

   p11是一个默认分区，所有大于20171211的记录都会在这个分区。MAXVALUE是一个无穷大的值。p11是一个可选分区。如果在定义表的没有指定的这个分区，当我们插入大于20171211的数据的时候，会收到一个错误。

我们在执行查询的时候，必须带上分区字段。这样可以使用分区剪裁功能

```sql
mysql> explain partitions select * from my_range_datetime where hiredate >= '20171207124503' and hiredate<='20171210111230'; 
+----+-------------+-------------------+--------------+------+---------------+------+---------+------+--------+-------------+
| id | select_type | table             | partitions   | type | possible_keys | key  | key_len | ref  | rows   | Extra       |
+----+-------------+-------------------+--------------+------+---------------+------+---------+------+--------+-------------+
|  1 | SIMPLE      | my_range_datetime | p7,p8,p9,p10 | ALL  | NULL          | NULL | NULL    | NULL | 400061 | Using where |
+----+-------------+-------------------+--------------+------+---------------+------+---------+------+--------+-------------+
1 row in set (0.03 sec)
```

注意执行计划中的partitions的内容，只查询了p7，p8，p9，p10三个分区，由此来看，使用to_days函数确实可以实现分区裁剪。

- **LIST分区**

​     LIST分区和RANGE分区类似，区别在于**LIST是枚举值列表的集合，RANGE是连续的区间值的集合**。二者在语法方面非常的相似。同样**建议LIST分区列是非null列，否则插入null值如果枚举列表里面不存在null值会插入失败**，这点和其它的分区不一样，**RANGE分区会将其作为最小分区值存储，HASH\KEY分为会将其转换成0存储，主要LIST分区只支持整形，非整形字段需要通过函数转换成整形.**

```sql
create table t_list( 
　　a int(11), 
　　b int(11) 
　　)(partition by list (b) 
　　partition p0 values in (1,3,5,7,9), 
　　partition p1 values in (2,4,6,8,0) 
　　);
```

- **HASH分区**

​     我们在实际工作中经常遇到像会员表的这种表。并没有明显可以分区的特征字段。但表数据又非常庞大。为了把这类的数据进行分区打散mysql 提供了hash分区。基于给定的分区个数，将数据分配到不同的分区，**HASH分区只能针对整数进行HASH，对于非整形的字段只能通过表达式将其转换成整数**。表达式可以是mysql中任意有效的函数或者表达式，对于非整形的HASH往表插入数据的过程中会多一步表达式的计算操作，所以不建议使用复杂的表达式这样会影响性能。

Hash分区表的基本语句如下：

```sql
CREATE TABLE my_member (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    created DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY HASH(id)
PARTITIONS 4;
```

注意：
1. HASH分区可以不用指定PARTITIONS子句，如上文中的PARTITIONS 4，则**默认分区数为1**。
2. 不允许只写PARTITIONS，而不指定分区数。
3. 同RANGE分区和LIST分区一样，PARTITION BY HASH (expr)子句中的**expr返回的必须是整数值**。
4. HASH分区的底层实现其实是基于MOD函数。譬如，对于下表

```sql
CREATE TABLE t1 (col1 INT, col2 CHAR(5), col3 DATE) PARTITION BY HASH( YEAR(col3) ) PARTITIONS 4; 
```

如果你要插入一个col3为“2017-09-15”的记录，则分区的选择是根据以下值决定的：
MOD(YEAR(‘2017-09-01’),4) = MOD(2017,4) = 1

- **KEY分区**

KEY分区其实跟HASH分区差不多，不同点如下：
1. KEY分区允许多列，而HASH分区只允许一列。

2. 如果在有主键或者唯一键的情况下，key中分区列可不指定，默认为主键或者唯一键，如果没有，则必须显性指定列。

3. KEY分区对象必须为列，而不能是基于列的表达式。

4. KEY分区和HASH分区的算法不一样，PARTITION BY HASH (expr)，MOD取值的对象是expr返回的值，而PARTITION BY KEY (column_list)，基于的是列的MD5值。

格式如下：
```sql
CREATE TABLE k1 (
    id INT NOT NULL PRIMARY KEY,    
    name VARCHAR(20)
)
PARTITION BY KEY()
PARTITIONS 2;
```

**总结：**
1. MySQL分区中如果存在主键或唯一键，则分区列必须包含在其中。
2. 对于原生的RANGE分区，LIST分区，HASH分区，分区对象返回的只能是整数值。
3. 分区字段不能为NULL，要不然怎么确定分区范围呢，所以尽量NOT NULL

### 1.4、什么情况下会出问题
1. NULL值会使分区过滤无效。
2. 分区列和索引列不匹配：如果定义的索引索引列和分区列不匹配，会导致查询无法进行分区过滤。
3. 选择分区的成本可能很高：可以通过限制分区数量来缓解此问题，根据经验，对大多数系统来首，100个所有的分区是没有问题的。
4. 打开并锁住所有底层表的成本可能很高：这个操作在分区过滤之前发生。
5. 维护分区的成本可能很高：例如重组分区。

### 1.5、查询优化
1. 分区最大的优点就是优化器可以根据分区函数来过滤一些分区。根据粗粒度索引的优势，通过分区过滤通常可以让查询扫描更少的数据(在某些场景下)。所以，对于访问分区表来说，很重要的一点是要在WHERE。
2. 条件带入到分区列中。有时候即使看似多余的也要带上，这样就可以让优化器过滤掉无需访问的分区。如果没有这些条件，MySQL就需要让对应存储引擎访问这个表的所有分区，如果表非常大，可能会非常慢。
3.  MySQL只能在使用分区函数的列本身进行比较时才能过滤分区，而不能根据表达式的值去过滤分区，即使这个表达式就是分区函数也不行。
4. 一个很重要的原则是：即便在创建分区时可以使用表达式，但在查询时却只能根据列来过滤分区。

## 2、视图

### 2.1、视图的概念

>    视图是一个虚拟表，**是从数据库中一个或多个表中导出来的表，其内容由查询定义**。同真实表一样，视图包含一系列带有名称的列和行数据。但是，数据库中只存放了视图的定义，而并没有存放视图中的数据。这些数据存放在原来的表中。使用视图查询数据时，数据库系统会从原来的表中取出对应的数据。因此，视图中的数据是依赖于原来的表中的数据的。一旦表中的数据发生改变，显示在视图中的数据也会发生改变。

视图是存储在数据库中的查询的SQL语句，它主要出于两种原因：**安全原因**，视图可以隐藏一些数据，例如，员工信息表，可以用视图只显示姓名、工龄、地址，而不显示社会保险号和工资数等；另一个原因是可**使复杂的查询易于理解和使用**。

### 2.2、视图的作用

   对其中所引用的基础表来说，视图的作用类似于**筛选**。定义视图的筛选可以来自当前或其他数据库的一个或多个表，或者其他视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。视图的作用归纳为如下几点。

**1、简单性**

   看到的就是需要的。视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件。

**2、安全性**

视图的安全性可以防止未授权用户查看特定的行或列，使有权限用户只能看到表中特定行的方法，如下：
（1）在表中增加一个标志用户名的列。
（2）建立视图，使用户只能看到标有自己用户名的行。
（3）把视图授权给其他用户。

**3、逻辑数据独立性**

视图可以使应用程序和数据库表在一定程度上独立。如果没有视图，程序一定是建立在表上的。有了视图之后，程序可以建立在视图之上，从而程序与数据库表被视图分割开来。视图可以在以下几个方面使程序与数据独立。

（1）如果应用建立在数据库表上，当数据库表发生变化时，可以在表上建立视图，通过视图屏蔽表的变化，从而使应用程序可以不动。

（2）如果应用建立在数据库表上，当应用发生变化时，可以在表上建立视图，通过视图屏蔽应用的变化，从而使数据库表不动。

（3）如果应用建立在视图上，当数据库表发生变化时，可以在表上修改视图，通过视图屏蔽表的变化，从而使应用程序可以不动。

（4）如果应用建立在视图上，当应用发生变化时，可以在表上修改视图，通过视图屏蔽应用的变化，从而使数据库可以不动。

### 2.3、视图的实现算法
merge: 合并算法，尽可能使用这个

Temptable：临时表算法。如果视图中包含GROUP BY, DISTINCT, 任何聚合函数, UNION, 子查询等只要无法在原表记录和视图记录建立一一映射的场景中，mysql都将用临时表算法实现视图。

![image]({{ site.url }}/assets/img/数据库/7.1.png)

### 2.4、视图的操作

- **创建视图**

MySQL中，创建视图是通过CREATE VIEW语句实现的。其语法如下：
```sql
CREATE [OR REPLACE] [ALGORITHM={UNDEFINED|MERGE|TEMPTABLE}]
VIEW 视图名[(属性清单)]
AS SELECT语句
[WITH [CASCADED|LOCAL] CHECK OPTION];
```
参数说明：
（1）ALGORITHM：可选项，表示视图选择的算法。
（2）视图名：表示要创建的视图名称。
（3）属性清单：可选项，指定视图中各个属性的名词，默认情况下与SELECT语句中的查询的属性相同。
（4）SELECT语句：表示一个完整的查询语句，将查询记录导入视图中。
（5）WITH CHECK OPTION：可选项，表示更新视图时要保证在该视图的权限范围之内。

- **修改视图**

修改视图是指修改数据库中存在的视图，当基本表的某些字段发生变化时，可以通过修改视图来保持与基本表的一致性。

MySQL中通过CREATE OR REPLACE VIEW语句和ALTER VIEW语句来修改视图。
```sql
ALTER VIEW view_user
AS
	SELECT id,name FROM tb_user where id  in (select id from tb_user);
```

- **删除视图**

删除视图是指删除数据库中已存在的视图。删除视图时，只能删除视图的定义，不会删除数据。MySQL中，使用DROP VIEW语句来删除视图。但是，用户必须拥有DROP权限。
```sql
DROP VIEW IF EXISTS view_user;
```

## 3、外键约束

**实体完整性**是通过主键约束实现的，而参照完整性是通过外键约束实现的，两者都是为了**保证数据的完整性和一致性**。

### 3.1、**主表与从表**

若同一个数据库中，B表的外键与A表的主键相对应，则A表为主表，B表为从表。

假设学生表（学号，姓名，性别，专业号），专业表（专业号，专业名称），则学生表中的专业号为学生表的外键，其与专业表中“专业号”属性相关联，因此，专业表为主表，学生表为从表。

### 3.2、**外键约束**

外键约束是相关联的两个表之间的数据操作约束，包括删除，插入，更新等。理论上，在对关联数据表进行数据操作时，只改其一，不改其二，不符合关系数据库的参照完整性。

（1）更新
更新主表的某一个记录的主键值（其实，这种操作是不被允许的），系统会自动检测该主键值在从表中是否存在，若存在，则需要明确操作（一般默认为不被允许）；
更新从表的某一个记录的外键值，系统会自动检测欲更新的外键值在主表中是否存在，若不存在，则需要明确操作（一般默认为不被允许）；

（2）插入
向主表中插入一条新的记录，不会对现有从表造成影响；
向从表中插入一条新的记录，系统会检测外键对应的属性值在主表中是否存在，若存在，否则需要明确操作（一般默认为不被允许）；

（3）删除
从主表中删除一条记录，系统会自动检测该记录的主键值是否在从表中存在，若存在，则需要明确操作（一般默认为不被允许）；
从从表中删除一条记录，不会对主表造成影响；

### 3.3、外键约束的四种类型

  **CASCADE:** 从父表中删除或更新对应的行，同时自动的删除或更新子表中匹配的行。ON DELETE CANSCADE和ON UPDATE CANSCADE都被InnoDB所支持。

   **SET NULL:** 从父表中删除或更新对应的行，同时将子表中的外键列设为空。注意，这些在外键列没有被设为NOT NULL时才有效。ON DELETE SET NULL和ON UPDATE SET SET NULL都被InnoDB所支持。

   **NO ACTION:** InnoDB拒绝删除或者更新父表。

   **RESTRICT:** 拒绝删除或者更新父表。指定RESTRICT（或者NO ACTION）和忽略ON DELETE或者ON UPDATE选项的效果是一样的。

## 4、在mysql内部存储代码

通过**触发器、存储过程、函数**的形式存储代码，5.1开始，可在定时任务中存代码
不同类型的存储代码区别：执行上下文（输入输出），存储过程/函数都可以接收参数然后返回值，但是触发器和事件却不行；

## 5、游标

  游标实际上是**一种能从包括多条数据记录的结果集中每次提取一条记录的机制**。

  游标充当**指针**的作用。

  尽管游标能遍历结果中的所有行，但他一次只指向一行。

  游标的作用就是用于对查询数据库所返回的记录进行遍历，以便进行相应的操作。

## 6、绑定变量

​    当创建一个绑定变量 SQL 时，客户端会向服务器发送一个SQL语句的原型。服务器端收到这个SQL语句框架后，解析并存储这个SQL语句的部分执行计划，返回个客户端一个 SQL 语句处理句柄。以后每次执行这类查询，客户端都指定使用这个句柄。
   这样一个更新的sql语句分成了两部，第一步是把带?的sql语句发送到服务器做**预编译**。第二步就是设置参数并且执行了。
```sql
PreparedStatement pstmt = con.prepareStatement("UPDATE table4 SET m = ? WHERE x = ?");
pstmt.setString(1, "Hi");
for (int i = 0; i < 10; i++) {
pstmt.setInt(2, i);
int rowCount = pstmt.executeUpdate();
}
```

## 7、字符集和校对

**字符集**：是指一种从二进制编码到某类字符符号的映射，可以参考如何使用一个字节来表示英文字母。

**校对：**是指一组用于某个字符集的排序规则。

MySQL 4.1 和之后的版本中，每一类编码字符都有对应的字符集和校对规则。

每种字符集都可能有多种校对规则，并且都有一个默认的校对规则。

每个校对规则都是针对某个特定的字符集的，和其他的字符集没有关系。

MySQL 的设置可以分为两类：**创建对象时的默认值**、**在服务器和客户端通信时的设置**。

## 8、分布式事务

​     分布式事务通常采用2PC协议，全称Two Phase Commitment Protocol。该协议主要为了解决在分布式数据库场景下，所有节点间数据一致性的问题。在分布式事务环境下，事务的提交会变得相对比较复杂，因为多个节点的存在，可能存在部分节点提交失败的情况，即事务的ACID特性需要在各个数据库实例中保证。总而言之，在分布式提交时，**只要发生一个节点提交失败，则所有的节点都不能提交**，只有当所有节点都能提交时，整个分布式事务才允许被提交。
分布式事务通过2PC协议将提交分成两个阶段
1. prepare
2. commit/rollback

第一阶段的prepare只是用来询问每个节点事务是否能提交，只有当得到所有节点的“许可”的情况下，第二阶段的commit才能进行，否则就rollback。需要注意的是：prepare成功的事务，则必须全部提交。

## 9、查询缓存

### 9.1、mysql如何判断缓存命中

​    缓存存放在一个引用表中，通过一个哈希值引用，这个哈希值包括查询本身，数据库，客户端协议的版本等，**任何字符上的不同，例如空格，注释都会导致缓存不命中**。

当查询中有一些不确定的数据时，是不会缓存的，比方说now()，current_date()，自定义函数，存储函数，用户变量，字查询等。所以这样的查询也就不会命中缓存，但是还会去检测缓存的，因为**查询缓存在解析SQL之前**，所以MySQL并不知道查询中是否包含该类函数，只是不缓存，自然不会命中。

打开Qcache对读和写都会带来额外的消耗：
a、读查询开始之前必须检查是否命中缓存。
b、如果读查询可以缓存，那么执行完之后会写入缓存。
c、**当向某个表写入数据的时候，必须将这个表所有的缓存设置为失效**，如果缓存空间很大，则消耗也会很大，可能使系统僵死一段时间，因为这个操作是靠全局锁操作来保护的。
   对InnoDB表，当修改一个表时，设置了缓存失效，但是多版本特性会暂时将这修改对其他事务屏蔽，**在这个事务提交之前，所有查询都无法使用缓存，直到这个事务被提交，所以长时间的事务，会大大降低查询缓存的命中**。

### 9.2、查询缓存如何使用内存
MySQL用于查询的缓存的内存被分成一个个变长数据块，用来存储类型，大小，数据等信息。
当服务器启动的时候，会初始化缓存需要的内存，是一个完整的空闲块。当查询结果需要缓存的时候，先从空闲块中申请一个数据块大于参数query_cache_min_res_unit的配置，即使缓存数据很小，申请数据块也是这个，因为查询开始返回结果的时候就分配空间，此时无法预知结果多大。
分配内存块需要先锁住空间块，所以操作很慢，MySQL会尽量避免这个操作，选择尽可能小的内存块，如果不够，继续申请，如果存储完时有空余则释放多余的。
![image]({{ site.url }}/assets/img/数据库/7.2.png)
但是如果并发的操作，余下的需要回收的空间很小，小于query_cache_min_res_unit，不能再次被使用，就会产生**碎片**。如图：
![image]({{ site.url }}/assets/img/数据库/7.3.png)

### 9.3、什么情况下查询缓存能发挥作用

开启查询缓存可能使一个查询提高性能，但也可能影响其他查询。

a、**对于消耗很大的查询通常都是非常适合缓存的**，例如汇总计算查询。

b、**对于update,delete,insert特别频繁的表不适合使用**。

一个判断查询缓存是否命中的直接数据是**命中率**，就是查询缓存返回结果占从查询的比率，当MySQL接受到一个查询的时候**要么增加Qcache_hits的值要么增加Com_select的值**。
任何select没有命中缓存有以下一个可能

a、**查询语句包含函数或者结果太大超过配置无法被缓存**。

b、**查询没有被缓存过**。

c、**内存不足，被新的缓存替代，或者表的数据或结构变化**。

缓存碎片，内存不足，数据修改都会造成缓存失效。
可以使用Qcache_lowmem_prunes来查看有多少失效是由于内存不足导致的。
如果缓存的结果没有被任何select使用，那么这次缓存就是浪费时间和内存，可以通过查看Com_select和Qcache_inserts的值来看。如果每次查询都没有命中，然后查询结果还要写入缓存，那么这2个值应该差不多，而我们希望看到Qcache_inserts远远小于Com_select。

### 9.4、innodb和查询缓存

事务是否可以访问查询缓存取决于当前事务的ID，以及对应表上是否有锁。
当表上任何锁的时候，那么这个表的任何查询都无法被缓存的。
a、所有大于表内存字典中的事务ID的才可以使用查询缓存。例如当前系统事务ID是5，那么1-4的事务是不能使用查询缓存的。
b、实际上表的ID是系统版本号，所以当前事务自身后续的更新操作也无法都去和修改查询缓存。