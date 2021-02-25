---
title: Linux性能定位详解
date: 2021-1-8 23:29:53
categories:
- 操作系统
tags:
- 操作系统
---

# 1、CPU

![]({{ site.url }}/assets/img/Linux/1.1.jpg)


- 第一行

> 10:01:23 — 当前系统时间
> 126 days, 14:29 — 系统已经运行了126天14小时29分钟（在这期间没有重启过）
> 2 users — 当前有2个用户登录系统
> load average: 1.15, 1.42, 1.44 —  系统负载。三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。  

​    **load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。** 

- 第二行

> Tasks — 任务（进程），系统现在共有183个进程，其中处于运行中的有1个，182个在休眠（sleep），stoped状态的有0个，zombie状态（僵尸）的有0个。 

   **僵尸进程**:一个子进程在其父进程没有调用wait()或waitpid()的情况下退出。这个子进程就是僵尸进程。如果其父进程还存在而一直不调用wait，则该僵尸进程将无法回收，等到其父进程退出后该进程将被init回收。

- 第三行: cpu状态 

> 6.7% us — 用户空间占用CPU的百分比。
> 0.4% sy — 内核空间占用CPU的百分比。
> 0.0% ni — 改变过优先级的进程占用CPU的百分比
> 92.9% id — 空闲CPU百分比
> 0.0% wa — IO等待占用CPU的百分比
> 0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
> 0.0% si — 软中断（Software Interrupts）占用CPU的百分比 

   在这里CPU的使用比率和windows概念不同，**如果你不理解用户空间和内核空间**，需要充充电了。 

- 第四行: 内存状态 

> 8306544k total — 物理内存总量（8GB）
> 7775876k used — 使用中的内存总量（7.7GB）
> 530668k free — 空闲内存总量（530M）
> 79236k buffers — 缓存的内存量 （79M） 

- 第五行: swap交换分区 

> 2031608k total — 交换区总量（2GB）
> 2556k used — 使用的交换区总量（2.5M）
> 2029052k free — 空闲交换区总量（2GB）
> 4231276k cached — 缓冲的交换区总量（4GB） 

​     第四行中使用中的内存总量（used）指的是现在**系统内核控制的内存数**，空闲内存总量（free）是**内核还未纳入其管控范围的数量**。纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并**不把这些可被重新使用的内存交还到free中去，因此在linux上free内存会越来越少，但不用为此担心**。

  如果出于习惯去计算可用内存数，这里有个近似的计算公式：**第四行的free + 第四行的buffers + 第五行的cached，按这个公式此台服务器的可用内存：530668+79236+4231276 = 4.7GB**。

 对于内存监控，在top里我们要**时刻监控第五行swap交换分区的used**，如果**这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了**。

- 第六行： 各进程（任务）的状态监控 

> PID — 进程id
> USER — 进程所有者
> PR — 进程优先级
> NI — nice值。负值表示高优先级，正值表示低优先级
> VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
> RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
> SHR — 共享内存大小，单位kb
> S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
> %CPU — 上次更新到现在的CPU时间占用百分比
> %MEM — 进程使用的物理内存百分比
> TIME+ — 进程使用的CPU时间总计，单位1/100秒
> COMMAND — 进程名称（命令名/命令行） 

 在top基本视图中，按键盘数字“1”，可监控每个逻辑CPU的状况。

 敲击键盘“b”（打开/关闭加亮效果） 

 敲击键盘“x”（打开/关闭排序列的加亮效果） 

 通过”shift + >”或”shift + <”可以向右或左改变排序列 

 top -d 刷新时间



## 2、内存

 `free`命令习惯上有以下几种形式： 

> ```shell
> free -k # 以KB为单位显示内存使用情况
> free -m # 以MB为单位显示内存使用情况
> free -g # 以GB为单位显示内存使用情况
> free -h # 以人类友好的方式显示内存使用情况
> ```

 当我们输入`free -m`时，系统就会输出以下内容： 

```shell
[root@Test_MC]# free -m

             total       used       free     shared    buffers     cached
Mem:         32168      30119       2048          0       4438      11097
-/+ buffers/cache:      14583      17584
Swap:        31996       1899      30097
```

现在对`free`命令输出的每行进行详细的解释：

- `total`：内存总数，物理内存总数
- `used`：已经使用的内存数
- `free`：空闲的内存数
- `shared`：多个进程共享的内存总额
- `buffers`：缓冲内存数
- `cached`：缓存内存数
- `- buffers/cached`：应用使用内存数
- `+ buffers/cached`：应用可用内存数
- `Swap`：交换分区，虚拟内存

