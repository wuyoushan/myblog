---
title: 2.2.6验证同步synchronized(this)代码块是锁定当前对象的
date: 2017-07-23 13:44:18
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
和synchronized方法一样，synchronized(this)代码块也是锁定当前对象的
```java
/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class Task {
    public void otherMethod(){
        System.out.println("------------------run--otherMethod");
    }
    public void doLongTimeTask(){
        synchronized (this){
            for (int i = 0; i <1000 ; i++) {
                System.out.println("synchronized threadName="
                +Thread.currentThread().getName()+" i="+(i+1));
            }
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Task task;

    public ThreadA(Task task){
        super();
        this.task=task;
    }

    @Override
    public void run() {
        super.run();
        task.doLongTimeTask();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Task task;

    public ThreadB(Task task){
        super();
        this.task=task;
    }

    @Override
    public void run() {
        super.run();
        task.otherMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
        Task task=new Task();
        ThreadA a=new ThreadA(task);
        a.start();
        Thread.sleep(10);
        ThreadB b=new ThreadB(task);
        b.start();
    }
}
```
程序运行结果如下:
```java
synchronized threadName=Thread-0 i=212
synchronized threadName=Thread-0 i=213
synchronized threadName=Thread-0 i=214
------------------run--otherMethod
synchronized threadName=Thread-0 i=215
synchronized threadName=Thread-0 i=216
synchronized threadName=Thread-0 i=217
```
运行的结果为异步的。
修改Task.java如下：
```java
/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class Task {
    synchronized public void otherMethod(){
        System.out.println("------------------run--otherMethod");
    }
    public void doLongTimeTask(){
        synchronized (this){
            for (int i = 0; i <1000 ; i++) {
                System.out.println("synchronized threadName="
                +Thread.currentThread().getName()+" i="+(i+1));
            }
        }
    }
}
```
运行结果如下:
```java
synchronized threadName=Thread-0 i=997
synchronized threadName=Thread-0 i=998
synchronized threadName=Thread-0 i=999
synchronized threadName=Thread-0 i=1000
------------------run--otherMethod
```
从上结果可知运行结果是同步的。

> 摘选自 java多线程核心编程技术-2.2.6