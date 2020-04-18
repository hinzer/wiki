---
title: push代码
toc: false
date: 2020-03-28 08:24:37
tags: [Git]
---


### 用法
```
wjf@ubuntu:base$ pwd
/home/wjf/miui/umi-q/frameworks/base
wjf@ubuntu:base$ repo info .
Manifest branch: miui-q-umi-stable
Manifest merge branch: refs/heads/stable
Manifest groups: all,-notdefault
----------------------------
Project: platform/frameworks/base
Mount path: /home/wjf/miui/umi-q/frameworks/base
Current revision: miui-q-umi-stable
Local Branches: 0
----------------------------
wjf@ubuntu:base$ git push ssh://wangjianfeng1@gerrit.pt.miui.com:29418/platform/frameworks/base HEAD:refs/for/miui-q-umi-stable
```

- `git push` git语法表示远程推送,`git push help`查看详细情况
- `ssh://wangjianfeng1@gerrit.pt.miui.com:29418/platform/frameworks/base`表示使用ssh协议访问gerrit服务器的29418端口，通过url定位到frameworks/base这个目录，是要推送的目录
- `HEAD:refs/for/miui-q-umi-stable`,HEAD指向当前的本地分支，refs/for/miui-q-umi-stable表示远程分支名。

*ps: 在修改目录下，git remote -v命令查看代码服务器的git仓库的链接，repo info .获取gerrit仓库远程提交点（分支名）*

### 参考
> - [SSH协议语法格式](https://www.cnblogs.com/sparkdev/p/6071533.html)