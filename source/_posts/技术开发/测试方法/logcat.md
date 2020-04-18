---
title: bugreport
toc: false
date: 2020-03-28 08:28:37
tags: [Android, Log]
---


### 理解
logcat是抓log的工具，从android logging系统抓取日志。

1、logcat相关目录
```text
代码位置：`/system/core/logcat/`
编译生成的可执行文件位于：`out/target/product/umi/system/bin/logcat`(umi是产品名)
对应到设备端的可执行文件：`/system/bin/logcat`
```

2、日志缓冲区
```text
radio：查看包含无线装置/电话相关消息的缓冲区。
events：查看已经过解译的二进制系统事件缓冲区消息。
main：查看主日志缓冲区（默认），不包含系统和崩溃日志消息。
system：查看系统日志缓冲区（默认）。
crash：查看崩溃日志缓冲区（默认）。
all：查看所有缓冲区。
default：报告 main、system 和 crash 缓冲区。 
```

### 使用规范
1、[过滤日志输出](https://developer.android.com/studio/command-line/logcat?hl=zh-cn#filteringOutput)
```bash
# tag:priority  标记:优先级
$ adb shell logcat ActivityManager:I MyApp:D *:S
```

2、[控制日志输出格式](https://developer.android.com/studio/command-line/logcat?hl=zh-cn#outputFormat)
```bash
# -v <format>
$ adb shell logcat -v thread
```

3、[查看备用日志缓冲区](https://developer.android.com/studio/command-line/logcat?hl=zh-cn#alternativeBuffers)
```bash
# -b <buffer>
$ adb shell logcat -b radio
```

### 命令速查
```bash
# 获取help
$ adb shell logcat --help

# 查log
$ adb shell logcat -b system > logSystem.txt  #查询此时system的日志，并且保存在logSystem.txt的文件中
^C
```

### 参考链接
> - [Logcat 命令行工具](https://developer.android.com/studio/command-line/logcat?hl=zh-cn)
> - [Android Log机制、Logcat及MIUI 284日志介绍](https://wiki.n.miui.com/pages/viewpage.action?pageId=181967386)
