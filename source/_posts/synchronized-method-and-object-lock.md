---
title: 2.1.4synchronized方法与对象锁
date: 2017-07-15 10:51:32
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {
    public void methodA(){
        try {
            System.out.println("begin methodA threadName=" +
                    Thread.currentThread().getName());
            Thread.sleep(1000);
            System.out.println("end");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private MyObject object;

    public ThreadA(MyObject object){
        super();
        this.object=object;
    }

    @Override
    public void run() {
        super.run();
       object.methodA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private MyObject object;

    public ThreadB(MyObject object){
        super();
        this.object=object;
    }

    @Override
    public void run() {
        super.run();
        object.methodA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        MyObject object=new MyObject();
        ThreadA threadA=new ThreadA(object);
        threadA.setName("A");

        ThreadB threadB=new ThreadB(object);
        threadB.setName("B");
        threadA.start();
        threadB.start();
    }
}

```
程序运行候的结果为:
```java
begin methodA threadName=A
begin methodA threadName=B
end
end
```

更改MyObject.java中的代码:
```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {
    synchronized public void methodA(){
        try {
            System.out.println("begin methodA threadName=" +
                    Thread.currentThread().getName());
            Thread.sleep(1000);
            System.out.println("end");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

```
更改后得运行结果为：
```java
begin methodA threadName=A
end
begin methodA threadName=B
end
```
通过上面的实验结论，调用关键字synchronized声明的方法一定是排队运行的。另外需要牢牢记住“共享”这两个字，只有共享资源的读写访问才需要同步化，如果不是共享资源，那么根本就没有同步的必要。


那其他的方法被调用时会是什么效果呢？如何查看到Lock锁对象效果呢？
```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {
    synchronized public void methodA(){
        try {
            System.out.println("begin methodA threadName=" +
                    Thread.currentThread().getName());
            Thread.sleep(1000);
            System.out.println("end endTime="+System.currentTimeMillis());
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    public void methodB(){
        try {
            System.out.println("begin methodB threadName=" +
                    Thread.currentThread().getName()+"begin time"+
            System.currentTimeMillis());
            Thread.sleep(1000);
            System.out.println("end");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private MyObject object;

    public ThreadA(MyObject object){
        super();
        this.object=object;
    }

    @Override
    public void run() {
        super.run();
       object.methodA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private MyObject object;

    public ThreadB(MyObject object){
        super();
        this.object=object;
    }

    @Override
    public void run() {
        super.run();
        object.methodB();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        MyObject object=new MyObject();
        ThreadA threadA=new ThreadA(object);
        threadA.setName("A");

        ThreadB threadB=new ThreadB(object);
        threadB.setName("B");
        threadA.start();
        threadB.start();
    }
}

```
程序的运行结果为:
```java
begin methodA threadName=A
begin methodB threadName=Bbegin time1491525491404
end
end endTime=1491525492404

```
通过上面的实验可以得知，虽然线程A先持有了object对象的锁，但线程B完全可以异步调用非synchronized类型的方法。

如果将MyObject.java文件中的methodB()方法前加上synchronized关键字，代码如下：
```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {
    synchronized public void methodA(){
        try {
            System.out.println("begin methodA threadName=" +
                    Thread.currentThread().getName());
            Thread.sleep(1000);
            System.out.println("end endTime="+System.currentTimeMillis());
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    synchronized public void methodB(){
        try {
            System.out.println("begin methodB threadName=" +
                    Thread.currentThread().getName()+"begin time"+
            System.currentTimeMillis());
            Thread.sleep(1000);
            System.out.println("end");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

```
程序的运行结果为:
```java
begin methodA threadName=A
end endTime=1491525875727
begin methodB threadName=Bbegin time1491525875727
end
```
此实验的结论是:
1. A线程先持有object对象的Lock锁，B线程可以以异步的方式调用object对象中的非synchronized类型的方法。
2.  A线程先持有object对象的Lock锁，B线程如果在这时调用object对象中的synchronized类型的方法则需等待，也就是同步。

> 摘选自 java多线程核心编程技术-2.1.4