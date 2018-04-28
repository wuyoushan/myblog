---
title: 1.2.3非线程安全
date: 2017-07-09 15:26:21
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
**非线程安全** 非线程安全主要是指多个线程对同一对象中的同一实例变量进行操作时会出现值被更变、值不同步的情况，进而影响程序的执行流程。

```java
/**
 * 本类模拟成一个登录的Servlet组件
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class LoginServlet
{
    private static String usernameRef;
    private static String passwordRef;

    public static void doPost(String username,String password){
        try{
            usernameRef=username;
            if( username.equals("a")){
                Thread.sleep(5000);
            }
            passwordRef=password;
            System.out.println("username="+usernameRef+" password="+password);
        }catch(InterruptedException ex){
           ex.printStackTrace();
        }

    }
}

/**
 * A登录
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class ALogin extends Thread {
    @Override
    public void run() {
        LoginServlet.doPost("a","aa");
    }
}

/**
 * B登录
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class BLogin extends Thread {
    @Override
    public void run() {
        LoginServlet.doPost("b","bb");
    }
}

```
#### 运行结果(非线程安全的)
```java 
username=b password=bb
username=b password=aa
```
#### 在LoginServlet的doPost添加synchronized关键字。
```java
synchronized public static void doPost(String username,String password){
        try{
            usernameRef=username;
            if( username.equals("a")){
                Thread.sleep(5000);
            }
            passwordRef=password;
            System.out.println("username="+usernameRef+" password="+password);
        }catch(InterruptedException ex){
           ex.printStackTrace();
        }

    }
```
#### 运行结果(线程安全的)
```java
username=a password=aa
username=b password=bb
```
> 摘选自 java多线程核心编程技术-1.2.3