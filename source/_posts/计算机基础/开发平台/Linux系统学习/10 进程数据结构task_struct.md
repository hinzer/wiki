---
title: 10 进程数据结构task_struct
toc: false
date: {{ date }}
tags: [Linux, note]
---

### 基本概念
在Linux里面，无论是进程，还是线程，到了内核里面，我们统一都叫任务（Task），由一个统一的结构`task_struct`进行管理。
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zdGF0aWMwMDEuZ2Vla2Jhbmcub3JnL3Jlc291cmNlL2ltYWdlLzc1LzJkLzc1YzRkMjhhOWQyZGFhNGFjYzExMDc4MzJiZTg0ZTJkLmpwZWc?x-oss-process=image/format,png)


### 分析`task_struct`
对源码检索`stask_struct`关键字，发现文件`include/linux/sched.h`有这个结构体定义,结构非常长。下面借用专栏中总结的框图
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zdGF0aWMwMDEuZ2Vla2Jhbmcub3JnL3Jlc291cmNlL2ltYWdlLzFjL2JjLzFjOTE5NTZiNTI1NzRiNjJhNDQxOGE3YzY5OTNkOGJjLmpwZWc?x-oss-process=image/format,png)

### 补充知识
**1、 系统上查看进程信息**
可以通过`/proc/pid`下的文件查看进程的相关信息。或者直接通过一些[常用命令](https://garlicspace.com/2019/07/03/),比如
- `ps`查看进程
- `pstree`查看进程的依赖关系
- `lsof`命令用于查看你进程开打的文件，打开文件的进程，进程打开的端口(TCP、UDP)。找回/恢复删除的文件。和`fuser`命令用于报告进程使用的文件和网络套接字。

### 参考资料
> - [极客时间专栏 - 进程数据结构（上）：项目多了就需要项目管理系统](https://time.geekbang.org/column/article/91550)
> - [极客时间专栏 - 进程数据结构（中）：项目多了就需要项目管理系统](https://time.geekbang.org/column/article/92151)
