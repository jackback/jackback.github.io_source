---
title: Input系统——Android Input系统
date: 2017-04-12 12:22:33
tags: [input, input系统, event]
categories: Android
---

# Input系统结构解析

AndroidInput系统——JNI NativeInputManager InputManger InputReader 
![AndroidInput系统——NativeInputManager JNI](https://drive.google.com/open?id=1DoS3SayZNpAqAMpjQ8h0O1eSb1nb9r9V7c47TjvqO2o)

<!-- more -->

AndroidInput系统——InputReader 
![AndroidInput系统——InputReader ](https://drive.google.com/open?id=1A78eIrYTHg2nh-HrWJMpsMu64HitGj7dk_ciSajtex4)

AndroidInput系统——InputDispatcher 
![AndroidInput系统——InputDispatcher ](https://drive.google.com/open?id=19KStX_1m2rNCqjCouwW5DFWOY-e1ZbfDTcYAXrPSzOA)

AndroidInput系统——EventHub 
![AndroidInput系统——EventHub ](https://drive.google.com/open?id=1QAC0GFiARVOJcv6vugVNEiqIMkGZX0XXhiCuIDssvtk)



# Android Input系统相关结构体

### 1. input_device_id
    struct input_device_id {
     kernel_ulong_t flags;
     __u16 bustype;  // BUS_USB  BUS_BLUETOOTH  
     __u16 vendor;
     __u16 product;
     __u16 version;
     kernel_ulong_t evbit[INPUT_DEVICE_ID_EV_MAX / BITS_PER_LONG + 1];

     kernel_ulong_t keybit[INPUT_DEVICE_ID_KEY_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t relbit[INPUT_DEVICE_ID_REL_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t absbit[INPUT_DEVICE_ID_ABS_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t mscbit[INPUT_DEVICE_ID_MSC_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t ledbit[INPUT_DEVICE_ID_LED_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t sndbit[INPUT_DEVICE_ID_SND_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t ffbit[INPUT_DEVICE_ID_FF_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t swbit[INPUT_DEVICE_ID_SW_MAX / BITS_PER_LONG + 1];
     kernel_ulong_t driver_info;
    };


### 2. input_event (input.h)
    struct input_event {
        struct timeval time;
        __u16 type;
        __u16 code;
        __s32 value;
    };

### 3. RawEvent (EventHub.h)
    struct RawEvent {
        nsecs_t when;       time
        int32_t deviceId;   ID    = 1       ID         = 0
        int32_t type;       EV_REL= 0x02    EV_SYN     = 0x00
        int32_t code;       REL_X = 0x00    SYN_REPORT = 0
        int32_t value;      Value = 0x01    Value      = 0x00
    };







# 调试相关

### 查看event注册设备
    shell@rk3368_32:/dev/input # cat /proc/bus/input/devices
    I: Bus=0000 Vendor=0000 Product=0003 Version=206a
    N: Name="synaptics_dsx"
    P: Phys=synaptics_dsx/touch_input
    S: Sysfs=/devices/ff140000.i2c/i2c-2/2-0020/input/input0
    U: Uniq=
    H: Handlers=event0 cpufreq ddr_freq 
    B: PROP=2
    B: EV=b
    B: KEY=420 0 0 0 0 0 8000 0 0 0 0
    B: ABS=2638000 3
    
    I: Bus=0019 Vendor=0000 Product=0000 Version=0000
    N: Name="rotary.35"
    P: Phys=
    S: Sysfs=/devices/rotary.35/input/input1
    U: Uniq=
    H: Handlers=event1 
    B: PROP=0
    B: EV=5
    B: REL=1
    
    I: Bus=0019 Vendor=0001 Product=0001 Version=0100
    N: Name="rk29-keypad"
    P: Phys=gpio-keys/input0
    S: Sysfs=/devices/ff100000.adc/key.38/input/input2
    U: Uniq=
    H: Handlers=event2 ddr_freq keychord 
    B: PROP=0
    B: EV=3
    B: KEY=8000 100000 0 0 0
    
    I: Bus=0000 Vendor=0000 Product=0000 Version=0000
    N: Name="temperature"
    P: Phys=
    S: Sysfs=/devices/ff160000.i2c/i2c-4/4-005b/input/input3
    U: Uniq=
    H: Handlers=event3 
    B: PROP=0
    B: EV=9
    B: ABS=40

### 查看Handler个数
    shell@rk3368_box:/dev/input # ls -l                                            
    crw-rw---- root     input     13,  64 2017-04-12 11:15 event0
    crw-rw---- root     input     13,  65 2017-04-12 11:15 event1
    crw-rw---- root     input     13,  66 2017-04-14 08:12 event2
    crw-rw---- root     input     13,  67 2017-04-14 08:12 event3
    
    cat event0 
    cat event1

### getevent 获取事件
    # getevent -h
    Usage: getevent [-t] [-n] [-s switchmask] [-S] [-v [mask]] [-d] [-p] [-i] [-l] [-q] [-c count] [-r] [device]
        -t: show time stamps
        -n: don't print newlines
        -s: print switch states for given bits
        -S: print all switch states
        -v: verbosity mask (errs=1, dev=2, name=4, info=8, vers=16, pos. events=32, props=64)
        -d: show HID descriptor, if available
        -p: show possible events (errs, dev, name, pos. events)
        -i: show all device info and possible events
        -l: label event types and names in plain text
        -q: quiet (clear verbosity mask)
        -c: print given number of events then exit
        -r: print rate events are received
    
    常用命令组合：
        getevent -p    // see all of the keys and axes a device reports
        getevent -ip   // get more information, including HID mapping tables and debugging information
        getevent -lp   // option to display textual labels for all event codes
        getevent -lp
        getevent -r -q    监控设备的sendevent事件 
                

    getevent -p             
    add device 1: /dev/input/event3
      name:     "temperature"
    add device 2: /dev/input/event1
      name:     "rotary.35"
    add device 3: /dev/input/event0
      name:     "synaptics_dsx"
    add device 4: /dev/input/event2
      name:     "rk29-keypad"

    /dev/input/event1: 0002 0000 00000001   ID=1    EV_REL=0x02   REL_X = 0x00    Value = 0x01
    /dev/input/event1: 0000 0000 00000000   ID=0    EV_SYN 0x00   SYN_REPORT = 0  Value = 0x00
    /dev/input/event1: 0002 0000 00000001       
    /dev/input/event1: 0000 0000 00000000       
    /dev/input/event1: 0002 0000 00000001
    /dev/input/event1: 0000 0000 00000000
    /dev/input/event1: 0002 0000 ffffffff       EV_REL=0x02   REL_X = 0x00    Value = 0xffffffffff
    /dev/input/event1: 0000 0000 00000000
    /dev/input/event1: 0002 0000 ffffffff



### dumpsys
  dumpsys input：  
    To dump the input system’s state (Event Hub State,Input Reader State,Input Dispatcher State)
