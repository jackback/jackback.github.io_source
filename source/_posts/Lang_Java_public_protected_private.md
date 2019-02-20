---
title: Java public/protected/private
date: 2016-05-28 00:00:00
tags: [public,protected,private]
categories: Java
---

用于修饰类或者修饰成员变量。

# 1. 基本规则

### public(公有访问权限)
  修饰类成员变量/方法        //基类和派生类可见
  修饰类                     //公共类，所有包可见
  ?修饰内部类?

### protected(继承访问权限)
  修饰类成员变量/方法        //基类和派生类可见
  修饰内部类                 //基类和派生类可见；包内可见

### default (friendly) (包访问权限)
  默认权限是包内访问。即未指定权限关键字的类成员，只有包内的类的成员方法才可以访问。
  类无修饰: 包内可见
  成员变量/方法无修饰: 包内可见
  内部类无修饰: 包内可见?

### private(私有访问权限)
  修饰类成员变量/方法        //基类可见
  修饰内部类                 //基类可见

### 继承派生权限
  Java 使用extends关键字进行派生基类，只有公共继承



# 1. public 用法示例
/*Animal.java*/
package me.jingg.java;         //Java 以包为访问单元(一个包可有多个文件多个类)

public abstract class Animal { //公共类: 可以通过import me.jingg.java.Animal 继承使用
    protected String name;
    protected int age;

    private void eatBase() {}
    public void eat() {}
    public void run() {}
    public void showInfo() {}
}

/*AccessRule.java*/
package me.jingg.java; 
//一个Java文件只能有一个public类(public的类名必须与文件名一致)，一个Java文件是一个编译单元，每个编译单元都有单一的公共接口---即public类
public class AccessRule extends Animal implements xxx {  //公共类: 可以通过import me.jingg.java.AccessRule 引用
    private eat_fish_num; //static一般在有初始值的，值固定的 (private表示私有) (final: 1.不能再改;2.不会被重载; 3.不会被继承.)
    
    
    public static void main(int argc, char **argv) {
        
    }

    
}

# 2. 抽象接口

































