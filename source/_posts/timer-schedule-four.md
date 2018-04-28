---
title: 5.1.5方法scheduleAtFixedRete(TimerTask task,Date firstTime,long period)的测试
date: 2017-07-23 22:46:29
tags: [java,定时器timer,读书笔记]
categories: [java,读书笔记]
---

方法schedule和方法scheduleAtFixedRate都会按顺序执行，所以不要考虑非线程安全的情况。
方法schedule和方法scheduleAtFixedRate只在于不延时的情况。

使用schedule方法:如果执行任务的时间没有被延时，那么下一次任务的执行时间参考的是上一次任务的“开始”时的时间来计算。

使用scheduleAtFixedRate方法:如果执行任务的时间没有被延时，那么下一次任务的执行时间参考的是上一次任务的“结束”时的时间来计算。

延时的情况则没有区别，也就是使用schedule或scheduleAtFixedRate方法都是如果执行任务的时间被延时，那么下一次任务的执行时间参考的是上一次任务“结束”时的时间来计算。

**1. 测试schedule方法任务不延时**

```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    private static Timer timer=new Timer();

    private static int runCount=0;

    static public class MyTask1 extends TimerTask {
        @Override
        public void run() {
            try{
                System.out.println("1 begin 运行了！时间为:"+new Date());
                Thread.sleep(1000);
                System.out.println("1 end 运行了！时间为:"+new Date());
                runCount++;
                if (runCount==5){
                    timer.cancel();
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        try {
            MyTask1 task = new MyTask1();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-05-19 08:44:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef,3000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Fri May 19 08:44:00 CST 2017当前时间:Mon May 22 08:12:39 CST 2017
1 begin 运行了！时间为:Mon May 22 08:12:39 CST 2017
1   end 运行了！时间为:Mon May 22 08:12:40 CST 2017
1 begin 运行了！时间为:Mon May 22 08:12:42 CST 2017
1   end 运行了！时间为:Mon May 22 08:12:43 CST 2017
1 begin 运行了！时间为:Mon May 22 08:12:45 CST 2017
1   end 运行了！时间为:Mon May 22 08:12:46 CST 2017
1 begin 运行了！时间为:Mon May 22 08:12:48 CST 2017
1   end 运行了！时间为:Mon May 22 08:12:49 CST 2017
1 begin 运行了！时间为:Mon May 22 08:12:51 CST 2017
1   end 运行了！时间为:Mon May 22 08:12:52 CST 2017
```
控制台大印的结果证明，在不延迟的情况下，如果执行任务的时间没有被延时，则下一次执行任务的时间是上一次任务的开始时间加上delay时间

