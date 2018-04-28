---
title: 1.8.2suspend与resume方法的缺点——独占
date: 2017-07-13 22:31:10
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

在使用suspend与resume方法时，如果使用不当，极易造成公共的同步对象的独占，使得其他线程无法访问公共同步对象。
```java
/**
 * @author wuyoushan
 * @date 2017/3/27.
 */
public class SynchronizedObject {

    synchronized  public void printString(){
        System.out.println("begin");
        if(Thread.currentThread().getName().equals("a")){
            System.out.println("a线程永远suspend了！");
            Thread.currentThread().suspend();
        }
        System.out.println("end");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try {
            final SynchronizedObject object = new SynchronizedObject();
            Thread thread1 = new Thread() {
                @Override
                public void run() {
                    object.printString();
                }
            };

            thread1.setName("a");
            thread1.start();
            Thread.sleep(1000);
            Thread thread2=new Thread(){
                @Override
                public void run() {
                    System.out.println("thread2启动了，但进入不了printString()方法！只打印1个begin");
                    System.out.println("因为printString()方法被a线程锁定并且永远suspend暂停了！");
                    object.printString();
                }
            };
            thread2.start();
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

```
运行结果为：
```java
begin
a线程永远suspend了！
thread2启动了，但进入不了printString()方法！只打印1个begin
因为printString()方法被a线程锁定并且永远suspend暂停了！
```

还有另外一种独占锁的情况也要格外的注意，稍有不慎，就会掉进“坑”里。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private long i=0;

    @Override
    public void run() {
       while (true){
          i++;
       }
    }
}


/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try {
            MyThread thread = new MyThread();
            thread.start();
            Thread.sleep(1000);
            thread.suspend();
            System.out.println("main end!");
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}

```
运行结果为：
```java
main end!
```

如果将线程类MyThread.java更改如下：
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private long i=0;

    @Override
    public void run() {
       while (true){
          i++;
           System.out.println("i="+i);
       }
    }
}
```
再次运行程序，控制台将不打印main end，运行结果如下
```java
i=98713
i=98714
i=98715
i=98716
i=98717
i=98718
i=98719
```
出现这样结果的原因是，当程序运行到println()方法内部停止时，同步锁未被释放。方法println()源代码如下:
```java
public void println(String x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }
```
这导致当前PrintStream对象的println()方法一直呈“暂停”状态，并且“锁未释放”，
而main()方法中的代码System.out.println("main end!");迟迟不能执行打印。
虽然suspuend()方法是过期作废的方法，但还是有必要研究它过期作废的原因，这是很有意义的。

> 摘选自 java多线程核心编程技术-1.8.2