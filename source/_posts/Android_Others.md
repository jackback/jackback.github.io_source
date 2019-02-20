---
title: Android 杂项
date: 2016-08-08 15:13:00
tags: 
categories: Android
---

# 命令行启动jar包
export CLASSPATH=/system/framework/sg.jar
trap "" HUP
exec app_process /system/bin com.china_liantong.commands.SGCmdToolManager systemtask exec "ping -c 1 192.168.0.1"


exec app_process /system/bin com.china_liantong.commands.SGCmdToolManager systemtask exec "ping -c 1 192.168.0.1"


# 获取包名
String callingApp = context.getPackageManager().getNameForUid(Binder.getCallingUid());
context.getPackageName 区别


# sendBroadcast & BroadcastReceiver
MainActivity.java
public class MainActivity extends Activity { 
    private static final String MY_ACTION = "com.android.notification.MY_ACTION";  
    public void onCreate(Bundle savedInstanceState) { 
        btn.setOnClickListener(listener);
    }
    private OnClickListener listener = new OnClickListener() { 
        @Override 
        public void onClick(View v) {
            // 实例化Intent  
            Intent intent = new Intent();  
            // 设置Intent action属性  
            intent.setAction(MY_ACTION);  
            // 发起广播  
            sendBroadcast(intent);
        }
    }
}

MyReceiver.java
public class MyReceiver extends BroadcastReceiver{  
    @Override 
    public void onReceive(Context context, Intent intent) {
        // 实例化Intent  
        Intent i = new Intent();  
        // 在新的任务中启动Activity  
        i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);  
        // 设置Intent启动的组件名称  
        i.setClass(context, SecondActivity.class);  
        // 启动Activity显示通知  
        context.startActivity(i);
    }
}


SecondActivity.java
public class SecondActivity extends Activity {
    public void onCreate(Bundle savedInstanceState) {
        
    }
}



AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?> 
<manifest xmlns:android="http://schemas.android.com/apk/res/android" 
      package="com.android.notification" 
      android:versionCode="1" 
      android:versionName="1.0"> 
    <uses-sdk android:minSdkVersion="10" />
    <application android:icon="@drawable/icon" android:label="@string/app_name"> 
        <activity android:name=".MainActivity" 
                  android:label="@string/app_name"> 
            <intent-filter> 
                <action android:name="android.intent.action.MAIN" /> 
                <category android:name="android.intent.category.LAUNCHER" /> 
            </intent-filter> 
        </activity> 
        <receiver android:name="MyReceiver"> 
            <intent-filter> 
                <action android:name="com.android.notification.MY_ACTION"/> 
            </intent-filter> 
        </receiver>   
        <activity android:name="SecondActivity"/> 
    </application> 
</manifest> 



# 屏幕超时休眠配置：
packages/apps/Settings/res/values/arrays.xml       //菜单项配置

frameworks/base/packages/SettingsProvider/res/values/defaults.xml   // 默认值配置
Lollipop/device/rockchip/common/overlay_screenoff/frameworks/base/packages/SettingsProvider/res/values/defaults.xml  //默认值配置


# adb connect 提示 unable to connect to解决方法

使用ADB 连接Android设备时，提示连接失败，在Android设备中执行以下命令，重试

setprop service.adb.tcp.port 5555
stop adbd
start adbd


