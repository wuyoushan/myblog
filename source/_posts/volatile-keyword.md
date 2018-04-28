---
title: 2.3volatile关键字
date: 2017-07-23 14:39:20
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
关键字volatile的主要作用是使变量在多个线程间可见。

#### 关键字volatile与死循环
如果不是在多继承的情况下，使用继承Thread类和实现Runable接口在取得程序运行的结果上并没有什么太大的区别。如果一旦出现“多继承”的情况，则用实现Runable接口的方式来处理多线程的问题就是很有必要的。

本节将用实现Runable接口的方式来继续理解多线程技术的使用，并且使用关键字volatile来实验在并发情况下的一些特性。此案例也同样适用于继承自Thread类
```java
/**
 * @author wuyoushan
 * @date 2017/5/3.
 */
public class PrintString {
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
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        PrintString printStringService=new PrintString();
        printStringService.printStringMethod();
        System.out.println("我要停止它！stopThread="+Thread.currentThread().getName());
        printStringService.setContinuePrint(false);
    }
}

```
程序开始运行后，根本停不下来。结果如下:
```java
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
```
停不下来的原因主要是main线程一直在处理while()循环，导致程序不能继续执行后面的代码，解决的办法当然就是多线程。

> 摘选自 java多线程核心编程技术-2.3