---
title: bindService
tags: [Android, service, bindService]
categories: Android
date: 2019-03-20 15:20:16
---


# bindService
    https://blog.csdn.net/u013553529/article/details/54605659
    https://blog.csdn.net/u013553529/article/details/54698123

    
    https://www.jianshu.com/p/03fb12a96b30
    
    
    1. service的onCreate只会执行一次，onBind也只会执行一次，onStartCommand可以执行多次
        多次调用startService会多次执行onstartCommand。
        startService -> onCreate(第一次start是调用,只一次) -> onStartCommand(可多次) 
            -> 运行中 -> stopService -> onDestroy(只会一次(没有绑定时执行stopService时))
        bindService  -> onCreate(只一次) -> onBind(只需要调用一次) -> (onServiceConnected,可多次,一个conn一次) 
            -> 运行中 -> unbindService -> onUnbind(最后一次解绑时) -> onDestroy(最后一次解绑时,且没有startService时)
        
        
    
## 二级标题

### 三级标题

#### 四级标题 

##### 五级标题

###### 六级标题


<!-- more -->



