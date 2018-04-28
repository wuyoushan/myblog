---
title: 2.3.2解决同步死循环
date: 2017-07-23 14:42:11
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
```java
/**
 * @author wuyoushan
 * @date 2017/5/3.
 */
public class PrintString implements Runnable{
    private boolean isContinuePrint=true;

    public boolean isContinuePrint() {
        return isContinuePrint;
    }

    public void setContinuePrint(boolean continuePrint) {
        isContinuePrint = continuePrint;
    }

    public void printStringMethod(){
        try {
            while (isContinuePrint == true) {
                System.out.println("run printStringMethod threadName=" +
                        Thread.currentThread().getName());
                Thread.sleep(1000);
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        printStringMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        PrintString printStringService=new PrintString();
        new Thread(printStringService).start();
        System.out.println("我要停止它！stopThread="+Thread.currentThread().getName());
        printStringService.setContinuePrint(false);
    }
}
```
程序的运行结果为：
```java
我要停止它！stopThread=main
run printStringMethod threadName=Thread-0
```
但当上面的示例代码的格式运行在-server服务器模式中64bit的JVM上时，会出现死循环。解决的办法是使用volatile关键字

关键字volatile的作用是强制从公共堆栈中取得变量的值，而不是从线程私有数据栈中取得变量的值。

> 摘选自 java多线程核心编程技术-2.3.2