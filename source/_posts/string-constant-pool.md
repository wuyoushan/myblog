---
title: 2.2.10数据类型String的常量池特性
date: 2017-07-23 13:52:30
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
    public static void print(String stringParam){
        try{
            synchronized (stringParam) {
                while (true) {
                    System.out.println(Thread.currentThread().getName());
                    Thread.sleep(1000);
                }
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
       super.run();
        service.print("AA");
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

   private Service service;

    public ThreadB(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.print("AA");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      Service service=new Service();
      ThreadA a=new ThreadA(service);
      a.setName("A");
      a.start();
      ThreadB b=new ThreadB(service);
      b.setName("B");
      b.start();
    }
}
```
程序的运行结果为：
```java
A
A
A
A
```
出现这样的情况就是因为String的两个值都是AA，两个线程持有相同的锁，所以造成线程B不能执行。这就是String常量池所带来的问题。因此在大多数情况下，同步synchronized代码块都不使用String作为锁对象，而改用其他，比如new Object()实例化一个Object对象，但它并不放入缓存中。

```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
    public static void print(Object object){
        try{
            synchronized (object) {
                while (true) {
                    System.out.println(Thread.currentThread().getName());
                    Thread.sleep(1000);
                }
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
       super.run();
        service.print(new Object());
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

   private Service service;

    public ThreadB(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.print(new Object());
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      Service service=new Service();
      ThreadA a=new ThreadA(service);
      a.setName("A");
      a.start();
      ThreadB b=new ThreadB(service);
      b.setName("B");
      b.start();
    }
}
```
程序的运行结果为：
```java
A
B
A
B
B
A
B
A
B
A
```
交替打印的原因是持有的锁不是一个。

> 摘选自 java多线程核心编程技术-2.2.10