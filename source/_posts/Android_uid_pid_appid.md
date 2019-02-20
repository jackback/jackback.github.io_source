---
title: Android Pid/Tid/UserId/PackageName/ProcessName/shareUserId/签名
date: 2016-08-31 14:43:33
tags: [APK反编译,APK签名]
categories: Android
---

这篇博文只能算是笔记了，比较杂。


Pid         : Linux 进程ID
Tid         : Linux 线程ID，或者任务ID
UID         : Android中的UID不同于Linux中的UID，Linux中UID是针对多用户操作系统中用于区分用户的；而Android中的UID是分配给各个进程使用，用来做权限管理的。
userID      : Android自从Android 4.2.2 之后就有支持多用户的拓展，只不过现在只有user 只有一个 = 0，故userID = 0。
AppID       : 单用户时和UID值是一样的。
shareUserId : 拥有同一个UID的多个APK可以配置成运行在同一个进程中. 所以默认就是可以互相访问任意数据. 也可以配置成运行成不同的进程, 同时可以访问其他APK的数据目录下的数据库和文件. 就像访问本程序的数据一样.
              e.g. android:sharedUserId="android.uid.system" "android.uid.phone" "android.uid.bluetooth" 
                                        "android.uid.shell" "android.uid.calendar" "android.uid.shared" "android.media" "android.uid.systemui" 

              e.g. android:sharedUserId="com.rainwii.share"

签名 : 1. Android 标准签名为platform  media shared testkey/releasekey； 2. 第三方签名  
       可以在源码的/build/target/product/security里面看到对应的密钥，其中shared.pk8代表私钥，shared.x509.pem公钥，一定是成对出现的。

LOCAL_CERTIFICATE : 配置应用使用什么签名 platform  media shared 或者第三方签名 presigned， 默认配置的话使用testkey/releasekey .


注：
shareUserID一样的，签名也必须相同


<!-- more -->

1. 
所有运行的App进程信息list： mActivityManager.getRunningAppProcesses()可以使用for 循环来解析

2. 
ps信息---------
UID            PID   PPID  VSIZE   RSS   WCHAN    PC          package name
u0_a324        15713 231   1549320 79328 ffffffff 00000000 S  com.china_liantong.sgtest

Uid Pid Tid 及PackageName：
Uid = Binder.getCallingUid()；  //返回当前进程Uid或者远程调用进程的Uid   //eg:10324
Uid = context.getApplicationContext().getApplicationInfo().uid;   //eg:10324
Uid = android.os.Process.myUid();   //eg:10324
Pid = android.os.Process.myPid();   //eg:15713
Tid = android.os.Process.myTid();  线程ID，一般情况都是单线程应该  == Pid  ？？？  //15713

PackageName = context.getPackageName();   //com.china_liantong.sgtest
PackageName = AppGlobals.getPackageManager().getNameForUid(Uid);    //com.china_liantong.sgtest
             (使用限制：不在sdk中，不能在外部apk开始中使用)
PackageName = context.getPackageManager().getNameForUid(Uid); //com.china_liantong.sgtest




3. 
UserId = 如果支持多用户：Uid/100000         否则：0（一般都是0）       高六位
AppId = Uid % 100000   即AppId 是Uid 的低五位      //因为user=0, UID<100000, 所以APPID=Uid
    （范围Process.FIRST_APPLICATION_UID=10000----Process.LAST_APPLICATION_UID=19999）

Uid：  //10324
    如果支持多用户的话（一般userId 都是0，所以Uid = AppId）
        UserId * 100000 + (AppId % 100000);
    否则
        appId

Gid：  //固定值9997
    如果系统支持多用户的话：
        userId*100000 + 9997
    否则
        9997

uid <  10000 系统进程 root system install keystore logd shell dhcp radio                //个人见解
uid >= 10000 有多用户的概念设计保留，但是目前userid都是=0   uid = userid*100000 + appid    //个人见解
                10000~99999   给user 0 分配的UID = 100000 * 0 + AppID = AppID，所以单用户时uid=AppID
                100000~999999 给user 1 分配的UID
                ...


UID值设定（系统核心进程的UID是固定的<1000-9999>，其他应用的UID是在安装时随机生成的userID * 100000 + AppID      <appID=10000-99999>）

appID=10000-99999 细分：
    application appid:          [10000, 19999]
    shared application appid:   [50000, 59999]
    isolated appid:             [99000, 99999]



>ROOT_UID           = 0;
//Java层系统 ——“zygote”
SYSTEM_UID          = 1000;     //shareId
//Java层系统应用，指定UID
PHONE_UID           = 1001;  
BLUETOOTH_UID       = 1002;     //Bluetooth service process.
SHELL_UID           = 2000;  
LOG_UID             = 1007;  
WIFI_UID            = 1010;     //WIFI supplicant process
MEDIA_UID           = 1013;     //mediaserver process.
DRM_UID             = 1019;     //DRM process.
SDCARD_RW_GID       = 1015;  
VPN_UID             = 1016;     //the group that controls VPN services.
NFC_UID             = 1027;     //NFC service process.
MEDIA_RW_GID        = 1023;     //internal media storage.
PACKAGE_INFO_GID    = 1032;     //installed package details
SHELL_UID           = 2000;     //user shell.

//应用程序UID范围  
FIRST_APPLICATION_UID   = 10000;  
LAST_APPLICATION_UID    = 19999;  
//fully isolated sandboxed processes UID范围  
FIRST_ISOLATED_UID      = 99000;  
LAST_ISOLATED_UID       = 99999;  




