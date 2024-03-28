---
title: "Interview Java Thread"
date: 2024-02-10T23:53:56+08:00
draft: true
tags: ["Java"]
featuredImage: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
---



### 多线程

**多线程的好处：**

1. **提高程序性能：** 充分利用多核处理器，同时执行多个线程，加速任务完成速度，提高程序整体性能。
2. **提高系统吞吐量：** 多线程可以使系统同时处理多个任务，提高系统的吞吐量，提高资源利用率。
3. **异步编程：** 多线程可以实现异步编程，允许在后台执行任务，不阻塞主线程，提高程序的并发性。
5. **资源共享：** 多线程可以共享内存空间，使得线程之间可以方便地共享数据，提高灵活性。

**多线程的问题：**

1. **竞态条件（Race Condition）：** 多个线程同时访问共享数据，可能导致数据不一致或错误的结果，需要使用同步机制来避免。
2. **死锁（Deadlock）：** 多个线程因为相互等待对方释放资源而陷入无限等待的状态，导致程序无法继续执行。
3. **线程安全性问题：** 在没有适当同步的情况下，多个线程对共享数据的并发访问可能导致数据损坏或不一致。
4. **上下文切换开销：** 线程的切换会引入一定的开销，当线程数量过多时，可能导致性能下降。

平时说的多线程既包括并发（Concurrency）又包括并行（Parallelism）。

- **并发：** 多线程的并发是指多个线程在同一时刻处于活动状态，但并不一定同时执行。在单个 CPU 上，通过快速地切换执行不同线程的任务，实现了看似同时执行的效果。

- **并行：** 多线程的并行是指多个线程真正同时在多个处理器核心或多个 CPU 上执行各自的任务。

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

### synchronized

`synchronized` 是 Java 中的关键字，用于实现同步机制，确保多个线程在访问共享资源时能够安全地协调执行，避免竞争条件。

- **用法：** 可以用在方法声明上，也可以用在代码块中。例如：

  ```java
  public synchronized void synchronizedMethod() {
      // 同步的方法体
  }
  
  public void someMethod() {
      synchronized (this) {
          // 同步的代码块
      }
  }
  ```

- **使用场景：** 适用于多个线程共享资源的情况，例如共享数据结构的读写操作，确保线程安全。需要注意的是，使用 `synchronized` 会带来性能开销，因此在某些场景下可能会选择使用更灵活的锁机制，如 `ReentrantLock`。

- **原理：**当一个线程进入一个`synchronized`方法或代码块时，JVM 在字节码层面会插入 `monitorenter` 指令，该指令尝试获取对象关联的 monitor 锁。具体流程如下：

  1. **线程尝试获取锁：** 进入 `synchronized` 方法或代码块时，执行 `monitorenter` 指令，尝试获取对象的 monitor 锁。
  2. **锁状态检查：** 如果 monitor 处于无锁状态，线程成功获取锁，将 monitor 标记为被当前线程占用。执行同步块内的代码。
  3. **锁升级：**  JVM 使用锁升级的方式来提高性能。初始时，对象的监视器对象是无锁状态，当一个线程获取锁后，它会变成偏向锁。如果有其他线程尝试获取该锁，偏向锁会升级为轻量级锁，这时候其他线程仍然可以比较容易地竞争锁。如果轻量级锁竞争失败，它将升级为重量级锁，这时候会涉及到操作系统层面的互斥量等。
  4. **锁释放：** 执行同步块后，JVM 插入 `monitorexit` 指令，释放 monitor 锁。其他等待的线程可以竞争获取锁。

### ReentrantLock

`ReentrantLock`（可重入锁）是Java中的一种锁机制，允许线程多次获取同一把锁。这种锁的特点是允许同一线程在持有锁的同时多次获取它，而不会发生死锁。

**用法：**创建锁，获取锁，释放锁

