---
title: 'Redis'
category: '知识库'
status: 'reference'
organized: 2026-07-10
---

> [[Redisson]]
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

- 常用命令

|   命令    |	含义    |	Java 对应   |
|   ----    |  ---- |   ----    |
|   LPUSH	|   左侧插入	|   leftPush()  |
|   RPUSH	|   右侧插入	|   rightPush() |
|   LPOP	|   左侧弹出	|   leftPop()   |
|   RPOP	|   右侧弹出    |	rightPop()  |
|   LRANGE	|   获取范围    |	range() |
|   LTRIM	|   裁剪列表    |	trim()  |
|   LLEN	|   获取长度    |	size()  |
|   BLPOP	|   阻塞左弹出  |	leftPop(key, timeout)   |

### Set 结构示例

- 元素不重复
- 无序


```java

// set添加元素
redisTemplate.opsForSet().add(
    "article:" + articleId + ":likes",
    userId.toString()
);


// 判断是否存在
Boolean isExist = redisTemplate.opsForSet().isMember(
    "article:" + articleId + ":likes",
    userId.toString()
);

// 删除元素

redisTemplate.opsForSet().remove(
    "article:" + articleId + ":likes",
    userId.toString()
);

// 统计set长度
Long count = redisTemplate.opsForSet().size(
    "article:" + articleId + ":likes"
);

// 随机获取set内某个值 不删除
String winner = redisTemplate.opsForSet().randomMember("lottery:2026");

// 随机获取set内某个值并删除
String winner = redisTemplate.opsForSet().pop("lottery:2026");

// 查询两个set 的交集
Set<String> commonFriends = redisTemplate.opsForSet().intersect(
    "user:1:friends",
    "user:2:friends"
);

// 求两个 set 并集并去重
redisTemplate.opsForSet().union("set:a", "set:b");

// 求两个set 差集
redisTemplate.opsForSet().difference("set:a", "set:b");

```

- 常用命令

|   命令    |	含义    |	Java 对应   |
| ----  |   ----    |   ----    |
|   SADD    |	添加元素    |	add()   |
|   SREM    |	删除元素    |	remove()    |
|   SISMEMBER	|   判断是否存在    |	isMember()  |
|   SCARD	|   集合大小    |	size()  |
|   SMEMBERS	|   获取所有元素	|   members()   |
|   SINTER	|   交集	|   intersect() |
|   SUNION	|   并集    |	union() |
|   SDIFF	|   差集    |	difference()    |
|   SRANDMEMBER |	随机取元素  |	randomMember()  |
|   SPOP    |	随机弹出元素    |	pop()   |


### ZSet

- 元素不重复
- 每个元素都有一个 score，Redis 会根据 score 从小到大排序

```java

// 添加元素
redisTemplate.opsForZSet().add(
    "rank:article:view",
    "article:" + articleId,
    count
);


// 增加元素的 score
redisTemplate.opsForZSet().incrementScore(
    "rank:article:view",
    "article:" + articleId,
    1
);

// 倒序获取指定排名的元素
Set<String> topArticles = redisTemplate.opsForZSet().reverseRange(
    "rank:article:view",
    0,
    9
);

// 正序获取指定排名的元素
Set<String> topArticles = redisTemplate.opsForZSet().range(
    "rank:article:view",
    0,
    9
);

// 倒序获取指定排名的元素 和 score
Set<ZSetOperations.TypedTuple<String>> topArticles =
    redisTemplate.opsForZSet().reverseRangeWithScores(
        "rank:article:view",
        0,
        9
);

// 获取元素的倒序排名
Long rank = redisTemplate.opsForZSet().reverseRank(
    "rank:article:view",
    "article:" + articleId
);

// 获取某个元素的分数 score
Double score = redisTemplate.opsForZSet().score(
    "rank:article:view",
    "article:" + articleId
);


```

- 常用命令

|   命令	|   含义	|   Java 对应   |
|   ----    |   ----    |   ----    |
|   ZADD	|   添加元素和分数  |	add()   |
|   ZINCRBY |   增加分数    |	incrementScore()    |
|   ZREVRANGE	|   从高到低取范围	|   reverseRange()  |
|   ZRANGE	|   从低到高取范围	|   range() |
|   ZREVRANK	|   查询倒序排名	|   reverseRank()   |
|   ZRANK   |	查询正序排名	|   rank()  |
|   ZSCORE	|   查询分数	|   score() |
|   ZREM	|   删除元素	|   remove()    |


### Bitmap

- 适合给用户做签到场景
- bitmap 是一种基于 String 的位操作
- bitmap 存储的是一组位置连续的数据结构， 如果存储两个值  位置分别是  0， 9999。 那么占用空间会根据最大位置数去使用，若 场景 offset 不连贯，会导致大量空间浪费。
- Bitmap 由 key + offset + value 组成。
  - key 表示一组数据
  - offset 表示这组数据的位置。 
  - value 取值为 ： 0|1

```java

// 指定用户key ： 日期，  位置（从0开始），true; 记录用户 2026-06 第三天已签到
redisTemplate.opsForValue().setBit(
    "sign:" + userId + ":2026-06",
    2,
    true
);


// 获取 用户 2026 06 第三天的值
Boolean signed = redisTemplate.opsForValue().getBit(
    "sign:" + userId + ":2026-06",
    2
);
```

### HyperLogLog

- 适合大规模uv统计
- 可以用来做 “去重计数”，但结果结果是 近似统计
- 可以用于计算网站的访客数量

```java

// 添加用户访问记录
redisTemplate.opsForHyperLogLog().add(
    "uv:2026-06-16",
    userId.toString()
);

// 统计 uv
Long uv = redisTemplate.opsForHyperLogLog().size("uv:2026-06-16");
```

### Geo

- 用于存储地理位置

```java

// 添加地理位置
redisTemplate.opsForGeo().add(
    "shop:geo", // key
    new Point(121.4737, 31.2304), // 门店 经纬度
    "shop:1001" // 门店 id
);

// 查询附近5公里门店
Distance distance = new Distance(5, Metrics.KILOMETERS);

Circle circle = new Circle(
    new Point(121.4737, 31.2304),
    distance
);

GeoResults<RedisGeoCommands.GeoLocation<String>> results =
    redisTemplate.opsForGeo().radius("shop:geo", circle); 

```


### Stream

- 较于 List 更适合做可靠消息队列
- 支持： 消息Id、消息持久化、消费组、ack确认、pending 未确认消息、多消费者协助

```java

// 添加消息
Map<String, String> message = Map.of(
    "orderId", "1001",
    "event", "PAID"
);

redisTemplate.opsForStream().add(
    "order:events",
    message
);

// 创建消费组
redisTemplate.opsForStream().createGroup(
    "order:events",
    "order-group"
);

// 消费者获取消息 
// MapRecord<S,K,V>
// S：Stream 的 key 类型，比如 "order:events"，通常是 String
// K：消息里的 field 类型，比如 "orderId"、"event"
// V：消息里的 value 类型，比如 "1001"、"PAID"
List<MapRecord<String, Object, Object>> records =
    redisTemplate.opsForStream().read(
        Consumer.from("order-group", "consumer-1"),
        StreamOffset.create("order:events", ReadOffset.lastConsumed())
    );


```