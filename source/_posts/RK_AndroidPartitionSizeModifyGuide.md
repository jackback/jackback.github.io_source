---
title: RK平台Android分区大小修改说明
date: 2017-04-11 09:01:57
tags: [分区, RK]
categories: Android
---


不同于一般的Android系统（一般Android系统分区信息在BoardConfig.mk BoardConfigCommon.mk中），RK平台的分区信息文件在自带的烧录工具包中的parameter文件中

<!-- more -->

###### 如下是板子的parameter文件:
	FIRMWARE_VER: 5.1.0
	MACHINE_MODEL: Geekbox
	MACHINE_ID: 007
	MANUFACTURER: RK3368
	MAGIC: 0x5041524B
	ATAG: 0x00200800
	MACHINE: 3368
	CHECK_MASK: 0x80
	PWR_HLD: 0,0,A,0,1
	#KERNEL_IMG: 0x00280000
	#FDT_NAME: rk-kernel.dtb
	#RECOVER_KEY: 1,1,0,20,0
	#in section; per section 512(0x200) bytes
	CMDLINE: console=ttyS2 androidboot.baseband=N/A androidboot.selinux=permissive androidboot.hardware=rk30board androidboot.console=ttyS2 init=/init mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00002000@0x00004000(trust),0x00002000@0x00006000(misc),0x00008000@0x00008000(resource),0x00008000@0x00010000(kernel),0x00010000@0x00018000(boot),0x00010000@0x00028000(recovery),0x00038000@0x00038000(backup),0x00040000@0x00070000(cache),0x00002000@0x000B0000(kpanic),0x00200000@0x000B2000(system),0x00008000@0x002B2000(metadata),0x00002000@0x002BA000(baseparamer),0x00400000@0x002BC000(userdata),0x00020000@0x006BC000(radical_update),-@0x006DC000(user)


分区信息在mtdparts=rk29xxnand:字段后面部分，所以我们修改这部分内容就可以了
###### 这部分信息格式介绍 如：
	0x00002000@0x00002000(uboot)  分区大小计算为=0x00002000*512Bytes = 4MBytes
	↑分区的sector数，每个sector=512Bytes
	           ↑分区起始地址
	                      ↑分区名

    0x00200000@0x002BC000(userdata)  分区大小0x00200000*512Bytes = 1073741824 Bytes

    注: sector 512Bytes 逻辑单位???
        page: FLASH操作的基本单位。  最小读/写单元 page， 1page = 4096Bytes = 4KB；
        block:最小擦除单元 block，1block = (32 * page) = (32 * 4KB) = 131072Bytes = 128KB；     
        

上述mtdparts=rk29xxnand:字段中所有分区都是连续的，当前分区起始地址+当前分区size和下个分区起始地址都是连续的，所以当修改了其中某个分区后，在它后面的分区的地址都要顺序往前或往后。
###### 以下是我将userdata分区缩小为1GB，将user分区扩大1GB的例子：
	修改前:                            size             offset
    0x00002000@0x00000000(parameter)//4MB               4MB
	0x00002000@0x00002000(uboot),   //4MB               8MB
	0x00002000@0x00004000(trust),   //4MB               12MB
	0x00002000@0x00006000(misc),    //4MB               16MB
	0x00008000@0x00008000(resource),//16MB              32MB
	0x00008000@0x00010000(kernel),  //16MB              48
	0x00010000@0x00018000(boot),    //32MB              80
	0x00010000@0x00028000(recovery),//32MB              112
	0x00038000@0x00038000(backup),  //112MB             224
	0x00040000@0x00070000(cache),   //128MB             352
	0x00002000@0x000B0000(kpanic),  //4MB               356
	0x00200000@0x000B2000(system),  //1024MB            1380
	0x00008000@0x002B2000(metadata),//16MB              1396
	0x00002000@0x002BA000(baseparamer), //4MB           1400MB
	0x00400000@0x002BC000(userdata),    //2048MB        3448
	0x00020000@0x006BC000(radical_update), //64MB       3512
	         -@0x006DC000(user)            //212MB/11394MB  ...        
	
	修改后:
	0x00002000@0x00002000(uboot),   //4MB               8MB
	0x00002000@0x00004000(trust),   //4MB               12MB
	0x00002000@0x00006000(misc),    //4MB               16MB
	0x00008000@0x00008000(resource),//16MB              32MB
	0x00008000@0x00010000(kernel),  //16MB              48
	0x00010000@0x00018000(boot),    //32MB              80
	0x00010000@0x00028000(recovery),//32MB              112
	0x00038000@0x00038000(backup),  //112MB             224
	0x00040000@0x00070000(cache),   //128MB             352
	0x00002000@0x000B0000(kpanic),  //4MB               356
	0x00200000@0x000B2000(system),  //1024MB            1380
	0x00008000@0x002B2000(metadata),//16MB              1396
	0x00002000@0x002BA000(baseparamer), //4MB           1400MB
	0x00200000@0x002BC000(userdata),    //1024MB        2424MB
	0x00020000@0x004BC000(radical_update), //64MB       2488MB
	         -@0x004DC000(user)            //1024+212MB = 1236MB   

