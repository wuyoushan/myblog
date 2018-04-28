---
title: 2.1.8同步不具体有继承性
date: 2017-07-15 11:06:59
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---

同步不可以继承
```java
/**
 * @author wuyoushan
 * @date 2017/4/11.
 */
public class Main {

    synchronized public void serviceMethod(){
        try {
            System.out.println("int main 下一步sleep begin threadName="+
            Thread.currentThread().getName()+" time="+
            System.currentTimeMillis());
            Thread.sleep(5000);
            System.out.println("int main 下一步sleep end threadName="+
                    Thread.currentThread().getName()+" time="+
                    System.currentTimeMillis());
        }catch(InterruptedException e){
            e.printStackTrace();
        }

    }
}

/**
 * @author wuyoushan
 * @date 2017/4/11.
 */
public class Sub extends Main {

    @Override
    public void serviceMethod() {
        try {
            System.out.println("int sub 下一步sleep begin threadName="+
                    Thread.currentThread().getName()+" time="+
                    System.currentTimeMillis());
            Thread.sleep(5000);
            System.out.println("int sub 下一步sleep end threadName="+
                    Thread.currentThread().getName()+" time="+
                    System.currentTimeMillis());
            super.serviceMethod();
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

    private Sub sub;

    public ThreadA(Sub sub){
        super();
        this.sub=sub;
    }

    @Override
    public void run() {
        super.run();
        sub.serviceMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/4/4.
 */
public class ThreadB extends Thread{

    private Sub sub;

    public ThreadB(Sub sub){
        super();
        this.sub=sub;
    }

    @Override
    public void run() {
        super.run();
        sub.serviceMethod();
    }
}

/**
 * @author wuyoushan
 * @date 2017/3/14.
 */
public class Test {
    public static void main(String[] args) {
       Sub subRef=new Sub();
        ThreadA a=new ThreadA(subRef);
        a.setName("A");
        a.start();
        ThreadB b=new ThreadB(subRef);
        b.setName("B");
        b.start();
    }
}

```
程序的结果:
```java
int sub 下一步sleep begin threadName=A time=1491871403168
int sub 下一步sleep begin threadName=B time=1491871403169
int sub 下一步sleep end threadName=A time=1491871408169
int main 下一步sleep begin threadName=A time=1491871408169
int sub 下一步sleep end threadName=B time=1491871408170
int main 下一步sleep end threadName=A time=1491871413169
int main 下一步sleep begin threadName=B time=1491871413169
int main 下一步sleep end threadName=B time=1491871418169
```
从上面第一行，第二行可以看到:运行结果是不同步的。由此可知，同步是不能继承，所以还得在子类的方法中添加synchronized关键字，添加以后的运行效果。
```java
int sub 下一步sleep begin threadName=A time=1491872987559
int sub 下一步sleep end threadName=A time=1491872992559
int main 下一步sleep begin threadName=A time=1491872992559
int main 下一步sleep end threadName=A time=1491872997560
int sub 下一步sleep begin threadName=B time=1491872997560
int sub 下一步sleep end threadName=B time=1491873002560
int main 下一步sleep begin threadName=B time=1491873002560
int main 下一步sleep end threadName=B time=1491873007560
```

> 摘选自 java多线程核心编程技术-2.1.8