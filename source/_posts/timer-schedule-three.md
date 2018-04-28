---
title: 5.1.4方法schedule(TimerTask task,long delay,long period)的测试
date: 2017-07-23 22:45:28
tags: [java,定时器timer,读书笔记]
categories: [java,读书笔记]
---
该方法的作用是以执行schedule(TimerTask task,long delay,long period)方法当前的时间为参考时间，在此时间基础上延迟指定的毫秒数，再以某一间隔时间无限次数地执行某一任务。
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {

    static public class MyTask extends TimerTask {
        @Override
        public void run() {
            System.out.println("运行了！时间为:" +new Date());
        }
    }

    public static void main(String[] args) {
        Timer timer = new Timer();
        MyTask task = new MyTask();
        System.out.println("当前时间:"+new Date().toString());
        timer.schedule(task,3000,5000);
    }
}

```
程序运行后的结果为:
```java
当前时间:Fri May 19 08:16:04 CST 2017
运行了！时间为:Fri May 19 08:16:07 CST 2017
运行了！时间为:Fri May 19 08:16:12 CST 2017
运行了！时间为:Fri May 19 08:16:17 CST 2017
运行了！时间为:Fri May 19 08:16:22 CST 2017
运行了！时间为:Fri May 19 08:16:27 CST 2017
```
凡是使用方法中带有period参数的，都是无限循环执行TimerTask中的任务。

> 摘选自 java多线程核心编程技术-5.1.4