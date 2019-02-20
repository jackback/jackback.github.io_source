---
title: Input系统——TYPE CODE 
date: 2017-04-12 12:22:33
tags: [input, input系统, event]
categories: Android
---



# Event Type&Code

    EV_SYN:  - 用于事件间的分割标志。事件可能按时间或空间进行分割，就像在多点触摸协议中的例子。
        SYN_REPORT:    - 当多个输入数据在同一时间发生变化时，SYN_REPORT用于把这些数据进行打包和包同步。
                    例如，一次鼠标的移动可以上报   REL_X和REL_Y两个数值，然后发出一个SYN_REPORT。
                    下一次鼠标移动可以再次发出REL_X和REL_Y两个数值，然后经跟这另一个SYN_REPORT。
    
        SYN_MT_REPORT: - 用于同步和分离触摸事件
        SYN_DROPPED:   - 用来指出evdev客户的事件队列的的缓冲区溢出。

    EV_KEY:  - 用来描述键盘，按键或者类似键盘设备的状态变化。
        KEY_<name>:    - EV_KEY事件采取KEY_<name> 或BTN_<name>的形式，
                            比如，KEY_A代表键盘上的A键，当一个按键被按下时，一个带有按键编码和value为1的事件被发出。
                            当一个按键被释放时，一个value为0的事件被发出。有些硬件当按键重复时会发出事件，这些事件的value值为2。
                            通常，KEY_<name>用作键盘上的按键，而BTN_<name>则用于开关按钮事件。 
                       - 普通键盘按键     0x0~0x100     0x160~0x2ff
                       - 游戏柄的按键     0x100~0x110   0x120~0x140
                       - 鼠标按键         BTN_MOUSE-BTN_LEFT-0x110-左键；  BTN_RIGHT-0x111-右键；  BTN_MIDDLE-0x112-中键；
        BTN_<name>:    - see above

<!-- more -->

    EV_REL:  - 用来描述相对坐标轴上数值的变化，例如：鼠标向左方移动了5个单位。
        REL_X 0x00      //X 轴
        REL_Y 0x01      //Y 轴
        REL_Z 0x02      //Z 轴
        REL_RX 0x03
        REL_RY 0x04
        REL_RZ 0x05
        REL_HWHEEL 0x06 //水平方向horizontal的滚轮
        REL_DIAL 0x07
        REL_WHEEL 0x08  //垂直方向vertical的滚轮
        REL_MISC 0x09
        REL_MAX 0x0f
        REL_CNT (REL_MAX+1)
        
    EV_ABS:  - 用来描述相对坐标轴上数值的变化，例如：描述触摸屏上坐标的值。
        ABS_X 0x00
        ABS_Y 0x01
        ABS_Z 0x02
        ABS_RX 0x03
        ABS_RY 0x04
        ABS_RZ 0x05
        ABS_THROTTLE 0x06
        ABS_RUDDER 0x07
        ABS_WHEEL 0x08
        ABS_GAS 0x09
        ABS_BRAKE 0x0a
        ABS_DISTANCE 0x19 - 用来描述触摸工具离触摸表面的距离
        ABS_MT_<name> 用于描述多手指触摸输入设备。详情请参考内核文档：multi-touch-protocol.txt。

    EV_MSC:  - 当不能匹配现有的类型时，使用该类型进行描述。
    EV_SW:   - 用来描述具备两种状态的输入开关。
        SW_LID 0x00  用来指出笔记本电脑的屏幕是否合上
        
    EV_LED:  - 用于设置或查询设备上的LED灯的开和关。
    EV_SND:  - 用来给设备输出提示声音。
    EV_REP:  - 用于可以自动重复的设备（autorepeating）。
    EV_FF:   - 用来给输入设备发送强制回馈命令。（震动？）
        FF_RUMBLE 0x50
        FF_CONSTANT 0x52

    EV_PWR:  - 特别用于电源开关的输入。.
    EV_FF_STATUS:  - 用于接收设备的强制反馈状态。



# INPUT_DEVICE_ClASS 来区分
    INPUT_DEVICE_CLASS_CURSOR  
        有BTN_MOUSE按钮 且 REL_X及REL_Y 

    INPUT_DEVICE_CLASS_TOUCH | INPUT_DEVICE_CLASS_TOUCH_MT 
        有BTN_TOUCH  ABS_MT_POSITION_X ABS_MT_POSITION_Y 且没有游戏键

    INPUT_DEVICE_CLASS_JOYSTICK 


    INPUT_DEVICE_CLASS_SWITCH
        任一个即可


    INPUT_DEVICE_CLASS_VIBRATOR
        有FF_RUMBLE-0x50


