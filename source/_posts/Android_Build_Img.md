---
title: Android编译系统——ramdisk.img system.img userdata.img recovery.img目标编译过程
date: 2017-04-11 17:35:57
tags: [Android, 编译, image]
categories: Android
---


## image 介绍
	boot.img      Android 的初始文件映像，负责初始化并加载 system 分区，一般Android系统这个镜像中包含ramdisk.img根文件系统+kernel（RK平台的boot.img是没包含kernel部分的）
	recovery.img  恢复模式系统启动镜像。内容和boot.img雷同（RK平台recovery.img包含kernel部分），但是其中的ramdisk.img（ramdisk-recovery.img）有些许不同：
                  recovery.fstab文件、sbin下部分工具、build.prop、init.bootmode.emmc.rc、sepolicy等

	ramdisk.img   是根文件系统，是root目录打包成的镜像（ramdisk-recovery.img）
	system.img    包括了主要的包、库等文件
	userdata.img  包括了一些用户数据

<!-- more -->


## image 目标编译过程
	---build/core/main.mk中---
	默认目标，即目标all
	.PHONY: droid
	DEFAULT_GOAL := droid
	$(DEFAULT_GOAL):             //root目标droid
	
	droid: droidcore dist_files  //1级目标，droidcore

	droidcore: files \			 //2级目标，files systemimage ...
		systemimage \
		$(INSTALLED_BOOTIMAGE_TARGET) \
		$(INSTALLED_RECOVERYIMAGE_TARGET) \
		$(INSTALLED_USERDATAIMAGE_TARGET) \
		$(INSTALLED_CACHEIMAGE_TARGET) \
		$(INSTALLED_VENDORIMAGE_TARGET) \
		$(INSTALLED_FILES_FILE)

	---build/core/Makefile中---
	INSTALLED_SYSTEMIMAGE := $(PRODUCT_OUT)/system.img
	BUILT_SYSTEMIMAGE := $(systemimage_intermediates)/system.img
	FULL_SYSTEMIMAGE_DEPS := $(INTERNAL_SYSTEMIMAGE_FILES) $(INTERNAL_USERIMAGES_DEPS)
	INSTALLED_FILES_FILE := $(PRODUCT_OUT)/installed-files.txt
	

	systemimage: $(INSTALLED_SYSTEMIMAGE)		//3级目标 ./system.img
	
	$(INSTALLED_SYSTEMIMAGE): $(BUILT_SYSTEMIMAGE) $(RECOVERY_FROM_BOOT_PATCH) | $(ACP)    //4级目标  _./system.img
	
	$(BUILT_SYSTEMIMAGE): $(FULL_SYSTEMIMAGE_DEPS) $(INSTALLED_FILES_FILE)                 //5级目标  
		$(call build-systemimage-target,$@)


	BUILT_USERDATAIMAGE_TARGET := $(PRODUCT_OUT)/userdata.img
	INSTALLED_USERDATAIMAGE_TARGET := $(BUILT_USERDATAIMAGE_TARGET)
	INTERNAL_USERDATAIMAGE_FILES := \                                                      //4级目标
    	$(filter $(TARGET_OUT_DATA)/%,$(ALL_DEFAULT_INSTALLED_MODULES))

	$(INSTALLED_USERDATAIMAGE_TARGET): $(INTERNAL_USERIMAGES_DEPS) \                       //3级目标
                                   $(INTERNAL_USERDATAIMAGE_FILES)
		$(build-userdataimage-target)

	define build-userdataimage-target  //做userdata.img分区build操作
	  $(call pretty,"Target userdata fs image: $(INSTALLED_USERDATAIMAGE_TARGET)")
	  @mkdir -p $(TARGET_OUT_DATA)
	  @mkdir -p $(userdataimage_intermediates) && rm -rf $(userdataimage_intermediates)/userdata_image_info.txt
	  $(call generate-userimage-prop-dictionary, $(userdataimage_intermediates)/userdata_image_info.txt, skip_fsck=true)
	  $(hide) PATH=$(foreach p,$(INTERNAL_USERIMAGES_BINARY_PATHS),$(p):)$$PATH \
	      ./build/tools/releasetools/build_image.py \
	      $(TARGET_OUT_DATA) $(userdataimage_intermediates)/userdata_image_info.txt $(INSTALLED_USERDATAIMAGE_TARGET)
	  $(hide) $(call assert-max-image-size,$(INSTALLED_USERDATAIMAGE_TARGET),$(BOARD_USERDATAIMAGE_PARTITION_SIZE))
	endef

	
	注 system.img userdata.img 会依赖skip_userdata.img := 这个变量 及 嵌套依赖device/.../BoardConfig.mk中的TARGET_USERIMAGES_USE_EXT4、BOARD_SYSTEMIMAGE_PARTITION_SIZE、BOARD_USERDATAIMAGE_PARTITION_SIZE和



