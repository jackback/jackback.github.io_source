---
title: C++的构造函数和析构函数
date: 2016-05-28 00:00:00
tags: [constructor, destructor]
categories: C++
---


构造函数调用顺序：Base::Constructor->Sub::Constructor 
析构函数调用顺序：Sub::Destructor->Base::Destructor
一个类中只能有一个默认构造函数（提供了无参数构造函数或者默认参数构造函数，都属于默认构造函数，只能选其一）

# 基本
#include <iostream>
using namespace std;

class Base {
public:
    Base()  { cout << "Base:Base()" << endl;}
    virtual ~Base() { cout << "Base:~Base()" << endl;}
    
    virtual void showInfo() { cout << "Base::showInfo()" << endl;}
};

class Sub : public Base {
private:
    int data;
public:
    //Sub() {data = 0;}                   //默认构造函数（只选其一）
    Sub(int data=4) {this->data = data; cout << "Sub:~Sub(int data=4)" << endl;} //默认构造函数（只选其一）
    void v3()     { cout << "Call Sub::v3, data = " << data << endl; }
    
    ~Sub() {cout << "Sub:~Sub()" << endl;}
    
    virtual void showInfo() { cout << "Sub::showInfo()" << endl;}
};

int main(int argc, char** argv) {

    Sub* s = new Sub();
    s->showInfo();
    delete s;
    
    Base* s2 = new Sub();
    s2->showInfo();
    delete s2; 
    /*如果基类析构函数不加virtual;这样的删除只能属于静态编译调用的是基类析构，
    所以只删除基类对象，而我们想要的是调用Sub类析构(Sub类析构会自动调用Base类析构)
    */
    return 0;
}

# 私有构造/析构函数
class singleton {
private:
   singleton()  {}
   ~singleton() {}

public:
    static singleton* GetSingleton(){
        return new singleton();
    }
};


