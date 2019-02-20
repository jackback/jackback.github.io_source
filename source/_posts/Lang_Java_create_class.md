---
title: Java中创建（实例化）对象的五种方式 及 基本数据类型
date: 2016-09-30 12:22:33
tags: [class, instance, reflect]
categories: Java
---

首先，Java中创建（实例化）对象的五种方式，如下
1、用new语句创建对象，这是最常见的创建对象的方法。
2、通过工厂方法返回对象，如：String str = String.valueOf(23); 
3、运用反射手段,调用java.lang.Class或者java.lang.reflect.Constructor类的newInstance()实例方法。如：Object obj = Class.forName("java.lang.Object").newInstance(); 
4、调用对象的clone()方法。
5、通过I/O流（包括反序列化），如运用反序列化手段，调用java.io.ObjectInputStream对象的 readObject()方法。



# 示例

    package org.whb.test.demo724;
    /*
     * 测试Cloneable接口的使用
     * 包含第一种和第三种方法clone()
     * 不过要注意在clone()中深复制和潜复制的理解
     * 实例化对象 
     */
    class Person implements Cloneable{
        private String name;
        private int age;
     
      public Person( String name,int age) {
        this.name = name; 
        this.age = age;
      }
     
      public int getAge() {
       return age;
      }
      
      public void setAge(int age) {
       this.age = age;
      }
     
     public String getName() {
      return name;
     }
     
     public void setName(String name){
          this.name =name;
        }
 
    @Override
     public Object clone() throws CloneNotSupportedException {
      // TODO Auto-generated method stub
      return super.clone();
     }
      @Override
     public String toString() {
      // TODO Auto-generated method stub
      return "姓名是："+name+"; 年龄是："+age;
     }
       
    }

    public class TestClone{
     public static void main(String[] args){
       Person p1 = new Person("王豪博",25);
       System.out.println(p1);
       Person p2 =null;
       try {
         p2 = (Person)p1.clone();
       } catch (CloneNotSupportedException e) {
        // TODO Auto-generated catch block
         e.printStackTrace();
       }
       p2.setName("春香");
       p2.setAge(24);
       System.out.println(p2);
      }
    }
    
    /*
     * 通过反射对对象进行初始化
     * 注意必须有无参数的Constructor
     * 实例化Class类然后调用newInstance()方法
     */
    package org.whb.test.demo715;
    class Person{
        private int age;
        private String name;

        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public String toString(){
            return "年龄是："+this.age+"  姓名是："+this.name;
        } 

    }

    public class TestClass {
        public static void main(String[] args){
            Class< ?> c1 = null;
            try{
                c1 = Class.forName("org.whb.test.demo715.Person");
            }catch(ClassNotFoundException e){
                e.printStackTrace();
            }   
            Person p1 = null;
            try {
                p1 =(Person)c1.newInstance();
            } catch (InstantiationException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
     
            p1.setAge(12);
            p1.setName("haobo");
            System.out.println(p1);
        }
    }


    /*
     * 对象的序列化和反序列化测试类.

        *1、序列化是干什么的？
        简单说就是为了保存在内存中的各种对象的状态（也就是实例变量，不是方法），并且可以把保存的对象状态再读出来。虽然你可以用你自 己的各种各样的方法来保存object states，但是Java给你提供一种应该比你自己好的保存对象状态的机制，那就是序列化。
        *2、什么情况下需要序列化 
        a）当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；
        b）当你想用套接字在网络上传送对象的时候；
        c）当你想通过RMI传输对象的时候；
        *
        *3、相关注意事项
        a）序列化时，只对对象的状态进行保存，而不管对象的方法；
        b）当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；
        c）当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；
        d）并非所有的对象都可以序列化，,至于为什么不可以，有很多原因了,比如：
        1.安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行rmi传输 等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的。
        2.资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分 配，而且，也是没有必要这样实现。

    */

    package org.whb.test.demo724;
    import java.io.*; 
    import java.util.Date;

    public class ObjectSaver { 
        public static void main(String[] args) throws Exception { 
            ObjectOutputStream out = new ObjectOutputStream (new FileOutputStream("D:/objectFile.swf"));

            //序列化对象 
            Customer customer = new Customer("haobo", 24); 
            out.writeObject("你好!"); 
            out.writeObject(new Date()); 
            out.writeObject(customer); 
            out.writeInt(123); //写入基本类型数据 
            out.close(); 

            //反序列化对象 
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("D:/objectFile.swf")); 
            System.out.println("obj1=" + (String) in.readObject()); 
            System.out.println("obj2=" + (Date) in.readObject()); 
            Customer obj3 = (Customer) in.readObject(); 
            System.out.println("obj3=" + obj3); 
            int obj4 = in.readInt(); 
            System.out.println("obj4=" + obj4); 
            in.close(); 
        } 
    } 
    
    class Customer implements Serializable { 
        private static final long serialVersionUID = -88175599799432325L; 
        private String name; 
        private int age; 
        public Customer(String name, int age) { 
            this.name = name; 
            this.age = age; 
        }
        public String toString() { 
            return "name=" + name + ", age=" + age; 
        } 
    }

    /* 打印输出
     * obj1=你好!
     * obj2=Sat Jul 24 21:18:19 CST 2010
     * obj3=name=haobo, age=24
     * obj4=123
     */


[![basic_type.png](https://i.loli.net/2018/12/15/5c14f2f89941e.png)](https://i.loli.net/2018/12/15/5c14f2f89941e.png)



# 附（C++中实例化对象）
A *a = new A; 和 A *a=new A(); （推荐使用这个）都是调用A类的默认构造函数，指针a在函数栈中，指向内容在堆中。
但是如果单独声明一个A 类变量，如：A a；则调用的是默认构造函数，对象a整个在函数栈中。
但是不能写成 A a(); 来调用默认构造函数！！因为这种形式会被识别成一个：名称为a的不接受任何参数，返回值为A类型的函数！
