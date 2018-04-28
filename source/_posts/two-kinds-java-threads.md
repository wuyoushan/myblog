---
title: java线程有两种
date: 2017-08-08 18:26:26
tags: [java,多线程]
categories: [java]
---
## java线程有两类

### 1. 用户线程:运行在前台,执行具体的任务

### 2. 程序的主线程、连接网络的子线程等都是用户线程

**守护线程:** 运行在后台,为其他前台线程服务
1. 特点:
  
  - 一旦所有用户线程都结束运行,守护线程会随JVM一起结束工作
  
2. 应用:
- 数据库连接池中监测线程

- JVM虚拟机启动后的监测线程
- 最常见的守护线程:垃圾回收线程.


**如何设置守护线程**

可以通过调用Thread类的setDaemon(true)方法来设置当前的线程为守护线程

注意事项:
1. setDaemon(true)必须在start()方法之前调用,否则会抛出IllegalThreadStateException异常

2. 在守护线程中产生的新线程也是守护线程
3. 不是所有的任务都可以分配给守护线程来执行,比如读写操作或计算逻辑

```java
class DaemonThread implements Runnable{
    @Override
    public void run() {
        System.out.println("进入守护进程"+Thread.currentThread().getName());
        try {
            writeToFile();
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("退出守护进程"+Thread.currentThread().getName());
    }

    private void writeToFile() throws Exception{
        File fileName=new File("C:"+File.separator+"daemon.txt");
        OutputStream os=new FileOutputStream(fileName,true);
        int count=0;
        while (count<999){
            os.write(("\r\nword"+count).getBytes());
            System.out.println("守护进程"+Thread.currentThread().getName()+"向文件中写入了word"+count++);
            Thread.sleep(1000);
        }
    }
}

public class DaemonThreadDemo {

    public static void main(String[] args) {
        System.out.println("进入主进程"+Thread.currentThread().getName());
        DaemonThread daemonThread=new DaemonThread();
        Thread thread=new Thread(daemonThread);
        thread.setDaemon(true);
        thread.start();

        Scanner sc=new Scanner(System.in);
        sc.next();
        System.out.println("退出退出主进程"+Thread.currentThread().getName());
    }
}
```

向程序输入:dd
```java
进入主进程main
进入守护进程Thread-0
守护进程Thread-0向文件中写入了word0
守护进程Thread-0向文件中写入了word1
守护进程Thread-0向文件中写入了word2
守护进程Thread-0向文件中写入了word3
dd守护进程Thread-0向文件中写入了word4

退出退出主进程main
```

程序的运行结果:

![images](/learning/myblog/public/img/daemon20170730143936.png)
