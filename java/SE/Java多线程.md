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

- 任务队列：如果使用有界队列，当队列饱和时并超过最大线程数时就会执行拒绝策略；而如果使用无界队列，因为任务队列永远都可以添加任务，所以设置 maximumPoolSize 没有任何意义。
  - ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列（数组结构可配合指针实现一个环形队列）。
  - LinkedBlockingQueue： 一个由链表结构组成的有界阻塞队列，在未指明容量时，容量默认为 Integer.MAX_VALUE。
  - PriorityBlockingQueue： 一个支持优先级排序的无界阻塞队列，对元素没有要求，可以实现 Comparable 接口也可以提供 Comparator 来对队列中的元素进行比较。跟时间没有任何关系，仅仅是按照优先级取任务。
  - DelayQueue：类似于PriorityBlockingQueue，是二叉堆实现的无界优先级阻塞队列。要求元素都实现 Delayed 接口，通过执行时延从队列中提取任务，时间没到任务取不出来。
  - SynchronousQueue： 一个不存储元素的阻塞队列，消费者线程调用 take() 方法的时候就会发生阻塞，直到有一个生产者线程生产了一个元素，消费者线程就可以拿到这个元素并返回；生产者线程调用 put() 方法的时候也会发生阻塞，直到有一个消费者线程消费了一个元素，生产者才会返回。
  - LinkedBlockingDeque： 使用双向队列实现的有界双端阻塞队列。双端意味着可以像普通队列一样 FIFO（先进先出），也可以像栈一样 FILO（先进后出）。
  - LinkedTransferQueue： 它是ConcurrentLinkedQueue、LinkedBlockingQueue 和 SynchronousQueue 的结合体，但是把它用在 ThreadPoolExecutor 中，和 LinkedBlockingQueue 行为一致，但是是无界的阻塞队列。
- 拒绝策略：当线程池的线程数达到最大线程数时，需要执行拒绝策略。拒绝策略需要实现 RejectedExecutionHandler 接口，并实现 rejectedExecution(Runnable r, ThreadPoolExecutor executor) 方法。
  - AbortPolicy（默认）：丢弃任务并抛出 RejectedExecutionException 异常。
  - CallerRunsPolicy：由调用线程处理该任务。
  - DiscardPolicy：丢弃任务，但是不抛出异常。可以配合这种模式进行自定义的处理方式。
  - DiscardOldestPolicy：丢弃队列最早的未处理任务，然后重新尝试执行任务。

- ThreadPoolExecutor(推荐)
  - 线程数设置依据：
    - CPU 密集型(任务主要消耗 CPU 资源（如计算、压缩、加密）)：CPU核心数 + 1
    - IO 密集型(任务主要在执行 IO 操作（如文件读写、数据库、网络请求）)：2 * CPU核心数 或更多
  - 构造方法：
    - corePoolSize（必需）：核心线程数。默认情况下，核心线程会一直存活，但是当将 allowCoreThreadTimeout 设置为 true 时，核心线程也会超时回收。
    - maximumPoolSize（必需）：线程池所能容纳的最大线程数。当活跃线程数达到该数值后，后续的新任务将会阻塞。
    - keepAliveTime（必需）：线程闲置超时时长。如果超过该时长，非核心线程就会被回收。如果将 allowCoreThreadTimeout 设置为 true 时，核心线程也会超时回收。
    - unit（必需）：指定 keepAliveTime 参数的时间单位。常用的有：TimeUnit.MILLISECONDS（毫秒）、TimeUnit.SECONDS（秒）、TimeUnit.MINUTES（分）。
    - workQueue（必需）：任务队列。通过线程池的 execute() 方法提交的 Runnable 对象将存储在该参数中。其采用阻塞队列实现。
    - threadFactory（可选）：线程工厂。用于指定为线程池创建新线程的方式。
    - handler（可选）：拒绝策略。当达到最大线程数时需要执行的饱和策略。

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

- 使用 Executors 工具类(不推荐)：
  - 定长线程池（FixedThreadPool）
    - 特点：只有核心线程，线程数量固定，执行完立即回收，任务队列为链表结构的有界队列。
    - 应用场景：控制线程最大并发数。
  - 定时线程池（ScheduledThreadPool ）
    - 特点：核心线程数量固定，非核心线程数量无限，执行完闲置 10ms 后回收，任务队列为延时阻塞队列。
    - 应用场景：执行定时或周期性的任务。
  - 可缓存线程池（CachedThreadPool）
    - 特点：无核心线程，非核心线程数量无限，执行完闲置 60s 后回收，任务队列为不存储元素的阻塞队列。
    - 应用场景：执行大量、耗时少的任务。
  - 单线程化线程池（SingleThreadExecutor）
    - 特点：只有 1 个核心线程，无非核心线程，执行完立即回收，任务队列为链表结构的有界队列。
    - 应用场景：不适合并发但可能引起 IO 阻塞性及影响 UI 线程响应的操作，如数据库操作、文件操作等。

