---
title: 2.1synchronized同步方法
date: 2017-07-15 10:36:22
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
“线程安全”与“非线程安全”相关的技术点，它们是学习多线程技术时一定会遇到的经典问题。“非线程安全”其实会在多个线程对同一个对象中的实例变量进行并发访问是发生时发生，产生的后果就是“脏读”，也就是取到的数据其实是被更改过的。，而"线程安全"就是以获得的实例变量的值是经过同步处理的，不会出现脏读的现象。

#### 2.1.1方法内的变量
“非线程安全”问题存在于“实例变量”中，如果是方法内部的私有变量则不存在“非线程安全”问题，所得结果也就是“线程安全”的了。
```java
/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class HasSelfPrivateNum {
    public void addI(String username){
        try {
            int num = 0;
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
文件ThreadA.java代码如下：
```java
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

```
文件ThreadB.java代码如下:
```java
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
运行结果如下:
```java
a set over!
b set over!
b num=200
a num=100
```
> 摘选自 java多线程核心编程技术-2.1