**2. 测试schedule方法任务**
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    private static Timer timer=new Timer();

    private static int runCount=0;

    static public class MyTask1 extends TimerTask {
        @Override
        public void run() {
            try{
                System.out.println("1 begin 运行了！时间为:"+new Date());
                Thread.sleep(5000);
                System.out.println("1   end 运行了！时间为:"+new Date());
                runCount++;
                if (runCount==5){
                    timer.cancel();
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        try {
            MyTask1 task = new MyTask1();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-05-20 08:44:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef,2000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Sat May 20 08:44:00 CST 2017当前时间:Mon May 22 08:23:35 CST 2017
1 begin 运行了！时间为:Mon May 22 08:23:35 CST 2017
1   end 运行了！时间为:Mon May 22 08:23:40 CST 2017
1 begin 运行了！时间为:Mon May 22 08:23:40 CST 2017
1   end 运行了！时间为:Mon May 22 08:23:45 CST 2017
1 begin 运行了！时间为:Mon May 22 08:23:45 CST 2017
1   end 运行了！时间为:Mon May 22 08:23:50 CST 2017
1 begin 运行了！时间为:Mon May 22 08:23:50 CST 2017
1   end 运行了！时间为:Mon May 22 08:23:55 CST 2017
1 begin 运行了！时间为:Mon May 22 08:23:55 CST 2017
1   end 运行了！时间为:Mon May 22 08:24:00 CST 2017
```
从控制台打印的结果来看，如果执行任务的时间被延迟，那么下一次任务的执行时间以上一次任务“结束”时的时间为参考计算

**3. 测试scheduleAtFixedRate方法任务不延时**
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    private static Timer timer=new Timer();

    private static int runCount=0;

    static public class MyTask1 extends TimerTask {
        @Override
        public void run() {
            try{
                System.out.println("1 begin 运行了！时间为:"+new Date());
                Thread.sleep(2000);
                System.out.println("1   end 运行了！时间为:"+new Date());
                runCount++;
                if (runCount==5){
                    timer.cancel();
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        try {
            MyTask1 task = new MyTask1();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-05-20 08:44:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef,3000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Sat May 20 08:44:00 CST 2017当前时间:Mon May 22 08:25:46 CST 2017
1 begin 运行了！时间为:Mon May 22 08:25:46 CST 2017
1   end 运行了！时间为:Mon May 22 08:25:48 CST 2017
1 begin 运行了！时间为:Mon May 22 08:25:49 CST 2017
1   end 运行了！时间为:Mon May 22 08:25:51 CST 2017
1 begin 运行了！时间为:Mon May 22 08:25:52 CST 2017
1   end 运行了！时间为:Mon May 22 08:25:54 CST 2017
1 begin 运行了！时间为:Mon May 22 08:25:55 CST 2017
1   end 运行了！时间为:Mon May 22 08:25:57 CST 2017
1 begin 运行了！时间为:Mon May 22 08:25:58 CST 2017
1   end 运行了！时间为:Mon May 22 08:26:00 CST 2017
```
控制台打印的结果证明，如果执行任务的时间没有被延迟时，则下一次执行任务的时间是上一次任务的开始时间加上delay时间。

**4. 测试scheduleAtFixedRate方法任务延时**
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    private static Timer timer=new Timer();

    private static int runCount=0;

    static public class MyTask1 extends TimerTask {
        @Override
        public void run() {
            try{
                System.out.println("1 begin 运行了！时间为:"+new Date());
                Thread.sleep(5000);
                System.out.println("1   end 运行了！时间为:"+new Date());
                runCount++;
                if (runCount==5){
                    timer.cancel();
                }
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        try {
            MyTask1 task = new MyTask1();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-05-20 08:44:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.scheduleAtFixedRate(task,dateRef,2000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Sat May 20 08:44:00 CST 2017当前时间:Mon May 22 08:46:03 CST 2017
1 begin 运行了！时间为:Mon May 22 08:46:03 CST 2017
1   end 运行了！时间为:Mon May 22 08:46:08 CST 2017
1 begin 运行了！时间为:Mon May 22 08:46:08 CST 2017
1   end 运行了！时间为:Mon May 22 08:46:13 CST 2017
1 begin 运行了！时间为:Mon May 22 08:46:13 CST 2017
1   end 运行了！时间为:Mon May 22 08:46:18 CST 2017
1 begin 运行了！时间为:Mon May 22 08:46:18 CST 2017
1   end 运行了！时间为:Mon May 22 08:46:23 CST 2017
1 begin 运行了！时间为:Mon May 22 08:46:23 CST 2017
1   end 运行了！时间为:Mon May 22 08:46:28 CST 2017
```
从控制台打印的结果来看，如果执行任务的时间被延迟时，那么下一次任务的执行时间以上一次任务“结束”时的时间为参考来计算。

**5. 验证schedule方法不具有追赶执行性**
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
            System.out.println("1 begin 运行了！时间为:"+new Date());
            System.out.println("1   end 运行了！时间为:"+new Date());
        }
    }

    public static void main(String[] args) {
        try {
            MyTask1 task = new MyTask1();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-05-20 08:44:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef,5000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Sat May 20 08:44:00 CST 2017当前时间:Mon May 22 08:39:29 CST 2017
1 begin 运行了！时间为:Mon May 22 08:39:29 CST 2017
1   end 运行了！时间为:Mon May 22 08:39:29 CST 2017
1 begin 运行了！时间为:Mon May 22 08:39:34 CST 2017
1   end 运行了！时间为:Mon May 22 08:39:34 CST 2017
1 begin 运行了！时间为:Mon May 22 08:39:39 CST 2017
1   end 运行了！时间为:Mon May 22 08:39:39 CST 2017
1 begin 运行了！时间为:Mon May 22 08:39:44 CST 2017
1   end 运行了！时间为:Mon May 22 08:39:44 CST 2017
```
**6. 验证scheduleAtFixedRate方法具有追赶执行性**
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
            System.out.println("1 begin 运行了！时间为:"+new Date());
            System.out.println("1   end 运行了！时间为:"+new Date());
        }
    }

    public static void main(String[] args) {
        try {
            MyTask1 task = new MyTask1();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-05-20 08:44:00";
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.scheduleAtFixedRate(task,dateRef,2000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
end 运行了！时间为:Mon May 22 08:52:24 CST 2017
1 begin 运行了！时间为:Mon May 22 08:52:24 CST 2017
1   end 运行了！时间为:Mon May 22 08:52:24 CST 2017
1 begin 运行了！时间为:Mon May 22 08:52:24 CST 2017
1   end 运行了！时间为:Mon May 22 08:52:24 CST 2017
1 begin 运行了！时间为:Mon May 22 08:52:24 CST 2017
1   end 运行了！时间为:Mon May 22 08:52:24 CST 2017
1 begin 运行了！时间为:Mon May 22 08:52:24 CST 2017
1   end 运行了！时间为:Mon May 22 08:52:24 CST 2017
1 begin 运行了！时间为:Mon May 22 08:52:24 CST 2017
1   end 运行了！时间为:Mon May 22 08:52:24 CST 2017
1 begin 运行了！时间为:Mon May 22 08:52:24 CST 2017
1   end 运行了！时间为:Mon May 22 08:52:24 CST 2017
```
两个时间段内所对应的Task任务被“补充性”执行了，这就是Task任务追赶执行的特性。

> 摘选自 java多线程核心编程技术-5.1.5