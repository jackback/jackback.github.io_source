---
title: C++的类型转换——隐式、显式转换(xxx_cast)
date: 2016-05-28 12:22:33
tags: [类型转换, static_cast, dynamic_cast, reinterpret_cast, const_cast]
categories: C++
---

# 第一部分. 隐式类型转换
    算数运算float + int => float  、  赋值int = float、传参、返回值


# 第二部分.显式类型转换
    C风格
        (type) value

    C++风格
        static_cast、dynamic_cast、reinterpret_cast和const_cast 

<!-- more -->

### static_cast<type-id> (expression)
    该运算符把expression转换为type-id类型，但没有运行时类型检查来保证转换的安全性，如下几种用法：
    a. 用于类层次结构中基类和子类之间指针或引用的转换。
        进行上行转换（把子类的指针或引用转换成基类表示）是安全的；
        进行下行转换（把基类指针或引用转换成子类指针或引用）时，由于没有动态类型检查，所以是不安全的。
    b. 用于基本数据类型之间的转换，如把int转换成char，把int转换成enum。这种转换的安全性也要开发人员来保证。
    c. 把void指针转换成目标类型的指针(不安全!!)
    e. 把任何类型的表达式转换成void类型。

    注意：static_cast不能转换掉expression的const、volatile、或者__unaligned属性。

    来源：为什么需要static_cast强制转换？
    情况1：void指针->其他类型指针
    情况2：改变通常的标准转换
    情况3：避免出现可能多种转换的歧义

### dynamic_cast<type-id> (expression)
    该运算符把expression转换成type-id类型的对象。Type-id必须是类的指针、类的引用或者void *；如果type-id是类指针类型，那么expression也必须是一个指针，如果type-id是一个引用，那么expression也必须是一个引用。
    dynamic_cast主要用于类层次间的上行转换和下行转换，还可以用于类之间的交叉转换。在类层次间进行上行转换时，dynamic_cast和static_cast的效果是一样的；在进行下行转换时，dynamic_cast具有类型检查的功能，比static_cast更安全。

    class Base{
    public:
        int m_iNum;
        virtual void foo();
    };
    class Derived:public Base{
    public:
        char *m_szName[100];
    };
    void func(Base *pb){  //pb可能是Base or Derived
        Derived *pd1 = static_cast<Derived *> (pb);
        Derived *pd2 = dynamic_cast<Derived *> (pb);
    }

    在上面的代码段中，如果pb实际指向一个Derived类型的对象，pd1和pd2是一样的，并且对这两个指针执行Derived类型的任何操作都是安全的；
    如果pb实际指向的是一个Base类型的对象，那么pd1将是一个指向该对象的指针，对它进行Derived类型的操作将是不安全且非法的（如访问m_szName），而pd2将是一个空指针(即0，因为dynamic_cast失败)。
    另外要注意：
    dynamic_cast 转换时Base要有虚函数，否则会编译出错；static_cast则没有这个限制。这是由于运行时类型检查需要运行时类型信息，而这个信息存储在类的虚函数表（关于虚函数表的概念，详细可见<Inside c++ object model>）中，只有定义了虚函数的类才有虚函数表，没有定义虚函数的类是没有虚函数表的。
    
    dynamic_cast还支持交叉转换（cross cast）。如下代码所示。
    class Base {
    public:
        int m_iNum;
        virtual void f(){}
    };
    class Derived1 : public Base{
    };
    class Derived2 : public Base {
    };
    void foo() {
        Derived1 *pd1 = new Drived1();
        pd1->m_iNum = 100;
        Derived2 *pd2 = static_cast<Derived2 *>(pd1);   //Compile error
        Derived2 *pd2 = dynamic_cast<Derived2 *>(pd1);  //pd2 is NULL
        delete pd1;
    }
    在函数foo中，使用static_cast进行转换是不被允许的，将在编译时出错；而使用 dynamic_cast的转换则是允许的，结果是空指针。

    来源：为什么需要dynamic_cast强制转换？
    简单的说，当无法使用virtual函数的时候

