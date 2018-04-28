---
title: 2.1.7出现异常，锁自动释放
date: 2017-07-15 11:05:05
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

当一个线程值执行的代码出现异常时，其所持有的锁会自动释放
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    synchronized public void testMethod(){
        if(Thread.currentThread().getName().equals("a")){
            System.out.println("ThreadName="+Thread.currentThread().getName()+
            " run beginTime="+System.currentTimeMillis());
            int i=1;
            while(i==1){
                if((""+Math.random()).substring(0,8).equals("0.123456")){
                    System.out.println("ThreadName="
                    +Thread.currentThread().getName()
                    +" run exceptionTime="
                    +System.currentTimeMillis());
                    Integer.parseInt("a");
                }
            }
        }else{
            System.out.println("Thread B run Time="+System.currentTimeMillis());
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service){
        super();
        this.service=service;
    }

    @Override
    public void run() {
        super.run();
        service.testMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Service service;

    public ThreadB(Service service){
        super();
        this.service=service;
    }

    @Override
    public void run() {
        super.run();
        service.testMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class Test {
    public static void main(String[] args) {
       try{
           Service service=new Service();
           ThreadA a=new ThreadA(service);
           a.setName("a");
           a.start();
           a.sleep(200);
           Thread.sleep(500);
           ThreadB b=new ThreadB(service);
           b.setName("b");
           b.start();
       }catch (InterruptedException e){
           e.printStackTrace();
       }
    }
}

```
程序的运行结果为:
```java
ThreadName=a run beginTime=1491870317217
ThreadName=a run exceptionTime=1491870317336
Exception in thread "a" java.lang.NumberFormatException: For input string: "a"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:492)
	at java.lang.Integer.parseInt(Integer.java:527)
	at wys.test.Service.testMethod(Service.java:20)
	at wys.test.ThreadA.run(ThreadA.java:19)
Thread B run Time=1491870317918
```
线程a出现异常并释放锁，线程b进入方法正常打印，实验的结论就是出现异常的锁被自动释放了。

> 摘选自 java多线程核心编程技术-2.1.7