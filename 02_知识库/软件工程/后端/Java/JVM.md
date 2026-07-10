---
title: 'JVM'
category: '知识库'
status: 'reference'
organized: 2026-07-10
---

# JVM

- JVM 运行时内存区域

```
线程 1
└── Java 虚拟机栈
    └── main 栈帧
        └── user 引用
              │
              ▼
堆
└── User 对象
      │
      ▼
元空间
└── User 类元信息
```

## 线程

- 线程私有
  - 程序计数器
  - java 虚拟机栈
  - 本地方法栈
- 线程共享
  - 堆
  - 方法区、元空间

### 为什么虚拟机栈要线程私有

- 因为每个线程都有自己的方法调用过程，它们的方法调用链不同，局部变量也不同，所以不能够共用一个栈。
- 堆是多个线程共享的，所以多个线程能够访问同一个对象。

## 栈

- 栈放方法调用过程中的局部数据。
- 方法调用会向栈添加栈帧；执行完毕后弹出。

## 堆

- 堆存放对象、数组
    ```java
    对象
        对象头：锁信息、GC 年龄、类型指针等
        实例数据：基础数据类型存放值，引用数据类型存放引用地址
        对齐填充：让对象大小更规整
    ```
- 对象创建大致流程
    ```
    类加载检查
    JVM 先看 User 这个类有没有被加载过。

    分配内存
    在堆里给 User 对象分一块空间。

    初始化零值
    比如 int 默认是 0，引用类型默认是 null。

    设置对象头
    里面会放一些 JVM 管理对象需要的信息，比如对象属于哪个类、GC 标记、锁信息等。

    执行构造方法
    也就是调用 User() 构造器，给字段赋程序指定的值。
    ```

## 方法区|元空间

- Java 8 以前 方法区被称为 永久代 PermGen
- Java 8 以后 永久代被移除，改为元空间 Metaspace
- 主要存储
  - 类信息
  - 字段信息
  - 方法信息
  - 运行时常量池
  - 静态变量
  - 即时编译器 JIT 编译后的代码缓存等



## 程序计数器

- 用于记录当前线程执行到哪一行字节码的指示器。
- 在 cpu 线程切换时需要使用程序计数器使 JVM 知道线程执行到那里了。
- 特性
  - 线程私有
  - 占用空间小
  - 唯一一个不会出现OOM的区域

## 本地方法栈

- 和 java 虚拟机栈类似，但本地方法栈仅服务 Native 方法。



## GC 垃圾回收

### 引用类型

- 强引用：普通 new 出来的引用
- 软引用：内存不够时才回收
- 弱引用：只要发生 GC 就可能回收
- 虚引用：不能直接拿对象，主要用于跟踪回收

### Stop The World 简称 STW

- GC 时，JVM 会暂停用户线程，让 GC 线程专心做垃圾回收。
- 逻辑简述
  - 业务线程暂停
  - GC 线程工作
  - GC 完成
  - 业务线程继续

### 吞吐量 和 低延迟

- 吞吐量
  - 程序总运行时间里，业务线程真正执行代码的比例
  - 计算逻辑
    ```
    业务代码执行 99 秒
    GC 执行 1 秒

    吞吐量： 99%

    吞吐量高，适合：
      批处理
      后台计算
      数据分析任务
    ```
- 低延迟
  - 单次 GC 停顿时间尽量短
  - 低延迟适合：
    - 接口服务
    - 交易系统
    - 实时应用
    - 游戏服务器

### GC 判断一个对象是否是垃圾

- 可达性分析：从GC Roots 中是否能够找到这些对象
  1. 虚拟机栈中局部变量引用的对象
  2. 方法区中静态变量引用的对象
  3. 方法区中常量引用的对象
  4. Native 方法栈中 JNI 引用的对象
  5. 正在运行的线程对象
  6. 被 synchronized 锁持有的对象
  - 如果没有其他地方引用这个对象，它就从 GC Roots 找不到了，所以叫 ·不可达对象· 可被回收

- 引用计数：对象被引用一次，计数+1；引用失效 计数-1；计数为0回收；
  - 循环引用问题：外部已无法访问对象，但对象引用始终未释放；
    ```java

    class Node {
        Node next;
    }

    Node a = new Node();
    Node b = new Node();

    a.next = b;
    b.next = a;

    a = null;
    b = null;
    ```

