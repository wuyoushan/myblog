---
title: 2.2.14内置类与同步：实验1
date: 2017-07-23 14:06:29
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
本实验测试的案例是在内置类中有两个同步方法，但使用的却是不同的锁，打印的结果也是异步的。
```java
/**
 * @author wuyoushan
 * @date 2017/4/28.
 */
public class OutClass {
    static class Inner{
        public void method1(){
            synchronized ("其他的锁"){
                for (int i = 1; i <=10 ; i++) {
                    System.out.println(Thread.currentThread().getName()+" i="+i);
                    try{
                        Thread.sleep(100);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }
        }

        public synchronized void method2(){
            for (int i = 11; i <=20; i++) {
                System.out.println(Thread.currentThread().getName()+" i="+i);
                try{
                    Thread.sleep(100);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        final OutClass.Inner inner=new OutClass.Inner();
        Thread t1=new Thread(new Runnable() {
            @Override
            public void run() {
                inner.method1();
            }
        },"A");

        Thread t2=new Thread(new Runnable() {
            @Override
            public void run() {
                inner.method2();
            }
        },"B");
        t1.start();
        t2.start();
    }
}
```
程序的运行结果为：
```java
B i=14
B i=15
A i=5
A i=6
B i=16
```
由于持有不同的“对象监视器”，所以打印结果就是乱序的。

> 摘选自 java多线程核心编程技术-2.2.14