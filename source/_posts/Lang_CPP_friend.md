---
title: C++ 友元
date: 2016-05-28 12:22:33
tags: [friend]
categories: C++
---

C++ 友元
友元机制，开个后门，以让友元函数、类可以访问类内部私有成员。不过这个机制也破坏了类的封装性。


# 1. 友元函数（这种函数是类外部定义函数）
友元函数是类外部定义的函数，而将函数声明加friend放在需要访问的类的内部。

#include <iostream>
using namespace std;
class A
{
public:
    friend void set_show(int x, A &a);      //该函数是友元函数的声明
private:
    int data;
};

void set_show(int x, A &a)  //友元函数定义，为了访问类A中的成员
{
    a.data = x;
    cout << a.data << endl;
}

int main(void)
{
    class A a;

    set_show(1, a);

    return 0;
}


# 2. 友元类
友元类的所有成员函数都是另一个类的友元函数。声明加friend放在需要访问的类的内部。

关于友元类的注意事项：
(1) 友元关系不能被继承。
(2) 友元关系是单向的，不具有交换性。若类B是类A的友元，类A不一定是类B的友元，要看在类中是否有相应的声明。
(3) 友元关系不具有传递性。若类B是类A的友元，类C是B的友元，类C不一定是类A的友元，同样要看类中是否有相应的申明。

#include <iostream>
using namespace std;

class A
{
public:
    friend class C;                         //这是友元类的声明
private:
    int data;
};

class C             //友元类定义，为了访问类A中的成员
{
public:
    void set_show(int x, A &a) { a.data = x; cout<<a.data<<endl;}
};

int main(void)
{
    class A a;
    class C c;

    c.set_show(1, a);

    return 0;
}


# 3. 友元成员函数

类A的成员函数是类B的友元函数。

class B;

class A {
  public:
    friend void B::B_is_A_friend_function(A & tempA);
    
  private:
    int a;
};

class B{
  public:
    void B::B_is_A_friend_function(A & tempA) {
        cout << tempA.a << endl;
    }
};





















