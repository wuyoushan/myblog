---
title: 线程总结
date: 2017-08-08 18:26:47
tags:
---
## 线程创建的两种方式:
1. 继承Thread类:

```java
class MyThread extends Thread{
    @Override
    public void run() {
        //// TODO
    }
}

MyThread mt=new MyThread();
mt.start();
```

2. 实现Runnable接口:
```java
class MyThread implements Runnable {
    @Override
    public void run() {
        //// TODO
    }
}

MyThread mt=new MyThread();
mt.start();
```
## 线程创建的两种方式比较
1. Runnable方式可以避免Thread方式由于java单继承特性带来的缺陷

2. Runnable的代码可以被多个线程(Thread实例)共享,适合于多个线程处理同一资源的情况

## Thread模拟买票

### 1. 继承Thread类
```java
//定义一个线程类
class MyThread extends Thread{
	private int ticketsCount = 5;  //一共有5张火车票
	private String name;   //窗口，就是线程的名字
	
	public MyThread(String name){
		this.name = name;
	}
	
	//创建启动线程的方法
	@Override
	public void run() {
		while(ticketsCount > 0){
			ticketsCount--;  //如果还有票，就卖掉一张
			System.out.println(name +"卖了1张票，剩余票数为"+ticketsCount);
		}
	}
}

public class TicketsThread {

	public static void main(String[] args) {
		//创建3个线程，模拟三个窗口卖票
		MyThread mt1 = new MyThread("窗口1");
		MyThread mt2 = new MyThread("窗口2");
		MyThread mt3 = new MyThread("窗口3");
		
		//启动这三个线程，就是窗口开始卖票
		mt1.start();
		mt2.start();
		mt3.start();

	}

}
```
程序的输出结果:
```java
/*
* 总共只有5张火车票，每个窗口卖了5次，结果卖出15张票，这是不合理的
*/

窗口1卖了1张票，剩余票数为4
窗口1卖了1张票，剩余票数为3
窗口1卖了1张票，剩余票数为2
窗口1卖了1张票，剩余票数为1
窗口1卖了1张票，剩余票数为0
窗口2卖了1张票，剩余票数为4
窗口2卖了1张票，剩余票数为3
窗口2卖了1张票，剩余票数为2
窗口2卖了1张票，剩余票数为1
窗口2卖了1张票，剩余票数为0
窗口3卖了1张票，剩余票数为4
窗口3卖了1张票，剩余票数为3
窗口3卖了1张票，剩余票数为2
窗口3卖了1张票，剩余票数为1
窗口3卖了1张票，剩余票数为0
```
### 2. 实现Runnable 接口
```java
//创建一个线程类
class MyThread1 implements Runnable{
	private int ticketsCount = 5;  //一共有5张火车票
	
	@Override
	public void run() {
		//如果ticketsCount不为0，说明还有火车票，可以继续卖
		while(ticketsCount > 0 ){
			ticketsCount--;
			//Thread.currentThread().getName()   获得当前线程的名字
			System.out.println(Thread.currentThread().getName() +"卖了1张票，剩余票数为"+ticketsCount);
		}
		
	}
}

public class TicketsThread1 {

	public static void main(String[] args) {
		MyThread1 mt = new MyThread1();
		//创建三个线程,来模拟三个售票窗口
		Thread th1 = new Thread(mt,"窗口1");
		Thread th2 = new Thread(mt,"窗口2");
		Thread th3 = new Thread(mt,"窗口3");
		//启动三个线程，就是三个窗口开始卖票
		th1.start();
		th2.start();
		th3.start();

	}

}
```
程序的运行结果
```java
窗口1卖了1张票，剩余票数为4
窗口1卖了1张票，剩余票数为3
窗口1卖了1张票，剩余票数为2
窗口1卖了1张票，剩余票数为1
窗口1卖了1张票，剩余票数为0
```

