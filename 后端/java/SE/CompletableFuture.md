# CompletableFuture

> `CompletableFuture` 是 Java 8 引入的一个强大的异步编程工具，位于 java.util.concurrent 包中，用于处理异步任务、组合任务以及异常处理。

- 核心原理
  - 异步执行：任务可以在独立的线程（通常由线程池如 `ForkJoinPool.commonPool()` 执行）运行，主线程无需等待任务完成。
  - 完成触发：任务完成后，结果或异常会存储在 `CompletableFuture` 中，触发后续的回调操作。
  - 灵活组合：支持链式调用、并行执行、组合多个任务等，适合复杂异步流程。
- 关键方法
  - 创建异步任务：
    - `CompletableFuture.runAsync(Runnable, Executor)`：执行无返回值的异步任务。
    - `CompletableFuture.supplyAsync(Supplier<T>, Executor)`：执行有返回值的异步任务。
  - 链式调用
    - `CompletableFuture.thenApply(Function<T, U>)`
      - 用于简单的结果转换，函数返回普通值（非 `CompletableFuture`）。
    - `CompletableFuture.thenCompose(Function<T, CompletableFuture<R>>)`
      - 用于处理返回 `CompletableFuture` 的函数，自动“展平”结果，避免嵌套的 `CompletableFuture`。
    - `CompletableFuture.thenAccept(Consumer<T>)`
      - 用于处理结果，但无返回值。
      - 方法由调用线程执行。
    - `CompletableFuture.thenAcceptAsync(Consumer<T>)`
    - 用于处理结果，但无返回值。
    - 方法由子线程异步执行。
    - 任务完成
      - `CompletableFuture.complete(T value)`：手动完成任务，设置结果。
      - `CompletableFuture.completeExceptionally(Throwable ex)`：手动标记任务异常。
    - 异常处理
      - `CompletableFuture.handle(BiFunction<T, Throwable, R>)`：成功或失败均执行,统一处理结果和异常。
      - `CompletableFuture.exceptionally(Function<Throwable, T>)`：仅异常时执行,仅处理异常，提供默认值。
    - 获取结果
      - `get()`：阻塞等待结果（不推荐）。
      - `join()`：类似 get()，但抛出未检查异常。
      - `getNow(T defaultValue)`：立即获取结果或默认值。
    - 组合多个任务：
      - `CompletableFuture.allOf(futures)`
      - `CompletableFuture.anyOf(futures)`
    - 合并多个任务结果
      - `.thenCombine`
      - `.thenAcceptBoth`：用于合并两个任务的结果。
    - 设置超时
      - `CompletableFuture.orTimeout(3, TimeUnit.SECONDS)`
  - 关于 `join()` 阻塞获取结果

    ```java
    /**
     * 1. 等所有任务完成后再处理结果；若有一个任务未完成则阻塞主线程。
     * 2. 若某一任务出现异常，可以更早的集中捕获所有异常，或进行统一超时处理。
     * 3. 相当于执行所有任务，若某一任务未结束，则阻塞主线程；
     **/
    CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();

    /**
     * 1. 若仅对每个任务单独join()。当某个任务长时间未响应则会阻塞后续任务处理。无法达到全部任务同步执行的效果。需要先使用上面的AllOf().join()。
     * 2. 若某一任务出现异常，则可能中断整体任务的执行。
     * 3. 推荐先使用一阻塞所有任务，再使用join()处理执行结果。
     * 4. 相当于遍历所有任务逐个阻塞获取执行结果，若某一未执行完毕，则阻塞到当前步骤。
     **/
    futures.stream()
    .map(CompletableFuture::join)
    .filter(Objects::nonNull)
    .collect(Collectors.toList());

    ```

  - 最佳实践
    - 优先使用线程池：通过 `supplyAsync(Supplier, Executor)` 指定自定义线程池（如 `Executors.newFixedThreadPool`），避免默认 `ForkJoinPool` 资源竞争。
    - 避免阻塞调用：尽量使用链式调用和回调（如 `.thenAccept`），避免 `get()` 或 `join()`。
    - 合理处理异常：总是使用 `.handle` 或 `.exceptionally` 捕获异常，避免未处理的异常导致程序失败。
    - 注意任务编排：使用 `.thenCompose` 展平嵌套的 `CompletableFuture`，提高代码可读性。
    - 资源管理：在高并发场景下，监控线程池大小和任务负载，避免资源耗尽。
    - ComplateFuture 阻塞多异步任务获取所有结果

