---
title: 2.2.7将任意对象作为对象监视器
date: 2017-07-23 13:46:21
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
多个线程调用同一个对象中的不同名称的synchronized同步方法或synchronized(this)同步代码块时，调用的效果就是按顺序执行，也就是同步的，阻塞的。
这说明synchronized同步方法或synchronized(this)同步代码块分别有两种作用。
1. synchronized同步方法
 synchronized同步方法或synchronized(this)同步代码块调用呈阻塞状态。
同一时间只有一个线程可以执行synchronized同步方法中的代码

2. synchronized(this)同步代码块
对其他synchronized同步方法或synchronized(this)同步代码块调用呈阻塞状态
同一时间只有一个线程可以执行synchronized(this)同步代码块中的代码

在前面的学习中，使用synchronized(this)格式同步代码块，其实java还支持对“任意对象”作为“对象监听器”来实现同步的功能。这个“任意对象”大多数是实例变量及方法的参数，使用格式为synchronized(非this对象)


根据前面对synchronized(this)同步代码块的作用总结可知，synchronized(非this对象)格式的作用只有1种：synchronized(非this对象x)同步代码块中的代码

当持有“对象监视器”为同一个对象的前提下，同一时间只有一个线程可以执行synchronized(非this对象x)同步代码块中的代码。

```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    private String usernameParam;
    private String passwordParam;
    private String anyString=new String();

    public void setUsernamePassword(String username,String password) {
        try{
            synchronized (anyString){
                System.out.println("线程名称为:"+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"进入同步块");
                usernameParam=username;
                Thread.sleep(3000);
                passwordParam=password;
                System.out.println("线程名称为:"+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"离开同步块");
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service){
        super();
        this.service=service;
    }

    @Override
    public void run() {
       service.setUsernamePassword("a","aa");
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Service service;

    public ThreadB(Service service){
        super();
        this.service=service;
    }

    @Override
    public void run() {
        service.setUsernamePassword("b","bb");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        Service service=new Service();
        ThreadA a=new ThreadA(service);
        a.setName("A");
        a.start();
        ThreadB b=new ThreadB(service);
        b.setName("B");
        b.start();
    }
}
```
程序的运行结果为:
```java
线程名称为:A在1492391031020进入同步块
线程名称为:A在1492391034020离开同步块
username:a password:aa
线程名称为:B在1492391034020进入同步块
线程名称为:B在1492391037020离开同步块
username:b password:bb
```
锁非this对象具有一定的优点：如果在一个类中有很多个synchronized方法，这是虽然能实现同步，但会受到阻塞，所以影响运行效率；但如果使用同步代码块锁非this对象，则synchronized(非this)代码块中的程序与同步方法是异步的，不与其他锁this同步方法争抢this锁，则可大大提升运行效率。

