---
title: Java反射：getClass   x.class Class.forName
date: 2016-05-27 15:13:00
tags: [reflect]
categories: Android
---

# 1. class装载验证流程
Class的装载过程（也就是从字节码文件到生成类的实例这个过程）分为三个阶段
loading（装载），linking（连接）和initializing（实例化）

[![Java_reflect.png](https://i.loli.net/2018/12/15/5c14f2f8c25fb.png)](https://i.loli.net/2018/12/15/5c14f2f8c25fb.png)

### 1.1 加载
装载类的第一个阶段
通过类的全限定名(eg: com.company.Employee)取得类的二进制流
转为方法区数据结构
在Java堆中生成对应的java.lang.Class对象

### 1.2 链接 -> 验证
目的：保证Class流的格式是正确的

文件格式的验证

是否以0xCAFEBABE开头
版本号是否合理
元数据验证

是否有父类
继承了final类？
非抽象类实现了所有的抽象方法
字节码验证 (很复杂)

运行检查
栈数据类型和操作码数据参数吻合
跳转指令指定到合理的位置
符号引用验证

常量池中描述类是否存在
访问的方法或字段是否存在且有足够的权限


### 1.3 链接 -> 准备

分配内存，并为类设置初始值 （方法区中，关于方法区请查看Java内存区域）

public static int v=1;
在准备阶段中，v会被设置为0
在初始化的<clinit>中才会被设置为1
对于static final类型（常量），在准备阶段就会被赋上正确的值
public static final  int v=1;


###1.4 链接 -> 解析

### 1.5 初始化

执行类构造器<clinit>

static变量 赋值语句
static{}语句

# 2. 什么是类装载器ClassLoader

ClassLoader是一个抽象类
ClassLoader的实例将读入Java字节码将类装载到JVM中
ClassLoader可以定制，满足不同的字节码流获取方式（譬如从网络中加载，从文件中加载）
ClassLoader负责类装载过程中的加载阶段


# 3. Android中java反射使用
Java的每个类必需被JVM加载到虚拟机中，然后就有一个运行时类对象，该Class对象中保存了创建对象所需的所有信息。


### 方法1.1：用于类提前已被加载，通过全限定名来获得类，法1——xxx.class
可以用.class返回此 Object 的运行时类Class对象，如：   //.class一般用于获得类型，JVM中加载已经加载过的类
Class<?> clazz = com.android.systemui.statusbar.SystemBars.class    
clazz.newInstance();   //实例化

### 方法1.2：用于类提前已被加载，通过实例对象来获取类，法2——getClass
也可以用getClass()获得。  //
获得此对象后可以利用此Class对象的一些反射特性进行操作，
例如：//使用前类已加载
Class<?> clazz = this.getClass(); //用缺省构造函数创建一个该类的Class<?>对象
clazz.newInstance();  //用缺省构造函数创建一个该类的对象
clazz.getInterfaces(); //获得此类实现的接口信息
clazz.getMethods();  //获得此类实现的所有公有方法

### 方法2： 加载类到JVM中(使用全限定名)，并初始化 ——Class.forName
Class<?> clazz = Class.forName("类名，如'java.lang.Thread' "); //动态加载类，并返回具有指定名的类的 Class 对象
在加载完成后，一般还要调用Class下的newInstance( )静态方法来实例化对象以便操作

Class.forName有两个调用方法
    1. 常用的 Class.forName("xx.xx")      //查找并加载类，最后执行了类static代码
    2. 多参数 Class.forName("xx.xx",true,CALLClass.class.getClassLoader())      //可指定是否初始化static语句，及指定类加载器

### 方法3：加载类到JVM(使用全限定名)，但不初始化——loadClass
Class<?> clazz = loader.loadClass("xx.xx");  或者mContext.getClassLoader().loadClass(clsName); 都只是加载了类，但是没有执行类静态代码
使用loadClass只有执行clazz.NewInstance()才能够初始化类




# 参考
http://blog.csdn.net/sunyujia/article/details/2501709











