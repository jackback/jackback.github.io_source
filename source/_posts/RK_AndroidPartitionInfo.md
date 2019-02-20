---
title: RK平台分区信息
date: 2017-04-11 17:35:57
tags: [RK, 分区]
categories: RK
---


RK 分区信息和一般Android 系统分区有些许区别，下面我们来看看

<!-- more -->

## RK平台分区信息与eMMC partitions对应
	lrwxrwxrwx root     root    2013-01-21 08:50 uboot            -> /dev/block/mmcblk0p1
	lrwxrwxrwx root     root    2013-01-21 08:50 trust            -> /dev/block/mmcblk0p2
	lrwxrwxrwx root     root    2013-01-21 08:50 misc             -> /dev/block/mmcblk0p3
	lrwxrwxrwx root     root    2013-01-21 08:50 resource         -> /dev/block/mmcblk0p4
	lrwxrwxrwx root     root    2013-01-21 08:50 kernel           -> /dev/block/mmcblk0p5
	lrwxrwxrwx root     root    2013-01-21 08:50 boot             -> /dev/block/mmcblk0p6
	lrwxrwxrwx root     root    2013-01-21 08:50 recovery         -> /dev/block/mmcblk0p7
	lrwxrwxrwx root     root    2013-01-21 08:50 backup           -> /dev/block/mmcblk0p8
	lrwxrwxrwx root     root    2013-01-21 08:50 cache            -> /dev/block/mmcblk0p9
	lrwxrwxrwx root     root    2013-01-21 08:50 kpanic           -> /dev/block/mmcblk0p10
	lrwxrwxrwx root     root    2013-01-21 08:50 system           -> /dev/block/mmcblk0p11
	lrwxrwxrwx root     root    2013-01-21 08:50 metadata         -> /dev/block/mmcblk0p12
	lrwxrwxrwx root     root    2013-01-21 08:50 baseparamer      -> /dev/block/mmcblk0p13
	lrwxrwxrwx root     root    2013-01-21 08:50 userdata         -> /dev/block/mmcblk0p14
	lrwxrwxrwx root     root    2013-01-21 08:50 radical_update   -> /dev/block/mmcblk0p15
    lrwxrwxrwx root     root    2013-01-21 08:50 user             -> /dev/block/mmcblk0p16

    mmcblk0: p1 p2 p3 p4 p5 p6 p7 p8 p9 p10 p11 p12 p13 p14 p15 p16 p17

    /mnt/external_sd vfat                                            /dev/block/mmcblk1p1

## RK平台各分区介绍
	uboot.img
	trust.img
	misc.img      RK平台，misc分区内容用于启动模式切换，是升级基带还是进入Recovery（misc分区的内容可以被按键出发改变???）
	resource.img  RK平台资源映像，内含开机图片和内核的设备树信息。
	kernel
	boot.img      Android 的初始文件映像，负责初始化并加载 system 分区，一般Android系统这个镜像中包含ramdisk.img根文件系统+kernel（RK平台的boot.img是没包含kernel部分的）
		ramdisk.img   是根文件系统，是root目录打包成的镜像（ramdisk-recovery.img）
	recovery.img  恢复模式系统启动镜像。内容和boot.img雷同（RK平台recovery.img包含kernel部分），但是其中的ramdisk.img（ramdisk-recovery.img）有些许不同：
                  recovery.fstab文件、sbin下部分工具、build.prop、init.bootmode.emmc.rc、sepolicy等
	backup
	cache
	kpanic
	
	system.img    包括了主要的包、库等文件
	metadata
	baseparameter
	userdata.img  包括了一些用户数据
	radical_update
	user


## 另：
	RK平台生成boot.img recovery.img system.img userdata.img是使用自己的mkimage.sh工具讲root system data 目录生成的
	再使用工具mkupdate.sh生成update.img的（不带userdata.img）
	烧写userdata.img 需要使用./upgrade_tool DI userdata userdata.img 单独烧写


