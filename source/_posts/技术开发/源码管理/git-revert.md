---
title: git revert
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---

### 理解
revert还原提交，撤销已经存在的commit的所有更改，原来的commit将保留，并用新commit来记录还原后的结果。`git help revert`命令查看具体描述。


### 使用准则
无


### 命令速查
```
git revert HEAD 		# 撤销当前HEAD指向的patch上的更改

git revert commit		# 撤销制定commitid表示的patch上的更改

# merge之后的revert
git revert -m 1 HEAD 	# HEAD指向的节点有两个父节点,-m 1保留父节点1，撤销父节点2带来的改变


```


### 参考
> - [工具 - 高级合并](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%AB%98%E7%BA%A7%E5%90%88%E5%B9%B6)
> - [git revert 用法](https://www.cnblogs.com/0616--ataozhijia/p/3709917.html)
> - [回滚错误的修改](https://github.com/geeeeeeeeek/git-recipes/wiki/2.6-%E5%9B%9E%E6%BB%9A%E9%94%99%E8%AF%AF%E7%9A%84%E4%BF%AE%E6%94%B9)