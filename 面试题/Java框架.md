# Java 框架

## 数据操作框架

### Hibernate 和 Mybatis 的区别

- 相同点
  - 都属于 ORM 框架
  - 都是对 JDBC 的包装
  - 都属于持久层框架

- 不同点
  - Hibernate 是面向对象的，Mybatis 是面向 sql 的
  - Hibernate 查询映射实体对象必须全字段查询
  - Hibernate 是全自动的 ORM ， Mybatis 是半自动额 ORM
  - Hibernate 支持级联操作， Mybatis 则没有
  - Hibernate 编写 HQL(Hibernate Query Language)  查询数据库大大减低了对象和数据库的耦合性，Mybatis 提供动态 SQL，需要手写 SQL， 与数据库之间的耦合度取决于程序员所写的 SQL 的方法，所以 Hibernate 的移植性大于 Mybatis
  - Hibernate 有方言跨数据库， Mybatis 依赖于具体的数据库
  - Hibernate 拥有完整的日志系统， Mybatis 则相对欠缺

- Mybatis 的优点
  - 基于 SQL 编程，相对灵活，不会对应用程序或数据库现有设计造成影响
  - SQL 写在 XML 里，解除 SQL 与程序代码的耦合，便于统一管理
  - 提供XML 标签，支持编写动态 SQL 语句，并可重用
  - 与 JDBC 相比，冗余代码相对较少，不需要手动开关连接
  - 与各种数据库有较好的兼容性（支持 JDBC 的数据库都能够兼容）
  - 能够与 Spring 较好的集成
  - 提供映射标签，支持对象与数据库的 ORM 字段关系映射

- Mybatis 的缺点
  - SQL 语句的编写工作量较大，当字段较多、关联表多时，编写语句有一定难度
  - SQL 语句依赖于数据库，导致数据库移植性差，不能够随意更换数据库
  
### MyBatis 框架使用场景

1. 专注 SQL 本身
2. 对性能有较高要求，或需求变化较多的项目

### Mybatis 是如何将 SQL 执行结果封装为目标对象的？有哪些形式？

1. 使用 <resultMap> 标签，建立映射关系
2. 使用 SQL 列的别名功能，建立关系

## Spring 框架

### Spring 中 beanFactory 和 ApplicationContext 的联系和区别

> beanFactory 是 Spring 中较为原始的 Factory, 无法支持 Spring 的许多插件，如 AOP、Web应用等。
> ApplicationContext 接口是通过 beanFatory 接口派生而来的，除了具备 beanFactory 接口的功能外，还具备资源访问、事件传播、国际化消息访问等功能。

1. 使用 ApplicationContext 配置 Bean 默认配置是 singleton, 无论是否使用，都会被实例化。优点是预先加载，缺点是浪费内存。
2. 使用 Beanfactory 实例化对象时，配置的 bean 等到使用的时候才会被实例化。优点是节约内存，缺点是速度比较慢。
3. 没有特殊情况下，应该使用具备更多功能的 ApplicationContex。

### Spring IOC 有几种注入方式

- 构造器注入
- set 方法注入
- 字段注入

- xml 注入
- 注解注入
- config 配置类注入

### Spring MVC 工作流程

1. 用户发送 HTTP 请求
2. 请求进入 DispatcherServlet（前端控制器）
3. DispatcherServlet 查找对应的 Handler（Controller 方法）
4. 通过 HandlerAdapter 调用 Controller
5. Controller 处理业务逻辑，返回 ModelAndView / 数据
6. DispatcherServlet 交给 ViewResolver 解析视图
7. 渲染视图（或直接返回 JSON）
8. 返回 HTTP 响应给客户端

### Java 动态代理

| 对比点 | JDK 动态代理 | CGLIB |
| ------ | -------- | ------------- |
| 实现原理  | 基于接口     | 基于继承（生成子类）    |
| 是否需要接口 | 必须有接口    | 不需要接口         |
| 生成方式  | Java 反射  | 字节码生成（ASM）    |
| 代理对象类型 | 接口类型     | 目标类类型         |
| 方法限制   | 只能代理接口方法 | 不能代理 final 方法 |
| 类限制    | 无        | 不能是 final 类   |

## Redis

### 什么是 Redis 持久化， RDB 和 AOF 比较
> 持久化就是把内存的数据歇到磁盘中去，防止服务宕机后内存数据丢失。

1. AOF 文件比 RDB 更新频率高，优先使用 AOF 还原数据
2. AOF 比 RDB 更安全也更大
3. RDB 性能比 AOF 好

### Redis 适用场景

1. 会话缓存
2. 队列
3. 计数器、排行榜
4. 发布、订阅

### Redis 事务

- 可以确保运行命令的原子性操作

### Redis 的淘汰策略

| 策略           | 行为              |
| ------------ | --------------- |
| noeviction   | 不淘汰，直接报错        |
| allkeys-lru  | 淘汰最近最少使用        |
| allkeys-lfu  | 淘汰最不常用          |
| volatile-lru | 只淘汰设置了过期时间的 key |
| volatile-ttl | 淘汰即将过期的 key     |

### Redis 如何保证处理速度

1. redis 是单线程 k-v 内存型可持久化数据库
2. redis 单行程模式别面了线程的上线问切换的损耗
3. redis 采用 IO 多路复用技术，可以很好解决多请求并发的问题

