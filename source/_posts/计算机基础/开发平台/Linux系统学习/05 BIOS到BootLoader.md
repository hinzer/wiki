---
title: 05 BIOS到BootLoader
toc: false
date: 2020-04-11 8:22:02
tags: [Linux, note]
---


### 初步了解
**1、 回顾**
之前理解了x86架构下的工作模式，计算机系统的核心是**CPU、内存、总线**来干活的。但是x86提供的是开放的硬件平台，需要配合对应的操作系统，才能发挥最大的作用。
另外随着计算机技术的衍变，32位系统之后的x86架构已经有**实模式**和**保护模式**两种模式。系统启动之前的BIOS阶段在实模式，之后工作在保护模式。
- 实模式，兼容原来16位系统设计出的模式。只能寻址1M，每个段最多64K
- 保护模式，对于32位系统，能够寻址4G。

**2、 目的**
学习目的: 操作系统不是在板子上电就直接运行的，中间一定有一个过程。学习了解linux系统启动之前cpu做了哪些准备，内核如何被加载到内存上运行。

**3、 总结**
板子上电，先读取ROM中的固件代码，做出一个基本输入输出系统。这一阶段为`BIOS时期`。
BIOS从启动盘（一般是硬盘第一个扇区）开始加载引导代码，进一步初始化硬件，实模式升级为保护模式。这一阶段为`BootLoader时期`。
BootLoader将一系列工作做完了，最重要的一步就是加载系统内核kernel到内存运行了。控制权移交给内核之后，BootLoader时期结束，然后开始内核的部分了。

<br>

### BIOS阶段
> BIOS是固化在ROM上的一段程序，如果你自己安装过操作系统，刚启动的时候，按某个组合键，显示器会弹出一个蓝色的界面。能够调整启动顺序的系统，就是我说的 BIOS，然后我们就可以先执行它。

**1、 内存地址空间**
cpu工作模式是实模式，这时只有1M的内存地址空间
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zdGF0aWMwMDEuZ2Vla2Jhbmcub3JnL3Jlc291cmNlL2ltYWdlLzVmL2ZjLzVmMzY0ZWY1YzlkMWEzYjFkOWJiNzE1M2JkMTY2YmZjLmpwZWc?x-oss-process=image/format,png)
1. x86系统中，cpu将内存地址0xF0000 到 0xFFFFF 这 64K 映射给 ROM。
2. 主板上电，cpu将CS寄存器置为0xffff，ip寄存器置0x0000，所以第一条指令指向的地址是0xfff0(实模式下，cs<<4 + ip)。这里有一个jmp指令，跳转到rom中做初始化的代码。

**2、 程序流程**
主板上电，CPU先从ROM中加载BIOS程序，BIOS进行硬件相关的初始化工作。主要有2件事情
1. 检查硬件环境
2. 是建立中断程序和中断向量表，同时把结果显示在显示器上

<br>

### BootLoader阶段
> 光有BIOS还不够，还要从硬盘上搞到操作系统。引导操作系统这一阶段就是BootLoader。

**1、 引导管理器grub2**
Linux一般通过grub来做系统引导程序。
系统上提供了grub2工具，grub2用户配置文件`/etc/default/grub`，系统会根据用户配置自动生成`/boot/grub/grub.cfg`。常用命令
```
# 重新生成配置文件
grub-mkconfig -o /boot/grub/grub.cfg
 
# 将Grub 2安装到硬盘引导扇区
grub-install --root-directory=/ /dev/sda
```
**2、 相关文件**
使用 grub2-install /dev/sda，可以将启动程序安装到相应的位置。其中有`boot.img`、`core.img`
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zdGF0aWMwMDEuZ2Vla2Jhbmcub3JnL3Jlc291cmNlL2ltYWdlLzJiLzZhLzJiODU3M2JiYmYzMWZjMGNiMDQyMGUzMmQwN2IxOTZhLmpwZWc?x-oss-process=image/format,png)
(1)boot.img

在BIOS平台下，boot.img是grub启动的第一个img文件，它被写入到MBR中或分区的boot sector中，因为boot sector的大小是512字节，所以该img文件的大小也是512字节。

boot.img唯一的作用是读取属于core.img的第一个扇区并跳转到它身上，将控制权交给该扇区的img。由于体积大小的限制，boot.img无法理解文件系统的结构，因此grub2-install将会把core.img的位置硬编码到boot.img中，这样就一定能找到core.img的位置。

[此处参考](https://www.cnblogs.com/f-ck-need-u/p/7094693.html#blog122)

(2)core.img

core.img根据diskboot.img、kernel.img和一系列的模块被grub2-mkimage程序动态创建。core.img中嵌入了足够多的功能模块以保证grub能访问/boot/grub，并且可以加载相关的模块实现相关的功能，例如加载启动菜单、加载目标操作系统的信息等，由于grub2大量使用了动态功能模块，使得core.img体积变得足够小。

core.img中包含了多个img文件的内容，包括diskboot.img/kernel.img等。

[此处参考](https://www.cnblogs.com/f-ck-need-u/p/7094693.html#blog122)

**3、 引导流程**
BIOS完成之后，先加载boot.img到内容中运行，boot.img 将控制权交给 diskboot.img 后，diskboot.img 的任务就是将 core.img 的其他部分加载进来，先是解压缩程序 lzma_decompress.img，再往下是 kernel.img，最后是各个模块 module 对应的映像。这里需要注意，它不是 Linux 的内核，而是 grub 的内核。
再kernel.img中选择加载真正的linux kernel(对应之前的grub用户配置文件)。

<br>

### 补充知识
**1、 从实模式切换到保护模式**
在bootloader过程中，lzma_decompress.img 做了一个重要的决定，就是调用 real_to_prot，cpu从实模式切换到保护模式.切换到保护模式要干很多工作，大部分工作都与内存的访问方式有关。
- 第一项是启用分段，就是在内存里面建立段描述符表，将寄存器里面的段寄存器变成段选择子，指向某个段描述符，这样就能实现不同进程的切换了。
- 第二项是启动分页。能够管理的内存变大了
- 打开第21根地址线 Gate A20，cpu从20位总线到32位总线访问内存

**2、 硬件基础(主引导扇区、分区引导扇区、分区表)**
一个硬盘实际上由一个个扇区组成，它起始的一部分扇区为主引导扇区，包括MBR（主引导纪录）和DPT（分区表）
硬盘可以有多个分区，每个分区起始的一部分扇区，为分区引导扇区。分区引导扇区之后的部分，为文件系统的索引，不同的文件系统采用不同的索引，文件系统通过它定位文件在硬盘上的位置。
对硬盘的读写操作，通过文件系统来完成；引导扇区中的内容，我们不能够在文件系统中进行操作，而需要专用软件，比如引导管理器。

<br>

### 参考资料
[主引导记录 - 百科](https://zh.wikipedia.org/wiki/%E4%B8%BB%E5%BC%95%E5%AF%BC%E8%AE%B0%E5%BD%95)
[Grub2配置](http://linux-wiki.cn/wiki/zh-hans/Grub2%E9%85%8D%E7%BD%AE)
[grub2详解(翻译和整理官方手册)](https://www.cnblogs.com/f-ck-need-u/p/7094693.html)
[An introduction to the Linux boot and startup processes](https://opensource.com/article/17/2/linux-boot-and-startup)
[An introduction to GRUB2 configuration for your Linux machine](https://opensource.com/article/17/3/introduction-grub2-configuration-linux)

