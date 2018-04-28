---
title: 1.8.3suspend与resume方法的缺点——不同步
date: 2017-07-13 22:35:47
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
在使用suspend与resume方法时也容易出现因为线程的暂停而导致数据不同步的情况。
```java
/**
 * @author wuyoushan
 * @date 2017/3/29.
 */
public class MyObject {
    private String userName="1";
    private String password="11";

    public void setValue(String u,String p){
        this.userName=u;
        if(Thread.currentThread().getName().equals("a")){
            System.out.println("停止a线程!");
            Thread.currentThread().suspend();
        }
        this.password=p;
    }

    public void printUserNamePassword(){
        System.out.println(userName+" "+password);
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) throws InterruptedException {
        final MyObject myObject=new MyObject();
        Thread thread1=new Thread(){
            @Override
            public void run() {
                super.run();
                myObject.setValue("a","aa");
            }
        };
        thread1.setName("a");
        thread1.start();
        Thread.sleep(500);
        Thread thread2=new Thread(){
            @Override
            public void run() {
                super.run();
                myObject.printUserNamePassword();
            }
        };
        thread2.start();
    }
}
```
运行结果如下:
```java
停止a线程!
a 11
```
> 摘选自 java多线程核心编程技术-1.8.3