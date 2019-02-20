---
title: C++ 基本语法
date: 2016-05-28 00:00:00
tags: [constructor, destructor]
categories: C++
---


基本语法：new、


# new关键字
### new创建基本类型变量
void *ptr  = new char; new short; new int; new float; //delete ptr
void *ptr  = new int(5);    //初始值5
void *ptr  = new int[100];  //100个元素的int数组;   delete []ptr
void **ptr = new int[5][6]; //delete [][]ptr

### new创建类
Test test;    Test test = Test();  Test * pTest = new Test(1)
Test test(1); Test test = Test(1); Test* pTest = new Test(); 


# reinterpret_cast <new_type> 


