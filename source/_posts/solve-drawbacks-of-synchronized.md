---
title: 2.2.3用同步代码块解决同步方法的弊端
date: 2017-07-18 08:10:31
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

```java
/**
 * 线程任务
 * 用同步代码块解决同步方法的弊端
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class Task {
    private String getData1;
    private String getData2;

    public void doLongTimeTask(){
        try {
            System.out.println("begin task");
            Thread.sleep(3000);
            String privateGetData1="长时间处理任务后从远程返回的值1 threadName="+
                    Thread.currentThread().getName();
            String privateGetData2="长时间处理任务后从远程返回的值2 threadName="+
                    Thread.currentThread().getName();
            synchronized (this){
                getData1=privateGetData1;
                getData2=privateGetData2;
            }
            System.out.println(getData1);
            System.out.println(getData2);
            System.out.println("end task");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

/**
 * 常量定义
 * 用同步代码块解决同步方法的弊端
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class CommonUtils {
    public static long beginTime1;
    public static long endTime1;
    public static long beginTime2;
    public static long endTime2;
}


/**线程1
 * 用同步代码块解决同步方法的弊端
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class MyThread1 extends Thread {
    private Task task;

    public MyThread1(Task task){
        this.task=task;
    }

    @Override
    public void run() {
        super.run();
        CommonUtils.beginTime1=System.currentTimeMillis();
        task.doLongTimeTask();
        CommonUtils.endTime1=System.currentTimeMillis();
    }
}

/**
 * 线程二
 * synchronized方法的弊端
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class MyThread2 extends Thread {
    private Task task;

    public MyThread2(Task task){
        this.task=task;
    }

    @Override
    public void run() {
        super.run();
        CommonUtils.beginTime2=System.currentTimeMillis();
        task.doLongTimeTask();
        CommonUtils.endTime2=System.currentTimeMillis();
    }
}

/**运行实例
 * 用同步代码块解决同步方法的弊端
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class Run {
    public static void main(String[] args) {
        Task task=new Task();
        MyThread1 thread1=new MyThread1(task);
        thread1.start();
        MyThread2 thread2=new MyThread2(task);
        thread2.start();
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long beginTime=CommonUtils.beginTime1;
        if (CommonUtils.beginTime2<CommonUtils.beginTime1){
            beginTime=CommonUtils.beginTime2;
        }
        long endTime=CommonUtils.endTime1;
        if (CommonUtils.endTime2>CommonUtils.endTime1){
            endTime=CommonUtils.endTime2;
        }
        System.out.println("耗时:"+((endTime-beginTime)/1000));
    }
}

```
程序的运行结果为：
```java
begin task
begin task
长时间处理任务后从远程返回的值1 threadName=Thread-1
长时间处理任务后从远程返回的值2 threadName=Thread-0
end task
长时间处理任务后从远程返回的值1 threadName=Thread-0
长时间处理任务后从远程返回的值2 threadName=Thread-0
end task
耗时:3

```
通过上面的实验可以得知，当一个线程访问object的一个synchronized同步代码块时，另一个线程仍然可以访问改object对象中的非synchronized(this)同步代码块。

实验进行到这里，虽然时间缩短，运行效率加快，但同步synchronized代码块真的是同步的吗？真的持有当前调用对象的锁吗？答案为是，但必须用代码的方式来进行验证

> 摘选自 java多线程核心编程技术-2.2.3