---
title: Shell学习-sed和awk编程
date: 2021-1-8 23:29:53
categories:
- Shell
tags:
- Shell
---

## 1、Sed的工作原理

​     sed(stream editor)是一个“**非交互式**”面向字符流的编辑器，用来把文档或字符串里面的文字经过一系列编辑命令转换为另一种格式输出，默认情况下，输出到终端屏幕上，也可以经过处理输出到文件中。

​     sed流编辑器在1个文件或1个输入中每次只能处理1行并显示到显示器。该命令在vi编辑器中也可以使用，在称为“**模式空间**”的临时缓冲处理已保存的行。每次处理完临时缓冲的行，该行就传到显示器。结束行处理后，就会从临时缓冲删除，接着读取并处理下一行，然后显示出来。输入文件的最后一行处理完成就终止sed命令。保存于临时缓冲的每行都要被处理，**所以源文件不会被更改或损坏**。

   ![]({{ site.url }}/assets/img/shell/3.1.jpg)


## 2、Sed的用法

sed的命令行格式为：

```shell
sed [-nefri] ‘command’ 输入文本    
```

常用选项：

| 字符 | 意义                                                         |
| :--: | :----------------------------------------------------------- |
|  -n  | 使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN的资料一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。 |
|  -e  | 直接在指令列模式上进行 sed 的动作编辑。                      |
|  -f  | 直接将 sed 的动作写在一个档案内， -f filename 则可以执行 filename 内的sed 动作。 |
|  -r  | sed 的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)。 |
|  -i  | 直接修改读取的档案内容，而不是由萤幕输出 。                  |

常用命令：

| 字符 | 意义                                                         |
| :--: | ------------------------------------------------------------ |
|  a   | 新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行) |
|  c   | 取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行   |
|  d   | 删除，因为是删除啊，所以 d 后面通常不接任何咚咚              |
|  i   | 插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行) |
|  p   | 列印，亦即将某个选择的资料印出。通常 p 会与参数 sed -n 一起运作 |
|  s   | 取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦 |

举例：

```shell
删除某行
[root@localhost ruby] # sed '1d' ab              #删除第一行 
[root@localhost ruby] # sed '$d' ab              #删除最后一行
[root@localhost ruby] # sed '1,2d' ab           #删除第一行到第二行
[root@localhost ruby] # sed '2,$d' ab           #删除第二行到最后一行

显示某行
[root@localhost ruby] # sed -n '1p' ab           #显示第一行 
[root@localhost ruby] # sed -n '$p' ab           #显示最后一行
[root@localhost ruby] # sed -n '1,2p' ab        #显示第一行到第二行
[root@localhost ruby] # sed -n '2,$p' ab        #显示第二行到最后一行

使用模式进行查询
[root@localhost ruby] # sed -n '/ruby/p' ab    #查询包括关键字ruby所在所有行
[root@localhost ruby] # sed -n '/\$/p' ab        #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义

增加一行或多行字符串
[root@localhost ruby]# cat ab
Hello!
ruby is me,welcome to my blog.
end
[root@localhost ruby] # sed '1a drink tea' ab  #第一行后增加字符串"drink tea"
Hello!
drink tea
ruby is me,welcome to my blog. 
end
[root@localhost ruby] # sed '1,3a drink tea' ab #第一行到第三行后增加字符串"drink tea"
Hello!
drink tea
ruby is me,welcome to my blog.
drink tea
end
drink tea
[root@localhost ruby] # sed '1a drink tea\nor coffee' ab   #第一行后增加多行，使用换行符\n
Hello!
drink tea
or coffee
ruby is me,welcome to my blog.
end

代替一行或多行
[root@localhost ruby] # sed '1c Hi' ab                #第一行代替为Hi
Hi
ruby is me,welcome to my blog.
end
[root@localhost ruby] # sed '1,2c Hi' ab             #第一行到第二行代替为Hi
Hi
end

替换一行中的某部分
格式：sed 's/要替换的字符串/新的字符串/g'   （要替换的字符串可以用正则表达式）
[root@localhost ruby] # sed -n '/ruby/p' ab | sed 's/ruby/bird/g'    #替换ruby为bird
[root@localhost ruby] # sed -n '/ruby/p' ab | sed 's/ruby//g'        #删除ruby

插入
[root@localhost ruby] # sed -i '$a bye' ab         #在文件ab中最后一行直接输入"bye"
[root@localhost ruby]# cat ab
Hello!
ruby is me,welcome to my blog.
end
bye

删除匹配行

sed -i '/匹配字符串/d'  filename  （注：若匹配字符串是变量，则需要“”，而不是‘’。记得好像是）

替换匹配行中的某个字符串

sed -i '/匹配字符串/s/替换源字符串/替换目标字符串/g' filename
```

