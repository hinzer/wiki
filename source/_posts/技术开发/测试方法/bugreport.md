---
title: bugreport
toc: false
date: 2020-03-28 08:28:37
tags: [Android, Log]
---


### 理解
原生android系统支持`adb bugreport`命令，生成日志包含设备日志、堆栈轨迹和其他诊断信息，可以帮助您查找和修复应用中的错误

1、目录结构
```
mi@ubuntu:bugreport$ tree -L 1
.
├── bugreport-dipper_ru-QKQ1.190828.002-2020-03-26-16-15-04.txt  #最重要的文件
├── dumpstate_board.txt
├── dumpstate_log.txt
├── FS
├── lshal-debug
├── main_entry.txt
├── outfile.log
├── proto
└── version.txt

3 directories, 6 files
```

### 常用操作
1、[ANR和死锁](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#anrs-deadlocks)
```text
# 找出无响应的应用(系统会终止该进程并将堆栈转储到 /data/anr)
grep "am_anr" bugreport-2015-10-01-18-13-48.txt  # 为二进制事件日志中的 am_anr 执行 grep 命令
grep "ANR in" bugreport-2015-10-01-18-13-48.txt  # 为 logcat 日志（其中包含关于发生 ANR 时是什么在占用 CPU 的更多信息）中的 ANR in 执行 grep 命令

# 查找堆栈跟踪( ANR 对应的堆栈跟踪 --> 进程主线程)
------ VM TRACES AT LAST ANR
------ TRACES JUST NOW 和 

# 查找死锁(系统服务器发生死锁，监控程序最终会将其终止)
WATCHDOG KILLING SYSTEM PROCESS
```

2、[Activity](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#activities)
```text
# 查看聚焦状态的activity(崩溃期间处于聚焦状态的 Activity 表示当前用户操作)
grep "am_focused_activity" bugreport-2015-10-01-18-13-48.txt

# 查看进程启动事件
grep "Start proc" bugreport-2015-10-01-18-13-48.txt

# 设备是否发生系统颠簸
grep -e "am_proc_died" -e "am_proc_start" bugreport-2015-10-01-18-13-48.txt
```

3、[内存](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#memory)
4、[广播](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#broadcasts)
5、[显示器争用](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#monitor%20contention)
6、[后台编译](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#background-compilation)
7、[叙述](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#narrative)
8、[电源](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#power)
9、[程序包](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#packages)
10、[进程](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#processes)
11、[扫描](https://source.android.com/source/read-bug-reports.html?hl=zh-cn#scans)

### 参考链接
> - [获取并阅读错误报告](https://developer.android.com/studio/debug/bug-report)
> - [阅读错误报告](https://source.android.com/source/read-bug-reports.html?hl=zh-cn)
> - [Android Log机制、Logcat及MIUI 284日志介绍](https://wiki.n.miui.com/pages/viewpage.action?pageId=181967386)