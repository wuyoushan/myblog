---
title: 2.2.12多线程的死锁
date: 2017-07-23 13:57:29
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
java线程死锁是一个经典的问题，因为不同的线程都在等待根本不可能被释放的锁，从而导致所有的任务都无法继续完成。在多线程技术中，“死锁”是必须避免的，因为这会造成线程的“假死”
```java
/**
 * @author wuyoushan
 * @date 2017/4/25.
 */
public class DealThread implements Runnable {

    public String username;
    public Object lock1=new Object();
    public Object lock2=new Object();

    public void setFlag(String username){
        this.username=username;
    }

    @Override
    public void run() {
        if (username.equals("a")){
            synchronized (lock1){
                try{
                    System.out.println("username="+username);
                    Thread.sleep(3000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                synchronized (lock2){
                    System.out.println("按lock1->lock2代码顺序执行了");
                }
            }
        }

        if (username.equals("b")){
            synchronized (lock2){
                try{
                    System.out.println("username="+username);
                    Thread.sleep(3000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                synchronized (lock1){
                    System.out.println("按lock2->lock1代码顺序执行了");
                }
            }
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try {
            DealThread t1=new DealThread();
            t1.setFlag("a");

            Thread thread1=new Thread(t1);
            thread1.start();
            Thread.sleep(100);

            t1.setFlag("b");
            Thread thread2=new Thread(t1);
            thread2.start();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为：
```java
username=a
username=b
```
死锁是程序设计的bug，在设计程序时就要避免双方互相持有对方的锁情况。需要说明的是，本实验使用synchronized嵌套的代码结构来实现死锁，其实不使用嵌套的synchronized代码结构也会出现死锁，与嵌套不嵌套无任何关系，不要被代码结构所误导。只要互相等待对方释放锁就有可能出现死锁。

> 摘选自 java多线程核心编程技术-2.2.12