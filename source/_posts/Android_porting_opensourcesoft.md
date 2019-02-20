---
title: 移植开源程序到Android中
date: 2017-04-13 12:22:33
tags: [porting]
categories: Android
---

1. 安装独立的交叉编译链：
    下载ndk 包， 解压
    cd build/tools
    ./make-standalone-toolchain.sh --install-dir=/opt/ndk-standalone-10-arm --platform=android-22 --toolchain=arm-linux-androideabi-4.8

    注1  老的ndk才需要使用--ndk-dir=../../，    ndk10使用  --platform=android-L，    ndk13使用  --platform=android-22
    注2：需要将/opt/ndk-standalone-10-arm/bin 添加到PATH 路径中

2. 编译libusb-1.0.so库
    需要略微添加下timeval 等结构体 等

    #define TIMESPEC_TO_TIMEVAL(tv, ts)         \         
    {                                           \
        (tv)->tv_sec = (ts)->tv_sec;            \
        (tv)->tv_usec = (ts)->tv_nsec / 1000;   \
    } while (0)	 


    方法1 使用原始编译框架，交叉编译
        ./configure --build=x86_64-linux-gnu  --host=arm-linux-androideabi --target=arm-linux-androideabi --prefix=/opt/libusb-1.0.9/
        make
        注：--prefix 编译生成库的路径，也叫安装路径，使用绝对路径
    方法2 使用Android ndk ndk-build进行编译（需要编译Application.mk Android.mk 等文件）    【推荐】
        ndk-build clean 
        ndk-build 
    方法3 copy  ti-sdk-am335x-evm-08.00.00.00 中的libusb-1.0.so 库

3. 编译libssl.so libcrypto.so 库 （Android集成的openssl）
    方式1： 使用Android 中openssl 编译  【推荐】
        需要配置openssl.config和openssl.trusty.config  去掉no-dtls1 宏，否则将不支持DTLS特性  （即通过 arm-linux-androideabi-            
            nm ../../out/target/product/rk3368_box/obj/SHARED_LIBRARIES/libssl_intermediates/LINKED/libssl.so | grep DTLSv1_server_method 查询不到函数符号）
        ./import_openssl.sh import <path-to-openssl-1.0.1j.tar.gz>     //重新按照该openssl.config生成编译配置文件
        mm -B

    方式2： （下载开源openssl源码编译， 参见openssl-1.1.0c/Configurations/10-main.conf 介绍，此方法用户编译原生openssl库）
        export ANDROID_NDK=/opt/android-ndk-r13b
        export CROSS_SYSROOT=/opt/android-ndk-r13b/platforms/android-22/arch-arm
        export CROSS_COMPILE=arm-linux-androideabi-                   //等同于选项--cross-compile-prefix

        ./Configure android-armeabi  --openssldir=/opt/openssl-1.1.0c/out/config --prefix=/opt/openssl-1.1.0c/out
        make install

        make clean && make distclean
        export ANDROID_NDK=/opt/android-ndk-r10b
        export CROSS_SYSROOT=$ANDROID_NDK/platforms/android-L/arch-arm
        export CROSS_COMPILE=arm-linux-androideabi-
        export OPENSSL_HOME=/work2/mygo/src/bitbucket.org/portus/smartgateway/NodePro/Zwave/lib/openssl
        ./Configure android-armeabi no-threads --openssldir=$OPENSSL_HOME/out/config --prefix=$OPENSSL_HOME/out
        make
        make install



4. bridge-utils （主要是需要brctl工具）
    ./configure --build=x86_64-linux-gnu  --host=arm-linux-androideabi --target=arm-linux-androideabi  --prefix=/opt/bridge-utils-1.4/out/
    make clean
    make
    make install

5. 编译zipgateway （基于arm Android目标运行环境）
    ./configure --build=x86_64-linux-gnu  --host=arm-linux-androideabi --target=arm-linux-androideabi 
    make CC=/opt/ndk-standalone-10-arm/bin/arm-linux-androideabi-gcc beaglebone 
    或者  make CROSS="arm-linux-androideabi-" beaglebone   【推荐】


### 其他

android ndk编译开源linux软件方法
NDK=/opt/android-ndk-r10b
SYSROOT=$NDK/platforms/android-L/arch-arm
CC="$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin/arm-linux-androideabi-gcc --sysroot=$SYSROOT"
$CC --version


cmake -DOPENSSL_ROOT_DIR=/work/project/geekbox/Lollipop/external/openssl/include -DOPENSSL_LIBRARIES=/work/project/geekbox/Lollipop/out/target/product/rk3368_box/system/lib
./configure 