```java
ReentrantLock lock = new ReentrantLock();
if (lock.tryLock()) { // 使用 tryLock() 方法尝试获取锁，返回 true 表示成功获取锁。
    try {
        // 临界区代码
    } finally {
        lock.unlock();
    }
} else {
    // 处理获取锁失败的情况
}
```

**使用场景：**

1. **替代`synchronized`：** 提供了比 `synchronized` 更灵活的同步控制，适用于一些复杂的同步需求。

2. **可中断的等待：** `ReentrantLock` 支持可中断的锁获取操作，有助于处理死锁情况。

   ```java
   import java.util.concurrent.locks.ReentrantLock;
   import java.util.concurrent.locks.Lock;
   
   public class InterruptibleLockExample {
       private final Lock lock = new ReentrantLock();
   
       public void performTask() throws InterruptedException {
           try {
               lock.lockInterruptibly(); // 可中断地获取锁
               // 执行任务
           } finally {
               lock.unlock();
           }
       }
   
       public static void main(String[] args) {
           InterruptibleLockExample example = new InterruptibleLockExample();
           Thread thread = new Thread(() -> {
               try {
                   example.performTask();
               } catch (InterruptedException e) {
                   System.out.println("Thread interrupted");
               }
           });
   
           thread.start();
   
           // 在某个时刻中断线程
           thread.interrupt();
       }
   }
   ```

3. **公平锁和非公平锁：** 可以选择创建公平锁（按照请求的顺序分配锁）或非公平锁。

   ```java
   import java.util.concurrent.locks.ReentrantLock;
   import java.util.concurrent.locks.Lock;
   
   public class FairnessExample {
       // 公平锁
       private final Lock fairLock = new ReentrantLock(true);
   
       public void performTaskFair() {
           try {
               fairLock.lock();
               // 执行任务
           } finally {
               fairLock.unlock();
           }
       }
   }
   ```

4. **条件变量：** `ReentrantLock` 提供了 `Condition` 对象，可用于线程间的协调和通信。可以实现分组唤醒需要唤醒的线程们，而不是像 `synchronized` 要么随机唤醒一个线程要么唤醒全部线程

   ```java
   import java.util.concurrent.locks.Condition;
   import java.util.concurrent.locks.ReentrantLock;
   import java.util.concurrent.locks.Lock;
   
   public class ConditionVariableExample {
       private final Lock lock = new ReentrantLock();
       private final Condition condition = lock.newCondition();
       private boolean conditionMet = false;
   
       public void awaitCondition() throws InterruptedException {
           lock.lock();
           try {
               while (!conditionMet) {
                   condition.await(); // 等待条件满足
               }
               // 执行任务
           } finally {
               lock.unlock();
           }
       }
   
       public void signalCondition() {
           lock.lock();
           try {
               conditionMet = true;
               condition.signal(); // 唤醒等待线程
           } finally {
               lock.unlock();
           }
       }
   }
   ```

**原理：**

1. **Reentrant特性：** 允许同一线程多次获取锁，通过维护一个持有锁的线程计数器来实现。

2. **公平性：** 可选择公平或非公平锁。公平锁按照请求的顺序分配锁，而非公平锁允许插队。

3. **AQS（AbstractQueuedSynchronizer）：** `ReentrantLock` 的实现依赖于AQS，它是用于构建锁和其他同步工具的框架。

4. **Condition：** 提供了类似 `Object.wait()` 和 `Object.notify()` 的条件变量，允许线程精确地等待或唤醒。

### **volatile**

`volatile` 是 Java 的一个关键字，用来确保将变量的更新操作通知到其他线程。在访问 `volatile` 变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此 `volatile` 变量是一种比 `synchronized`  关键字更轻量级的同步机制。

**特性：**

1. **可见性：** 当一个线程修改了 `volatile` 变量的值，其他线程能够立即看到这个变化。

2. **禁止指令重排：** `volatile` 修饰的变量的读写操作不能被重排序，保证了操作的原子性。

