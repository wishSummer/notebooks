# Spring Event

## Spring Event 概念

> Spring Event（Application Event）是 Spring 容器内的事件发布/监听机制，用于在系统内部解耦业务流程。

- 核心作用：
  - 解耦：发布者不需要知道监听者是谁
  - 扩展性强：可以新增监听逻辑而不影响主流程
  - 支持异步执行（业务常用）
  - 支持事务事件（提交成功后再触发）
- 设计概念
  - 事件
    - `ApplicationEvent`: 事件消息
  - 事件监听器
    - `ApplicationListener`: 监听事件并处理
  - 事件发布者
    - `ApplicationEventPublisher`: 发布事件
    - `ApplicationContext extends ApplicationEventPublisher`: 使用子类也可发布事件
  - 事件管理者
    - `ApplicationEventMulticaster`: `Spring` 默认提供的事件广播器；广播其会在容器刷新过程中进行实例化、初始化
  
## Spring Event 的基本组件

1. `Event`（事件）：普通 `POJO` 即可（`Spring Boot 2.x / 3.x` 无需继承 `ApplicationEvent`。）

    ```java
    public class UserRegisterEvent {
        private final Long userId;
        private final String username;

        public UserRegisterEvent(Long userId, String username) {
            this.userId = userId;
            this.username = username;
        }

        public Long getUserId() { return userId; }
        public String getUsername() { return username; }
    }
    ```

2. `Publisher`（事件发布者）通过 `ApplicationEventPublisher` 发布事件。

    ```java
    @Service
    public class UserService {

        private final ApplicationEventPublisher publisher;

        public UserService(ApplicationEventPublisher publisher) {
            this.publisher = publisher;
        }

        public void register(String username) {
            Long userId = 1001L;

            // 发布事件
            publisher.publishEvent(new UserRegisterEvent(userId, username));
        }
    }
    ```

3. `Listener`（事件监听器）异步消费消息

```java
/**
 * 创建配置开启@EnableAsync
 */
@Configuration
@EnableAsync
public class AsyncConfig {

    /**可选，自定义线程池 */
    @Bean("eventExecutor")
    public Executor eventExecutor() {
        ThreadPoolTaskExecutor exec = new ThreadPoolTaskExecutor();
        exec.setCorePoolSize(5);
        exec.setMaxPoolSize(20);
        exec.setQueueCapacity(50);
        exec.setThreadNamePrefix("event-");
        exec.initialize();
        return exec;
    }

}


/**
 * 监听事件，异步消费消息
 */
@Component
public class AsyncEmailListener {

    @Async
    // @Async("eventExecutor") 配合自定义线程池（可选）
    @EventListener
    // @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT) 若想要事件在事务提交后执行（避免数据还没落库就发通知）
    public void handle(UserRegisterEvent event) {
        System.out.println("【异步】发送邮件给：" + event.getUsername());
    }
}

```

## 注意事项

1. 使用 `@Async` 必须加 `@EnableAsync`
2. 必须通过 `Spring` 代理调用：

    ```java
    @Autowired
    private TestService service;

    this.asyncMethod(); // ❌完全不会异步
    service.asyncMethod(); // ✔ OK
    ```

3. 事件是容器内广播，不会跨服务