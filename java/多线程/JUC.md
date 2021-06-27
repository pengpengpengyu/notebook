# JUC

## 简介

java.util.concurrent工具包

**业务：普通的线程代码 Thread**

**Runnable 没有返回值、效率相比Callable低**

## 线程和进程

> 进程： 一个程序，例如QQ.exe,Music.exe程序的集合，一个线程往往包含多个线程，至少包含一个；java默认有两个线程：main,GC
>
> 线程：开启一个进程Typora,写字，自动保存即是线程负责的

### 并发、并行

并发：多线程操作同一个资源

- CPU单核，模拟出来多条线程，快速交替

并行：

- CPU多核，多个线程同时执行

### wait/sleep区别

1. 来自不同的类

   wait -- Object

   sleep -- Thread

2. 关于锁的释放

   wait会释放锁，sleep不会释放锁

3. 使用范围不同

   wait必须在同步代码块中

   sleep可以在任意位置

## Lock锁

> Synchronized和Lock的区别

1. Synchronized内置Java关键字，Lock是一个Java类
2. Synchronized 无法判断获取锁的状态,Lock可以判断是否获取到了锁
3. Synchronized会自动释放锁，Lock必须要手动释放锁，如果不释放锁就形成了死锁
4. Synchronized线程1（获得锁）,线程2就一直等待;Lock锁就不一定会等待
5. Synchronized可重入锁，不可以中断，非公平；Lock可重入锁，可以中断，可以设置公不公平
6. Synchronized适合锁少量的代码同步问题；Lock适合锁大量的同步代码

> Synchronized、Lock锁Demo

```java
package com.example.juc.lock;

/**
 * 线程就是一个单独的资源类，没有任何的附属操作
 * 1.属性、方法
 * 并发：多线程操作同一资源
 */
public class SaleTicketDemo1 {

    public static void main(String[] args) {
        Ticket ticket = new Ticket();

        new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                ticket.sale();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                ticket.sale();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                ticket.sale();
            }
        }, "C").start();


    }
}

class Ticket {

    private int num = 30;

    // Synchronized锁
    public synchronized void sale() {
        if (num > 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "卖出去了第" + (num--)+ "张票,剩余：" + (num));
        }
    }
}
```

```javascript
package com.example.juc.lock;

import java.util.concurrent.locks.ReentrantLock;

/**
 * 线程就是一个单独的资源类，没有任何的附属操作
 * 1.属性、方法
 * 并发：多线程操作同一资源
 */
public class SaleTicketDemo2 {

    public static void main(String[] args) {
        Ticket2 ticket2 = new Ticket2();

        new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                ticket2.sale();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                ticket2.sale();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                ticket2.sale();
            }
        }, "C").start();


    }
}

class Ticket2 {

    private int num = 30;
    // Lock锁
    ReentrantLock lock = new ReentrantLock();

    public synchronized void sale() {
        try {
            lock.lock();
            if (num > 0) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "卖出去了第" + (num--) + "张票,剩余：" + (num));
            }
        } finally {
            lock.unlock();
        }
    }
}
```

## 生产者和消费者问题

> Synchronized版本

```java
package com.example.juc.lock;

/**
 * 生产者，消费者测试
 */
public class PCTest {

    public static void main(String[] args) {
        A a = new A();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    a.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    a.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

/**
 * 操作类
 */
class A {
    private int num = 0;

    // 生产者方法，+1
    public synchronized void increment() throws InterruptedException {
        if (num != 0) {
            this.wait();
        }
        System.out.println(Thread.currentThread().getName() + "---> " + (++num));
        this.notify();
    }

    // 消费者方法, -1
    public synchronized void decrement() throws InterruptedException {
        if (0 == num) {
            this.wait();
        }

        System.out.println(Thread.currentThread().getName() + "---> " + (--num));
        this.notify();
    }
}
```

> JUC版本

```java
package com.example.juc.lock;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 生产者，消费者测试
 */
public class PCTest2 {

    public static void main(String[] args) {
        int count = 500;
        B a = new B();

        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                try {
                    a.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                try {
                    a.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                try {
                    a.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                try {
                    a.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

/**
 * 操作类
 */
class B {
    private int num = 0;

    // 锁
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    // 生产者方法，+1
    public void increment() throws InterruptedException {
        lock.lock();
        try {
            while (num != 0) {
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "---> " + (++num));
            condition.signalAll();
        } finally {
            lock.unlock();
        }

    }

    // 消费者方法, -1
    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (0 == num) {
                condition.await();
            }

            System.out.println(Thread.currentThread().getName() + "---> " + (--num));
            condition.signalAll();
        } finally {
            lock.unlock();
        }

    }
}
```

