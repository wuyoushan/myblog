---
title: 2.3.4非原子的特性
date: 2017-07-23 14:45:03
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
关键字volatile虽然增加了实例变量在多个线程之间的可见性，但它却不具备同步性，那么也就不具备原子性。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    volatile public static int count;

    private static void addCount(){
        for (int i = 0; i < 100; i++) {
            count++;
        }
        System.out.println("count="+count);
    }

    @Override
    public void run() {
       addCount();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        MyThread[] mythreadArray=new MyThread[100];
        for (int i = 0; i < 100; i++) {
            mythreadArray[i]=new MyThread();
        }
        for (int i = 0; i < 100; i++) {
            mythreadArray[i].start();
        }
    }
}
```
程序的运行结果为：
```java
count=3400
count=3500
count=3600
count=9900
count=9900
count=9900
```
更改自定义线程类MyThread.java文件如下:
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    volatile public static int count;

    //注意一定要添加static关键字
    //这样synchronized与static锁的内容就是MyThread.class类
    //不使用static使用的是内置锁,会出现脏的输出，类似于上面的输出结果
    //也就达到同步的效果了
    synchronized private static void addCount(){
        for (int i = 0; i < 100; i++) {
            count++;
        }
        System.out.println("count="+count);
    }

    @Override
    public void run() {
       addCount();
    }
}

```
程序的运行效果为:
```java
count=9600
count=9700
count=9800
count=9900
count=10000
```
在上面的代码中，如果使用了private static void addCount()前加入synchronized同步关键字，也就没有必要再使用volatile关键字来声明count变量了。

关键字volatile主要使用场合是在多个线程中可以感知实例变量被更改了，并且可以获得最新的值使用，也就是用多线程读取共享变量是可以获取得最新值使用

关键字volatile提示线程每次从共享内存中读取变量，而不是从有的内存中读取，这样就保证了同步数据的可见性。但这里需要注意的是：如果修改实例变量中的数据，比如i++，也就是i=i+1，则这样的操作其实并不是一个原子操作。也就是非线程安全的。表达式i++的操作步骤分解如下：
1. 从内存中取出i的值
2. 计算i的值
3. 将i的值写到内存中

假如在第2步计算值的时候，另外一个线程也修改i的值，那么这个时候就会出现脏数据。解决的办法其实就是使用synchronized关键字，这个知识点在前面的案例中已经介绍过了。所以说volatile本身并不处理数据的原子性，而是强制对数据的读写及时影响到主内存的。

对于用volatile修饰的变量，JVM虚拟机只是保证从主内存加载到线程工作内存的值是最新的。volatile关键字解决的是变量读时的可见性问题，但无法保证原子性，对于多个线程访问同一个实例变量还是需要加锁同步。

> 摘选自 java多线程核心编程技术-2.3.4