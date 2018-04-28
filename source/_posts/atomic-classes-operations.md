---
title: 2.3.5使用原子类进行i++操作
date: 2017-07-23 14:47:14
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
除了在i++操作时使用synchronized关键字实现同步外，还可以使用AtomicInteger原子类进行实现

原子操作是不能分割的整体，没有其他线程能够中断或检查正在原子操作中的变量。一个原子类型就是一个原子操作可用的类型，它可以在没有锁的情况下做到线程安全
```java
/**
 * @author wuyoushan
 * @date 2017/5/5.
 */
public class AddCountThread extends Thread {

    private AtomicInteger count =new AtomicInteger(0);
    @Override
    public void run() {
        for (int i=0;i<10000;i++){
            System.out.println(count.incrementAndGet());
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
       AddCountThread countThread=new AddCountThread();
        Thread t1=new Thread(countThread);
        t1.start();

        Thread t2=new Thread(countThread);
        t2.start();

        Thread t3=new Thread(countThread);
        t3.start();

        Thread t4=new Thread(countThread);
        t4.start();

        Thread t5=new Thread(countThread);
        t5.start();
    }
}
```
程序运行后，成功累加到50000
```java
49994
49995
49996
49997
49998
49999
50000
```
> 摘选自 java多线程核心编程技术-2.3.5