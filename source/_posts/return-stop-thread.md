---
title: 1.7.8使用return停止线程
date: 2017-07-13 22:28:28
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

将方法interrupt()与return结合使用也能实现停止线程的效果。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {
    
    @Override
    public void run() {
       while (true){
           if(this.isInterrupted()){
               System.out.println("停止了！");
               return;
           }
           System.out.println("timer="+ System.currentTimeMillis());
       }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try{
            MyThread thread=new MyThread();
            thread.start();
            Thread.sleep(200);
            thread.interrupt();
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
运行结果为：
```java
timer=1490575062355
timer=1490575062355
timer=1490575062355
timer=1490575062355
timer=1490575062355
timer=1490575062355
timer=1490575062355
停止了！
```
不过还是建议使用“抛异常”的方法来实现线程的停止，因为在catch块中还可以将异常向上抛，使线程停止的事件得以传播。

> 摘选自 java多线程核心编程技术-1.7.8
