---
title: 5.1.2方法schedule(TimerTask task,Date firstTime,long period)的测试
date: 2017-07-23 22:43:10
tags: [java,定时器timer,读书笔记]
categories: [java,读书笔记]
---
该方法的作用是在指定的期之后，按指定的间隔周期性地无限循环地执行某一任务。

**1. 计划任务晚于当前任务:在未来执行的效果**
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    static public class MyTask extends TimerTask {
        @Override
        public void run() {
            System.out.println("运行了！时间为:"+new Date());
        }
    }

    public static void main(String[] args) {
        try{
            MyTask task=new MyTask();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-5-15 8:09:00";
            Timer timer=new Timer();
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef,4000);
        }catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Tue May 16 08:09:00 CST 2017当前时间:Tue May 16 08:22:17 CST 2017
运行了！时间为:Tue May 16 08:22:17 CST 2017
运行了！时间为:Tue May 16 08:22:21 CST 2017
运行了！时间为:Tue May 16 08:22:25 CST 2017
运行了！时间为:Tue May 16 08:22:29 CST 2017
```
从运行的结果来看，每隔4秒运行一次TimerTask任务，并且是无限期地重复执行。

**2. 计划任务早于当前时间：提前运行的效果**
如果计划时间早于当前时间，则立即执行task任务。
```java
public static void main(String[] args) {
        try{
            MyTask task=new MyTask();
            SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString="2017-5-15 8:09:00";
            Timer timer=new Timer();
            Date dateRef=sdf.parse(dateString);
            System.out.println("字符串时间:"+dateRef.toString()+"当前时间:"+new Date().toString());
            timer.schedule(task,dateRef,4000);
        }catch (ParseException e) {
            e.printStackTrace();
        }
}
```
程序的运行结果为:
```java
字符串时间:Mon May 15 08:09:00 CST 2017当前时间:Tue May 16 08:30:33 CST 2017
运行了！时间为:Tue May 16 08:30:33 CST 2017
运行了！时间为:Tue May 16 08:30:37 CST 2017
运行了！时间为:Tue May 16 08:30:41 CST 2017
```

**3. 任务执行时间被延时**
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    static public class MyTaskA extends TimerTask {
        @Override
        public void run() {
            try {
                System.out.println("A运行了！时间为:" + new Date());
                Thread.sleep(5000);
                System.out.println("A结束了！时间为:" + new Date());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        try {
            MyTaskA taskA = new MyTaskA();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString = "2017-5-15 8:09:00";
            Timer timer = new Timer();
            Date dateRef = sdf.parse(dateString);
            System.out.println("字符串时间:" + dateRef.toString() + "当前时间:" + new Date().toString());
            timer.schedule(taskA, dateRef, 4000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果:
```java
字符串时间:Mon May 15 08:09:00 CST 2017当前时间:Tue May 16 08:39:18 CST 2017
A运行了！时间为:Tue May 16 08:39:18 CST 2017
A结束了！时间为:Tue May 16 08:39:23 CST 2017
A运行了！时间为:Tue May 16 08:39:23 CST 2017
A结束了！时间为:Tue May 16 08:39:28 CST 2017
```
任务被延时但还是一个一个顺序执行

**4. TimerTask类的cancel()方法**

TimerTask类中的cancel()方法的作用是将自身从任务队列中清除
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    static public class MyTaskA extends TimerTask {
        @Override
        public void run() {
            System.out.println("A运行了！时间为:" + new Date());
            this.cancel();
        }
    }

    static public class MyTaskB extends TimerTask {
        @Override
        public void run() {
            System.out.println("B运行了！时间为:" + new Date());
        }
    }

    public static void main(String[] args) {
        try {
            MyTaskA taskA = new MyTaskA();
            MyTaskB taskB = new MyTaskB();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString = "2017-5-18 8:09:00";
            Timer timer = new Timer();
            Date dateRef = sdf.parse(dateString);
            System.out.println("字符串时间:" + dateRef.toString() + "当前时间:" + new Date().toString());
            timer.schedule(taskA, dateRef, 4000);
            timer.schedule(taskB, dateRef, 4000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果:
```java
字符串时间:Thu May 18 08:09:00 CST 2017当前时间:Thu May 18 08:47:24 CST 2017
A运行了！时间为:Thu May 18 08:47:24 CST 2017
B运行了！时间为:Thu May 18 08:47:24 CST 2017
B运行了！时间为:Thu May 18 08:47:28 CST 2017
```
TimerTask类的cancel()方法是将自身从任务队列中被移除，其他任务不受影响。

**5. Timer类的cancel()方法**
和TimerTask类中的cancel()方法清除自身不同，Timer类中的cancel()方法的作用是将任务队列中的全部任务清空。
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    private static Timer timer=new Timer();
    static public class MyTaskA extends TimerTask {
        @Override
        public void run() {
            System.out.println("A运行了！时间为:" + new Date());
            timer.cancel();
        }
    }

    static public class MyTaskB extends TimerTask {
        @Override
        public void run() {
            System.out.println("B运行了！时间为:" + new Date());
        }
    }

    public static void main(String[] args) {
        try {
            MyTaskA taskA = new MyTaskA();
            MyTaskB taskB = new MyTaskB();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateString = "2017-5-18 8:09:00";
            Date dateRef = sdf.parse(dateString);
            System.out.println("字符串时间:" + dateRef.toString() + "当前时间:" + new Date().toString());
            timer.schedule(taskA, dateRef, 4000);
            timer.schedule(taskB, dateRef, 4000);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为:
```java
字符串时间:Thu May 18 08:09:00 CST 2017当前时间:Thu May 18 08:56:39 CST 2017
A运行了！时间为:Thu May 18 08:56:39 CST 2017
```
全部任务都被清除，并且进程被销毁，按钮由红色变成灰色。

**6. Timer的cancel()方法注意事项**
Timer类中的cancel()方法有时并一定会停止执行计划任务，而是正常执行。
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    static int i=0;

    static public class MyTask extends TimerTask {
        @Override
        public void run() {
            System.out.println("正常执行了" +i);
        }
    }

    public static void main(String[] args) {
        while (true) {
            try {
                i++;
                Timer timer = new Timer();
                MyTask task = new MyTask();
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                String dateString = "2017-5-18 8:09:00";
                Date dateRef = sdf.parse(dateString);
                timer.schedule(task, dateRef);
                timer.cancel();
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    }
}
```
程序运行后的部分结果为:
```java
正常执行了2
正常执行了163
正常执行了347
正常执行了542
正常执行了622
正常执行了739
正常执行了751
正常执行了852
正常执行了1233
```
这是因为Timer类中的cancel()方法有时并没有争抢到queue锁，所以TimerTask类中的任务继续正常执行。

> 摘选自 java多线程核心编程技术-5.1.2