---
title: Android Log
date: 2016-05-26 15:13:00
tags: [log, trace]
categories: Android
---


# Android 各层中日志打印功能的应用

### 1. HAL层

头文件：
#include <utils/Log.h>
Android.mk中：

LOCAL_SHARED_LIBRARIES := liblog libcutils


对应的级别 打印方法  
VERBOSE LOGV()
DEBUG LOGD()
INFO LOGI()
WARN LOGW()
ERROR LOGE()

方法：
LOGD("%d, %s", int, char* )


### 2. JNI层

头文件：
#include <utils/Log.h>

对应的级别 打印方法  
VERBOSE LOGV()
DEBUG LOGD()
INFO LOGI()
WARN LOGW()
ERROR LOGE()

方法：
LOGD("%d, %s", int, char* )


### 3. FRAMEWORK层

import android.util.Slog;

对应的级别 打印方法  
VERBOSE Slog.v()
DEBUG Slog.d()
INFO Slog.i()
WARN Slog.w()
ERROR Slog.e()

方法：
Slog.d(TAG, "something to say.");


### 4. Java层

import android.util.Log;

对应的级别 打印方法  
VERBOSE Log.v()
DEBUG Log.d()
INFO Log.i()
WARN Log.w()
ERROR Log.e()

方法：
Log.d(TAG, "something to say.");



--------------------------------------------------------------


# 一、LOG等级：Verbose,Debug,Info,Warn,Error, Assert,Suppress】 
    Verbose最低级别，什么信息都输出；A是最高级别的日志，即assert；S表示Suppress，即停止该日志的输出。


# 二、选用标准：

Verbose：开发调试过程中一些详细信息，Verbose等级的Log，请不要在user版本中出现

>Java示例
>   import android.os.Build;  
    import android.util.Log
    final public Boolean isEng =Build.TYPE.equals("eng");
    if (isEng)
        Log.v(“LOG_TAG”,“LOG_MESSAGE”);

>C/C++示例
>   #include<cutils/log.h>     -----------C/C++示例----------
    char value[PROPERTY_VALUE_MAX];   int isEng=0;
    property_get("ro.build.type",value, "user");
    isEng=strcmp(value, "eng");
    if (isEng)
        ALOGV();

Debug: 用于调试的信息，编译进产品，但可以在运行时关闭，Debug等级的log,默认不开启，通过终端命令开启;

>Java示例
>   import android.util.Log      ---------------------------------
    final String TAG="MyActivity";
    final public Boolean LOG_DEBUG = Log.isLoggable(TAG, Log.DEBUG); 
    if (LOG_DEBUG)
        Log.d(“LOG_TAG”,“LOG_MESSAGE”);
>    
>   运行时开启log: 在终端输入：setprop log.tag.MyActivity DEBUG
    运行时关闭log: 在终端输入：setprop log.tag.MyActivity INFO

>C/C++示例
>   #include<cutils/log.h>   -----------------
    #define LOG_CTL "debug.MyActivity.enablelog"
    charvalue[PROPERTY_VALUE_MAX]; 
    int isDebug=0;
    property_get(LOG_CTL, value, "0");
    isDebug = strcmp(value,"1");
    if (isDebug)
        ALOGD();
>  
>   运行时开启log: 在终端输入：setprop debug.MyActivity.enablelog 1
    运行时关闭log: 在终端输入：setprop debug.MyActivity.enablelog 0


附：Log.isLoggable(TAG, log.DEBUG)   //这个函数表示通过setprop设置的level需要小于DEBUG，值才为True。（默认setprop配置的值是INFO，INFO>DEBUG所以你不配置时，值为false的），通过setprop log.tag.<TAG>  <LEVEL>  通过设置属性的方式来改变默认level，也可以将这些属性按照log.tag.<TAG>=<LEVEL>的形式，写入/data/local.prop中；

Info:例如一些运行时的状态信息，这些状态信息在出现问题的时候能提供帮助。
Warn：警告系统出现了异常，即将出现错误。
Error：系统已经出现了错误。

注：Info、Warn、Error等级的Log禁止作为普通的调试信息使用，这些等级的Log是系统出现问题时候的重要分析线索，如果随意使用，将给Log分析人员带来极大困扰。请参考前面的等级介绍合理使用。

注2：禁止使用new Exception("print trace").printStackTrace()或者Log. getStackTraceString(Exception)方式打印普通调试信息，因为这种方式打印Log非常消耗系统资源。此种方式打印Log一般只出现try..catch某个异常使用。

注3：Log的tag命名，使用Activity名称或者类、模块的名称，不要出现自己的姓名拼音或其他简称。在c++/c代码中调用ALOGD等宏函数，参数没有传入tag,需要在文件头部#define LOG_TAG"YOUR_TAG_NAME"。

注4：Log的内容，不要出现公司名称、个人名称或相关简称，Log内容不要出现无意义的内容，如连续的等号或星号或连续的数字等，Log内容要方便其他分析Log的人员查看。

注5：Log输出的频率需要控制,例如1s打印一次的Log，尽量只在eng版本使用，user版本如需开启，请默认关闭，通过设置setprop命令来开启。



# 三、使用方法

c++、c 层输出log：
#include <cutils/log.h>
ALOGV(),     ALOGD(),    ALOGI(),    ALOGW(),    ALOGE()

Java层输出log：
import android.util.Log
Log.v(String tag, String msg),    Log.d(String tag, String msg),    
Log.i(String tag, String msg),    Log.w(String tag, String msg), Log.e(String tag, String msg)


# 四、log显示格式
log示例：I/SysInfoService(  488): SysInfoWorkThread running
            等级/TAG(PID)       :消息

# 五、过滤
logcat消息过滤
    logcat -c 清除log
    logcat -s TAG:D -v [process只显示pid] [tag 只显示 priority/Tag] [thread 显示process:thread and priority/tag]
                                        [raw只显示message][time显示date-time-priority/Tag PID ][long 将元数据和消息分行]



# 六、Java核心库打印
Java核心库在Android工程的\libcore\luni\src\main\java目录下。
在Android的java核心库中是无法使用Logcat打印的，因为Android的java层要使用Java库，即Java库是Android的Java运行的前提，所以。。。

经过调试，发现其实运用Java的Logger可以很简单的实现(或许Android已经做了处理)，在adb logcat中输出。好了，演示一下。

引入包 
import java.util.logging.Logger;  
static Logger logger = Logger.getLogger("mytag"); 
logger.info("blala");  
然后在adb logcat直接输出就行了。标签是mytag



