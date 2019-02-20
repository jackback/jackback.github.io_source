---
title: Android am cmd
date: 2016-00-00 00:00:00
tags: [am]
categories: Android
---


am命令实的实现方式在Am.java，最终几乎都是调用ActivityManagerService相应的方法来完成的，am monitor除外。比如前面概述中介绍的命令am start -a android.intent.action.VIEW -d http://gityuan.com， 启动Acitivty最终调用的是ActivityManagerService类的startActivityAsUser()方法来完成的。再比如am kill-all命令，最终的实现工作是由ActivityManagerService的killBackgroundProcesses()方法完成的。

### 命令格式及列表
am [subcommand] [options]


|命令	                            |功能	                    |实现方法|
|:----------------------------------|:--------------------------|:--------------------------------|
|am start [options] <INTENT>	    |启动Activity	            |startActivityAsUser|
|am startservice <INTENT>	        |启动Service	            |startService|
|am stopservice <INTENT>	        |停止Service	            |stopService|
|am broadcast <INTENT>	            |发送广播	                |broadcastIntent|
|am kill <PACKAGE>	                |杀指定后台进程	            |killBackgroundProcesses|
|am kill-all	                    |杀所有后台进程	            |killAllBackgroundProcesses|
|am force-stop <PACKAGE>	        |强杀进程	                |forceStopPackage|
|am hang	                        |系统卡住	                |hang|
|am restart	                        |重启	                    |restart|
|am bug-report	                    |创建bugreport	            |requestBugReport|
|am dumpheap <pid> <file>	        |进程pid的堆信息输出到file	|dumpheap|
|am send-trim-memory <pid> <level>	|收紧进程的内存	            |setProcessMemoryTrimLevel|
|am monitor	                        |监控	                    |MyActivityController.run|


### am用法详解 -- [options]

#### 启动Activity
am start [options] <INTENT>， 实现原理： 存在-W参数则调用startActivityAndWait()方法来运行，否则startActivityAsUser()。
[options]参数：
*   -D: 允许调试功能
*   -W: 等待app启动完成
*   -R <COUNT>: 重复启动Activity COUNT次
*   -S: 启动activity之前，先调用forceStopPackage()方法强制停止app.
*   –opengl-trace: 运行获取OpenGL函数的trace
*   –user <USER_ID> | current: 指定用户来运行App,默认为当前用户。
*   –start-profiler <FILE>: 启动profiler，并将结果发送到 <FILE>;
*   -P <FILE>: 类似 –start-profiler，不同的是当app进入idle状态，则停止profiling
*   –sampling INTERVAL: 设置profiler 取样时间间隔，单位ms;



#### 收紧内存
am send-trim-memory  <pid> <level>
用例: 
向pid=12345的进程，发出level=RUNNING_LOW的收紧内存命令
am send-trim-memory 12345 RUNNING_LOW

<level>参数:
level取值范围为： HIDDEN、RUNNING_MODERATE、BACKGROUND、RUNNING_LOW、MODERATE、RUNNING_CRITICAL、COMPLETE


#### 其他
am的子命令，startservice, stopservice, broadcast, kill, profile start, profile stop, dumpheap的可选参数都允许设置--user <USER_ID>。目前市面上的绝大多数手机还是单用户模式，故可以忽略该参数，默认为当前用户。

例如：启动id=10010的用户的指定service。
am startservice --user 10010 <INTENT>




#### am用法详解 -- <INTENT>
Intent的参数和flags较多，本文为方便起见，分为3种类型参数，常用参数，Extra参数，Flags参数。

*   -a <ACTION>: 指定Intent action， 实现原理Intent.setAction()；
*   -n <COMPONENT>: 指定组件名，格式为{包名}/.{主Activity名}，实现原理Intent.setComponent(）；
*   -d <DATA_URI>: 指定Intent data URI
*   -t <MIME_TYPE>: 指定Intent MIME Type
*   -c <CATEGORY> [-c <CATEGORY>] ...]:指定Intent category，实现原理Intent.addCategory()
*   -p <PACKAGE>: 指定包名，实现原理Intent.setPackage();
*   -f <FLAGS>: 添加flags，实现原理Intent.setFlags(int )，紧接着的参数必须是int型；


am start -a android.intent.action.VIEW
am start -n com.yuanhh.app/.MainActivity
am start -d content://contacts/people/1
am start -t image/png
am start -c android.intent.category.APP_CONTACTS




##### Extra参数

*基本类型*

|参数	|-e/-es	|-esn	        |-ez	    |-ei	|-el	|-ef	|-eu	|-ecn       |
|-------|-------|---------------|-----------|-------|-------|-------|-------|-----------|
|类型	|String	|(String)null	|boolean	|int	|long	|float	|uri	|component  |

比如参数es是Extra String首字母简称，实例：
  am start -n com.yuanhh.app/.MainActivity -es website gityuan.com
此处-es website gityuan.com，等价于Intent.putExtra(“website”, “gityuan.com”);

*数组类型*

参数	-esa	-eia	-ela	-efa
数组类型	String[]	int[]	long[]	float[]

比如参数eia，是Extra int array首字母简称，多个value值之间以逗号隔开，实例：
am start -n com.yuanhh.app/.MainActivity -ela weekday 1,2,3,4,5
此处-ela weekday 1,2,3,4,5，等价于Intent.putExtra(“weekday”, new int[]{1,2,3,4,5});

*ArrayList类型*

参数	-esal	-eial	-elal	-efal
List类型	String	int	long	float

比如参数efal，是Extra float Array List首字母简称，多个value值之间以逗号隔开，实例：
am start -n com.yuanhh.app/.MainActivity -efal nums 1.2,2.2
此处-efal nums 1.2,2.2，等价于先构造ArrayList变量，再通过putExtra放入第二个参数。


##### Flags参数
在参数类型1中，提到有-f <FLAGS>，是通过Intent.setFlags(int )方法，来设置Intent的flags.本小节也是关于flags，是通过Intent.addFlags(int )方法。如下所示，所有的flags参数。

[--grant-read-uri-permission] [--grant-write-uri-permission]
[--grant-persistable-uri-permission] [--grant-prefix-uri-permission]
[--debug-log-resolution]
[--exclude-stopped-packages] [--include-stopped-packages]
[--activity-brought-to-front] [--activity-clear-top]
[--activity-clear-when-task-reset] [--activity-exclude-from-recents]
[--activity-launched-from-history] [--activity-multiple-task]
[--activity-no-animation] [--activity-no-history]
[--activity-no-user-action] [--activity-previous-is-top]
[--activity-reorder-to-front] [--activity-reset-task-if-needed]
[--activity-single-top] [--activity-clear-task]
[--activity-task-on-home]
[--receiver-registered-only] [--receiver-replace-pending]


例如，发送action=”broadcast.demo”的广播，并且对于forceStopPackage()的应用不允许接收该广播，命令如下：
am broadcast -a broadcast.demo --exclude-stopped-packages


用例：
am start -n com.android.dialer/com.android.dialer.DialtactsActivity

From:http://gityuan.com/2016/02/27/am-command/