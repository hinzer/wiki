---
title: 01 添加Product
toc: false
date: 2020-04-05 8:03:10
tags: [record, Android]
---


### 理解概念
1、Product
在android源码正式编译之前选择Product，使用`lunch product-xxx`，这一步操作理解为预先对要编译的源码进行一系列的配置。在android-10的源码中，将一个Product配置分成三个部分:

- BoardConfig.mk: 芯片硬件相关配置，分区设置等
- product.mk: 一个产品的软件相关的配置，比如内置哪些软件模块，由AndroidProducts.mk 中的PRODUCT_MAKEFILES指定
- AndroidProducts.mk: 指定 product 配置,并把 product 添加到 lunch 选择项中

2、组织结构
Google为AOSP源码内置了Product配置，位于源码的`build/target`目录:
```text
hinzer@ubuntu:target$ pwd
/home/hinzer/source/android-10/build/target
hinzer@ubuntu:target$ tree -L 1
.
├── board
├── OWNERS
└── product

2 directories, 1 file

```
同时也允许第三方定制Product配置，在源码`device`目录下。待会自定义Product在这个目录下:
```text
hinzer@ubuntu:device$ tree mi 
mi 		# 公司名
└── pure 	# device名(写为Product,与product区分),一个device可对应多个product
    ├── AndroidProducts.mk  # 指定product配置，添加lunch选项
    ├── BoardConfig.mk 		# 硬件配置 boardconfig
    └── product01.mk 		# 软件配置 product

1 directory, 3 files
```

### 自定义product
模仿aosp源码的Product配置,就引用了`build/target/board/generic_x86_64/BoardConfig.mk`和`build/target/board/generic_x86_64/BoardConfig.mk`的配置。然后进行自定义

1、创建`device/[company]/[device]`目录
``` bash
hinzer@ubuntu:android-10$ mkdir -p ./device/mi/pure
```

2、分别添加`AndroidProducts.mk`、`product.mk`、`BoardConfig.mk`配置文件
``` text
hinzer@ubuntu:pure$ ls
AndroidProducts.mk  BoardConfig.mk  product01.mk

# 1.添加 AndroidProducts.mk
hinzer@ubuntu:pure$ cat AndroidProducts.mk 
PRODUCT_MAKEFILES := \
    $(LOCAL_DIR)/product01.mk   # 指定 product

COMMON_LUNCH_CHOICES := \
    product01-eng 				# 添加lunch选项

# 2.添加 BoardConfig.mk 
hinzer@ubuntu:pure$ cat BoardConfig.mk 
include $(SRC_TARGET_DIR)/board/generic_x86_64/BoardConfig.mk  # 这里直接饮用

# 3.添加 product01.mk 
hinzer@ubuntu:pure$ cat product01.mk 
$(call inherit-product, $(SRC_TARGET_DIR)/product/aosp_x86_64.mk)

PRODUCT_NAME   := product01 # product名(与文件保持一致)
PRODUCT_DEVICE := pure      # device名，BoardConfig.mk相关
```

3、lunch刚才创建的product，编译
``` bash
hinzer@ubuntu:android-10$ source ./build/envsetup.sh
hinzer@ubuntu:android-10$ lunch product01-eng
hinzer@ubuntu:android-10$ make -j4
```

4、验证
``` bash
# 运行虚拟机
hinzer@ubuntu:android-10$ emulator   # 查看Android version信息，编译时间、产品名是否对应
```

### 理论补充
1、`build variants`
aosp为build系统提供三种Product配置，文档里叫做[`build variants`](https://source.android.com/setup/develop/new-device#build-variants),分别是:

- eng : 对应到工程版。编译打包所有模块。表示adbd处于ROOT状态，所有调试开关打开
- userdebug : 对应到用户调试版。打开调试开关，但并没有放开ROOT权限
- user : 对应到用户版。关闭调试开关，关闭ROOT权限。最终发布到用户手上的版本，通常都是user版。


### 参考资料
> - [AOSP开发文档 - 添加新设备](https://source.android.com/setup/develop/new-device)
> - [Android系统开发入门-2.添加product](http://qiushao.net/2019/11/19/Android%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/2-%E6%B7%BB%E5%8A%A0product/)