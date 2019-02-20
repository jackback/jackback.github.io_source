---
title: Android NDK移植杂项
date: 2016-00-00 00:00:00
tags: [Android, NDK]
categories: Android
---


# 1
android上gcc编译与linux上有点点差别：
 ndk中gcc不需要需要显式指定-lpthread。Android pthread系列函数libc_bionic.a(即bionic C库)中，所以在默认就会链接。


