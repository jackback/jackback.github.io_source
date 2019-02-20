---
title: Android JNI介绍及使用
date: 2016-10-08 12:22:33
tags: [JNI, native调用]
categories: Android
---

# JNI 介绍 #
JNI是Java Native Interface的缩写，它提供了若干的API实现了Java和其他语言的通信（主要是C&C++）。


# JNI 类型 #

[![001_JavaJNIType.png](https://i.loli.net/2018/12/15/5c14f2f87c115.png)](https://i.loli.net/2018/12/15/5c14f2f87c115.png)

jni中结构体层次结构

[![002_JaveJNIType2.png](https://i.loli.net/2018/12/15/5c14f2f8833c4.png)](https://i.loli.net/2018/12/15/5c14f2f8833c4.png)


下面是访问String的一些方法：

    GetStringUTFChars 将jstring转换成为UTF-8格式的char*    /  ReleaseStringUTFChars  释放指向UTF-8格式的char*的指针
    GetStringChars    将jstring转换成为Unicode格式的char*  /  ReleaseStringChars     释放指向Unicode格式的char*的指针
    
    NewStringUTF      创建一个UTF-8格式的String对象
    NewString         创建一个Unicode格式的String对象
    GetStringUTFLengt 获取 UTF-8格式的char*的长度　
    GetStringLength   获取Unicode格式的char*的长度

<!-- more -->

# JNI 实现 #

如果需要在Java中使用到native接口的话，可以使用JNI这个方式来达到目的：

##### 1. Java侧使用方法
    public class AudioFlingerBinderTest extends TestCase {                                                                                                                                                        
        private static native boolean native_test_setMasterMute();     //声明native函数，都是以native_开头                                                                                                                                                             
        static {                                                                                                                                                        
            System.loadLibrary("ctssecurity_jni");   //加载jni库，该函数会找到对应的动态库，然后首先试图找到 
                                                     // "JNI_OnLoad"函数，如果该函数存在，则调用它。             
        }                                                                                                                                                                  
        
        //使用native_test_setMasterMute()                                                                                                          
        .....                                                                                                                                                               
    }                                                                                                                                                

##### 2. Java侧Android.mk编写    
    Android.mk
    LOCAL_PATH:= $(call my-dir)
    
    include $(CLEAR_VARS)
    ...
    LOCAL_JNI_SHARED_LIBRARIES := libctssecurity_jni libcts_jni
    ...


##### 3. JNI侧库ctssecurity_jni的Android.mk
    Android.mk------------------------------
    LOCAL_PATH:= $(call my-dir)
    
    include $(CLEAR_VARS)
    
    LOCAL_MODULE := libctssecurity_jni
    LOCAL_MODULE_TAGS := optional
    LOCAL_SRC_FILES := \
    		CtsSecurityJniOnLoad.cpp \
    		android_security_cts_CharDeviceTest.cpp \
                    .....
    LOCAL_C_INCLUDES := $(JNI_H_INCLUDE)
    LOCAL_SHARED_LIBRARIES := libnativehelper liblog libbinder libutils libmedia libselinux libdl
    
    include $(BUILD_SHARED_LIBRARY)


##### 4. JNI侧库ctssecurity_jni

###### 4.1 实现native函数

    jboolean android_security_cts_AudioFlinger_test_setMasterMute(JNIEnv* env __unused, jobject thiz __unused) {
        //具体实现
    }

or

    jboolean android_security_cts_AudioFlinger_test_setMasterMute(JNIEnv* env, jclass clazz) {
        //具体实现
    }

###### 4.2 注册native函数
    static JNINativeMethod gMethods[] = {
        { "native_test_setMasterMute", "()Z",
                (void *) android_security_cts_AudioFlinger_test_setMasterMute },
    };


    int register_android_security_cts_AudioFlingerBinderTest(JNIEnv* env) {
        jclass clazz = env->FindClass("android/security/cts/AudioFlingerBinderTest");
        return env->RegisterNatives(clazz, gMethods, sizeof(gMethods) / sizeof(JNINativeMethod));
    }

or

    int register_android_security_cts_AudioFlingerBinderTest(JNIEnv* env) {
        int res = jniRegisterNativeMethods(env, "android/security/cts/AudioFlingerBinderTest",
                gMethods, NELEM(gMethods));
        LOG_FATAL_IF(res < 0, "Unable to register native methods.");
        return res;
    }


###### 4.3 jni库的主函数/加载入口：
    #include <jni.h>
    #include <stdio.h>
    #include “xxx.h”
    
    jint JNI_OnLoad(JavaVM *vm, void *reserved) {
        JNIEnv *env = NULL;
    
        if (vm->GetEnv((void **) &env, JNI_VERSION_1_4) != JNI_OK) {
            return JNI_ERR;
        }
        if (register_android_security_cts_AudioFlingerBinderTest(env)) {
            return JNI_ERR;
        }
    
        return JNI_VERSION_1_4;
    }

