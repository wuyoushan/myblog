---
title: 2.3.7synchronized代码块有volatile同步的功能
date: 2017-07-23 14:51:08
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
关键字synchronized可以使多个线程访问同一资源具有同步性，而且它还具有将线程工作内存中的私有变量与公共内存中的变量同步的功能。
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
   private boolean isContinueRun=true;

    public void runMethod(){
        while (isContinueRun==true){

        }
        System.out.println("停下来了！");
    }

    public void stopMethod(){
        isContinueRun=false;
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
       service.runMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Service service;

    public ThreadB(Service service) {
        super();
        this.service = service;

    }

    @Override
    public void run() {
        service.stopMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      try{
         Service service=new Service();
          ThreadA a=new ThreadA(service);
          a.start();
          Thread.sleep(1000);
          ThreadB b=new ThreadB(service);
          b.start();
          System.out.println("已经发起停止的命令了！");
      }catch (InterruptedException e){
            e.printStackTrace();
      }
    }
}
```
以-server服务器模式运行此项目，出现死循环。程序的运行结果为:
```java
已经发起停止的命令了！
```
出现了死循环。得到这个结果是各线程间的数据值没有可视化造成的，而关键字synchronized可以具有可视化。

更改Service.java代码如下：
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
   private boolean isContinueRun=true;

    public void runMethod(){
        String anyString=new String();
        while (isContinueRun==true){
            synchronized (anyString){

            }
        }
        System.out.println("停下来了！");
    }

    public void stopMethod(){
        isContinueRun=false;
    }
}
```
再以-server服务器模式运行程序后可以正常退出。运行结果如下:
```java
已经发起停止的命令了！
停下来了！
```
关键字synchronized可以保证在同一时刻，只有一个线程可以执行某个方法或某一个代码块。它包含两个特征：互斥性和可见性。同步synchronized不仅可以解决一个线程看到对象处于不一致的状态，还可以保证进入同步方法或者同步代码块的每个线程，都看到由同一个锁保护之前所有的修改效果。

学习多线程并发，要着重“外练互斥，内修可见”，这是掌握多线程、学习多线程并发的重要技术点。

> 摘选自 java多线程核心编程技术-2.3.7