**原理：**

`volatile` 的原理涉及到主内存与工作内存之间的交互。在Java中，每个线程都有自己的工作内存，线程对变量的操作首先在工作内存中进行，然后通过主内存来实现可见性和禁止指令重排序。

1. **可见性：** 当一个线程写入一个 `volatile` 变量时，它会将该变量的值刷新到主内存。其他线程读取这个变量时，会从主内存中获取最新的值，而不是使用自己的工作内存中的缓存。这确保了对 `volatile` 变量的写操作对其他线程是可见的。

2. **禁止指令重排：** 在读取和写入 `volatile` 变量的操作前后，会插入内存屏障，防止指令重排序。

**使用场景：**

1. **状态标志：** 多线程协同工作时，通过改变一个变量的状态来通知其他线程，`volatile` 可以确保状态的及时可见性。

2. **双重检查锁定：** 在单例模式中，使用 `volatile` 可以确保在多线程环境中正确地初始化单例对象。

   ```java
   public class Singleton {
       // volatile 确保了在 instance 被初始化时，其他线程能够立即看到这个变化。
       private static volatile Singleton instance;
   
       private Singleton() {
           // 私有构造函数，防止外部实例化
       }
   
       public static Singleton getInstance() {
           if (instance == null) {
               synchronized (Singleton.class) {
                   // 再次检查，确保在同步块内对象仍未被初始化
                   if (instance == null) {
                       instance = new Singleton();
                   }
               }
           }
           return instance;
       }
   }
   ```


使用 `volatile` 必须同时满足下面两个条件才能保证在并发环境的线程安全：

1. **对变量的写操作不依赖于当前值**（比如 `i++`），或者说是单纯的变量赋值（`boolean flag = true`）。
2. 不同的 `volatile` 变量之间不能互相依赖。只有在状态真正独立于程序内其他内容时才能使用 `volatile`。

### ThreadLocal

`ThreadLocal` 是一个 Java 中的类，它提供了线程内的局部变量。每个线程都有自己独立的变量副本，互不影响。需要小心使用，因为不当的管理可能导致内存泄漏。

**原理：**

通过 `ThreadLocalMap` 实现的，每个线程内部都有一个 `ThreadLocalMap`，其中保存了各个 `ThreadLocal` 对应的值。线程局部变量通过 `ThreadLocal` 对象的 `get` 和 `set` 方法来访问和修改。

**使用场景：**

适用于需要在多线程环境中存储线程的局部变量数据，并保证线程安全的场景。`ThreadLocal` 在以下场景中很有用：

1. **用户信息管理：** 在一个使用线程池的Web服务器中，可以使用 `ThreadLocal` 存储用户的身份信息，确保每个请求处理线程都能访问正确的用户上下文信息，而不必在每个方法中显式传递用户对象。

```java
public class UserContext {
    private static final ThreadLocal<User> userThreadLocal = new ThreadLocal<>();

    public static void setUser(User user) {
        userThreadLocal.set(user);
    }

    public static User getUser() {
        return userThreadLocal.get();
    }
}

// 在请求处理中设置和获取用户信息
UserContext.setUser(currentUser);
// 在其他方法中获取用户信息，无需显式传递
User user = UserContext.getUser();
```

2. **数据库连接管理：** 在数据库连接池中，可以使用 `ThreadLocal` 存储每个线程的数据库连接，确保每个线程都使用自己的连接，避免混淆。

```java
public class DBConnectionManager {
    private static final ThreadLocal<Connection> connectionThreadLocal = new ThreadLocal<>();

    public static Connection getConnection() {
        // 获取当前线程的数据库连接
        return connectionThreadLocal.get();
    }

    public static void setConnection(Connection connection) {
        // 设置当前线程的数据库连接
        connectionThreadLocal.set(connection);
    }
}
```

