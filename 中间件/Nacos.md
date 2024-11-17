# docker环境下搭建Nacos
> 需要先保证数据库连接，以及nacos基础表创建。
* docker-compose.yml
```shell
version: "3.0"
services:
  nacos:
    image: nacos/nacos-server
    container_name: nacos-server
    env_file:
      - ./env/custom-application-config.env
    volumes:
      - ./standalone-logs/:/home/nacos/logs
      - ./init.d/application.properties:/home/nacos/conf/application.properties
    ports:
      - "8848:8848"
      - "9848:9848"
    restart: on-failure
networks:
  default:
    external:
      name: default-overlay
```
* `custom-application-config.env`
```shell
PREFER_HOST_MODE=hostname
MODE=standalone
SPRING_DATASOURCE_PLATFORM=mysql
NACOS_AUTH_IDENTITY_KEY=2222
NACOS_AUTH_IDENTITY_VALUE=2xxx
```
* `nacos` application.properties 添加配置`mysql`环境变量  
  ```yml
  // MYSQL_HOST：mysql在docker-compose.yml 服务名
  spring.datasource.url=jdbc:mysql://${MYSQL_HOST}:${MYSQL_PORT}/nacos
  spring.datasource.username=root
  spring.datasource.password=root
  ```

# Nacos：Dynamic Naming and Configeration Service

    - 服务发现和服务健康监测
    - 动态配置服务
    - 动态DNS服务
    - 服务及其元数据管理

## Nacos服务器

    - 使用Nacos、Consul等服务发现、配置管理的分布式中间件，需要下载并启动对应的服务器。
    - bootstrap.yml中nacos.config.server-addr配置的路径后添加/nacos，访问Nacos配置中心。
    - 进入nacos\bin\，运行startup.cmd 文件开启服务器。若是单片机则运行  startup.cmd -m standalone
    - Nacos设置配置文件存储读取数据库https://blog.csdn.net/qq_43743882/article/details/119139469

## 注意事项

```
- 启动类上标注：@EnableDiscoveryClient ，注册服务
- 配置中心添加的配置文件名称以及对应的ApplicationName不能够以 - 作为间隔符，与配置文件声明规则冲突。
- ApplicationName不能够以 _ 作为间隔符，负载均衡找不到服务。

```

# OpenFeign

    - 声明式客户端服务调用
    - 调用负载均衡

## 注意事项

    - 启动类上标注：@EnableFeignClients
    - 请求调用类标注：@FeignClient(Value="")  value值为服务名
    - @LoadBalanced标注在方法上，方法上使用@RequestMapping（""）指定请求接口

## 超时控制

OpenFeign默认超时时间为1s，超过1s就会返回错误页面。如果我们的接口处理业务确实超过1s,就需要对接口进行超时配置，如下：

```java
ribbon: #设置feign客户端连接所用的超时时间，适用于网络状况正常情况下，两端连接所用时间
  ReadTimeout: 1000 #指的是建立连接所用时间
  ConnectTimeout: 1000 #指建立连接后从服务读取到可用资源所用时间
```

## 日志增强

    1. yml配置

```java
logging:
  level:
    com.yuyue.online.springcloud.consumer81.service.TestService: debug #指定openfeign日志以什么级别监控哪个接口（可多个）
```

    2. 配置日志bean

    NONE：默认，不显示任何日志；
    BASIC: 仅记录请求方法、URL、响应状态码及执行时间；
    HEADERS：除了BASIC中定义的信息之外，还有请求头和响应头信息；
    FULL：除了HEADERS中定义的信息之外，还有请求的正文和响应数据。

```java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenFeignConfig {
    @Bean
    Logger.Level feignLogLevel(){
        return Logger.Level.FULL;
    }
}
```

