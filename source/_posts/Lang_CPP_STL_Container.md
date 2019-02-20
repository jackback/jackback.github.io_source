---
title: C++ STL之vector list map set
date: 2016-05-28 11:22:33
tags: [vector,list,map,set]
categories: C++
---

STL C++中有两种类型的容器：顺序容器和关联容器，顺序容器主要有：vector、list、deque等。其中vector表示一段连续的内存地址，基于数组的实现，list表示非连续的内存，基于链表实现。deque与vector类似，但是对于首元素提供删除和插入的双向支持。关联容器主要有map和set。map是key-value形式的，set是单值。map和set只能存放唯一的key值，multimap和multiset可以存放多个相同的key值。

容器类自动申请和释放内存，我们无需new和delete操作。

# vector<T>
一段连续的内存地址，基于数组的实现
``` CPP
#include<vector>
//1. 定义和初始化
vector<int> vec1; 默认初始化，vec1为空
vector<int> vec2(vec1);  //使用vec1初始化vec2
vector<int> vec3(vec1.begin(),vec1.end());//使用vec1初始化vec2
vector<int> vec4(10);    //10个值为0的元素
vector<int> vec5(10,4);  //10个值为4的元素
//add/remove
vec1.push_back(int) //尾部添加元素
vec1.insert(vec1.end(),5,3);    //从vec1.back位置插入5个值为3的元素
vec1.pop_back();              //删除末尾元素
vec1.erase(vec1.begin(),vec1.begin()+2);//删除vec1[0]-vec1[2]之间的元素，不包括vec1[2]其他元素前移

vector<int>::iterator iter = vec1.begin();    //获取迭代器首地址
vector<int>::const_iterator c_iter = vec1.begin();   //获取const类型迭代器

vec1.clear();                 //清空元素

//2. check, compare and size
vec1.size()
vec1.empty()
(vec1==vec2)

//3. read/write
vec1[0]

//4. 遍历
int length = vec1.size(); //下标法
for(int i=0;i<length;i++) {
    cout<<vec1[i];
}

vector<int>::iterator iter = vec1.begin(); //迭代器法
for(;iter != vec1.end();iter++) {
    cout<<*iter;
}
```

#deque<T>
deque与vector类似，但是对于首元素提供删除和插入的双向支持
deque还支持从开始端插入数据：push_front。其余的类似vector操作方法的使用.

# list<T>
表示非连续的内存，基于链表实现

```CPP
#include<list>
//1.定义和初始化
list<int> lst1;          //创建空list
list<int> lst2(3);       //创建含有三个元素的list
list<int> lst3(3,2); //创建含有三个元素为2的list
list<int> lst4(lst2);    //使用lst2初始化lst4
list<int> lst5(lst2.begin(),lst2.end());  //同lst4

//2.常用操作方法
lst1.assign(lst2.begin(),lst2.end());  //分配值,3个值为0的元素
lst1.push_back(10);                    //末尾添加值
lst1.pop_back();                   //删除末尾值
lst1.begin();                      //返回首值的迭代器
lst1.end();                            //返回尾值的迭代器
lst1.clear();                      //清空值
bool isEmpty1 = lst1.empty();          //判断为空
lst1.erase(lst1.begin(),lst1.end());                        //删除元素
lst1.front();                      //返回第一个元素的引用
lst1.back();                       //返回最后一个元素的引用
lst1.insert(lst1.begin(),3,2);         //从指定位置插入个3个值为2的元素
lst1.rbegin();                         //返回第一个元素的前向指针
lst1.remove(2);                        //相同的元素全部删除
lst1.reverse();                        //反转
lst1.size();                       //含有元素个数
lst1.sort();                       //排序
lst1.unique();                         //删除相邻重复元素
//3.遍历 迭代器法
for(list<int>::const_iterator iter = lst1.begin();iter != lst1.end();iter++)
{
    cout<<*iter;
}
```

# map<K, V>
key-value形式的,对于迭代器来说，可以修改实值，而不能修改key。map会根据key自动排序.

```CPP
#include<map>
//1.定义和初始化
map<int,string> map1;                  //空map

//2.常用操作方法
map1[3] = "Saniya";                    //添加元素
map1.insert(map<int,string>::value_type(2,"Diyabi"));//插入元素
//map1.insert(pair<int,string>(1,"Siqinsini"));
map1.insert(make_pair<int,string>(4,"V5"));
string str = map1[3];                  //根据key取得value，key不能修改
map<int,string>::iterator iter_map = map1.begin();//取得迭代器首地址
int key = iter_map->first;             //取得key
string value = iter_map->second;       //取得value
map1.erase(iter_map);                  //删除迭代器数据
map1.erase(3);                         //根据key删除value
map1.size();                       //元素个数
map1.empty();                       //判断空
map1.clear();                      //清空所有元素

//3.遍历
for(map<int,string>::iterator iter = map1.begin();iter!=map1.end();iter++)
{
   int keyk = iter->first;
   string valuev = iter->second;
}




```

# set
set的含义是集合，它是一个有序的容器，里面的元素都是排序好的支持插入、删除、查找等操作，就像一个集合一样，所有的操作都是严格在logn时间内完成，效率非常高。set和multiset的区别是：set插入的元素不能相同，但是multiset可以相同，set默认是自动排序的，使用方法类似list。




From:https://www.cnblogs.com/cxq0017/p/6555533.html