##### 注：
	1.分区的大小要以4MB对齐！  
	2.分区总大小不能超过flash的总长度



##### 修改parameter后效果：
	shell@rk3368_32:/ # df
	Filesystem               Size     Used     Free   Blksize
	/dev                   495.2M    36.0K   495.2M   4096
	/sys/fs/cgroup         495.2M    12.0K   495.2M   4096
	/mnt/asec              495.2M     0.0K   495.2M   4096
	/mnt/obb               495.2M     0.0K   495.2M   4096
	/system               1004.2M   403.0M   601.2M   4096
	/cache                 122.0M   140.0K   121.8M   4096
	/metadata               11.7M    40.0K    11.7M   4096
	/data                  991.9M    87.5M   904.4M   4096
	/mnt/shell/emulated    991.9M    87.5M   904.4M   4096
	/mnt/internal_sd         1.2G   136.0K     1.2G   8192
	/mnt/secure/asec         1.2G   136.0K     1.2G   8192



### 一般Android平台分区修改
	eMMC 分区信息
	# cat /proc/partitions                                       
	major      minor  #blocks  name
	 254        0     520912   zram0
	 179        0   15267840   mmcblk0
	 179        1       4096   mmcblk0p1
	 179        2       4096   mmcblk0p2
	 179        3       4096   mmcblk0p3
	 179        4      16384   mmcblk0p4
	 179        5      16384   mmcblk0p5
	 179        6      32768   mmcblk0p6
	 179        7      32768   mmcblk0p7
	 179        8     114688   mmcblk0p8
	 179        9     131072   mmcblk0p9
	 179       10       4096   mmcblk0p10
	 179       11    1048576   mmcblk0p11
	 179       12      16384   mmcblk0p12
	 179       13       4096   mmcblk0p13
	 179       14    2097152   mmcblk0p14
	 179       15      65536   mmcblk0p15
	 179       16   11667456   mmcblk0p16
	mmcblk0即emmc的容量（单位kb）

	#ls /dev/block/platform/ff0f0000.rksdmmc/by-name/* -l 
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


	即userdata 分区 大小为2097152*1024KB = 2147483648B

	BoardConfig.mk 中修改指定变量即可：
	BOARD_BOOTIMAGE_PARTITION_SIZE := 16777216
	BOARD_RECOVERYIMAGE_PARTITION_SIZE := 16793600
	BOARD_SYSTEMIMAGE_PARTITION_SIZE := 2147483648
	BOARD_OEMIMAGE_PARTITION_SIZE := 67108864
	BOARD_USERDATAIMAGE_PARTITION_SIZE := 25253773312 ---> 2147483648B
	BOARD_CACHEIMAGE_PARTITION_SIZE := 268435456
	BOARD_CACHEIMAGE_FILE_SYSTEM_TYPE := ext4
	BOARD_FLASH_BLOCK_SIZE := 131072

