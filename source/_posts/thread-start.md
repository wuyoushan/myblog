---
title: 1.2.1线程的启动顺序与start()的执行顺序无关
date: 2017-07-09 14:32:12
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
执行start()方法的顺序不代表线程启动的顺序。
```java
/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class MyThread extends Thread {

    private int i;
    public MyThread(String name){
        super(name);
    }
    public MyThread(String name,int i){
        super(name);
        this.i=i;
    }
    @Override
    public void run() {
        super.run();
        System.out.println("当前线程为:"
                +Thread.currentThread().getName()+
                "\t输出的值为:"+i);
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class Test {
    public static void main(String[] args) {
        MyThread t1=new MyThread("MyThread1",1);
        MyThread t2=new MyThread("MyThread2",2);
        MyThread t3=new MyThread("MyThread3",3);
        MyThread t4=new MyThread("MyThread4",4);
        MyThread t5=new MyThread("MyThread5",5);
        MyThread t6=new MyThread("MyThread6",6);
        MyThread t7=new MyThread("MyThread7",7);
           t1.start();
           t2.start();
           t3.start();
           t4.start();
           t5.start();
           t6.start();
           t7.start();
    }
}
```
#### 执行结果
```java 
当前线程为:MyThread1	输出的值为:1
当前线程为:MyThread3	输出的值为:3
当前线程为:MyThread5	输出的值为:5
当前线程为:MyThread4	输出的值为:4
当前线程为:MyThread2	输出的值为:2
当前线程为:MyThread7	输出的值为:7
当前线程为:MyThread6	输出的值为:6
```
> 摘选自 java多线程核心编程技术-1.2.1