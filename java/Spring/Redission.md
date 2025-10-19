# Redission

## Redission 实现定时任务

> 若程序异常退出，已刷新到阻塞队列的任务会丢失，但在延迟队列到期的任务不会丢失，在程序启动后会立即搬运到阻塞队列进行消费。

- pom.xml 依赖

```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson-spring-boot-starter</artifactId>
  <version>3.27.2</version>
</dependency>
```

- config

```java
import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RedissonConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Value("${spring.redis.password:}") // 可选
    private String redisPassword;

    @Bean(destroyMethod = "shutdown")
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.useSingleServer()
              .setAddress("redis://" + redisHost + ":" + redisPort)
              .setPassword(redisPassword.isEmpty() ? null : redisPassword);
        return Redisson.create(config);
    }
}

```

- Service 添加任务

```java
@Service
@RequiredArgsConstructor
public class WhiteListService {

    private final RedissonClient redissonClient;

    // 添加任务到redis 延迟队列方法
    private void scheduleWhiteListExpiryTasks() {
        // 获取 阻塞队列
        RBlockingQueue<Integer> blockingQueue = redissonClient.getBlockingQueue("whiteListExpireQueue");
        // 监听 延迟队列，若存在到期任务，则刷新到阻塞队列
        RDelayedQueue<Integer> delayedQueue = redissonClient.getDelayedQueue(blockingQueue);

        long delay = Duration.between(LocalDateTime.now(), wl.getEndAt()).toMillis();
        delayedQueue.offer(key, delay, TimeUnit.MILLISECONDS);
    }
}


```

- Event 任务触发

```java
@Component
@RequiredArgsConstructor
public class WhiteListExpireConsumer {

    private final RedissonClient redissonClient;

    @EventListener(ApplicationReadyEvent.class)
    public void startConsume() {
        new Thread(() -> {
            /** 获取阻塞队列，用于获取任务*/
            RBlockingQueue<Integer> blockingQueue = redissonClient.getBlockingQueue("whiteListExpireQueue");
            /** 获取延迟队列，确保程序启动时立刻开始监听延迟队列中是否存在到期任务，并刷新到阻塞队列进行消费*/
            RDelayedQueue<Integer> delayedQueue = redissonClient.getDelayedQueue(blockingQueue);
            while (true) {
                try {
                    Integer id = blockingQueue.take(); // 阻塞直到任务到期，获取到期任务。
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "white-list-expire-consumer").start();
    }
}
```
