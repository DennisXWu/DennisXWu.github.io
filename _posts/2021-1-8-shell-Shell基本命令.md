---
title: Shell学习-sed和awk编程
date: 2021-1-8 23:29:53
categories:
- Shell
tags:
- Shell
---

# 1、Vim编辑器

![]({{ site.url }}/assets/img/shell/4.1.png)


## 2、I/O重定向和管道

### 2.1、标准输出

![]({{ site.url }}/assets/img/shell/4.2.png)


  2>&1表示将标准错误（2）也传递到标准输出（1）传递的位置。通常使用“执行命令”>/dev/null 2>&1格式将标准错误信息传递到/dev/null设备上，**该重定向实例删除了所有执行命令相关的标准错误和标准输出信息**。

```shell
cat linuxer.txt > /dev/null 2>&1
cat test2.txt 2>/dev/null >/dev/null 
```

### 2.2、标准输入

标准输入是通过键盘输入数据，也可以不适用键盘，而通过文件输入：

```shell
sort <  [要输入的文件名]
```

### 2.3、管道

 表示前一个命令的结果值将称为之后写入的指令的输入。这就是**管道**，也就是说，一个由管道连接的标准输出将成为另外一个命令的标准输入。

![]({{ site.url }}/assets/img/shell/4.3.png)