## 3、awk原理

​     awk是一种用于处理文本的编程语言工具，以其作者姓氏第一个字母的组合来命名，可以在命令行使用，但更多作为脚本来使用。**awk可以做更细分的处理，通过指定分隔符将一行（一条记录）划分为多个字段，以字段为单位处理文本**。awk中支持C语法，可以有分支条件判断、循环语句等，相当于一个小型编程语言。

### 4、awk用法

awk命令的格式：

```shell
awk 'BEGIN{ commands } pattern{ commands } END{ commands }'
```

第一步：运行BEGIN{ commands }语句块中的语句。

第二步：从文件或标准输入(stdin)读取一行。然后运行pattern{ commands }语句块，它逐行扫描文件，从第一行到最后一行反复这个过程。直到文件所有被读取完成。

第三步：当读至输入流末尾时。运行END{ commands }语句块。

BEGIN语句块在awk開始从输入流中读取行之前被运行，这是一个可选的语句块，比方变量初始化、打印输出表格的表头等语句通常能够写在BEGIN语句块中。

END语句块在awk从输入流中读取全然部的行之后即被运行。比方打印全部行的分析结果这类信息汇总都是在END语句块中完毕，它也是一个可选语句块。

pattern语句块中的通用命令是最重要的部分，它也是可选的。假设没有提供pattern语句块，则默认运行{ print }，即打印每个读取到的行。awk读取的每一行都会运行该语句块。这三个部分缺少任何一部分都可以。

**awk的环境变量**：

|    变量     | 描述                                    |
| :---------: | --------------------------------------- |
|     $n      | 当前记录的第n个字段，字段间由FS分隔。   |
|     $0      | 完整的输入记录。                        |
|    ARGC     | 命令行参数的数目。                      |
|   ARGIND    | 命令行中当前文件的位置(从0开始算)。     |
|    ARGV     | 包含命令行参数的数组。                  |
|   CONVFMT   | 数字转换格式(默认值为%.6g)              |
|   ENVIRON   | 环境变量关联数组。                      |
|    ERRNO    | 最后一个系统错误的描述。                |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)。            |
|  FILENAME   | 当前文件名。                            |
|     FNR     | 同NR，但相对于当前文件。                |
|     FS      | 字段分隔符(默认是任何空格)。            |
| IGNORECASE  | 如果为真，则进行忽略大小写的匹配。      |
|     NF      | 当前记录中的字段数。                    |
|     NR      | 当前记录数。                            |
|    OFMT     | 数字的输出格式(默认值是%.6g)。          |
|     OFS     | 输出字段分隔符(默认值是一个空格)。      |
|     ORS     | 输出记录分隔符(默认值是一个换行符)。    |
|   RLENGTH   | 由match函数所匹配的字符串的长度。       |
|     RS      | 记录分隔符(默认是一个换行符)。          |
|   RSTART    | 由match函数所匹配的字符串的第一个位置。 |
|   SUBSEP    | 数组下标分隔符(默认值是\034)。          |

**awk的运算符**

| 运算符                  | 描述                                 |
| ----------------------- | ------------------------------------ |
| = += -= *= /= %= ^= **= | 赋值                                 |
| ?:                      | C条件表达式                          |
| \|\|                    | 逻辑或                               |
| &&                      | 逻辑与                               |
| **~      ~!**           | 匹配正则表达式和**不匹配正则表达式** |
| < <= > >= != ==         | 关系运算符                           |
| 空格                    | 连接                                 |
| + -                     | 加，减                               |
| * / &                   | 乘，除与求余                         |
| + - !                   | 一元加，减和逻辑非                   |
| ^ ***                   | 求幂                                 |
| ++ --                   | 增加或减少，作为前缀或后缀           |
| $                       | 字段引用                             |
| in                      | 数组成员                             |

