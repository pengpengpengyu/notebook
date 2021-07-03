# JUC（二）

## 阻塞队列

> 写入

如果队列满了，就必须阻塞等待

> 取

如果队列是空的，就必须阻塞等待生产

> 使用场景：

多线程，线程池

### 方法

| 方式         | 抛出异常  | 有返回值，不抛异常 | 阻塞等待 | 超时等待                  |
| ------------ | --------- | ------------------ | -------- | ------------------------- |
| 添加         | add()     | offer()            | put()    | offer(元素,延时时间,单位) |
| 移除         | remove()  | poll()             | take()   | poll(延时时间,单位)       |
| 检测队首元素 | element() | peek()             |          |                           |

### SynchronousQueue 同步队列

- 没有容量
- 进入一个元素，必须等待取出来之后才能再插入下一个

## 线程池

> 池化技术

程序的运行本质：占用系统的资源，优化资源的使用 ==> 池化技术

### 线程池的好处

- 降低资源消耗
- 提高响应速度
- 方便管理

### 三大方法

- Executors.newSingleThreadExecutor(); // 单个线程 默认最大21亿
- Executors.newFixedThreadPool(3); 创建固定大小的线程池 默认最大21亿
- Executors.newCachedThreadPool();  可伸缩的线程池 最大21亿

### 七大参数

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
   }

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

 public ThreadPoolExecutor(int corePoolSize, // 核心线程池大小
                              int maximumPoolSize, // 最大核心线程池大小
                              long keepAliveTime,  // 超时没人调用就会释放，超时时间
                              TimeUnit unit, // 超时单位
                              BlockingQueue<Runnable> workQueue,  // 阻塞队列，阻塞队列满时，触发最大核心线程
                              ThreadFactory threadFactory,  // 线程工厂
                              RejectedExecutionHandler handler  // 拒绝策略) {  // 阻塞队列满并且最大核心线程满，触发拒绝策略
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

### 四大拒绝策略

- AbortPolicy 抛出异常
- CallerRunsPolicy 由主线程执行，哪里来的哪里去
- DiscardOldestPolicy  尝试和最早的任务竞争，如果最早的任务没有结束，当前任务将被丢弃，不抛异常
- DiscardPolicy    丢掉任务，不抛异常

### 最大核心线程数定义

- CPU密集型，几核，就是几，可以保证CPU的效率最高
  - `Runtime.getRuntime().availableProcessors()` 获取CPU的线程数
- IO密集型， 判断程序中十分消耗IO的线程数*2

## 异步回调

Feture设计的初衷：对将来的某个事件的结果进行建模

`CompletableFuture<T>` 异步回调的Feture实现类

```java
package com.example.juc.future;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

public class Demo01 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

       /* CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName() + "ruAsync ==>Void");
        });

        System.out.println("1111111");
        future.get();*/


        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "supplyAsync==>Integer");
            try {
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            int a = 1 / 0;
            return 1024;
        });

        future = future.whenComplete((t, u) -> {  // 成功方法
            System.out.println("t===" + t);
            System.out.println("u===" + u);
        }).exceptionally(e -> {  // 失败方法
            System.out.println(e.getMessage());
            return 233;
        });

        System.out.println("aaaaaa");

        System.out.println(future.get());
    }
}
```

## JMM

Java内存模型，一个不存在的东西，相当于一个约定。

1. 线程解锁前，必须把共享变量立刻刷新
2. 线程加锁前，必须读取内存中的最新值放到工作内存中
3. 加锁和解锁操作的是同一把锁

### 内存交互8中方式

1. lock(锁定)，作用于主内存中的变量，把变量标识为线程独占的状态。
2. read(读取)，作用于主内存的变量，把变量的值从主内存传输到线程的工作内存中，以便下一步的load操作使用。
3. load(加载)，作用于工作内存的变量，把read操作主存的变量放入到工作内存的变量副本中。
4. use(使用)，作用于工作内存的变量，把工作内存中的变量传输到执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
5. assign(赋值)，作用于工作内存的变量，它把一个从执行引擎中接受到的值赋值给工作内存的变量副本中，每当虚拟机遇到一个给变量赋值的字节码指令时将会执行这个操作。
6. store(存储)，作用于工作内存的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用。
7. write(写入)：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。
8. unlock(解锁)：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。

1. 不允许read、load、store、write操作之一单独出现，也就是read操作后必须load，store操作后必须write。
2. 不允许线程丢弃他最近的assign操作，即工作内存中的变量数据改变了之后，必须告知主存。
3. 不允许线程将没有assign的数据从工作内存同步到主内存。
4. 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是对变量实施use、store操作之前，必须经过load和assign操作。
5. 一个变量同一时间只能有一个线程对其进行lock操作。多次lock之后，必须执行相同次数unlock才可以解锁。
6. 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值。在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值。
7. 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量。
8. 一个线程对一个变量进行unlock操作之前，必须先把此变量同步回主内存。

## Volatile

1. 保证可见性，多个线程可以感知同一个变量的变化

2. 不保证原子性，线程在执行任务的时候，要么同时成功，要么同时失败

   - 不加Lock和Synchronzied怎么保证原子性

     > 使用原子类解决原子性问题AtomicInteger,AtomicLong...

## 各种锁

### 公平锁、非公平锁

公平锁：不能插队，必须按顺序执行

```java
public ReentrantLock() {
        sync = new NonfairSync();
    }
```

非公平锁：可以插队执行，默认都是非公平锁

```java
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

### 可重入锁

（递归锁）

**允许同一个线程多次获取同一把锁**。比如一个递归函数里有加锁操作，递归过程中这个锁会阻塞自己吗？如果不会，那么这个锁就是**可重入锁**（因为这个原因可重入锁也叫做**递归锁**）**。**

Java里只要以Reentrant开头命名的锁都是可重入锁，而且**JDK提供的所有现成的Lock实现类，包括synchronized关键字锁都是可重入的。**

### 自旋锁

所谓自旋，说白了就是一个 while(true) 无限循环

```java
package com.example.juc.lock;

import java.util.concurrent.atomic.AtomicReference;

public class SpinLock {

    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void myLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "===> myLock");
        // 自旋锁
        // 如果当前是null设置值为thread
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    // 解锁
    public void unLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "===> myUnlock");
        atomicReference.compareAndSet(thread, null);
    }
}
```



```java
package com.example.juc.lock;

import java.util.concurrent.TimeUnit;

/**
 * T1在睡眠，T2在自旋
 */
public class TestSpinLock {

    public static void main(String[] args) throws InterruptedException {
        SpinLock spinLock = new SpinLock();

        new Thread(() -> {
            spinLock.myLock();

            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                spinLock.unLock();
            }
        }, "T1").start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(() -> {
           spinLock.myLock();

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                spinLock.unLock();
            }
        }, "T2").start();
    }
}

```

