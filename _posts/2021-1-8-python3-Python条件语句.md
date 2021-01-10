---
title: Python学习-Python条件语句
date: 2021-1-8 23:29:53
categories:
- Python
tags:
- Python
---

## 1、条件语句

if条件语句得格式如下：

```python
if 判断条件：
    执行语句……
else：
    执行语句……
```

 if 语句的判断条件可以用>（大于）、<(小于)、==（等于）、>=（大于等于）、<=（小于等于）来表示其关系。 

 当判断条件为多个值时，可以使用以下形式： 

```python
if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……
```

if语句还可以嵌套：

```python
if 条件1:
    条件1满足执行的动作
    if 满足条件1的基础上的条件2:
        ...
    else:
    条件2不满足的情况下
else:
    条件1不满足时，执行的动作
```

## 2、switch语句

 我们知道Python中没有类似C++或者Java中的`switch...case`语句，我们可以使用多个`if...elif...else`进行模拟，但是这样的写法让代码看起来很凌乱，个人不是很推荐在代码中大量使用`if`语句。 

   我们可以使用字典（dict）得get方法来实现switch得功能。

```python
def switch_case(value):
    switcher = {
        0:"zero",
        1:"one",
        2:"two",
    }
    return switcher.get(value,"wrong value")
```

## 3、循环语句

### 3.1、while循环

 Python 编程中 while 语句用于循环执行程序，即在某条件下，循环执行某段程序，以处理需要重复处理的相同任务。 

```python
#!/usr/bin/python
 
count = 0
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"
```

其中，else可以省略，while语句还有另外两个重要得命令continue，break来跳过循环，continue用于跳过该次循环，break则是用于退出循环。

### 3.2、for循环

Python for循环可以遍历任何序列得项目，如一个列表或者一个字符串。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
for letter in 'Python':     # 第一个实例
   print '当前字母 :', letter
 
fruits = ['banana', 'apple',  'mango']
for fruit in fruits:        # 第二个实例
   print '当前水果 :', fruit
 
print "Good bye!"
```

 在 python 中，for … else 表示这样的意思，for 中的语句和普通的没有区别，else 中的语句会在循环正常执行完（即 for 不是通过 break 跳出而中断的）的情况下执行，while … else 也是一样。 

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
for num in range(10,20):  # 迭代 10 到 20 之间的数字
   for i in range(2,num): # 根据因子迭代
      if num%i == 0:      # 确定第一个因子
         j=num/i          # 计算第二个因子
         print '%d 等于 %d * %d' % (num,i,j)
         break            # 跳出当前循环
   else:                  # 循环的 else 部分
      print num, '是一个质数'
```

python range()函数可以创建一个整数列表，一般用在for循环中：

```python
range(start, stop[, step])
```

参数说明：

- start: 计数从 start 开始。默认是从 0 开始。例如range（5）等价于range（0， 5）;
- stop: 计数到 stop 结束，但不包括 stop。例如：range（0， 5） 是[0, 1, 2, 3, 4]没有5
- step：步长，默认为1。例如：range（0， 5） 等价于 range(0, 5, 1)
