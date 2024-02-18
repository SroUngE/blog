---
title: "Interview Java Thread"
date: 2024-02-10T23:53:56+08:00
draft: false
tags: ["Java"]
featuredImage: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
---

### 线程的创建

1.继承 `Thread` 类，重写 `run()` 方法；

```java
public class MyThread extends Thread {
    @Override
	public void run() {
		System.out.println(Thread.currentThread().getName());
	}
	public static void main(String[] args) {
		MyThread mThread1=new MyThread();
		MyThread mThread2=new MyThread();
		MyThread myThread3=new MyThread();
		mThread1.start();
		mThread2.start();
		myThread3.start();
	}
}
```

2.实现 `Runnable` 接口，重写 `run()` 方法；

```java
public class MyThread implements Runnable {
    @Override
	public void run() {					
        System.out.println(Thread.currentThread().getName());
	}
	public static void main(String[] args) {
		MyThread Thread1=new MyThread();
		Thread mThread1=new Thread(Thread1,"线程1");
		Thread mThread2=new Thread(Thread1,"线程2");
		Thread mThread3=new Thread(Thread1,"线程3");
		mThread1.start();
		mThread2.start();
		myThread3.start();
	}
}
```

**实现 `Runnable` 接口比继承 `Thread` 类所具有的优势：**

1. **多继承问题：** Java的单继承机制限制了直接扩展`Thread`类的灵活性，而通过实现`Runnable`接口，你可以轻松地继承其他类。
2. **代码组织：** 将任务与线程分离，使得代码更易于组织和维护，也更符合面向对象设计的原则，提高了代码的清晰度。
3. **共享资源：** 使用`Runnable`接口创建线程允许多个线程共享相同的`Runnable`实例，这在某些情况下比直接使用`Thread`更有优势，提高了代码的可维护性。
4. 当涉及到线程池时，使用`Runnable`接口的优势更为明显：

   - **资源管理：** 线程池更好地管理系统资源，通过重复使用线程，减少线程创建和销毁的开销。使用`Runnable`接口允许将任务提交到线程池执行，而不是创建新线程，提高了资源利用率。
   - **任务隔离：** 通过将任务表示为`Runnable`对象，线程池可以更灵活地管理和调度任务。这样，你可以更容易地控制任务的执行方式、优先级等。


3.使用 `Executor` 工具类来创建线程池；

通过 `Executor` 的工具类可以创建三种类型的普通线程池：

- **FixThreadPool**：固定大小的线程池。使用于为了满足资源管理需求而需要限制当前线程数量的场合。使用于负载比较重的服务器。
- **SingleThreadPoolExecutor** ：单线程池。需要保证顺序执行各个任务的场景。
- **CashedThreadPool**：缓存线程池。当提交任务速度高于线程池中任务处理速度时，缓存线程池会不断的创建线程适用于提交短期的异步小程序，以及负载较轻的服务器。

```java 
public static void main(String[] args) {
    ExecutorService ex = Executors.newFixedThreadPool(5);
    for(int i=0;i<5;i++) {
        ex.submit(new Runnable() {			
                    @Override
                    public void run() {
                        System.out.println(Thread.currentThread().getName());					
                    }
            });
    }
    ex.shutdown();
}

```

4.实现 `Callable` 接口；

