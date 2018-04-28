---
title: 3.3类ThreadLocal的使用
date: 2017-07-23 16:06:54
tags: [java,threadLocal,读书笔记]
categories: [java,读书笔记]
---
变量值的共享可以使用public static变量的形式，所有的线程都使用同一个public static变量。如果想实现每一个线程都有自己的共享变量该如何解决呢？JDK中提供的类ThreadLocal正是为了解决这样的问题。

类ThreadLocal主要解决的就是每个线程绑定自己的值，可以将ThreadLocal类比喻成全局存放数据的盒子，盒子中可以存储每个线程的私有数据

**3.3.1 方法get()与null**
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static ThreadLocal t1=new ThreadLocal();

    public static void main(String[] args) {
        if(t1.get()==null){
            System.out.println("从未放过值");
            t1.set("我的值");
        }
        System.out.println(t1.get());
        System.out.println(t1.get());
    }
}
```
程序的运行结果:
```java
从未放过值
我的值
我的值
```
从程序的运行结果来看，第一调用t1对象的get()方法时返回的值是null，通常调用set()方法赋值后顺利取出值并打印到控制台上。类ThreadLocal解决的是变量在不同线程间的隔离性，也就是不同线程拥有自己的值，不同线程中的值是可以放入ThreadLocal类中进行保存的。

**3.3.2  验证线程变量的隔离性**
```java
/**
 * @author wuyoushan
 * @date 2017/5/23.
 */
public class Tools {
    public static  ThreadLocal t1=new ThreadLocal();
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    @Override
    public void run() {
       try{
           for (int i = 0; i <100; i++) {
                 Tools.t1.set("ThreadA"+(i+1));
               System.out.println("ThreadA get Value="+Tools.t1.get());
               Thread.sleep(200);
           }
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    @Override
    public void run() {
        try{
            for (int i = 0; i <100; i++) {
                Tools.t1.set("ThreadB"+(i+1));
                System.out.println("ThreadB get Value="+Tools.t1.get());
                Thread.sleep(200);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    public static void main(String[] args) {
        try {
            ThreadA a=new ThreadA();
            ThreadB b=new ThreadB();
            a.start();
            b.start();
            for (int i = 0; i < 100; i++) {
                Tools.t1.set("Main"+(i+1));
                System.out.println("Main get Value="+Tools.t1.get());
                Thread.sleep(200);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
Main get Value=Main97
ThreadB get Value=ThreadB97
ThreadA get Value=ThreadA97
Main get Value=Main98
ThreadB get Value=ThreadB98
ThreadA get Value=ThreadA98
Main get Value=Main99
ThreadB get Value=ThreadB99
ThreadA get Value=ThreadA99
Main get Value=Main100
ThreadB get Value=ThreadB100
ThreadA get Value=ThreadA100
```
虽然3个线程都向t1对象中set()数据值，但每个线程还是能取出自己的数据。

```java
/**
 * @author wuyoushan
 * @date 2017/5/23.
 */
public class Tools {
    public static  ThreadLocal<Date> t1=new ThreadLocal();
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    @Override
    public void run() {
       try{
           for (int i = 0; i <100; i++) {
               if(Tools.t1.get()==null){
                   Tools.t1.set(new Date());
               }
               System.out.println("A "+Tools.t1.get().getTime());
               Thread.sleep(100);
           }
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    @Override
    public void run() {
        try{
            for (int i = 0; i <100; i++) {
                if(Tools.t1.get()==null){
                    Tools.t1.set(new Date());
                }
                System.out.println("B "+Tools.t1.get().getTime());
                Thread.sleep(100);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    public static void main(String[] args) {
        try {
            ThreadA a=new ThreadA();
            a.start();
            Thread.sleep(1000);
            ThreadB b=new ThreadB();
            b.start();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
A 1495499916793
A 1495499916793
B 1495499917792
A 1495499916793
A 1495499916793
B 1495499917792
A 1495499916793
B 1495499917792
B 1495499917792
A 1495499916793
```
运行结果只有两种时间。

在第一次调用ThreadLocal类的get()方法返回值是null，怎么样实现第一次调用get()不返回null呢？也就是具有默认值的效果。

**3.3.3 解决get()返回null问题**
```java
/**
 * @author wuyoushan
 * @date 2017/5/24.
 */
public class ThreadLocalExt extends ThreadLocal {

    @Override
    protected Object initialValue() {
        return "我是默认值，第一次get不再为null";
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    private static ThreadLocalExt t1=new ThreadLocalExt();

    public static void main(String[] args) {
        if(t1.get()==null){
            System.out.println("从未放过值");
            t1.set("我的值");
        }
        System.out.println(t1.get());
        System.out.println(t1.get());
    }
}
```
程序的运行结果为:
```java
我是默认值，第一次get不再为null
我是默认值，第一次get不再为null
```
此案例仅仅证明main线程有自己的值，那其他线程是否会有自己的初始值呢？

**3.3.4 再次验证线程变量的隔离性**
```java
/**
 * @author wuyoushan
 * @date 2017/5/23.
 */
public class Tools {
    public static  ThreadLocalExt t1=new ThreadLocalExt();
}

/**
 * @author wuyoushan
 * @date 2017/5/24.
 */
public class ThreadLocalExt extends ThreadLocal {

    @Override
    protected Object initialValue() {
        return new Date().getTime();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    @Override
    public void run() {
       try{
           for (int i = 0; i <10; i++) {
               System.out.println("在ThreadA线程中取值= "+Tools.t1.get());
               Thread.sleep(100);
           }
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    public static void main(String[] args) {
        try {
            for (int i = 0; i < 10; i++) {
                System.out.println("在Main线程中取值="+Tools.t1.get());
                Thread.sleep(100);
            }
            Thread.sleep(5000);
            ThreadA a=new ThreadA();
            a.start();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
在Main线程中取值=1495583100820
在Main线程中取值=1495583100820
在Main线程中取值=1495583100820
在Main线程中取值=1495583100820
在ThreadA线程中取值= 1495583106829
在ThreadA线程中取值= 1495583106829
在ThreadA线程中取值= 1495583106829
在ThreadA线程中取值= 1495583106829
```
子线程和父线程各有各自所拥有的值

> 摘选自 java多线程核心编程技术-3.3