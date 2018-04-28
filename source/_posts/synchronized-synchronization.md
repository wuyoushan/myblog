---
title: 2.2.5synchronized代码块间的同步性
date: 2017-07-18 08:14:39
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

代码块的访问将被阻塞，这说明synchronized使用的“对象监视器”是一个。
```java
/**
 * @author wuyoushan
 * @date 2017/4/14.
 */
public class ObjectService {
    public void serviceMethodA(){
        try{
            synchronized (this){
                System.out.println("A begin time="+System.currentTimeMillis());
                Thread.sleep(2000);
                System.out.println("A end    end="+System.currentTimeMillis());
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
    public void serviceMethodB(){
        synchronized (this){
            System.out.println("B begin time="+System.currentTimeMillis());
            System.out.println("B end    end="+System.currentTimeMillis());
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private ObjectService objectService;

    public ThreadA(ObjectService objectService){
        super();
        this.objectService=objectService;
    }

    @Override
    public void run() {
        super.run();
       objectService.serviceMethodA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private ObjectService objectService;

    public ThreadB(ObjectService objectService){
        super();
        this.objectService=objectService;
    }

    @Override
    public void run() {
        super.run();
        objectService.serviceMethodB();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        ObjectService objectService=new ObjectService();
        ThreadA a=new ThreadA(objectService);
        a.setName("a");
        a.start();

        ThreadB b=new ThreadB(objectService);
        b.setName("b");
        b.start();
    }
}
```
程序的运行结果为：
```java
A begin time=1492130124674
A end    end=1492130126675
B begin time=1492130126675
B end    end=1492130126675
```
synchronized(this)代码块是锁定当前对象的

> 摘选自 java多线程核心编程技术-2.2.5