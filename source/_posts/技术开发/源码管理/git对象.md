---
title: git 对象
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---

## 窥探
git对文件内容管理核心是基于`键值对数据库`，位于`.git/objects`。通过`key-value`方式管理内容。
对象数据库中存储的三种对象，每一个对象在数据库中以`key-value`形式存在
- 数据对象（blob object），记录文件内容,解决内容存储问题
- 树对象（tree object），它能解决文件名保存的问题，也允许我们将多个文件组织到一起
- 提交对象（commit object），对应一次提交，解决项目快照的问题

![对象关系图](https://www.git-scm.com/book/en/v2/images/data-model-3.png)



## 命令速查
1、**通用操作**
```
find .git/objects -type f  	# 列出所有git对象

git cat-file -p <key> 		# 查看对象内容信息
git cat-file -t <key> 		# 查看对象类型

git cat-file -p master^{tree} #查看master分支上最近提交的树对象
```
*PS: 尝试`git add`、`git commit`等上层命令，然后`find .git/objects -type f`查看git对象的变化。
发现：git每次add文件，添加数据对象；commit之后，又创建了树对象、和提交对象。
于是得出结论：git会记录每一次变化的文件版本(生成blob对象)，对于没有修改的文件tree还是原来的指向。*


2、**创建数据对象**
```
# 内容写入数据库，返回key值
$ echo 'test content' | git hash-object -w --stdin	# -w表示写入数据库, --stdin表示从标准输入流读取


$ echo 'version 1' > test.txt
$ git hash-object -w test.txt 	#从文件读取内容写入数据库


```

3、**创建树对象**
```
# 为文件添加暂存区(index)
$ git update-index --add --cacheinfo 100644 \
  83baae61804e65cc73a7201a7252750c76066a30 test.txt	# --add表示这个文件此前没有添加到index, --cacheinfo表示添加的文件在数据库中 匹配此的参数格式为(文件模式 key 文件名)
$ git write-tree #通过暂存区创建树对象

# 方式二
$ echo 'new file' > new.txt
$ git update-index --add --cacheinfo 100644 \
  1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
$ git update-index --add new.txt
文件名)
$ git write-tree #通过暂存区创建树对象
```


4、**创建提交对象**
```
$ echo 'first commit' | git commit-tree d8329f		#(没有父提交)第一步提交

$ echo 'second commit' | git commit-tree 0155eb -p fdf4fc3  #-p参数制定父提交对象的key

```

5、**查看提交历史**
```
$ git log --stat 0155eb
```


## 参考
> - [Git 内部原理 - Git 对象](https://www.git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)
> - [Git 工具 - 重置揭密](https://www.git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)