### reinterpret_cast<type-id> (expression) 
    是C++中强制类型转换符，操作符修改了操作数类型，但仅仅是重新解释了给出的对象的比特模型而没有进行二进制转换。eg: void *p ===> int p
    type-id必须是一个指针、引用、算术类型、函数指针或者成员指针。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，再把该整数转换成原类型的指针，还可以得到原先的指针值）。

    reinterpret_cast是为了映射到一个完全不同类型的意思，这个关键词在我们需要把类型映射回原有类型时用到它。


### const_cast<type_id> (expression)
    该运算符用来修改类型的const或volatile属性。除了const 或volatile修饰之外， type_id和expression的类型是一样的。
    常量指针被转化成非常量指针，并且仍然指向原来的对象；
    常量引用被转换成非常量引用，并且仍然指向原来的对象；
    常量对象被转换成非常量对象。
    Voiatile和const类试。举如下一例：
    class B{
    public:
        int m_iNum;
    }
    void foo(){
        const B b1;
        b1.m_iNum = 100; //comile error
        B b2 = const_cast<B>(b1);  // 去掉const 属性
        b2. m_iNum = 200; //fine
    }
    上面的代码编译时会报错，因为b1是一个常量对象，不能对它进行改变；
    使用const_cast把它转换成一个非常量对象，就可以对它的数据成员任意改变。注意：b1和b2是两个不同的对象。




### 比较static_cast Vs. reinterpret_cast
    reinterpret_cast是为了映射到一个完全不同类型的意思，这个关键词在我们需要把类型映射回原有类型时用到它。我们映射到的类型仅仅是为了故弄玄虚和其他目的，这是所有映射中最危险的。(这句话是C++编程思想中的原话)

    static_cast 和 reinterpret_cast 操作符修改了操作数类型。它们不是互逆的； static_cast 在编译时使用类型信息执行转换，在转换执行必要的检测(诸如指针越界计算, 类型检查). 其操作数相对是安全的。另一方面；reinterpret_cast 仅仅是重新解释了给出的对象的比特模型而没有进行二进制转换，例子如下：
    int n=9; double d=static_cast < double > (n);
    上面的例子中, 我们将一个变量从 int 转换到 double。这些类型的二进制表达式是不同的。 要将整数 9 转换到 双精度整数 9，static_cast 需要正确地为双精度整数 d 补足比特位。其结果为 9.0。而reinterpret_cast 的行为却不同:
    int n=9;
    double d=reinterpret_cast<double & > (n);
    这次, 结果有所不同. 在进行计算以后, d 包含无用值. 这是因为 reinterpret_cast 仅仅是复制 n 的比特位到 d, 没有进行必要的分析.
    因此, 你需要谨慎使用 reinterpret_cast.


    C++primer第五章里写了编译器隐式执行任何类型转换都可由static_cast显示完成;reinterpret_cast通常为操作数的位模式提供较低层的重新解释
    1、C++中的static_cast执行非多态的转换，用于代替C中通常的转换操作。因此，被做为隐式类型转换使用。比如： 
    int i; 
    float f = 166.7f; 
    i = static_cast<int>(f); 
    此时结果，i的值为166。 
    2、C++中的reinterpret_cast主要是将数据从一种类型的转换为另一种类型。所谓“通常为操作数的位模式提供较低层的重新解释”也就是说将数据以二进制存在形式的重新解释。比如：
    int i; 
    char *p = "This is a example."; 
    i = reinterpret_cast<int>(p); 
    此时结果，i与p的值是完全相同的。reinterpret_cast的作用是说将指针p的值以二进制（位模式）的方式被解释为整型，并赋给i，一个明显的现象是在转换前后没有数位损失。

  
    

C++的类型转换明显比C里面的用()的类型转换要严谨很多。首先，C++样式的类型转换编写 的代码更容易维护，想想看，只要搜索“static_cast”，就可以找出代码中所有使用了静态转换的代码，比C里面五花八门的 (long) (long*) (double )好找多了吧。另外，reinterpret_cast，看字面意思就知道，是保持二进制位不变，用另一种格式来重新解释，我们也知道计算机里面只有 bit,到底代表什么类型，整形还是浮点型，全在于应用程序怎么解释每一个bit，reinterpret_cast给了我们这么一个机会。而C里面要想 二进制重解释，你要么拷贝内存，要么还是用()来强制类型转换，某些特定情况下，()强制转换其实是二进制的重解释，毫无疑问，C风格的转换会让代码变得 难懂。