## image解压方法
	ramdisk.img   用file命令查看显示ramdisk.img: gzip compressed data, from Unix，采用cpio打包，gzip压缩的
		cp -a ramdisk.img 000/ramdisk.img.gz
		cd 000/
		gunzip ramdisk.img.gz //得到ramdisk.img 文件，但是和原始的ramdisk.img是不一样的，大小要大不少
		mkdir root
		cd root
		cpio –i -F ../ramdisk.img
		
	system.img    用file命令查看显示system.img: data
		cp -a system.img 000/system.img
		cd 000/
		out/host/linux-x86/bin/simg2img system.img system.ext4.img //用simg2img工具把system.img转为为ext4文件格式
		mkdir system                                       //建立system目录
		sudo mount -t ext4 -o loop system.ext4.img system  //将ext4格式的镜像挂在到system目录

	userdata.img  用file命令查看显示userdata.img: data
		同system.img

	recovery.img  用file命令查看显示recovery.img: data
		需要使用外部工具https://github.com/xiaolu/mkbootimg_tools.git  mkboot命令
		./mkbootimg_tools/mkboot recovery.img recovery

	boot.img      用file命令查看显示boot.img: data
		需要使用外部工具https://github.com/xiaolu/mkbootimg_tools.git  mkboot命令
		./mkbootimg_tools/mkboot boot.img boot 

## image压缩方法
	a. ramdisk 使用cpio -o -H | gzip 生成ramdisk.img
	b. system userdata recovery都可以使用mkuserimg.sh来压缩成镜像
	   mkuserimg.sh -s 镜像源目录 镜像输出文件 ext4 挂在点 分区size FILE_CONTEXTS文件


## 单独编译image
	make systemimage - system.img
	make userdataimage - userdata.img
	make ramdisk - ramdisk.img
	make snod - 快速打包system.img (with this command, it will build a new system.img very quickly. 
                well, you cannot use "make snod" for all the situations. it would not check the dependences. 
                if you change some code in the framework which will effect other applications)


## 附:
	simg2img解压缩工具: Convert Android sparse images to raw images. 在out/host/linux-x86/bin/simg2img可以找到
	用法：simg2img <sparse_image_files> <raw_image_file>

	mkuserimg.sh镜像制作工具，在out/host/linux-x86/bin/mkuserimg.sh可以找到，工具中会调用make_ext4fs
	用法：mkuserimg.sh [-s] SRC_DIR OUTPUT_FILE EXT_VARIANT MOUNT_POINT SIZE [-j <journal_size>]
             [-T TIMESTAMP] [-C FS_CONFIG] [-B BLOCK_LIST_FILE] [FILE_CONTEXTS]

		例如制作system.img和userdata.img时：
		Running:  mkuserimg.sh -s out/target/product/rk3368_box/system out/target/product/rk3368_box/obj/PACKAGING/systemimage_intermediates/system.img ext4 system 1073741824 out/target/product/rk3368_box/root/file_contexts
		make_ext4fs -s -T -1 -S out/target/product/rk3368_box/root/file_contexts -l 1073741824 -a system out/target/product/rk3368_box/obj/PACKAGING/systemimage_intermediates/system.img out/target/product/rk3368_box/system
		
		Running:  mkuserimg.sh -s out/target/product/rk3368_box/data out/target/product/rk3368_box/userdata.img ext4 data 2147483648 out/target/product/rk3368_box/root/file_contexts
		make_ext4fs -s -T -1 -S out/target/product/rk3368_box/root/file_contexts -l 2147483648 -a data out/target/product/rk3368_box/userdata.img out/target/product/rk3368_box/data
        
	make_ext4fs工具：用于Android平台上制作ext4文件系统的镜像，在out/host/linux-x86/bin/make_ext4fs
	用法：make_ext4fs [ -l <len> ] [ -j <journal size> ] [ -b <block_size> ]
					 [ -g <blocks per group> ] [ -i <inodes> ] [ -I <inode size> ]
					 [ -L <label> ] [ -f ] [ -a <android mountpoint> ]
					 [ -S file_contexts ] [ -C fs_config ] [ -T timestamp ]
					 [ -z | -s ] [ -w ] [ -c ] [ -J ] [ -v ] [ -B <block_list_file> ] <filename> [<directory>]
			-l 分区大小 支持512M 以及1073741824 单位为bytes
			-s 生成ext4的S模式制作
            -T timestamp  // -1 mean what? fix me
			-a 挂在点
			-S  file_contexts文件
			<filename>  生成的目的文件
			<directory> 源文件目录



## 附2——一般Android系统的boot.img格式(RK平台的boot.img是没包含kernel部分的)
	因为boot.img的格式比较简单，它主要分为三大块（有的可能有四块）
	
	+—————–+
	| boot header | 1 page
	+—————–+
	| kernel | n pages
	+—————–+
	| ramdisk | m pages
	+—————–+
	| second stage | o pages
	+—————–+
	n = (kernel_size + page_size – 1) / page_size
	m = (ramdisk_size + page_size – 1) / page_size
	o = (second_size + page_size – 1) / page_size

	a. all entities are page_size aligned in flash
	b. kernel and ramdisk are required (size != 0)
	c. second is optional (second_size == 0 -> no second)



