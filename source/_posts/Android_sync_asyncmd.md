---
title: Android 中同步和并发实现的几个方式
date: 2016-10-21 15:13:00
tags: [ synchronized, ReentrantLock]
categories: Android
---

# 1. 关键字synchronized 
实例方法同步/静态方法同步
class Demo{ 
    public synchronized void print(){ //实例方法
        System.out.println("print");
    }
    public static synchronized void print2(){ //静态方法
        System.out.println("print2");
    }
}
Demo demo1;    //同一时间段只能有一个线程可以访问demo1.print   
Demo demo2;    //可以同时访问demo1.print  和 demo2.print
                           //同一时间段只能有一个线程可以访问demo1.print2 或者 demo2.print2



实例方法中同步块    //分为对象锁和全局的Class锁，
静态方法中同步块    //只有全局锁

static synchronized方法也相当于全局锁synchronized(xxx.class)   如果{}范围相同的话
synchronized方法也相当于synchronized(this)    如果{}范围相同的话


# 2. 类java.util.concurrent.locks.ReentrantLock  （比synchronized 功能更全更灵活， 还多了锁投票，定时锁等候和中断锁）

lock()         如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁
tryLock()    如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false；
tryLock(long timeout,TimeUnit unit)    
                  如果获取了锁定立即返回true，如果别的线程正持有锁，会等待参数给定的时间，在等待的过程中，如果获取了锁定，就返回true，如果等待超时，返回false；
lockInterruptibly    
                  如果获取了锁定立即返回，如果没有获取锁定，当前线程处于休眠状态，直到或者锁定，或者当前线程被别的线程中断


异同：
synchronized是在JVM层面上实现的，不但可以通过一些监控工具监控synchronized的锁定，而且在代码执行时出现异常，JVM会自动释放锁定，但是使用Lock则不行，lock是通过代码实现的，要保证锁定一定会被释放，就必须将unLock()放到finally{}中

在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态；



# 3. AtomicBoolean、AtomicInteger、AtomicLong、AtomicReference: 
在32位操作系统中，64位的long 和 double 变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。
区别：
和上面的类似，不激烈情况下，性能比synchronized略逊，而激烈的时候，也能维持常态。激烈的时候，Atomic的性能会优于ReentrantLock一倍左右。但是其有一个缺点，就是只能同步一个值，一段代码中只能出现一个Atomic的变量，多于一个同步无效。因为他不能在多个Atomic之间同步。 



# 4. 特殊域变量(volatile)实现线程同步
    a.volatile关键字为域变量的访问提供了一种免锁机制， 
    b.使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新， 
    c.因此每次使用该域就要重新计算，而不是使用寄存器中的值 
    d.volatile不会提供任何原子操作，它也不能用来修饰final类型的变量 


# 5. 使用局部变量实现线程同步

如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本， 
    副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。



明天看下ThreadLocal
