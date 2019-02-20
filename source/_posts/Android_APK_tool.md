---
title: APK 反编译及签名工具
date: 2017-04-12 12:22:33
tags: [APK反编译, APK签名]
categories: Android
---


# 反编译/解包/再编译

### 1. Apktool——APK反编译及再打包工具
通过反编译APK中XML文件，直接可以查看
https://ibotpeaches.github.io/Apktool/

下载安装Apktool （Windows/Linux/Mac）
https://ibotpeaches.github.io/Apktool/install/

Apktool 通常用法
反编译APK     `apktool d[ecode] test.apk`
再打包成APK   `apktool b[uild] test`


### 2. dex2jar
将apk中的classes.dex转化成Jar文件
https://github.com/pxb1988/dex2jar


### 3. JD-GUI
反编译工具，可以直接查看Jar包的源代码
https://code.google.com/archive/p/innlab/downloads
https://pan.baidu.com/disk/home#list/vmode=list&path=%2Fapps%2FAPK-Track%E5%B7%A5%E5%85%B7


<!-- more -->

# 签名

### 1. keytool 
是个密钥和证书管理工具,可以用来生成证书.
`keytool -genkey -keystore test.keystore  -alias test -keyalg RSA -validity 10000`
>   参数解释:
    -genkey 产生证书文件
    -keystore 指定密钥库的.keystore文件中
    -keyalg 指定密钥的算法,这里指定为RSA(非对称密钥算法)
    -validity 为证书有效天数，这里我们写的是10000天
    -alias 产生别名

### 2. jarsigner 
工具利用密钥仓库中的信息来产生或校验 Java 存档 (JAR) 文件的数字签名

可以使用jarsigner 来签名,例子如下:
`jarsigner -verbose -keystore test.keystore -signedjar -signed.apk unsigned.apk 'test.keystore'`
>   参数说明:
    -verbose：指定生成详细输出
    -keystore：指定数字证书存储路径
    -signedjar：该选项的三个参数为 签名后的apk包 未签名的apk包 数字证书别名(注意顺序)
    




# To Do
后续将qq记事本中《加密和签名（及反编译）》 整合到本文 ...




# SeeAlso
《Android APP加密和签名》
