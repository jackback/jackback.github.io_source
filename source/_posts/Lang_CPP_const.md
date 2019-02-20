---
title: C++的const用法
date: 2016-05-28 12:22:33
tags: [const]
categories: C++
---

C++ const 允许指定一个语义约束，编译器会强制实施这个约束，允许程序员告诉编译器某值是保持不变的。如果在编程中确实有某个值保持不变，就应该明确使用const，这样可以获得编译器的帮助。
const 有几种用法。下面将详细讲解。

1. const 全局变量
        用法： const int a;           //     const data
              const int *p;          //     const data, non-const pointer
              int * const p;         // non-const data, const pointer
              const int * const p;   //     const data, const pointer
              
2. const 函数局部变量 (包括函数参数、返回值等)
        用法：const int a; const int *p;int * const p;const int * const p;
             const int* function(const int *a);  // 返回值必须传给const data,non-const pointer 的变量
             void int function(const int& a);

3. const 成员变量
        用法：成员常量不能被修改，只能在构造函数初始化列表中赋值
        
4. const 成员方法
        用法： 不能修改任何成员变量（mutable修饰的变量除外），不能调用非const成员函数
        定义格式：void function() const {}
5. const 类对象、类指针、类引用


6. 将Const类型转化为非Const类型的方法
采用const_cast 进行转换。该运算符用来修改类型的const或volatile属性。

