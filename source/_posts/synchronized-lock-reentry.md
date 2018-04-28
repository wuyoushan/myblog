---
title: 2.1.6synchronized锁重入
date: 2017-07-15 11:01:17
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

关键字synchronized拥有锁重入的功能，也就是在使用synchronized时，当一个线程得到一个对象锁后，再次请求此对象锁时是可以再次得到该对象锁的。这也证明在一个synchronized方法/块的内部调用本类的其他synchronized方法/块时，是永远可以得到锁的。
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    synchronized public void service1(){
        System.out.println("service1");
        service2();
    }

    synchronized public void service2() {
        System.out.println("service2");
        service3();
    }

    synchronized public void service3() {
        System.out.println("service3");
    }
}

/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
        Service service=new Service();
        service.service1();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        MyThread thread=new MyThread();
        thread.start();
    }
}

```
程序的运行结果:
```java
service1
service2
service3
```
“可重入锁”的概念是:自己可以再次获取自己的内部锁。
比如有1条线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。

可重入锁也支持在父子类继承的环境中
```java
/**
 * @author wuyoushan
 * @date 2017/4/11.
 */
public class Main {
    public int i=10;

    synchronized public void operateMainMethod(){
        try {
            i--;
            System.out.println("main print i=" + i);
            Thread.sleep(100);
        }catch(InterruptedException e){
            e.printStackTrace();
        }

    }
}

/**
 * @author wuyoushan
 * @date 2017/4/11.
 */
public class Sub extends Main {
    synchronized public void operateISubMethod(){
        try{
            while (i>0) {
                i--;
                System.out.println("sub print i="+i);
                Thread.sleep(100);
                this.operateMainMethod();
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
       Sub sub=new Sub();
        sub.operateISubMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        MyThread thread=new MyThread();
        thread.start();
    }
}
```
程序运行结果为:
```java
sub print i=9
main print i=8
sub print i=7
main print i=6
sub print i=5
main print i=4
sub print i=3
main print i=2
sub print i=1
main print i=0
```
此程序说明，当存在父类继承关系时，子类是完全可以通过“可重入锁”调用父类的同步方法的。

> 摘选自 java多线程核心编程技术-2.1.6