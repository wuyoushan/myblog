---
title: 2.3.6原子类也并不安全
date: 2017-07-23 14:49:16
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
原子类在具有有逻辑性的情况下输出结果也具有随机性
```java
/**
 * @author wuyoushan
 * @date 2017/4/19.
 */
public class MyService {
    public static AtomicLong atomicLong=new AtomicLong();

    public void addNum(){
        System.out.println(Thread.currentThread().getName()+"加了100之后的值是:"
        +atomicLong.addAndGet(100));
        atomicLong.addAndGet(1);
    }
}

/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private MyService myService;

    public MyThread(MyService myService) {
        super();
        this.myService = myService;
    }

    @Override
    public void run() {
        super.run();
        myService.addNum();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
      try{
            MyService service=new MyService();
            MyThread[] array=new MyThread[5];
          for (int i = 0; i < array.length; i++) {
              array[i]=new MyThread(service);
          }

          for (int i = 0; i < array.length; i++) {
              array[i].start();
          }
          Thread.sleep(1000);
          System.out.println(service.atomicLong.get());
      }catch (InterruptedException e){
            e.printStackTrace();
      }
    }
}
```
程序的运行结果为:
```java
Thread-2加了100之后的值是:100
Thread-3加了100之后的值是:201
Thread-4加了100之后的值是:502
Thread-0加了100之后的值是:401
Thread-1加了100之后的值是:301
505
```
打印顺序出错了，应该是每加1次100再加1次1.出现这样的情况是因为addAndGet()方法是原子的，但方法和方法之间的调用却不是原子的。解决这样的问题必须要同步。

更改MyService.java文件代码如下:
```java
/**
 * @author wuyoushan
 * @date 2017/4/19.
 */
public class MyService {
    public static AtomicLong atomicLong=new AtomicLong();

    synchronized public void addNum(){
        System.out.println(Thread.currentThread().getName()+"加了100之后的值是:"
        +atomicLong.addAndGet(100));
        atomicLong.addAndGet(1);
    }
}
```
程序的运行结果为：
```java
Thread-0加了100之后的值是:100
Thread-1加了100之后的值是:201
Thread-2加了100之后的值是:302
Thread-3加了100之后的值是:403
Thread-4加了100之后的值是:504
505
```
从运行的结果可以看到，是每次加100再加1，这就是我们想要得到的过程，结果505的同时还保证在过程中的累加的顺序也是正确的。

> 摘选自 java多线程核心编程技术-2.3.6