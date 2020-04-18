---
title: 06 kernel初始化
toc: false
date: 2020-04-11 8:23:02
tags: [Linux, note]
---




### 初步了解
**1、 回顾**
经过了BootLoader阶段，此时cpu从实模式转换成保护模式，有了更强的寻址能力，kernel也已经加载到内存了。系统内核开始运行
在kernel源码`init/main.c`文件中，内核的启动从入口函数`start_kernel() `。其中进行一系列的初始化`XXXX_init`

**2、目的**
结合源码，了解内核启动阶段开始都做了哪些初始化。

**3、总结**
一些关键的初始化函数，原文中已经有总结好的图片
![](https://static001.geekbang.org/resource/image/cd/01/cdfc33db2fe1e07b6acf8faa3959cb01.jpeg)

还有细心的同学总结了比较详细的笔记
``` 
- 内核初始化, 运行 `start_kernel()` 函数(位于 init/main.c), 初始化做三件事
    - 创建样板进程, 及各个模块初始化
    - 创建管理/创建用户态进程的进程
    - 创建管理/创建内核态进程的进程
---
- 创建样板进程,及各个模块初始化
    - 创建第一个进程, 0号进程. `set_task_stack_end_magic(&init_task)` and `struct task_struct init_task = INIT_TASK(init_task)`
    - 初始化中断, `trap_init()`. 系统调用也是通过发送中断进行, 由 `set_system_intr_gate()` 完成.
    - 初始化内存管理模块, `mm_init()`
    - 初始化进程调度模块, `sched_init()`
    - 初始化基于内存的文件系统 rootfs, `vfs_caches_init()`
        - VFS(虚拟文件系统)将各种文件系统抽象成统一接口
    - 调用 `rest_init()` 完成其他初始化工作
---
- 创建管理/创建用户态进程的进程, 1号进程
    - `rest_init()` 通过 `kernel_thread(kernel_init,...)` 创建 1号进程(工作在用户态).
    - 权限管理
        - x86 提供 4个 Ring 分层权限
        - 操作系统利用: Ring0-内核态(访问核心资源); Ring3-用户态(普通程序)
    - 用户态调用系统调用: 用户态-系统调用-保存寄存器-内核态执行系统调用-恢复寄存器-返回用户态
    - 新进程执行 kernel_init 函数, 先运行 ramdisk 的 /init 程序(位于内存中)
        - 首先加载 ELF 文件
        - 设置用于保存用户态寄存器的结构体
        - 返回进入用户态
        - /init 加载存储设备的驱动
     - kernel_init 函数启动存储设备文件系统上的 init
---
- 创建管理/创建内核态进程的进程, 2号进程
    - `rest_init()` 通过 `kernel_thread(kthreadd,...)` 创建 2号进程(工作在内核态).
    - `kthreadd` 负责所有内核态线程的调度和管理
```

下面我会根据文章和这份笔记，结合代码捋一下流程。

>在线阅读linux内核源码: [https://elixir.bootlin.com/linux/latest/source](https://elixir.bootlin.com/linux/latest/source)

<br>

### 各个模块的初始化
**1、 0号进程**
定位到入口函数[start_kernel(void)](https://elixir.bootlin.com/linux/latest/source/init/main.c#L785)，发现有一处调用[set_task_stack_end_magic(&init_task);](https://elixir.bootlin.com/linux/latest/source/init/main.c#L790)。这里kernel刚启动创建的第一个进程，pid为0，唯一一个没有通过 fork 或者 kernel_thread 产生的进程。
```c
set_task_stack_end_magic(&init_task);
```

**2、 各个模块初始化**
同样在[start_kernel(void)](https://elixir.bootlin.com/linux/latest/source/init/main.c#L785)可以定位到其他的初始化调用，其中有关键的几个
``` c
trap_init(); //系统调用相关 设置中断门

mm_init();	//内存管理

sched_init();	//调度模块

vfs_caches_init();	//rootfs文件系统

rest_init(); 		//其他方面的init
```

<br>

### 用户态祖先进程的创建
在rest_init()函数中有一处[kernel_thread(kernel_init, NULL, CLONE_FS)](https://elixir.bootlin.com/linux/latest/source/init/main.c#L626),创建了pid为1的进程，这是第一个用户进程，是所有其他用户进程的鼻祖进程。

**内核态到用户态**
这是一个用户进程，需要在用户态运行。一般用户程序是从`用户态--到内核态--返回用户态`的过程。当前执行 kernel_thread 这个函数的时候，就在内核态。如何直接从内核态到用户态呢？
在kernel_init()函数中有一处`kernel_init_freeable()`调用，查看[定义处](https://elixir.bootlin.com/linux/latest/source/init/main.c#L1454)有
``` c
if (!ramdisk_execute_command)
    ramdisk_execute_command = "/init";
```
回到kernel_init()函数，有对应的执行代码
``` c
  if (ramdisk_execute_command) {
    ret = run_init_process(ramdisk_execute_command);
......
  }
......
  if (!try_to_run_init_process("/sbin/init") ||
      !try_to_run_init_process("/etc/init") ||
      !try_to_run_init_process("/bin/init") ||
      !try_to_run_init_process("/bin/sh"))
    return 0;

```
如果我们打开 run_init_process 函数，会发现它调用的是 do_execve。
do_execve是一个内核系统调用，它的作用是运行一个执行文件。其中调用链为`do_execve->do_execveat_common->exec_binprm->search_binary_handler`
```  c
int search_binary_handler(struct linux_binprm *bprm)
{
  ......
  struct linux_binfmt *fmt;
  ......
  retval = fmt->load_binary(bprm);
  ......
}

```
在这里，它会尝试运行ramdisk 的`/init`，或者普通文件系统上的`/sbin/init`、`/etc/init`、`/bin/init`、`/bin/sh`。加载ELF文件，只要有一个启动起来就可以了。

比较关心程序如何'恢复'到用户态的，其实最后内核空间中保存了用户态运行的上下文，最后只要切换上下文，恢复那些寄存器（CS、DS、IP、SP），然后下一条指令，就从用户态开始运行了。

<br>

### 内核态祖先进程的创建
继续在rest_init()中查看有一处调用[kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES)](https://elixir.bootlin.com/linux/latest/source/init/main.c#L638)，这里创建了第三个进程，pid为2，是内核态所有task的祖先，作用是将内核态所有的task进行统一的调度和管理。

<br>

### 补充知识
**1、 用户态、内核态**
\# 基本概念
进入保护模式后，为避免多进程对资源访问的混乱。使用x86提供的权限访问机制，有Ring0...Ring3 4种权限。
用户进程一般放在Ring3，我们称为`用户态`
核心驱动代码一般放在Ring0，称为`内核态`

\# 系统调用
用户程序要访问核心资源，需要通过系统提供的系统调用接口，从用户态进入内核态。这个过程就是这样的：用户态 - 系统调用 - 保存寄存器 - 内核态执行系统调用 - 恢复寄存器 - 返回用户态，然后接着运行。
![](https://static001.geekbang.org/resource/image/d2/14/d2fce8af88dd278670395ce1ca6d4d14.jpg)


**2、 ramdisk的作用**
kernel启动过程中，一开始到用户态的是ramdisk的init，后来会启动真正根文件系统上的init，成为所有用户态进程的祖先。
为什么没有直接从根文件系统上加载init，这是因为文件系统一定存在一个存储设备上。要对设备访问需要驱动程序，而对内存可以直接访问。所以想在内存上建立一个`假文件系统`，先运行要访问存储设备的驱动程序，有了驱动就能设置根文件系统，就能启动根文件系统上的init程序了。

<br>

### 参考资料
> - [极客时间- 内核初始化](https://time.geekbang.org/column/article/90109)
