---
title: Android 权限的实现
date: 2016-05-28 12:22:33
tags: [permission]
categories: Android
---


1. 权限
每个程序在安装时都有建立一个系统ID，如app_15，用以保护数据不被其它应用获取。Android根据不同的用户和组，
分配不同权限，比如访问SD卡，访问网络等等。底层映射为Linux权限。


2. Android权限的实现
    1) 第一层：由应用设置，修改AndroidManifest.xml，形如：
    <uses-permission android:name=”android.permission.INTERNET”/>

        应用申请权限
        1) 应用开发者通过AndroidManifest.xml中<uses-permission>指定对应权限，再映射到底层的用户和组，默认情况下不设定特殊的权限。  
           AndroidManifest加入权限后系统安装程序时会在图形界面中提示权限

        2) 如果是缺少某个权限（程序中使用的某种权限而在AndroidManifest.xml中并未声名），
           程序运行时会在logcat中打印出错误信息requires <permission>
        3) 与某个进程使用相同的用户ID
           应用程序可与系统中已存在的用户使用同一权限，需要在AndroidManifest.xml中设置sharedUserId，如android:sharedUserId="android.uid.shared"，作用是获得系统权限，
           但是这样的程序属性只能在build整个系统时放进去（就是系统软件）才起作用，共享ID的程序必须是同一签名的

    2) 第二层：框架层，权限对应组，frameworks/base/data/etc/platform.xml，形如：
    <permission name=”android.permission.INTERNET”>
        <group gid=inet” />
    </permission>

    3) 第三层：系统层，系统的权限，system/core/include/private/android_filesystem_config.h,形如：
    #define AID_INET 3003      //建立SOCKET的权限
    ……
    { “inet”, AID_INET, },


3. 系统权限
    特殊权限的用户
        a)  system     uid 1000
        b)  radio      uid 1001

4. permission权限查看
        $ adb shell
        $ pm list permissions


5. framework层对权限的判断
    1) 相关源码实现
        frameworks/base/services/Java/com/android/server/PackageManagerService.java
        frameworks/base/services/java/com/android/server/am/ActivityManagerService.java
    2) 在系统层，如何查看某个应用的权限
        a) 在应用进程开启时，ActivityManagerService.java会在logcat中输出该应用的权限，形如：
            I/ActivityManager(1730): Start proc com.anbdroid.phone for restart com.android.phone:pid=2605 uid=1000 gids={3002,3001,3003}
           即它有3001,3002,3003三个权限：访问蓝牙和建立socket
        b) 注意：此打印输出在应用第一次启动时。如果进程已存在，需要先把对应进程杀掉，以保证该进程重新启动，才能显示
        c) 具体实现，见： framewors/base/services/java/com/android/server/am/ActivityManagerService.java的
           函数startProcessLocked()，其中取其组信息的具本语句是mContext.getPackageManager().getPackageGids(app.info.packageName);


6. 参考
http://wenku.baidu.com/view/7754a4360b4c2e3f5727634e.html


