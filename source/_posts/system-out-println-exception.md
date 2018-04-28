---
title: 1.2.4留意i--与System.out.println()的异常
date: 2017-07-09 15:29:31
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

解决非线程安全问题使用的是synchronized关键字。但println()方法与i++使用时"有可能"出现另外一种异常情况。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private int i=5;

    @Override
    public void run() {
        super.run();
        System.out.println("i="+(i--)+"threadName="+Thread.currentThread().getName());
        //注意：代码i--由前面项目中单独一行运行改成在当前项目中在println()方法中直接打印
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        Thread t1=new Thread(myThread);
        Thread t2=new Thread(myThread);
        Thread t3=new Thread(myThread);
        Thread t4=new Thread(myThread);
        Thread t5=new Thread(myThread);
        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
    }
}
```
程序运行后根据概率还是会出现非线程安全问题
```java
i=5 threadName=Thread-1
i=2 threadName=Thread-5
i=3 threadName=Thread-4
i=4 threadName=Thread-3
i=4 threadName=Thread-2
```
虽然println()方法在内部是同步的，但i--的操作却是进入println()之前发生的，所以有发生线程安全问题的概率。println()源码如下:
```java
public void println(String x) {
    synchronized (this) {
        print(x);
        newLine();
    }
}
```
所以，为了防止发生非线程安全问题，还是应该继续使用同步方法。

> 摘选自 java多线程核心编程技术-1.2.4