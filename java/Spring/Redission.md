# Redission

## Redission 定时任务

### RBlockingQueue

### RDelayedQueue

* 方法
  * RDelayedQueue.offer(key, delay, TimeUnit) 添加任务

* RDelayedQueue
  * 本质是一个延迟队列，它由一个 ZSET（Sorted Set）存储元素以及到期时间
    * key : 根据元素信息封装成一个唯一的uuid
    * value : key(offer提供的key)
    * source : (当前时间毫秒值 + 延迟时间)得到的到期时间，并根据当前字段排序
  * 当offer多个相同key的任务，任务之间并不会相互干扰，实际是在延迟队列中生成两个任务。
  * 延迟队列任务到期后并不会刷新到阻塞队列，需要Redission RDelayedQueue实例处理。
  * 一个 目标队列（BlockingQueue 等） 用来存放真正“到期”的元素组成。
* offer(e, delay, unit) 的逻辑大致是：
  * 计算到期时间戳：System.currentTimeMillis() + delay
  * 将 (e, 到期时间戳) 放入 Redis 的 ZSET
  * 到期时由 Redisson 的 watchdog 任务 或 Lua 脚本 将元素移动到目标队列

[Redission 定时任务实例](../业务场景实现实例/Redission-定时任务.md)