# OpenFeign 实例模板

* 注意事项
  * 调用方
    * 导入pom依赖。
    * 启动类添加注解。
    * 定义接口方法，调用服务名应和被调用服务名称相同，并被同一注册中心管理。
  * 被调用方
    * 无需导入pom依赖。
    * 服务名应与调用方配置相同。
    * 只要正常写好 REST API 接口（用 @RestController 暴露 HTTP endpoint） 就行。

## 使用模板

* Pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

* Yml 可选配置

```java
feign:
  client:
    config:
      default:
        connectTimeout: 2000   # 连接超时(ms)
        readTimeout: 5000      # 读取超时(ms)
        loggerLevel: FULL      # 日志级别: NONE, BASIC, HEADERS, FULL
```

* Application 启动类添加注解

```java
@SpringBootApplication
@EnableFeignClients(basePackages = "com.example.client")
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

* 定义接口

```java
package com.example.client;

import com.example.dto.UserDTO;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

// name = 服务名（和服务注册中心里一致，比如 Consul/Eureka 里的 serviceId）
// path  = 公共前缀（可选）
@FeignClient(name = "user-service", path = "/users")
public interface UserClient {

    @GetMapping("/{id}")
    UserDTO getUserById(@PathVariable("id") Long id);
}
```

* 服务调用

```java
@Service
public class OrderService {

    private final UserClient userClient;

    public OrderService(UserClient userClient) {
        this.userClient = userClient;
    }

    public void createOrder(Long userId) {
        UserDTO user = userClient.getUserById(userId);
        System.out.println("下单用户：" + user.getName());
    }
}
```
