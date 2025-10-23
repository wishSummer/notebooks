# Java 多线程学习笔记

---

## 一、多线程基础概念

### 1.1 进程与线程的区别

- 进程是资源分配的基本单位，一个进程包含多个线程。
- 线程是 CPU 调度的基本单位。
- 线程共享进程资源。

### 1.2 为什么需要多线程

- 提高 CPU 利用率
- 改善程序响应性
- 简化建模复杂任务

### 1.3 Java 线程与操作系统线程的关系

- JVM 线程映射模型
  - 绿色线程(已淘汰)：所有线程的调度都由 `JVM` 自己实现，而不是依赖操作系统。虽然跨平台性更强，JVM 统一调度机制，但是因为 `JVM` 无法访问操作系统的并行机制故不能利用多核 `CPU` 。
  - 本地线程(当前主流)：每一个 `Java Thread` 对象都映射为一个 操作系统级线程。JVM 不再自己调度线程，而是将调度任务交给操作系统。线程真正的执行由底层操作系统（如 Windows、Linux）来决定。

### 1.4 Java 线程模型 & Java 内存模型（JMM）

> `JMM` 是一组用于定义 `Java` 在多线程环境下，各线程之间如何共享变量、何时可见彼此的修改的抽象概念，定义了 `Java` 中的 `可见性、原子性、有序性` 原则。

- 主内存与工作内存
  - 主内存（Main Memory）：所有线程共享的内存区域，存放所有实例变量（堆）
  - 工作内存（Working Memory）：每个线程私有，存放主内存中变量的副本（类似 CPU 缓存）
  - 线程从主内存读取变量副本到工作内存中，在线程内使用或修改，最终刷新回主内存。线程之间不能直接访问彼此的工作内存，只能通过主内存进行通信。
- Happens-Before 原则
  - 程序顺序规则：单线程内，前面的操作先于后面的操作
  - 锁规则：一个 unlock 操作先于之后对同一个锁的 lock
  - volatile 规则：写入一个 volatile 变量先于后面对它的读取（使用volatile需要满足：对volatile变量的修改不依赖于volatile变量的原值）
  - 线程启动规则：Thread.start() 先于子线程中任何操作
  - 线程终止规则：Thread.join() 先于主线程检测子线程状态或返回值
  - 传递性规则：A happens-before B，B happens-before C ⇒ A happens-before C
- 内存屏障(在Java中，内存屏障作用在volatile 修饰变量读写时，synchronized 块进入/退出时，final 变量在构造函数安全发布时)
  - Load Barrier：读屏障：防止读操作乱序执行
  - Store Barrier：写屏障：保证写入操作不会被 CPU 重排
  - Full Barrier：读写全部屏障：保障读写顺序性
- final、synchronized、volatile 的内存语义
  - volatile：禁止指令重排序（保证有序性），保证可见性，但不保证原子性。
  - synchronized：保证可见性、原子性，有序性，本质上基于 JVM 的 Monitor（对象监视器）。
  - final：构造函数安全发布：只要对象在构造完成前未被其他线程“逸出”，final 修饰的字段是可见的。不能修改，但引用类型不能保证其内部状态不可变（需要额外措施）。

---

## 二、线程创建与启动

### 2.1 继承 Thread 类

- 重写 `run()` 方法
- 调用 `start()` 启动线程

```java

public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}
```

### 2.2 实现 Runnable 接口

- 实现 `run()` 方法
- 使用 `Thread` 构造器启动

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable is running");
    }
}
```

### 2.3 实现 Callable 接口 + FutureTask

- 与 Runnable 的区别：有返回值、可抛出异常
- 使用 `FutureTask` 获取结果

```java
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "Callable is running";
    }
}
```

### 2.4 线程池方式

> 避免频繁创建/销毁线程，降低系统开销；
> 控制并发线程数量，避免过载；
> 提供任务队列与拒绝策略，增强系统弹性；
> 支持 任务调度、延迟执行、周期性执行。

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
                              }

        ExecutorService executor = new ThreadPoolExecutor(
            2, 4, 60, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(10),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy()
        );

        executor.execute(() -> {});

        executor.shutdown(); // 设置线程池的状态为SHUTDOWN，然后中断所有没有正在执行任务的线程
        executor.shutdownNow(); // 设置线程池的状态为 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表
    ```

---

## 三、线程生命周期与状态转换

### 3.1 六种状态

- NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED
  - 新建（New）

  - 就绪（Runnable）

  - 运行（Running）

  - 阻塞（Blocked）

  - 等待（Waiting） / 超时等待（Timed Waiting）

  - 终止（Terminated）

  - Thread.join() 和 Thread.sleep() 的作用与区别
  
  - `New` new Thread(() -> {}) 初始化线程 -> `Runnable` start() 启动线程进入可运行状态 ->
    - `Running` 执行状态                                                        ->
    - `Ready`   未获取到cpu资源，进入就绪状态                                     ->
  - `Blocked` 未获取到锁，进入阻塞状态                                            -> 获取到锁后，进入`Runnable`状态获取cpu资源
  - `Waiting` 调用 Object.wait() Thread.join() LockSupport.park() 进入等待状态   -> notify()/notifyAll() 唤醒 或 join()的任务执行结束或中断
  - `Timed Waiting` 调用 Object.wait(long) Thread.sleep(long) LockSupport.parkNanos(long) 进入超时等待状态 -> notify()/notifyAll() 唤醒 或 join()的任务执行结束或中断
  - `Terminated` 线程执行结束，进入终止状态

### 3.2 状态转换图

    ![多线程状态转换](/assets/java多线程.png "多线程状态转换")


### 3.3 方法对状态的影响

方法 | 状态变化 | 额外注意事项  
--- | --- | ---  
start() | NEW ➔ RUNNABLE | 启动新线程  
run() | - | 普通方法调用    
yield() | RUNNABLE ➔ RUNNABLE | 主动让出CPU    
sleep() | RUNNABLE ➔ TIMED_WAITING | 不释放锁，时间到自动就绪  
join() | RUNNABLE ➔ WAITING/TIMED_WAITING | 当前线程等待另一个线程结束 
wait() | RUNNABLE ➔ WAITING/TIMED_WAITING | 释放锁，等待被唤醒 
notify() | WAITING ➔ BLOCKED | 唤醒一个，抢锁  
notifyAll() | WAITING ➔ BLOCKED | 唤醒全部，抢锁  

- `start()`
  - 状态变化：`New` -> `Runnable`
  - 作用：启动线程，进入可运行状态。
  - 仅只能调用一次。
  - start() 是异步的：调用后线程什么时候真正开始执行，由操作系统线程调度器决定。
- `run()`
  - 直接调用run()方法，并不会启动线程，仅是普通方法调用
- `yield()`  
  - 作用：线程主动让出 CPU 时间片，重新进入就绪（`Runnable`）状态，让调度器重新选择线程运行。
  - 不释放锁。
  - 不保证让出的线程一定不会被立即再次调度。
- `sleep(time)`
  - 状态变化：`RUNNABLE` -> `TIMED_WAITING`
  - 作用：当前线程休眠 time 毫秒，在休眠期间不会竞争 CPU。
  - 不会释放锁。
  - 时间到后自动回到就绪（`Runnable`）状态，等待 CPU 调度。
- `join()`  
  - 作用：让当前线程等待调用`join()` 的线程执行结束。
  - 状态变化：用 `join()` 的线程 ➔ 进入 `WAITING`（如果指定了超时时间进入 `TIMED_WAITING`）。
- `wait()`
  - 作用：让线程在某个对象锁上等待。
  - 状态变化：`RUNNABLE` ➔ `WAITING`（如果指定超时时间 `TIMED_WAITING`）
  - 必须在 synchronized 块内调用，否则抛异常。
  - 调用 wait() 会 释放当前持有的锁，允许其他线程获得锁资源。
  - 等待期间，线程休眠，直到被 notify() 或 notifyAll() 唤醒。
- `notify()`
  - 作用：唤醒在同一个锁对象上等待的一个线程（随机选一个）。
  - 状态变化：被唤醒的线程从 `WAITING` ➔ `BLOCKED`（重新去竞争锁）
  - 调用 notify() 的线程仍然持有锁，唤醒的线程需要等待它释放锁才能继续运行。
- `notifyAll()`
  - 作用：唤醒在同一个锁对象上等待的所有线程。
  - 状态变化：所有被唤醒的线程从 `WAITING` ➔ `BLOCKED`
  - 这些线程会一起去竞争锁，胜出的线程先继续执行，其他线程继续阻塞等待。

---

## 四、线程同步与锁机制

### 4.1 线程安全问题

> 当多个线程同时访问同一个对象时，如果不用额外的同步措施，程序仍然能表现出正确的行为，那么这个对象是线程安全的。否则就存在线程安全问题。

- 竞态条件
  - 定义：当多个线程访问共享数据，并且至少有一个线程在修改数据，而且修改的执行顺序影响结果。
  - 触发条件：并发访问、修改共享数据、执行顺序影响最终结果。
  - 示例：

  ```java
  Thread 1: 读取 count = 5
  Thread 2: 读取 count = 5
  Thread 1: 计算 5 + 1 = 6
  Thread 2: 计算 5 + 1 = 6
  Thread 1: 写回 6
  Thread 2: 写回 6（覆盖了 Thread 1 的更新！）
  ```
  
- 可见性问题
  - 定义：当一个线程修改了共享变量的值，其他线程没有立即看到这个变化，就产生了可见性问题。
  - 解决方法：
    - volatile 关键字：保证修改立即同步到主内存，并让其他线程立刻读到。
    - 加锁（比如 synchronized）：隐式保证可见性。
  - 示例：

```java
// 如果 flag 不是 volatile，Thread 2 可能永远看不到 flag = true，一直死循环！
// Java 的线程可能把变量的值缓存到 CPU 寄存器或者工作内存中，不会每次都去主内存读取！
// 缓存值和主内存值不同步，就产生了可见性问题。
volatile boolean flag = false;

Thread 1:
    flag = true; // 修改flag

Thread 2:
    while (!flag) {
        // 做事
    }
```

### 4.2 synchronized 关键字（隐式锁）

特性 | 说明
--- | ---
加锁对象 | 任意 Java 对象（Object）
加锁范围 | 方法（实例方法或静态方法）、代码块
锁类型 | 对象锁 or 类锁
锁粒度 | 粗（整个方法或指定块）
可重入性 | 支持（同一个线程可以重复获得锁）
阻塞方式 | 阻塞式（非公平），等待获得锁

- 由`JVM`自动加锁、释放锁。
- 隐式的可重入锁
- synchronized 的加锁方式
  - 方法所(同步方法)
    - 实例方法加锁：锁的是当前实例对象（this）。
    - 静态方法加锁：锁的是当前类的 Class 对象（class字节码对象）。

    ```java
    // 锁的是当前对象 this
    public synchronized void instanceMethod() {
        // 临界区
    }

    // 锁的是当前类的 Class 对象
    public static synchronized void staticMethod() {
        // 临界区
    }

    ```

  - 代码块锁(同步代码块)
    - 显式指定要锁定的对象，可以更细粒度控制同步范围，提高并发性。
    - 锁对象可以是：
      - this（当前对象）
      - 自定义的 Object 对象（如：private final Object lock = new Object();）
      - Class 对象（synchronized (MyClass.class) {}）

    ```java
    public void method() {
        synchronized (this) {
            // 临界区
        }
    }

    public void method2(Object lock) {
        synchronized (lock) {
            // 临界区
        }
    }
    ```

- 类锁 vs 对象锁

比较点 | 类锁（静态方法或 MyClass.class） | 对象锁（实例方法或 this）
--- | --- | ---
锁的对象 | 类的 Class 对象 | 具体实例对象
作用范围 | 所有实例共享一个锁 | 每个实例有自己的锁
是否互相影响 | 影响到所有对象 | 仅影响自己这一个对象
用法场景 | 需要所有实例同步，如全局资源同步 | 只需同步单个实例资源

### 4.3 volatile 关键字

- 保证可见性，但不保证原子性：一个线程修改后，其他线程可以立即看到最新值。
- 禁止指令重排：确保代码执行顺序符合预期，避免因 CPU/编译器优化导致的问题。
  - 示例：双重检查锁

  ```java
  public class Singleton {
      private static volatile Singleton instance;

      public static Singleton getInstance() {
          if (instance == null) {           // 第一次检查
              synchronized (Singleton.class) {
                  if (instance == null) {    // 第二次检查
                      instance = new Singleton(); // 分为3步：1. 分配内存 2. 初始化 3. 引用赋值
                  }
              }
          }
          return instance;
      }
  }
  ```

- 使用场景与局限：volatile 不能保证原子性！
  - 示例：虽然可以保证读写可见性，但多个操作之间仍然可能被中断。比如，i++ 其实是三步操作（读 -> 加 -> 写），volatile无法保证它们是一个整体执行的。
  - 解决方式：使用 加锁（如 `synchronized`）或者使用 原子类（如 `AtomicInteger`）

  ```java
  volatile int count = 0;

  public void increment() {
      count++;
  }

  ```

- （内存语义、适用场景）
  - 适用的场景
    - 状态标志变量（如停止线程标志 stop）
    - 单例模式中的 DCL（双重检查锁）
    - 发布和初始化简单对象（不涉及复杂依赖）
  - 不适用的场景
    - 需要保证复合操作原子性的地方（比如 ++, --, add 等操作）
    - 涉及到一致性要求高的多步骤复杂操作
  
  - volatile 通过在读写时插入内存屏障，强制 CPU 或编译器在关键位置同步主存和禁止乱序执行，从而实现可见性和有序性，但不保证操作的原子性。
    - 写 volatile：会在写操作前插入 StoreStore  写后插入 StoreLoad 屏障
    - 读 volatile：会在读操作后插入 LoadLoad  读后插入 LoadStore 屏障
  
屏障类型 | 说明 | 作用
---|---|---
StoreStore Barrier | 保证前面的普通写操作在 volatile 写之前完成 | 写屏障，控制写的顺序
StoreLoad Barrier | 保证volatile 写前的所有读写操作，对后续读写操作可见 | 非常重，既控制读也控制写
LoadLoad Barrier | 保证volatile 读前的所有普通读操作已经完成 | 读屏障，控制读的顺序
LoadStore Barrier | 保证volatile 读前的所有普通写操作已经完成 | 防止写乱序

### 4.4 显式锁机制（JUC 锁）

> 指程序员手动控制加锁与释放锁，通常通过 JUC（java.util.concurrent 包）提供的锁对象，比如 ReentrantLock、ReadWriteLock 等。

- `ReentrantLock`
  - ReentrantLock 是一种可重入、独占的显式锁。
  - 支持公平锁和非公平锁（默认非公平）。
  - 支持中断锁获取、限时锁获取。
  - 需要手动加锁、手动释放锁。
  - 特性

  特性 | 说明
  ---|---
  可重入性 | 同一线程可以多次获得同一把锁，类似 synchronized
  公平性 | new ReentrantLock(true) 创建公平锁，按照请求锁的顺序分配锁
  可中断 | 线程在等待锁时可以被中断（lockInterruptibly()）
  超时获取锁 | tryLock(long time, TimeUnit unit)
  条件变量支持 | 可以细粒度地控制线程等待/通知（Condition）
  - 示例：

  ```java
  Lock lock = new ReentrantLock();
  lock.lock();  // 加锁
  try {
      // 临界区代码
  } finally {
      lock.unlock();  // 释放锁，必须在 finally 块中！
  }

  ```