# TOOL_TYPE
    AMOTION_EVENT_TOOL_TYPE_UNKNOWN 0
        类型未知，或者都无关联项，例如 trackball or other non-pointing device.
    AMOTION_EVENT_TOOL_TYPE_FINGER  1
        The tool is a finger 触摸类工具
    AMOTION_EVENT_TOOL_TYPE_STYLUS  2
        类似触针类工具，例如手写笔等
    AMOTION_EVENT_TOOL_TYPE_MOUSE   3
        The tool is a mouse or trackpad.  鼠标或者触摸板
    AMOTION_EVENT_TOOL_TYPE_ERASER  4
        The tool is an eraser or a stylus being used in an inverted posture. 擦除类工具，或者一类反向针式工具


# MotionEvent
    用于报告运动事件，例如 mouse, pen, finger, trackball 
    Motion events may hold either absolute or relative movements and other data, depending on the type of device.

    ACTION_BUTTON_PRESS
    ACTION_BUTTON_RELEASE
    ACTION_CANCEL  // The current gesture has been aborted.
    ACTION_SCROLL  // The motion event contains relative vertical and/or horizontal scroll offsets.
    ACTION_UP      // 
    AXIS_SCROLL    // Generic scroll axis of a motion event.


# AINPUT_SOURCE
    基本类型
    AINPUT_SOURCE_CLASS_MASK        0x000000ff      //         
    AINPUT_SOURCE_UNKNOWN           0x00000000      // 
    AINPUT_SOURCE_CLASS_BUTTON      0x00000001      // 输入设备有按钮和键
    AINPUT_SOURCE_CLASS_POINTER     0x00000002      // 与显示相关的指针式设备
    AINPUT_SOURCE_CLASS_NAVIGATION  0x00000004      // 
    AINPUT_SOURCE_CLASS_TRACKBALL   0x00000004      // 轨迹导航设备
    AINPUT_SOURCE_CLASS_POSITION    0x00000008      // 绝对定位设备（与显示无关的定位）
    AINPUT_SOURCE_CLASS_JOYSTICK    0x00000010      // 
    AINPUT_SOURCE_CLASS_NONE        0x00000000      // joystick 操作杆，绝对运动
    
    衍生类型
    AINPUT_SOURCE_KEYBOARD          0x00000101      // 
    AINPUT_SOURCE_DPAD              0x00000201      // 
    AINPUT_SOURCE_GAMEPAD           0x00000401      // 
    AINPUT_SOURCE_TOUCHSCREEN       0x00001002      // 
    AINPUT_SOURCE_MOUSE             0x00002002      // 
    AINPUT_SOURCE_STYLUS            0x00004002      //
    AINPUT_SOURCE_BLUETOOTH_STYLUS  0x0000c002      // 蓝牙指针，蓝牙鼠标?   API 23
    AINPUT_SOURCE_TRACKBALL         0x00010004      // 
    AINPUT_SOURCE_MOUSE_RELATIVE    0x00020004      // Android O Developer Preview
    AINPUT_SOURCE_TOUCHPAD          0x00100008      //
    AINPUT_SOURCE_TOUCH_NAVIGATION  0x00200000      // 
    AINPUT_SOURCE_ROTARY_ENCODER    0x00400000      // Android O Developer Preview  
    AINPUT_SOURCE_JOYSTICK          0x01000010      // 
    AINPUT_SOURCE_HDMI              0x02000001      // 
    AINPUT_SOURCE_ANY               0xffffff00      //


# AINPUT_KEYBOARD_TYPE
    AINPUT_KEYBOARD_TYPE_NONE 0                     // There is no keyboard
    AINPUT_KEYBOARD_TYPE_NON_ALPHABETIC 1           // The keyboard supports a complement of alphabetic keys.
    AINPUT_KEYBOARD_TYPE_ALPHABETIC 2               // The keyboard is not fully alphabetic



# Input device configuration (.idc)
  Input device configuration files (.idc files) contain device-specific configuration properties that affect the behavior of input devices.

  **Location**
    /system/usr/idc/  or   /data/system/devices/idc/

  **idc文件example如下**

    # This is an example of an input device configuration file.
    # It might be used to describe the characteristics of a built-in touch screen.
    
    # This is an internal device, not an external peripheral attached to the USB
    # or Bluetooth bus.
    device.internal = 1
    
    # The device should behave as a touch screen, which uses the same orientation
    # as the built-in display.
    touch.deviceType = touchScreen
    touch.orientationAware = 1
    
    # Additional calibration properties...
    keyboard.layout = qwerty
    keyboard.characterMap = qwerty
    keyboard.orientationAware = 1
    keyboard.builtIn = 1
    
    cursor.mode = navigation
    cursor.orientationAware = 1

  **Common属性语法**

    device.internal // Specifies whether the input device is an internal built-in component 
                        as opposed to an externally attached (most likely removable) peripheral.
                        If the value is 0, the device is external.
                        If the value is 1, the device is internal.
                        If the value is not specified, the default value is 0 for all devices on the USB (BUS_USB) or Bluetooth (BUS_BLUETOOTH) bus, 1 otherwise.

                        This property determines default policy decisions regarding wake events.
    


