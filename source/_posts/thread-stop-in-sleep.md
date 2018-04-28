---
title: 1.7.4在沉睡中停止
date: 2017-07-13 22:19:58
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

如果线程在sleep()状态下停止线程，会是什么效果呢？
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
        super.run();
        try {
            System.out.println("run begin");
            Thread.sleep(200000);
            System.out.println("run end");
        }catch(InterruptedException e){
            System.out.println("在沉睡中被停止！进入catch！"+this.isInterrupted());
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        try {
            MyThread myThread=new MyThread();
            myThread.start();
            Thread.sleep(200);
            myThread.interrupt();
        } catch (InterruptedException e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end");
    }
}

```
程序的运行结果如下：
```java
run begin
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at wys.test.MyThread.run(MyThread.java:15)
end
在沉睡中被停止！进入catch！false
```
从输出的结果来看，如果在sleep状态下停止某一线程，会进入catch语句中，并清除停止状态值，使之变成false

前一个实验是先sleep然后再用interrupt()停止，与之相反的操作在学习线程时也要注意。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
        super.run();
        try {
            for (int i=0;i<100000;i++){
                System.out.println("i="+(i+1));
            }
            System.out.println("run begin");
            Thread.sleep(200000);
            System.out.println("run end");
        }catch(InterruptedException e){
            System.out.println("先停止，再遇到了sleep！进入catch！");
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        myThread.start();
        myThread.interrupt();
        System.out.println("end!");
    }
}
```
运行结果如下:
```java
i=99996
i=99997
i=99998
i=99999
i=100000
run begin
先停止，再遇到了sleep！进入catch！
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at wys.test.MyThread.run(MyThread.java:18)
```

> 摘选自 java多线程核心编程技术-1.7.4