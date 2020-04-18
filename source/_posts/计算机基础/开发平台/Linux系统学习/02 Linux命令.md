---
title: 02 Linux命令
toc: false
date: 2020-03-21 22:01:02
tags: [Linux, note]
---

> 阅读刘超老师的《趣谈Linux操作系统》，然后整理了这篇笔记，文章中讲了多种常见的Linux命令。我挑2个我认为挺重要的操作，`运行程序`和`安装软件`的命令整理一下。


### 运行程序
通过命令行让Linux执行`程序`，有以下几种方式，也决定`进程`已什么方式运行。
**1、交互式运行**
```
$ ./filename  # 交互式运行，Ctrl+C可以结束这个在执行的进程
```

**2、后台方式运行**
```bash
# 脱离终端后台运行，并将log输出到xxx.outfile文件。
# nohup命令使终端关闭也不影响进程(进程正常是终端进程fork过来的，父进程挂起....), 2&>1表示将标准输出合并错误输出到xxx.outfile,&设置进程后台运行
$ nohup ./command > xxx.outfile 2&>1 &  		# 后台运行进程

# ps -ef |grep 关键字过滤出进程信息，通过awk '{print $2}'找出进程id，然后通过xargs命令传递给kill -9 ，最终干掉这个进程
$ ps -ef |grep 关键字 |awk '{print $2}'|xargs kill -9 			# kill 这个进程的方法
```


**3、服务方式运行**
```bash
#  systemctl工具管理服务
$ systemctl enable service-name
$ systemctl start service-name
$ systemctl stop service-name
.....
```

*现在有一个小问题*
> Q: 后台运行的进程和服务都是可以脱离终端独立存在的，那么两者有什么区别呢？
> <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
> A：查到[系统服务](https://baike.baidu.com/item/%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1/11027121?fr=aladdin)的概念，总结两点区别：1、服务是系统功能的进程；进程是用户的进程。2、服务不会与用户交互，在后台默默运行(这点和后台进程一样)


<br><br>

### 安装软件
无论是`Ubuntu`系还是`CentOS`系的Linux发行版，总有几种安装软件的方式，`下载安装包`、`通过软件管家`、`直接下载压缩包`或者通过`源码编译`。
**1、下载安装包安装**
```bash
$ dpkg -i xxxx.deb   # 如果是chentos的话，使用rpm命令
```

**2、通过软件管家安装**
```bash
$ apt-get install xxxx    # 如果是chentos的话，使用yum命令
```

**3、下载压缩包安装**
```bash
export PATH=XXX/bin:PATH 			#将可执行文件bin添加到PATH变量，可将这个命令配置在~/.bashrc文件，每次重启Linux加载这个文件
```


**4、源码编译安装**
```bash
# 对当前环境评估，--prefix指定安装路径
$ ./configure --prefix=/usr/local/program

# 编译生成安装包
$ make

# 安装软件
$ make install
```

### 总结
引用文章中总结的图片
![Linux常用命令](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zdGF0aWMwMDEuZ2Vla2Jhbmcub3JnL3Jlc291cmNlL2ltYWdlLzg4L2U1Lzg4NTViYjY0NWQ4ZWNjMzVjODBhYTg5Y2RlNWQxNmU1LmpwZw?x-oss-process=image/format,png)

### 课后作业
课后要求是安装jdk和mysql，搭建一个数据库服务。我没有去做，不过我找到一个部署的教程,很有参考意义:
[使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/chapter-20.html#2021_Mysql)

<br>



## 参考资料
> - [快速上手几个Linux命令：每家公司都有自己的黑话](https://time.geekbang.org/column/article/88761)