# Key Layout Files (.kl)
  详见 https://source.android.com/devices/input/key-layout-files 

    Key layout files (.kl files) map Linux key codes and axis codes to Android key codes and axis codes and specify associated policy flags. Device-specific key layout files are:
    
    Required for internal (built-in) input devices with keys, including special keys such as volume, power, and headset media keys.
    Optional for other input devices but recommended for special-purpose keyboards and joysticks.
    If no device-specific key layout file is available, the system chooses a default instead.

###  Location
  /system/usr/keylayout/ or /data/system/devices/keylayout/

### Generic Key Layout File
  The system provides a special built-in generic key layout file called Generic.kl. This key layout is intended to support a variety of standard external keyboards and joysticks. Do not modify the generic key layout!

### Syntax
  A key layout file is a plain text file consisting of key or axis declarations and flags.
  
  **Key Declarations**  
    key         Linux key code number               Android key code name
    key usage   HID usage(Hex 32-bit integer)       Android key code name
    
  **Axis Declarations**  
    axis        Linux axis code number              Android axis code name 
    Basic Axes
    Split Axes
    Inverted Axes
    Center Flat Position Option

  **Comments**
    Comment lines begin with # and continue to the end of the line:
    `# A comment!`

  **Examples** 

    -- Keyboard --
    key 1     ESCAPE
    key 2     1
    key 3     2
    key 4     3
    key 5     4
    key 6     5
    key 7     6
    key 8     7
    key 9     8
    key 10    9
    key 11    0
    key 12    MINUS
    key 13    EQUALS
    key 14    DEL

    -- System Controls --
    # This is an example of a key layout file for basic system controls,
    # such as volume and power keys which are typically implemented as GPIO pins
    # the device decodes into key presses.

    key 114   VOLUME_DOWN
    key 115   VOLUME_UP
    key 116   POWER

    -- Capacitive Buttons -- 
    # This is an example of a key layout file for a touch device with capacitive buttons.
    key 139    MENU           VIRTUAL
    key 102    HOME           VIRTUAL
    key 158    BACK           VIRTUAL
    key 217    SEARCH         VIRTUAL

    -- Headset Jack Media Controls --
    # This is an example of a key layout file for headset mounted media controls.
    # A typical headset jack interface might have special control wires or detect known
    # resistive loads as corresponding to media functions or volume controls.
    # This file assumes that the driver decodes these signals and reports media
    # controls as key presses.
    key 163   MEDIA_NEXT
    key 165   MEDIA_PREVIOUS
    key 226   HEADSETHOOK

    -- Joystick --
    # This is an example of a key layout file for a joystick.
    
    # These are the buttons that the joystick supports, represented as keys.
    key 304   BUTTON_A
    key 305   BUTTON_B
    key 307   BUTTON_X
    key 308   BUTTON_Y
    key 310   BUTTON_L1
    key 311   BUTTON_R1
    key 314   BUTTON_SELECT
    key 315   BUTTON_START
    key 316   BUTTON_MODE
    key 317   BUTTON_THUMBL
    key 318   BUTTON_THUMBR
    
    # Left and right stick.
    # The reported value for flat is 128 in a range of -32767 to 32768, which is absurd.
    # This confuses applications that rely on the flat value because the joystick
    # actually settles in a flat range of +/- 4096 or so. We override it here.
    axis 0x00 X flat 4096
    axis 0x01 Y flat 4096
    axis 0x03 Z flat 4096
    axis 0x04 RZ flat 4096
    
    # Triggers.
    axis 0x02 LTRIGGER
    axis 0x05 RTRIGGER
    
    # Hat.
    axis 0x10 HAT_X
    axis 0x11 HAT_Y

### Virtual Soft Keys
  // to do

### Validation
  You should validate your key layout files using the Validate Keymaps tool.


# Key Character Map Files (.kcm)
  Key character map files (.kcm files) are responsible for mapping combinations of Android key codes with modifiers to Unicode characters.

  **Location**
    /system/usr/keychars/
    /data/system/devices/keychars/

  **Generic Key Character Map File**
    Generic.kcm

  **Virtual Key Character Map File**
    Virtual.kcm

  **Syntax**
    Keyboard Type Declaration
    Key Declarations
    Comments



# Keyboard Devices
# Touch Devices






# 附1：
HID号码即Hidden ID code 隐含码；
PID号码即Public ID code 公开码。

# 附2(参考资料)
https://source.android.com/devices/input/overview
https://www.kernel.org/doc/Documentation/input/event-codes.txt
https://www.kernel.org/doc/Documentation/input/input.txt
https://www.kernel.org/doc/Documentation/input/multi-touch-protocol.txt
https://developer.android.com/reference/android/support/wearable/input/RotaryEncoder.html


