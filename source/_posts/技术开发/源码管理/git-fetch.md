---
title: git fetch
toc: false
date: 2020-03-28 08:24:37
tags: [Git]
---

### 理解
从远程取最新的patch或者分支，引用的信息记录到`.git/FETCH_HEAD`这个文件中。可以借助`git fetch --help`查看具体描述。可以操作一个分支`git pull = git getch + git merge`，也可以单独pick一个patch`git fetch + git cherry-pick`。


### 使用准则
无

### 命令速查
```
### 从gerrit上取一个patch，然后pick到当前分支
git fetch ssh://wangjianfeng1@git.mioffice.cn:29418/device/xiaomi/merlin refs/changes/17/909617/1
git cherry-pick FETCH_HEAD

### 获取远程库的分支更新，然后merge到本地分支
git fetch origin master:tmp #从远程仓库master分支获取最新，在本地建立tmp分支
git diff tmp #將當前分支和tmp進行對比
git merge tmp #合并tmp分支到当前分支
```


### 参考
> - [git fetch 和git pull 的差别](https://www.cnblogs.com/qiu-Ann/p/7902855.html)