---
title: Android中Vector、ArrayList、LinkedList和HashMap/ArrayMap/SparseArray簇
date: 2019-03-14 12:22:33
tags: [Vector, ArrayList, LinkedList, HashMap, ArrayMap, SparseArray]
categories: Android
---


# 先说：Iterator、Iterable、Collection 接口

public interface Iterator<E>接口：     包含hasNext next remove三个方法
public interface Iterable<T> 迭代接口：   包含一个iterator() 方法
public interface Collection<E> extends Iterable<E>：继承Iterable接口(Iterable是接口，使用的关键字是extends，因为Collection也是接口),并包含add remove clear等操作方法；contains equals isEmpty 等检测方法；size iterator toArray工具方法。


AbstractCollection  集合的类化，抽象类：   有添加删除特性的抽象类 
                    public abstract class AbstractCollection<E> implements Collection<E> {}
                    需要实现：
                        public boolean add(E object) {throw}
                        迭代接口iterator(); （实例化一个实现接口Iterator<E>的类，并作为返回值即可）
                        public abstract int size();


# 再来看接口 List 和 ListIterator            
public interface List<E> extends Collection<E> {}             List在Collection的基础上添加了index的特性。
        接口重写：add、clear
        public void add(int location, E object);
        public E get(int location);
        public int indexOf(Object object);
        public int lastIndexOf(Object object);  //最后一次出现的index
        public ListIterator<E> listIterator(); //见ListIterator
        public ListIterator<E> listIterator(int location);  //从指定位置开始迭代
        public E set(int location, E object); 
        public List<E> subList(int start, int end); 

public interface ListIterator<E> extends Iterator<E> {} 在Iterator基础上添加了连贯性的特点
        除了add hasNext remove 添加了hasPrevious  nextIndex();   previous   previousIndex(); set(E object); 等操作



# 看看List的类化——AbstractList 
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {}
            集Collection和List于一身的抽象List类
 

# ArrayList类
public class ArrayList<E> extends AbstractList<E> implements Cloneable, Serializable, RandomAccess {}
    Object[] array;    数组进行存储元素


# Vector类 （带同步功能的List）
public class Vector<E> extends AbstractList<E> implements List<E>,
        RandomAccess, Cloneable, Serializable {

......................................

# LinkedList类 （双链表，队列特性）
public class LinkedList<E> extends AbstractSequentialList<E> implements
        List<E>, Deque<E>, Queue<E>, Cloneable, Serializable {


.......................................


# HashMap类

Map接口 public interface Map<K,V>  包含k-v 对的集合


# ArrayMap类


# SparseMap簇类 
### 特点, 以及在什么场景使用?
    Key为int型的Map时使用.
    android希望我们用SparseArray在一些情况下代替HashMap来使用，因为它有更好的性能

    
### 基本操作
    put(int key, E value);
    get(int key); //E value
    get(int key, E valueIfKeyNotFound)
    delete(int key)
    removeAt(int index)
    keyAt(int index) //int key; 通过index得到该index位置的key
    valueAt(int index)
    
    
### 遍历
    //按照添加顺序, 遍历key,然后再得到value
    SparseArray<E> sparseArray = new SparseArray<>();

    for(int i = 0; i < sparseArray.size(); i++) {
        int key = sparseArray.keyAt(i);
        E value = sparseArray.get(key);
    }
    //按照添加顺序, 直接遍历value;
    SparseArray<E> sparseArray = new SparseArray<>();

    for(int i = 0; i < sparseArray.size(); i++) {
        E value = sparseArray.valueAt(i);
    }


# 其他
    据key-value中的value类型不同，android又给封装了SparseIntArray, SparseBooleanArray, SparseLongArray等等，
    使用方法和SparseArray都大同小异，只要你会使用Map，那么你就会使用SparseArray。




    
# TO DO
https://www.cnblogs.com/LipeiNet/p/5888513.html