## 示例模板

### 学习示例

```java
import java.util.concurrent.CompletableFuture;

public class CompletableFutureChaining {
    public static void main(String[] args) {
        // 模拟异步获取用户ID
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
                throw new RuntimeException("Task failed!");
            }
            return 123;
        });

        // 使用 thenApply 转换结果
        CompletableFuture<String> userNameFuture = future.thenApply(id -> {
            return "User-" + id; // 简单转换
        });

        // 使用 thenCompose 进行异步查询用户名
        CompletableFuture<String> userDetailsFuture = future.thenCompose(id -> {
            return CompletableFuture.supplyAsync(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "Details of User-" + id; // 异步操作
            });
        });

        
        // 使用 exceptionally 处理异常
        CompletableFuture<String> exceptionallyFuture = future.exceptionally(throwable -> {
            System.out.println("Exceptionally caught: " + throwable.getMessage());
            return "Recovered Value";
        });

        // 使用 handle 处理结果或异常
        CompletableFuture<String> handleFuture = future.handle((result, throwable) -> {
            if (throwable != null) {
                System.out.println("Handle caught: " + throwable.getMessage());
                return "Handled Value";
            }
            return result;
        });

        // 输出结果
        exceptionallyFuture.thenAccept(result -> System.out.println("Exceptionally Result: " + result));
        handleFuture.thenAccept(result -> System.out.println("Handle Result: " + result));


        // 输出结果
        userNameFuture.thenAccept(name -> System.out.println("Name: " + name));
        userDetailsFuture.thenAccept(details -> System.out.println("Details: " + details));

        // 等待完成
        exceptionallyFuture.join();
        handleFuture.join();
        userNameFuture.join();
        userDetailsFuture.join();
    }
}
```

### 全局线程池配置最佳实践示例

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Executor;

// 用户信息
record User(int id, String name) {}

// 订单
record Order(int orderId, double amount) {}

// 用户资料
record UserProfile(User user, List<Order> orders) {}

// 二次封装的 DTO
record UserProfileDTO(
    int userId,
    String userName,
    List<Order> orders,
    LocalDateTime timestamp, // 额外字段
    String status // 额外字段
) {}

/**
 * 服务层：异步查询用户和订单
*/
@Service
class UserService {
    // 自定义线程池（通过 Spring 配置）
    @Autowired
    private Executor taskExecutor;

    @Async
    public CompletableFuture<User> fetchUserAsync(int userId) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000); // 模拟网络延迟
                if (userId < 0) {
                    throw new IllegalArgumentException("Invalid user ID");
                }
                return new User(userId, "User-" + userId);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new RuntimeException("User fetch interrupted", e);
            }
        }, taskExecutor);
    }

    @Async
    public CompletableFuture<List<Order>> fetchOrdersAsync(int userId) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000); // 模拟网络延迟
                return Arrays.asList(
                    new Order(userId * 10 + 1, 100.0),
                    new Order(userId * 10 + 2, 200.0)
                );
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new RuntimeException("Order fetch interrupted", e);
            }
        }, taskExecutor);
    }

    // 二次封装逻辑
    private UserProfileDTO encapsulateProfile(User user, List<Order> orders) {
        return new UserProfileDTO(
            user.id(),
            user.name(),
            orders,
            LocalDateTime.now(), // 添加时间戳
            orders.isEmpty() ? "No Orders" : "Has Orders" // 添加状态
        );
    }
}

/**
 * 控制器：处理前端请求
*/
@RestController
@EnableAsync
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/user-profile")
    public CompletableFuture<UserProfile> getUserProfile(@RequestParam int userId) {
        return userService.fetchUserAsync(userId)
            .thenCompose(user -> userService.fetchOrdersAsync(user.id())
                // 异步二次封装
                .thenApplyAsync(orders -> userService.encapsulateProfile(user, orders))
            )
            .handle((profile, throwable) -> {
                if (throwable != null) {
                    // 记录异常（生产环境使用日志框架）
                    System.err.println("Error: " + throwable.getMessage());
                    // 返回默认值或抛出自定义异常
                    return new UserProfile(new User(-1, "Unknown"), List.of());
                }
                return profile;
            });
    }
}

