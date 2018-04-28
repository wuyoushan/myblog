---
title: java多线程 Callable和Future的使用
date: 2017-10-24 22:26:00
tags: [java,多线程,读书笔记]
categories: [java,读书笔记]
---
## Callable和Future的定义

### Callable
Callable 接口类似于 Runnable，两者都是为那些其实例可能被另一个线程执行的类设计的。但是 Runnable 不会返回结果，并且无法抛出经过检查的异常。 


### Future
Future 表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。计算完成后只能使用 get 方法来获取结果，如有必要，计算完成前可以阻塞此方法。取消则由 cancel 方法来执行。还提供了其他方法，以确定任务是正常完成还是被取消了。一旦计算完成，就不能再取消计算。如果为了可取消性而使用 Future 但又不提供可用的结果，则可以声明 Future<?> 形式类型、并返回 null 作为底层任务的结果。 

### FutureTask
此类提供了对 Future 的基本实现。仅在计算完成时才能获取结果；如果计算尚未完成，则阻塞 get 方法。一旦计算完成，就不能再重新开始或取消计算。 

可使用 FutureTask 包装 Callable 或 Runnable 对象。因为 FutureTask 实现了 Runnable，所以可将 FutureTask 提交给 Executor 执行。

### 使用Callable和Future,FutureTask实现的一些例子

#### 例子1: Future和Callable的简单使用
```java
/**
 * Created by wuyoushan on 2017/10/24.
 */
public class CallableImpl implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        Thread.sleep(1000);
        int i=ThreadLocalRandom.current().nextInt(100);
        System.out.println("CallableImpl返回值为:"+i);
//        throw new IllegalArgumentException();
        return i;
    }
}

/**
 * Created by wuyoushan on 2017/10/24.
 */
public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService threadPool = new ThreadPoolExecutor(4, 16, 20L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024));

        Callable callable = new CallableImpl();
        Future<Integer> future = threadPool.submit(callable);

        System.out.println("haha");
        System.out.println("futurn="+future.get());

        Future<Integer> future1 = threadPool.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return 123;
            }
        });
        System.out.println("haha");
        System.out.println("future1="+future1.get());
    }
}

```
##### 运行结果为:
```java
haha
CallableImpl返回值为:82
futurn=82
haha
future1=12312
```
从上面获取到的异步执行的结果，future.get()方法将会阻塞顺序执行的过程(同理抛出异常的时候，同样会阻塞顺序进程),**但当没有使用future.get()方法时则不会阻塞**。

#### 例子2: FutureTask的简单使用
```java
/**
 * Created by wuyoushan on 2017/10/24.
 */
public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long startTime=System.currentTimeMillis();
        ExecutorService threadPool = new ThreadPoolExecutor(4, 16, 20L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024));

        FutureTask<Integer> futureTask=new FutureTask<>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                Thread.sleep(1000);
                return 123;
            }
        });

        FutureTask<Integer> futureTask1=new FutureTask<>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                Thread.sleep(2000);
                return 321;
            }
        });

        threadPool.execute(futureTask);
        new Thread(futureTask1).start();
        
        long endTime=System.currentTimeMillis();
        System.out.println("花费时间为="+(endTime-startTime));
        
        System.out.println("futureTask="+futureTask.get());
        System.out.println("futureTask1="+futureTask1.get());
        System.out.println("haha");

        long endTime1=System.currentTimeMillis();
        System.out.println("花费时间为="+(endTime1-startTime));
    }
}
```
##### 运行结果为:
```java
花费时间为=5
futureTask=123
futureTask1=321
haha
花费时间为=2005
```
FutureTask创建的异步线程，线程池执行的方式不一样的，同时FutureTask还可以使用Thread()执行获取异步结果。其他的使用和Future基本相同

