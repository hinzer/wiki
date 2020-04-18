---
title: 05 删除android内置apk
toc: false
date: 2020-04-05 8:05:10
tags: [record, Android]
---



### 两种方法
为了在编译阶段将内置apk给异常，下面提供2中方法。
- 直接找到添加这个apk的mk文件，从 PRODUCT_PACKAGES 中删除
- 通过添加模块，`LOCAL_OVERRIDES_PACKAGES`定义要覆盖的apk


### 目录结构
```
hinzer@ubuntu:android-10$ tree ./device/mi/pure/
./device/mi/pure/
├── Android.mk
├── AndroidProducts.mk
├── BoardConfig.mk
├── product01.mk
```

### 操作过程
**1、 直接从 PRODUCT_PACKAGES 中删除**
```
# step1 找到apk添加到 PRODUCT_PACKAGES 的那个mk文件
$ mgrep Contacts

# step2 从改mk文件中移除配置项
.....

# step3 清理system目录，重新编译
$ rm -rf out/target/product/pure/system

# step4 验证
make -j4 && emulator
```

**2、 通过 LOCAL_OVERRIDES_PACKAGES 删除**
1. 在Product下添加一个模块`remove_unused_module`
```
$ vim Android.mk

include $(CLEAR_VARS)
LOCAL_MODULE := remove_unused_module
LOCAL_MODULE_TAGS := optional

LOCAL_MODULE_CLASS := FAKE     # 指定编译输出的目录为 $(PRODUCT_OUT)/fake_packages
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)

LOCAL_OVERRIDES_PACKAGES += \
   Contacts \
   Email  			#这里添加要覆盖的apk

include $(BUILD_SYSTEM)/base_rules.mk

$(LOCAL_BUILT_MODULE):
	$(hide) echo "Fake: $@"
	$(hide) mkdir -p $(dir $@)
	$(hide) touch $@

PACKAGES.$(LOCAL_MODULE).OVERRIDES := $(strip $(LOCAL_OVERRIDES_PACKAGES))
```
2. 将`remove_unused_module`添加到对应product文件的PRODUCT_PACKAGES配置
```
PRODUCT_PACKAGES += remove_unused_module
```
3. 清理`out/target/product/pure/system`目录，验证
```
$ rm -rf out/target/product/pure/system
$ make -j4 && emulator
```


### 原理补充(PRODUCT_PACKAGES)
对于第一种方法，通过`mgrep`命令能够搜索到这个mk文件定义了PRODUCT_PACKAGES，直接移除就ok。对于第二种方法，在 main.mk 里面有对 `OVERRIDES_PACKAGES` 进行处理（在[android-10源码](https://cs.android.com/android/platform/superproject/+/master:build/make/core/main.mk?q=_pif_overrides&ss=android%2Fplatform%2Fsuperproject#1064)中对这个关键词进行检索）
```
# Lists most of the files a particular product installs, including:
# - PRODUCT_PACKAGES, and their LOCAL_REQUIRED_MODULES
# - PRODUCT_COPY_FILES
# The base list of modules to build for this product is specified
# by the appropriate product definition file, which was included
# by product_config.mk.
# Name resolution for PRODUCT_PACKAGES:
#   foo:32 resolves to foo_32;
#   foo:64 resolves to foo;
#   foo resolves to both foo and foo_32 (if foo_32 is defined).
#
# Name resolution for LOCAL_REQUIRED_MODULES:
#   If a module is built for 2nd arch, its required module resolves to
#   32-bit variant, if it exits. See the select-bitness-of-required-modules definition.
# $(1): product makefile
define product-installed-files
  $(eval _mk := $(strip $(1))) \
  $(eval _pif_modules := \
    $(PRODUCTS.$(_mk).PRODUCT_PACKAGES) \
    $(if $(filter eng,$(tags_to_install)),$(PRODUCTS.$(_mk).PRODUCT_PACKAGES_ENG)) \
    $(if $(filter debug,$(tags_to_install)),$(PRODUCTS.$(_mk).PRODUCT_PACKAGES_DEBUG)) \
    $(if $(filter tests,$(tags_to_install)),$(PRODUCTS.$(_mk).PRODUCT_PACKAGES_TESTS)) \
    $(if $(filter asan,$(tags_to_install)),$(PRODUCTS.$(_mk).PRODUCT_PACKAGES_DEBUG_ASAN)) \
    $(call auto-included-modules) \
  ) \
  $(eval ### Filter out the overridden packages and executables before doing expansion) \
  $(eval _pif_overrides := $(call module-overrides,$(_pif_modules))) \
  $(eval _pif_modules := $(filter-out $(_pif_overrides), $(_pif_modules))) \
  $(eval ### Resolve the :32 :64 module name) \
  $(eval _pif_modules_32 := $(patsubst %:32,%,$(filter %:32, $(_pif_modules)))) \
  $(eval _pif_modules_64 := $(patsubst %:64,%,$(filter %:64, $(_pif_modules)))) \
  $(eval _pif_modules_rest := $(filter-out %:32 %:64,$(_pif_modules))) \
  $(eval ### Note for 32-bit product, 32 and 64 will be added as their original module names.) \
  $(eval _pif_modules := $(call get-32-bit-modules-if-we-can, $(_pif_modules_32))) \
  $(eval _pif_modules += $(_pif_modules_64)) \
  $(eval ### For the rest we add both) \
  $(eval _pif_modules += $(call get-32-bit-modules, $(_pif_modules_rest))) \
  $(eval _pif_modules += $(_pif_modules_rest)) \
  $(call expand-required-modules,_pif_modules,$(_pif_modules),$(_pif_overrides)) \
  $(filter-out $(HOST_OUT_ROOT)/%,$(call module-installed-files, $(_pif_modules))) \
  $(call resolve-product-relative-paths,\
    $(foreach cf,$(PRODUCTS.$(_mk).PRODUCT_COPY_FILES),$(call word-colon,2,$(cf))))
endef
```
有定义
`$(eval _pif_modules := $(filter-out $(_pif_overrides), $(_pif_modules)))`
[filter-out](https://seisman.github.io/how-to-write-makefile/functions.html#filter-out)是Makefile语法支持的函数，从`$(_pif_modules)`中 反选过滤出`$(_pif_overrides)`之外的所有modules。

> 涉及到android build系统，我现在也还没搞清楚逻辑链，有待补充。



### 参考资料
> - [Android系统开发入门-6.删除Android原生内置APK](http://qiushao.net/2019/12/12/Android%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91%E5%85%A5%E9%97%A8/6-%E5%88%A0%E9%99%A4%E5%8E%9F%E7%94%9F%E5%86%85%E7%BD%AEAPK/)