```java
package com.example.juc.lock;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 按照顺序执行
 */
public class PCTest3 {

    public static void main(String[] args) {
        A3 a3 = new A3();
        int count = 30;
        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                a3.printA();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                a3.printB();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < count; i++) {
                a3.printC();
            }
        }, "C").start();
    }
}

class A3 {

    private int num = 1;

    Lock lock = new ReentrantLock();
    Condition conditionA = lock.newCondition();
    Condition conditionB = lock.newCondition();
    Condition conditionC = lock.newCondition();

    public void printA() {
        lock.lock();
        try {
            while (1 != num) {
                conditionA.await();
            }
            num = 2;
            System.out.println(Thread.currentThread().getName() + " ---> AAAAAAAAAAAAAAA");
            conditionB.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();
        try {
            while (2 != num) {
                conditionB.await();
            }
            num = 3;
            System.out.println(Thread.currentThread().getName() + " ---> BBBBBBBBBB");
            conditionC.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();
        try {
            while (3 != num) {
                conditionC.await();
            }
            num = 1;
            System.out.println(Thread.currentThread().getName() + " ---> CCCCCCCCC");
            conditionA.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

## 8锁现象

> **锁对象**

```java
package com.example.juc.lock.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 关于锁的8个问题
 * Synchronized 获取的同一个对象的锁(此时锁的是对象)，谁先拿到谁执行，所以必须等待发释放锁以后，打电话才能获取锁
 * 结果：发短信，打电话
 */
public class Test1 {

    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> phone.sendSms()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone.call()).start();
    }
}

class Phone {

    // 此时锁的是对象
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 此时锁的是对象
    public synchronized void call() {
        System.out.println("打电话");
    }
}
```

```java
package com.example.juc.lock.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 关于锁的8个问题
 * 由于hello()方法并没有获取锁，不是同步方法，所以会立即执行，不会受锁的影响
 * hello,发短信
 */
public class Test2 {

    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(() -> phone.sendSms()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // new Thread(() -> phone.call()).start();

        new Thread(() -> phone.hello()).start();
    }
}

class Phone2 {

    // 此时锁的是对象
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 此时锁的是对象
    public synchronized void call() {
        System.out.println("打电话");
    }

    public void hello() {
        System.out.println("hello");
    }
}

```

```java
package com.example.juc.lock.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 关于锁的8个问题
 * Synchronized 锁的是对象，phone,phone2 是两个不同的对象，所以调用方法执行顺序按延时时间执行
 * 结果： 打电话，再发短信
 */
public class Test2 {

    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        Phone2 phone2 = new Phone2();

        new Thread(() -> phone.sendSms()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
         new Thread(() -> phone2.call()).start();

        // new Thread(() -> phone.hello()).start();
    }
}

class Phone2 {

    // 此时锁的是对象
    public synchronized void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 此时锁的是对象
    public synchronized void call() {
        System.out.println("打电话");
    }

    public void hello() {
        System.out.println("hello");
    }
}
```

> **锁类**

```java
package com.example.juc.lock.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 关于锁的8个问题
 * Synchronized 由于是静态方法，此时锁的是类，只有一把锁
 * 结果：发短信，打电话
 */
public class Test3 {

    public static void main(String[] args) {
        Phone3 phone = new Phone3();
        new Thread(() -> phone.sendSms()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone.call()).start();
    }
}

class Phone3 {

    // 因为是静态方法，此时锁的是类
    public synchronized static void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 此时锁的是对象
    public synchronized static void call() {
        System.out.println("打电话");
    }
}
```

```java
package com.example.juc.lock.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 关于锁的8个问题
 * Synchronized 由于是静态方法，此时锁的是类，只有一把锁
 * 结果：发短信，打电话
 */
public class Test3 {

    public static void main(String[] args) {
        // 由于当前调用的是静态方法，锁的仍然是类，即使多个对象，仍然只有一把锁
        Phone3 phone = new Phone3();
        Phone3 phone3 = new Phone3();
        new Thread(() -> phone.sendSms()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone3.call()).start();
    }
}

class Phone3 {

    // 因为是静态方法，此时锁的是类
    public synchronized static void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    // 此时锁的是对象
    public synchronized static void call() {
        System.out.println("打电话");
    }
}
```

```java
package com.example.juc.lock.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 关于锁的8个问题
 * 一个静态同步方法，一个普通同步方法，一个普通方法，一个对象
 * 普通方法不受锁的影响，会在同步方法线程sleep期间执行;静态同步方法锁的是类，普通同步方法锁的是对象
 * 结果：hello,打电话，发短信
 */
public class Test4 {

    public static void main(String[] args) {

        Phone4 phone = new Phone4();
        // Phone4 phone1 = new Phone4();
        new Thread(() -> phone.sendSms()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone.call()).start();

        new Thread(() -> phone.hello()).start();
    }
}

class Phone4 {

    // 因为是静态方法，此时锁的是类
    public synchronized static void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }


    public synchronized void call() {
        System.out.println("打电话");
    }

    public void hello() {
        System.out.println("hello");
    }
}
```

```java
package com.example.juc.lock.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 关于锁的8个问题
 * 一个静态同步方法，一个普通同步方法，一个普通方法，两个对象
 * 普通方法不受锁的影响，会在同步方法线程sleep期间执行;静态同步方法锁的是类，普通同步方法锁的是对象
 * 结果：hello/打电话（顺序随机），发短信
 */
public class Test4 {

    public static void main(String[] args) {

        Phone4 phone = new Phone4();
        Phone4 phone1 = new Phone4();
        new Thread(() -> phone.sendSms()).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> phone.hello()).start();
        new Thread(() -> phone1.call()).start();

    }
}

class Phone4 {

    // 因为是静态方法，此时锁的是类
    public synchronized static void sendSms() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }


    public synchronized void call() {
        System.out.println("打电话");
    }

    public void hello() {
        System.out.println("hello");
    }
}
```

> 总结

new this 锁的是具体的一个对象

static 锁的是类