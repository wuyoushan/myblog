---
title: 1.7.7释放锁的不良后果
date: 2017-07-13 22:26:50
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

使用stop()释放锁将会给数据造成不一致的结果。如果出现这样的情况，程序处理的数据就有可能遭到破坏，最终导致程序执行的流程出错，一定要特别注意。
```java
/**
 * @author wuyoushan
 * @date 2017/3/27.
 */
public class SynchronizedObject {
    private String username="a";
    private String password="aa";

    public void printString(String username,String password){
        try{
            this.username=username;
            Thread.sleep(100000);
            this.password=password;
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private SynchronizedObject object;

    public MyThread(SynchronizedObject object) {
        this.object = object;
    }

    @Override
    public void run() {
       object.printString("b","bb");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try{
            SynchronizedObject object=new SynchronizedObject();
            MyThread thread=new MyThread(object);
            thread.start();
            Thread.sleep(500);
            thread.stop();
            System.out.println(object.getUsername()+" "+object.getPassword());
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

```
程序的运行结果如下：
```java
b aa

```
由于stop()方法已经在JDK中被标明是“作废/过期”的方法，显然它在功能上具有缺陷，所以不建议在程序中使用stop()方法

> 摘选自 java多线程核心编程技术-1.7.7