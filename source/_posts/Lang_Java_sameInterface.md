---
title: Java中父类和子类可以同时实现一个接口，有什么作用？
date: 2016-09-19 12:22:33
tags: 
categories: Java
---

看下面的代码片段，父类和子类都实现了接口A

    Class Base `implements  A`
    Class Sub extends Base  `implements  A`

就是说子类Sub实现A有什么用？

我原来认为是多余的。实际上， 子类再次实现父类实现的接口是为了强制子类(SUB)重写父类(BASE)中实现的所有接口中的方法。


现实中的例子：
父类：public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {}

子类：public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable {}