- 垃圾回收算法
  - 标记清除
    - 逻辑: 标记所有可达对象，清理不可达对象；
    - 优点：
      - 实现简单
      - 不需要移动对象
    - 缺点：
      - 会产生内存碎片，内存使用不连续；
      ```
      内存: 若创建一个5MB 的对象，无法分配内存。
      [A][垃圾][B][垃圾][C]
      [空 2MB][对象][空 3MB][对象][空 1MB]
      ```
  - 标记复制
    - 逻辑: 把内存分成两份，将存活对象复制到另一块，然后一次性清空原区域；
    - 优点:
      - 没有内存碎片
      - 回收后内存连续
    - 缺点 : 
      - 浪费一部分空间
      - 对象存活率高时，复制成本大。
  - 标记整理
    - 逻辑: 标记存活对象；将存活对象向一端移动；清除边界外的垃圾；
    - 优点:
      - 内存连续，不存在内存碎片
    - 缺点: 
      - 移动对象成本较高
  - 分代收集
    - 通常将堆分为:
      - 老年代: 对象存活状态稳定，存活时间较长； 老年代对象存活率高，适合使用 `标记清除` 、`标记整理`
        - 什么对象会直接进老年代？
          1. 年龄达到晋升阈值
          2. 大对象可能直接进入老年代
          3. Survivor 空间放不下，部分对象提前进入老年代
          4. 动态年龄判断导致提前晋升
      - 年轻代: 对象存活状态不稳定，存活时间较短: 适合使用 `标记复制`
        - Eden 区 : 大多数新建对象创建时放入
        - Survivor From : Minor GC 后依旧存活的对象从Eden 晋升到 Survivor，对象经历多次 GC 后依旧存活，持续晋升，最后移动到 老年代
        - Survivor To : Minor GC 会将 From 中存活对象移动到 To，并清除From中死亡对象；后续再次GC触发，会将To中存活对象移动到From，并清除To中死亡对象。  
    - 逻辑
      1. 通常对象刚创建进入 Eden
      2. Eden 快满时会触发 Minor GC| Young GC 
      3. Eden 死亡对象会被回收
      4. 存活对象会移动 Survivor
      5. Survivor From 对象移动到 Survivor To
      6. 对象年龄 + 1
      7. From 和 To 对象交换 (Survivor 的 From 和 To 不是固定身份，而是每次 GC 后会交换角色)

- GC 分类
  - Minor GC / Young GC：
    - 只回收年轻代
  - Major GC / Old GC：
    - 主要回收老年代
  - Full GC：
    - 回收整个堆，通常还会涉及方法区/元空间等
    - Full GC 通常会比Minor GC 更慢；因为Full GC 需要处理整个堆，甚至包括方法区。

### 常见 GC 收集器

- Serial
  - 特点
    - 单线程
    - GC 时会 STW
    - 简单直接
    - 高吞吐
  - 缺点
    - GC 时会存在较明显延迟
  - 运行逻辑
    - 暂停所有业务线程
    - 一个 GC 线程开始清理
    - 清理完成恢复业务线程
- Parallel
  - 特点
    - 多线程 GC
    - GC 时会 STW
    - 提高吞吐量
  - 缺点
    - GC 时会存在较明显延迟
  - 运行逻辑
    - 暂停所有业务线程
    - 多个 GC 线程开始清理
    - 清理完成恢复业务线程
- CMS (Concurrent Mark Sweep)
  - 特点
    - 并发
    - 低停顿
    - 标记-清除
    - 主要用于老年代回收
    - 将比较耗时的标记、清除操作，尽量和业务线程并发处理。
  - 缺点
    - CMS 基于 标记-清除实现，会产生内存碎片。
    - 并发阶段用户线程还在运行，会产生新的垃圾
    - GC 和 业务线程同时运行会抢占 CPU 。
  - 运行逻辑
    - 初始标记 : STW
    - 并发标记 : 用户线程 + GC 线程同时运行
    - 重新标记 : STW，修正并发期间变化的引用
    - 并发清除 : 用户线程 + GC 线程同时运行
