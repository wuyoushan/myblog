---
title: 2.1.3多个对象多个锁
date: 2017-07-15 10:48:42
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

```java
/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class HasSelfPrivateNum {

    private int num=0;

    synchronized public void addI(String username){
        try {
            if (username.equals("a")) {
                num = 100;
                System.out.println("a set over!");
                Thread.sleep(200);
            }else{
                num=200;
                System.out.println("b set over!");
            }
            System.out.println(username+" num="+num);
        }catch (Exception ex){
            ex.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private HasSelfPrivateNum numRef;

    public ThreadA(HasSelfPrivateNum numRef){
        super();
        this.numRef=numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("a");
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private HasSelfPrivateNum numRef;

    public ThreadB(HasSelfPrivateNum numRef){
        super();
        this.numRef=numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("b");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
       HasSelfPrivateNum numRef1=new HasSelfPrivateNum();
       HasSelfPrivateNum numRef2=new HasSelfPrivateNum();
        ThreadA threadA=new ThreadA(numRef1);
        threadA.start();

        ThreadB threadB=new ThreadB(numRef2);
        threadB.start();
    }
}

```
程序的运行结果如下:
```java
a set over!
b set over!
b num=200
a num=100
```
上面的示例是两个线程分别访问同一个类的两个不同实例的相同名称的同步方法，效果却是以异步的方式运行的。本示例由于创建了2个业务对象，在系统中产生出2个锁，所以运行结果是异步的，打印的效果就是先打印b,然后打印a。

从上面程序运行结果来看，虽然在HasSelfPrivateNum.java中使用了synchronized关键字，但打印的顺序却不是同步的，是交叉的。为什么是这样的结果呢？

关键字synchronized取得的锁都是对象锁，而不是把一段代码或方法（函数）当作锁，所以在上面的示例中，哪个线程先执行带synchronized关键字的方法，哪个线程就持有该方法所属对象的锁Lock,那么其他线程只能呈等待状态，前提是多个线程访问的是同一个对象。

> 摘选自 java多线程核心编程技术-2.1.3