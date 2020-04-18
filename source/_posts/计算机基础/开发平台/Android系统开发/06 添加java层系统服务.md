---
title: 06 添加java层系统服务
toc: false
date: 2020-04-05 8:28:10
tags: [record, Android]
---


### 引入概念
目前对android系统体系了解比较少，主要区分一下`服务`、`系统服务`这两个概念
- [Android服务](https://developer.android.com/guide/components/services)是一个后台运行的组件，执行长时间运行且不需要用户交互的任务。在android开发中作为一个`应用组件`,通过继承类`extern Service`来使用。
- [Android系统服务](https://blog.csdn.net/u010753159/article/details/52193061)。理解为随着andorid系统启动运行的service，分为`本地守护进程`、`Native系统服务`和`Java系统服务`。

有相同点更有不同点，但请不要把两个概念弄混淆了!!!\
然后下面记录一下`添加自定义一个java系统服务`的步骤，参考于[qiushao](http://qiushao.net/2019/12/20/Android%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/7-%E6%B7%BB%E5%8A%A0java%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1/)大神的blog。基于`android-10`版本的AOSP源码，

### 添加服务
**1、 定义服务接口**
首先我们得定义我们的服务名是什么，提供什么样的接口。在`frameworks/base/core/java/android`目录下添加pure文件夹
```text
$ tree frameworks/base/core/java/android/pure
frameworks/base/core/java/android/pure
└── IHelloService.aidl     # 使用 aidl 定义服务接口

0 directories, 1 file
```
定义接口文件`IHelloService.aidl`，模块名`IHelloService`,接口`hello`将实现播放指定路径的音频文件
```text
package android.pure;
interface IHelloService {
    void hello(String name);
}

```
`frameworks/base/Android.bp`文件中找到模块名`framework-defaults`，添加
```text
"core/java/android/pure/IHelloService.aidl",
```
*此时，进入 `framework/base` 目录执行 `mm -j` 命令编译 `framework.jar` 模块。
编译成功后，会在 `out/soong/.intermediates/frameworks/base/framework/android_common/gen/aidl/frameworks/base/core/java/android/pure` 目录生成 `IHelloService.java` 这个文件*


**2、 实现接口**
然后在`frameworks/base/services/core/java/com/android/server`下创建`HelloService.java`文件(接口实现) 内容如下
```java
package com.android.server;

import android.pure.IHelloService;
import android.util.Log;

public class HelloService extends IHelloService.Stub {
    private final String TAG = "HelloService";

    public HelloService() {
        Log.d(TAG, "create hello service");
    }

    @Override
    public void hello(String name) {
        Log.d(TAG, "hello " + name);
    }
}
```

**3、 将服务添加到 ServiceManager**
修改 `frameworks/base/services/java/com/android/server/SystemServer.java` 文件，在`startOtherServices`方法里面增加以下代码
```text
// add hello service
traceBeginAndSlog("HelloService");
ServiceManager.addService("HelloService", new HelloService());
traceEnd();
```

**4、 编译验证 & 系统无法启动**
现在已经实现的`HelloService`接口模块，并添加到`ServiceManager`。开始尝试整编下android源码
```bash
$ source ./build/envsetup.sh   # 导出环境变量(之前执行过了)
$ lunch product01-eng          # 选择Product
$ make api-stubs-docs-update-current-api -j4            # 更新api接口
$ make -j4            # 编译
```

然后启动`emulator`虚拟机，发现一直停留在logo界面，说明系统没起来。。这时候可以adb调试，我们查看一下log记录
```bash
$ emulator      # 发现android系统界面没起来
...
...
$ adb shell logcat -b all > logSystem.txt    # 抓取android层的 log
...
^C
```
日志中检索我们想要的关键字`HelloService`,发现
```text
04-02 01:24:25.871  2224  2224 I SystemServer: HelloService
04-02 01:24:25.871  1528  1528 I auditd  : avc:  denied  { add } for service=HelloService pid=2224 uid=1000 scontext=u:r:system_server:s0 tcontext=u:object_r:default_android_service:s0 tclass=service_manager permissive=0
04-02 01:24:25.871  2224  2224 E System  : ******************************************
04-02 01:24:25.871  2224  2224 E System  : ************ Failure starting system services
04-02 01:24:25.871  2224  2224 E System  : java.lang.SecurityException
04-02 01:24:25.871  2224  2224 E System  :  at android.os.BinderProxy.transactNative(Native Method)
04-02 01:24:25.871  2224  2224 E System  :  at android.os.BinderProxy.transact(BinderProxy.java:510)
04-02 01:24:25.871  2224  2224 E System  :  at android.os.ServiceManagerProxy.addService(ServiceManagerNative.java:156)
04-02 01:24:25.871  2224  2224 E System  :  at android.os.ServiceManager.addService(ServiceManager.java:192)
04-02 01:24:25.871  2224  2224 E System  :  at android.os.ServiceManager.addService(ServiceManager.java:161)
04-02 01:24:25.871  2224  2224 E System  :  at com.android.server.SystemServer.startOtherServices(SystemServer.java:920)
04-02 01:24:25.871  2224  2224 E System  :  at com.android.server.SystemServer.run(SystemServer.java:512)
04-02 01:24:25.871  2224  2224 E System  :  at com.android.server.SystemServer.main(SystemServer.java:349)
04-02 01:24:25.871  2224  2224 E System  :  at java.lang.reflect.Method.invoke(Native Method)
04-02 01:24:25.871  2224  2224 E System  :  at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:492)
04-02 01:24:25.871  2224  2224 E System  :  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:908)
04-02 01:24:25.871  2224  2224 D SystemServerTiming: HelloService took to complete: 1ms
04-02 01:24:25.871  2224  2224 E AndroidRuntime: *** FATAL EXCEPTION IN SYSTEM PROCESS: main
```
然后定位到这一句`04-02 01:24:25.871  1528  1528 I auditd  : avc:  denied  { add } for service=HelloService pid=2224 uid=1000 scontext=u:r:system_server:s0 tcontext=u:object_r:default_android_service:s0 tclass=service_manager permissive=0`,不理解没关系，google一下。发现是selinux方面的原因。

为了进一步验证猜想，将系统中的`selinux`关掉试试。通过`setenforce 0`命令，重启系统发现正常了，印证了之前的猜想，确实selinux配置的原因。
```bash
$ adb shell setenforce 0   # 禁用selinux

$ emulator -wipe-data      # 擦掉data区，重启系统

```


**5、 设置selinux规则**
然后我们只需要添加这个自定义服务`HelloService`相关的 SELinux 规则。为了方便之后验证，打开selinux
```bash
$ adb shell setenforce 1   # 打开selinux
```
Android 10 的 selinux 规则是放在 `system/sepolicy` 目录下的。不怎么了解SELinux规则，可以参考现有的系统服务的规则去添加，这里参考的是 `network_time_update_service` 服务。
```bash
$ cd system/sepolicy    # selinux规则在这个目录
$ grep -nr network_time_update_service   # 查找network_time_update_service服务相关的selinux配置
```
涉及到的文件很多，有部分文件是不需要修改的，我们先把找到的所有 service.te 和 service_contexts 都参考 network_time_update_service 加上 HelloService 的配置。
```text
$ find -name service.te
./prebuilts/api/27.0/public/service.te   # 需要
./prebuilts/api/28.0/public/service.te   # 需要
./prebuilts/api/28.0/private/service.te
./prebuilts/api/29.0/public/service.te    # 需要
./prebuilts/api/29.0/private/service.te
./prebuilts/api/26.0/public/service.te   # 需要
./public/service.te                 # 需要
./private/service.te
$ find -name service_contexts
./reqd_mask/service_contexts
./prebuilts/api/27.0/private/service_contexts   # 需要
./prebuilts/api/28.0/private/service_contexts   # 需要
./prebuilts/api/29.0/private/service_contexts       # 需要
./prebuilts/api/26.0/private/service_contexts   # 需要
./private/service_contexts         # 需要
```
其中
- `service_contexts`上添加`HelloService                              u:object_r:HelloService:s0`;
- `service.te`上添加`type HelloService, system_server_service, service_manager_type;`。

6、 编译验证
最后再次编译一遍，启动届满没有什么问题了。adb进入系统查看一下有没有这个服务
```

$ source ./build/envsetup.sh   # 导出环境变量(之前执行过了)
$ lunch product01-eng          # 选择Product
$ make api-stubs-docs-update-current-api -j4            # 更新api接口
$ make -j4            # 编译
....
....
$ adb shell
pure:/ # service list | grep HelloService                                                                                                                                                               
16      HelloService: [android.pure.IHelloService]
pure:/ #
```

### 错误记录
**1、 添加服务后,`make -j4`编译系统报错**
```text
error: Added package android.pure [AddedPackage]
Aborting: Found compatibility problems checking the public API against the API in /home/mi/source/android-10/frameworks/base/api/current.txt
-e 
******************************
You have tried to change the API from what has been previously approved.

To make these errors go away, you have two choices:
   1. You can add '@hide' javadoc comments to the methods, etc. listed in the
      errors above.

   2. You can update current.txt by executing the following command:
         make api-stubs-docs-update-current-api

      To submit the revised current.txt to the main Android repository,
      you will need approval.
******************************

09:05:24 ninja failed with: exit status 1

#### failed to build some targets (06:37 (mm:ss)) ####
```
解决思路: 没有更新服务接口，编译前执行`make api-stubs-docs-update-current-api -j4`


**2、添加selinux后,编译报错**
```text
SELinux: The following public types were found added to the policy without an entry into the compatibility mapping file(s) found in private/compat/V.v/V.v[.ignore].cil, where V.v is the latest API level.
HelloService

See examples of how to fix this:
https://android-review.git.corp.google.com/c/platform/system/sepolicy/+/781036
https://android-review.git.corp.google.com/c/platform/system/sepolicy/+/852612

[ 20% 5/24] build out/target/product/pure/obj/ETC/treble_sepolicy_tests_28.0_intermediates/treble_sepolicy_tests_28.0
FAILED: out/target/product/pure/obj/ETC/treble_sepolicy_tests_28.0_intermediates/treble_sepolicy_tests_28.0
/bin/bash -c "(out/host/linux-x86/bin/treble_sepolicy_tests -l    out/host/linux-x86/lib64/libsepolwrap.so  -f out/target/product/pure/obj/ETC/plat_file_contexts_intermediates/plat_file_contexts  -f out/target/product/pure/obj/ETC/vendor_file_contexts_intermediates/vendor_file_contexts    -b out/target/product/pure/obj/ETC/built_plat_sepolicy_intermediates/built_plat_sepolicy -m out/target/product/pure/obj/ETC/treble_sepolicy_tests_28.0_intermediates/28.0_mapping.combined.cil    -o out/target/product/pure/obj/ETC/treble_sepolicy_tests_28.0_intermediates/built_28.0_plat_sepolicy -p out/target/product/pure/obj/ETC/sepolicy_intermediates/sepolicy     -u out/target/product/pure/obj/ETC/built_plat_sepolicy_intermediates/base_plat_pub_policy.cil     --fake-treble ) && (touch out/target/product/pure/obj/ETC/treble_sepolicy_tests_28.0_intermediates/treble_sepolicy_tests_28.0 )"
SELinux: The following public types were found added to the policy without an entry into the compatibility mapping file(s) found in private/compat/V.v/V.v[.ignore].cil, where V.v is the latest API level.
HelloService

See examples of how to fix this:
https://android-review.git.corp.google.com/c/platform/system/sepolicy/+/781036
https://android-review.git.corp.google.com/c/platform/system/sepolicy/+/852612

21:45:48 ninja failed with: exit status 1

#### failed to build some targets (16 seconds) ####

```
解决思路: selinux没配置好，没有将配置覆盖所有需要的te文件。检查本地配置，按照上面的方法，在配置一下。

**3、启动emulator之后，系统没起来，同时log报错**
```text
hinzer@ubuntu:android-10$ emulator -wipe-data
emulator: WARNING: Couldn't find crash service executable /home/hinzer/source/android-10/prebuilts/android-emulator/linux-x86_64/emulator64-crash-service

emulator: WARNING: system partition size adjusted to match image file (3083 MB > 800 MB)

qemu_ram_alloc_user_backed: call
context mismatch in svga_surface_destroy   # 这个错误
....
```
解决方法: 通过网络查询到[解决方法](https://unix.stackexchange.com/questions/422993/syslogs-getting-spammed-with-context-mismatch-in-svga-sampler-view-destroy),由于我这边是VM上运行Ubuntu虚拟机，然后在跑emulator。我需要把VM设置中的显示器中的图形渲染关闭，这一操作需要关闭虚拟机，重启后在运行emulator 验证就没有问题了。

### 小结
- 查看日志 使用`logcat -b all`，不清楚不要过滤处理，查看全部log
- 不确定的情况下，可以系统上直接启动/禁止`selinux`验证猜想，然后进一步调试


### 参考资料
> - [Android系统开发入门-7.添加java层系统服务](http://qiushao.net/2019/12/20/Android%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/7-%E6%B7%BB%E5%8A%A0java%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1/)
> - [官方文档 - 服务概览](https://developer.android.com/guide/components/services)
> - [官方文档- SELinux](https://source.android.com/security/selinux)
> - [Android 系统服务](https://blog.csdn.net/u010753159/article/details/52193061)








