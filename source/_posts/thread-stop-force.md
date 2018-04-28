---
title: 1.7.5能停止的线程——暴力停止
date: 2017-07-13 22:22:09
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
使用stop()方法停止线程则是非常暴力的。
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
        super.run();
        try {
            while(true){
                i++;
                System.out.println("i="+i);
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
        try {
            MyThread myThread=new MyThread();
            myThread.start();
            Thread.sleep(8000);
            myThread.stop();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果为：
```java
i=1
i=2
i=3
i=4
i=5
i=6
i=7
i=8

Process finished with exit code 0
```
> 摘选自 java多线程核心编程技术-1.7.5