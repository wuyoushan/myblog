---
title: 2.2.16锁对象的改变
date: 2017-07-23 14:11:30
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
在将任何数据类型作为同步锁时，需要注意的是，是否有多个线程同时持有锁对象，如果同时持有相同的锁对象，则这些线程之间就是同步的；如果分别获得锁对象，这些线程之间就是异步的。
```java
/**
 * @author wuyoushan
 * @date 2017/4/19.
 */
public class MyService {
    private String lock="123";

    public void testMethod(){
        try {
            synchronized (lock) {
                System.out.println(Thread.currentThread().getName() + " begin " +
                        System.currentTimeMillis());
                lock = "456";
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + " end " +
                        System.currentTimeMillis());
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private MyService service;

    public ThreadA(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
       super.run();
        service.testMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

   private MyService service;

    public ThreadB(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.testMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
        MyService service=new MyService();
        ThreadA a=new ThreadA(service);
        a.setName("A");
        ThreadB b=new ThreadB(service);
        b.setName("B");
        a.start();
        Thread.sleep(50);
        b.start();
    }
}
```
程序运行后的结果为:
```java
A begin 1493684592096
B begin 1493684592146
A end 1493684594096
B end 1493684594146
```
程序的结果是异步输出的。因为50毫秒过后B取得的锁是“456”

```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
        MyService service=new MyService();
        ThreadA a=new ThreadA(service);
        a.setName("A");
        ThreadB b=new ThreadB(service);
        b.setName("B");
        a.start();
//        Thread.sleep(50);
        b.start();
    }
}
```
去掉Thread.sleep(50)，程序的运行结果如下:
```java
A begin 1493685002103
A end 1493685004104
B begin 1493685004104
B end 1493685006104
```
线程A和B持有的锁都是“123”，虽然将锁改成了“456”，但是结果还是同步的，因为A和B共同争抢的锁是“123”。

还需要提示一下，只要对象不变，即使对象的属性被改变，运行的结果还是同步。
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
   public void serviceMethodA(UserInfo userInfo){
        synchronized (userInfo){
            try {
                System.out.println(Thread.currentThread().getName());
                userInfo.setUserName("abcabcabc");
                Thread.sleep(3000);
                System.out.println("end! time="+System.currentTimeMillis());
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
   }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;
    private UserInfo userInfo;

    public ThreadA(Service service,UserInfo userInfo) {
        super();
        this.service = service;
        this.userInfo=userInfo;
    }

    @Override
    public void run() {
       super.run();
       service.serviceMethodA(userInfo);
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Service service;
    private UserInfo userInfo;

    public ThreadB(Service service,UserInfo userInfo) {
        this.service = service;
        this.userInfo=userInfo;
    }

    @Override
    public void run() {
        super.run();
        service.serviceMethodA(userInfo);
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try {
            Service service = new Service();
            UserInfo userInfo = new UserInfo();
            ThreadA a = new ThreadA(service, userInfo);
            a.setName("a");
            a.start();
            Thread.sleep(50);

            ThreadB b = new ThreadB(service, userInfo);
            b.setName("b");
            b.start();
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
程序的运行结果为：
```java
a
end! time=1493685934399
b
end! time=1493685937399
```
> 摘选自 java多线程核心编程技术-2.2.16