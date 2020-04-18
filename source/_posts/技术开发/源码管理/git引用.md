---
title: git 引用
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---

### 理解
git用文件保存patch的哈希值，这个文件名代表一个分支。指针、分支、引用理解是表示一个概念。


![git本地仓库](https://git-scm.com/book/en/v2/images/data-model-4.png)

1、在`.git`目录下查看
```
HEAD 			#指向当前分支
refs/heads/   	#分支，记录本地commit对象
refs/tags/  	#tag也记录commit对象，但是通常不会改变
refs/remotes/origin/ 	#服务器映射下来的远程只读分支
```

2、查看HEAD内容
```bash
mi@ubuntu:test3$ cd .git
mi@ubuntu:.git$ cat HEAD
ref: refs/heads/master
mi@ubuntu:.git$ cat refs/heads/master
ca82a6dff817ec66f44342007202690a93763949

mi@ubuntu:.git$ git show HEAD
commit ca82a6dff817ec66f44342007202690a93763949 (HEAD -> master, origin/master)

```


### 参考
> - [Git 内部原理 - Git 引用](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%BC%95%E7%94%A8)