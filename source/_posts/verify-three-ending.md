---
title: 2.2.8细化验证3个结论
date: 2017-07-23 13:49:04
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
“synchronized(非this对象x)”格式的写法是将x对象本身作为“对象监视器”，这样就可以得出以下3个结论：
1. 当多个线程同时执行synchronized(x){}同步代码块时呈同步效果
2. 当其他线程执行x对象中synchronized同步方法时呈同步效果
3. 当其他线程执行x对象方法里面的synchronized(this)代码块时也呈现同步效果
4. 但需要注意：如果其他线程调用不加synchronized关键字的方法时，还是异步调用。
 

当多个线程同时执行synchronized(x){}同步代码块时呈同步效果
```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {

}

/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    public void testMethod1(MyObject object){
        synchronized (object){
            try{
                System.out.println("testMethod1  getLock time="
                +System.currentTimeMillis()+"run ThreadName="
                +Thread.currentThread().getName());
                Thread.sleep(2000);
                System.out.println("testMethod1  releaseLock time="
                        +System.currentTimeMillis()+"run ThreadName="
                        +Thread.currentThread().getName());

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
    private MyObject object;

    public ThreadA(Service service, MyObject object) {
        super();
        this.service = service;
        this.object = object;
    }

    @Override
    public void run() {
       super.run();
        service.testMethod1(object);
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Service service;
    private MyObject object;

    public ThreadB(Service service, MyObject object) {
        super();
        this.service = service;
        this.object = object;
    }

    @Override
    public void run() {
        super.run();
        service.testMethod1(object);
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
      Service service=new Service();
      MyObject object=new MyObject();
      ThreadA a=new ThreadA(service,object);
        a.setName("a");
        a.start();
        ThreadB b=new ThreadB(service,object);
        b.setName("b");
        b.start();
    }
}

```
程序的运行结果是：
```java
testMethod1  getLock time=1492734868657run ThreadName=a
testMethod1  releaseLock time=1492734870658run ThreadName=a
testMethod1  getLock time=1492734870658run ThreadName=b
testMethod1  releaseLock time=1492734872658run ThreadName=b
```
同步的原因是使用了同一个“对象监视器”。如果使用不同的“对象监视器”会出现什么样的效果呢？

修改类文件Run.java，代码如下：
```java
/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
      Service service=new Service();
      MyObject object1=new MyObject();
      MyObject object2=new MyObject();
      ThreadA a=new ThreadA(service,object1);
      a.setName("a");
      a.start();
      ThreadB b=new ThreadB(service,object2);
      b.setName("b");
      b.start();
    }
}

```
运行结果为:
```java
testMethod1  getLock time=1492735877980run ThreadName=a
testMethod1  getLock time=1492735877981run ThreadName=b
testMethod1  releaseLock time=1492735879982run ThreadName=b
testMethod1  releaseLock time=1492735879982run ThreadName=a
```


**当其他线程执行x对象中synchronized同步方法时呈同步效果**
```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {

    synchronized public void speedPrintString(){
        System.out.println("speedPrintString  getLock time="
                +System.currentTimeMillis()+"run ThreadName="
                +Thread.currentThread().getName());
        System.out.println("--------------------");
        System.out.println("speedPrintString  releaseLock time="
                +System.currentTimeMillis()+"run ThreadName="
                +Thread.currentThread().getName());

    }
}

/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    public void testMethod1(MyObject object){
        synchronized (object){
            try{
                System.out.println("testMethod1  getLock time="
                +System.currentTimeMillis()+"run ThreadName="
                +Thread.currentThread().getName());
                Thread.sleep(5000);
                System.out.println("testMethod1  releaseLock time="
                        +System.currentTimeMillis()+"run ThreadName="
                        +Thread.currentThread().getName());

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
    private MyObject object;

    public ThreadA(Service service, MyObject object) {
        super();
        this.service = service;
        this.object = object;
    }

    @Override
    public void run() {
       super.run();
        service.testMethod1(object);
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private MyObject object;

    public ThreadB(MyObject object) {
        super();
        this.object = object;
    }

    @Override
    public void run() {
        super.run();
        object.speedPrintString();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
      Service service=new Service();
      MyObject object=new MyObject();
      ThreadA a=new ThreadA(service,object);
      a.setName("a");
      a.start();
      Thread.sleep(100);
      ThreadB b=new ThreadB(object);
      b.setName("b");
      b.start();
    }
}

```
程序的运行结果为:
```java
testMethod1  getLock time=1492736535766run ThreadName=a
testMethod1  releaseLock time=1492736540766run ThreadName=a
speedPrintString  getLock time=1492736540766run ThreadName=b
--------------------
speedPrintString  releaseLock time=1492736540766run ThreadName=b
```

**验证第3个结论**
当其他线程执行x对象方法里面的synchronized(this)代码块时也呈现同步效果。
修改MyObject.java的类，代码如下。
```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {

    synchronized public void speedPrintString(){
        synchronized (this) {
            System.out.println("speedPrintString  getLock time="
                    + System.currentTimeMillis() + "run ThreadName="
                    + Thread.currentThread().getName());
            System.out.println("--------------------");
            System.out.println("speedPrintString  releaseLock time="
                    + System.currentTimeMillis() + "run ThreadName="
                    + Thread.currentThread().getName());
        }
    }
}
```
代码的运行结果如下:
```java
testMethod1  getLock time=1492916942775run ThreadName=a
testMethod1  releaseLock time=1492916947775run ThreadName=a
speedPrintString  getLock time=1492916947775run ThreadName=b
--------------------
speedPrintString  releaseLock time=1492916947775run ThreadName=b
```

> 摘选自 java多线程核心编程技术-2.2.8