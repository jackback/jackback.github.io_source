---
title: Android SEPolicy策略规则 
date: 2016-10-12 18:22:33
tags: [SEAndroid, SEPolicy, SELinux]
categories: Android
---




# SEAndroid中各种实体的SContext 定义
介绍进程、文件/目录、服务、系统属性等SContext 定义

##### 进程的SContext（以system_server进程为例）（对应每个进程的te文件）
	u:r:system_server:s0           system    494   186   system_server
	u表示user SELinux用户
	r表示SELinux角色
	system_server表示domain，该域的权限定义在策略文件system_server.te中
	s0表示MLS中的级别，Android中可忽略

##### file和dir的SContext（以system目录为例）（在file_contexts定义）
	drwxr-xr-x root     root              u:object_r:system_file:s0 system
	object_r表示文件对象
	system_file表示文件类型

##### fs_use(在fs\_use定义)
	// to do

##### 生成 文件系统的 SContext （在genfs_contexts定义）
	genfscon rootfs / u:object_r:rootfs:s0
	genfscon usbfs  / u:object_r:usbfs:s0

##### property的SContext（在property_contexts定义）
    *                       u:object_r:default_prop:s0    //默认property SContext
    hw.                     u:object_r:system_prop:s0

##### service的SContext（在service_contexts定义）
	*                       u:object_r:default_android_service:s0   //默认服务SContext
	package                 u:object_r:system_server_service:s0     //package 服务  （在shell中执行service list查看所有服务）

<!-- more -->

##### APP的SContext （在mac_permissions.xml+seapp_contexts+xxx定义，详见 APP安全策略）
  APP的seinfo: 根据apk的签名决定app的seinfo(platform、default)

    1. 特殊APP——system_server (因为它也是有zygote产生的，所以还是也在seapp_contexts中定义)
    isSystemServer=true domain=system_server                  // isSystemServer返回true，则domain=system_server
    2. 通过user + seinfo + [name]来决定domain 和 type
    user=cts seinfo=* name=roidJUnitRunner domain=ctsrunner_app type=app_data_file  // user 是cts，则domain=ctsrunner_app type=app_data_file CTS测试APP
    user=system domain=system_app type=system_app_data_file   // user是 system seinfo默认是platform签名，则domain=system_app type=system_app_data_file 系统APP
                                                              // (有android平台签名和system权限)
    user=bluetooth domain=bluetooth type=bluetooth_data_file  // bluetooth
    user=nfc domain=nfc type=nfc_data_file                    // nfc
    user=radio domain=radio type=radio_data_file              // radio
    user=shared_relro domain=shared_relro                     // shared_relro
    user=shell domain=shell type=shell_data_file              // shell
    user=_isolated domain=isolated_app                        // _isolated
    user=_app seinfo=platform domain=platform_app type=app_data_file  // 平台APP(有android平台签名，没有system权限)
    user=_app domain=untrusted_app type=app_data_file                 // 其他的都是第三方(untrusted_app)APP (没有Android平台签名，没有system权限)


##### APP安全策略
  APP的SContext 确定了APP domain和type ，下面就是不同类型的app domain的安全策略te文件

	app.te 应用给所有APP的安全策略（这里的所有APP指的是所有由zygote产生的进程），appdomain域——APP的基域
	
	system_server.te system_server的安全策略
	ctsrunner_app.te CTS APP 的安全策略
	system_app.te    系统 APP的安全策略
	bluetooth.te     bluetooth APP的安全策略
	nfc.te           nfc APP 的安全策略
	radio.te         radio APP的安全策略
	shared_relro.te  shared_relro APP的安全策略
	shell.te         通过shell 启动的APP的安全策略
	isolated_app.te  isolated APP的安全策略         AID_ISOLATED_START (99000) ---> AID_ISOLATED_END (99999).
	platform_app.te  第三方平台 APP的安全策略
	untrusted_app.te 其他的都是第三方 APP的安全策略 APP_AID (10000) ---> AID_ISOLATED_START (99000)




## 安全策略语法
  上面都将各个实体的SContext 逐一罗列出来了，下面讲下*.te 文件语法    

##### 关键字的关键字 （有点拗口，但确实是这样的）
  attribute       用于指定xxx_type和domain关键字的关键字  
  type            用于指定domain域的关键字的关键字   
  typeattribute   单独将某个type和某个或多个attribute关联起来   

### 基本语法     

    我们再来看看最常见的权限设置语句：
    allow netd sysfs:file write;
    1）allow：表示授权。除了allow之外，还有allowaudit、dontaudit、neverallow等；       这条语句的语法为：
    2）netd：sourcetype，也叫Domain，Subject。
    3）sysfs：targettype，它代表其后的file所对应的Type。
    4）file：objectclass，它代表能够给Domain操作的对象。例如file、dir、socket等，Android中SecurityClass的定义在security_classes中。在Android系统中，有一些特殊的Class，如property_service，binder等。
    5）write：在该类objectclass中所定义的操作，例如file类支持ioctl，read，write等操作。access_vectors中定义了所有objectclass支持的操作。

    根据SELinux规范，完整的allow语句格式为：
    rule_name source_type target_type:class perm_set;
    1）如果有多个source_type，target_type，class或perm_set，可以用"{}"括起来；       注意：使用allow语句的时候，可以使用下面的一些小技巧来简化命令书写；
    2）"~"号，表示除了"~"以外；
    3）"-"号，表示去除某项内容；
    4）"*"号，表示所有内容

### 高级语法
  对于一般的*.te文件编写，基本的语法就差不多够了，下面是高级语法，说高级语法就是相当于一个批量生成sepolicy规则的的语法，让你少些点规则         