- `ReadWriteLock`
  - ReadWriteLock 把锁拆成了读锁和写锁两种模式。
  - 读读共享，读写互斥，写写互斥。
  - 读锁（shared）：多个线程可以同时持有。
  - 写锁（exclusive）：同一时间只允许一个线程持有，且阻止其他读写线程。
  - 可重入锁

  ```java
  ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
  Lock readLock = rwLock.readLock();
  Lock writeLock = rwLock.writeLock();

  // 读
  readLock.lock();
  try {
      // 读取共享资源
  } finally {
      readLock.unlock();
  }

  // 写
  writeLock.lock();
  try {
      // 修改共享资源
  } finally {
      writeLock.unlock();
  }

  ```

- `StampedLock`
  - StampedLock 是 JDK 8 引入的更高性能的读写锁。基于乐观读思想，比传统的读写锁性能更好。
  - 悲观读锁（类似 ReentrantReadWriteLock 的读锁）
  - 写锁
  - 乐观读（Optimistic Read，不加锁）
  - 不可重入锁

  ```java
  StampedLock stampedLock = new StampedLock();
  long stamp = stampedLock.readLock(); // 悲观读
  try {
      // 读取共享资源
  } finally {
      stampedLock.unlockRead(stamp);
  }

  // 乐观读
  long stamp = stampedLock.tryOptimisticRead();
  if (stampedLock.validate(stamp)) {
      // 读取共享资源
  }
  ```

  特性 | 说明
  ----|---
  支持乐观读 | 尝试读时不加锁，性能高，但需要最后 `validate`
  更细粒度控制 | 通过 `stamp`（邮戳）标识锁的状态
  不支持重入 | 不像 `ReentrantLock` 可重入，`StampedLock` 不是可重入锁

- `ReentrantLock` 适合一般场景，`ReadWriteLock` 适合读多写少，`StampedLock` 适合极端读多写少、追求性能极限的场景。

### 4.5 原子类（Atomic）

> 在并发编程中，多线程修改共享变量时，容易出现竞态条件问题（数据不一致）。
> synchronized 可以保证线程安全，但性能开销较大（阻塞+唤醒）。
> Java 并发包（java.util.concurrent.atomic）提供了轻量级的原子操作类，利用硬件层面的原子指令（如 CAS），实现更高效的线程安全操作。

- 原子类的基本原理：核心机制是 CAS 比较并交换操作（Compare-And-Swap）
  - 读取内存中的旧值。
  - 将旧值与期望值比较。
  - 如果一致，则更新为新值；否则不更新，并可以重试
- 底层实现：
  - 通过 CPU 提供的原子指令（如 CMPXCHG）。
  - 在 Java 中由 Unsafe 类提供的 compareAndSwapXXX 方法封装。
  
  ```java
  if (内存中的值 == 期望值) {
      更新为新值
  } else {
      失败，可能重试
  }
  ```

- `AtomicInteger`
  - get() 获取当前值
  - set(int newValue) 设置新值
  - incrementAndGet() 自增+1
  - decrementAndGet() 自减-1
  - addAndGet(int delta) 增加指定数值
  - compareAndSet(int expect, int update) CAS操作

```java
AtomicInteger atomicInteger = new AtomicInteger(0);
atomicInteger.incrementAndGet(); // 自增，变为1
atomicInteger.addAndGet(5);       // 加5，变为6
boolean success = atomicInteger.compareAndSet(6, 10); // 如果当前是6，则更新为10
```

- `AtomicReference`
  - 可以保证对象引用的原子性更新。
  - 适用于多个线程操作对象指针场景。
  - 只是保证引用本身的原子性，不保证对象内部字段的线程安全！

```java
AtomicReference<String> atomicReference = new AtomicReference<>("Hello");
atomicReference.compareAndSet("Hello", "World");
```

- `LongAdder`
  - 为什么需要 LongAdder？
    - 在高并发场景下，AtomicInteger/AtomicLong的单点CAS竞争严重，导致性能下降。
    - LongAdder 通过分段累加，降低了竞争，提高了吞吐量。
  - LongAdder 的原理
    - 内部维护了一个Cell 数组，每个 Cell 里存一部分值。
    - 线程更新时，先定位到一个 Cell，仅在本地 Cell 上 CAS 更新，减少冲突。
    - 取总值时，把所有 Cell 的值加起来。

```java
LongAdder longAdder = new LongAdder();
longAdder.increment(); // 累加1
longAdder.add(5);      // 累加5
long value = longAdder.sum(); // 获取总和
```

- 原子类使用场景

| 需求 | 推荐使用   |
---|---
|   少量线程更新，竞争小 | AtomicInteger, AtomicLong
|   高并发场景，频繁累加 | LongAdder    |
|   更新引用（对象指针） | AtomicReference  |
|   更新字段且包含逻辑校验（如版本号） | AtomicReference + 版本管理 |

### 4.6 公平锁与非公平锁

- 公平锁
  - 线程按照请求锁的顺序来获得锁。
  - 先来先服务，排队机制，不会插队。
  - 保证每个线程最终都能拿到锁，避免饿死（Starvation）。
- 非公平锁
  - 线程请求锁时，直接竞争，不管之前有没有线程排队。
  - 有可能出现插队现象（新线程突然获得锁，排队线程继续等待）。
  - 提高吞吐量，因为减少了线程调度带来的切换开销。
- 对比
  - 公平锁维护一个线程队列（排队），线程调度开销大，性能下降。
  - 非公平锁允许“加塞”，新线程如果正好能立刻拿到锁，就减少了等待和唤醒的开销。

-| 公平锁 | 非公平锁
---|---|---
优点 | 保证顺序、公平性 | 提高吞吐量，减少等待
缺点 | 效率较低，可能导致整体性能下降 | 可能导致某些线程长时间得不到锁（饿死）

### 4.7 死锁、活锁与饥饿现象

- 死锁
  - 多个线程互相等待对方持有的锁，形成循环等待，导致所有线程都无法继续执行。
  - 死锁产生的必要条件

  条件 | 解释
  ---|---
  互斥 | 同一资源同一时间只能由一个线程占用
  占有且等待 | 线程持有资源的同时，申请其他资源
  不可抢占 | 已持有的资源不能被强制剥夺
  循环等待 | 多个线程形成资源的循环等待链

- 活锁
  - 线程没有阻塞，而是不断地相互让步，结果都无法继续向前推进。
- 饥饿
  - 某个线程因为长时间得不到资源而无法执行。
  - 原因
    - 高优先级线程总是占用 CPU，低优先级线程一直被抢占。
    - 同步机制不合理（例如使用了非公平锁），导致某些线程很少获得锁。

### 4.8 可重入锁，不可重入锁

- 可重入锁
  - 如果一个线程已经持有了锁，在它再次请求这个锁时，可以成功获取到（而不会被阻塞），这类锁称为可重入锁。
  - 一个线程 自己可以多次获得 同一把锁，每次加锁后都需要对应的释放（unlock）。
  - 锁内部维护了一个计数器（lock count），记录锁被同一个线程加锁的次数。
  
  ```java
  ReentrantLock lock = new ReentrantLock();

  lock.lock();
  try {
      lock.lock(); // 同一线程再次加锁，可以成功
      try {
          // 业务逻辑
      } finally {
          lock.unlock();
      }
  } finally {
      lock.unlock();
  }
  ```

- 不可重入锁
  - 如果一个线程已经持有了锁，在它再次请求该锁时，不能直接获得锁（会阻塞或者死锁），这种锁叫做不可重入锁。
  - 同一线程，不能再次进入临界区。
  - 容易造成死锁，特别是递归调用、或者在锁保护的方法里调用另一个锁保护的方法时。

---

## 五、线程间通信

### 5.1 wait/notify机制

- wait/notify 是什么？
  - wait()、notify()、notifyAll()是Object类提供的方法（注意：不是Thread的方法）
  - 用于线程之间通信，协调执行时机
  - 通常配合synchronized锁一起使用
- 使用场景与注意事项
  - 生产者-消费者模式（最典型）
  - 线程间协作：一个线程等待条件成立，另一个线程负责通知
  - 实现有限资源的同步访问
  - 本质：一个线程在等待某个条件发生，另一个线程在条件满足后发出通知。
- 基本使用规则

条目 | 内容
---|---
必须持有对象锁 | 必须在synchronized(obj){}内部调用wait/notify，否则抛IllegalMonitorStateException
wait释放锁 | wait()调用后线程立即释放锁，进入等待队列
notify不释放锁 | notify()调用只是通知，锁要等synchronized代码块执行完后才释放
唤醒多个线程 | 用notifyAll()，防止遗漏
避免假唤醒 | wait()前后用while循环检查条件而不是if

- 实现生产者-消费者模式

```java
class Storage {
    private int product = 0;

    public synchronized void produce() throws InterruptedException {
        while (product >= 1) {  // 仓库已满，生产者等待
            wait();
        }
        product++;
        System.out.println(Thread.currentThread().getName() + " 生产了一个产品。当前库存：" + product);
        notifyAll(); // 通知消费者消费
    }

    public synchronized void consume() throws InterruptedException {
        while (product <= 0) {  // 仓库为空，消费者等待
            wait();
        }
        product--;
        System.out.println(Thread.currentThread().getName() + " 消费了一个产品。当前库存：" + product);
        notifyAll(); // 通知生产者生产
    }
}

public class ProducerConsumerTest {
    public static void main(String[] args) {
        Storage storage = new Storage();

        // 生产者线程
        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    storage.produce();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "生产者").start();

        // 消费者线程
        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    storage.consume();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "消费者").start();
    }
}
```

注意点 | 解释
---|---
wait、notify必须在synchronized块内调用 | 必须持有对象锁
使用while检查条件 | 防止虚假唤醒（spurious wakeup）
notify只唤醒一个线程，notifyAll唤醒所有 | 大多数场景推荐notifyAll()
wait会释放锁，notify不会立即释放锁 | notify后需等待退出synchronized

### 5.2 Condition 接口

> Condition是java.util.concurrent.locks.Condition接口，配合Lock（通常是ReentrantLock）使用。
> 类似于 Object 的 wait()、notify()、notifyAll()，但功能更强大、更加灵活。
> 可以让不同的线程在不同的“等待队列”中等待（而synchronized+wait/notify只能共用一条等待队列）。

- 为什么要用 Condition？
  - 可以创建多个等待队列（支持多个 Condition 对象）
  - 可以精确唤醒（指定哪个条件的线程被唤醒）
  - 配合Lock使用，能更加细粒度控制线程通信
  - 避免全部唤醒，提升效率
- 常用方法

方法 | 作用
---|---
await() | 当前线程等待，同时释放锁
signal() | 唤醒一个等待线程
signalAll() | 唤醒所有等待线程

- 多条件队列的实现

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Storage {
    private int product = 0;
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();  // 生产者等待队列
    private final Condition notEmpty = lock.newCondition(); // 消费者等待队列

    public void produce() throws InterruptedException {
        lock.lock();
        try {
            while (product >= 1) { // 仓库已满
                notFull.await();
            }
            product++;
            System.out.println(Thread.currentThread().getName() + " 生产了一个产品，库存：" + product);
            notEmpty.signal(); // 唤醒消费者
        } finally {
            lock.unlock();
        }
    }

    public void consume() throws InterruptedException {
        lock.lock();
        try {
            while (product <= 0) { // 仓库为空
                notEmpty.await();
            }
            product--;
            System.out.println(Thread.currentThread().getName() + " 消费了一个产品，库存：" + product);
            notFull.signal(); // 唤醒生产者
        } finally {
            lock.unlock();
        }
    }
}

public class ProducerConsumerWithCondition {
    public static void main(String[] args) {
        Storage storage = new Storage();

        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    storage.produce();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "生产者").start();

