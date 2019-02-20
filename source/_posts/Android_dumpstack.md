---
title: Android 打印函数调用栈
date: 2016-05-26 15:13:00
tags: [stack, trace]
categories: Android
---

# 0. ActivityThread

    StackTraceElement st[]= Thread.currentThread().getStackTrace();  
    for(int i=0;i<st.length;i++)  
        System.out.println(i+":"+st[i]);  


# 1. Java层打印函数调用栈的方法
### 方法1:
    StackTraceElement[] stes = new Throwable().getStackTrace();
    if(stes.length >= 1)
    {
      for(int i = 1; i < stes.length; i++)
      {
        Log.d(TAG, "File:" + stes[i].getFileName() + ", Line: " + stes[i].getLineNumber() + ", MethodName:" + stes[i].getMethodName());
      }
      
      for(StackTraceElement elem : stes)
      {
        System.out.println(elem.getClassName() + " " + elem.getMethodName());
      }
    }

### 方法2:
    StackTraceElement[] stes = Thread.getAllStackTraces().get(Thread.currentThread());
    for(StackTraceElement elem : stes)
    {
      System.out.println(elem.getClassName() + " " + elem.getMethodName());
    }

### 方法3:
    Throwable throwable = new Throwable(); Log.w(LOGTAG, Log.getStackTraceString(throwable)); 
    或者
    Log.d(TAG,Log.getStackTraceString(new Throwable()));

### 方法3:
    java.util.Map<Thread, StackTraceElement[]> ts = Thread.getAllStackTraces();
    StackTraceElement[] stes = ts.get(Thread.currentThread());
    for (StackTraceElement elem : stes)
    {
      Log.d(TAG, "SS ", elem.toString());
    }

### 方法4:
    (new Exception()).printStackTrace();
    或
    Exception ex = new Exception("dingran");
    ex.printStackTrace();
    备注:此方法打印出的TAG是在W/System.err(4275)中;

### 方法5:
    RuntimeException here = new RuntimeException("here");
    here.fillInStackTrace();
    Log.w(TAG, "Called: " + this, here);


# 2. Native层打印调用栈方法
### 1、打印C++函数调用栈:
    #include <utils/CallStack.h>
    CallStack stack;
    stack.update();
    stack.dump();
    LOCAL_SHARED_LIBRARIES += libutils

备注:下面操作是可选操作,但加上去之后会有一些额外的功能;
    #define HAVE_DLADDR 1 --> 可以从lib中自己转成C++代码行,不需要手动反编译;
    #define HAVE_CXXABI 1 --> 将C++已被name mangling的函数名转化为源文件中定义的函数名;
并在文件frameworks/base/libs/utils/Android.mk中大约105行(LOCAL_SHARED_LIBRARIES)后添加
    ifeq ($(TARGET_OS),linux)
    LOCAL_SHARED_LIBRARIES += libdl
    endif
重新编译push生成的libutils.so到/system/lib/目录下,重启设备;
此外,由于CallStack.dump中使用的LOGD进行的打印,因此需要将后台的Log Level设置为D一下才能出来;

### 2、打印C函数调用栈:
可以参考CallStack.cpp的实现,通过调用_Unwind_Backtrace完成;


# 3. Kernel层打印调用栈方法
使用函数dump_stack()来打印出函数之间的调用关系;
另外,使用宏BUG()、BUG_ON(xxxx)、WARN()、WARN_ON()、BUILD_BUG_ON()和函数panic("foo is %ld\n",foo)也可以在内核中打印调试信息;
其中,宏WARN_ON()最终会调用函数dump_stack();



