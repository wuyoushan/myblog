---
title: 2.2.15内置类与同步：实验2
date: 2017-07-23 14:08:00
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
本实验测试同步代码块synchronized(class2)对class2上锁后，其他线程只能以同步的方式调用class2中的静态同步方法。
```java
/**
 * @author wuyoushan
 * @date 2017/4/28.
 */
public class OutClass {
    static class InnerClass1{
        public void method1(InnerClass2 class2){
            String threadName=Thread.currentThread().getName();
            synchronized (class2){
                System.out.println(threadName+" 进入InnerClass1类中的method1方法");
                for (int i = 1; i < 10 ; i++) {
                    System.out.println(Thread.currentThread().getName()+" i="+i);
                    try{
                        Thread.sleep(100);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
                System.out.println(threadName+" 离开InnerClass1类中的method1方法");
            }
        }

        public synchronized void method2(){
            String threadName=Thread.currentThread().getName();
            System.out.println(threadName+" 进入InnerClass1类中的method2方法");
            for (int j = 0; j <10; j++) {
                System.out.println("j="+j);
                try{
                    Thread.sleep(100);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
            System.out.println(threadName+" 离开InnerClass1类中的method2方法");
        }
    }

    static class InnerClass2{
        public synchronized void method1(){
            String threadName=Thread.currentThread().getName();
            System.out.println(threadName+" 进入InnerClass2类中的method1方法");
            for (int k = 0; k < 10; k++) {
                System.out.println("k="+k);
                try{
                    Thread.sleep(100);
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
            System.out.println(threadName+" 离开InnerClass2类中的method1方法");
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        final OutClass.InnerClass1 in1=new OutClass.InnerClass1();
        final OutClass.InnerClass2 in2=new OutClass.InnerClass2();
        Thread t1=new Thread(new Runnable() {
            @Override
            public void run() {
                in1.method1(in2);
            }
        },"T1");

        Thread t2=new Thread(new Runnable() {
            @Override
            public void run() {
                in1.method2();
            }
        },"T2");

        Thread t3=new Thread(new Runnable() {
            @Override
            public void run() {
                in2.method1();
            }
        },"T3");
        t1.start();
        t2.start();
        t3.start();
    }
}
```
程序的运行结果为：
```java
T1 进入InnerClass1类中的method1方法
T2 进入InnerClass1类中的method2方法
T1 i=1
j=0
T1 i=2
....
T1 离开InnerClass1类中的method1方法
T3 进入InnerClass2类中的method1方法
j=9
k=0
```
> 摘选自 java多线程核心编程技术-2.2.15