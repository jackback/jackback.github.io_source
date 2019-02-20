---
title: Input系统——Linux 输入子系统
date: 2017-04-12 12:22:33
tags: [input, input系统, event]
categories: Android
---


待整理完善。。。

### 1. struct input_dev
    struct input_dev {
    	const char *name; //名字 
    	const char *phys;
    	const char *uniq;
    	struct input_id id;
    
    	unsigned long propbit[BITS_TO_LONGS(INPUT_PROP_CNT)];
    
    	unsigned long evbit[BITS_TO_LONGS(EV_CNT)];    // 设备所支持的事件类型

    	unsigned long keybit[BITS_TO_LONGS(KEY_CNT)];  // 按键类型 
    	unsigned long relbit[BITS_TO_LONGS(REL_CNT)];  // 相对值类型
    	unsigned long absbit[BITS_TO_LONGS(ABS_CNT)];  // 绝对值类型
    	unsigned long mscbit[BITS_TO_LONGS(MSC_CNT)];  // 混合类型
    	unsigned long ledbit[BITS_TO_LONGS(LED_CNT)];  // LED类型
    	unsigned long sndbit[BITS_TO_LONGS(SND_CNT)];  // 声音类型
    	unsigned long ffbit[BITS_TO_LONGS(FF_CNT)];    // 力反馈
    	unsigned long swbit[BITS_TO_LONGS(SW_CNT)];    // 开关类型???

<!-- more -->

    	unsigned int hint_events_per_packet;
    
    	unsigned int keycodemax;
    	unsigned int keycodesize;
    	void *keycode;
    
    	int (*setkeycode)(struct input_dev *dev,
    			  const struct input_keymap_entry *ke,
    			  unsigned int *old_keycode);
    	int (*getkeycode)(struct input_dev *dev,
    			  struct input_keymap_entry *ke);
    
    	struct ff_device *ff;
    
    	unsigned int repeat_key;
    	struct timer_list timer;
    
    	int rep[REP_CNT];
    
    	struct input_mt *mt;
    
    	struct input_absinfo *absinfo;
    
    	unsigned long key[BITS_TO_LONGS(KEY_CNT)];
    	unsigned long led[BITS_TO_LONGS(LED_CNT)];
    	unsigned long snd[BITS_TO_LONGS(SND_CNT)];
    	unsigned long sw[BITS_TO_LONGS(SW_CNT)];
    
    	int (*open)(struct input_dev *dev);
    	void (*close)(struct input_dev *dev);
    	int (*flush)(struct input_dev *dev, struct file *file);
    	int (*event)(struct input_dev *dev, unsigned int type, unsigned int code, int value);
    
    	struct input_handle __rcu *grab;
    
    	spinlock_t event_lock;
    	struct mutex mutex;
    
    	unsigned int users;
    	bool going_away;
    
    	struct device dev;
    
    	struct list_head	h_list;
    	struct list_head	node;
    
    	unsigned int num_vals;
    	unsigned int max_vals;
    	struct input_value *vals;
    
    	bool devres_managed;
    };


### 2. 注册/注销函数
    int input_register_device搜索(struct input_dev *dev)
    void input_unregister_device(struct input_dev *dev)


### 3. 设备所支持的事件类型有
    EV_KEY 按键
    EV_REL 相对坐标
    EV_ABS绝对坐标
    EV_MSC 其它
    EV_LED LED
    EV_SND 声音
    EV_FF 力反馈
    EV_SW 开关???


### 4. 操作函数
    set_bit(EV_KEY, button_dev.evbit)   // 设置支持的事件
    void input_report_key(struct input_dev *dev,unsigned int code, int value)   // 在中断过程中中，内核层使用input_report_key等函数向用户空间报告，然后应用程序读取状态。
    void input_report_rel(struct input_dev *dev,unsigned int code, int value)   // ..
    void input_report_abs(struct input_dev *dev,unsigned int code, int value)   // ..
    input_sync()  // 用于事件同步，它告知事件的接收者：驱动已经发出了一个完整的报告。


### 5. 事件代码

EV_KEY 事件代码
    
    代码值0~127为键盘上的按键代码，0x110~0x116为鼠标上按键代码，其中0x110(BTN_LEFT)为鼠标左键，0x111(BTN_RIGHT)为鼠标右键,0x112(BTN_ MIDDLE)为鼠标中键。其它代码含义请参看include/linux/input.h文件


EV_KEY 事件值
    
    按键按下时值为1,松开时值为0



### evdev