```java
class MyCallable implements Callable<String> {
    @Override
    public Integer call() throws Exception {
        return "Callable 任务执行完成";
    }
}

public static void main(String[] args) {
    FutureTask<String> task = new FutureTask<>(new MyCallable());\
    Thread thread = new Thread(task)
    thread.start();
    try {
        String result = result.get(); 
        System.out.println(result);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

### 线程的状态

1. **新建（New）：** 使用`new Thread()`创建线程对象。

2. **就绪（Runnable）：** 调用线程对象的`start()`方法将线程置于就绪状态，等待系统调度执行。

3. **运行（Running）：** 当线程得到调度并执行时，运行相应的`run()`方法。

4. **阻塞（Blocked）：** 调用`sleep()`、`wait()`等方法，或者在获取锁失败时会使线程进入阻塞状态。`wait()` 会释放持有的锁，`sleep()` 是不会释放持有的锁

5. **等待（Waiting）：** 调用`wait()`方法将线程置于等待状态，等待其他线程的通知或中断。

6. **计时等待（Timed Waiting）：** 调用`sleep()`、`join()`等带有时间限制的方法。

7. **终止（Terminated）：** 线程执行完毕或因异常而结束，线程对象进入终止状态。

### 常见的线程方法

**`sleep`：**

- `sleep()` 是一个静态方法，用于使当前线程暂停执行一段指定的时间。
- 通常用于在程序中引入时间延迟，不涉及线程同步。
- 释放CPU，**不释放对象锁**

```java
try {
    Thread.sleep(1000); // 暂停执行1秒钟
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

**`wait`：**

- `wait()` 是一个实例方法，通常与对象的锁结合使用，用于在多线程环境中实现线程之间的协作。
- 在调用 `wait()` 时，线程释放对象的锁，进入等待状态，直到其他线程调用相同对象的 `notify()` 或 `notifyAll()` 方法唤醒它。
- 释放CPU，**释放对象锁**

```java
synchronized (someObject) {
    try {
        someObject.wait(); // 当前线程等待，释放锁
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

**`yield`：**

- `yield` 是一个线程控制的关键字，用于提示调度器当前线程愿意放弃当前的执行权，让其他具有相同或更高优先级的线程有机会执行。
- 它并不会释放锁，只是让同优先级或更高优先级的其他线程有机会执行。

```java
Thread.yield(); // 提示调度器当前线程愿意让出执行权
```

**`join`：**

- `join` 是一个线程控制的方法，用于等待调用该方法的线程执行完毕。
- 在一个线程中调用另一个线程的 `join` 方法，当前线程将被阻塞，直到被调用的线程执行完毕。

```java
Thread thread = new Thread(() -> {
    // 线程执行的代码
});
thread.start();
try {
    thread.join(); // 等待thread执行完毕
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

**`notify` 和 `notifyAll`：**

- `notify` 和 `notifyAll` 是用于线程间通信的方法，通常与 `wait` 配合使用。
- `notify` 用于唤醒在对象上等待的一个线程，而 `notifyAll` 则唤醒所有等待的线程，哪一个线程将会第一个处理取决于操作系统的实现
- 这两个方法应该在同步块中调用，通常使用对象的锁来确保线程安全。

```java
synchronized (someObject) {
    someObject.notify(); // 唤醒一个等待的线程
    // 或
    someObject.notifyAll(); // 唤醒所有等待的线程
}
```

### 线程安全

Java 中确保线程安全有几种方式：

1. **互斥同步（Mutex Synchronization）：** 使用 `synchronized` 关键字或 `ReentrantLock` 等锁机制，确保在同一时刻只有一个线程可以访问共享资源，避免并发访问引起的数据不一致性。
2. **非阻塞同步（Non-blocking Synchronization）：** 使用并发包中的CAS（Compare and Swap）等原子操作，通过无锁的方式实现线程安全。这样的机制允许多个线程并发访问共享资源而不需要阻塞等待锁。
3. **无同步：** **不可变性（Immutability）：** 确保对象在创建后不能被修改，避免了多线程环境下的竞态条件。不可变对象本身是线程安全的。**线程本地变量（ThreadLocal）：** 每个线程都拥有自己独立的变量副本，避免了对共享变量的同步需求。线程本地变量是一种无同步的方案。

### 锁

锁是一种同步机制，用于控制对共享资源的访问。锁的目的是确保在任何给定的时刻，只有一个线程或进程能够访问共享资源，以防止并发访问导致数据不一致或竞争条件等问题。

**锁的种类**

下面是一些常见的锁的种类，按照不同的属性和特性进行分类：

1. **公平锁（Fair Lock）：** 所有线程按照请求锁的顺序获得锁，避免线程饥饿。如 `ReentrantLock`
2. **非公平锁（Unfair Lock）：** 不考虑线程请求锁的顺序，允许新来的线程插队成功获取锁。如 `synchronized`
3. **可重入锁（Reentrant Lock）：** 允许同一线程在持有锁的情况下再次获取锁，避免死锁。如 `ReentrantLock`、`synchronized`
4. **不可重入锁（Non-reentrant Lock）：** 不允许同一线程在持有锁的情况下再次获取锁，避免重复获取。如 `NonReentrantLock`
5. **乐观锁（Optimistic Locking）：** 假设多线程操作不会产生冲突，直接执行操作，如果发现冲突则回退或重试。如 `Atomic`
6. **悲观锁（Pessimistic Locking）：** 假设多线程操作会产生冲突，先获取锁再执行操作，防止冲突。如 `ReentrantLock`
7. **独享锁（Exclusive Lock）：** 一次只允许一个线程或进程持有锁。如 `ReentrantLock`
8. **共享锁（Shared Lock）：** 多个线程或进程可以同时持有锁，用于读取操作。如 `ReentrantReadWriteLock`
9. **自旋锁（Spin Lock）：** 线程在获取锁时不会立即阻塞，而是通过自旋等待锁的释放，避免上下文切换的开销。如 `Atomic`
10. **偏向锁（Biased Locking）：** 当只有一个线程访问同步块时，为该线程获取锁而不涉及竞争。如 `synchronized`
11. **轻量级锁（Lightweight Lock）：** 针对多线程竞争不激烈的场景，通过CAS（比较并交换）来减小锁的开销。如 `synchronized`
12. **重量级锁（Heavyweight Lock）：** 针对多线程竞争激烈的场景，采用传统的互斥方式，可能涉及线程的阻塞和唤醒。如 `synchronized`

锁的开销相对较大，而CAS（比较并交换）的开销相对较小的主要原因是它们在实现同步时采用了不同的策略。

1. **锁的开销大：**
   - **阻塞和唤醒开销：** 当使用传统的互斥锁（如重量级锁）时，如果一个线程获取了锁，而另一个线程尝试获取同一把锁时，后者会被阻塞，直到前者释放锁。这种阻塞和唤醒的机制涉及操作系统的系统调用，开销较大。
   - **上下文切换开销：** 当线程被阻塞时，操作系统需要进行上下文切换，从当前线程切换到另一个等待的线程，这也会导致开销增加。

2. **CAS的开销小：**
   - **原子性操作：** CAS是一种原子性操作，不需要阻塞线程。它使用硬件级别的原子性指令，比如 `compareAndSet` 操作。因为它是原子性的，所以不需要阻塞其他线程，避免了大部分阻塞和唤醒的开销。
   - **无锁操作：** 在一些情况下，CAS可以实现无锁算法。无锁算法允许多个线程并发地进行操作，而不会阻塞其他线程。这在低争用情况下能够提高性能。

总体而言，CAS的开销较小是因为它不涉及线程的阻塞和唤醒，而是通过硬件级别的原子性操作来实现。然而，在高争用的情况下，CAS也可能存在自旋等待的开销，因此在不同的情境下需要权衡选择适当的同步机制。
