---
title: android源码管理工具
toc: false
date: 2020-03-28 00:11:37
tags: [Git, Repo, Android]
---


> Google使用git和repo管理AOSP源码。

### 概念区分
1、Git和git(https://source.android.com/setup/develop#git)
Git是版本控制系统，我们使用git工具进行代码仓库和分支的管理。比如我可以使用`git clone`从远程仓库的一个分支下载代码到本地，可以`git push`将本地仓库的某一个分支推送到远程仓库的分支，关于git使用可以参考[progit](https://www.git-scm.com/book/en/v2)
Google将android源码拆分成许多个Git仓库，我们知道每一个git仓库下有`.git`文件
```
hinzer@ubuntu:android-10$ find -name ".git"
./developers/demos/.git
./developers/build/.git
./developers/samples/android/.git
./.repo/repo/.git
./.repo/manifests/.git
./cts/.git
./platform_testing/.git
./prebuilts/go/linux-x86/.git
./prebuilts/go/darwin-x86/.git
./prebuilts/build-tools/.git
./prebuilts/clang/host/linux-x86/.git
./prebuilts/clang/host/darwin-x86/.git
./prebuilts/checkcolor/.git
./prebuilts/android-emulator/.git
./prebuilts/asuite/.git
./prebuilts/gradle-plugin/.git
./prebuilts/manifest-merger/.git
^C
```

2、Repo和repo(https://source.android.com/setup/develop#repo)
然后使用一个Repo仓库对这些拆分开来的Git仓库集中起来进行管理，在源码根目录下有一个`.repo`文件，其中`manifest.xml`是一个清单文件，记录了`远程分支`、`本地分支`、`本地目录`之间对应关系。
```
hinzer@ubuntu:android-10$ tree .repo -L 1
.repo
├── manifests       # 所有清单文件保存
├── manifests.git
├── manifest.xml     # 重要，当前清单文件的指向！！
├── project.list
├── project-objects
├── projects
└── repo

5 directories, 2 files

```
在`.repo`目录之前，还有一个repo工具(通过它来初始化repo仓库)，这是一个python写的脚本，可以直接阅读源码(也就是可执行文件的位置)查看代码逻辑。
```
hinzer@ubuntu:android-10$ whereis repo
repo: /home/hinzer/bin/repo
hinzer@ubuntu:android-10$ cat /home/hinzer/bin/repo
```

*PS: google将android源码拆分成许多个Git仓库，又通过Repo将这些拆分还原回一个android源码。像不像计算机原理中的`化整为零，还零为整`的思想？*


### 平时使用
1、Repo和Git
Repo 并非用来取代 Git，只是为了让您在 Android 环境中更轻松地使用 Git。一般我们使用`repo`命令建立Repo仓库，同步android源码；使用`git`命令对我们修改的其中一个模块提交。

2、Gerrit仓库
一个网页系统，用于代码审核，也方便查看别人提交的patch。

3、OpenGrok
网页系统，在线阅读源码的利器。

4、Android Studio
用于开发 Android 应用的官方集成开发环境 (IDE)。


### 相关资料
> - [Git 文档](https://git-scm.com/doc)
> - [Repo 命令参考文档](https://source.android.com/setup/develop/repo)