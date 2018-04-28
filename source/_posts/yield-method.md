---
title: 1.9yield方法
date: 2017-07-13 22:37:22
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
yield方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU执行时间。但放弃的时间不确定，也有可能刚刚放弃，马上又获得CPU时间片。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
       long beginTime=System.currentTimeMillis();
        int count=0;
        for (int i=0;i<50000000;i++){
            //Thread.yield();
            count=count+(i+1);
        }
        long endTime=System.currentTimeMillis();
        System.out.println("用时:"+(endTime-beginTime)+"毫秒！");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread=new MyThread();
        myThread.start();
    }
}
```
程序的运行结果为:
```java
用时:33毫秒！
```
将代码：
```java
//Thread.yield();
```
去掉注释符号，再次运行，运行结果如下:
```java
用时:10546毫秒！
```
运行变慢的原因是。yield()方法将CPU让给其他资源导致速度的变慢。

> 摘选自 java多线程核心编程技术-1.9