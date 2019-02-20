---
title: C++ public/protected/private
date: 2016-05-28 12:22:33
tags: [public,protected,private]
categories: C++
---

C++ public protected private关键字
用于成员变量，成员函数权限的设置；以及用于类继承权限的设置。


# 1. 成员权限设置
|权限	    |类内部	   |该类对象	|子类（派生类）	|友元函数|
|:----------|:---------|:-----------|:--------------|:-------|
|private	|可访问    |不可访问    |不可访问       |可访问  |
|public	    |可访问	   |可访问      |可访问	        |可访问  |
|protected	|可访问	   |不可访问    |可访问         |可访问  |


# 2. 类继承权限设置
C++的类和对象的权限(相对于基类成员)

|继承方式    |基类对象    |派生类继承的基类成员权限变化                                        |派生类对象|
|------------|------------|--------------------------------------------------------------------|----------|
|public      |可访问      |public->public    <br> protected->protected <br> private->private   |可访问    |
|protected   |不可访问    |public->protected <br> protected->protected <br> private->private   |不可访问  |
|private     |不可访问	  |public->private   <br> private->private     <br> protected->private |不可访问  |
























