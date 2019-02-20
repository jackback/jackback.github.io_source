---
title: Android init.rc
date: 2016-11-25 12:22:33
tags: [init.rc]
categories: Android
---



init.rc主要有两大元素:   Actions和Service

一种是定时任务——Actions，由以下构成：
    on <trigger>
        <command>
        <command>
        ...

一种是固定顺序的任务(初始化启动，可以重启)，即Services 
    service <name> <binaray path name> [ <argument> ]*
        <option>
        <option>
        ...


<!-- more -->


以下是<option>分别详解
critical        //标识设备核心服务，如果在四分钟内，退出超过四次，设备进入recovery模式
disabled    // 和class 一起使用时，也不会随着class一起启动，必须自己显式的用服务名启动 。在init.rc的条件中触发或者在代码中通过ctl.start来触发
                  // 如果该服务选项中没有disabled定义，则在init.rc中解析到这个服务的时候，会马上执行这个服务。而如果在服务的选项中增加了disabled定义，
                  // 则该服务不会在init.rc中启动，需要我们在代码中使用以下方法来启动：property_set("ctl.start", "wifi_dhcpcd");上句话的意思是，
                  // 我要启动wifi_dhcpd这个服务了。

setenv <name> <value>  //启动该服务时，设置环境变量 name 为value
socket   ...    //建立socket
user <username>  //启动该服务前，切换到username 用户
group <groupname> [ <groupname> ]*   //启动该服务前，切换到组
seclabel <securitycontext>    //启动前切换到安全上下文
oneshot      //服务存在是不重启
class <name>   //指定一个class名，所有指定相同class名的服务将可能被一起启动或停止
onrestart    //启动时执行一条命令(和Actions中的<commman>可选项一样)，可指定的命令如下：
    


以下是Triggers分别详解
<triggers>   //是一个字符串，被用于匹配不同种类的事件，用于触发一个动作发生
 有如下trigger：
        boot  //这是第一个在init启动时发生的trigger
        <name>=<value>  //这种trigger表示，在name=value时触发。

        device-added-<path>
        device-removed-<path>  //这种trigger表示当设备节点被添加或者被移除时触发
        service-exited-<name>   //这种trigger表示当指定服务<name>退出时触发


以下是Command分别详解

exec <path> [ <argument> ]*    //fork一个进程，执行program，避免执行内置命令，这可能会导致init卡主
export <name> <value>    // 设置一个全局变量， 这个全局变量将会被所有后面启动的进程继承
ifup <interface>                  //bringup 网口interface
import <filename>              //解析一个类似init.rc的配置文件，作为拓展
hostname <name>              //设置hostname
chdir <directory>                 //切换工作目录
chmod <octal-mode> <path>   //改变模式，八进制
chown <owner> <group> <path>   
chroot <directory>   //改变进程运行的root目录
class_start <serviceclass>   如果<serviceclass>没运行的话，运行<serviceclass>
class_stop <serviceclass>   如果<serviceclass>在运行的话，停止<serviceclass>
domainname <name>      设置域名
enable <servicename>      如果该服务没有被指定禁用的话，启动这个未启动的服务。
insmod <path>   //加载ko模块
mkdir <path> [mode] [owner] [group]   
mount <type> <device> <dir> [ <mountoption> ]*    
restorecon <path> [ <path> ]*   
restorecon_recursive <path> [ <path> ]*        
setcon <securitycontext>
setenforce 0|1      
setprop <name> <value>  
setrlimit <resource> <cur> <max>   //Set the rlimit for a resource.
setsebool <name> <value>    //Set SELinux boolean <name> to <value>.       <value> may be 1|true|on or 0|false|off
start <service>   //启动一个未启动的服务
stop <service>    //停止一个启动的服务
symlink <target> <path>   //创建一个符号链接
sysclktz <mins_west_of_gmt>    //
trigger <event>    //触发另外一个时间，用于动作同步。
wait <path> [ <timeout> ]    //轮训是否该文件存在，如果存在，return，或者timeout时间达到时，也退出，未指定timeout，默认等5s
write <path> <string>   //写字符串到文件中。不是追加





系统属性
init.action //=当前执行的动作的action名，没有action名，即为空
init.command    //=当前执行的命令的action名，没有command名，即为空
init.svc.<name>   //=服务名为<name>的服务状态，可选值为("stopped", "running", "restarting")



调试notes：
默认，被init进程执行的子进程 stdout和stderr都会进入/dev/null中，如果你需要调试你的程序的话，你可以
使用Android logwrapper，它将会重定向stdout/stderr 到Android log 系统（你可以通过logcat导出）
用法：service zip /system/bin/logwrapper /system/bin/zipgateway





