---
title: ctags
toc: false
date: 2020-03-16 07:46:33
tags: [ctags]
---


> ctags是方便阅读源代码的工具。开发者在linux平台下和vim编辑器配合使用,这种策略经常被用于linux源码阅读。

## 开发环境
```bash
hinzer@ubuntu:~$ uname -a
Linux ubuntu 5.3.0-40-generic #32~18.04.1-Ubuntu SMP Mon Feb 3 14:05:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

关于2.6.11版本的kernel源码下载
```bash
hinzer@ubuntu:source$ wget -O kernel https://mirrors.edge.kernel.org/pub/linux/kernel/v2.6/linux-2.6.11.tar.gz
hinzer@ubuntu:source$ tar -xzvf kernel.tar.gz 
```


## 安装
```bash
sudo apt-get install ctags -y
```

## 配置
1. 在当前目录下生成索引文件
```bash
hinzer@ubuntu:source$ cd linux-2.6.11/
hinzer@ubuntu:linux-2.6.11$ ctags -R .	#生成索引tags
```


2. `sudo vim /etc/vim/vimrc`配置vim
```bash
hinzer@ubuntu:~$ sudo vim ~/.vimrc #添加 set tags=/home/hinzer/source/linux-2.6.11/tags;
```



## 使用演示
1. 命令行索引tag
```bash
hinzer@ubuntu:linux-2.6.11$ ctags -R .	#生成索引tags
hinzer@ubuntu:linux-2.6.11$ ll tags
hinzer@ubuntu:linux-2.6.11$ vim -t main  #查找main函数
```


2. vim中使用ctags命令
```text
:ts  	#tagslist,列出索引list
:tp 	#tagspreview 上一个tag
:tn 	#tagsnext 下一个tag
Ctrl+ ] 	#通过光标位置 跳转到定义处
Ctrl+ T 	#返回上一步的光标位置
```


<br>

## 参考资料
> - [百度百科ctags](https://baike.baidu.com/item/ctags/3470337)
> - [linux kernel](www.kernel.org)
