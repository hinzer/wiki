---
title: git 调试
toc: false
date: 2020-03-20 09:20:14
tags: [Git]
---

### 概念理解
`git blame`和`git bisect`能帮助调试git项目，找到出bug的原因。

### 操作方法
1、文件标注
使用`git blame`能显示任何文件中每行最后一次修改的提交记录。`git blame --help`查看具体描述
```
mi@ubuntu:base$ git blame Android.bp -L 230,231  #查看Android.bp的230-231行提交记录，
#commit id   #提交者        #时间                      #行          #内容
7c469179ce2a (junyulai      2019-01-16 20:23:34 +0800 230)         "core/java/android/net/ISocketKeepaliveCallback.aidl",
e40eab608af2 (Benedict Wong 2018-11-14 17:50:13 -0800 231)         "core/java/android/net/ITestNetworkManager.aidl",
```


2、二分查找
`git bisect`能在commit区间中检出中间的patch，通过不断地二分查找，最终定位到带bug的patch
```
# 开始
$ git bisect start  # 启动二分
$ git bisect bad 	# 当前提交有bug
$ git bisect good <good_commit>  #指定已知的最后一次正常状态是哪次提交

# 测试 --> 二分判断
$ git bisect good  # 当前提交无bug
$ git bisect bad   # 当前提交有bug

# 结束
$ git bisect reset
```

### 参考
[Git 工具 - 使用 Git 调试](https://www.git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E4%BD%BF%E7%94%A8-Git-%E8%B0%83%E8%AF%95)