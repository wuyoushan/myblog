---
title: 1.7.1停不了的线程
date: 2017-07-09 15:33:16
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
停止一个线程意味着在线程处理完任务之前停掉正在做的操作，也就是放弃当前的操作。虽然这看起来非常简单，但是必须做好防范措施，以便达到预期的效果。停止一个线程可以用Thread.stop()方法，但最好不用它。虽然它确实可以停止一个正在运行的线程，但是这个方法是不安全的(unsafe),而且是已被弃用作废的(deprecated),在将来的java版本中，这个方法将不可用或不被支持。

大多数停止一个线程的操作使用Thread.interrupt()方法,尽管方法的名称是“停止，中止”的意思，但这个方法不会终止一个正在运行的线程，还需要加入一个判断才可以完成线程的停止。

#### 在java中有以下3种方法可以终止正在运行的线程:
1. 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
2. 使用stop()方法强行终止线程，但是不推荐使用这个方法，因为stop和suspend及resume一样，都是作废过期的方法，使用它们可能产生不可预料的结果
3. 使用interrupt方法中断线程。


#### 停不了的线程
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

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        myThread.start();
        try {
            Thread.sleep(2000);
            //中断线程
            myThread.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
从程序的运行结果来看，调用interrupt方法并没有停止线程。
```java
i=499994
i=499995
i=499996
i=499997
i=499998
i=499999
i=500000
```

> 摘选自 java多线程核心编程技术-1.7.1