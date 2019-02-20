---
title: C++的static用法
date: 2016-05-28 12:22:33
tags: [static]
categories: C++
---


C++ 中static 关键字用法可以从两个方面来区分，一个是面向过程，另一个是面向对象。面向过程有三种用法，面向对象有两种用法。下文将详细讲解。

# 用法详解（从作用域，生命周期角度来解释）
> 面向过程

文件内全局static变量 （和C一样）
    作用域：本文件内
    生命周期：程序退出

函数内static变量 （和C一样）
    作用域：本函数内
    生命周期：程序退出


> 面向对象

类中static成员变量
    用法：使用时不用实例化，与类实例无关。（静态数据成员存储在全局数据区，静态数据成员定义时要分配空间，所以不能在类声明中定义。应该在类外定义。此外，在类体外定义时不能指定static关键字）。
    初始化格式：<数据类型><类名>::<静态数据成员> = <值>
    作用域：遵从类的public、private、protected访问规则
    生命周期：程序退出

类中static成员方法
    用法：可以引用static成员变量。无法访问属于类对象的非静态数据成员变量/函数。此外，类体外定义时不能指定static关键字
         调用static成员方法方式： XClass::xxx_static_fun()、XClassInstant.xxx_static_fun()、XClassInstantP->xxx_static_fun()
    作用域：遵从类的public、private、protected访问规则
    生命周期：N/A



# 附1（可执行文件内存布局）
![exec_view.bmp](https://i.loli.net/2018/12/15/5c14f2fa28125.bmp)

1. .text 代码段存放可执行指令
2. .data 初始化后数据 {
    a. 只读数据区(const全局变量 字符串)
    b. 已初始化可读数据区——全局区(全局变量) 
    c. 已初始化可读数据区——全局静态区(static全局变量) 
3. .bss 未初始化数据（只记录数据需要多少空间，不为其真正分配内存）
    a. 未初始化静态和全局变量
4. .heap 运行时malloc动态分配的数据占用空间
    a. 动态内存分配
5. .stack 函数运行时分配的自动变量占用空间
    a. 局部变量、函数参数
6. 从可执行文件a.out的角度来讲，如果一个数据未被初始化那就不需要为其分配空间，所以.data和.bss一个重要的区别就是.bss并不占用可执行文件的大小，它只是记载需要多少空间来存储这些未初始化数据，而不分配实际的空间


# 附2（参考）
http://blog.csdn.net/kerry0071/article/details/25741425/