/**
 * Spring Boot 配置：自定义线程池
*/
@Configuration
class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(4); // 核心线程数
        executor.setMaxPoolSize(8);  // 最大线程数
        executor.setQueueCapacity(100); // 队列容量
        executor.setThreadNamePrefix("Async-Thread-"); // 线程名前缀
        executor.initialize();
        return executor;
    }
}
```

### 局部线程池，批量请求列表示例

  ```java
  import jakarta.annotation.PreDestroy;
  import org.springframework.scheduling.annotation.Async;
  import org.springframework.scheduling.annotation.EnableAsync;
  import org.springframework.stereotype.Service;

  import java.time.LocalDateTime;
  import java.util.List;
  import java.util.concurrent.CompletableFuture;
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  import java.util.stream.Collectors;

  @Service
  @EnableAsync
  public class ApiService {
      // 局部线程池
      private final ExecutorService executor = Executors.newFixedThreadPool(8);

      // 模拟异步调用外部 API
      @Async
      public CompletableFuture<ApiResult> callApiAsync(String id) {
          return CompletableFuture.supplyAsync(() -> {
              try {
                  // 模拟 API 调用延迟
                  Thread.sleep(1000);
                  if (id == null || id.isEmpty()) {
                      throw new IllegalArgumentException("Invalid ID: " + id);
                  }
                  // 模拟返回数据
                  return new ApiResult(id, "Data for " + id);
              } catch (InterruptedException e) {
                  Thread.currentThread().interrupt();
                  throw new RuntimeException("API call interrupted for ID: " + id, e);
              }
          }, executor).orTimeout(2, java.util.concurrent.TimeUnit.SECONDS); // 设置超时
      }

      // 二次封装单个结果
      private ResultDTO encapsulateResult(ApiResult result, String status) {
          return new ResultDTO(
              result.id(),
              result.data(),
              status,
              LocalDateTime.now()
          );
      }

    // 二次封装失败结果
    private ResultDTO encapsulateError(String id, Throwable throwable) {
        return new ResultDTO(
            id,
            null,
            "Failed: " + throwable.getMessage(),
            LocalDateTime.now()
        );
    }

    // 批量异步调用 API 并汇总
    public CompletableFuture<ApiResponseDTO> processApiCalls(List<String> ids) {
        // 将每个 ID 映射为异步任务
        List<CompletableFuture<ResultDTO>> futures = ids.stream()
            .map(id -> callApiAsync(id)
                // 异步处理每个结果的封装
                .thenApplyAsync(result -> encapsulateResult(result, "Success"), executor)
                // 单独处理每个任务的异常
                .exceptionally(throwable -> encapsulateError(id, throwable))
            )
            .collect(Collectors.toList());

        // 等待所有任务完成并汇总
        return CompletableFuture.allOf(futures.toArray(CompletableFuture<?>[]::new))
        // allOf().join() 会阻塞当前线程； thenApplyAsync() 开启异步，子线程调用join() 并不会影响整体性能；
            .thenApplyAsync(v -> {
                // 收集所有结果
                List<ResultDTO> results = futures.stream()
                    .map(CompletableFuture::join) // 此时 allOf 已确保完成，无阻塞
                    .collect(Collectors.toList());
                // 判断整体状态
                boolean allSuccess = results.stream()
                    .allMatch(dto -> dto.status().startsWith("Success"));
                String overallStatus = allSuccess ? "All Success" : "Partial Failure";

                // 返回汇总 DTO
                return new ApiResponseDTO(results, ids.size(), overallStatus);
            }, executor)
            // 整体异常处理（兜底）
            .handle((response, throwable) -> {
                if (throwable != null) {
                    System.err.println("Error processing API calls: " + throwable.getMessage());
                    return new ApiResponseDTO(List.of(), ids.size(), "Failed");
                }
                return response;
            });
    }

    // 关闭线程池
    @PreDestroy
    public void shutdownExecutor() {
        executor.shutdown();
    }
}
```
