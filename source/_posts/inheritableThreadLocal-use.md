---
title: 3.4 类InheritableThreadLocal的使用
date: 2017-07-23 16:11:23
tags: [java,threadLocal,读书笔记]
categories: [java,读书笔记]
---

使用类InheritableThreadLocal可以在子线程中取得父线程继承下来的值

**3.4.1 值继承**
使用InheritableThreadLocal类可以让子线程从父线程中取得值。
```java
/**
 * @author wuyoushan
 * @date 2017/5/24.
 */
public class InheritableThreadLocalExt extends InheritableThreadLocal {

    @Override
    protected Object initialValue() {
        return new Date().getTime();
    }
}

/**
 * @author wuyoushan
 * @date 2017/5/23.
 */
public class Tools {
    public static  InheritableThreadLocalExt t1=new InheritableThreadLocalExt();
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
在Main线程中取值=1495584062991
在Main线程中取值=1495584062991
在Main线程中取值=1495584062991
在ThreadA线程中取值= 1495584062991
在ThreadA线程中取值= 1495584062991
在ThreadA线程中取值= 1495584062991
```

**3.4.2 值继承再修改**
如果在值继承的同时还可以对值进行进一步的处理那就更好了。
```java
/**
 * @author wuyoushan
 * @date 2017/5/24.
 */
public class InheritableThreadLocalExt extends InheritableThreadLocal {

    @Override
    protected Object initialValue() {
        return new Date().getTime();
    }

    @Override
    protected Object childValue(Object parentValue) {
        return parentValue+" 我在子线程加的~";
    }
}

/**
 * @author wuyoushan
 * @date 2017/5/23.
 */
public class Tools {
    public static  InheritableThreadLocalExt t1=new InheritableThreadLocalExt();
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
在Main线程中取值=1495584458175
在Main线程中取值=1495584458175
在Main线程中取值=1495584458175
在ThreadA线程中取值= 1495584458175 我在子线程加的~
在ThreadA线程中取值= 1495584458175 我在子线程加的~
在ThreadA线程中取值= 1495584458175 我在子线程加的~
```
但在使用InheritableThreadLocal类需要注意一点的是，如果子线程在取得值的同时，主线程将InheritableThreadLocal中的值进行更改，那么子线程取到的值还是旧值

> 摘选自 java多线程核心编程技术-3.4