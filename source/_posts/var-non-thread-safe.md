---
title: 2.1.2实例变量非线程安全
date: 2017-07-15 10:46:46
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

#### 2.1.2实例变量非线程安全
如果多个线程共同访问1个对象中的实例变量，则有可能出现“非线程安全”问题。
```java
/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class HasSelfPrivateNum {

    private int num=0;

    public void addI(String username){
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
       HasSelfPrivateNum numRef=new HasSelfPrivateNum();
        ThreadA threadA=new ThreadA(numRef);
        threadA.start();

        ThreadB threadB=new ThreadB(numRef);
        threadB.start();
    }
}

```
程序运行结果为:
```java
a set over!
b set over!
b num=200
a num=200
```
本实验是两个线程同时访问一个没有同步的方法，如果两个线程同时操作业务对象中的实例变量，则有可能会出现“非线程安全”问题。修改HasSelfPrivateNum.java文件如下
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

```
重新运行程序，运行结果为:
```java
a set over!
a num=100
b set over!
b num=200
```
> 摘选自 java多线程核心编程技术-2.1.2