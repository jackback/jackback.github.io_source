---
title: Android fstab介绍
date: 2017-04-07 16:01:57
tags: fstab
categories: Android
---

## ubuntu fstab文件格式
	下面是/etc/fatab文件的一个示例行：
	fs_spec     fs_file fs_type     fs_options  fs_dump    fs_pass
	/dev/hda1   /       ext2        defaults    1          1    

	fs_spec - 该字段定义希望加载的文件系统所在的设备或远程文件系统，对于一般的本地块设备情况来说：IDE设备一般描述为
              /dev/hdaXN，X是IDE设备通道 (a, b, or c)，N代表分区号；SCSI设备一描述为/dev/sdaXN。对于NFS情况，格式一般为:
	         例 如：`knuth.aeb.nl:/'。对于procfs，使用`proc'来定义。
	fs_file - 该字段描述希望的文件系统加载的目录点，对于swap设备，该字段为none；对于加载目录名包含空格的情况，用40来
              表示空格。
	fs_type - 定义了该设备上的文件系统，一般常见的文件类型为ext2 (linux设备的常用文件类型)、vfat(Windows系统的fat32格
              式)、NTFS、iso9600等。
	fs_options - 指定加载该设备的文件系统是需要使用的特定参数选项，多个参数是由逗号分隔开来。对于大多数系统使用"defaults"
                 就可以满足需要。其他常见的选项包括：
				选项含义:
				ro      以只读模式加载该文件系统
				sync    不对该设备的写操作进行缓冲处理，这可以防止在非正常关机时情况下破坏文件系统，但是却降低了计算机速度
				user    允许普通用户加载该文件系统
				quota   强制在该文件系统上进行磁盘定额限制
				noauto  不再使用mount －a命令（例如系统启动时）加载该文件系统

	fs_dump 该选项被"dump"命令使用来检查一个文件系统应该以多快频率进行转储，若不需要转储就设置该字段为0
	fs_pass 该字段被fsck命令用来决定在启动时需要被扫描的文件系统的顺序，根文件系统"/"对应该字段的值应该为1，
		            	其他文件系统应该为2。若该文件系统无需在启动时扫描则设置该字段为0

<!-- more -->

##### ubuntu上实例:
	<file system>                             <mount point>   <type>  <options>      <dump>  <pass>
	/ was on /dev/sda1 during installation
	UUID=757fbb2f-6ee4-4a05-ad2e-0c16b3edc982 /               ext4    errors=remount-ro 0     1
	/home was on /dev/sda6 during installation
	UUID=a018cd99-6608-43fc-adea-319a5f04fb29 /home           ext4    defaults        0       2
	/opt was on /dev/sda7 during installation
	UUID=0351ac71-4e1d-4194-8d7f-4d9e873e5830 /opt            ext4    defaults        0       2
	/work was on /dev/sda8 during installation
	UUID=3f49c9cc-c9cc-48c5-aa8e-058a1d1ec7ad /work           ext4    defaults        0       2
	/work2 was on /dev/sda9 during installation
	UUID=e2f54160-fb2d-4517-af54-13393f80ef5f /work2          ext4    defaults        0       2
	swap was on /dev/sda5 during installation
	UUID=d15cbce2-bff1-4241-9c3c-6811f4a1d67d none            swap    sw              0       0

## Android fstab格式及实例
	https://source.android.com/devices/storage/config.html
	<src>                                                 <mnt_point>          <type>    <mnt_flags>     <fs_mgr_flags>
	/devices/ff0f0000.rksdmmc/mmc_host/mmc                /mnt/internal_sd     vfat      defaults        voldmanaged=internal_sd:15,noemulatedsd
	/dev/block/platform/ff0f0000.rksdmmc/by-name/system   /system              ext4      ro,noatime,nodiratime,noauto_da_alloc     wait,resize,check
	
	src: 设备路径
	mnt_point: 挂在点
	type: volume中文件系统类型, 对于external cards, type 为vfat
	mnt_flags: Vold ignores this field and it should be set to defaults
	fs_mgr_flags: Vold ignores any lines in the unified fstab that do not include the voldmanaged= flag in this field. 
			   This flag must be followed by a label describing the card, and a partition number or the word auto.
			   Here is an example: voldmanaged=sdcard:auto. 
			   Other possible flags are nonremovable, encryptable=sdcard, noemulatedsd, and encryptable=userdata.
	
	
	待续...