        new Thread(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    storage.consume();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "消费者").start();
    }
}
```

### 5.3 管道通信

> Java 管道是一种特殊的流，用于在线程之间传递数据。它通常由一个输入管道流和一个输出管道流组成。输入管道流用于从一个线程读取数据，而输出管道流用于将数据写入另一个线程。这两个管道流之间的数据传输是单向的，即数据只能从输入流传输到输出流。

- 字节流
  - `PipedInputStream` / `PipedOutputStream`
- 字符流
  - `PipedReader` / `PipedWriter`
- 只能线程间通信，不能单线程使用。
- 一次只能一个线程写，一个线程读，否则可能抛异常或数据错乱。
- 不支持多对多（一写多读 / 多写一读要自己控制同步）
- 管道有内部缓冲区（默认大约 1024 bytes），写太快而读太慢可能造成阻塞。
  - 写入速度太快，缓冲区满了 → write() 会阻塞等待
  - 读取速度太快，缓冲区空了 → read() 会阻塞等待
- 管道本质是双向绑定，下面两种方式实际上并没有区别。
- 使用 `ObjectInputStream` 等对象流包装管道可以达到响应需求。

  ```java
  import java.io.IOException;
  import java.io.PipedReader;
  import java.io.PipedWriter;

  public class PipeCharExample {
      public static void main(String[] args) throws IOException {
          PipedReader reader = new PipedReader();
          PipedWriter writer = new PipedWriter();

          // 连接
          writer.connect(reader);

          // 或
          reader.connect(writer);

      }
  }

  ```

- 使用缓冲流包装管道
  
  ```java
  BufferedReader reader = new BufferedReader(new InputStreamReader(pipedInputStream));
  BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(pipedOutputStream));
  // 或
  BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);
  BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(outputStream);

    // 线程1：写入数据
  Thread thread1 = new Thread(() -> {
      try {
          bufferedOutputStream.write("Data from Thread 1".getBytes());
          bufferedOutputStream.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
  });

  // 线程2：读取数据
  Thread thread2 = new Thread(() -> {
      try {
          byte[] buffer = new byte[1024];
          int bytesRead = bufferedInputStream.read(buffer);
          String data = new String(buffer, 0, bytesRead);
          System.out.println("Thread 2 received: " + data);
          bufferedInputStream.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
  });

  ```

- 示例：

  ```java
  import java.io.*;
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  import java.util.concurrent.atomic.AtomicInteger;

  /**
   * 管道通信，多生产者、消费者示例。
   */
  public class AdvancedPipeDemo {
      public static void main(String[] args) throws IOException {
          // 管道流
          PipedOutputStream pos = new PipedOutputStream();
          PipedInputStream pis = new PipedInputStream(pos);

          BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(pos));
          BufferedReader reader = new BufferedReader(new InputStreamReader(pis));

          // 线程池管理
          ExecutorService executor = Executors.newFixedThreadPool(5);

          // 生产者数量
          int producerCount = 3;
          // 活跃生产者计数器
          AtomicInteger activeProducers = new AtomicInteger(producerCount);

          // 启动多个生产者
          for (int i = 1; i <= producerCount; i++) {
              executor.execute(new Producer(writer, "Producer-" + i, activeProducers));
          }

          // 启动多个消费者
          int consumerCount = 2;
          for (int i = 1; i <= consumerCount; i++) {
              executor.execute(new Consumer(reader, activeProducers, "Consumer-" + i));
          }

          executor.shutdown();
      }

      static class Producer implements Runnable {
          private final BufferedWriter writer;
          private final String name;
          private final AtomicInteger activeProducers;

          public Producer(BufferedWriter writer, String name, AtomicInteger activeProducers) {
              this.writer = writer;
              this.name = name;
              this.activeProducers = activeProducers;
          }

          @Override
          public void run() {
              try {
                  for (int i = 1; i <= 5; i++) {
                      synchronized (writer) { // 多生产者写入时要同步
                          String message = name + " - Data " + i;
                          writer.write(message + "\n");
                          writer.flush();
                          System.out.println(name + " wrote: " + message);
                      }
                      Thread.sleep((long) (Math.random() * 400)); // 模拟不稳定生产速度
                  }
              } catch (IOException | InterruptedException e) {
                  e.printStackTrace();
              } finally {
                  // 生产者结束时减少活跃数量
                  if (activeProducers.decrementAndGet() == 0) {
                      // 最后一个生产者发送关闭信号
                      try {
                          synchronized (writer) {
                              writer.write("EOF\n");
                              writer.flush();
                          }
                          writer.close();
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
              }
          }
      }

      static class Consumer implements Runnable {
          private final BufferedReader reader;
          private final AtomicInteger activeProducers;
          private final String name;

          public Consumer(BufferedReader reader, String name, AtomicInteger activeProducers) {
              this.reader = reader;
              this.name = name;
              this.activeProducers = activeProducers;
          }

          @Override
          public void run() {
              try {
                  String line;
                  while ((line = reader.readLine()) != null) {
                      if ("EOF".equals(line)) {
                          System.out.println(name + " detected EOF, exiting.");
                          break;
                      }
                      System.out.println(name + " processed: " + line);
                      Thread.sleep((long) (Math.random() * 500)); // 模拟消费速度
                  }
              } catch (IOException | InterruptedException e) {
                  // 注意可能在管道关闭时抛出异常，需要优雅处理
                  System.out.println(name + " encountered exception and exits: " + e.getMessage());
              } finally {
                  try {
                      reader.close(); // 消费者自己关闭流（idempotent，安全）
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
  }
  ```

---

## 六、线程池与并发工具类（JUC）

### 6.1 Executor 框架

- 接口
  - `Executor`: 最基础接口，只有 execute(Runnable command) 方法（负责提交任务）
  - `ExecutorService`：继承 Executor，增加了生命周期管理（shutdown、awaitTermination等）+ 提交任务（submit支持Future）
  - `ScheduledExecutorService`: 定时任务线程池接口（可以延迟执行、周期性执行任务）
- 实现类：
  - `Executors`: 工厂工具类，用来快速创建各种常用线程池
  - `ThreadPoolExecutor`: 最核心、最强大的线程池实现（通常推荐直接用它）
  - `ScheduledThreadPoolExecutor`: 定时/周期性任务专用线程池
  - `FixedThreadPool`: 固定大小线程池,适合任务量稳定的场景
  - `CachedThreadPool`: 缓存线程池、自动扩容，适合任务量突发型、短时任务
  - `SingleThreadExecutor`: 单线程线程池,保证顺序执行任务
  - `ScheduledThreadPool`: 定时任务线程池,支持延迟、定时

### 6.2 ThreadPoolExecutor 参数详解

- `ThreadPoolExecutor`(推荐)
  - 线程数设置依据：
    - `CPU` 密集型(任务主要消耗 CPU 资源（如计算、压缩、加密）)：CPU核心数 + 1
    - `IO` 密集型(任务主要在执行 IO 操作（如文件读写、数据库、网络请求）)：2 * CPU核心数 或更多
  - 构造方法：
    - `corePoolSize`（必需）：核心线程数。默认情况下，核心线程会一直存活，但是当将 `allowCoreThreadTimeout` 设置为 `true` 时，核心线程也会超时回收。
    - `maximumPoolSize`（必需）：线程池所能容纳的最大线程数。当活跃线程数达到该数值后，后续的新任务将会阻塞。
    - `keepAliveTime`（必需）：线程闲置超时时长。如果超过该时长，非核心线程就会被回收。如果将 `allowCoreThreadTimeout` 设置为 `true` 时，核心线程也会超时回收。
    - `unit`（必需）：指定 `keepAliveTime` 参数的时间单位。常用的有：`TimeUnit.MILLISECONDS`（毫秒）、`TimeUnit.SECONDS`（秒）、`TimeUnit.MINUTES`（分）。
    - `workQueue`（必需）：任务队列。通过线程池的 `execute()` 方法提交的 `Runnable` 对象将存储在该参数中。其采用阻塞队列实现。
      - 任务队列：如果使用有界队列，当队列饱和时并超过最大线程数时就会执行拒绝策略；而如果使用无界队列，因为任务队列永远都可以添加任务，所以设置 maximumPoolSize 没有任何意义。
        - `ArrayBlockingQueue`：一个由数组结构组成的有界阻塞队列（数组结构可配合指针实现一个环形队列）。
        - `LinkedBlockingQueue`： 一个由链表结构组成的有界阻塞队列，在未指明容量时，容量默认为 Integer.MAX_VALUE。
        - `PriorityBlockingQueue`： 一个支持优先级排序的无界阻塞队列，对元素没有要求，可以实现 `Comparable` 接口也可以提供 `Comparator` 来对队列中的元素进行比较。跟时间没有任何关系，仅仅是按照优先级取任务。
        - `DelayQueue`：类似于`PriorityBlockingQueue`，是二叉堆实现的无界优先级阻塞队列。要求元素都实现 Delayed 接口，通过执行时延从队列中提取任务，时间没到任务取不出来。
        - `SynchronousQueue`： 一个不存储元素的阻塞队列，消费者线程调用 `take()` 方法的时候就会发生阻塞，直到有一个生产者线程生产了一个元素，消费者线程就可以拿到这个元素并返回；生产者线程调用 `put()` 方法的时候也会发生阻塞，直到有一个消费者线程消费了一个元素，生产者才会返回。
        - `LinkedBlockingDeque`： 使用双向队列实现的有界双端阻塞队列。双端意味着可以像普通队列一样 `FIFO`（先进先出），也可以像栈一样 `FILO`（先进后出）。
        - `LinkedTransferQueue`： 它是`ConcurrentLinkedQueue`、`LinkedBlockingQueue` 和 `SynchronousQueue` 的结合体，但是把它用在 `ThreadPoolExecutor` 中，和 `LinkedBlockingQueue` 行为一致，但是是无界的阻塞队列。
    - `threadFactory`（可选）：线程工厂。用于指定为线程池创建新线程的方式。
    - `handler`（可选）：拒绝策略。当达到最大线程数时需要执行的饱和策略。
      - 拒绝策略：当线程池的线程数达到最大线程数时，需要执行拒绝策略。拒绝策略需要实现`RejectedExecutionHandler` 接口，并实现 `rejectedExecution(Runnable r, ThreadPoolExecutor executor) `方法。
        - `AbortPolicy`（默认）：丢弃任务并抛出 `RejectedExecutionException` 异常。
        - `CallerRunsPolicy`：由调用线程处理该任务。
        - `DiscardPolicy`：丢弃任务，但是不抛出异常。可以配合这种模式进行自定义的处理方式。
        - `DiscardOldestPolicy`：丢弃队列最早的未处理任务，然后重新尝试执行任务。

- `ThreadPoolExecutor` 示例

  ```java
  import java.util.concurrent.*;
  import java.util.concurrent.atomic.AtomicInteger;

  /**
   * 线程池工厂 - 生产级写法
   */
  public class CustomThreadPoolFactory {

      /**
       * 创建通用线程池
       *
       * @param poolName        线程池名称前缀
       * @param corePoolSize    核心线程数
       * @param maxPoolSize     最大线程数
       * @param queueCapacity   队列容量
       * @param keepAliveSecond 空闲线程存活时间（秒）
       * @return 自定义线程池
       */
      public static ThreadPoolExecutor createThreadPool(String poolName,
                                                        int corePoolSize,
                                                        int maxPoolSize,
                                                        int queueCapacity,
                                                        int keepAliveSecond) {
          return new ThreadPoolExecutor(
                  corePoolSize,
                  maxPoolSize,
                  keepAliveSecond,
                  TimeUnit.SECONDS,
                  new LinkedBlockingQueue<>(queueCapacity),
                  new NamedThreadFactory(poolName),
                  new CustomRejectedExecutionHandler(poolName)
          );
      }

      /**
       * 自定义线程工厂 - 给线程起名字
       */
      static class NamedThreadFactory implements ThreadFactory {
          private final AtomicInteger threadNumber = new AtomicInteger(1);
          private final String namePrefix;

          NamedThreadFactory(String namePrefix) {
              this.namePrefix = namePrefix + "-pool-";
          }

          @Override
          public Thread newThread(Runnable r) {
              Thread t = new Thread(r, namePrefix + threadNumber.getAndIncrement());
              if (t.isDaemon()) {
                  t.setDaemon(false);
              }
              if (t.getPriority() != Thread.NORM_PRIORITY) {
                  t.setPriority(Thread.NORM_PRIORITY);
              }
              return t;
          }
      }

      /**
       * 自定义拒绝策略 - 打日志、报警
       */
      static class CustomRejectedExecutionHandler implements RejectedExecutionHandler {
          private final String poolName;

          CustomRejectedExecutionHandler(String poolName) {
              this.poolName = poolName;
          }

          @Override
          public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
              System.err.println("【线程池拒绝执行】线程池：" + poolName
                      + " 当前活跃线程数：" + executor.getActiveCount()
                      + " 队列大小：" + executor.getQueue().size());
              // 这里可以扩展，比如发送报警通知
              throw new RejectedExecutionException("任务被线程池拒绝执行：" + poolName);
          }
      }
  }
  ```

### 6.3 常见线程池类型,使用 `Executors` 工具类(不推荐)

- 定长线程池（`FixedThreadPool`）
  - 特点：只有核心线程，线程数量固定，执行完立即回收，任务队列为链表结构的有界队列。
  - 应用场景：控制线程最大并发数。
- 定时线程池（`ScheduledThreadPool`）
  - 特点：核心线程数量固定，非核心线程数量无限，执行完闲置 10ms 后回收，任务队列为延时阻塞队列。
  - 应用场景：执行定时或周期性的任务。
- 可缓存线程池（`CachedThreadPool`）
  - 特点：无核心线程，非核心线程数量无限，执行完闲置 60s 后回收，任务队列为不存储元素的阻塞队列。
  - 应用场景：执行大量、耗时少的任务。
- 单线程化线程池（`SingleThreadExecutor`）
  - 特点：只有 1 个核心线程，无非核心线程，执行完立即回收，任务队列为链表结构的有界队列。
  - 应用场景：不适合并发但可能引起 IO 阻塞性及影响 UI 线程响应的操作，如数据库操作、文件操作等。
- 不推荐使用 `Executors` 工具类的原因
  - `FixedThreadPool` 和 `SingleThreadExecutor`：主要问题是堆积的请求处理队列均采用 `LinkedBlockingQueue`(无界)，可能会耗费非常大的内存，甚至 OOM。
  - `CachedThreadPool` 和 `ScheduledThreadPool`：主要问题是线程数最大数是 Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至 OOM。

### 6.4 Fork/Join 框架

> `Fork/Join` 是 `Java7` 引入的一个 高效并行计算框架。主要目标是：把大任务拆成小任务并行执行，最后合并结果。
> 适合需要 递归划分 的任务。任务能拆分成很多小块，可以并发处理。

- 工作窃取算法
  - 核心概念：
    - 每个线程维护一个双端队列（`Deque`）。
    - 线程自己总是从 头部（栈顶）取任务执行。
    - 如果自己空闲了，会从其他线程的尾部偷任务来做！（`Work Stealing`）
  - 好处：
    - 减少线程间争抢，提升 CPU 利用率。
    - 动态负载均衡，避免有的线程闲着，有的线程很忙。

- `ForkJoinPool`
  - 线程池，专门管理 `Fork/Join` 任务
- `ForkJoinTask<V>`
  - 抽象任务类，支持 `fork` 和 `join`
- `RecursiveTask`
  - 有返回值的任务（常用）
- `RecursiveAction`
  - 无返回值的任务

- 示例：查找文件

  ```java
  import java.io.File;
  import java.util.ArrayList;
  import java.util.List;
  import java.util.concurrent.RecursiveTask;

  public class FileSearchTask extends RecursiveTask<List<File>> {

      private static final int THRESHOLD = 10; // 子目录超过多少就分割
      private File directory;
      private String keyword;

      public FileSearchTask(File directory, String keyword) {
          this.directory = directory;
          this.keyword = keyword;
      }

      @Override
      protected List<File> compute() {
          List<File> result = new ArrayList<>();
          List<FileSearchTask> subTasks = new ArrayList<>();

          File[] files = directory.listFiles();
          if (files != null) {
              List<File> dirs = new ArrayList<>();

              for (File file : files) {
                  if (file.isDirectory()) {
                      dirs.add(file);
                  } else {
                      if (file.getName().contains(keyword)) {
                          result.add(file);
                      }
                  }
              }

              if (dirs.size() > THRESHOLD) {
                  // 目录太多，拆分子任务
                  for (File dir : dirs) {
                      FileSearchTask task = new FileSearchTask(dir, keyword);
                      task.fork();
                      subTasks.add(task);
                  }

                  for (FileSearchTask task : subTasks) {
                      result.addAll(task.join());
                  }
              } else {
                  // 目录少，直接递归
                  for (File dir : dirs) {
                      result.addAll(new FileSearchTask(dir, keyword).compute());
                  }
              }
          }

          return result;
      }
  }

  import java.io.File;
  import java.util.List;
  import java.util.concurrent.ForkJoinPool;

  public class FileSearchExample {
      public static void main(String[] args) {
          // 要搜索的根目录
          File rootDir = new File("D:/BigDataFolder"); 

          // 关键词
          String keyword = "report";

          ForkJoinPool pool = new ForkJoinPool();

          FileSearchTask task = new FileSearchTask(rootDir, keyword);

          long start = System.currentTimeMillis();

          List<File> matchingFiles = pool.invoke(task);

          long end = System.currentTimeMillis();

          System.out.println("找到 " + matchingFiles.size() + " 个匹配文件");
          for (File file : matchingFiles) {
              System.out.println(file.getAbsolutePath());
          }

          System.out.println("搜索耗时: " + (end - start) + " ms");

          pool.shutdown();
      }
  }

  ```

- 示例：组合 `CompletableFuture` 异步访问远程API

  ```java
  import java.util.ArrayList;
  import java.util.List;
  import java.util.concurrent.*;

  public class ForkJoinWithFutureTask extends RecursiveTask<List<String>> {

      private static final int THRESHOLD = 1000;
      private List<String> data;
      private int start;
      private int end;

      private ExecutorService httpExecutor = Executors.newCachedThreadPool();

      public ForkJoinWithFutureTask(List<String> data, int start, int end) {
          this.data = data;
          this.start = start;
          this.end = end;
      }

      @Override
      protected List<String> compute() {
          if (end - start <= THRESHOLD) {
              List<CompletableFuture<String>> futures = new ArrayList<>();
              for (int i = start; i < end; i++) {
                  String item = data.get(i);
                  // 每个数据提交异步处理
                  CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> remoteCall(item), httpExecutor);
                  futures.add(future);
              }

              // 收集所有结果
              List<String> results = new ArrayList<>();
              for (CompletableFuture<String> future : futures) {
                  results.add(future.join()); // 阻塞等待
              }
              return results;
          } else {
              int mid = (start + end) / 2;
              ForkJoinWithFutureTask left = new ForkJoinWithFutureTask(data, start, mid);
              ForkJoinWithFutureTask right = new ForkJoinWithFutureTask(data, mid, end);

              left.fork();
              List<String> rightResult = right.compute();
              List<String> leftResult = left.join();

              List<String> total = new ArrayList<>();
              total.addAll(leftResult);
              total.addAll(rightResult);
              return total;
          }
      }

      private String remoteCall(String input) {
          // 模拟远程API调用
          try {
              Thread.sleep(100); // 假设每个远程请求耗时100ms
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
          }
          return "Processed-" + input;
      }
  }

  import java.util.ArrayList;
  import java.util.List;
  import java.util.concurrent.ForkJoinPool;

  public class ForkJoinWithFutureExample {
      public static void main(String[] args) {
          List<String> dataList = new ArrayList<>();
          for (int i = 0; i < 10_000; i++) {
              dataList.add("Item-" + i);
          }

          ForkJoinPool pool = new ForkJoinPool();

          ForkJoinWithFutureTask task = new ForkJoinWithFutureTask(dataList, 0, dataList.size());

          long start = System.currentTimeMillis();

          List<String> result = pool.invoke(task);

          long end = System.currentTimeMillis();

          System.out.println("处理完成, 总数: " + result.size());
          System.out.println("耗时: " + (end - start) + " ms");

          pool.shutdown();
      }
  }
  ```

### 6.5 并发辅助工具类

- `CountDownLatch`
  - 用途：一组线程等待其他线程完成某件事情后再继续执行。
  - 特点：一次性，计数器只能递减到0，不能重置。
  - 当前线程阻塞等待子线程执行完毕，直到计数器为0或超时，才继续执行。

  ```java
  import java.util.concurrent.CountDownLatch;

  public class CountDownLatchExample {
      public static void main(String[] args) throws InterruptedException {
          int threadCount = 5;
          CountDownLatch latch = new CountDownLatch(threadCount);

          for (int i = 0; i < threadCount; i++) {
              new Thread(() -> {
                  System.out.println(Thread.currentThread().getName() + " 执行任务完成");
                  latch.countDown(); // 任务完成，计数器减1
              }).start();
          }

          latch.await(); // 等待所有线程完成
          System.out.println("所有线程执行完毕，主线程继续！");
      }
  }
  ```

- `CyclicBarrier`
  - 用途：让一组线程互相等待，到达屏障点后一起继续执行。
  - 特点：可以循环使用，计数器可以重置。

  ```java
  import java.util.concurrent.BrokenBarrierException;
  import java.util.concurrent.CyclicBarrier;

  public class CyclicBarrierExample {
      public static void main(String[] args) {
          int threadCount = 3;
          CyclicBarrier barrier = new CyclicBarrier(threadCount, () -> {
              System.out.println("所有线程到达屏障，统一开始下一阶段任务！");
          });

          for (int i = 0; i < threadCount; i++) {
              new Thread(() -> {
                  try {
                      System.out.println(Thread.currentThread().getName() + " 准备完毕，等待其他人...");
                      barrier.await(); // 到达屏障点，等待
                      System.out.println(Thread.currentThread().getName() + " 开始执行后续任务！");
                  } catch (InterruptedException | BrokenBarrierException e) {
                      e.printStackTrace();
                  }
              }).start();
          }
      }
  }

  ```

- `Semaphore`
  - 用途：控制同时访问的线程数量。
  - 特点：信号量可以控制资源数量，获取/释放。

  ```java
  import java.util.concurrent.Semaphore;

  public class SemaphoreExample {
      public static void main(String[] args) {
          Semaphore semaphore = new Semaphore(3); // 允许最多3个线程同时访问

          for (int i = 0; i < 10; i++) {
              new Thread(() -> {
                  try {
                      semaphore.acquire(); // 获取许可
                      System.out.println(Thread.currentThread().getName() + " 获得许可，正在执行...");
                      Thread.sleep(2000); // 模拟任务执行
                      System.out.println(Thread.currentThread().getName() + " 释放许可");
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
                      semaphore.release(); // 释放许可
                  }
              }).start();
          }
      }
  }

  ```

- `Exchanger`
  - 用途：两个线程之间交换数据。
  - 特点：一对一交换，双方到达后交换数据。

  ```java
  import java.util.concurrent.Exchanger;

  public class ExchangerExample {
      public static void main(String[] args) {
          Exchanger<String> exchanger = new Exchanger<>();

          new Thread(() -> {
              try {
                  String data = "数据A";
                  System.out.println("线程1准备交换：" + data);
                  String result = exchanger.exchange(data);
                  System.out.println("线程1交换得到：" + result);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }).start();

          new Thread(() -> {
              try {
                  String data = "数据B";
                  System.out.println("线程2准备交换：" + data);
                  String result = exchanger.exchange(data);
                  System.out.println("线程2交换得到：" + result);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }).start();
      }
  }
  ```

- `Phaser`
  - 用途：比 CyclicBarrier 更强大的阶段同步器，支持动态增加/注销参与线程。
  - 特点：多阶段、多批次。

  ```java
  import java.util.concurrent.Phaser;

  public class PhaserExample {
      public static void main(String[] args) {
          Phaser phaser = new Phaser(3); // 3个参与者

          for (int i = 0; i < 3; i++) {
              int finalI = i;
              new Thread(() -> {
                  System.out.println("线程" + finalI + " 完成第1阶段任务");
                  phaser.arriveAndAwaitAdvance(); // 第一阶段
                  System.out.println("线程" + finalI + " 完成第2阶段任务");
                  phaser.arriveAndAwaitAdvance(); // 第二阶段
              }).start();
          }
      }
  }
  ```

- 总结：

工具类 | 主要功能 | 特点
---|---|---
`CountDownLatch` | 等待其他线程完成 | 一次性，计数器不能重置
`CyclicBarrier` | 多线程到达屏障一起出发 | 可循环重用
`Semaphore` | 控制并发访问数量 | 获取/释放许可
`Exchanger` | 线程间交换数据 | 成对出现
`Phaser` | 多阶段同步器 | 动态注册/注销，支持多阶段

---

## 七、并发集合

### 7.1 Map

- `ConcurrentHashMap`
  - `JDK7`：分段锁机制
  - `JDK8`：`CAS` + 链表/红黑树优化
  - `ConcurrentHashMap（分段锁`/JDK8优化）
  - 核心数据结构
    - `val` 和 `next` 都是 `volatile` 保证可见性
    - `CAS` + `synchronized` 控制并发

    ```java
    // Node 是基础结构
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash; // 缓存 key 的哈希值，加快定位
        final K key; 
        volatile V val;  // value
        volatile Node<K,V> next; // 链表结构
    }
    ```

  - 核心成员变量

    ```java
    // 真正存储数据的数组
    transient volatile Node<K,V>[] table;

    // 扩容时使用的下一张表
    private transient volatile Node<K,V>[] nextTable;

    // 记录大小变化的辅助对象
    private transient volatile long baseCount;

    /**
     * sizeCtl 控制初始化、扩容的阈值 
     * <0：正在扩容，
     * -1 表示某个线程正在初始化表
     * =0：未初始化 0：正常情况下，表示下一次扩容的阈值
     *  */ 
    private transient volatile int sizeCtl;

    // transferIndex 扩容时使用的分段迁移索引
    private transient volatile int transferIndex;
    ```

  - `put` 流程（核心）：

    ```arduino
    put(key, val)
        ↓
    计算 hash
        ↓
    定位 bucket（table[i]）
        ↓
    判断：
    - 桶为空 → CAS 直接插入
    - 桶非空且锁空闲 → synchronized 加锁链表/树
        ↓
    链表插入 or 树插入
        ↓
    插入后判断链表是否需要树化（TREEIFY_THRESHOLD）
        ↓
    插入后判断是否需要扩容（size > threshold）

    ```

  - `push` 步骤解析

    步骤 | 细节
    ---|---
    ① hash | 对 key 做扰动处理，提高随机性，减少冲突。
    ② 定位桶 | (n - 1) & hash 快速定位数组下标（n是数组长度）。
    ③ CAS 尝试插入 | 如果 table[i] 是 null，直接 CAS 插入，提高并发性能。
    ④ synchronized 锁定桶 | 如果冲突，锁住链表头节点进行链表操作。
    ⑤ 链表/树插入 | 如果链表太长，超过 8，且数组大于 MIN_TREEIFY_CAPACITY，转成红黑树,前提是数组长度必须 >= 64，否则优先扩容。
    ⑥ 统计节点数 | 更新 baseCount。
    ⑦ 判断扩容 | 超过 sizeCtl 则触发扩容 transfer。

    ```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        if (key == null || value == null) throw new NullPointerException();
        int binCount = 0;
        // 循环查找，直到找到位置或 table 为空
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            // 若table 为空，初始化 table
            if (tab == null || (n = tab.length) == 0)
                tab = initTable(); // 初始化 table
            // 计算 hash 值，定位桶位置 若桶为空，CAS 插入
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                // 桶为空，CAS 插入
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                    break; // 插入成功则退出循环
            }
            // 若桶被 MOVED，正在扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f); // 正在扩容，协助迁移
            
            // hash 值对应桶不为空k 存在冲突，进入 synchronized 块锁定该 bin 的头结点
            else {
                V oldVal = null;
                synchronized (f) { // 锁定首节点，防止并发修改
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) { // 是链表
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // 链表匹配，替换旧值，退出循环
                                if (e.hash == hash &&
                                    ((ek = e.key) == key || (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // 尾插新节点
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key, value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            // 红黑树插入逻辑
                        }
                    }
                }
                // 链表长度超过阈值（8）转为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    // 链表转换红黑树, 前提是数组长度必须 >= 64，否则优先扩容。
                    treeifyBin(tab, i); // 链表转换红黑树
                // 若hash 值重复，则返回被替换旧值
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
        addCount(1L, binCount); // 更新元素数量并检查是否触发扩容
        return null;
    }
    ```

  - 扩容机制（transfer）
    - 每个线程搬自己的一部分桶，分段扩容,多线程并行搬迁数据
    - 使用 `transferIndex` 变量做分段
    - 给原数组翻倍
    - 低位（e.hash & oldCap == 0）留在原位置
    - 高位（否则）迁移到 newTable[i + oldCap]

    ```java
    // 负责将 tab 中的数据迁移到 nextTable 中（容量是原来的 2 倍）
    private final void transfer(Node<K,V>[] tab, Node<K,V> f) {
        int n = tab.length; // 原 table 长度
        Node<K,V>[] nextTab = nextTable;

        // 如果目标表不存在，则初始化
        if (nextTab == null) {
            try {
                // 新表容量是原表的两倍
                nextTab = (Node<K,V>[]) new Node<?,?>[n << 1];
            } catch (Throwable ex) {
                // 创建失败，放弃扩容，避免死循环
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            // 将新表保存到成员变量
            nextTable = nextTab;
            // 初始化迁移的起始索引为旧表长度，从尾部开始抢任务
            transferIndex = n;
        }

        // stride 决定每个线程迁移的最小单元，防止粒度太细
        int stride = (n > NCPU) ? (n >>> 3) / NCPU : n; // NCPU 是 CPU 核数
        if (stride < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE;

        // i 是迁移用的工作索引，从高位向低位遍历
        int bound = transferIndex;
        boolean advance = true;     // 是否向前推进索引 i
        boolean finishing = false;  // 是否进入最终迁移完成收尾阶段

        // 遍历旧表
        for (int i = n - 1; i >= 0 || finishing; ) {
            Node<K,V> e; // 当前槽位元素
            int fh;      // 当前节点的 hash 值

            if (advance) {
                --i;
                // 当前槽为空，标记为已迁移，放 ForwardingNode 占位
                if (i >= 0 && (e = tabAt(tab, i)) == null) {
                    advance = casTabAt(tab, i, null, new ForwardingNode<K,V>(nextTab));
                }
                // 当前槽是 ForwardingNode，说明迁移过了，跳过
                else if (i >= 0 && (fh = e.hash) == MOVED) {
                    advance = true;
                }
                // 否则说明此槽位未迁移，进入同步块处理
                else {
                    synchronized (e) {
                        // 再次确认 tab[i] 没变，避免被其他线程抢先修改
                        if (tabAt(tab, i) == e) {
                            // 定义两组链表：低位（放 i）和高位（放 i + n）
                            Node<K,V> loHead = null, loTail = null;
                            Node<K,V> hiHead = null, hiTail = null;
                            Node<K,V> next;

                            // 遍历链表，分类拆分到 low/high 链表中
                            do {
                                next = e.next;
                                if ((e.hash & n) == 0) { // hash & n == 0 -> 低位链表
                                    if (loTail == null)
                                        loHead = e;
                                    else
                                        loTail.next = e;
                                    loTail = e;
                                } else { // 否则放高位链表
                                    if (hiTail == null)
                                        hiHead = e;
                                    else
                                        hiTail.next = e;
                                    hiTail = e;
                                }
                            } while ((e = next) != null);

                            // 将低位链表放到新表索引 i
                            setTabAt(nextTab, i, loHead);
                            // 将高位链表放到新表索引 i + n
                            setTabAt(nextTab, i + n, hiHead);
                            // 将旧表位置标记为已迁移（ForwardingNode）
                            setTabAt(tab, i, new ForwardingNode<K,V>(nextTab));
                            advance = true;
                        }
                    }
                }
            }
            // 如果当前线程没有可处理的任务区间，进入收尾阶段
            else if (--bound <= 0) {
                finishing = true;
            }
        }

        // 迁移完成后，清理变量
        nextTable = null;
        table = nextTab;
        // 更新扩容阈值（0.75 倍新容量）
        sizeCtl = (n << 1) - (n >>> 2);
    }
    ```

  - 树化机制（红黑树转换）
    - 链表长度超过 8 且数组长度大于 64 时，把链表转成红黑树（TreeBin）
    - 树操作基本是 `synchronized` 加锁下进行
    - 搜索复杂度由 `O(n)` 变成 `O(logn)`
    - TreeNode 继承自 Node，增加了 prev、parent 等指针；
    - 红黑树是通过双向链表 + 平衡逻辑插入实现的；
    - TreeBin 是包装红黑树的容器，封装了平衡逻辑。

    ```java
    final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; 
        //  桶为空，直接返回
        if (tab == null || (b = tabAt(tab, index)) == null || b.hash < 0)
            return;
        // 桶不为空，且长度大于 8，且数组长度大于 64，则进行树化
        synchronized (b) {
            // 桶为空，直接返回
            if (tabAt(tab, index) == b) {
                TreeNode<K,V> hd = null, tl = null;
                // 遍历链表，将链表节点转换成红黑树
                for (Node<K,V> e = b; e != null; e = e.next) {
                    TreeNode<K,V> p = new TreeNode<>(e.hash, e.key, e.val, null);
                    if ((p.prev = tl) == null)
                        hd = p;
                    else
                        tl.next = p;
                    tl = p;
                }
                setTabAt(tab, index, new TreeBin<>(hd)); // 红黑树包装器
            }
        }
    }
    ```

  - 线程安全解析
    - 存放节点数组 `table`、节点连接字段：val、值：· 都由 `volatile` 修饰保证关键字段可见性（如 table, val, next）
    - `CAS(Compare And Swap)` 保证 `table[i]` 空桶插入无锁
    - `synchronized` | 保证链表/树插入安全

- HashTable(弃用)
  - 虽然线程安全，但全表加锁，性能极差。

### 7.2 List、Set

- CopyOnWriteArrayList
  - 基于数组的线程安全 List。
  - 写时复制：修改操作（如 add, remove）会先复制原数组，再修改副本，最后替换原引用。
  - 通过加锁和复制数组的方式，写操作不会影响正在读取的线程，适用于 读多写少 的并发场景。
  
  ```java
    // add 方法内部简化
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();  // 加锁保证线程安全
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1); // 复制数组
            newElements[len] = e; // 添加元素
            setArray(newElements); // 替换原数组
            return true;
        } finally {
            lock.unlock();
        }
    }
  ```

- CopyOnWriteArraySet
  - 基于 CopyOnWriteArrayList 实现的线程安全 Set。
  - 元素唯一性通过 list.contains() 判断实现。
  
  ```java
    public boolean add(E e) {
        return al.contains(e) ? false : al.add(e);
    }
  ```

### 7.3 Queue

#### 7.3.1 阻塞队列 BlockingQueue

- `BlockingQueue` 是 java.util.concurrent 包下的接口。
- 是一种支持阻塞的队列：
  - 当队列满时，插入操作会被阻塞（等待空间可用）。
  - 当队列空时，移除操作会被阻塞（等待元素可用）。
- 常用方法：

类型 | 抛出异常方法 | 返回特殊值（true/false 或 null） | 阻塞 | 超时
---|---|---|---|---
插入 | add(e) | offer(e) | put(e) | offer(e, timeout, unit)
移除 | remove() | poll() | take() | poll(timeout, unit)
检查 | element() | peek() | - | -

- add(e) remove() element() 方法不会阻塞线程。当不满足约束条件时，会抛出IllegalStateException 异常。例如：当队列被元素填满后，再调用add(e)，则会抛出异常。
- offer(e) poll() peek() 方法即不会阻塞线程，也不会抛出异常。例如：当队列被元素填满后，再调用offer(e)，则不会插入元素，函数返回false。
- 要想要实现阻塞功能，需要调用put(e) take() 方法。当不满足约束条件时，会阻塞线程。

- `ArrayBlockingQueue`
  - 基于数组实现的有界阻塞队列。
  - 插入/删除操作遵循**先进先出（FIFO）**原则。
  - 容量固定，构造时必须指定大小。
  - 内部维护一个数组、一个锁，保证线程安全。
  
  ```java
  BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
  queue.put(1);  // 阻塞插入
  Integer value = queue.take();  // 阻塞取出
  ```

- `LinkedBlockingQueue`
  - 基于链表的阻塞队列
  - 默认容量是 Integer.MAX_VALUE（非常大），也可以自定义大小。
  - 插入/移除通常分别使用不同的锁，提高并发性能。
  - 支持有界或无界模式（建议自定义上限，避免 OOM）。
  - 大量生产/消费场景下，性能更好。

  ```java
  BlockingQueue<String> queue = new LinkedBlockingQueue<>();
  queue.offer("hello");
  String item = queue.poll();
  ```

- `SynchronousQueue`
  - 容量为 0 的特殊队列！
  - 每个 put 必须等待一个 take，反之亦然，一对一配对传递元素。
  - 非常适合直接移交任务，而不是缓存元素。
  - 不存储元素，只负责线程之间的数据交接。
  - 非常适合高并发直接移交任务的场景，比如线程池中直接将任务交给线程执行。
  
  ```java
  BlockingQueue<String> queue = new SynchronousQueue<>();
  new Thread(() -> {
      try {
          queue.put("test");
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
  }).start();

  new Thread(() -> {
      try {
          String value = queue.take();
          System.out.println(value);
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
  }).start();
  ```

- `PriorityBlockingQueue`
  - 基于优先级的无界阻塞队列。
  - 元素按照自定义优先级排序，而不是 FIFO 顺序。
  - 内部使用**最小堆（Heap）**结构实现。
  - 队列可以动态增长，不会固定大小（无界队列）。
  - 元素必须是可比较的（Comparable），或者插入时提供比较器（Comparator）。
  - 插入元素不会阻塞（无界，插入永远不会失败），取元素为空时才阻塞。
  - 需要自定义对象优先级，可以让对象实现 Comparable 接口或者传入 Comparator。

  ```java
  import java.util.concurrent.PriorityBlockingQueue;

  class Task implements Comparable<Task> {
      private final String name;
      private final int priority; // 数字越小，优先级越高

      public Task(String name, int priority) {
          this.name = name;
          this.priority = priority;
      }

      @Override
      public int compareTo(Task other) {
          return Integer.compare(this.priority, other.priority);
      }

      @Override
      public String toString() {
          return "Task{name='" + name + "', priority=" + priority + '}';
      }
  }
  ```

  ```java
  import java.util.concurrent.PriorityBlockingQueue;

  class Producer implements Runnable {
      private final PriorityBlockingQueue<Task> queue;

      public Producer(PriorityBlockingQueue<Task> queue) {
          this.queue = queue;
      }

      @Override
      public void run() {
          try {
              // 生产一些任务
              queue.put(new Task("Low Priority Task", 5));
              System.out.println(Thread.currentThread().getName() + " produced Low Priority Task");

              Thread.sleep(500); // 模拟生产时间

              queue.put(new Task("High Priority Task", 1));
              System.out.println(Thread.currentThread().getName() + " produced High Priority Task");

              Thread.sleep(300);

              queue.put(new Task("Medium Priority Task", 3));
              System.out.println(Thread.currentThread().getName() + " produced Medium Priority Task");

          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
          }
      }
  }

  class Consumer implements Runnable {
      private final PriorityBlockingQueue<Task> queue;

      public Consumer(PriorityBlockingQueue<Task> queue) {
          this.queue = queue;
      }

      @Override
      public void run() {
          try {
              while (true) {
                  Task task = queue.take(); // take() 阻塞等待
                  System.out.println(Thread.currentThread().getName() + " consumed " + task);

                  Thread.sleep(200); // 模拟消费时间
              }
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
          }
      }
  }


  import java.util.concurrent.PriorityBlockingQueue;

  public class PriorityBlockingQueueMultiThreadExample {
      public static void main(String[] args) {
          PriorityBlockingQueue<Task> queue = new PriorityBlockingQueue<>();

          // 启动生产者
          Thread producer = new Thread(new Producer(queue), "Producer");
          // 启动消费者
          Thread consumer = new Thread(new Consumer(queue), "Consumer");

          producer.start();
          consumer.start();
      }
  }
  ```

- `DelayQueue`
  - 基于优先级队列（PriorityQueue）实现的延时队列。
  - 元素必须实现 Delayed 接口，且按照过期时间排序。
  - 元素到达指定的延时时间后才能被取出，否则取操作会阻塞。
  - 队列无界。
  - 常用于定时任务、超时控制场景。

  ```java
  class DelayedTask implements Delayed {
      private long expire;

      public DelayedTask(long delay) {
          this.expire = System.currentTimeMillis() + delay;
      }

      @Override
      public long getDelay(TimeUnit unit) {
          return unit.convert(expire - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
      }

      @Override
      public int compareTo(Delayed o) {
          return Long.compare(this.expire, ((DelayedTask) o).expire);
      }
  }

  // 使用
  DelayQueue<DelayedTask> queue = new DelayQueue<>();
  queue.put(new DelayedTask(3000));  // 3秒后可取出
  System.out.println(queue.take());  // 阻塞直到到达时间
  ```

- `LinkedBlockingDeque`
  - 基于链表实现的双端阻塞队列（Deque = Double Ended Queue）。
  - 可以在**队头（front）或队尾（rear）**插入、移除元素。
  - 支持先进先出（FIFO）和后进先出（LIFO）两种操作。
  - 支持容量限制（默认 Integer.MAX_VALUE），建议设定容量防止内存溢出。
  - 插入、删除分别独立锁，并发性能良好。

  ```java
  LinkedBlockingDeque<String> deque = new LinkedBlockingDeque<>(10);

  deque.putFirst("first"); // 队头插入
  deque.putLast("last");   // 队尾插入

  System.out.println(deque.takeFirst()); // 取出 first
  System.out.println(deque.takeLast());  // 取出 last
  ```

#### 7.3.2 非阻塞队列

- `ConcurrentLinkedQueue`
  - 基于链表实现的线程安全、非阻塞队列。
  - 内部采用 CAS（Compare-And-Swap）无锁机制 来保证并发安全。
  - 属于宽松一致性，即不保证严格的实时一致性（最终一致性）。
  - 是无界队列（没有容量限制）。
  - `ConcurrentLinkedQueue` 使用链表节点连接元素。
  - 通过 head 和 tail 两个原子指针来管理队列。
  - 依赖 CAS 自旋算法 来实现插入/删除，而不是加锁。
  - 如何通过不加锁实现线程安全。
    - 无锁`（Lock-Free）`+ `CAS（Compare-And-Swap）`原子操作
    - 入队（`add/offer`）和出队（`poll`）操作，都是通过 自旋 `CAS` 保证多线程下的数据一致性。
    - `CAS` 是一种底层硬件指令：比较内存中某个位置的值是否等于预期值，如果是则更新成新值，否则重试。
    - `ConcurrentLinkedQueue` 内部维护了 `volatile` 的 `head` 和 `tail`，确保线程对节点的可见性。
    - 由于不加锁，避免了线程挂起/唤醒的开销，因此在 高并发环境下比传统加锁（比如 `synchronized`）性能更好。

  ```java
  import java.util.concurrent.ConcurrentLinkedQueue;

  public class ConcurrentLinkedQueueExample {
      public static void main(String[] args) {
          ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();

          // 入队
          queue.add("A");
          queue.add("B");
          queue.offer("C");

          // 出队
          System.out.println(queue.poll()); // 输出 A
          System.out.println(queue.peek()); // 输出 B (但不移除)
          System.out.println(queue.poll()); // 输出 B
          System.out.println(queue.poll()); // 输出 C
          System.out.println(queue.poll()); // 输出 null（队列空）
      }
  }
  ```

- `ConcurrentLinkedDeque`
  - 基于非阻塞算法（`lock-free`） 的双端队列（`Deque`），是线程安全的，支持并发地从 两端插入/移除元素。
  - 它使用的是 CAS（Compare-And-Swap）算法 来实现线程安全，避免了传统的加锁操作。
  - 用于高并发场景，无需容量限制，任务不需等

    ```java
    import java.util.concurrent.*;

    public class ConcurrentLinkedDequeDemo {

        public static void main(String[] args) {
            ConcurrentLinkedDeque<Integer> deque = new ConcurrentLinkedDeque<>();

            // 启动两个线程同时添加元素
            Runnable producer = () -> {
                for (int i = 0; i < 10; i++) {
                    deque.addFirst(i); // 从头部添加
                    System.out.println(Thread.currentThread().getName() + " addFirst: " + i);
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException ignored) {}
                }
            };

            Runnable consumer = () -> {
                for (int i = 0; i < 10; i++) {
                    Integer val = deque.pollLast(); // 从尾部取出
                    System.out.println(Thread.currentThread().getName() + " pollLast: " + val);
                    try {
                        Thread.sleep(80);
                    } catch (InterruptedException ignored) {}
                }
            };

            new Thread(producer, "Producer").start();
            new Thread(consumer, "Consumer").start();
        }
    }

    ```

---

## 八、高级并发特性

### 8.1 内存语义与原子性工具

- `volatile` 保证可见性与禁止指令重排。
- `Atomic*` 类利用 `CAS` 和内存屏障实现原子性。
- `LongAdder` 高并发下分段累加，减轻竞争。
  - 为解决 `AtomicLong` 在高并发场景下，由于所有线程竞争同一个 `value` 字段，`CAS` 失败率高、重试频繁，性能下降严重。
  - `LongAdder` 的设计思想 —— 分段累加（`Striped64`）
    - 将一个变量拆分为多个变量（`Cell`）分段累加，最后求和时汇总结果。
    - 将并发压力分散到多个变量上，从而减少争抢。

### 8.2 CAS（Compare-And-Swap）机制

- 核心原理
  - `CAS` 操作包含三个参数：
    - 内存位置（V）：要更新的变量的内存地址。
    - 预期值（A）：调用者认为当前的内存值应该是多少。
    - 新值（B）：如果预期值匹配，将要写入的新值。
  - 操作流程：
    - 读取内存位置V的当前值。
    - 比较当前值是否等于预期值A。
    - 如果相等，则将内存位置V的值更新为新值B。
    - 如果不相等，则操作失败，不修改内存值。
- 优点
  - 无锁（`Lock-Free`）：`CAS` 避免了传统锁机制（如互斥锁），减少了线程阻塞和上下文切换的开销。
  - 高并发性能：适合高并发场景，尤其是读多写少的场景。
  - 简单高效：CAS操作通常由硬件直接支持，执行速度快。
  - `Java` 原子类、大多数并发容器均基于此构建。
- 缺点
  - `ABA` 问题：
    - 问题描述：线程1读取值为A，线程2将值从A改为B再改回A，线程1的CAS操作会认为值未被修改，实际上中间发生了变化。
    - 解决方法：使用版本号或时间戳（如AtomicStampedReference在Java中）。
  - 自旋开销：
    - 如果 `CAS` 失败，线程可能需要循环重试（自旋），在高竞争场景下可能导致 `CPU` 资源浪费。
  - 仅限单一变量：
    - `CAS` 适合操作单个变量，无法直接处理多个变量的原子操作。
- 典型应用
  - 原子类：
    - Java中的 `AtomicInteger` `、AtomicReference` 等使用 `CAS` 实现线程安全的计数器或引用更新。
    - 示例：`AtomicInteger` 的 `incrementAndGet()` 方法内部通过 `CAS` 实现无锁自增。
  - 无锁数据结构：
    - 无锁队列、栈等数据结构常使用 `CAS` 来确保线程安全。
  - 乐观锁：
  - 数据库中的乐观锁机制常基于 `CAS` 思想，通过版本号检查数据是否被修改。

### 8.3 自旋锁、自适应锁

- 自旋锁：
  - 定义
    - 自旋锁是一种忙等待（`busy-waiting`）的锁机制，线程在尝试获取锁失败时不会立即阻塞，而是通过循环（自旋）反复检查锁是否可用，直到获取锁或达到某种条件。
  - 核心原理
    - 线程尝试通过原子操作（如CAS）获取锁。
    - 如果锁被占用，线程不会进入阻塞状态，而是循环检查锁的状态（忙等待）。
    - 一旦锁被释放，线程通过CAS尝试再次获取锁。
  - 优点
    - 低延迟：在锁持有时间短的场景下，自旋锁避免了线程阻塞和上下文切换的开销，获取锁的延迟较低。
    - 简单实现：自旋锁的实现逻辑简单，适合轻量级同步。
    - 适合高并发：在多核系统中，短时间的自旋等待可以快速响应锁的释放。
  - 缺点
    - CPU浪费：自旋期间线程占用CPU资源，长时间自旋可能导致性能下降。
    - 不适合长任务：如果锁持有时间长，自旋的开销会显著增加。
    - 公平性问题：自旋锁不保证先来先得，可能导致某些线程长时间无法获取锁。
  - 适用场景
    - 锁持有时间短：如临界区操作很快（几条指令）。
    - 多核系统：自旋锁在多核CPU上效果更好，因为自旋线程可以在其他核心上运行。
    - 高并发低冲突：竞争不激烈时，自旋锁效率高。
- 自适应锁：根据前次等待情况动态决定是否自旋。
  - 定义
    - 自适应锁是一种优化后的锁机制，能够根据运行时环境（如锁竞争程度、锁持有时间）动态调整线程的行为，结合自旋和阻塞的优点，以提高性能和资源利用率。
  - 核心原理
    - 自适应锁在尝试获取锁时，会根据历史数据或当前竞争情况决定是自旋等待还是阻塞。
    - 如果锁很快会被释放（短临界区），线程选择自旋；如果锁持有时间较长或竞争激烈，线程选择阻塞，进入等待队列。
    - 实现通常依赖运行时信息（如锁的统计数据）或操作系统支持。
  - 典型实现
    - `Java` 的 `synchronized`：`JVM` 中的 `synchronized` 关键字在高版本中使用了自适应锁优化。
      - 初始阶段：尝试自旋（轻量级锁，基于 `CAS` ）。
      - 竞争加剧：升级为重量级锁（阻塞，依赖操作系统互斥量）。
  - 自适应自旋：如 `Linux` 内核的自适应自旋锁，根据锁持有者的状态（是否在运行）决定自旋还是休眠。

### 8.4 并发设计模式

- Immutable Object（不可变对象）
  - 定义
    - 不可变对象是指对象在创建后其状态（字段值）不可修改的对象。所有字段通常是final的，且对象的行为通过返回新实例或只读操作实现。
  - 核心原理
    - 无状态修改：对象的字段在构造时初始化，之后不可更改。
    - 线程安全：由于状态不可变，多个线程访问时无需同步机制，避免了并发问题。
    - 副本机制：修改操作通过创建新对象实现，而不是修改原有对象。
  - 使用场景
    - 需要共享只读数据（如配置、常量）。
    - 高并发场景下需要确保线程安全。
    - 简化并发逻辑，避免锁的使用。
  - 优点
    - 线程安全：天然线程安全，无需加锁。
    - 简化设计：减少并发控制的复杂性。
    - 缓存：不可变对象适合作为缓存键或共享资源。
  - 缺点
    - 内存开销：每次修改生成新对象，可能增加内存使用。
    - 性能开销：对象创建和垃圾回收可能影响性能。
    - 适用性有限：不适合需要频繁修改状态的场景。
- Guarded Suspension（保护暂停）
  - 定义
    - 保护暂停模式用于线程间的协作，当某个条件不满足时，线程暂停（等待），直到条件满足后再继续执行。通常结合锁和条件变量（如wait/notify或Condition）实现。
  - 核心原理
    - 条件检查：线程检查是否满足执行条件（如队列不为空）。
    - 暂停等待：条件不满足时，线程调用wait()进入等待状态，释放锁。
    - 唤醒继续：其他线程修改条件后通过notify()或notifyAll()唤醒等待线程。
  - 使用场景
    - 生产者-消费者模型（如阻塞队列）。
    - 线程需要等待特定条件（如资源可用）。
    - 异步任务的同步等待。
  - 优点
    - 高效协作：线程仅在条件满足时运行，避免忙等待。
    - 资源节约：等待线程释放锁，降低CPU占用。
    - 灵活性：可处理复杂的条件逻辑。
  - 缺点
    - 复杂性：需要正确管理锁和条件变量，易出错。
    - 死锁风险：不当使用可能导致死锁或线程饥饿。
    - 通知丢失：如果notify调用早于wait，可能导致线程永久等待。
- Two-phase Termination（二阶段终止）
  - 定义
    - 二阶段终止模式用于优雅地终止线程，确保线程在收到终止信号后能完成必要清理工作（如释放资源、保存状态）后再退出。
  - 核心原理
    - 第一阶段：发送终止信号（如设置标志或中断）。
    - 第二阶段：线程检查终止信号，执行清理逻辑后退出。
    - 通常使用volatile标志或线程中断机制实现。
  - 使用场景
    - 需要安全关闭线程（如服务线程、定时任务）。
    - 线程需要释放资源（如文件句柄、数据库连接）。
    - 长期运行的任务（如事件循环）。
  - 优点
    - 优雅终止：确保线程安全退出，避免资源泄漏。
    - 可控性：允许线程完成必要工作后再停止。
  - 缺点
    - 延迟退出：清理工作可能导致终止延迟。
    - 复杂清理：若清理逻辑复杂，可能增加维护成本。
    - 中断局限：依赖中断机制时，可能因线程阻塞而失效。
- Worker Thread（工作线程）
  - 定义
    - 工作线程模式通过一组线程（线程池）处理任务，任务被提交到任务队列，工作线程从队列中获取任务并执行，适用于异步任务处理。
  - 核心原理
    - 任务队列：任务被放入共享队列（如阻塞队列）。
    - 工作线程：多个线程从队列中取出任务执行。
    - 线程复用：线程执行完任务后继续处理下一个任务，避免频繁创建线程。
  - 使用场景
    - 高并发任务处理（如Web服务器、数据库查询）。
    - 需要异步执行的任务（如日志记录、消息处理）。
    - 线程池管理（如Java的ExecutorService）。
  - 优点
    - 高效性：线程复用减少创建/销毁开销。
    - 可扩展：通过调整线程池大小适应负载。
    - 解耦：任务提交与执行分离，简化设计。
  - 缺点
    - 资源竞争：任务队列可能成为瓶颈。
    - 复杂管理：需要管理线程池大小和任务优先级。
    - 任务延迟：高负载下任务可能排队等待。
- Thread-per-Message（消息分发）
  - 定义
    - 消息分派模式为每个消息或请求分配一个独立线程处理，强调任务的独立性和异步性。通常用于高吞吐量的请求处理场景。
  - 核心原理
    - 消息触发：每个消息/请求到达时，创建一个新线程处理。
    - 独立执行：线程处理完消息后终止，无需复用。
    - 异步处理：消息发送者无需等待处理完成。
  - 使用场景
    - 轻量级、短生命周期的任务（如网络请求处理）。
    - 高吞吐量场景（如聊天服务器、事件驱动系统）。
    - 不需要线程复用的场景。
  - 优点
    - 简单性：每个任务独立，逻辑清晰。
    - 隔离性：任务间互不影响，适合错误隔离。
    - 高吞吐量：适合快速响应的场景。
  - 缺点
    - 高开销：频繁创建/销毁线程导致性能开销。
    - 资源限制：线程数过多可能耗尽系统资源。
    - 不适合长任务：长时间运行的任务会导致线程堆积。

### 8.5 ThreadLocal

- 实现原理
  - `ThreadLocal` 本身不存储数据，仅作为 `ThreadLocalMap` 的键。
  - 每个线程的 `ThreadLocalMap` 是独立的，互不影响。
  - `ThreadLocalMap` 使用弱引用存储键（ThreadLocal 对象），以便在 `ThreadLocal` 实例被垃圾回收时清理无用条目（但可能导致内存泄漏，详见下文）。
  - 底层原理：`Thread` 类中有一个 `ThreadLocalMap` 成员。
    - 每个线程（`Thread` 对象）内部维护一个 `ThreadLocalMap` `实例，ThreadLocalMap` 是一个定制化的哈希表，键是 `ThreadLocal` 对象，值是线程的变量副本。
    - ThreadLocalMap 由 ThreadLocal 类内部定义，存储在 Thread 类的 threadLocals 字段中。
  - 操作流程：
    - 设置值：调用 ThreadLocal.set(T value) 时，ThreadLocal 会获取当前线程的 ThreadLocalMap，将 (ThreadLocal, value) 键值对存入该 Map。
    - 获取值：调用 ThreadLocal.get() 时，ThreadLocal 从当前线程的 ThreadLocalMap 中查找对应 ThreadLocal 的值。
    - 移除值：调用 ThreadLocal.remove() 会从当前线程的 ThreadLocalMap 中删除对应 ThreadLocal 的键值对。
  - 结构图
  
  ```text
    Thread 1
    ├── threadLocals: ThreadLocalMap
    │    ├── Entry: [ThreadLocal1, Value1]
    │    ├── Entry: [ThreadLocal2, Value2]
    Thread 2
    ├── threadLocals: ThreadLocalMap
    │    ├── Entry: [ThreadLocal1, Value3]
    │    ├── Entry: [ThreadLocal2, Value4]
  ```

  - 使用场景：
    - 用户会话信息：如 Web 应用中将用户 Session 信息绑定到当前线程，避免跨线程干扰。
    - 数据库连接：为每个线程维护独立的数据库连接（如 Connection 对象），避免连接池竞争。
    - 事务管理：在 Spring 框架中，ThreadLocal 用于管理事务上下文（如 Transaction 对象）。

- 内存泄漏问题
  - 问题描述
    - `ThreadLocal` 可能导致内存泄漏，特别是在线程池场景中。内存泄漏的根源在于 `ThreadLocalMap` 的设计和线程生命周期。
    - `ThreadLocalMap` 的弱引用：
    - `ThreadLocalMap` 的键（`ThreadLocal` 对象）是弱引用，当 `ThreadLocal` 实例没有强引用时，会被垃圾回收。但是，值（线程的变量副本）是强引用，即使 `ThreadLocal` 键被回收，值仍然保留在 `ThreadLocalMap` 中，导致内存无法释放。
    - 线程池场景：
      - 线程池中的线程是长期存活的，`ThreadLocalMap` 不会随着任务结束而销毁。
      - 如果任务没有调用 `ThreadLocal.remove()` 清理值，`ThreadLocalMap` 中的键值对会一直存在，导致内存泄漏。
    - 内存泄漏示意图：

    ```text
    Thread (长期存活)
    ├── threadLocals: ThreadLocalMap
    │    ├── Entry: [null, Value] // ThreadLocal 被回收，值仍存留
    ```

  - 解决方案：
    - 显式清理：
      - 在使用完 ThreadLocal 后，调用 remove() 方法清理 ThreadLocalMap 中的键值对。
    - 使用 try-finally 块
    - 避免长期存活的 ThreadLocal：
      - 尽量将 ThreadLocal 实例定义为局部变量或短生命周期对象，避免长期持有。
      - 使用静态 ThreadLocal 时需格外注意清理。
    - 框架支持：
      - 在 Spring 等框架中，事务管理器通常会自动清理 ThreadLocal（如 TransactionSynchronizationManager）。
      - 使用线程池时，可以通过拦截器或任务包装器自动清理。

    ```java
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;

    public class ThreadLocalLeakExample {
        private static final ThreadLocal<String> context = new ThreadLocal<>();

        public static void main(String[] args) {
            ExecutorService executor = Executors.newFixedThreadPool(2);

            for (int i = 0; i < 5; i++) {
                executor.submit(() -> {
                    try {
                        context.set("Data-" + Thread.currentThread().getName());
                        System.out.println("Thread: " + Thread.currentThread().getName() + ", Context: " + context.get());
                    } finally {
                        context.remove(); // 防止内存泄漏
                    }
                });
            }

            executor.shutdown();
        }
    }
    ```

- `InheritableThreadLocal` 用于子线程继承
  - 定义
    - `InheritableThreadLocal` 是 `ThreadLocal` 的子类，允许子线程继承父线程的 `ThreadLocal` 值。
  - 核心原理
    - Thread 类中有一个 `inheritableThreadLocals` 字段（类型也是 ThreadLocalMap），专门存储 `InheritableThreadLocal` 的键值对。
    - 当创建子线程时，父线程的 `inheritableThreadLocals` 会被复制到子线程的 `inheritableThreadLocals` 中。
    - 子线程可以直接访问继承的值，也可以覆盖自己的值，而不影响父线程。
  - 优点
    - 简化上下文传递：无需显式传递父线程的上下文。
    - 线程隔离：子线程的修改不会影响父线程。
    - 灵活性：支持自定义继承逻辑（通过重写 `childValue` 方法）。
  - 缺点
    - 有限继承：仅在子线程创建时复制，无法动态同步父线程的变化。
    - 内存泄漏风险：与 `ThreadLocal` 类似，需注意清理 `inheritableThreadLocals。`
    - 适用性有限：仅适合父子线程关系的场景。

    ```java
    public class InheritableThreadLocalExample {
        private static final InheritableThreadLocal<String> context = new InheritableThreadLocal<>();

        public static void main(String[] args) {
            context.set("Parent-Context");

            Thread child = new Thread(() -> {
                System.out.println("Child thread context: " + context.get());
                context.set("Child-Context"); // 子线程修改自己的副本
                System.out.println("Child thread modified context: " + context.get());
            });

            child.start();

            try {
                child.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("Parent thread context: " + context.get()); // 父线程不受影响
            context.remove(); // 清理
        }
    }
    ```

  - 自定义继承逻辑
    - 可以通过重写 InheritableThreadLocal 的 childValue 方法自定义子线程的值：

    ```java
    InheritableThreadLocal<String> context = new InheritableThreadLocal<>() {
        @Override
        protected String childValue(String parentValue) {
            return parentValue + "-Inherited"; // 自定义继承值
        }
    };
    ```

  - 总结

    特性 |   说明
    ---|---
    实现原理    | 每个线程通过 ThreadLocalMap 存储独立的变量副本，ThreadLocal 作为键。
    使用场景 |   用户会话、数据库连接、事务管理、格式化工具等需要线程隔离的场景。
    内存泄漏问题 |   ThreadLocalMap 中的值可能是强引用，需显式调用 remove() 清理。
    InheritableThreadLocal |   子线程可继承父线程的 ThreadLocal 值，适合上下文传递，需注意清理。

### 8.6 VarHandle 类

- 替代 Unsafe 的安全 API。
- 提供对字段/数组/静态变量的原子性访问。
- 支持内存语义控制，如 volatile、acquire/release 等。

    ```java
    import java.lang.invoke.MethodHandles;
    import java.lang.invoke.VarHandle;

    public class VarHandleDemo {
        private int value = 0;
        private static final VarHandle VALUE;

        static {
            try {
                VALUE = MethodHandles.lookup().findVarHandle(VarHandleDemo.class, "value", int.class);
            } catch (Exception e) {
                throw new Error(e);
            }
        }

        public void update() {
            VALUE.set(this, 10); // 普通写
            VALUE.getAndAdd(this, 5); // 原子加
        }
    }
    ```

### 8.7 CompletableFuture

[CompletableFuture](./CompletableFuture.md/#completablefuture)

### 8.8 VirtualThread

- jdk 21、spring boot 3.2 以上使用
- 虚拟线程是一种由 JVM 管理的轻量级线程，与传统的平台线程（Platform Thread，绑定到操作系统的线程）不同，它不直接映射到 OS 线程，而是由 JVM 在少量平台线程（称为载体线程）上调度大量虚拟线程。
- 核心特性
  - 轻量级：
    - 虚拟线程的创建和销毁开销极低（内存占用仅几百字节），支持创建数百万个虚拟线程。
    - 相比之下，平台线程每个占用约 1MB 栈内存，创建过多会导致资源耗尽。
  - 高并发：
    - 适合 I/O 密集型任务（如 Web 请求、数据库查询、API 调用），因为虚拟线程在阻塞操作（如 I/O）时会自动让出载体线程，减少线程阻塞。
  - 简化并发模型：
    - 虚拟线程允许使用阻塞式编程（如同步代码）代替复杂的异步或回调模型（如 CompletableFuture），代码更直观。
    - 例如，传统的 Thread.sleep、InputStream.read 等阻塞操作在虚拟线程中高效运行。
  - 兼容性：
    - 虚拟线程与现有的 Java 并发 API（如 ExecutorService、Thread）兼容。
    - 无需修改代码，现有同步代码可在虚拟线程上运行，获得异步性能。
  - 调度机制：
    - 虚拟线程由 JVM 的调度器（基于 ForkJoinPool）管理，自动处理上下文切换。
    - 当虚拟线程遇到阻塞（如 I/O、锁等待），JVM 会将其“卸载”到内存，让载体线程执行其他虚拟线程。
- 功能对比

特性 | Virtual Thread | Platform Thread
---|---|---
资源占用 | 轻量级（几百字节） | 重量级（约 1MB 栈内存）
创建数量 | 可创建数百万个 | 受限于 OS 线程数（通常几千个）
阻塞成本 | 阻塞时自动卸载，低成本 | 阻塞占用 OS 线程，高成本
适用场景 | I/O 密集型（如 Web、数据库、API 调用） | CPU 密集型（如计算任务）
编程模型 | 阻塞式代码，简单直观 | 异步/回调复杂，或阻塞成本高
调度 | JVM 调度（ForkJoinPool） | OS 调度

- 示例：

  ```java
  import jakarta.annotation.PreDestroy;
  import org.springframework.stereotype.Service;
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RequestBody;
  import org.springframework.web.bind.annotation.RestController;

  import java.time.LocalDateTime;
  import java.util.ArrayList;
  import java.util.List;
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  import java.util.concurrent.Future;

  // 数据模型
  record ApiResult(String id, String data) {}
  record ResultDTO(String id, String data, String status, LocalDateTime timestamp) {}
  record ApiResponseDTO(List<ResultDTO> results, int total, String overallStatus) {}

  /**
   * 服务层：使用虚拟线程处理 API 调用
   */
  @Service
  public class ApiService {
      // 虚拟线程 ExecutorService
      private final ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

      // 模拟同步 API 调用
      private ApiResult callApiSync(String id) {
          try {
              // 模拟阻塞调用
              Thread.sleep(1000);
              if (id == null || id.isEmpty()) {
                  throw new IllegalArgumentException("Invalid ID: " + id);
              }
              return new ApiResult(id, "Data for " + id);
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
              throw new RuntimeException("API call interrupted for ID: " + id, e);
          }
      }

      // 二次封装结果
      private ResultDTO encapsulateResult(ApiResult result, String status) {
          return new ResultDTO(result.id(), result.data(), status, LocalDateTime.now());
      }

      // 二次封装错误
      private ResultDTO encapsulateError(String id, Throwable throwable) {
          return new ResultDTO(id, null, "Failed: " + throwable.getMessage(), LocalDateTime.now());
      }

      /**
       * 非阻塞api调用
       */
      public CompletableFuture<ApiResponseDTO> processApiCalls(List<String> ids) {
          return CompletableFuture.supplyAsync(() -> {
              // 现有逻辑
              List<Future<ResultDTO>> futures = new ArrayList<>();
              for (String id : ids) {
                  futures.add(executor.submit(() -> {
                      try {
                          return encapsulateResult(callApiSync(id), "Success");
                      } catch (Exception e) {
                          return encapsulateError(id, e);
                      }
                  }));
              }
              // 收集结果
              List<ResultDTO> results = new ArrayList<>();
              for (Future<ResultDTO> future : futures) {
                  try {
                      results.add(future.get(3, TimeUnit.SECONDS));
                  } catch (Exception e) {
                      results.add(encapsulateError("Unknown", e));
                  }
              }
              boolean allSuccess = results.stream().allMatch(dto -> dto.status().startsWith("Success"));
              return new ApiResponseDTO(results, ids.size(), allSuccess ? "All Success" : "Partial Failure");
          }, executor);
      }

      // 关闭 ExecutorService
      @PreDestroy
      public void shutdownExecutor() {
          executor.close();
      }
  }

  /**
   * 控制器
   */
  @RestController
  public class ApiController {
      private final ApiService apiService;

      public ApiController(ApiService apiService) {
          this.apiService = apiService;
      }

      @PostMapping("/process-apis")
      public ApiResponseDTO processApis(@RequestBody List<String> ids) {
          return apiService.processApiCalls(ids);
      }
  }
  ```

- Spring Boot 配置（可选）：
  - 如果需要全局虚拟线程支持，可在 application.properties 中启用：

```java
spring.threads.virtual.enabled=true
```

### 8.9 响应式编程（基础）

> Reactive Streams 是一个轻量级规范（2015 年发布），定义了异步流处理的标准接口，重点解决**背压（Backpressure）**问题。
> 提供非阻塞、异步的流处理机制，支持生产者和消费者之间的动态流量控制。
> 主要组成部分：Reactive Streams, Project Reactor, RxJava

#### Reactive Streams

- 核心概念
  - Publisher
    - 数据生产者，发布数据流（可能是无限的）。
    - 负责向订阅者推送数据（onNext）、错误（onError）或完成信号（onComplete）。
    - 接口：

    ```java
    interface Publisher<T> { 
        void subscribe(Subscriber<? super T> subscriber); 
    }
    ```

  - Subscriber
    - 数据消费者，订阅 Publisher 并处理数据。
    - 通过 `onSubscribe` 获取 `Subscription，控制数据流。`
    - 接口：

    ```java
    interface Subscriber<T> { 
        void onSubscribe(Subscription subscription); 
        void onNext(T item); 
        void onError(Throwable throwable); 
        void onComplete();
    }
    ```

  - Subscription
    - 订阅关系，连接 `Publisher` 和 `Subscriber。`
    - request(n) 用于请求 n 条数据，实现背压；cancel() 取消订阅。
    - 接口：

    ```java
    interface Subscription { 
        void request(long n); 
        void cancel(); 
    }
    ```

  - Processor
    - 同时是 Publisher 和 Subscriber，用于数据流转换或处理。
    - 例如，过滤、映射等操作。
- 背压（Backpressure）
  - 定义：当数据生产者（Publisher）生产数据的速度快于消费者（Subscriber）处理速度时，背压机制允许消费者通过 Subscription.request(n) 控制数据流速，防止内存溢出或系统崩溃。
  - 实现：Subscriber 动态请求数据量（如 request(10)），Publisher 根据请求发送数据。
- 关键原则
  - 异步非阻塞：数据处理在异步线程中执行，避免阻塞主线程。
  - 事件驱动：基于事件（onNext、onError、onComplete）处理数据流。
  - 背压支持：确保生产者和消费者协调工作。
  - 标准化：Reactive Streams 规范被 JDK 9（Flow API）、Project Reactor、RxJava 等实现。

- 示例代码：

```java
import java.util.concurrent.Flow.*;
import java.util.concurrent.SubmissionPublisher;

public class FlowExample {
    public static void main(String[] args) throws InterruptedException {
        // 创建 Publisher
        SubmissionPublisher<String> publisher = new SubmissionPublisher<>();

        // 创建 Subscriber
        Subscriber<String> subscriber = new Subscriber<>() {
            private Subscription subscription;

            @Override
            public void onSubscribe(Subscription subscription) {
                this.subscription = subscription;
                subscription.request(1); // 初始请求 1 条数据
            }

            @Override
            public void onNext(String item) {
                System.out.println("Received: " + item);
                subscription.request(1); // 请求下一条
            }

            @Override
            public void onError(Throwable throwable) {
                System.err.println("Error: " + throwable.getMessage());
            }

            @Override
            public void onComplete() {
                System.out.println("Completed");
            }
        };

        // 订阅
        publisher.subscribe(subscriber);

        // 发布数据
        publisher.submit("Item 1");
        publisher.submit("Item 2");
        publisher.submit("Item 3");
        publisher.close(); // 完成

        Thread.sleep(1000); // 等待处理
    }
}
```

#### Project Reactor

- Project Reactor 是 Spring 生态的默认响应式框架，基于 Reactive Streams 规范，广泛用于 Spring WebFlux（响应式 Web 框架）。它提供了两种核心类型：
  - Mono：表示 0 或 1 个元素的异步流（如单个 API 响应）。
  - Flux：表示 0 到 N 个元素的异步流（如列表、流式数据）。
- 核心特性
  - 非阻塞：所有操作异步执行，适合高并发场景。
  - 背压支持：内置背压机制，消费者控制数据流速。
  - 操作符丰富：提供类似 Stream API 的操作符（如 map、filter、flatMap），支持数据转换和组合。
  - 错误处理：通过 onErrorReturn、onErrorResume 等操作符处理异常。
  - 调度器：通过 Schedulers 控制任务执行线程（如 parallel、boundedElastic）。
- 示例代码：
  - 说明：
    - `Flux`：处理 `List<String>`，每个 ID 异步调用 API。
    - `publishOn()` 和 `subscribeOn` 用于指定数据流在哪个线程上执行。
      - `subscribeOn`，控制整个流的起点（初始化），决定数据源和上游操作的线程，对下游无直接影响（除非无 publishOn）。 指定初始化运行线程，若后续无 publishOn，则默认使用 subscribeOn 的线程。
      - `publishOn()`，控制流的某一段（当前publishOn()到下一个publishOn()之间的操作），切换后续操作和订阅者的线程，不影响上游。从声明 publicOn 开始，到下一个 publishOn 之间，所有操作和订阅者都运行在第一个 publishOn() 所指定的线程。
    - `subscribe()` 是一个激活流的动作，触发整个数据管道（从数据源到操作符到消费者）的执行。
      - `subscribe()` 本身的调用线程不一定执行消费逻辑，具体线程由 `subscribeOn` 和 `publishOn` 决定。
      - `Spring WebFlux`：控制器返回 `Mono/Flux`，`Spring` 自动订阅，开发者无需显式调用 `subscribe()`。
    - `flatMap`：将同步调用包装为 `Mono`，在异步线程（`Schedulers.boundedElastic`）执行。
    - `onErrorResume`：捕获异常，返回默认结果。
    - `collectList`：将 `Flux` 收集为 `Mono<List>`。
    - `map`：封装为 `ApiResponseDTO`。
  - 调度器（`Schedulers`）
    - `Schedulers.immediate()`：当前线程执行。
    - `Schedulers.single()`：单一线程，适合低并发。
    - `Schedulers.boundedElastic()`：弹性线程池，适合 `I/O` 密集型任务。
    - `Schedulers.parallel()`：固定线程池，适合 `CPU` 密集型任务。

```java
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.core.scheduler.Schedulers;

public class ReactorExample {
    public static void main(String[] args) throws InterruptedException {
        // 创建 Flux
        Flux<String> flux = Flux.just("id1", "id2", "id3")
            // 异步调用 API（模拟）
            .flatMap(id -> Mono.fromCallable(() -> callApiSync(id))
                .subscribeOn(Schedulers.boundedElastic()) // 异步线程
            )
            // 二次封装
            .map(data -> new ResultDTO(data, "Success"))
            // 错误处理
            .onErrorResume(e -> Flux.just(new ResultDTO("Unknown", "Error: " + e.getMessage())))
            // 收集结果
            .collectList()
            // 转换为 Mono
            .map(results -> new ApiResponseDTO(results, results.size(), "Success"));

        // 订阅并打印
        flux.subscribe(response -> System.out.println("Response: " + response));

        Thread.sleep(2000); // 等待异步完成
    }

    // 模拟同步 API 调用
    private static String callApiSync(String id) throws InterruptedException {
        Thread.sleep(1000);
        if (id.equals("id3")) {
            throw new RuntimeException("API failed for " + id);
        }
        return "Data for " + id;
    }

    // DTO 定义
    record ResultDTO(String id, String status) {}
    record ApiResponseDTO(List<ResultDTO> results, int total, String overallStatus) {}
}
```

- 示例：
  - 自定义订阅

    ```java
    Flux.just("A", "B").subscribe(
    item -> System.out.println("Item: " + item), // onNext
    error -> System.err.println("Error: " + error), // onError
    () -> System.out.println("Completed") // onComplete
    );
    ```

  - 切换线程

    ```java
    Flux.just("A", "B")
        .publishOn(Schedulers.parallel()) // 切换线程
        .subscribe(item -> System.out.println(Thread.currentThread().getName() + ": " + item));
    ```

  - 错误处理

    ```java
    // 返回默认值
    Flux.just("A", "B")
        .map(s -> { if (s.equals("B")) throw new RuntimeException("Error"); return s; })
        .onErrorReturn("Default")
        .subscribe(System.out::println);

    // 返回新的流
    Flux.just("A", "B")
    .map(s -> { if (s.equals("B")) throw new RuntimeException("Error"); return s; })
    .onErrorResume(e -> Flux.just("C", "D"))
    .subscribe(System.out::println);
    ```

  - 背压控制

    ```java
    Flux.range(1, 100)
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10); // 初始请求 10 条
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println(value);
            request(1); // 每次请求 1 条
        }
    });
    ```

  - Spring WebFlux

  ```java
  import org.springframework.web.bind.annotation.PostMapping;
  import org.springframework.web.bind.annotation.RequestBody;
  import org.springframework.web.bind.annotation.RestController;
  import reactor.core.publisher.Flux;
  import reactor.core.publisher.Mono;
  import reactor.core.scheduler.Schedulers;

  import java.time.LocalDateTime;
  import java.util.List;

  @RestController
  public class ApiController {
      private final ApiService apiService;

      public ApiController(ApiService apiService) {
          this.apiService = apiService;
      }

      @PostMapping("/process-apis")
      public Mono<ApiResponseDTO> processApis(@RequestBody List<String> ids) {
          return apiService.processApiCalls(ids);
      }
  }

  @Service
  class ApiService {
      private ApiResult callApiSync(String id) throws InterruptedException {
          Thread.sleep(1000);
          if (id.equals("id3")) {
              throw new RuntimeException("API failed for " + id);
          }
          return new ApiResult(id, "Data for " + id);
      }

      public Mono<ApiResponseDTO> processApiCalls(List<String> ids) {
          return Flux.fromIterable(ids)
              .flatMap(id -> Mono.fromCallable(() -> callApiSync(id))
                  .subscribeOn(Schedulers.boundedElastic()) // 异步 I/O
                  .map(result -> new ResultDTO(result.id(), result.data(), "Success", LocalDateTime.now()))
                  .onErrorResume(e -> Mono.just(new ResultDTO(id, null, "Failed: " + e.getMessage(), LocalDateTime.now())))
              )
              .publishOn(Schedulers.parallel()) // 切换线程处理结果
              .collectList()
              .map(results -> new ApiResponseDTO(
                  results,
                  ids.size(),
                  results.stream().allMatch(dto -> dto.status().equals("Success")) ? "All Success" : "Partial Failure"
              ));
      }

      record ApiResult(String id, String data) {}
      record ResultDTO(String id, String data, String status, LocalDateTime timestamp) {}
      record ApiResponseDTO(List<ResultDTO> results, int total, String overallStatus) {}
  }
  ```

CompletableFuture< T> 返回值为什么没有对CompletableFuture的封装

ThreadLocal 和线程安全变量的区别？

CAS 如何实现原子操作？有什么缺点？

Unsafe 和 VarHandle 有什么区别？

如何使用 CompletableFuture 实现异步任务编排？

Java 17 引入的并发优化有哪些？

---

## 九、性能调优与问题排查

### 9.1 合理设置线程数

- CPU 密集型：核心数 + 1
  - CPU 密集型任务：计算密集（如加密、图像处理），线程数接近 CPU 核心数，避免过多上下文切换。
- IO 密集型：核心数 * 2
  - I/O 密集型任务：涉及阻塞操作（如网络请求、数据库查询），线程数可适当增加，弥补等待时间。
- 获取运行环境核心数

```java
int coreCount = Runtime.getRuntime().availableProcessors();
```

### 9.2 锁优化技巧

- 锁粗化 TODO
  - 定义：将多个小锁合并为一个大锁，减少锁获取/释放的开销。
  - 场景：频繁加锁的循环或连续操作。
- 锁消除
  - 定义：JVM 通过逃逸分析（Escape Analysis）移除不必要的锁。
  - 场景：局部对象（无逃逸）的同步操作。
- 锁分离
  - 定义：将锁拆分为更细粒度的锁，减少竞争。
  - 场景：读写分离（如 ReadWriteLock）。
- 无锁并发
  - 定义：使用 CAS（Compare-And-Swap）或原子类（如 AtomicInteger）代替锁。
  - 场景：计数器、状态标志。
- 实践建议
  - 优先细粒度锁：减少锁范围，降低竞争。
  - 使用并发工具：ConcurrentHashMap、LongAdder 等替代传统锁。
  - JVM 优化：确保 -XX:+DoEscapeAnalysis 和 -XX:+EliminateLocks 启用。
  - 监控锁竞争：使用 JVisualVM 或 Async Profiler 分析锁开销。

### 9.3 常见并发问题

- 死锁：资源循环等待
- 活锁
  - 定义：线程不断尝试操作但无法前进（如两线程互相让步）。
  - 场景：动态锁顺序调整失败。
  - 解决：
    - 引入随机性（如随机等待）。
    - 简化竞争逻辑。
- 线程饥饿
  - 定义：某些线程因优先级低或锁竞争无法获得资源。
  - 场景：高优先级线程霸占 CPU。
  - 解决：
    - 使用公平锁（如 ReentrantLock(true)）。
    - 调整线程优先级（谨慎使用）。
- 上下文切换过多的开销
  - 定义：线程频繁切换导致 CPU 开销增加。
  - 原因：线程数过多、锁竞争频繁。
  - 解决：
    - 优化线程数（参考 9.1）。
    - 减少锁竞争（使用无锁或细粒度锁）。
    - 使用虚拟线程（Java 21+）降低切换成本。

### 9.4 常用诊断工具

- jstack（查看线程状态）
- jconsole / jvisualvm（图形化监控）
- Arthas（运行时诊断）
- 性能分析：Async Profiler, JFR, JProfiler

---

## 十、Java 内存模型（JMM）

### 10.1 主内存与工作内存

- 核心概念
  - 每个线程拥有自己的工作内存
  - 主内存：JVM 的共享内存区域，存储所有`实例变量`、`静态变量`和`数组对象`。
  - 工作内存：每个线程的私有内存，缓存主内存变量的副本（如局部变量、方法参数、线程栈）。
  - 交互规则：
    - 线程对变量的读写操作在工作内存中进行。
    - 工作内存的变量副本需通过主内存同步（如读/写操作）。
    - JMM 控制主内存与工作内存的交互，确保一致性。
- 工作流程
  - 读（read）：从主内存读取变量到工作内存。
  - 加载（load）：将读取的变量放入工作内存副本。
  - 使用（use）：线程使用工作内存中的变量副本。
  - 赋值（assign）：线程修改工作内存中的变量副本。
  - 存储（store）：将修改后的副本写入主内存。
  - 写（write）：主内存更新变量值。
- 关键点
  - 工作内存隔离导致可见性问题：一个线程的修改对其他线程不可见，除非通过同步机制。
  - JMM 通过 volatile、synchronized 等保证可见性。
- 实践建议
  - 使用 volatile 或 synchronized 确保变量修改对其他线程可见。
  - 避免直接依赖线程缓存，可能导致数据不一致。

### 10.2 happens-before 原则

- 保证线程间的可见性和有序性
- 核心概念
  - happens-before 是 JMM 的核心规则，定义了操作间的内存可见性和执行顺序。
  - 如果操作 A happens-before 操作 B，则 A 的结果对 B 可见，且 A 在 B 之前执行。
- 主要规则
  - 程序顺序规则：同一线程内，前面的操作 happens-before 后续操作。
  - 示例：x = 1; y = x; 中，x = 1 happens-before y = x。
  - 监视器锁规则：解锁操作 happens-before 后续的加锁操作。
  - 示例：synchronized 块的释放 happens-before 其他线程的获取。
  - volatile 变量规则：对 volatile 变量的写 happens-before 后续的读。
  - 示例：volatile int x; x = 1; happens-before 其他线程的 x 读。
  - 线程启动规则：Thread.start() happens-before 线程的任何操作。
  - 线程终止规则：线程的任何操作 happens-before 其他线程检测到终止（如 join()）。
  - 传递性：如果 A happens-before B，B happens-before C，则 A happens-before C。

### 10.3 内存屏障与指令重排序

- 编译器和 `CPU` 的优化行为
- 核心概念
  - 指令重排序：编译器和 `CPU` 为优化性能，可能调整指令执行顺序。
  - 编译器重排序：如调整无关指令顺序。
  - `CPU` 重排序：乱序执行（如指令流水线）。
- 内存屏障（Memory Barrier）：强制指令按特定顺序执行，防止重排序。
  - 类型：
    - LoadLoad：确保读操作顺序。
    - StoreStore：确保写操作顺序。
    - LoadStore：读后写顺序。
    - StoreLoad：写后读顺序（最强屏障）。
- JMM 中的内存屏障
  - JMM 通过内存屏障实现 happens-before：
    - volatile 写：插入 StoreStore 和 StoreLoad 屏障。
    - volatile 读：插入 LoadLoad 和 LoadStore 屏障。
    - synchronized：加锁/解锁插入屏障，确保操作原子性和可见性。

### 10.4 final 语义

- 对构造器完成后的可见性保证
- 核心概念
  - final 字段在对象构造完成后提供可见性保证，防止其他线程看到未初始化的值。
  - JMM 确保：
    - final 字段的写在构造函数结束前完成。
    - 其他线程在构造函数完成后看到 final 字段的初始化值。
- 规则
  - 构造器安全：final 字段初始化后，其他线程无需同步即可安全访问。
  - 禁止重排序：final 字段的写不会与构造函数外的操作重排序。
  - 限制：仅适用于 final 字段，非 final 字段需额外同步。
- 实践建议
  - 使用 final 字段增强不可变性。
  - 避免构造函数中 this 逃逸。
  - 结合 volatile 或 synchronized 保护非 final 字段。

### 10.5 synchronized 与 volatile 的内存语义

- volatile 的 happens-before 关系
- synchronized 的内存语义
  - 加锁：
    - 进入 synchronized 块时，线程从主内存刷新变量副本到工作内存（清空工作内存）。
    - 插入 Load 屏障，确保读取最新值。
  - 释放锁：
    - 退出 synchronized 块时，将工作内存的修改写入主内存。
    - 插入 Store 屏障，确保修改可见。
  - 效果：
    - 提供原子性（临界区操作不可分割）。
    - 提供可见性（修改对后续加锁线程可见）。
    - 提供有序性（防止重排序）。
- volatile 的内存语义
  - 写操作：
    - 将修改写入主内存，刷新其他线程的工作内存。
    - 插入 StoreStore 和 StoreLoad 屏障，防止重排序。
  - 读操作：
    - 从主内存读取最新值。
    - 插入 LoadLoad 和 LoadStore 屏障，确保顺序。
  - 效果：
    - 提供可见性：写操作对后续读可见。
    - 提供有序性：防止指令重排序。
    - 无原子性：不保证复合操作（如 i++）的原子性。
- synchronized vs volatile

|   特性    |   synchronized    |	volatile    |
|---|---|---|
|   原子性  |	提供（临界区）  |	不提供（需用 Atomic 类）    |
|   可见性	|   提供    |	提供    |
|   有序性  |	提供    |	提供    |
|   性能	|   较高开销（锁竞争）  |	较低开销（无锁）    |
|   场景    |	复杂同步（如多步操作）  |	简单状态标志、单变量    |

- 实践建议
  - synchronized：用于需要原子性和复杂逻辑的场景（如计数器、事务）。
  - volatile：用于状态标志、单变量读写（如 boolean 标志）。
  - Atomic 类：结合 volatile 和 CAS，提供无锁原子操作。

---

## 知识点

### ThreadLocal 和线程安全变量的区别？

- 定义：
  - ThreadLocal
    - 提供线程局部变量，每个线程拥有独立的变量副本，互不干扰。
    - 通过 ThreadLocal<T> 存储数据，使用 set(T)、get() 访问。
    - 常用于线程隔离场景（如用户上下文、事务管理）。
    - 弱引用，使用完毕后需要手动remove清理，否则容易造成内存泄漏。
  - 线程安全变量
    - 线程安全变量允许多线程共享数据，但通过同步避免竞争。
    - 指在多线程环境中能安全访问的变量，通常通过同步机制（如 synchronized、Lock）或并发类（如 AtomicInteger、ConcurrentHashMap）实现。
- 区别

|   特性    |   ThreadLocal |   线程安全变量    |
| --- | --- | --- |
|   数据隔离    |   每个线程独立副本，互不干扰  | 共享数据，多线程访问需同步  |
|   用途 |   线程局部存储（如上下文传递） |   共享状态管理（如计数器、缓存）  |
|   同步开销   | 无同步开销，线程隔离 |   有同步开销（如锁、CAS） |
|   典型类 |   ThreadLocal<T> |   AtomicInteger, ConcurrentHashMap    |
|   内存管理    | 需手动清理（防止内存泄漏）  | 由 JVM 垃圾回收管理 |
|使用场景 |   用户认证信息、日志上下文 |   计数器、线程安全的集合  |


### CAS 如何实现原子操作？有什么缺点？

> CAS（Compare-And-Swap，比较并交换） 是一种无锁的原子操作，基于硬件指令（如 cmpxchg）实现线程安全。它通过比较内存值与预期值是否相等来决定是否更新。

- 原理:
  - CAS 操作接受三个参数：内存位置（V）、预期值（A）、新值（B）。
  - 步骤：
    - 检查内存位置 V 的当前值是否等于 A。
    - 如果相等，将 V 更新为 B；否则，不更新。
  - 硬件保证 CAS 是一个原子操作，避免多线程干扰。
- 优点
  - 无锁，性能优于锁（低竞争场景）。
  - 简单高效，适合计数器、状态标志等。
- 缺点
  - ABA 问题：
    - 如果值从 A 变为 B 再变回 A，CAS 可能误认为未被修改。
    - 解决：使用版本号或 AtomicStampedReference
  - 高竞争开销：
    - 在高并发下，CAS 可能多次失败（因其他线程修改值），导致重试（自旋），增加 CPU 开销。
    - 解决：降低竞争（如分段锁）或使用其他机制（如 LongAdder）。
  - 仅限单变量：
    - CAS 只能操作单个变量，无法直接实现复杂原子操作（如多变量更新）。
    - 解决：使用锁或事务性机制。
  - 自旋开销：
    - 失败时自旋可能导致线程空转，浪费 CPU。
    - 解决：限制重试次数或使用 LongAdder。

### 多线程启动一个多线程类 与 多线程启动多个多线程类

- 对于同一个Runnable实例启动的多个线程：
  - 共享的资源（线程不安全）：
    - 该Runnable对象的所有成员变量（实例变量）
    - 该对象引用的其他共享对象（如集合、文件流等外部资源）
    - 类的静态变量（属于类级别，所有实例共享）
  
  - 线程私有的资源（线程安全）：
    - 方法内的局部变量（包括参数变量）
    - 用ThreadLocal修饰的变量
    - 每个线程独立的调用栈、程序计数器等运行时数据

  - Thread类自身的成员变量（如线程名name等）

- 关键结论：
  - 成员变量共享：只要多个线程操作同一个对象实例，其成员变量就是共享的

  - 执行过程独立：每个线程会独立执行run()方法，拥有自己的方法调用栈和局部变量

  - 静态变量特殊：即使是不同Runnable实例，静态变量也是全局共享的

```java
// 线程不安全示例
public class Demo7 {
    public static void main(String[] args) {
        //线程不安全
        Runnable run = new Ticket();
        new Thread(run).start();
        new Thread(run).start();
        new Thread(run).start();
    }
     static class Ticket implements Runnable{
        //总票数
        private int count = 5;
        @Override
        public void run() {
            while (count>0){
                //卖票
                System.out.println("正在准备卖票");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                count--;
                System.out.println("卖票结束,余票："+count);
            }
        }
    }
}

```
