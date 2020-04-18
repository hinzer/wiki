---
title: git stash、git clean
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---

### 理解
贮藏（stash）会处理工作目录的脏的状态——即跟踪文件的修改与暂存的改动——然后将未完成的修改保存到一个栈上， 而你可以在任何时候重新应用这些改动（甚至在不同的分支上）。或在清理(clean)文件。


### 使用准则
无


### 命令速查
```
$ git stash push 			# stash跟踪文件的修改与暂存的改动
$ git stash push --keep-index 		# --keep-index 选项使存储的同时保留索引。
$ git stash push --all				# -u 选项存储untracked文件，
$ git stash push -u 				# stash全部文件(包括被忽略文件)

$ git stash list 			# 列出当前的stash
$ git stash apply   		# 应用stash, 加上--index 选项可以让之前暂存的文件重新暂存
$ git stash drop stash@{0}	# 移除stash,
$ git stash pop 			# 应用stash@{0},并移除它

$ git stash branch dev  	# 创建新分支dev，然后应用stash,然后drop stash


$ git clean -f -d 			# 移除工作目录中所有未追踪的文件以及空的子目录(不包括被忽略文件)
$ git clean -n 				# 演戏以下，加-n参数
$ git clean -n -x 			# -x选择清理忽略文件
$ git clean -x -i 			# -i交互模式
```


### 参考
> - [Git 工具 - 贮藏与清理](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E8%B4%AE%E8%97%8F%E4%B8%8E%E6%B8%85%E7%90%86)