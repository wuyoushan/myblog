---
title: 5.1.3方法schedule(TimerTask task,long delay)的测试
date: 2017-07-23 22:44:18
tags: [java,定时器timer,读书笔记]
categories: [java,读书笔记]
---

**5.1.3方法schedule(TimerTask task,long delay)的测试**
该方法的作用是以执行schedule(TimerTask task,long delay)方法当前的时间为参考时间，在此时间基础上延迟指定的毫秒数后执行一次TimerTask任务。
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
        timer.schedule(task, 7000);
    }
}
```
任务task被延迟7秒执行。程序的运行结果为:
```java
当前时间:Fri May 19 08:07:08 CST 2017
运行了！时间为:Fri May 19 08:07:15 CST 2017
```

> 摘选自 java多线程核心编程技术-5.1.3