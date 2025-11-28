# Spring Cloud

```
你已经开始接触 Spring Cloud Gateway，可以继续深入：

📌 核心模块

Spring Cloud Gateway（自定义 Filter 原理）

服务注册/发现（Eureka/Nacos/Consul）

Spring Cloud LoadBalancer

OpenFeign 调用与重试机制

Resilience4j（熔断、限流、隔离仓）

Spring Cloud Config / Bus
```

	服务注册与发现：Consul、Nacos
	配置中心：Consul config、Nacos
	负载均衡：Ribbon（已停止更新）、Loadbalancer
	服务远程调用:OpenFeign
	熔断器：Hystrix、Sentinel
	网关：Gateway
	链路追踪：Sleuth
	事务：Seate
	消息总线：Spring cloud Bus
	Spring Cloud是一系列框架的有序集合，利用Spring Boot特性简化了分布式系统基础设施的开发，如：服务发现注册、配置中心、消息中心、消息总线、负载均衡、断路器、数据监控等。最终组合出一套简单易懂、易部署、易维护的分布式系统开发工具包。
	### 优点：
		1. 复杂可控
		2. 独立部署
		3. 技术选型灵活
		4. 容错高
		5. 可扩展
## Bootstrap应用程序上下文
	Spring Cloud应用程序通过创建“ bootstrap ”上下文来运行，该上下文是主应用程序的父上下文。它负责从外部源加载配置属性，并负责解密本地外部配置文件中的属性。这两个上下文共享一个Environment，它是任何Spring应用程序的外部属性的来源。默认情况下，引导程序属性（不是bootstrap.properties，而是引导程序阶段加载的属性）具有较高的优先级，因此它们不能被本地配置覆盖。
# Spring Cloud Commons：通用抽象
	服务发现，负载平衡和断路器之类的模式将它们带到一个通用的抽象层，可以由所有Spring Cloud客户端使用，而与实现无关（例如，使用Eureka或Consul进行的发现） ）。
## @EnableDiscoveryClient
	SpringCloudCommons提供了@EnableDiscoveryClient注解。这将寻找META-INF/spring.factories与DiscoveryClient接口的实现。DiscoveryClient的实现在org.springframework.cloud.client.discovery.EnableDiscoveryClient键下将配置类添加到spring.factories。DiscoveryClient实现的示例包括Spring Cloud Netflix Eureka，Spring Cloud Consul发现和Spring Cloud Zookeeper发现。

	默认情况下，DiscoveryClient的实现会自动将本地Spring Boot服务器注册到远程发现服务器。可以通过在@EnableDiscoveryClient中设置autoRegister=false来禁用此行为。
### DiscoveryClient实例
	DiscoveryClient接口扩展了Ordered。当使用多个发现客户端时，允许定义返回的发现客户端的顺序，类似于如何注册由Spring应用程序加载的beans。默认情况下，任何DiscoveryClient的顺序都设置为0。如果要为自定义DiscoveryClient实现设置不同的顺序，则只需覆盖getOrder()方法，以便它返回适合您的设置的值。除此之外，您可以使用属性来设置Spring Cloud提供的DiscoveryClient实现的顺序，其中包括ConsulDiscoveryClient，EurekaDiscoveryClient和ZookeeperDiscoveryClient。为此，您只需要将spring.cloud.{clientIdentifier}.discovery.order（对于Eureka，则为eureka.client.order）属性设置为所需的值。
## 服务注册
	自定义注册：Commons现在提供一个ServiceRegistry接口，该接口提供诸如register(Registration)和deregister(Registration)之类的方法，这些方法使您可以提供自定义的注册服务。Registration是标记界面。
	自动注册
	
	ServiceRegistry自动注册：默认情况下，ServiceRegistry实现会自动注册正在运行的服务。要禁用该行为，可以设置：* @EnableDiscoveryClient(autoRegister=false)以永久禁用自动注册。* spring.cloud.service-registry.auto-registration.enabled=false通过配置禁用行为。
	ServiceRegistry自动注册Events
	服务自动注册时将触发两个事件。注册服务之前会触发名为InstancePreRegisteredEvent的第一个事件。注册服务后，将触发名为InstanceRegisteredEvent的第二个事件。您可以注册ApplicationListener，以收听和响应这些事件。

	[注意]
	如果将spring.cloud.service-registry.auto-registration.enabled设置为false，则不会触发这些事件
	
### 服务注册表执行器端点
	Spring Cloud Commons提供了一个/service-registry执行器端点。该端点依赖于Spring应用程序上下文中的Registration bean。使用GET调用/service-registry会返回Registration的状态。对具有JSON正文的同一终结点使用POST会将当前Registration的状态更改为新值。JSON正文必须包含带有首选值的status字段。请参阅更新状态时用于允许值的ServiceRegistry实现的文档以及为状态返回的值。例如，Eureka的受支持状态为UP，DOWN，OUT_OF_SERVICE和UNKNOWN。
	
## Spring RestTemplate作为负载均衡器客户端
	RestTemplate可以自动配置为在后台使用负载均衡器客户端。要创建负载均衡的RestTemplate，请创建RestTemplate @Bean并使用@LoadBalanced限定符，如以下示例所示：

	@Configuration
	public class MyConfiguration {

		@LoadBalanced
		@Bean
		RestTemplate restTemplate() {
			return new RestTemplate();
		}
	}

	public class MyClass {
		@Autowired
		private RestTemplate restTemplate;

		public String doOtherStuff() {
			String results = restTemplate.getForObject("http://stores/stores", String.class);
			return results;
		}
	}
	[警告]	警告
	RestTemplate bean不再通过自动配置创建。各个应用程序必须创建它。
	URI需要使用虚拟主机名（即服务名，而不是主机名）。Ribbon客户端用于创建完整的物理地址。有关如何设置RestTemplate的详细信息，请参见RibbonAutoConfiguration。
	
### 重试失败的请求
	可以配置负载均衡的RestTemplate以重试失败的请求。默认情况下，禁用此逻辑。您可以通过在应用程序的类路径中添加Spring重试来启用它。负载平衡的RestTemplate遵循与重试失败的请求有关的某些Ribbon配置值。您可以使用client.ribbon.MaxAutoRetries，client.ribbon.MaxAutoRetriesNextServer和client.ribbon.OkToRetryOnAllOperations属性。如果要通过对类路径使用Spring重试来禁用重试逻辑，则可以设置spring.cloud.loadbalancer.retry.enabled=false。
	
## 多个RestTemplate对象
	如果您想要一个RestTemplate不是负载均衡的，请创建一个RestTemplate bean并注入它。要访问负载均衡的RestTemplate，请在创建@Bean时使用@LoadBalanced限定符，如以下示例所示：
	@Configuration
	public class MyConfiguration {

		@LoadBalanced	//负载均衡
		@Bean
		RestTemplate loadBalanced() {
			return new RestTemplate();
		}

		@Primary
		@Bean
		RestTemplate restTemplate() {
			return new RestTemplate();
		}
	}

	public class MyClass {
	@Autowired
	private RestTemplate restTemplate;

		@Autowired
		@LoadBalanced
		private RestTemplate loadBalanced;

		public String doOtherStuff() {
			return loadBalanced.getForObject("http://stores/stores", String.class);
		}

		public String doStuff() {
			return restTemplate.getForObject("https://example.com", String.class);
		}
	}
# Spring Cloud Config