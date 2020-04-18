---
title: 打patch
toc: false
date: 2020-03-28 08:24:37
tags: [Git]
---


### 理解
patch是某一次提交给文件内容的改变，打patch是将某一次改变的内容应用到当前的版本库。

### 常规操作
```
# 生成patch
git diff ./ > xxx.patch  #将差异的内容制作成patch
mkdir update && git diff commit-id-time1 commit-id-time2 --name-only | xargs -i cp '{}' ./update/ --parents #制作patch 把两个commit-id 之间修改的文件复制到update目录中 而且会把中间的目录也一并生成


# 打patch
patch -p1 < xxx.patch
```


### 另外
不过我们有线上的gerrit仓库，日常使用`git fetch` + `git cherry-pick`效果是一样的，cherry-pick直接pick某一个patch.


### 参考
> - [你知道用git打补丁吗？](https://mp.weixin.qq.com/s/tf1Wyudp7l9XWM0ILAhZpQ)
> - [Git Commands - Patching](https://www.git-scm.com/book/en/v2/Appendix-C%3A-Git-Commands-Patching)