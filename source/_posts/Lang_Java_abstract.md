---
title: Java abstract
date: 2016-05-28 00:00:00
tags: [abstract, 抽象, 继承]
categories: Java
---

抽象可以修饰抽象类和抽象方法。
抽象类一般作为基类，提炼了派生类们相同的接口特征(但基类中不实现而在派生类中实现)和属性。
抽象接口，所在类一般也为基类，只有该方法可暂时不进行实现，而在派生类中实现。

1. 抽象类不能被实例化。
2. 抽象类中不一定要包含abstrace方法。即：抽象中可以没有abstract方法。
3. 一旦类中包含了abstract方法，那类该类必须声明为abstract类。

# 示例讲解
/*Animal.java ------------------------------------------*/
package me.jingg.java.AbstractTest;

public abstract class Animal { //abstract类不能实例化；供外部 包内类继承
    protected String name;
    protected int age;

    private void eatBase() {}
    public void eat() {}             //派生类可重写
    public void run() {}             //派生类可重写
    public abstract void showInfo(); //+abstract: 不确定具体函数定义的可以只声明, 后面再派生类中定义
}

/*Cat.java ----------------------------------*/
package me.jingg.java.AbstractTest;

public class Cat {
    final private eat_fish_num;

    public static main(int argc, char **argv) {
        
    }
    
    public void run() { //重写
        System.out.println("cat run")
    }
    public void showInfo() { //定义
        System.out.println("I'am a cat")
    }

}







