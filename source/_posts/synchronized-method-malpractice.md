---
title: 2.2.1synchronized方法的弊端
date: 2017-07-18 08:05:26
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
用关键字synchronized声明方法在某些情况下时有弊端的，比如A线程调用同步方法执行一个长时间的任务，那么B线程则必须等待比较长的时间。在这样的情况下可以使用synchronized同步语句块来解决

**synchronized方法的弊端**
```java
/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class Task {
    private String getData1;
    private String getData2;

    public synchronized void doLongTimeTask(){
        try {
            System.out.println("begin task");
            Thread.sleep(3000);
            getData1="长时间处理任务后从远程返回的值1 threadName="
                    +Thread.currentThread().getName();
            getData2="长时间处理任务后从远程返回的值2 threadName="
                    +Thread.currentThread().getName();
            System.out.println(getData1);
            System.out.println(getData2);
            System.out.println("end task");
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/12.
 */
public class CommonUtils {
    public static long beginTime1;
    public static long endTime1;
    public static long beginTime2;
    public static long endTime2;
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Task task;

    public ThreadA(Task task){
        super();
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
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Task task;

    public ThreadB(Task task){
        super();
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

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    public static void main(String[] args){
        Task task=new Task();
        ThreadA thread1=new ThreadA(task);
        thread1.start();
        ThreadB thread2=new ThreadB(task);
        thread1.start();

        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        long beginTime=CommonUtils.beginTime1;
        if(CommonUtils.beginTime2<CommonUtils.beginTime1){
            beginTime=CommonUtils.beginTime2;
        }

        long endTime=CommonUtils.endTime1;
        if (CommonUtils.endTime2>CommonUtils.endTime1){
            endTime=CommonUtils.endTime2;
        }

        System.out.println("耗时:"+(endTime-beginTime)/1000);
    }
}
```
程序的运行结果为:
```java
begin task
长时间处理任务后从远程返回的值1 threadName=Thread-0
长时间处理任务后从远程返回的值2 threadName=Thread-0
end task
begin task
长时间处理任务后从远程返回的值1 threadName=Thread-1
长时间处理任务后从远程返回的值2 threadName=Thread-1
end task
耗时：6

```
在使用synchronized关键字来声明方法public synchronized void doLongTimeTask()时从运行的时间上来看，弊端很明显(排序执行，花时间较长)，解决这样的问题可以使用synchronized同步块

> 摘选自 java多线程核心编程技术-2.2.1