---
title: Java Timer
date: 2019-03-14 12:22:33
tags: [Java, Timer]
categories: Java
---

# Timer的几点使用笔记


    //定义TimerTask;
    private class StateTimerTask extends TimerTask {
        @Override
        public void run() {
            //do something
        }
    }
    
    //定义Timer
    mTimer = new Timer();
    
    //入队列
    mTimer.schedule(task = new StateTimerTask(), delay);
    
    //task.cancle(); //取消队列, 后面不能再重复添加task这个来schedule,要重新new TimerTask才行
    
    //mTimer.cancle(); //取消定时器 , 后面不能再使用这个定时器进行schedule了