我们通过`free`命令查看机器空闲内存时，会发现`free`的值很小。这主要是因为，在Linux系统中有这么一种思想，内存不用白不用，因此它尽可能的cache和buffer一些数据，以方便下次使用。但实际上这些内存也是可以立刻拿来使用的。

在使用`free`命令时，我们都是需要重点关注`- buffers/cached`和`+ buffers/cached`。

- `- buffers/cached`，即`used - buffers/cached`，表示应用程序实际使用的内存
- `+ buffers/cached`，即`free + buffers/cached`，表示理论上都可以被使用的内存

可见`-buffers/cache`反映的是被程序实实在在吃掉的内存，而`+buffers/cache`反映的是可以挪用的内存总数。

## 3、网络

 netstat -nlpt 

![]({{ site.url }}/assets/img/Linux/1.2.png)


输出信息解释：

> Proto Recv-Q Send-Q Local Address Foreign Address State
>
> 1. Proto：网络连接的协议，一般就是 TCP 协议或者 UDP 协议。
> 2. Recv-Q：表示接收到的数据，已经在本地的缓冲中，但是还没有被进程取走。
> 3. Send-Q：表示从本机发送，对方还没有收到的数据，依然在本地的缓冲中，不具备 ACK 标志的数据包。
> 4. Local Address：本机的 IP 地址和端口号。
> 5. ForeignAddress：远程主机的 IP 地址和端口号。
> 6. State：状态。常见的状态主要有以下几种。
>    -LISTEN：监听状态，只有 TCP 协议需要监听，而 UDP 协议不需要监听。
>    -ESTABLISHED：已经建立连接的状态。如果使用"-I"选项，则看不到已经建立连接的状态。
>    -SYN_SENT：SYN 发起包，就是主动发起连接的数据包。
>    -SYN_RECV：接收到主动连接的数据包。
>    -FIN_WAIT1：正在中断的连接。
>    -FIN_WAIT2：已经中断的连接，但是正在等待对方主机进行确认。
>    -TIME_WAIT：连接已经中断，但是套接字依然在网络中等待结束。
>    -CLOSED：套接字没有被使用。

## 4、IO

   top 命令通过查看 **CPU 的 wa% 值来判断当前磁盘 IO 性能**，如果这个数值过大，很可能是磁盘 IO 太高了，当然也可能是其他原因，例如网络 IO 过高等。

![]({{ site.url }}/assets/img/Linux/1.3.jpg)


iostat 3 10 的意思是每3秒检测一次，一共检测10次

**%iowait 值得注意的一个地方，表示理论上越低表示磁盘越不繁忙。**
tps：每秒事务处理量，也就是没秒磁盘读写IO的次数（可以分为读tps和写tps）。
Blk_read/s：每秒的读的扇区（512byte）数。
Blk_wrtn/s：每秒的写的扇区（512byte）数。

 如果觉得字节扇区单位不直观，**可以用-k 选项** 

显示kB_read/s 就为每秒读的千字节数  kB_wrtn/s 每秒写的千字节数。

 ![]({{ site.url }}/assets/img/Linux/1.4.jpg)


要想显示详细的磁盘IO信息

用iostat –x 命令

![]({{ site.url }}/assets/img/Linux/1.5.jpg)


> rrqm/s：每秒读请求IO合并数。
> wrqm/s：每秒写请求IO合并数。
> r/s：rrqm后，每秒请求读的IO数。
> w/s：rrqm后，每秒请求写的IO数。
> rsec/s：每秒读扇区数。
> wsec/s：每秒写扇区数。
> rsec/s ，wsec/s与不加-x选项中的Blk_read/s，Blk_wrtn/s对应，加上-k 选项也将显示每秒读写的千字节数（rkB/s ，wkB/s）。
> await: 设备平均每次I/O操作花费的时间 (毫秒)。包括在队列中的请求所花费的时间和服务他们所花费的时间。（一般一个10k转的磁盘每次IO的总时间为7-8MS）
> svctm: 向设备发出的I / O请求的平均服务时间（毫秒）。（官网上说不要相信这个数据）。
> %util：一秒中有百分之多少的时间用于 I/O 操作，如果 ％util 接近 100％，说明产生的I/O请求太多，I/O系统已经满负荷。 
