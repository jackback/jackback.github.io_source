---
title: C++ virtual
date: 2016-05-28 11:22:33
tags: [vector,list,map,set]
categories: C++
---

C++ 中virtual 用于修饰函数，标记函数可以动态绑定

1. 虚函数
2. 纯虚函数(基类为抽象类)

# 虚函数表
编译器会给带虚函数的类添加一个虚函数表指针

```CPP
class Base {
public:
    virtual void v1() { cout << "Call Base::v1" << endl; }
    virtual void v2() { cout << "Call Base::v2" << endl; }
};

class Sub : public Base {
public:
    virtual void v2() { cout << "Call Sub::v2" << endl; }
    void v3()         { cout << "Call Sub::v3" << endl; }
};

int main () {
    //Only to explore principle
    cout << "size of Base: "<<sizeof(Base) << endl; //4 bytes

    typedef void(*Func)(void);
    Base b;
    Base *sPtr = new Sub();    //Sub的ptr

    long* vptr = (long*)(*(long*)sPtr) //Sub的ptr地址中第一个四字节是_vptr，即虚拟表指针。
    Func v1 = (Func)vptr[0];    //
    Func v2 = (Func)vptr[1];    //

    v1(); //Base::v1
    v2(); //Sub::v2
    
    //Normal code
    Base *bPtr = new Sub();
    bPtr->v1();   //静态调用基类v1
    bPtr->v2();   //动态调用派生类v2
    //bPtr->v3(); //编译都不能通过，基类没有v3成员，而且也没有开通动态绑定，无法定位到派生类的v3。
    
    return 0;
}
```

### 基类和派生类中虚函数内存示意图

单继承
[![003_CPP_vptr.png](https://i.loli.net/2018/12/22/5c1e0ba5a26af.png)](https://i.loli.net/2018/12/22/5c1e0ba5a26af.png)

多重继承
[![003_CPP_vptr2.png](https://i.loli.net/2018/12/22/5c1e15f7e4f75.png)](https://i.loli.net/2018/12/22/5c1e15f7e4f75.png)




From : https://www.cnblogs.com/hushpa/p/5707475.html


附：C++中的类所占内存空间
类所占内存的大小是由成员变量（静态变量除外）决定的，成员函数（这是笼统的说，后面会细说）是不计算在内的。
class CBase {}; //sizeof(CBase)=1； (C++要求每个实例在内存中都有独一无二的地址)
class CBase { int a; char p; }; //sizeof(CBase)=8; (4字节对齐)
class CBase { public: CBase(void); virtual ~CBase(void);  //sizeof(CBase)=12 (虚拟函数表指针_vptr占4个字节)
              private: int  a; char *p; };
class CSub : public CBase { public: CSub(void); ~CSub(void); virtual void test(); //sizeof(CSub)=16 (Base12+Sub4 以及共用_vptr)
                            private: int b; }; 

static修饰的静态变量：不占用内容，原因是编译器将其放在全局变量区。


