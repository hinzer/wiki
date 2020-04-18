---
title: git log
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---

### 理解
显示项目提交历史，通过参数选项可以控制log显示的方式。`git help log`查看具体描述。

git log 有两个高级用法：一是自定义提交的输出格式，二是过滤输出哪些提交。这两个用法合二为一，你就可以找到你项目中你需要的任何信息。

### 使用准则
- git log允许你查看你项目历史中任何需要的内容。

### 命令速查
1、常规使用,查看[git log常用选项](https://www.git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2#log_options)
```
git log --oneline --graph --all #简略显示各种分支的patch记录，个人比较下常用这个命令
git log --stat    # 显示每次提交的文件修改统计信息。
git log -p 	# 按补丁格式显示每个提交引入的差异。
```

2、定制化输出,查看[--pretty=format常用格式](https://www.git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2#pretty_format)
```
$ git log --pretty=format:"%h %s" --graph
$ git log --date=format:'%Y-%m-%d %H:%M:%S' --pretty=format:"%h-%an-%ad-%ae" --graph --all
```
- --date=format定制作者修订日期格式
- --pretty=format定制log记录显示
- --graph图形显示分支与合并历史
- --all显示所有分支

3、过滤出自己想要看到的log,查看[限制输出长度](https://www.git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2#limit_options)
```
$ git log -3  			#按次数，最近3次提交log
$ git log --since=2.weeks 	#按时间，最近两周提交log
$ git log --until=2020-03-20 	#按时间，2020-03-20前的提交log
$ git log --grep="update"	#仅显示提交说明中包含"update"的提交

```

4、如果要在 Git 源码库中查看 Junio Hamano 在 2008 年 10 月其间， 除了合并提交之外的哪一个提交修改了测试文件，可以使用下面的命令：
```
$ git log --pretty="%h - %s" --author='Junio C Hamano' --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/
5610e3b - Fix testcase failure when extended attributes are in use
acd3b9e - Enhance hold_lock_file_for_{update,append}() API
f563754 - demonstrate breakage of detached checkout with symbolic link HEAD
d1a43f2 - reset --hard/read-tree --reset -u: remove unmerged new paths
51a94af - Fix "checkout --track -b newbranch" on detached HEAD
b0ad11e - pull: allow "git pull origin $something:$current_branch" into an unborn branch
```


### 参考
> - [Git 基础 - 查看提交历史](https://www.git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)
> - [Git日志格式、颜色设置](https://jasonhzy.github.io/2016/05/05/git-log/)
> - [Linux下date命令，格式化输出，时间设置](https://blog.csdn.net/jk110333/article/details/8590746)