4. packages.list  
>pkgName                                    userId   debugFlag dataPath
固定UID的应用
android                                     1000     0 /data/system platform 3002,3001,1028,1015,3003                               //UID= system = 1000  
com.android.settings                        1000     0 /data/data/com.android.settings platform 3002,3001,1028,1015,3003            //android:sharedUserId="android.uid.system"     platform签名
com.android.providers.settings              1000     0 /data/data/com.android.providers.settings platform 3002,3001,1028,1015,3003  //android:sharedUserId="android.uid.system"  platform签名
com.android.inputdevices                    1000     0 /data/data/com.android.inputdevices platform 3002,3001,1028,1015,3003        //android:sharedUserId="android.uid.system"  platform签名
com.android.server.telecom                  1000     0 /data/data/com.android.server.telecom platform 3002,3001,1028,1015,3003      //android:sharedUserId="android.uid.system"   platform签名
com.android.keychain                        1000     0 /data/data/com.android.keychain platform 3002,3001,1028,1015,3003            //android:sharedUserId="android.uid.system"    platform签名
com.android.location.fused                  1000     0 /data/data/com.android.location.fused platform 3002,3001,1028,1015,3003      //android:sharedUserId="android.uid.system"    platform签名
com.fiship.cibn                             1000     0 /data/data/com.fiship.cibn platform 3002,3001,1028,1015,3003                 //android:sharedUserId="android.uid.system"     platform签名

>com.android.phone                          1001     0 /data/data/com.android.phone platform 1028,1015,3002,3001,3003               //android:sharedUserId="android.uid.phone"   platform签名
com.android.providers.telephony             1001     0 /data/data/com.android.providers.telephony platform 1028,1015,3002,3001,3003 //android:sharedUserId="android.uid.phone" platform签名

>com.android.bluetooth                      1002     0 /data/data/com.android.bluetooth platform 3003,3002,3001,1028,1015,3005,1016,3008 //android:sharedUserId="android.uid.bluetooth"  platform签名

>com.android.shell                          2000     0 /data/data/com.android.shell platform 3002,1028,1015,1023,3008               //android:sharedUserId="android.uid.shell" platform签名    


//未指定UID应用
>com.android.providers.calendar             10000    0 /data/data/com.android.providers.calendar default 3003,1028,1015             //android:sharedUserId="android.uid.calendar"   无指定签名

>com.android.contacts                       10001    0 /data/data/com.android.contacts default 3003,1028,1015                       //android:sharedUserId="android.uid.shared"     share签名
com.android.providers.contacts              10001    0 /data/data/com.android.providers.contacts default 3003,1028,1015             //android:sharedUserId="android.uid.shared"     share签名
com.android.providers.userdictionary        10001    0 /data/data/com.android.providers.userdictionary default 3003,1028,1015       //android:sharedUserId="android.uid.shared"     share签名

>com.android.defcontainer                   10002    0 /data/data/com.android.defcontainer platform 1028,1015,1023,2001,1035

>com.android.providers.media                10003    0 /data/data/com.android.providers.media default 1028,1015,1023,1024,2001,3003,3007         //android:sharedUserId="android.media" media签名
com.android.providers.downloads             10003    0 /data/data/com.android.providers.downloads default 1028,1015,1023,1024,2001,3003,3007     //android:sharedUserId="android.media" media签名
com.android.providers.downloads.ui          10003    0 /data/data/com.android.providers.downloads.ui default 1028,1015,1023,1024,2001,3003,3007  //android:sharedUserId="android.media" media签名

>com.android.externalstorage                10004    0 /data/data/com.android.externalstorage platform 1028,1015,1023
com.android.proxyhandler                    10005    0 /data/data/com.android.proxyhandler platform 3003
com.android.sharedstoragebackup             10006    0 /data/data/com.android.sharedstoragebackup platform 1028,1015,1023
com.android.systemui                        10007    0 /data/data/com.android.systemui platform 1028,1015,1035,3002,3001,3006       //android:sharedUserId="android.uid.systemui"  未绑定UID  platform签名
com.android.vpndialogs                      10008    0 /data/data/com.android.vpndialogs platform none
com.android.browser                         10009    0 /data/data/com.android.browser default 3003,1028,1015
com.android.calendar                        10010    0 /data/data/com.android.calendar default 3003
com.android.captiveportallogin              10011    0 /data/data/com.android.captiveportallogin platform 3003
com.android.certinstaller                   10012    0 /data/data/com.android.certinstaller platform none                           //platform签名
com.android.deskclock                       10013    0 /data/data/com.android.deskclock default 1028
com.android.documentsui                     10014    0 /data/data/com.android.documentsui platform none                             //platform签名
com.android.inputmethod.latin               10015    0 /data/data/com.android.inputmethod.latin default 1028,1015
com.android.launcher                        10016    0 /data/data/com.android.launcher default none                                 //share签名，与contact 有交互
com.coocaa.multiscreenservice               10017    0 /data/data/com.coocaa.multiscreenservice platform 3003,1028,1015
com.android.music                           10018    0 /data/data/com.android.music default 3003,1028,1015
com.android.pacprocessor                    10019    0 /data/data/com.android.pacprocessor platform 3003
com.android.packageinstaller                10020    0 /data/data/com.android.packageinstaller platform 1028
com.svox.pico                               10021    0 /data/data/com.svox.pico default 1028,1015
com.android.printspooler                    10022    0 /data/data/com.android.printspooler default none
com.android.provision                       10023    0 /data/data/com.android.provision platform none
com.android.quicksearchbox                  10024    0 /data/data/com.android.quicksearchbox default 3003
com.android.webview                         10025    0 /data/data/com.android.webview default none






附（缩写）：

PKCS：Public-Key Cryptography Standards (PKCS) ，公钥加密标准
PKCS#8：定义了保存private key信息的标准语法，用来保存private keys。PKCS#8 有两个版本，加密的和非加密的。 