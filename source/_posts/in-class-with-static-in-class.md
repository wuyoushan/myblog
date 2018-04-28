---
title: 2.2.13内置类与静态内置类
date: 2017-07-23 14:00:16
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
关键字synchronized的知识点还涉及内置类的使用。
```java
/**
 * @author wuyoushan
 * @date 2017/4/27.
 */
public class PublicClass {
    private String username;
    private String password;

    class PrivateClass{

        private String age;
        private String address;

        public String getAge() {
            return age;
        }

        public void setAge(String age) {
            this.age = age;
        }

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
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
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        PublicClass publicClass=new PublicClass();
        publicClass.setUsername("usernameValue");
        publicClass.setPassword("passwordValue");

        System.out.println(publicClass.getUsername()+" "+publicClass.getPassword());
        PublicClass.PrivateClass privateClass=publicClass.new PrivateClass();
        privateClass.setAge("ageValue");
        privateClass.setAddress("addressValue");
        System.out.println(privateClass.getAge()+" "+privateClass.getAddress());
    }
}
```
程序的运行结果为：
```java
usernameValue passwordValue
ageValue addressValue
```
如果PublicClass.java类和Run.java类不在同一个包中，则需要将PrivateClass内置声明成public公开的。

想要实例化内置类必须使用如下代码:
```java
PublicClass.PrivateClass privateClass=publicClass.new PrivateClass();
```

内置类还有一种叫作静态内置类。
```java
**
 * @author wuyoushan
 * @date 2017/4/27.
 */
public class PublicClass {
    private String username;
    private String password;

    static class PrivateClass{

        private String age;
        private String address;

        public String getAge() {
            return age;
        }

        public void setAge(String age) {
            this.age = age;
        }

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
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
 * @author wuyoushan
 * @date 2017/3/20.
 */
public class Run {
    public static void main(String[] args){
        PublicClass publicClass=new PublicClass();
        publicClass.setUsername("usernameValue");
        publicClass.setPassword("passwordValue");

        System.out.println(publicClass.getUsername()+" "+publicClass.getPassword());
        PublicClass.PrivateClass privateClass=new PublicClass.PrivateClass();
        privateClass.setAge("ageValue");
        privateClass.setAddress("addressValue");
        System.out.println(privateClass.getAge()+" "+privateClass.getAddress());
    }
}
```
程序的运行结果为：
```java
usernameValue passwordValue
ageValue addressValue
```
> 摘选自 java多线程核心编程技术-2.2.13