---
title: threadFactory的创建
date: 2017-11-05 10:11:28
tags: [java,线程池]
categories: [java]
---

java创建线程有两种方式

1. 使用Executors去创建

2. 使用ThreadPoolExecutor的方式去创建

### 阿里巴巴Java开发文档
#### (六)并发处理中

线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

#### Executors返回的线程池对象的弊端如下
- FixedThreadPool和SingleThreadPool:
允许的请求队列长为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM

- CachedThreadPool和ScheduledThreadPool:
允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM

### 那我们该如何用ThreadPoolExecutor创建线程池呢？

#### 在此我完全参考了dubbo中创建线程池的方法，代码如下。

#### ThreadPool.java代码
```java
/**
 * ThreadPool
 *
 * Created by wuyoushan on 2017/10/31.
 */
public interface ThreadPool {

    Executor getExecutor();
}
```

#### NamedThreadFactory.java代码
```java
/**
 * InternalThreadFactory
 *
 * Created by wuyoushan on 2017/11/1.
 */
public class NamedThreadFactory implements ThreadFactory {

    private static final AtomicInteger POOL_SEQ = new AtomicInteger(1);

    private final AtomicInteger mThreadNum = new AtomicInteger(1);

    private final String mPrefix;

    private final boolean mDaemo;

    private final ThreadGroup mGroup;

    public NamedThreadFactory() {
        this("pool-" + POOL_SEQ.getAndIncrement(), false);
    }

    public NamedThreadFactory(String prefix) {
        this(prefix, false);
    }

    public NamedThreadFactory(String prefix, boolean daemo) {
        mPrefix = prefix + "-thread-";
        mDaemo = daemo;
        SecurityManager s = System.getSecurityManager();
        mGroup = (s == null) ? Thread.currentThread().getThreadGroup() : s.getThreadGroup();
    }

    @Override
    public Thread newThread(Runnable runnable) {
        String name = mPrefix + mThreadNum.getAndIncrement();
        Thread ret = new Thread(mGroup, runnable, name, 0);
        ret.setDaemon(mDaemo);
        return ret;
    }

    public ThreadGroup getThreadGroup() {
        return mGroup;
    }
}
```

#### CachedThreadPool.java代码
```java
/**
 * 此线程池可伸缩，线程空闲一分钟后回收，新请求重新创建线程，来源于：<code>Executors.newCachedThreadPool()</code>
 * 
 * Created by wuyoushan on 2017/10/31.
 */
public class CachedThreadPool implements ThreadPool {

    @Override
    public Executor getExecutor() {
        String name = "Dubbo";
        int cores = 0;
        int threads = Integer.MAX_VALUE;
        int queues = 0;
        int alive = 60 * 1000;
        return new ThreadPoolExecutor(cores, threads, alive, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedThreadFactory(name, true), new ThreadPoolExecutor.AbortPolicy());
    }
}

```

#### FixedThreadPool.java代码
```java
/**
 * 此线程池启动时即创建固定大小的线程数，不做任何伸缩，来源于：<code>Executors.newFixedThreadPool()</code>
 *
 * Created by wuyoushan on 2017/10/31.
 */
public class FixedThreadPool implements ThreadPool{

    @Override
    public Executor getExecutor() {
        String name = "Dubbo";
        int threads = 200;
        int queues = 0;
        return new ThreadPoolExecutor(threads,threads,0, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedThreadFactory(name, true), new ThreadPoolExecutor.AbortPolicy());
    }
}
```

#### LimitedThreadPool.java代码
```java
/**
 * 此线程池一直增长，直到上限，增长后不收缩。
 *
 * Created by wuyoushan on 2017/10/31.
 */
public class LimitedThreadPool implements ThreadPool{

    @Override
    public Executor getExecutor() {
        String name = "Dubbo";
        int cores = 0;
        int threads = 200;
        int queues = 0;
        return new ThreadPoolExecutor(cores,threads,Long.MAX_VALUE, TimeUnit.MILLISECONDS,
                queues == 0 ? new SynchronousQueue<Runnable>() :
                        (queues < 0 ? new LinkedBlockingQueue<Runnable>()
                                : new LinkedBlockingQueue<Runnable>(queues)),
                new NamedThreadFactory(name, true), new ThreadPoolExecutor.AbortPolicy());
    }
}
```

### 注意:
上面的线程池的创建和JDK中Executors基于Executors和ThreadPoolExecutor是有所差异的。

1. dubbo中线程池创建的线程为守护进程，而使用Executors和ThreadPoolExecutor默认线程工厂创建的线程池为非守护线程。

2. dubbo中根据参数使用的阻塞队列，以及定义队列的长度都是不一样的。

#### 上面代码完全参考自dubbo项目源码
git地址 https://github.com/alibaba/dubbo