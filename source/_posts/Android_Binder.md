---
title: Android Binder
date: 2017-04-27 10:37:33
tags: [Binder, IPC, RPC]
categories: Android
---

# Binder 通信机制概述

Binder通信是一种类似于IPC 的机制，基于C/S通信结构，它是给servicemanager-server、server-client之间通信的工具，下面简单介绍具体实现原理：

1. 从表面上来看，是client通过获得一个server的代理接口，对server进行直接调用；
   实际上，代理接口中定义的方法与server中定义的方法是一一对应的。
2. client调用某个代理接口中的方法时，代理接口的方法会将client传递的参数打包成为Parcel对象；
   代理接口将该Parcel发送给内核中的binder driver。
   server会读取binder driver中的请求数据，如果是发送给自己的，解包Parcel对象，处理并将结果返回；
3. 整个的调用过程是一个同步过程，在server处理的时候，client会block住。

[![Binder_ProcessCall.png](https://i.loli.net/2018/12/15/5c14f2eb7a225.png)](https://i.loli.net/2018/12/15/5c14f2eb7a225.png)



<!-- more -->

# Binder 通信机制实现原理
Binder 通信机制实现IPC的方式和我们所了解的消息队列、信号量、管道等不同，他是通过Binder设备（一个实现数据交换的虚拟设备）来实现数据交换，
一次信息传递过程只发生一次拷贝，数据交换非常高效。Binder通信机制中有三个角色，service manager(简称SM)、service、client，它们在Android
系统都是有各自的进程在系统中运行，每个进程都与Binder设备建立了信息通信通道(通过open /dev/binder),通过binder设备实现SM、service、client
之间的信息交换。

底层实现机制：
SM          ---- handle 0
service1    ---- handle s1
service2    ---- handle s2
...
serviceN    ---- handle sN

client1    ---- handle c1
client2    ---- handle c2
...
clientN    ---- handle cN

SM      作为service manager，它负责管理service(说白了就是：响应service发送给它的消息)，service添加/service句柄的获取/serviceList
service 负责处理client的RPC请求（也就是响应client发送过来的消息）
client  就是主动发送请求一方。

binder机制中交换数据(发送消息)，例如service发送消息给SM，它是将信息通过binder驱动直接挂接到SM 关联的handle下，然后SM直接去读自己handle=0
下的消息并进行解析就可以了。从而完成一次数据交换。



Android系统中有如下几种场景：
1. 进程添加service 到SM中。
    a. 通过SM handle(默认固定为0)，发送消息给SM；
    b. SM进程的loop循环获取到发送给自己handle的消息并获取到service注册信息；
    c. SM将service信息注册到底层binder设备，binder设备给该service分派一个handle sX(关联service name)，功能就算完成。

2. client 使用service服务。（这其中包含两次binder）
    A. client 获取指定service句柄
        a. client所在进程发送消息给SM(handle固定为0)，即写入parcel信息到binder设备，挂接到SM的handle名下。
        b. SM 进程loop循环读取到发给自己信息，解析后，返回对应service name的service handle信息给client。
    B. client 使用service句柄使用service提供的接口
        a. client所在进程发送接口调用信息给（service handle），即写入parcel信息到binder设备，挂接到service的handle下；
        b. service 的IPCProcess joinThreadPool中循环获取到发送给自己的消息，处理并以parcel格式返回给binder设备中对应handle的client；
        c. 然后client调用过程完成。



# Binder 通信机制在Android中的实现

场景1实现：
servicemanager (frameworks\native\cmds\servicemanager\service_manager.c)
1. 打开/dev/binder设备，将fd映射到128k的内存
2. 将servermanger设置为binder设备的context manager(可以理解为ServiceManager可以将自己注册为Binder的管理员)
3. 循环读取发送给handle 0 的消息

Bn端循环响应server/client过来的请求
[![service_manager.png](https://i.loli.net/2018/12/15/5c14f2ebd3c9a.png)](https://i.loli.net/2018/12/15/5c14f2ebd3c9a.png)

C++中的Bp端（跑在使用SM的client或者server 的进程中的）
[![IServiceManager.png](https://i.loli.net/2018/12/15/5c14f2ebd27ed.png)](https://i.loli.net/2018/12/15/5c14f2ebd27ed.png)

Java中的Bp端（跑在使用SM的client或者server 的进程中的）
[![IServiceManager_Java.png](https://i.loli.net/2018/12/15/5c14f2ebd08e4.png)](https://i.loli.net/2018/12/15/5c14f2ebd08e4.png)


场景2实现：（以MediaPlayerService为例）
[![MediaPlayerService.png](https://i.loli.net/2018/12/15/5c14f2ec2e84f.png)](https://i.loli.net/2018/12/15/5c14f2ec2e84f.png)

















