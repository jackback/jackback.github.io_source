---
title: Android APP加密和签名
date: 2017-04-12 12:22:33
tags: [加密, 签名]
categories: Android
---



1. 基本概念
公钥加密、私钥解密是密送，保证消息即使公开也只有私钥持有者能读懂。
私钥加密、公钥解密是签名，保证消息来源是私钥持有者。

2. 签名的作用
Android系统不会安装没有进行签名的应用程序
    给apk签名可以带来以下好处：
            1. 应用程序升级
            2. 应用程序模块化：Android系统可以允许同一个证书签名的多个应用程序在一个进程里运行
            3. 代码或者数据共享：Android提供了基于签名的权限机制，那么一个应用程序就可以为另一个以
               相同证书签名的应用程序公开自己的功能。

3. Android标准签名
build/target/product/security目录中有四组默认签名供Android.mk在编译APK使用：
1、testkey/releasekey：普通APK，默认情况下使用。
2、platform：该APK完成一些系统的核心功能。经过对系统中存在的文件夹的访问测试，这种方式编译出来的APK所在进程的                        UID为system。
3、shared：该APK需要和home/contacts进程共享数据。
4、media：该APK是media/download系统中的一环。

生成platform media 等签名文件
/work/aurora/development/tools/make_key：
sh make_key releasekey '/C=CN/ST=HuBei/L=WuHan/O=Company/OU=Department/CN=YourName/emailAddress=YourE-mailAddress'

4. 第三方生成签名
使用/work/sign下的工具生成系统签名jks文件：（供AS 或者 eclipse 使用）
./keytool-importkeypair -k lt_system_me.jks -p liantong -pk8 platform.pk8 -cert platform.x509.pem -alias lt_system

单独给APK签名：
out/host/linux-x86/framework/signapk.jar
build/target/product/security/*
也可以从网上下载。使用方法，以platform为例：
java -jar ./signapk.jar platform.x509.pem platform.pk8 input.apk output.apk 
 (platform.x509.pem私钥 platform.pk8公钥 在build/target/product/security获取)

5. APP中签名后生成文件
APP中META-INF/CERT.RSA 文件数字签名以及一个数字证书
机构密钥标识符AuthorityKeyIdentifier
基本限制BasicConstraints
主体密钥标识符SubjectKeyIdentifier

Owner: CN=ZhaoXin, OU=ZhaoXin, O=ZhaoXin, L=Shanghai, ST=Shanghai, C=CN
Issuer: CN=ZhaoXin, OU=ZhaoXin, O=ZhaoXin, L=Shanghai, ST=Shanghai, C=CN
Serial number: 8e8825ec8c2efe44
Valid from: Tue Apr 22 16:58:39 CST 2014 until: Sat Sep 07 16:58:39 CST 2041
Certificate fingerprints:
         MD5:  55:16:F4:38:BD:49:BE:38:47:03:9B:B6:CA:AE:33:AD
         SHA1: 43:E0:FE:6E:76:BF:53:62:84:15:1B:BD:17:03:67:F1:56:E0:BB:18
         SHA256: 0B:6D:62:97:01:A8:5A:63:27:50:D1:EC:AD:47:59:B9:83:3E:0D:92:A5:E8:FE:95:9D:F0:7E:F2:DC:30:75:05
         Signature algorithm name: SHA1withRSA
         Version: 3

Extensions: 

#1: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: CF FF A6 88 A3 A8 ED 04   C1 60 2F FD 96 EC BC E9  .........`/.....
0010: D3 B4 5F AD                                        .._.
]
]

#2: ObjectId: 2.5.29.19 Criticality=false
BasicConstraints:[
  CA:true
  PathLen:2147483647
]

#3: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: CF FF A6 88 A3 A8 ED 04   C1 60 2F FD 96 EC BC E9  .........`/.....
0010: D3 B4 5F AD                                        .._.
]
]

6. 查看签名
查看签名
keytool -printcert -file META-INF/CERT.RSA

查看keystone信息：
keytool -list -keystore debug.keystore 

7. 其他
反编译
 ./apktool d[ecode] xxx.apk -o <dir>

8. AndroidStudio中签名配置
???
app 未知源检测原理（不同系统不一样吗，不同应用市场是不是也不一样？）
Y:\aurora\development\samples\Home    sample-Launcher 
系统Launcher启动流程分析


    signingConfigs {
        release {
            keyAlias 'lt_system'
            keyPassword 'liantong'
            storeFile file('D:/25264/Android/Project/SGTest/signedAPK/lt_system.jks')
            storePassword 'liantong'
        }
        release2 {
            keyAlias 'lt_system'
            keyPassword 'liantong'
            storeFile file('D:/25264/Android/Project/SGTest/signedAPK/lt_system_me.jks')
            storePassword 'liantong'
        }
    }



### SeeAlso
《APK 反编译及签名工具》