3. 当使用 `ThreadLocal` 时，内存泄漏可能发生在不及时清理 `ThreadLocal` 中的对象，导致对象一直被引用而无法被垃圾回收。以下是一个简单的例子：

```java
public class ThreadLocalLeakExample {
    private static ThreadLocal<Object> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        // 在一个线程中设置 ThreadLocal 值
        threadLocal.set(new Object());

        // 模拟线程结束，但未清理 ThreadLocal 中的对象
        // 实际情况中可能因为线程池的重用等原因导致 ThreadLocal 对象一直被引用
        Thread thread = new Thread(() -> {
            // 业务逻辑...

            // 线程结束，但是 ThreadLocal 中的对象仍然存在
        });
        thread.start();

        // 可能在这里没有及时调用 threadLocal.remove()
        // 导致 ThreadLocal 中的对象无法被释放，发生内存泄漏
    }
}
```

在上述例子中，线程结束时，`ThreadLocal` 中的对象没有被清理，**如果这个线程是由线程池提供的，可能会在其他任务中重用同一个线程**，从而导致 `ThreadLocal` 中的对象一直被引用，无法被垃圾回收，从而发生内存泄漏。要避免这种情况，需要在不再需要时显式调用 `threadLocal.remove()` 来清理 `ThreadLocal` 中的对象。

### CountDownLatch

`CountDownLatch` 是 Java 中的一个同步辅助类，用于等待某个事件执行完毕后再继续执行当前线程。它的核心思想是一个计数器，每个线程完成自己的任务时将计数器减一，当计数器为零时，所有等待的线程都会被唤醒。

**使用场景：**

1. **主线程等待其他线程完成初始化：** 主线程等待其他子线程完成初始化工作后再继续执行。
2. **并行任务的等待：** 多个任务并行执行，主线程等待所有任务完成后再进行下一步操作。