- G1 (Garbage First) `现代服务器默认使用`
  - 特点
    - G1 不再把堆物理上固定切成一整块年轻代、一整块老年代，而是把堆切成很多大小相等的小块(Region)
    - 每个 Region 可以扮演不同角色，可以更灵活的回收区域: Eden Region、Survivor Region、Old Region、Humongous Region
    - 可预测停顿时间
  - 运行逻辑·
    - G1 将内存划分为多个大小相同的 Region
    - Region 能够动态扮演不同角色
    - G1 会估算每个 Region 的垃圾回收比例和回收收益
      ```
      Region A：垃圾 80%
      Region B：垃圾 20%
      Region C：垃圾 60%
      ```
    - 根据配置最大停顿时间，G1 会优先回收垃圾最多的 Region。(最大停顿时间仅是目标，无强制约束)
      ```
      -XX:MaxGCPauseMillis=200
      ```
- ZGC
  - 特点
    - 低停顿
    - 并发回收
    - 支持大堆
    - 停顿时间和堆大小关系不大
    - 即使堆很大，也尽量让 STW 时间很短

## 类加载器

### 启动类加载器 BootStrap ClassLoader

- 主要负责加载 Java 核心类库
  ```java
  java.base 模块

  java.lang.String
  java.lang.Object
  java.lang.Class
  java.lang.System
  java.util.ArrayList
  ```


### 平台类加载器 Platform ClassLoader

- 主要加载 Java 平台相关模块，比如部分非核心但仍属于 JDK 平台的模块。

  ```java
  java.sql
  java.xml
  java.desktop
  java.management
  java.naming
  java.security.sasl
  jdk.httpserver
  jdk.crypto.ec
  jdk.jshell
  jdk.management
  ```


### 应用类加载器 Application ClassLoader

- 负责加载 classpath 下的类
- 项目 target/classes 或 build/classes 下的类
- 第三方 jar 包里的类 (maven 依赖、gradle 依赖)

### 双亲委派 (类加载机制)

- 类加载流程

  ```
  1. AppClassLoader 收到：我要加载 com.example.User
  2. AppClassLoader 先问父加载器 PlatformClassLoader
  3. PlatformClassLoader 再问父加载器 BootstrapClassLoader
  4. BootstrapClassLoader 尝试加载，发现不是核心类，加载不了
  5. PlatformClassLoader 尝试加载，发现不是平台类，加载不了
  6. AppClassLoader 最后自己从 classpath 里找
  7. 找到 com/example/User.class，加载成功
  ```

- 设计逻辑
  ```java
  比如你自己写一个：
  package java.lang;

  public class String {
  }

  如果没有双亲委派，应用类加载器可能会先加载你自己写的 java.lang.String。
  有了双亲委派后，加载 java.lang.String 时会优先交给 Bootstrap ClassLoader。Bootstrap 会加载 JDK 自带的 String，你的相同路径 String 就不会替代它。
  ```

- tomcat 为什么要打破 双亲委派
  - tomcat 可部署多个应用，不同项目可能使用不同版本依赖库，若使用双亲委派会导致多个应用共享一套父加载器加载的类，导致依赖相互影响。
  - tomcat 需要不同 web 应用之间，类相互隔离。
  - Web 应用又要共享 Tomcat 容器提供的公共类
  - tomcat 类加载简易流程图

    ```
    Bootstrap ClassLoader
            ↑
    Platform / Extension ClassLoader
            ↑
    Application ClassLoader
            ↑
    Common ClassLoader
            ↑
    WebAppClassLoader(app1)

    Common ClassLoader
            ↑
    WebAppClassLoader(app2)
    
    Common ClassLoader：
    加载 Tomcat 自己和所有 Web 应用共享的类

    WebAppClassLoader：
    加载某个 Web 应用自己的 /WEB-INF/classes 和 /WEB-INF/lib
    
    ```
## JVM 调优

### Jps

- 查看当前机器上的 Java 进程

```shell
jps -l

# 输出
12345 com.example.Application
23456 org.apache.catalina.startup.Bootstrap
```

## jstat