#### 例子3: 异步批量获取结果集
```java
public class ForFutureExec {

        private final static ExecutorService pool = new ThreadPoolExecutor(4, 16, 20L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024));


    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Long start=System.currentTimeMillis();

        List<Future> list=listFuture(1);

        for (Future future:list){
            System.out.println(future.get());
        }
        Long end=System.currentTimeMillis();
        System.out.println("消耗时间="+(end-start));
    }

    private static List<Future> listFuture(Integer id){
        Future<Integer> future1=pool.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return countNum1(id);
            }
        });

        Future<Integer> future2=pool.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return countNum2(id);
            }
        });

        Future<Integer> future3=pool.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return countNum3(id);
            }
        });

        List<Future> list=new ArrayList<>();
        list.add(future1);
        list.add(future3);
        list.add(future2);
        return list;
    }

    private static int countNum1(Integer id){
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Thread.currentThread().getId();
        int num=new Random().nextInt(10);
        String str=String.format("返回输出的结果=%d,当前线程=%d,休眠时间=%d",num,Thread.currentThread().getId(),1000);
        System.out.println(str);
        return num;
    }

    private static int countNum2(Integer id){
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Thread.currentThread().getId();
        int num=new Random().nextInt(50);
        String str=String.format("返回输出的结果=%d,当前线程=%d,休眠时间=%d",num,Thread.currentThread().getId(),2000);
        System.out.println(str);
        return num;
    }

    private static int countNum3(Integer id){
        try {
            Thread.sleep(6000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Thread.currentThread().getId();
        int num=new Random().nextInt(100);
        String str=String.format("返回输出的结果=%d,当前线程=%d,休眠时间=%d",num,Thread.currentThread().getId(),6000);
        System.out.println(str);
        return num;
    }
}
```
##### 运行结果:
由于list中添加顺序的不同导致输出的也不一样
```java
//由于list添加future的顺序不一样会导致输出的结果有所差异
list.add(future1);      //休眠1000
list.add(future3);      //休眠6000
list.add(future2);      //休眠2000
```
输出结果
```java
返回输出的结果=7,当前线程=9,休眠时间=1000
7
返回输出的结果=34,当前线程=10,休眠时间=2000
返回输出的结果=81,当前线程=11,休眠时间=6000
81
34
消耗时间=6005
```
当list的顺序改变时输出的结果也会改变
```java
list.add(future1);      //休眠1000
list.add(future2);      //休眠2000
list.add(future3);      //休眠6000
```
输出结果为:
```java
返回输出的结果=9,当前线程=9,休眠时间=1000
9
返回输出的结果=3,当前线程=10,休眠时间=2000
3
返回输出的结果=14,当前线程=11,休眠时间=6000
14
消耗时间=6004
```
#### 例子4: 异步批量获取结果集4
```java
/**
 * Created by wuyoushan on 2017/10/19.
 */
public class Test {

    private final static ExecutorService pool = new ThreadPoolExecutor(4, 16, 20L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024));


    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Long start=System.currentTimeMillis();

        List<Future> list=new ArrayList<>(8);
        //异步执行
        for (int i=1;i<4;i++){
            list.add(asynExec(i*1000));
        }

        //异步执行
        for (int j=4;j>1;j--){
            list.add(asynExec(j*1000));
        }

        for (Future future:list){
            System.out.println(future.get());
        }
        Long end=System.currentTimeMillis();
        System.out.println("异步执行消耗时间="+(end-start));

        Long start1=System.currentTimeMillis();
        List<Integer> list1=new ArrayList<>(4);
        //同步执行
        for (int i=1;i<4;i++){
            list1.add(synchronizedExec(i*1000));
        }
        //同步执行
        for (int j=4;j>1;j--){
            list1.add(synchronizedExec(j*1000));
        }

        for (int result:list1){
            System.out.println(result);
        }

        Long end1=System.currentTimeMillis();
        System.out.println("同步执行消耗时间="+(end1-start1));

    }

    private static Future<Integer> asynExec(int time){
        Future<Integer> future=pool.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return countNum(time);
            }
        });
        return future;
    }

    private static int synchronizedExec(int time){
        return countNum(time);
    }

    private static int countNum(int time){
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Thread.currentThread().getId();
        int num=new Random().nextInt(10);
        String str=String.format("返回输出的结果=%d,当前线程=%d,休眠时间=%d",num,Thread.currentThread().getId(),time);
        System.out.println(str);
        return num;
    }
}

```
##### 运行结果为:
```java
返回输出的结果=8,当前线程=9,休眠时间=1000
8
返回输出的结果=2,当前线程=10,休眠时间=2000
2
返回输出的结果=7,当前线程=11,休眠时间=3000
7
返回输出的结果=1,当前线程=12,休眠时间=4000
1
返回输出的结果=2,当前线程=10,休眠时间=2000
返回输出的结果=1,当前线程=9,休眠时间=3000
1
2
异步执行消耗时间=4033
返回输出的结果=4,当前线程=1,休眠时间=1000
返回输出的结果=8,当前线程=1,休眠时间=2000
返回输出的结果=0,当前线程=1,休眠时间=3000
返回输出的结果=3,当前线程=1,休眠时间=4000
返回输出的结果=9,当前线程=1,休眠时间=3000
返回输出的结果=4,当前线程=1,休眠时间=2000
4
8
0
3
9
4
同步执行消耗时间=15001
```
