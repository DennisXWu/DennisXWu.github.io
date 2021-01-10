---
title: Redis学习——Redis常用命令
date: 2021-1-8 23:29:53
categories:
- 大数据
tags:
- 大数据
---

## 1、key操作

```shell
#登陆客户端
./redis-cli -c -h ip -p port -a passwd

#检查给定 key 是否存在。
>exists hello
(integer) 1
#设置key的过期时间
> EXPIRE hello 30
(integer) 1
> ttl hello
(integer) 26

#查找所有符合给定模式 pattern 的 key 。
> keys hello
1) "hello"
KEYS * 匹配数据库中所有 key 。
KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。
KEYS h*llo 匹配 hllo 和 heeeeello 等。
KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo 。
#返回 key 所储存的值的类型
1> type hello
string

```

### 1.1、String操作

```shell
#1、字符串操作
> set hello world
OK
> get hello
"world"
> del hello
(integer) 1
> get hello
(nil)
```

### 1.2、List操作

```shell
#List操作
> rpush list-key item
(integer) 1

> rpush list-key item2
(integer) 2

> rpush list-key item
(integer) 3

> lrange list-key 0 -1
1) "item"
2) "item2"
3) "item"

> lindex list-key 1
"item2"

> lpop list-key
"item"

> lrange list-key 0 -1
1) "item2"
2) "item"
```

### 1.3、set操作

```shell
> sadd myset v1
(integer) 1
> sadd myset v2 v3
(integer) 2
> smembers myset
1) "v1"
2) "v3"
3) "v2"
#判断元素是否在集合中
> sismember myset v1
(integer) 1
#删除元素
> srem myset v1
(integer) 1
#获取集合中元素的个数
> scard myset
(integer) 2
#随机获取集合中的元素
> srandmember myset 1
1) "v2"
> srandmember myset -3
1) "v3"
2) "v3"
3) "v3"
#弹出元素
> spop myset
"v2"
> smembers myset
1) "v3"

#差集  {}将两个set放入同一个solt中 sdiffstore差集保留结果
> sdiff {my}set {my}set1
1) "1"
2) "{my}set1"
3) "777"
4) "2"

#并集  sinterstore
> sinter {my}set {my}set1 
1) "1"

#并集   sunionstore
> sunion {my}set {my}set1 
1) "1"
2) "{my}set1"
3) "777"
4) "2"
```

### 1.4、Hash操作

```shell
> HSET myhash field2 "foo2"
(integer) 1

> HGET myhash field1
"foo"

#获取所有的key-value值
> hgetall myhash
1) "field1"
2) "foo"
3) "field2"
4) "foo2"

#删除key
> hdel myhash field1
(integer) 1
```

### 1.5、ZSet操作

```shell
> zadd myzset 20 v2
(integer) 1

> zrange myzset 0 -1
1) "v1"
2) "v2"
3) "v3"

> zrange myzset 0 -2 withscores
1) "v1"
2) "10"
3) "v2"
4) "20"

#删除元素
> zrem myzset v1
(integer) 1
#统计元素
> zcard myzset
(integer) 2
#zincrby ：增减元素的score，格式是：zincrby zset的key 正负数字 项的值
> zincrby myzset -5 v2
"15"
> zrange myzset 0 -1 withscores
1) "v2"
2) "15"
3) "v3"
4) "30"
#zcount ： 获取分数区间内元素个数，格式是：zcount zset的key 起始score 终止score
> zcount myzset 15 30
(integer) 3
#zrank : 获取项在zset中的索引，格式是：zrank zset的key 项的值
> zrank myzset v3
(integer) 1
#zscore ：获取元素的分数，格式是：zscore zset的key 项的值，返回项在zset中的score
> zscore myzset v3
"30"

#交集
zinterstore myzset3 2 myzset myzset2
#并集
zunionstore myzset3 2 myzset myzset2
```

