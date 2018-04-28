---
title: 2.2.11同步synchronized方法无限等待与解决
date: 2017-07-23 13:54:22
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
同步方法易造成死循环
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
    synchronized public void methodA(){
        System.out.println("methodA begin");
        boolean isContinueRun=true;
        while (isContinueRun){

        }
        System.out.println("methodA end");
    }

    synchronized public void methodB(){
        System.out.println("methodB begin");
        System.out.println("methodB end");
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
        service.methodA();
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
        service.methodB();
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
      a.start();
      ThreadB b=new ThreadB(service);
      b.start();
    }
}
```
程序的运行结果为：
```java
methodA begin


```
线程B永远得不到运行的机会，锁死了。这时候可以用同步块来解决这样的问题。更改Service.java文件代码如下:
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
    Object object1=new Object();
    public void methodA(){
        synchronized (object1) {
            System.out.println("methodA begin");
            boolean isContinueRun = true;
            while (isContinueRun) {

            }
            System.out.println("methodA end");
        }
    }
    Object object2=new Object();
    public void methodB(){
        synchronized (object2) {
            System.out.println("methodB begin");
            System.out.println("methodB end");
        }
    }
}
```
程序的运行结果为：
```java
methodA begin
methodB begin
methodB end
```
> 摘选自 java多线程核心编程技术-2.2.11