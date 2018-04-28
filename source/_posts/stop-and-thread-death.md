---
title: 1.7.6方法stop()与java.lang.ThreadDeath异常
date: 2017-07-13 22:24:30
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

调用stop()方法时会抛出java.lang.ThreadDeath异常，但在通常的情况下，此异常不需要显式地捕捉。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private int i=0;
    @Override
    public void run() {
        try {
            this.stop();
        }catch(ThreadDeath e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        MyThread myThread=new MyThread();
        myThread.start();
    }
}

```
运行结果为：
```java
java.lang.ThreadDeath
	at java.lang.Thread.stop(Thread.java:836)
	at wys.test.MyThread.run(MyThread.java:14)

Process finished with exit code 0
```
方法stop()已经被作废，因为如果强制让线程停止则有可能使一些清理性的工作得不到完成。另外一个情况就是对锁定的对象进行了“解锁”，导致数据得不到同步的处理，出现数据不一致的问题。

> 摘选自 java多线程核心编程技术-1.7.6