以下是简单的代码示例：

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) {
        final int threadCount = 3;
        CountDownLatch latch = new CountDownLatch(threadCount);

        for (int i = 0; i < threadCount; i++) {
            Thread thread = new Thread(new WorkerThread(latch));
            thread.start();
        }

        try {
            // 主线程等待所有子线程执行完毕
            latch.await();
            System.out.println("All threads have completed their work.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    static class WorkerThread implements Runnable {
        private final CountDownLatch latch;

        WorkerThread(CountDownLatch latch) {
            this.latch = latch;
        }

        @Override
        public void run() {
            // 模拟线程执行一些操作
            System.out.println(Thread.currentThread().getName() + " is working");
            
            // 操作完成，计数器减一
            latch.countDown();
        }
    }
}
```

在这个例子中，主线程创建了多个子线程，并传递了一个 `CountDownLatch`，每个子线程在执行完任务后都会调用 `countDown()` 方法，表示自己的工作已完成。主线程通过 `await()` 方法等待计数器减为零，然后继续执行后续操作。

### CyclicBarrier

`CyclicBarrier`（回环栅栏）是 Java 中的同步辅助类，用于实现一组线程协同工作，要求直到所有线程都到达某个屏障点（barrier point）后再同时继续执行。与 `CountDownLatch` 不同，`CyclicBarrier` 可以循环使用，即当所有线程都到达屏障点后，计数器重新初始化，可以再次使用。

**使用场景：**

1. **分阶段并行计算：** 多个线程分阶段执行任务，每个阶段需要等待所有线程完成当前阶段的工作后再进入下一阶段。
2. **多线程游戏：** 在游戏中，可能需要等待所有玩家完成某个阶段的操作后再进行下一轮的游戏。

CyclicBarrier 中最重要的方法就是 await 方法，它有 2 个重载版本：
1. `public int await()`：用来挂起当前线程，直至所有线程都到达 barrier 状态再同时执行后续任
务；
2. `public int await(long timeout, TimeUnit unit)`：让这些线程等待至一定的时间，如果还有
线程没有到达 barrier 状态就直接让到达 barrier 的线程执行后续任务。

以下是一个简单的代码示例：

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
    public static void main(String[] args) {
        final int threadCount = 3;
        CyclicBarrier barrier = new CyclicBarrier(threadCount, () -> {
            System.out.println("All threads have reached the barrier. Starting the next phase.");
        });

        for (int i = 0; i < threadCount; i++) {
            Thread thread = new Thread(new WorkerThread(barrier));
            thread.start();
        }
    }

    static class WorkerThread implements Runnable {
        private final CyclicBarrier barrier;

        WorkerThread(CyclicBarrier barrier) {
            this.barrier = barrier;
        }

        @Override
        public void run() {
            // 模拟线程执行一些操作
            System.out.println(Thread.currentThread().getName() + " is working");

            try {
                // 等待其他线程到达屏障点
                barrier.await();
                System.out.println(Thread.currentThread().getName() + " has passed the barrier");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

在这个例子中，主线程创建了多个子线程，并传递了一个 `CyclicBarrier`，每个子线程在执行完任务后都会调用 `await()` 方法等待其他线程。当所有线程都到达屏障点时，屏障点的任务会执行，然后所有线程再一起继续执行。这个过程可以循环进行。

### Semaphore

`Semaphore` （信号量）是 Java 中的同步辅助类，用于限制同时访问某个资源的线程数量，从而保护共享资源。线程在访问共享资源前必须先获得许可，每次访问后释放许可。

**使用场景：**

1. **有限资源池：** 例如数据库连接池，限制可同时使用的连接数量。
2. **流量控制：** 控制同时进行网络请求的线程数量，以避免过载。
3. **线程数量控制：** 控制同时运行的线程数量，以避免系统资源被耗尽。

以下是一个简单的代码示例：

```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
    public static void main(String[] args) {
        final int threadCount = 5;
        final Semaphore semaphore = new Semaphore(2); // 允许同时2个线程访问

        for (int i = 0; i < threadCount; i++) {
            Thread thread = new Thread(new WorkerThread(semaphore));
            thread.start();
        }
    }

    static class WorkerThread implements Runnable {
        private final Semaphore semaphore;

        WorkerThread(Semaphore semaphore) {
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                // 获取许可证，如果没有许可证则阻塞等待
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName() + " is working");

                // 模拟线程执行一些操作

                System.out.println(Thread.currentThread().getName() + " has finished working");

                // 释放许可证
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

在这个例子中，主线程创建了多个子线程，并传递了一个 `Semaphore`，每个子线程在执行任务前会先尝试获取许可证，如果许可证不足则会被阻塞。线程执行完任务后释放许可证，其他等待的线程就有机会获取许可证，从而实现并发访问的控制。

### 线程池

Java中的线程池是一种用于管理和复用线程的机制。它的主要特点为：**线程复用，控制线程并发数，管理线程**

**原理：**
每一个 `Thread` 的类都有一个 `start()` 方法。我们可以继承重写 `Thread` 类，在其 `start()` 方法中不断循环调用传递过来的`Runnable`对象来实现线程的复用。 这就是线程池的实现原理。循环方法中不断获取 `Runnable` 是用 `Queue` 实现的，在获取下一个 `Runnable` 之前可以是阻塞的。

**组成：**

一个典型的线程池通常包含以下四个主要组成部分：

1. **任务队列（Task Queue）：** 用于存储等待执行的任务。任务可以在队列中等待，直到线程池中的线程准备好执行它们。

2. **线程池管理器（Thread Pool Manager）：** 负责线程的创建、销毁和管理。它维护核心线程池和最大线程池的大小，根据需要动态地调整线程数量。

3. **工作线程（Worker Threads）：** 线程池中的线程。这些线程从任务队列中获取任务，并执行任务的run方法。线程池中的线程可以是核心线程（保持活动状态，即使没有任务执行）或非核心线程（在需要时创建，但在空闲时可能会被回收）。

4. **任务（Tasks）：** 需要执行的工作单元。可以通过实现Runnable接口或Callable接口来表示任务。线程池负责执行这些任务。

**实现：**

![](https://s2.loli.net/2024/02/24/TfzK1E7M25yG3th.png)

`ThreadPoolExecutor` 的构造方法如下：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

1. **核心线程数 (`corePoolSize`)：** 保持活动的线程数量。
2. **最大线程数 (`maximumPoolSize`)：** 允许的最大线程数量。
3. **线程的闲置超时时间 (`keepAliveTime`)：** 当线程数量超过核心线程数时，多余的空闲线程等待新任务的最长时间。
4. **时间单位 (`unit`)：** 用于指定超时时间的单位。
5. **任务队列 (`workQueue`)：** 存储等待执行的任务的队列。
6. **线程工厂 (`threadFactory`)：** 创建新线程的工厂。一般用默认的就行。
7. **拒绝策略 (`handler`)：** 当线程池饱和且无法处理新任务时的处理方式。

**工作流程：**

1. **任务提交：** 客户端提交任务给线程池。

2. **核心线程处理：** 尝试使用核心线程处理任务。

3. **任务队列：** 若核心线程忙碌或已满，任务进入队列等待。

4. **非核心线程处理：** 若队列已满，创建非核心线程处理任务。

5. **拒绝策略：** 若线程达到最大数且队列满，按照设定的拒绝策略处理。

6. **任务执行：** 被选中的线程从队列中取任务执行，执行完可能返回池等待下个任务或被销毁。

7. **线程超时处理：**针对非核心线程，当它们闲置超过设定的时间，线程会被终止。

### 阻塞队列

Java中有几种常见的阻塞队列，它们适用于不同的场景。以下是一些常见的阻塞队列及其特点：

1. **LinkedBlockingQueue：** 基于链表实现的阻塞队列，容量可以是有界的或无界的。 `LinkedBlockingQueue` 对于生产者端和消费者端分别**采用了独立的锁来控制数据同步**，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。
2. **ArrayBlockingQueue：** 基于数组实现的有界阻塞队列。适用于**固定大小**的线程池，可以控制线程数和任务的并发度。
3. **PriorityBlockingQueue：** 是一个**支持优先级**的无界阻塞队列。元素根据它们的自然顺序或者被提供的比较器顺序被插入队列。适用于需要按照优先级顺序处理任务的场景。需要注意的是不能保证同优先级元素的顺序。
4. **DelayQueue：** 是一个支持延迟获取的无界阻塞队列。队列中的元素必须实现 `Delayed` 接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。适用于按照延迟时间执行任务的场景，比如缓存失效，定时任务调度。
5. **SynchronousQueue：** 一个**没有存储元素**的阻塞队列。每一个 put 操作必须等待一个 take 操作，否则不能继续添加元素。适用于直接传递任务的场景。

### 拒绝策略

拒绝策略是在线程池饱和且无法处理新任务时，决定如何拒绝新任务的一种策略。Java中的`ThreadPoolExecutor`提供了几种内置的拒绝策略，也支持自定义拒绝策略。以下是一些常见的拒绝策略及其适用场景：

1. **AbortPolicy（默认）：** 抛出`RejectedExecutionException`异常。适用于希望立即知道线程池无法处理新任务的场景，通常是在任务队列已满且线程已达到最大数量时。

2. **CallerRunsPolicy：** 由提交任务的线程执行该任务。适用于希望任务直接由调用者线程执行而不是放入队列的场景，可以减缓任务提交速度。

3. **DiscardOldestPolicy：** 丢弃队列中等待时间最长的任务，然后尝试重新提交新任务。适用于希望保留最新任务而丢弃旧任务的场景。

4. **DiscardPolicy：** 直接丢弃无法处理的新任务，不抛出异常。适用于希望简单丢弃无法处理的任务而不抛出异常的场景。
