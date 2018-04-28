---
title: 2.2.2 synchronized同步代码块的使用
date: 2017-07-18 08:08:17
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

当两个并发线程访问同一个对象object中的synchronized(this)同步代码块时，一段时间内只能有一个线程被执行，另一个线程必须等待当前线程执行完这个代码块以后以后才能执行该代码块。
```java
/**
 * synchronized同步代码的使用
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class ObjectService {
    public void serviceMethod(){
        try {
            synchronized (this){
                System.out.println("begin Time="+System.currentTimeMillis());
                Thread.sleep(2000);
                System.out.println("end end="+System.currentTimeMillis());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

/**ThreadA线程类
 * synchronized同步代码的使用
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class ThreadA extends Thread{
    private ObjectService service;

    public ThreadA(ObjectService service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.serviceMethod();
    }
}

/**
 * ThreadA线程类
 * synchronized同步代码的使用
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class ThreadB extends Thread{
    private ObjectService service;

    public ThreadB(ObjectService service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.serviceMethod();
    }
}

/**
 * 运行实例
 * synchronized同步代码的使用
 * @author wuyoushan
 * @date 2017/1/23.
 */
public class Run {
    public static void main(String[] args) {
        ObjectService service=new ObjectService();
        ThreadA a=new ThreadA(service);
        a.setName("a");
        a.start();
        ThreadB b=new ThreadB(service);
        b.setName("b");
        b.start();
    }
}
```
程序的运行结果为：
```java
begin Time=1492042714510
end end=1492042716511
begin Time=1492042716511
end end=1492042718511
```
上面的实验虽然使用了synchronized同步代码块，但执行的效率还是没有提高，执行的效果还是同步运行的。
如何用synchronized同步代码解决程序执行效率低的问题呢？

> 摘选自 java多线程核心编程技术-2.2.2