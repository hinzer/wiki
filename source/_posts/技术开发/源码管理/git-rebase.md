---
title: git rebase
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---

### 理解
rebase也是整合不同分支的方法，和merge不同的是它会改变提交历史。`git help rebase`命令查看具体描述。
假设当前HEAD指向`topic`分支，下面执行rebase命令(将topic上的patch打到master上，并改变历史)，使用`git rebase master`
```
rebase前:

                     A---B---C topic
                    /
               D---E---F---G master
rebase后:
                             A'--B'--C' topic
                            /
               D---E---F---G master
```
原理:首先找到这两个分支（即当前分支 topic、变基操作的目标基底分支 master 的最近共同祖先 E，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件， 然后将当前分支指向目标基底 G, 最后以此将之前另存为临时文件的修改依序应用

### 使用准则
- rebase操作可以让提交历史更加简介，但注意不要影响远程分支的提交历史记录。不要在公共分支上使用rebase
- 如果你想把rebase之后的master分支推送到远程仓库，Git会阻止你这么做，因为两个分支包含冲突。但你可以传入`--force`标记来强行推送。
- 


### 命令速查
```
git rebase master topic 	#master是基底分支，将topic分支上的修改在master上重放


git rebase --onto master server client  #选中在 client 分支里但不在 server 分支里的修改，将它们在 master 分支上重放


```


### 参考
> - [ Git 分支 - 变基](https://www.git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)
> - [Merge、Rebase 的选择](https://github.com/geeeeeeeeek/git-recipes/wiki/5.1-%E4%BB%A3%E7%A0%81%E5%90%88%E5%B9%B6%EF%BC%9AMerge%E3%80%81Rebase-%E7%9A%84%E9%80%89%E6%8B%A9)



