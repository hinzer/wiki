---
title: 07 编译android kernel
toc: false
date: 2020-04-05 8:35:10
tags: [record, Android]
---



### Android linux内核
Android并没有使用标准的Linux内核，而是做了很多的修改。AOSP网站提供了适应各种芯片或设备的linux内核源码的仓，只有少部分google设备支持的内核源码可以通过[repo构建](https://source.android.google.cn/setup/build/building-kernels?hl=zh-cn#downloading),其他的需要做一些编译配置。

### 手动编译内核
这里选择emulator模拟器跑Android内核，所以选择goldfish版本作为我的的Linux Kernel，下面通过编译goldfish内核来介绍这个过程。
按照[官方的教程](https://source.android.com/setup/build/building-kernels-deprecated)没有找到`xxx_defconfig`编译配置文件，之后在网上找到一篇相似的[博客](https://my.oschina.net/wuqingyi/blog/903421)，以下步骤基本上是按照那个教程来的。
1、下载源码
``` bash
hinzer@ubuntu:kernel$ git clone https://android.googlesource.com/kernel/goldfish
```
2、查看当前Android系统对应的内核版本
``` bash
hinzer@ubuntu:android-10$ emulator   # 虚拟机运行起来，点Setting -> Android version 查看内核版本信息
```
3、检出对应分支
``` bash
hinzer@ubuntu:kernel$ cd goldfish/
git branch -a
#....
#....
git checkout -b dev remotes/origin/android-goldfish-4.14-dev.20190417  # 对应的内核版本为4.14
#....
```
4、进行编译配置
``` bash
export PATH=$PATH:/home/hinzer/source/android-10/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin
export ARCH=x86_64
export CROSS_COMPILE=x86_64-linux-android-
export REAL_CROSS_COMPILE=x86_64-linux-android-
```
5、编译
``` bash
/home/hinzer/source/android-10/prebuilts/qemu-kernel/build-kernel.sh  --arch=x86_64    # 需要在kernel的源码的根目录下执行
```
但是出现error，没有发现"x86_64_emu_defconfig"这个文件。于是我将`x86_64_defconfig`改为`x86_64_emu_defconfig`
``` bash
cp -a /home/hinzer/source/android-10/kernel/goldfish/arch/x86/configs/x86_64_defconfig /home/hinzer/source/android-10/kernel/goldfish/arch/x86/configs/x86_64_emu_defconfig
/home/hinzer/source/android-10/prebuilts/qemu-kernel/build-kernel.sh  --arch=x86_64    # 继续编译
...
...
Kernel: arch/x86/boot/bzImage is ready  (#1)
Kernel x86_64_emu prebuilt images (kernel-qemu and vmlinux-qemu) copied to /tmp/kernel-qemu/x86_64-4.14.88 successfully !

```
6、加载内核并运行系统
emulator启动相关的参数，可参考[官方手册 - 命令行启动选项](https://developer.android.google.cn/studio/run/emulator-commandline#startup-options)
``` bash
# 编译生成的内核放在/tmp/kernel-qemu/x86_64-3.10.0/kernel-qemu
hinzer@ubuntu:android-10$ emulator -kernel /tmp/kernel-qemu/x86_64-4.14.88/kernel-qemu  # 配置参数
```
7、再次查看当前Android系统对应的内核版本


### 补充
将编译出的`kernel-qemu`加载到emulator上发现界面直接卡死，adb devices命令也连接不上。初步怀疑是编译的linux内核版本不对，导致无法正常启动。(这个问题暂时还没有解决，目前单独编译不影响)
另外，进一步研究了kernel源码下的`README`文件，发现`make ${PLATFORM}_defconfig`说明。想到之前按照官方的步骤可能出了一点差错，修改为
```
cd goldfish
export ARCH=x86     # cpu架构
export CROSS_COMPILE=x86_64-linux-android-
make x86_64_defconfig    # 将arch/$ARCH/configs/xxx_defconfig写入.config文件，编译阶段build系统会检索
make
```
success!!此时编译出的映像文件输出`arch/x86/boot/bzImage`,


### 参考资料
> - [手动编译内核](https://source.android.com/setup/build/building-kernels-deprecated)
> - [编译x86_64 android 7.1及goldfish内核](https://my.oschina.net/wuqingyi/blog/903421)
> - [Android内核开发：源码的版本与分支详解](https://blog.51cto.com/ticktick/1654759)
> - [Android 通用内核](https://source.android.google.cn/devices/architecture/kernel/android-common?hl=zh-cn)

