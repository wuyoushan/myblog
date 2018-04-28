---
title: 1.8暂停线程
date: 2017-07-13 22:30:06
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
暂停线程意味着此线程还可以恢复运行。在java多线程中，还可以使用suspend()方法暂停线程，使用resume()方法恢复线程的执行。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    private long i=0;

    public Long getI() {
        return i;
    }

    public void setI(long i) {
        this.i = i;
    }

    @Override
    public void run() {
       while (true){
          i++;
       }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        try{
            MyThread thread=new MyThread();
            thread.start();
            Thread.sleep(5000);
            //A段
            thread.suspend();
            System.out.println("A="+System.currentTimeMillis()+"i="+thread.getI());
            Thread.sleep(5000);
            System.out.println("A="+System.currentTimeMillis()+"i="+thread.getI());

            //B段
            thread.resume();
            Thread.sleep(5000);

            //C段
            thread.suspend();
            System.out.println("B="+System.currentTimeMillis()+"i="+thread.getI());
            Thread.sleep(5000);
            System.out.println("B="+System.currentTimeMillis()+"i="+thread.getI());

        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

```
从控制台的打印结果来看，线程的确被暂停了，而且还可以恢复成运行状态。
```java
A=1490576230229i=2056518687
A=1490576235229i=2056518687
B=1490576240229i=4141381898
B=1490576245229i=4141381898
```

> 摘选自 java多线程核心编程技术-1.8