- 查看 JVM 运行时统计信息，尤其是 GC 情况。
```shell
# 每 1000ms 打印一次 12345 这个 Java 进程的 GC 统计
jstat -gc 12345 1000

# YGC：Young GC 次数
# YGCT：Young GC 总耗时
# FGC：Full GC 次数
# FGCT：Full GC 总耗时
# GCT：GC 总耗时
```

## jmap

- 查看堆内存，导出堆转储文件。

```shell
# 看堆配置和使用情况。
jmap -heap 12345

# 导出 dump：
jmap -dump:format=b,file=heap.hprof 12345

# 分析堆内存 对象占用情况 工具
# MAT
# VisualVM
# JProfiler
```

## jstack

- 查看线程栈
- 作用
  - 死锁
  - 线程卡住
  - CPU 飙高
  - 请求阻塞

```shell
jstack 12345

# 使用方法
1. 用 top 找到 CPU 高的 Java 进程 PID
2. 用 top -Hp PID 找到 CPU 高的线程 ID
3. 把线程 ID 转成十六进制(例如线程ID = 12345;十六进制后可能是=0x3039;然后在 jstack 里找：nid=0x3039)
4. 用 jstack PID 导出线程栈
5. 在 jstack 结果里搜索十六进制 nid
6. 看这个线程卡在哪个方法
```


## jcmd

- 比较综合的工具，很多时候可以替代部分 jmap、jstack、jstat。

```shell
jcmd 12345 VM.flags
jcmd 12345 GC.heap_info
jcmd 12345 Thread.print
jcmd 12345 GC.heap_dump heap.hprof
```

## Arthas

- 阿里开源的 Java 诊断工具，线上排查很好用。
- 常用命令

```shell
dashboard：看整体状态
thread：看线程
jad：反编译类
watch：观察方法入参/返回值/异常
trace：查看方法调用耗时
stack：查看方法调用路径
heapdump：导出堆

# 若内存异常上涨; 导出堆
jmap -dump:format=b,file=heap.hprof <pid>

# 使用 MAT、VisualVM、JProfiler 之类工具分析。
# Dominator Tree
# Leak Suspects
# 对象数量异常的类
# 大集合对象，比如 HashMap、ArrayList、ConcurrentHashMap
# GC Roots 到对象的引用链
```


## 解释执行与即时编译 JIT

- JVM 编译运行逻辑 : Java 源码 -> javac 编译 -> .class 字节码 -> JVM 执行

### 解释执行

- 逻辑 : 一条字节码一条字节码地解释运行
- 优点
  - 启动快
  - 不用等编译
- 缺点
  - 长期运行性能不如机器码


### 即时编译 JIT

- 逻辑
  1. JVM 会观察哪些代码经常执行。
  2. JIT 会把这些热点字节码编译成本地机器码，后面再执行就更快。
  3. Java 不是单纯解释执行，而是 解释执行 + JIT 编译优化
- 优化逻辑
  - 方法内联
  - 逃逸分析
    - user 这个对象只在 test() 方法内部使用，没有返回出去，也没有赋给全局变量; JIT 判断对象没有逃出方法作用域，认为没有逃逸；
    - 如果没有逃逸，JVM 可能会进行 缩消除、栈上分配、标量消除优化
      ```java
      public void test() {
          User user = new User();
          user.name = "Alice";
      }

      ```
  - 锁消除
    - StringBuffer 的方法是同步的，有锁。但如果 JIT 分析发现：
    - sb 没有逃出 test() 方法
    - 不可能被其他线程访问那么锁就没有意义，JIT 可能把锁去掉。
    ```java
    public void test() {
        StringBuffer sb = new StringBuffer();
        sb.append("a");
        sb.append("b");
    }
    ```
  - 标量替换
    - 如果对象没有逃逸，JVM 甚至可能不创建完整对象，而是把对象字段拆开。
    ```java
    class Point {
        int x;
        int y;
    }

    public int sum() {
        Point p = new Point();
        p.x = 1;
        p.y = 2;
        return p.x + p.y;
    }

    <!-- 类似替换为 -->
    public int sum() {
        int x = 1;
        int y = 2;
        return x + y;
    }

    ```
  - 循环优化
  - 栈上分配
    - 如果对象没有逃逸，JVM 可能不把它分配到堆上，而是让它随方法栈帧一起销毁。