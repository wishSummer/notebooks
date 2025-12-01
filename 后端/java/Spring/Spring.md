# Spring

重点理解 Spring 的本质：容器、Bean、AOP、事务、资源管理

📌 学习重点

IoC / DI（控制反转 / 依赖注入）

Bean 的创建与生命周期

Bean Scope、Lazy、FactoryBean、ObjectProvider

AOP（切面编程）

动态代理（JDK vs CGLIB）

Advice、Pointcut、JoinPoint

Spring 事务（Tx）

@Transactional 的失效场景

事务传播机制 Propagation

@EnableTransactionManagement 的工作原理

常见核心模块

Environment、PropertySource

Resource / ResourceLoader

ApplicationEvent / Listener

为了真正“深入”，建议按下面顺序学习：

Spring 核心容器（IoC, AOP, 事务）

Spring MVC

Spring Boot

Spring Data / 数据访问

Spring Security → OAuth2 → JWT

Spring Cloud（包括 Gateway）

Spring 源码阅读

综合实践（微服务架构）

## IOC 控制反转

> 对象由 Spring 创建、管理、销毁
> 程序只“要”，不“生产”

### Bean 容器

Spring 的核心只有两个：

- BeanFactory（IoC 容器核心接口）
- ApplicationContext extents BeanFactory（更强大的容器）

### Bean 生命周期（Bean的创建到注销）

- 你需要特别关注：
  BeanPostProcessor

  AOP 底层就是通过它生成代理对象

  @Autowired、@ConfigurationProperties、AOP 全靠它

  BeanDefinition

  Spring 加载 bean 的“元数据”

  自动装配原理全部从这里解析
- Bean 的管理容器

## DI（依赖注入）原理

- Bean 注入方式

    | 注入方式             | 推荐程度  | 说明      |
    | ---------------- | ----- | ------- |
    | 构造器注入            | ⭐⭐⭐⭐⭐ | 最安全、最推荐 |
    | Setter 注入        | ⭐⭐⭐   | 可选      |
    | 字段注入（@Autowired） | ⭐     | 不建议，难测试 |

## Bean 作用域

    | Scope     | 说明          |
    | --------- | ----------- |
    | singleton | 默认，一个容器一个实例 |
    | prototype | 每次获取新实例     |
    | request   | Web 请求作用域   |
    | session   | Web 会话作用域   |

1. @Configuration 与 @Bean 的真正意义
   - @Configuration 的本质：它会被 CGLIB 代理，使 @Bean 方法成为“工厂方法”，从而保证 singleton Bean 不会重复创建。

   ```java
   @Configuration
   public class AppConfig {

     @Bean
     public UserService userService() {
         return new UserService();
     }
   }
   ```

## AOP

## Spring 启动流程

- 流程

```java

refresh() {
    1. prepareRefresh()                         // 环境准备
    2. obtainFreshBeanFactory()                 // 创建 BeanFactory
    3. prepareBeanFactory()                     // 填充基本组件
    4. invokeBeanFactoryPostProcessors()        // 配置类解析、扫描
    5. registerBeanPostProcessors()             // 注册 Bean 级增强器
    6. initMessageSource()                      // 国际化（可忽略）
    7. initApplicationEventMulticaster()        // 事件广播器
    8. onRefresh()                              // 模板方法（如 Web 容器初始化）
    9. registerListeners()                      // 注册监听器
   1.  finishBeanFactoryInitialization()        // 实例化所有单例 Bean
   2.  finishRefresh()                          // 发布事件
}

```

> 源码：`AbstractApplicationContext#refresh()`

## @Scheduled

- fixedDelay 表示在上一次任务完成后，延迟一段时间再执行下一次任务。这个延迟时间可以通过 initialDelay 属性设置。如果未设置 initialDelay，任务将在应用启动后立即执行，并且之后根据 fixedDelay 的设置周期性执行。
- fixedRate 表示在上一次任务开始执行后，一段时间间隔就执行下一次任务。类似于 fixedDelay，initialDelay 属性可以用来在应用启动后延迟执行第一次任务。应用启动后不会立即执行，需搭配initialDelay。
- cron 表达式是一种更加复杂的调度方式，它是基于日历的，可以指定非常灵活的定时规则。应用启动后不会立即执行。
- initialDelay 表示在应用启动后延迟多少时间才执行第一次任务。该属性可以与 fixedDelay 和 fixedRate 一起使用。但不能够和cron一起使用。