- 不推荐使用 Executors 工具类的原因
  - FixedThreadPool 和 SingleThreadExecutor：主要问题是堆积的请求处理队列均采用 LinkedBlockingQueue(无界)，可能会耗费非常大的内存，甚至 OOM。
  - CachedThreadPool 和 ScheduledThreadPool：主要问题是线程数最大数是 Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至 OOM。

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

- AtomicReference
  - 可以保证对象引用的原子性更新。
  - 适用于多个线程操作对象指针场景。
  - 只是保证引用本身的原子性，不保证对象内部字段的线程安全！

```java
AtomicReference<String> atomicReference = new AtomicReference<>("Hello");
atomicReference.compareAndSet("Hello", "World");
```

- LongAdder
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
需求 | 推荐使用
---|---
少量线程更新，竞争小 | AtomicInteger, AtomicLong
高并发场景，频繁累加 | LongAdder
更新引用（对象指针） | AtomicReference
更新字段且包含逻辑校验（如版本号） | AtomicReference + 版本管理

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

### 5.3 阻塞队列 BlockingQueue

- BlockingQueue 是 java.util.concurrent 包下的接口。
- 是一种支持阻塞的队列：
  - 当队列满时，插入操作会被阻塞（等待空间可用）。
  - 当队列空时，移除操作会被阻塞（等待元素可用）。
- 常用方法：

类型 | 抛出异常 | 返回特殊值（true/false 或 null） | 阻塞 | 超时
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

### 5.4 非阻塞队列

- `ConcurrentLinkedQueue`
  - 基于链表实现的线程安全、非阻塞队列。
  - 内部采用 CAS（Compare-And-Swap）无锁机制 来保证并发安全。
  - 属于 宽松一致性，即不保证严格的实时一致性（最终一致性）。
  - 是 无界队列（没有容量限制）。
  - ConcurrentLinkedQueue 使用 链表节点连接元素。
  - 通过 head 和 tail 两个原子指针来管理队列。
  - 依赖 CAS 自旋算法 来实现插入/删除，而不是加锁。
  - 如何通过不加锁实现线程安全。
    - 无锁`（Lock-Free）`+ `CAS（Compare-And-Swap）`原子操作
    - 入队（add/offer）和出队（poll）操作，都是通过 自旋 CAS 保证多线程下的数据一致性。
    - `CAS` 是一种底层硬件指令：比较内存中某个位置的值是否等于预期值，如果是则更新成新值，否则重试。
    - `ConcurrentLinkedQueue` 内部维护了 `volatile` 的 `head` 和 `tail`，确保线程对节点的可见性。
    - 由于 不加锁，避免了线程挂起/唤醒的开销，因此在 高并发环境下比传统加锁（比如 synchronized）性能更好。

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

### 5.5 管道通信

> Java 管道是一种特殊的流，用于在线程之间传递数据。它通常由一个输入管道流和一个输出管道流组成。输入管道流用于从一个线程读取数据，而输出管道流用于将数据写入另一个线程。这两个管道流之间的数据传输是单向的，即数据只能从输入流传输到输出流。

- `PipedInputStream` / `PipedOutputStream`
  - 字节流
- `PipedReader` / `PipedWriter`
  - 字符流
