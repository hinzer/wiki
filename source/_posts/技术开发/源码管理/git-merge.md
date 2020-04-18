---
title: git log
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---


### 理解
工具用来合并一个或者多个分支到你已经检出的分支中。 然后它将当前分支指针移动到合并结果上。`git help merge`命令查看具体描述。

将`topic`分支merge到`master`分支上(更新master分支)，使用`git merge topic`
```
合并前:
                     A---B---C topic
                    /
               D---E---F---G master
合并后:
                     A---B---C topic
                    /         \
               D---E---F---G---H master
```

### 使用准则
- merge或者pull之前本地仓库是干净的应当commit本地，保证merge之后不会被破坏。或者`git stash`储藏本地的修改
- merge冲突后可以使用`git status`查看分支状态，`git diff`显示工作区文件变化


### 命令速查
```
$ git merge slave   # merge slave分支到当前HEAD分支， --no-ff 
$ git merge slave --no-ff  # 不使用fast-forward模式，merge同时创建一个新的cmomit patch
$ git merge slave --allow-unrelated-histories   #
$ git merge -Xignore-space-change whitespace # 忽略空白merge

$ git merge --abort 	# (merge失败)恢复到合并前的状态
$ git merge --continue  # 冲突后执行

```


### 参考
> - [git help merge](https://git-scm.com/docs/git-merge/2.12.0)
> - [高级合并](https://www.git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%AB%98%E7%BA%A7%E5%90%88%E5%B9%B6)



