---
title: git reset、git checkout
toc: false
date: 2020-03-19 00:11:37
tags: [Git]
---

### 理解
1、[git三棵树](https://www.git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86),即
- HEAD，永远指向当前分支的最新一笔提交
- Index，索引(暂存区)
- Work Directory,工作目录

```
#git 底层命令
git cat-file -p HEAD 			#显示HEAD内容
git ls-tree -r HEAD 			#显示树对象的内容
git ls-files -s 				#显示索引(Index)的所有文件信息
tree 							#查看当前工作目录
```

2、git-reset
将当前HEAD重置为指定状态。`git help reset`命令查看具体描述。


### 使用准则
- 运行`git --hard reset`之前请考虑一下。如果由于执行这个选项导致工作目录中文件(文件已经提交)被覆盖，可以尝试`git reflog`找回。


### 命令速查

1、作用于某个patch
```
git reset --soft [patch]	#移动HEAD的指向，不改变Index和Work Directory
git reset --mixed [patch] #(默认reset)移动HEAD的指向，改变Index，但不改变Work Directory
git reset --hard [patch]	#移动HEAD的指向，改变Index和Work Directory
```

2、作用于某个path/file
```
git reset [path/file]	#通过当前HEAD指向的patch改变当前Index(恢复暂存区)
git reset [patch] [path/file] #通过指定patch改变当前Index
```

3、压缩提交
```
git reset --soft [patch] #HEAD移动到压缩提交的前一个patch，Index和Work Directory不变
git commit #通过Index创建新的patch
```


###git reset 和 git checkout
总结了两点重要的区别，
- 操作patch时，chekcout只移动HEAD指针本身(不改变HEAD分支)
- 操作path/file时，checkout会改变工作目录(类似git reset --hard)




### 参考
> - [Git 工具 - 重置揭密](https://www.git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)
> - [多种表示patch的方式](https://github.com/zlargon/git-tutorial/blob/master/branch/commit_tree.md)
