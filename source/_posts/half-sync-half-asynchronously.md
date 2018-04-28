---
title: 2.2.4一半同步，一半异步
date: 2017-07-18 08:12:16
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

本实验要说明：不在synchronized块中就是异步执行，在synchronized块中就是同步执行
```java
/**
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class Task {
    public void doLongTimeTask(){
        for(int i=0;i<100;i++){
            System.out.println("nosynchronized threadName= "
            +Thread.currentThread().getName()+" i= "+(i+1));
        }
        System.out.println("");
        synchronized (this){
            for(int i=0;i<100;i++){
                System.out.println("synchronized threadName= "
                        +Thread.currentThread().getName()+" i= "+(i+1));
            }
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/13.
 */
public class MyThread1 extends Thread{

    private Task task;
    public MyThread1(Task task) {
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
 * @date 2017/4/13.
 */
public class MyThread2 extends Thread{

    private Task task;
    public MyThread2(Task task) {
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
 * @date 2017/4/13.
 */
public class Run {
    public static void main(String[] args) {
        Task task=new Task();
        MyThread1 thread1=new MyThread1(task);
        thread1.start();

        MyThread2 thread2=new MyThread2(task);
        thread2.start();
    }
}
```
程序的结果不是固定的但从输出的结果来看是同步的：
```java
nosynchronized threadName= Thread-0 i= 56
nosynchronized threadName= Thread-1 i= 97
nosynchronized threadName= Thread-0 i= 57
nosynchronized threadName= Thread-1 i= 98
nosynchronized threadName= Thread-0 i= 58
nosynchronized threadName= Thread-1 i= 99
nosynchronized threadName= Thread-0 i= 59
nosynchronized threadName= Thread-1 i= 100
nosynchronized threadName= Thread-0 i= 60

nosynchronized threadName= Thread-0 i= 97
synchronized threadName= Thread-1 i= 37
nosynchronized threadName= Thread-0 i= 98
synchronized threadName= Thread-1 i= 38
nosynchronized threadName= Thread-0 i= 99
synchronized threadName= Thread-1 i= 39
nosynchronized threadName= Thread-0 i= 100

synchronized threadName= Thread-1 i= 40
synchronized threadName= Thread-1 i= 41
synchronized threadName= Thread-1 i= 42
synchronized threadName= Thread-1 i= 43
synchronized threadName= Thread-1 i= 44
synchronized threadName= Thread-1 i= 45
synchronized threadName= Thread-1 i= 46
synchronized threadName= Thread-1 i= 47
synchronized threadName= Thread-1 i= 48
```

> 摘选自 java多线程核心编程技术-2.2.4