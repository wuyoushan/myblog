---
title: 5.1定时器Timer的使用
date: 2017-07-23 22:41:16
tags: [java,定时器timer,读书笔记]
categories: [java,读书笔记]
---
在JDK库中Timer类主要负责计划任务的功能，也就是在指定的时间开始执行某一个任务。

Timer类的主要作用就是设置计划任务，但封装任务的类却是TimerTask类。

**2.1.1方法schedule(TimerTask task,Date Time)的测试**

该方法的作用是在指定的日期执行一次某一任务。

**1. 执行任务的时间晚于当前时间：在未来执行的效果**
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    private static Timer timer=new Timer();

    static public class MyTask extends TimerTask {
        @Override
        public void run() {
            System.out.println("运行了！时间为："+new Date());
        }
    }

    public static void main(String[] args) {
        try{
            MyTask task=new MyTask();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-5-12 7:45:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef);
        }catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序运行完后的结果为如下，任务虽然执行完了，但进程还未销毁，呈红色状态
```java
字符串时间:Fri May 12 07:45:00 CST 2017当前时间:Fri May 12 07:43:37 CST 2017
运行了！时间为：Fri May 12 07:45:00 CST 2017
```
为什么会出现这样的情况？
在创建Timer对象时，JDK源代码如下：
```java
public Timer(){
    this("Timer")
}

此构造方法调用的是如下构造方法

public Timer(String name) {
    thread.setName(name);
    thread.start();
}
```
查看构造方法可以得知，创建一个Timer就是启动一个新的线程，这个新启动的线程并不是守护线程，他一直在运行。

新建Timer改成守护线程
```java
/**
 * @author wuyoushan
 * @date 2017/5/12.
 */
public class Run1TimerIsDaemon {

    private static Timer timer=new Timer(true);

    static public class MyTask extends TimerTask{

        @Override
        public void run() {
            System.out.println("运行了！时间为:"+new Date());
        }
    }

    public static void main(String[] args) {
        try{
            MyTask task=new MyTask();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-5-12 8:02:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef);
        }catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Fri May 12 08:02:00 CST 2017当前时间:Fri May 12 08:01:16 CST 2017
```
程序运行后迅速结束当前的进程，并且TimerTask中的任务不再被运行，因为进程已经结束了。

**2. 计划时间早于当前时间：提前运行的效果**

如果执行任务的时间早于当前时间，则立即执行task任务。
```java
/**
 * @author wuyoushan
 * @date 2017/5/12.
 */
public class Run1TimerIsDaemon {

    private static Timer timer=new Timer(true);

    static public class MyTask extends TimerTask{

        @Override
        public void run() {
            System.out.println("运行了！时间为:"+new Date());
        }
    }

    public static void main(String[] args) {
        try{
            MyTask task=new MyTask();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-5-12 8:02:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef);
        }catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果如下所示,立即执行task任务:
```java
字符串时间:Fri May 12 08:02:00 CST 2017当前时间:Mon May 15 08:31:51 CST 2017
运行了！时间为：Mon May 15 08:31:51 CST 2017
```
**3. 多个TimerTask任务及延时的测试**

Timer中允许有多个TimerTask任务。
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    private static Timer timer=new Timer();

    static public class MyTask1 extends TimerTask {
        @Override
        public void run() {
            System.out.println("运行了！时间为："+new Date());
        }
    }

    static public class MyTask2 extends TimerTask {
        @Override
        public void run() {
            System.out.println("运行了！时间为："+new Date());
        }
    }

    public static void main(String[] args) {
        try{
            MyTask1 task1=new MyTask1();
            MyTask2 task2=new MyTask2();
            SimpleDateFormat sdf1=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            SimpleDateFormat sdf2=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString1="2017-5-15 8:48:00";
            String dateString2="2017-5-15 8:49:00";
            Date dateRef1=sdf1.parse(dateString1);
            Date dateRef2=sdf2.parse(dateString2);
            System.out.println("字符串1时间:"+dateRef1.toString()+"当前时间:"+new Date().toString());
            System.out.println("字符串2时间:"+dateRef2.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task1,dateRef1);
            timer.schedule(task2,dateRef2);
        }catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为：
```java
字符串1时间:Mon May 15 08:48:00 CST 2017当前时间:Mon May 15 08:47:42 CST 2017
字符串2时间:Mon May 15 08:49:00 CST 2017当前时间:Mon May 15 08:47:42 CST 2017
运行了！时间为：Mon May 15 08:48:00 CST 2017
运行了！时间为：Mon May 15 08:49:00 CST 2017
```
TimerTask是以队列的方式一个一个被顺序执行的，所以执行的时间有可能和预期的时间不一致，因为前面的任务有可能消耗的时间较长，则后面的任务运行的时间也会被延迟。
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    private static Timer timer=new Timer();

    static public class MyTask1 extends TimerTask {
        @Override
        public void run() {
            try {
                System.out.println("1 begin 运行了！时间为："+new Date());
                Thread.sleep(20000);
                System.out.println("1 end 运行了！时间为:"+new Date());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    static public class MyTask2 extends TimerTask {
        @Override
        public void run() {
            System.out.println("2 begin 运行了！时间为："+new Date());
            System.out.println("运行了！时间为:"+new Date());
            System.out.println("2 end 运行了！时间为:"+new Date());
        }
    }

    public static void main(String[] args) {
        try{
            MyTask1 task1=new MyTask1();
            MyTask2 task2=new MyTask2();
            SimpleDateFormat sdf1=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            SimpleDateFormat sdf2=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString1="2017-5-16 8:09:00";
            String dateString2="2017-5-16 8:09:00";
            Date dateRef1=sdf1.parse(dateString1);
            Date dateRef2=sdf2.parse(dateString2);
            System.out.println("字符串1时间:"+dateRef1.toString()+"当前时间:"+new Date().toString());
            System.out.println("字符串2时间:"+dateRef2.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task1,dateRef1);
            timer.schedule(task2,dateRef2);
        }catch (ParseException e) {
            e.printStackTrace();
        }
    }
}

```
程序的运行结果为:
```java
字符串1时间:Tue May 16 08:09:00 CST 2017当前时间:Tue May 16 08:08:11 CST 2017
字符串2时间:Tue May 16 08:09:00 CST 2017当前时间:Tue May 16 08:08:11 CST 2017
1 begin 运行了！时间为：Tue May 16 08:09:00 CST 2017
1 end 运行了！时间为:Tue May 16 08:09:20 CST 2017
2 begin 运行了！时间为：Tue May 16 08:09:20 CST 2017
运行了！时间为:Tue May 16 08:09:20 CST 2017
2 end 运行了！时间为:Tue May 16 08:09:20 CST 2017
```
由于task1需要用时20秒执行完任务，task1开始的时间是8:09以此时间为基准，向后延迟20秒，task2在8:09:20执行任务。因为Task是被放入队列中的，得一个一个顺序运行。

> 摘选自 java多线程核心编程技术-5.1