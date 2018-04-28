---
title: springboot构建镜像
date: 2017-08-09 11:43:45
tags: [docker,springboot]
category: [docker]
---
### 基于现成的java镜像，构建发布一个springboot应用的镜像

[一个简单的springboot地址](https://gitee.com/wuyoushan/learning/tree/master/springboot-demo)


#### 系统环境
- java 8
- Centos7.3
- docker 1.12.6

**由于springboot应用中内置了应用服务器(默认为tomcat)，根据个人喜好可以修改为jetty或者undertow等应用服务器。springboot程序的运行只依赖于jdk**



1. 基于现成的java进行构建springboot镜像
2. 首先从国内的docker镜像中pull拉取docker镜像(默认会从https://hub.docker.com 中拉取镜像)。由于国内的网络状态从官网镜像仓库拉会很不稳定。推荐使用国内进行。在这里我使用的是daocloud国内镜像
![image](/learning/myblog/public/img/springboot-search-java.png)

查找到的镜像

![image](/learning/myblog/public/img/springboot-docker-java-2.png)

单击进去查看镜像信息

![image](/learning/myblog/public/img/springboot-docker-java-3.png)

```bash
docker pull daocloud.io/library/java:openjdk-8u40
```
也可以将链接改成
```bash
docker pull daocloud.io/library/java:latest
```
表示拉取最新的镜像

拉取完,运行docker images命令查看，镜像已经拉到本地
```bash
[root@localhost log]# docker images;
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hub.c.163.com/library/tomcat   latest              a2fbbcebd67e        4 weeks ago         333.9 MB
hub.c.163.com/library/jetty    latest              2d50930b2488        12 weeks ago        319.9 MB
daocloud.io/library/java       openjdk-8u40        4aefdb29fd43        2 years ago         815.5 MB
```

创建Dockerfile文件,文件内容如下
```
FROM daocloud.io/library/java:latest
MAINTAINER yourname XXX@qq.com
VOLUME /tmp
ADD ./springboot-demo-1.0-SNAPSHOT.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```
运行docker build构建镜像(注意最后那个点表示的是当前路径)
```
docker build -t springboot-web:latest .
```

运行过程:
```
[root@localhost docker]# docker build -t springboot-web:latest .
Sending build context to Docker daemon 23.89 MB
Step 1 : FROM daocloud.io/library/java
 ---> d23bdf5b1b1b
Step 2 : MAINTAINER yourname XXX@qq.com
 ---> Running in e964c60f0905
 ---> 7c63ec5f8f5f
Removing intermediate container e964c60f0905
Step 3 : VOLUME /tmp
 ---> Running in abe539105b82
 ---> 738eeef06071
Removing intermediate container abe539105b82
Step 4 : ADD ./springboot-demo-1.0-SNAPSHOT.jar app.jar
 ---> dccf66b86b9f
Removing intermediate container 974b60d0e248
Step 5 : ENV JAVA_OPTS ""
 ---> Running in 7eeea5fda6a7
 ---> 591db4ec14b6
Removing intermediate container 7eeea5fda6a7
Step 6 : ENTRYPOINT sh -c java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar
 ---> Running in 9456005ad15e
 ---> eea988cca381
Removing intermediate container 9456005ad15e
Successfully built eea988cca381
[root@localhost docker]# 

```
构建成功查看构建成功的镜像
```
[root@localhost docker]# docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
springboot-web                 latest              dbaafca494ed        11 days ago         667 MB
hub.c.163.com/library/tomcat   latest              a2fbbcebd67e        6 weeks ago         333.9 MB
hub.c.163.com/library/jetty    latest              2d50930b2488        3 months ago        319.9 MB
daocloud.io/library/java       latest              d23bdf5b1b1b        6 months ago        643.1 MB

```
发现有一个springboot-web镜像名，表示我们构建的镜像已经在本地仓库中。

运行构造的springboot-web镜像
```
docker run -it -d -p 8080:8080 springboot-web
```
访问地址: http://localhost:8080/user/list 运行结果为:

![image](/learning/myblog/public/img/springboot-demo.png)