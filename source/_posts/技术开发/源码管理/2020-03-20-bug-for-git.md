---
title: git merge失败
toc: false
date: 2020-03-20 00:11:37
tags: [git, bug]
---

## 问题描述
git merge本地分支出现报错 fatal: refusing to merge unrelated histories
![git-merge.jpg](http://ww1.sinaimg.cn/large/0063ewMaly1gczozf2n8ij30vn0fognf.jpg)

## 第一反应
我现在都是在本地操作还没有远程，我得理解master分支merge到slave分支，应该直接fast forward过去才对。
如果把master分支干掉，直接在slave分支那个位置创建一个master分支应该也没什么影响。就是特别像知道为啥会出错这个merge。

## 问题分析
merge命令之后报错`fatal: refusing to merge unrelated histories。`，表示当前分支和slave分支不相关。有[对应的解决方案](https://yq.aliyun.com/articles/614459)
加上`--allow-unrelated-histories`参数，忽略这个问题。
然而又报错，根据进一步提示，`git status`发现当前工作目录确实存在冲突。解决完冲突commit之后，再次merge就可以了。

不记得当时具体做了什么操作了，又重新做了几遍还是没能把当时的情景复现出来(merge直接fast forward了)。后来分析应该是某种原因导致了文件冲突，进而影响之后的merge操作。


## 解决方法
如果是`git pull`或者`git push`报`fatal: refusing to merge unrelated histories`,直接在merge后加上`--allow-unrelated-histories`参数就ok了
如果是依然无效，不妨先`git status`查看一下当前版本库的状态有无问题(我这边是冲突引起的)。



## 参考资料
> - [解决Git中fatal: refusing to merge unrelated histories](https://yq.aliyun.com/articles/614459)
