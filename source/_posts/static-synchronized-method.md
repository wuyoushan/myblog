---
title: 2.2.9静态同步synchronized方法与synchronized(class)代码块
date: 2017-07-23 13:51:02
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
synchronized还可以应用在static静态方法上，如果这样写，那是对当前的*.java文件对应的Class类进行持锁。
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    synchronized  public static void printA(){
        try{
            System.out.println("线程名称为："+Thread.currentThread().getName()
                    +"在"+System.currentTimeMillis()+"进入printA");
            Thread.sleep(3000);
            System.out.println("线程名称为："+Thread.currentThread().getName()
                    +"在"+System.currentTimeMillis()+"离开printB");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    synchronized  public static void printB(){
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"进入printA");
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"离开printB");
    }
}

 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    @Override
    public void run() {
       super.run();
        Service.printA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    @Override
    public void run() {
        super.run();
        Service.printB();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      ThreadA a=new ThreadA();
      a.setName("A");
      a.start();
      ThreadB b=new ThreadB();
      b.setName("B");
      b.start();
    }
}
```
程序的运行结果为：
```java
线程名称为：A在1492993790410进入printA
线程名称为：A在1492993793410离开printA
线程名称为：B在1492993793410进入printB
线程名称为：B在1492993793410离开printB
```
从运行的结果来看，并没有什么特别之处，都是同步的效果，和将synchronized关键字加到static方法上使用的效果是一样的。其实还是有本子上的不同的，synchronized关键字加到static静态方法是是给Class类上锁的，而synchronized关键字加到非static静态方法上是给对象上锁。

**验证不是同一个锁**
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    synchronized  public static void printA(){
        try{
            System.out.println("线程名称为："+Thread.currentThread().getName()
                    +"在"+System.currentTimeMillis()+"进入printA");
            Thread.sleep(3000);
            System.out.println("线程名称为："+Thread.currentThread().getName()
                    +"在"+System.currentTimeMillis()+"离开printA");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    synchronized  public static void printB(){
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"进入printB");
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"离开printB");
    }

    synchronized  public void printC(){
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"进入printC");
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"离开printC");
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
       super.run();
        service.printA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

   private Service service;

    public ThreadB(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.printB();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadC extends Thread{

   private Service service;

    public ThreadC(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.printC();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      Service service=new Service();
      ThreadA a=new ThreadA(service);
      a.setName("A");
      a.start();
      ThreadB b=new ThreadB(service);
      b.setName("B");
      b.start();
      ThreadC c=new ThreadC(service);
      c.setName("C");
      c.start();
    }
}
```
程序的运行结果为：
```java
线程名称为：B在1492994339480进入printB
线程名称为：B在1492994339480离开printB
线程名称为：A在1492994339482进入printA
线程名称为：C在1492994339482进入printC
线程名称为：C在1492994339482离开printC
线程名称为：A在1492994342482离开printA
```
异步的原因是持有不同的锁，一个是对象锁，另外一个是Class锁，而Class锁可以对类的所有对象实例起作用

```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    synchronized  public static void printA(){
        try{
            System.out.println("线程名称为："+Thread.currentThread().getName()
                    +"在"+System.currentTimeMillis()+"进入printA");
            Thread.sleep(3000);
            System.out.println("线程名称为："+Thread.currentThread().getName()
                    +"在"+System.currentTimeMillis()+"离开printA");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    synchronized  public static void printB(){
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"进入printB");
        System.out.println("线程名称为："+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"离开printB");
    }

}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
       super.run();
        service.printA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

   private Service service;

    public ThreadB(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.printB();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      Service service1=new Service();
      Service service2=new Service();
      ThreadA a=new ThreadA(service1);
      a.setName("A");
      a.start();
      ThreadB b=new ThreadB(service2);
      b.setName("B");
      b.start();
    }
}
```
程序的运行结果为：
```java
线程名称为：A在1492995107179进入printA
线程名称为：A在1492995110186离开printA
线程名称为：B在1492995110186进入printB
线程名称为：B在1492995110186离开printB
```
同步synchronized(class)代码块的作用其实和synchronized static方法的作用一样。
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    public static void printA(){
        synchronized (Service.class){
            try {
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "进入printA");
                Thread.sleep(3000);
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "离开printA");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void printB() {
        synchronized (Service.class) {
            System.out.println("线程名称为：" + Thread.currentThread().getName()
                    + "在" + System.currentTimeMillis() + "进入printB");
            System.out.println("线程名称为：" + Thread.currentThread().getName()
                    + "在" + System.currentTimeMillis() + "离开printB");
        }
    }

}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
       super.run();
        service.printA();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

   private Service service;

    public ThreadB(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.printB();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      Service service1=new Service();
      Service service2=new Service();
      ThreadA a=new ThreadA(service1);
      a.setName("A");
      a.start();
      ThreadB b=new ThreadB(service2);
      b.setName("B");
      b.start();
    }
}
```
程序的运行结果为：
```java
线程名称为：A在1492995444496进入printA
线程名称为：A在1492995447497离开printA
线程名称为：B在1492995447497进入printB
线程名称为：B在1492995447497离开printB
```
> 摘选自 java多线程核心编程技术-2.2.9