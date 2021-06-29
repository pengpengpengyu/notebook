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

