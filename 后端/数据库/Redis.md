# Redis

## Redis 常用数据结构及场景

数据结构    |	Java    |   中常见用途  |	典型场景
---|---|---|---
String  |	普通缓存、计数器、验证码    |	用户信息缓存、短信验证码、文章阅读数
Hash    	|	存对象	|	用户对象、商品对象、购物车商品明细
List	|	有序列表、队列	|	消息队列、最新消息列表、时间线
Set	|	去重集合	|	点赞用户、抽奖用户、共同好友
Sorted Set / ZSet	|	带分数的排序集合	|	排行榜、热搜榜、延迟队列
Bitmap	|	位运算统计	|	用户签到、活跃用户统计
HyperLogLog	|	大规模去重计数	|	UV 统计、独立访客统计
Geo |   地理位置	附近的人、附近门店
Stream	| 消息流	|	可靠消息队列、异步任务流


- String
    ```java
    key: sms:code:13800138000
    value: 9527
    expire: 5 分钟
    ```
- Hash：用户对象，对象字段频繁读写，不想每次整体序列化
    ```java
    key: user:1001
    field: name -> 张三
    field: age -> 20
    field: city -> 上海
    ```
- List
    ```java
    key: message:queue
    value: ["消息1", "消息2", "消息3"]
    ```
- Set
    ```java
    key: post:1001:likes
    value: {1002, 1003, 1004} // 用户ID集合
    ```
- ZSet
    ```java
    key: leaderboard
    value: {userId:score} // 用户ID和分数的有序集合
    ```
- Bitmap
    ```java
    key: user:1001:sign
    value: [1, 0, 1, 1, 0, 1, 1] // 用户签到记录
    ```
- HyperLogLog
    ```java
    key: uv:2023-10-01
    value: [user_id1, user_id2, user_id3] // 独立访客ID集合
    ```
- Geo
    ```java
    key: user:1001:location
    value: {longitude: 120.1, latitude: 30.2} // 用户地理位置
    ```
- Stream
    ```java
    key: order:stream
    value: {id: "order_001", status: "created"} // 订单消息流
    ```

## Redis 常用命令

- Java 代码示例：

  ```java
  redisTemplate.opsForValue(); // String
  redisTemplate.opsForHash();  // Hash
  redisTemplate.opsForList();  // List
  redisTemplate.opsForSet();   // Set
  redisTemplate.opsForZSet();  // ZSet
  redisTemplate.opsForHyperLogLog(); // HyperLogLog
  redisTemplate.opsForGeo();  // Geo
  redisTemplate.opsForStream(); // Stream
  ```

## java 代码示例

### Hash 结构示例

- Hash 结构类似于 java Map<String, Map<String, String>>

```java
redisTemplate.opsForHash().put("user:1001", "name", "张三");
redisTemplate.opsForHash().put("user:1001", "age", 20);

// 读取单个字段
String name = (String) redisTemplate.opsForHash().get("user:1001", "name");

// 读取整个 Hash 结构
Map<Object, Object> userMap = redisTemplate.opsForHash().entries("user:1001");
```

### List 结构示例

- List 结构类似于 java List<String>
- 是一个双端列表，可以从两端进行插入和删除

```java

// 从左边放入：
redisTemplate.opsForList().leftPush(
    "user:" + userId + ":messages",
    messageJson
);

// 查询最新 10 条：
List<String> messages = redisTemplate.opsForList().range(
    "user:" + userId + ":messages",
    0,
    9
);

// 为了避免列表无限增长，可以只保留最新 100 条：
redisTemplate.opsForList().trim(
    "user:" + userId + ":messages",
    0,
    99
);

// 更推荐阻塞式取任务，这样没有任务时不会疯狂空转：
String task = redisTemplate.opsForList().leftPop(
    "task:queue",
    Duration.ofSeconds(5)
);

```