---
title: 1.7.3能停止的线程——异常法
date: 2017-07-13 22:17:53
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
        super.run();
        for (int i=0;i<500000;i++){
            if(this.interrupted()){
                System.out.println("已经是停止状态了！我要退出了！");
                break;
            }
            System.out.println("i="+(i+1));
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        try {
            MyThread myThread=new MyThread();
            myThread.start();
            Thread.sleep(2000);
            myThread.interrupt();
        } catch (InterruptedException e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end");
    }
}
```
#### 程序运行结果：
```java
i=274421
i=274422
i=274423
i=274424
i=274425
i=274426
已经是停止状态了！我要退出了！
end
```
上面的示例虽然停止了线程，但如果for语句下面还有语句，还是会继续运行的。示例如下：
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
        super.run();
        for (int i=0;i<500000;i++){
            if(this.interrupted()){
                System.out.println("已经是停止状态了！我要退出了！");
                break;
            }
            System.out.println("i="+(i+1));
        }
        System.out.println("我被输出，如果此代码是for又继续运行，线程并未停止！");
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        try {
            MyThread myThread=new MyThread();
            myThread.start();
            Thread.sleep(2000);
            myThread.interrupt();
        } catch (InterruptedException e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end");
    }
}
```
#### 运行结果如下:
```java
i=308968
i=308969
i=308970
i=308971
i=308972
i=308973
已经是停止状态了！我要退出了！
我被输出，如果此代码是for又继续运行，线程并未停止！
end
```
那该如何解决语句继续运行的问题呢？看一下更新后的代码。
```java
/**
 * MyThread线程测试
 * @author wuyoushan
 * @date 2017/3/21.
 */
public class MyThread extends Thread {

    @Override
    public void run() {
        super.run();
        try {
            for (int i = 0; i < 500000; i++) {
                if (this.interrupted()) {
                    System.out.println("已经是停止状态了！我要退出了！");
                    //抛出线程中断异常
                    throw new InterruptedException();
                }
                System.out.println("i=" + (i + 1));
            }
            System.out.println("我在for下面");
        }catch(InterruptedException e){
            System.out.println("进入MyThread.java类run方法中的catch了！");
            e.printStackTrace();
        }
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args) {
        try {
            MyThread myThread=new MyThread();
            myThread.start();
            Thread.sleep(2000);
            myThread.interrupt();
        } catch (InterruptedException e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end");
    }
}

```
运行结果如下:
```java
i=162100
i=162101
i=162102
i=162103
i=162104
i=162105
i=162106
已经是停止状态了！我要退出了！
进入MyThread.java类run方法中的catch了！
end
java.lang.InterruptedException
	at wys.test.MyThread.run(MyThread.java:18)

```

> 摘选自 java多线程核心编程技术-1.7.3