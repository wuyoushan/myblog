---
title: 1.7.2判断线程是否是停止的状态
date: 2017-07-09 16:18:53
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

在java的SDK中，Thread.java类里提供了两种方法。
1. this.interrupted():测试当前线程是否已中断。
2. this.isInterrupted():测试线程是否已经中断。
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
        for (int i=0;i<500000;i++){
            System.out.println("i="+(i+1));
        }
    }
}
```
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        try {
            MyThread myThread=new MyThread();
            myThread.start();
            Thread.sleep(1000);
            //中断线程
            myThread.interrupt();
            System.out.println("是否停止1？="+myThread.interrupted());
            System.out.println("是否停止2？="+myThread.interrupted());
        } catch (InterruptedException e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end");
    }
}
```
控制台打印的结果:
```java
是否停止1？=false
是否停止2？=false
end
```
从控制台打印的结果来看，线程并未停止，这也就证明了interrupted()方法的解释:测试当前线程是否已经中断。这个“当前线程”是main，它从未中断过，所以打印的结果是两个false

```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run2 {
    public static void main(String[] args) {
       Thread.currentThread().interrupt();
       System.out.println("是否停止1？="+Thread.interrupted());
       System.out.println("是否停止2？="+Thread.interrupted());
        System.out.println("end");
    }
}
```
控制台打印出来的结果:
```java
是否停止1？=true
是否停止2？=false
end
```
从上述的结果来看，方法interrupted()的确判断出当前线程是否是停止状态。但为什么第二个布尔值是false呢？查看一下官方帮助文档中对interrupted方法的解释:

**测试当前线程是否已经中断。线程的中断状态由该方法清除。换句话说，如果连续两次调用该方法，则第二次调用将返回false（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）**

文档已经解析得很详细，interrupted()方法具有清除状态的功能，所以第2次调用interrupted()方法返回的值是false

interrupted():测试当前线程是否已经是中断状态，执行后具有将状态标志置清除为false的功能。

isInterrupted()：测试线程Thread对象是否已经是中断状态，但不清除中断状态标志。

> 摘选自 java多线程核心编程技术-1.7.2