##### 域转换(DomainTransition,简称DT)   
  SEAndroid中，init进程的SContext为u:r:init:s0（在init.rc中使用” setcon u:r:init:s0”命令设置），而init创建的子进程显然不会也不可能拥有和init进程一样的SContext
  （否则根据TE，这些子进程也就在MAC层面上有了和init进程一样的权限）。那么这些子进程是怎么被打上和父进程不一样的SContex呢？
  在SELinux中，上述问题被称为DomainTransition，即某个进程的Domain切换到一个更合适的Domain中去。DomainTransition也是在安全策略文件中配置的，而且有相关的关键字。

    语法: type\_transition  source\_type  target\_type:class  default\_type
    表示source_type域的进程在对target_type类型的文件进行class定义的操作时，进程会切换到default_type域中   

    下面我们看个域转换的例子：
    type_transition init shell_exec:process init_shell
    这个例子表示：当init域的进程执行（process）shell_exec类型的可执行文件时，进程会从init域切换到init_shell域。

    那么，哪个文件是shell_exec类型呢？从file_contexts文件能看到，/system/bin/sh的安全属性是u:object_r:shell_exec:s0，
    也就是说init域的进程如果运行shell脚本的话，进程所在的域就会切换到init_shell域，这就是DomainTransition（简称DT）。

    请注意，DT也是SELinux安全策略的一部分，type_transition不过只是开了一个头而已，要真正实施成功这个DT，还至少需要下面三条权限设置语句：

    # 首先，你得让init域的进程要能够执行shell_exec类型的文件
    allow init shell_exec:file execute;
    # 然后，你需要告诉SELinux，允许init域的进程切换到init_shell域
    allow init init_shell:process transition;
    # 最后，你还得告诉SELinux，域切换的入口（entrypoint）是执行shell_exec类型的文件
    allow init_shell shell_exec:file entrypoint;

    看起来挺麻烦，不过SELinux支持宏定义，我们可以定义一个宏，把上面4个步骤全部包含进来。在SEAndroid中，所有的宏都定义在te_macros文件中，其中和DT相关的宏定义如下：
    # 定义domain_trans宏，$1,$2,$3代表宏的第一个，第二个…参数
    define(`domain_trans', `
    allow $1 $2:file { getattr open read execute };
    allow $1 $3:process transition;
    allow $3 $2:file { entrypoint open read execute getattr };
    … …
    ')
    # 定义domain_auto_trans宏，这个宏才是我们在te中直接使用的
    define(`domain_auto_trans', `
    # Allow the necessary permissions.
    domain_trans($1,$2,$3)
    # Make the transition occur by default.
    type_transition $1 $2:process $3;
    ')
    呵呵，是不是简单多了，domain_trnas宏在DT需要的最小权限的基础上还增加了一些权限，在te文件中可以直接用domain_auto_trans宏来显示声明域转换，如下：
    domain_auto_trans(init, shell_exec, init_shell)

##### 类型转换(TypeTransition,简称TT)  
  除了DT外，还有针对Type的Transition（简称TT）。举个例子，假设目录A的SContext为u:r:dir_a，那么默认情况下，
  在该目录下创建的文件的SContext就是u:r:dir_a，如果想让它的SContext发生变化，那么就需要TypeTransition。

    和DT类似，TT的关键字也是type_transition，而且要顺利完成Transition，也需要申请相关权限，废话不多说了，
    直接看te_macros是怎么定义TT所需的宏：
    # 定义file_type_trans宏，为Type Transition申请相关权限
    define(`file_type_trans', `
    allow $1 $2:dir ra_dir_perms;
    allow $1 $3:notdevfile_class_set create_file_perms;
    allow $1 $3:dir create_dir_perms;
    ')
    # 定义file_type_auto_trans(domain, dir_type, file_type)宏
    # 该宏的意思就是当domain域的进程在dir_type类型的目录创建文件时，该文件的SContext
    # 应该是file_type类型
    define(`file_type_auto_trans', `
    # Allow the necessary permissions.
    file_type_trans($1, $2, $3)
    # Make the transition occur by default.
    type_transition $1 $2:dir $3;
    type_transition $1 $2:notdevfile_class_set $3;
    ')

    栗子： file_type_auto_trans(app_domain, system_data_file, xxx_type)

### class集合 class
    // to do

### 权限集合 perm_set
    // to do




## 附 

### SELinux模式切换命令
  SELinux支持Disabled，Permissive，Enforce三种模式
  Disabled就不用说了，此时SELinux的权限检查机制处于关闭状态；
  Permissive模式就是SELinux有效，但是即使你违反了它的安全策略，它让你继续运行，但是会把你违反的内容记录下来。在策略开发的时候非常有用，相当于Debug模式；
  Enforce模式就是你违反了安全策略的话，就无法继续操作下去。
  在Eng版本使用setenforce命令，可以在Permissive模式和Enforce模式之间切换。

    usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]

### -Z——查看SContext 信息的命令
    ls -Z   // 文件目录的SContext
    ps -Z   // 进程的SContext

### chcon命令——更改文件的安全属性
    usage:  chcon <SContext> <path>

### restorecon命令——恢复文件原来的安全属性
  当文件的安全属性在安全策略配置文件里面有定义时，使用restorecon命令，可以恢复原来的安全属性

    usage:  restorecon [-DFnrRv] <path>

### id 命令——确认自己的SecurityContext
    $ id
    uid=2000(shell) gid=2000(shell) groups=1007(log) context=u:r:shell:s0
    # id
    uid=0(root) gid=0(root) groups=1007(log) context=u:r:su:s0






####参考： 
  http://blog.csdn.net/modianwutong/article/details/43114883

  http://blog.csdn.net/zhudaozhuan/article/details/50964832
