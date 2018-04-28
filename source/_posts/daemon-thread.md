---
title: 1.11守护线程
date: 2017-07-13 22:39:43
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
在java线程中有两种线程，一种是用户线程，另一种是守护线程。守护线程是一种特殊的线程，它的特殊有“陪伴”的含义，当进程中不存在非守护线程了，则守护线程自动销毁。典型的守护线程就是垃圾回收线程，当进程中没有非守护进程了，则垃圾回收线程也就没有存在的必要了，自动销毁。用一个比较通俗的比喻来解释一下“守护线程”：任何一个守护线程都是整个JVM中所有非守护线程的“保姆”，只要当前JVM实例中存在任何一个非守护线程没有结束，守护线程就在工作，只有当最后一个非守护线程结束时，守护线程才随着JVM一同结束工作。Daemon的作用就是为其他线程的运行提供便利服务，守护线程最经典的应用就是GC(垃圾回收器)，它是一个很称职 的守护者。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private  int i=0;

    @Override
    public void run() {
      try{
          while(true){
              i++;
              System.out.println("i="+(i));
              Thread.sleep(1000);
          }
      }catch(InterruptedException ex){
          ex.printStackTrace();
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
            MyThread thread = new MyThread();
            thread.setDaemon(true);
            thread.start();
            Thread.sleep(5000);
            System.out.println("我离开thread对象也不再打印了，也就是停止了！");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
运行结果如下:
```java
i=1
i=2
i=3
i=4
i=5
i=6
我离开thread对象也不再打印了，也就是停止了！
```
> 摘选自 java多线程核心编程技术-1.11