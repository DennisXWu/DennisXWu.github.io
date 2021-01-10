---
title: Python学习-Anacond的安装搭建
date: 2021-1-8 23:29:53
categories:
- Python
tags:
- Python
---

## 1、什么是 Anacond？

​      Anacond是一个python的发行版，包括了python和很多常见的软件库, 和一个包管理器conda。常见的科学计算类的库都包含在里面了，使得安装比常规python安装要容易。Anaconda是专注于数据分析的Python发行版本，包含了conda、Python等190多个科学包及其依赖项。 

## 2、Anacond下载安装

1、 下载地址：[www.anaconda.com/download/](https://www.anaconda.com/download/)

Anacond是跨平台的，同时支持Windows、macOS、Linux，我这里下载的是Windows X64的安装包
![]({{ site.url }}/assets/img/python/1.1.png)



 双击下载的安装包 ，一步步的Next，这里注意下，官方不建议我们把conda加入到环境变量，我这里默认安装的是python3.7的所以默认就是python3.7了，好了咱们继续Next，由于Anacond里面包含了大量的python的包。

![]({{ site.url }}/assets/img/python/1.2.png)

 至此Anacond安装完成，去对应的Anaconda3的安装目录下的Scripts目录下，执行`conda --version`,如果显示对应的版本信息，那么Anaconda3就安装成功了。 
![]({{ site.url }}/assets/img/python/1.3.png)




## 3、环境变量设置

在`系统环境变量path`中需要添加三个目录，如下(安装目录替换下) 
 `E:\Tool\Anaconda3` 
 `E:\Tool\Anaconda3\Scripts` 
 `E:\Tool\Anaconda3\Library\bin` 
 然后`win+R`打开cmd命令，执行`conda --version` 和`python --version`，查看是否能得到和上面一样的conda的版本信息。

### 3.1、配置国内镜像

- 清华镜像1 `conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/`

- 清华镜像2 `conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/`

- 中科大镜像 `conda config --add channels `

  `https://mirrors.ustc.edu.cn/anaconda/pkgs/free/`

- 检查配置

1、打开配置 `conda config --set show_channel_urls yes`

2、检查配置 `conda config --show channels`

![]({{ site.url }}/assets/img/python/1.4.png)


 配置完成后会在该目录`C:\Users\<你的用户名>`下生成一个对应的`.condarc`文件 

## 4、管理虚拟环境

  创建一个名为python27的环境，指定Python版本是2.7（不用管是2.7.x，conda会为我们自动寻找2.7.x中的最新版本） `conda create --name python27 python=2.7` 

|                    windows                     |                 Linux                 |           作用           |
| :--------------------------------------------: | :-----------------------------------: | :----------------------: |
|                   conda list                   |                                       |     查看安装了哪些包     |
|                 conda env list                 |                                       | 查看当前存在哪些虚拟环境 |
|               conda update conda               |                                       |   检查更新当前的conda    |
|                conda --version                 |                                       |    查看当前conda版本     |
|    conda create -n your_env_name python=X.X    |                                       |   创建python的虚拟版本   |
|             activate your_env_name             | source activate         your_env_name |       激活虚拟环境       |
|    conda install -n your_env_name [package]    |                                       |  对虚拟环境安装额外的包  |
|                   deactivate                   |           source deactivate           |       关闭虚拟环境       |
|            conda remove -n env_name            |                                       |       删除虚拟环境       |
| conda remove --name your_env_name package_name |                                       |    删除环境中的某个包    |
|   conda create -n flowers --clone snowflakes   |                                       |       复制一个环境       |

## 5、参考资料

1、 https://www.jianshu.com/p/edaa744ea47d 

2、 https://juejin.im/post/5c39eb036fb9a049c04340d1 