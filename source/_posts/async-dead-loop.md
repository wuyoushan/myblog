---
title: 2.3.3解决异步死循环
date: 2017-07-23 14:43:33
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
```java
/**
 * @author wuyoushan
 * @date 2017/5/3.
 */
public class RunThread extends Thread {

    private boolean isRuning=true;

    public boolean isRuning() {
        return isRuning;
    }

    public void setRuning(boolean runing) {
        isRuning = runing;
    }

    @Override
    public void run() {
        System.out.println("进入run了");
        while (isRuning==true){
        }
        System.out.println("线程被停止了！");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try{
            RunThread thread=new RunThread();
            thread.start();
            Thread.sleep(1000);
            thread.setRuning(false);
            System.out.println("已经赋值为false");
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为：
```java
进入run了
线程被停止了！
已经赋值为false
```
如果将JVM的运行参数设置为-server程序的运行结果为:
```java
进入run了
已经赋值为false
```
代码“System.out.println("线程被停止了！")”从未被执行。
是什么样的原因造成将JVM设置为-server时出现死循环呢？

在启动RunThread.java线程时，变量private boolean isRunning=true;存在于公共堆栈及线程的私有栈中。在JVM被设置为-server模式时为了线程运行的效率，线程一直在私有堆栈中取得isRunning的值是true。而代码thread.setRunning(false);虽然被执行，更新的却是公共堆栈中的isRunning变量值false，所以一直就是死循环的状态。

这个问题其实就是私有堆栈中的值和公共堆栈中的值不同步造成的。解决这样的问题就要使用volatile关键字了，他的主要作用就是当线程访问isRunning这个变量时，强制性从公共堆栈中进行取值。
```java
/**
 * @author wuyoushan
 * @date 2017/5/3.
 */
public class RunThread extends Thread {

    volatile private boolean isRuning=true;

    public boolean isRuning() {
        return isRuning;
    }

    public void setRuning(boolean runing) {
        isRuning = runing;
    }

    @Override
    public void run() {
        System.out.println("进入run了");
        while (isRuning==true){
        }
        System.out.println("线程被停止了！");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try{
            RunThread thread=new RunThread();
            thread.start();
            Thread.sleep(1000);
            thread.setRuning(false);
            System.out.println("已经赋值为false");
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
进入run了
线程被停止了！
已经赋值为false
```
通过使用volatile关键字，强制的从公共内存中读取变量的值。

使用volatile关键字增加了实例变量在多个线程之间的可见性。但volatile关键字最致命的缺点是不支持原子性。

下面将关键字synchronized和volatile进行比较
1. 关键字volatile是线程同步的轻量级实现，所以volatile性能肯定比synchronized要好，并且volatile只能修饰于变量，而synchronized可以修饰方法，以及代码块。随着JDK新版本的发布，synchronized关键字在执行效率上得到很大提升，在开发中使用synchronized关键字的比率还是比较大的。

2. 多线程访问volatile不会发生阻塞，而synchronized会出现阻塞。

3. volatile能保证数据的可见性，但不能保证原子性；而synchronized可以保证原子性，也可以间接保证可见性，因为它会将私有内存和公共内存中的数据做同步。

4. 再次重申一下，关键字volatile解决的是变量在多个线程之间的可见性；而synchronized关键字解决的是多个线程之间访问资源的同步性。


线程安全包含原子性和可见性两个方面，java的同步机制都是围绕这两个方面来确保线程安全的。

> 摘选自 java多线程核心编程技术-2.3.3