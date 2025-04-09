# Java 多线程学习笔记

---

## 一、多线程基础概念

### 1.1 进程与线程的区别

- 进程是资源分配的基本单位
- 线程是 CPU 调度的基本单位
- 线程共享进程资源

### 1.2 为什么需要多线程

- 提高 CPU 利用率
- 改善程序响应性
- 简化建模复杂任务

### 1.3 Java 线程与操作系统线程的关系

- JVM 线程映射模型
- 绿色线程与本地线程

### 1.4 Java 线程模型 & Java 内存模型（JMM）

- 主内存与工作内存
- Happens-Before 原则
- 内存屏障
- final、synchronized、volatile 的内存语义

---

## 二、线程创建与启动

### 2.1 继承 Thread 类

- 重写 `run()` 方法
- 调用 `start()` 启动线程

### 2.2 实现 Runnable 接口

- 实现 `run()` 方法
- 使用 `Thread` 构造器启动

### 2.3 实现 Callable 接口 + FutureTask

- 与 Runnable 的区别：有返回值、可抛出异常
- 使用 `FutureTask` 获取结果

### 2.4 线程池方式

- 使用 Executors 工具类：FixedThreadPool、CachedThreadPool 等
- 推荐直接使用 ThreadPoolExecutor

---

## 三、线程生命周期与状态转换

### 3.1 六种状态
- NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED
新建（New）

就绪（Runnable）

运行（Running）

阻塞（Blocked）

等待（Waiting） / 超时等待（Timed Waiting）

终止（Terminated）

Thread.join() 和 Thread.sleep() 的作用与区别

### 3.2 状态转换图（建议配图）

### 3.3 方法对状态的影响
- `start()`、`run()`、`yield()`  
- `sleep()`、`join()`  
- `wait()`、`notify()`、`notifyAll()`

---

## 四、线程同步与锁机制

### 4.1 线程安全问题

- 竞态条件
- 可见性问题

### 4.2 synchronized 关键字

- （方法/代码块/类锁/对象锁）
- 同步方法与同步代码块
- 类锁 vs 对象锁

### 4.3 volatile 关键字

- 保证可见性，但不保证原子性
- 禁止指令重排
- 使用场景与局限
- （内存语义、适用场景）

### 4.4 显式锁机制（JUC 锁）

- `ReentrantLock`
- `ReadWriteLock`
- `StampedLock`
- 显式锁：ReentrantLock、ReadWriteLock、StampedLock

### 4.5 原子类（Atomic）

- `AtomicInteger`, `AtomicLong`, `AtomicReference`
- `LongAdder` 优化高并发累加性能

### 公平锁与非公平锁

### 死锁、活锁与饥饿现象

---

## 五、线程间通信

### 5.1 wait/notify机制

- 使用场景与注意事项
- 实现生产者-消费者模式
- wait() / notify() / notifyAll()（使用条件、典型案例：生产者-消费者）

### 5.2 Condition 接口

- 与 `Lock` 配合使用
- 多条件队列的实现

### 5.3 阻塞队列 BlockingQueue

- 接口介绍
- 实现类：`ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue`

### 5.4 管道通信

- `PipedInputStream` / `PipedOutputStream`
- `PipedReader` / `PipedWriter`

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