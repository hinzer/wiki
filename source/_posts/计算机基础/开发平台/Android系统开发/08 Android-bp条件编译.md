---
title: 08 Android-bp条件编译
toc: false
date: {{ date }}
tags: [record, Android]
---


### 基本概念
1、 背景
条件编译为我们提供了一种`一套代码兼容多个版本`的解决方案，提高代码的复用率。
在Android7.0之前使用的是Makefle编译，makefile语法支持条件编译，配置到Android.mk文件。
在那以后，开始使用Ninja编译框架，只需要我们配置Android.bp文件，但是bp文件就是一个配置文件，不支持条件编译。
但条件编译又是强需求，所以 google 还是提供了一种条件编译的方法，下面我们就来学习一下。

2、 目的
把 platform sdk version 传给一个 Android.bp 模块的 cpp 代码。 

3、 文件组织结构
```text
hinzer@ubuntu:android-10$ tree ./device/mi/pure/hello
./device/mi/pure/hello  # 我们要编译的hello模块
├── Android.bp      # 
├── hello.cpp 		 # 
└── hello.go 		# 

0 directories, 3 files
```

### 关键步骤
1、 模块编译规则（Android.bp）
```
// add start
bootstrap_go_package {
    name: "soong-hello",
    pkgPath: "android/soong/hello",
    deps: [
        "soong-android",
        "soong-cc",
    ],
    srcs: [
          "hello.go",
    ],
    pluginFor: ["soong_build"],
}

cc_hello_binary {
    name: "hello_defaults",
}
// add end



cc_binary {
    name: "hello",

    // add start
    defaults: ["hello_defaults"],
    // add end

    srcs: ["hello.cpp"],
    vendor: true,
    shared_libs: [
    ],
}

```
查看[Android.bp中已知的模块类型](https://ci.android.com/builds/submitted/6373029/linux/latest/view/soong_build.html)，其中
- 类型为`bootstrap_go_package`的模块名soong-hello，指定了源文件`hello.go`。
- 类型`cc_hello_binary`是在`hello.go`里面进行定义，这里相当于编译条件。
- 类型`cc_binary`的模块名hello，通过hello_defaults模块合入配置，并编译源文件`hello.cpp`成二进制文件。

2、 添加hook机制（hello.go）
```
package hello

import (
        "android/soong/android"
        "android/soong/cc"
        "fmt"
)

func init() {
    android.RegisterModuleType("cc_hello_binary", helloDefaultsFactory)
}

func helloDefaultsFactory() (android.Module) {
    module := cc.DefaultsFactory()
    android.AddLoadHook(module, helloHook)
    return module
}

func helloHook(ctx android.LoadHookContext) {
    //AConfig() function is at build/soong/android/config.go
    fmt.Println("PlatformSdkVersion = ", ctx.AConfig().PlatformSdkVersion())
    fmt.Println("DeviceName = ", ctx.AConfig().DeviceName())

    type props struct {
        Cflags []string
    }
    p := &props{}
    p.Cflags = append(p.Cflags, "-DPLATFORM_SDK_VERSION=" + ctx.AConfig().PlatformSdkVersion())
    ctx.AppendProperties(p)
}

```
hello.go文件中配置hook机制，首先定义模块类型`cc_hello_binary`，然后是hook处理函数`helloHook`。如果这个`cc_hello_binary`被定义，表示编译条件成立，触发hook处理函数
- `init()`里面注册了一个新的模块类型 cc_hello_binary
- 对应的函数是`helloDefaultsFactory()`,需要注意的是其中`cc.DefaultsFactory`要根据模块类型的不同而不同,参考`build/soong/cc/xxx.go`中的定义
- 最后我们希望的条件编译内容可以在hook中定义，通过`ctx`来获取和添加配置信息，可添加的配置类型参考类型props的定义

3、 使用传入的配置（hello.cpp）
```
#include <cstdio>

int main() {
    printf("PLATFORM_SDK_VERSION = %d\n", PLATFORM_SDK_VERSION);
    return 0;
}

```

4、 将模块添加到Build系统（product.mk）
将模块添加到product.mk文件中的PRODUCT_PACKAGES变量(模块会随着编译打包到android系统中)。

### 编译验证
```
hinzer@ubuntu:android-10$ mm hello
...
hinzer@ubuntu:android-10$ emulator -wipe-data
...
hinzer@ubuntu:android-10$ adb shell
adb server version (41) doesn't match this client (39); killing...
* daemon started successfully
pure:/ # hello 					# 模块加载
PLATFORM_SDK_VERSION = 29
pure:/ # ^C
130|pure:/ # exit
```


### 思考总结
Android.bp只是一个json格式的配置文件，不支持条件语句，不支持控制流程语句。所有复杂问题都由用Go编写的编译逻辑处理。
对于bp文件，了解json格式，其他的靠以后慢慢积累；对于go语言，也不用单独学习，以后多写几个demo，总结套路就好。

### 参考资料
> - [官方文档 - Soong编译系统](https://source.android.com/setup/build)
> - [Android.bp 条件编译](http://qiushao.net/2020/02/05/Android%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/15-Anroid.bp%E6%9D%A1%E4%BB%B6%E7%BC%96%E8%AF%91/)


