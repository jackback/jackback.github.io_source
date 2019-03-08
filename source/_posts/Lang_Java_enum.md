---
title: Java enum
date: 2017-01-01 00:00:00
tags: enum
categories: Java
---


### 1. enum 基本概念和用法

enum Color { RED,BLUE,BLACK YELLOW,GREEN}; //创建时自动继承java.lang.Enum这个类

       (1)  ordinal()方法: 返回枚举值在枚举类种的顺序。这个顺序根据枚举值声明的顺序而定。  
                 Color.RED.ordinal();  //返回结果：0
                 Color.BLUE.ordinal();  //返回结果：1
       (2)  compareTo()方法: Enum实现了java.lang.Comparable接口，因此可以比较象与指定对象的顺序。Enum中的compareTo返回的是两个枚举值的顺序之差。
                             当然，前提是两个枚举值必须属于同一个枚举类，否则会抛出ClassCastException()异常。(具体可见源代码)
                 Color.RED.compareTo(Color.BLUE);  //返回结果 -1
       (2)  equals()方法： 比较两个枚举类对象的引用。
       (3.1)  values()方法： 静态方法，返回一个包含全部枚举值的数组。
                 Color[] colors=Color.values();
                 for(Color c:colors){ System.out.print(c + ","); }//返回结果：RED,BLUE,BLACK YELLOW,GREEN,
       (3.2)  toString()方法： 返回枚举常量的名称。
                 Color c=Color.RED;
                 System.out.println(c);//返回结果: RED
       (3.2)  name()方法: 返回枚举实例声明时的名字String //"RED","BLUE","BLACK" "YELLOW","GREEN"
       (5)  valueOf()方法： 这个方法和toString方法是相对应的，返回带指定名称的指定枚举类型的枚举常量。
                 Color.valueOf("BLUE");   //返回结果: Color.BLUE
       (8)  getDeclaringClass()方法: 返回该enum变量所属Class<E>

    总结：

    1. enum -> int:    
        int i =  enumValue.ordinal(); // enumValue = Color.RED;
        int i = Color.ordinal();

    2. int -> enum:    Color b = Color.values()[i]; // int i = 0 1 2 3 4;

    3. enum -> String:   
        String s = enumValue.name();    //enumValue = Color.RED;
        String s = enumValue.toString(); //enumValue = Color.RED;
        String s = Color.RED.name();
        String s = Color.RED.toString();
    4. String -> enum:   enumType.valueOf(name); //name = "RED"

### 2. enum类实现原理及扩展
final class enum Color extends java.lang.Enum {
    enum RED("红色"),    //自带实例,在类前部定义
    enum BLUE("蓝色"),   //自带实例,在类前部定义
    enum BLACK("黑色"),  //自带实例,在类前部定义
    enum YELLOW("黄色"), //自带实例,在类前部定义
    enum GREEN("绿色");  //自带实例,在类前部定义,最后一个需要加分号

    //todo 可新增成员变量和成员方法 及 构造函数
    private Color (String description) { //此处需要声明为private,就算不声明,只用于自带实例进行构建,在外部不能构建???
        this.description = description;
    }
    private String description;
    public String getDescr() { return description;}
    public static void main(String[ args]) {
        for (Color c : Color.values()) {
            System.out.println(c + ": " + c.getDescr());
        }
    }

}; 

    //Output:
    RED: 红色
    BLUE: 蓝色
    BLACK: 黑色
    YELLOW: 黄色
    GREEN: 绿色