- 只能线程间通信，不能单线程使用。
- 一次只能一个线程写，一个线程读，否则可能抛异常或数据错乱。
- 不支持多对多（一写多读 / 多写一读要自己控制同步）
- 管道有内部缓冲区（默认大约 1024 bytes），写太快而读太慢可能造成阻塞。
  - 写入速度太快，缓冲区满了 → write() 会阻塞等待
  - 读取速度太慢，缓冲区空了 → read() 会阻塞等待
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
  BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(outputStream);
  BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);

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

          public Consumer(BufferedReader reader, AtomicInteger activeProducers, String name) {
              this.reader = reader;
              this.activeProducers = activeProducers;
              this.name = name;
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

- 接口：`Executor`, `ExecutorService`, `ScheduledExecutorService`
- 实现类：`ThreadPoolExecutor`, `ScheduledThreadPoolExecutor`,FixedThreadPool、CachedThreadPool、SingleThreadExecutor、ScheduledThreadPool

### 6.2 ThreadPoolExecutor 参数详解

- corePoolSize、maximumPoolSize、keepAliveTime、workQueue
- 拒绝策略（CallerRunsPolicy 等）

### 6.3 常见线程池类型

- `FixedThreadPool`, `CachedThreadPool`, `SingleThreadExecutor`, `ScheduledThreadPool`

### 6.4 Fork/Join 框架

- 工作窃取算法
- `RecursiveTask` 和 `RecursiveAction`

### 6.5 并发辅助工具类

- `CountDownLatch`, `CyclicBarrier`, `Semaphore`, `Exchanger`, `Phaser`

---

## 七、并发集合

### 7.1 ConcurrentHashMap

- JDK7：分段锁机制
- JDK8：CAS + 链表/红黑树优化
- ConcurrentHashMap（分段锁/JDK8优化）

- CopyOnWriteArrayList / CopyOnWriteArraySet

- ConcurrentLinkedQueue / ConcurrentLinkedDeque

- BlockingQueue 系列（各种实现适用场景）

### 7.2 CopyOnWriteArrayList / CopyOnWriteArraySet

- 读写分离
- 适用读多写少场景

### 7.3 ConcurrentLinkedQueue / ConcurrentLinkedDeque

- 无锁队列，CAS实现

### 7.4 各种 BlockingQueue

- `ArrayBlockingQueue`, `LinkedBlockingQueue`
- `PriorityBlockingQueue`, `SynchronousQueue`, `DelayQueue`

---

## 八、高级并发特性

### 8.1 ThreadLocal

- 实现原理
- 内存泄漏问题
- `InheritableThreadLocal` 用于子线程继承

### 8.2 并发设计模式

- Immutable Object（不可变对象）
- Guarded Suspension（保护暂停）
- Two-phase Termination（二阶段终止）
- Worker Thread（工作线程）
- Thread-per-Message（消息分发）

### 高级并发模型与模式

- 并发设计模式：

  Immutable Object

  Guarded Suspension

  Two-phase Termination

  Worker Thread

  Thread-per-Message

### 8.3 CompletableFuture

- 异步编程模型
- 链式调用 `.thenApply`, `.thenCompose`
- 异常处理 `.handle`, `.exceptionally`

### 8.4 响应式编程（基础）

- Reactive Streams 规范
- 简介：Project Reactor, RxJava

---

## 九、性能调优与问题排查

### 9.1 合理设置线程数

- CPU 密集型：核心数 + 1
- IO 密集型：核心数 * 2

### 9.2 锁优化技巧

- 锁粗化、锁消除
- 锁分离
- 无锁并发

### 9.3 常见并发问题

- 死锁：资源循环等待
- 活锁、线程饥饿
- 上下文切换过多的开销

### 9.4 常用诊断工具

- jstack（查看线程状态）
- jconsole / jvisualvm（图形化监控）
- Arthas（运行时诊断）
- 性能分析：Async Profiler, JFR, JProfiler

---

## 十、Java 内存模型（JMM）

### 10.1 主内存与工作内存

- 每个线程拥有自己的工作内存

### 10.2 happens-before 原则

- 保证线程间的可见性和有序性

### 10.3 内存屏障与指令重排序

- 编译器和 CPU 的优化行为

### 10.4 final 语义

- 对构造器完成后的可见性保证

### 10.5 synchronized 与 volatile 的内存语义

- 加锁释放锁的内存屏障效果
- volatile 的 happens-before 关系

---

## 学习建议

- [ ] 每个知识点配合代码实践
- [ ] 使用可视化工具监控线程状态
- [ ] 阅读 JDK 源码理解实现原理
- [ ] 在实际项目中应用并发技巧
- [ ] 持续关注 Java 并发库的版本演进

---

* 对于同一个Runnable实例启动的多个线程：
  * 共享的资源（线程不安全）：
    * 该Runnable对象的所有成员变量（实例变量）
    * 该对象引用的其他共享对象（如集合、文件流等外部资源）
    * 类的静态变量（属于类级别，所有实例共享）
  
  * 线程私有的资源（线程安全）：
    * 方法内的局部变量（包括参数变量）
    * 用ThreadLocal修饰的变量
    * 每个线程独立的调用栈、程序计数器等运行时数据

  * Thread类自身的成员变量（如线程名name等）

* 关键结论：
  * 成员变量共享：只要多个线程操作同一个对象实例，其成员变量就是共享的

  * 执行过程独立：每个线程会独立执行run()方法，拥有自己的方法调用栈和局部变量

  * 静态变量特殊：即使是不同Runnable实例，静态变量也是全局共享的

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

线程的六种状态

notify() 如何确认唤醒线程？ 关联关系是如何产生的？

* 线程锁
  * synchronized
  * ReentrantLock
  * ReentrantReadWriteLock

* 多线程相关类：
  * Condition
  * CountDownLatch
  * CyclicBarrier
  * Semaphore
  * Exchanger
  * LockSupport
  * Future
  * ThreadPoolExecutor
  * CompletableFuture
  * CompletionService
  * FutureTask
  * Callable
  * Runnable
  * Thread
  * ThreadLocal
  * Atomic
  * ThreadPoolExecutor
  * CompletableFuture

* Runnable 与 Callable
* 线程池 ExecutorService
* CompletableFuture 函数式多线程执行