将Service.java文件代码更改如下:
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {

    private String usernameParam;
    private String passwordParam;

    public void setUsernamePassword(String username,String password) {
        try{
            String anyString=new String();
            synchronized (anyString){
                System.out.println("线程名称为:"+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"进入同步块");
                usernameParam=username;
                Thread.sleep(3000);
                passwordParam=password;
                System.out.println("线程名称为:"+Thread.currentThread().getName()
                +"在"+System.currentTimeMillis()+"离开同步块");
                 System.out.println("username:"+usernameParam+" password:"+passwordParam);
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
程序的运行结果如下所示:
```java
线程名称为:A在1492474468701进入同步块
线程名称为:B在1492474468702进入同步块
线程名称为:A在1492474471702离开同步块
username:b password:aa
线程名称为:B在1492474471702离开同步块
username:b password:bb
```
可见，使用“synchronized(非this对象x)同步代码块”格式进行同步操作时，对象监视器必须是同一个对象。如果不是同一个对象监视器，运行的结果就是异步调用了，就会交叉运行。

下面我们再用另外一个项目来验证一下使用“synchronized(非this对象x)同步代码块”格式时，持有不同的对象监听器时异步的效果。

```java

/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class Service {
    private String anyString=new String();
    public void a() {
        try{
            synchronized (anyString){
                System.out.println("a begin");
                Thread.sleep(3000);
                System.out.println("a end");
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    synchronized public void b() {
        System.out.println("b begin");
        System.out.println("b end");

    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private Service service;

    public ThreadA(Service service){
        super();
        this.service=service;
    }

    @Override
    public void run() {
        service.a();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Service service;

    public ThreadB(Service service){
        super();
        this.service=service;
    }

    @Override
    public void run() {
        service.b();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        Service service=new Service();
        ThreadA a=new ThreadA(service);
        a.setName("A");
        a.start();
        ThreadB b=new ThreadB(service);
        b.setName("B");
        b.start();
    }
}
```
程序的运行结果为:
```java
a begin
b begin
b end
a end
```
由于对象监视器的不同，所以运行结果就是异步的。
同步代码块放在非同步synchronized方法中进行声明，并不能保证调用方法的线程执行同步/顺序性，也就是线程调用方法的顺序是无序的，虽然在同步代码块中执行的顺序是同步的，这样极易出现“脏读”问题。

使用“synchronized(非this对象x)同步代码块”格式也可以解决“脏读”问题。但在解决脏读问题之前，先做一个实验，实验的目标是验证多个线程调用同一个方法是随机的。
```java
/**
 * @author wuyoushan
 * @date 2017/2/5.
 */
public class MyList{
    private List list=new ArrayList();
    synchronized public void add(String username){
        System.out.println("ThreadName="+Thread.currentThread().getName()
                +"执行了add方法");
        list.add(username);
        System.out.println("ThreadName="+Thread.currentThread().getName()
                +"退出了add方法");
    }
    synchronized public int getSize(){
        System.out.println("ThreadName="+Thread.currentThread().getName()
                +"执行了getSize方法");
        int sizeValue=list.size();
        System.out.println("ThreadName="+Thread.currentThread().getName()
                +"退出了getSize方法");
        return sizeValue;
    }

}

/**
 * @author wuyoushan
 * @date 2017/2/5.
 */
public class ThreadA extends Thread{
    private MyList list;

    public ThreadA( MyList list) {
        super();
        this.list = list;
    }

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            list.add("threadA"+(i+1));
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/2/5.
 */
public class ThreadB extends Thread {
    private MyList list;

    public ThreadB( MyList list) {
        super();
        this.list = list;
    }

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            list.add("threadA"+(i+1));
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/2/5.
 */
public class Run {
    public static void main(String[] args) {
        MyList myList=new MyList();
        ThreadA a=new ThreadA(myList);
        a.setName("A");
        a.start();
        ThreadB b=new ThreadB(myList);
        b.setName("B");
        b.start();
    }
}
```
程序运行后的结果如下:
```java
ThreadName=B执行了add方法
ThreadName=B退出了add方法
ThreadName=A执行了add方法
ThreadName=A退出了add方法
ThreadName=B执行了add方法
ThreadName=B退出了add方法
```
从运行的结果来看，同步代码块中的代码是同步打印的，当前线程的“执行”与“退出”是成对出现的。但线程A和线程B的执行却是异步的，这就有可能出现脏读的环境。由于线程执行方法的顺序不确定，所以当A和B两个线程执行带有分支判断的方法时，就会出现逻辑上的错误，有可能出现脏读。

```java
/**
 * @author wuyoushan
 * @date 2017/4/19.
 */
public class MyOneList {
    private List<String> list=new ArrayList<>();

    synchronized public void add(String data){
        list.add(data);
    }

    synchronized public int getSzie(){
        return list.size();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/19.
 */
public class MyService {
    public MyOneList addServiceMethod(MyOneList list,String data){
        try{
            if(list.getSzie()<1){
                Thread.sleep(2000);     //模拟从远程花费2秒取回数据
                list.add(data);
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        return list;
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{
    private MyOneList list;

    public ThreadA(MyOneList list){
        super();
        this.list=list;
    }

    @Override
    public void run() {
        MyService service=new MyService();
        service.addServiceMethod(list,"A");
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private MyOneList list;

    public ThreadB(MyOneList list){
        super();
        this.list=list;
    }

    @Override
    public void run() {
        MyService service=new MyService();
        service.addServiceMethod(list,"B");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
        MyOneList list=new MyOneList();

        ThreadA a=new ThreadA(list);
        a.setName("A");
        a.start();
        ThreadB b=new ThreadB(list);
        b.setName("B");
        b.start();
        Thread.sleep(6000);
        System.out.println("listSize="+list.getSzie());
    }
}

```
程序运行后的结果为:
```java
listSize=2
```
脏读出现了。出现的原因是两个线程以异步的方式返回list参数的size()大小。解决办法就是“同步化”
更改MyService.java类文件代码：
```java
/**
 * @author wuyoushan
 * @date 2017/4/19.
 */
public class MyService {
    public MyOneList addServiceMethod(MyOneList list,String data){
        try{
            synchronized (list) {
                if (list.getSzie() < 1) {
                    Thread.sleep(2000);     //模拟从远程花费2秒取回数据
                    list.add(data);
                }
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        return list;
    }
}

```
由于list参数对象在项目中是一份实例，是单例的，而且也正需要对list参数的getSize()方法做同步的调用，所以就对list参数进行同步处理。
程序的运行结果为:
```java
listSize=1
```

> 摘选自 java多线程核心编程技术-2.2.7