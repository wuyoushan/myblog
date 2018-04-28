---
title: 2.1.5脏读
date: 2017-07-15 10:53:30
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

实现多个线程调用同一个方法时，为避免数据出现交叉的情况，使用synchronized关键字进行同步

虽然在赋值时进行了同步，但在取值时有可能出现一些意想不到的意外，这种情况就是脏读。发生脏读的情况是在读取实例变量时，此值已经被其他线程更改过了。

```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class PublicVar {
    public String username="A";
    public String password="AA";

    synchronized public void setValue(String username,String password){
        try{
            this.username=username;
            Thread.sleep(5000);
            this.password=password;
            System.out.println("setValue method thread name="+
            Thread.currentThread().getName()+" username="+
            username+" password="+password);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    public void getValue(){
        System.out.println("getValue method thread name="+
                Thread.currentThread().getName()+" username="+
                username+" password="+password);
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadA extends Thread{

    private PublicVar publicVar;

    public ThreadA(PublicVar publicVar){
        super();
        this.publicVar=publicVar;
    }

    @Override
    public void run() {
        super.run();
        publicVar.setValue("B","BB");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class Test {
    public static void main(String[] args) {
       try{
           PublicVar publicVar=new PublicVar();
           ThreadA thread=new ThreadA(publicVar);
           thread.start();
            Thread.sleep(200);
           publicVar.getValue();
       }catch (InterruptedException e){
           e.printStackTrace();
       }
    }
}

```
程序的运行结果为：
```java
getValue method thread name=main username=B password=AA
setValue method thread name=Thread-0 username=B password=BB

```
出现脏读是因为public void getValue()方法并不是同步的，所以可以在任意时候进行调用。解决方法当然就是加上同步synchronized关键字，代码如下:
```java
/**
 * @author wuyoushan
 * @date 2017/4/10.
 */
public class PublicVar {
    public String username="A";
    public String password="AA";

    synchronized public void setValue(String username,String password){
        try{
            this.username=username;
            Thread.sleep(5000);
            this.password=password;
            System.out.println("setValue method thread name="+
            Thread.currentThread().getName()+" username="+
            username+" password="+password);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    synchronized public void getValue(){
        System.out.println("getValue method thread name="+
                Thread.currentThread().getName()+" username="+
                username+" password="+password);
    }
}

```
程序的运行结果为:
```java
setValue method thread name=Thread-0 username=B password=BB
getValue method thread name=main username=B password=BB
```
可见，方法setValue()和getValue()被依次执行。通过这个案例不仅要知道脏读是通过synchronized关键字解决的，还要知道如下内容:

**当A线程调用anyObject对象加入synchronized关键字的X方法时，A线程就获得了X方法的锁，更准确的讲，是获取了对象的锁，所以其他线程必须等A线程执行完毕才可以调用X方法，但B线程可以随意调用其他的非synchronized同步方法**

**当A线程调用anyObject对象加入synchronized关键字的X方法时，A线程就获得了X方法的所在对象的锁，所以其他线程必须等A线程执行完毕才可以调用X方法，而B线程如果调用声明了synchronized关键字的非X方法时，必须等A线程将X方法执行完，也就是释放对象锁后才可以调用。这时A线程已经执行了一个完整的任务，也就是说username和password这两个实例变量已经同时被赋值，不存在脏读的基本环境**

> 摘选自 java多线程核心编程技术-2.1.5