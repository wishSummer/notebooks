# RabbitMQ 参考：https://blog.csdn.net/kavito/article/details/91403659
	Producer -> connection -> exchange -> Queue -> connection -> Consumer
	生产者生产消息到交换机，由交换机将消息根据规则分发到消息队列，供消费者消费。
	
## AMQP和JMS
	MQ是消息通信的模型，并发具体实现。现在实现MQ的有两种主流方式：AMQP、JMS。

	两者间的区别和联系：

	JMS是定义了统一的接口，来对消息操作进行统一；AMQP是通过规定协议来统一数据交互的格式

	JMS限定了必须使用Java语言；AMQP只是协议，不规定实现方式，因此是跨语言的。

	JMS规定了两种消息模型；而AMQP的消息模型更加丰富

	JMS是针对JAVA的MQ具体接口实现，而AMQP则是定义了一套适用于多语言的规范。

## 常见MQ产品
	ActiveMQ：基于JMS

	RabbitMQ：基于AMQP协议，erlang语言开发，稳定性好

	RocketMQ：基于JMS，阿里巴巴产品，目前交由Apache基金会

	Kafka：分布式消息系统，高吞吐量

## MQ角色分工
	Broker：消息队列服务进程，此进程包括两个部分：Exchange和Queue
	Exchange：消息队列交换机，按一定的规则将消息路由转发到某个队列，对消息进行过虑。
	Queue：消息队列，存储消息的队列，消息到达队列并转发给指定的
	Producer：消息生产者，即生产方客户端，生产方客户端将消息发送
	Consumer：消息消费者，即消费方客户端，接收MQ转发的消息。

## 基础消息队列实现
	生产者向规定好的消息队列发送消息，多个消费者读取同一队列进行消费

### 未嵌入框架代码实现（未绑定交换机,队列模式）
	生产者：

        // 1、获取到连接
        Connection connection = ConnectionUtil.getConnection();
        // 2、从连接中创建通道，使用通道才能完成消息相关的操作
        Channel channel = connection.createChannel();
        // 3、声明（创建）队列
        //参数：String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        /**
         * 参数明细
         * 1、queue 队列名称
         * 2、durable 是否持久化，如果持久化，mq重启后队列还在
         * 3、exclusive 是否独占连接，队列只允许在该连接中访问，如果connection连接关闭队列则自动删除,如果将此参数设置true可用于临时队列的创建
         * 4、autoDelete 自动删除，队列不再使用时是否自动删除此队列，如果将此参数和exclusive参数设置为true就可以实现临时队列（队列不用了就自动删除）
         * 5、arguments 参数，可以设置一个队列的扩展参数，比如：可设置存活时间
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 4、消息内容
        String message = "Hello World!";
        // 向指定的队列中发送消息
        //参数：String exchange, String routingKey, BasicProperties props, byte[] body
        /**
         * 参数明细：
         * 1、exchange，交换机，如果不指定将使用mq的默认交换机（设置为""）
         * 2、routingKey，路由key，交换机根据路由key来将消息转发到指定的队列，如果使用默认交换机，routingKey设置为队列的名称
         * 3、props，消息的属性
         * 4、body，消息内容
         */
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println(" [x] Sent '" + message + "'");
        
        //关闭通道和连接(资源关闭最好用try-catch-finally语句处理)
        channel.close();

	消费者：
		// 获取到连接
        Connection connection = ConnectionUtil.getConnection();
        //创建会话通道,生产者和mq服务所有通信都在channel通道中完成
        Channel channel = connection.createChannel();
        // 声明队列
        //参数：String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        /**
         * 参数明细
         * 1、queue 队列名称
         * 2、durable 是否持久化，如果持久化，mq重启后队列还在
         * 3、exclusive 是否独占连接，队列只允许在该连接中访问，如果connection连接关闭队列则自动删除,如果将此参数设置true可用于临时队列的创建
         * 4、autoDelete 自动删除，队列不再使用时是否自动删除此队列，如果将此参数和exclusive参数设置为true就可以实现临时队列（队列不用了就自动删除）
         * 5、arguments 参数，可以设置一个队列的扩展参数，比如：可设置存活时间
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
		
		//设置每个消费者只能处理一条消息，该设置在手动模式下生效。（达到能者多劳效果）
		channel.basicQos(1);
		
        //实现消费方法
        DefaultConsumer consumer = new DefaultConsumer(channel){
            // 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
            /**
             * 当接收到消息后此方法将被调用
             * @param consumerTag  消费者标签，用来标识消费者的，在监听队列时设置channel.basicConsume
             * @param envelope 信封，通过envelope
             * @param properties 消息属性
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //交换机
                String exchange = envelope.getExchange();
                //消息id，mq在channel中用来标识消息的id，可用于确认消息已接收
                long deliveryTag = envelope.getDeliveryTag();
                // body 即消息体
                String msg = new String(body,"utf-8");
                System.out.println(" [x] received : " + msg + "!");
				
				// 手动进行ACK消息确认
				//channel.basicAck(envelope.getDeliveryTag,false);

            }
        };
        
        // 监听队列，第二个参数：是否自动进行消息确认。
        //参数：String queue, boolean autoAck, Consumer callback
        /**
         * 参数明细：
         * 1、queue 队列名称
         * 2、autoAck 自动回复，当消费者接收到消息后要告诉mq消息已接收，如果将此参数设置为tru表示会自动回复mq，如果设置为false要通过编程实现回复
         * 3、callback，消费方法，当消费者接收到消息要执行的方法
         */
		 //手动进行ACK消息确认
        //channel.basicConsume(QUEUE_NAME, false, consumer);
        channel.basicConsume(QUEUE_NAME, true, consumer);

## 订阅模式Publish/Subscribe
	1、一个生产者多个消费者
	2、每个消费者都有一个自己的队列
	3、生产者没有将消息直接发送给队列，而是发送给exchange(交换机、转发器)
	4、每个队列都需要绑定到交换机上，由交换机根据规则分配给队列
	5、生产者发送的消息，经过交换机到达队列，实现一个消息被多个消费者消费
	Exchanges：交换机一方面：接收生产者发送的消息。另一方面：知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。

	Exchange类型有以下几种：

	Fanout：广播，将消息交给所有绑定到交换机的队列

	Direct：定向，把消息交给符合指定routing key 的队列

	Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列
	#：匹配一个或多个词
	*：匹配不多不少恰好1个词

	Header：header模式与routing不同的地方在于，header模式取消routingkey，使用header中的 key/value（键值对）匹配队列。

	Header模式不展开了，感兴趣可以参考这篇文章https://blog.csdn.net/zhu_tianwei/article/details/40923131
	Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失
	


	
### 未嵌入框架代码实现（绑定交换机）
	生产者：声明交换机
		
		// 获取到连接
        Connection connection = ConnectionUtil.getConnection();
        // 获取通道
        Channel channel = connection.createChannel();
        // 声明exchange，指定类型为fanout
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
		
		// 声明exchange，指定类型为direct
        //channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

		//topic 匹配方式
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

        
        // 消息内容
        String message = "注册成功！！";
        // 发布消息到Exchange
        channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
		
		 // 发送消息，并且指定routing key 为：sms，只有短信服务能接收到消息
        channel.basicPublish(EXCHANGE_NAME, "sms", null, message.getBytes());
		
		 // 发送消息，并且指定routing key为：quick.orange.rabbit
        //channel.basicPublish(EXCHANGE_NAME, "quick.orange.rabbit", null, message.getBytes());
       
        System.out.println(" [生产者] Sent '" + message + "'");
        channel.close();
        connection.close();
	消费者：声明队列，并绑定交换机
	
		private final static String QUEUE_NAME = "fanout_exchange_queue_sms";//短信队列
	 
		private final static String EXCHANGE_NAME = "test_fanout_exchange";
	 
		public static void main(String[] argv) throws Exception {
			// 获取到连接
			Connection connection = ConnectionUtil.getConnection();
			// 获取通道
			Channel channel = connection.createChannel();
			// 声明队列
			channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	 
			// 绑定队列到交换机Fanout
			channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
			
			// 绑定队列到交换机，同时指定需要订阅的routing key。可以指定多个Direct
			channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "email");//指定接收发送方指定routing key为sms的消息
			
			//topic 匹配方式
			//channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "*.orange.*");
	 
			// 定义队列的消费者
			DefaultConsumer consumer = new DefaultConsumer(channel) {
				// 获取消息，并且处理，这个方法类似事件监听，如果有消息的时候，会被自动调用
				@Override
				public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
										   byte[] body) throws IOException {
					// body 即消息体
					String msg = new String(body);
					System.out.println(" [短信服务] received : " + msg + "!");
				}
			};
			// 监听队列,自动返回完成
			channel.basicConsume(QUEUE_NAME, true, consumer);

## Spring 整合rabbitMQ

	application.yml
	server:
	  port: 10086  
	spring:
	  application:
		name: mq-rabbitmq-producer
	  rabbitmq:
		host: 192.168.1.103
		port: 5672
		username: kavito
		password: 123456
		virtualHost: /kavito
		template:
		  retry:	//失败重试
			enabled: true	//开启失败重试
			initial-interval: 10000ms	//第一次重试的间隔时长
			max-interval: 300000ms		//最长重试间隔，超过这个间隔将不再重试
			multiplier: 2		//下次重试间隔的倍数，此处是2即下次重试间隔是上次的2倍
		  exchange: topic.exchange	//缺省的交换机名称，此处配置后，发送消息如果不指定交换机就会使用这个
		publisher-confirms：生产者确认机制，确保消息会正确发送，如果发送失败会有错误回执，从而触发重试

	config配置类
	@Configuration
	public class RabbitmqConfig {
		public static final String QUEUE_EMAIL = "queue_email";//email队列
		public static final String QUEUE_SMS = "queue_sms";//sms队列
		public static final String EXCHANGE_NAME="topic.exchange";//topics类型交换机
		public static final String ROUTINGKEY_EMAIL="topic.#.email.#";
		public static final String ROUTINGKEY_SMS="topic.#.sms.#";
	 
		//声明交换机
		@Bean(EXCHANGE_NAME)
		public Exchange exchange(){
			//durable(true) 持久化，mq重启之后交换机还在
			return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(true).build();
		}
	 
		//声明email队列
		/*
		 *   new Queue(QUEUE_EMAIL,true,false,false)
		 *   durable="true" 持久化 rabbitmq重启的时候不需要创建新的队列
		 *   auto-delete 表示消息队列没有在使用时将被自动删除 默认是false
		 *   exclusive  表示该消息队列是否只在当前connection生效,默认是false
		 */
		@Bean(QUEUE_EMAIL)
		public Queue emailQueue(){
			return new Queue(QUEUE_EMAIL);
		}
		//声明sms队列
		@Bean(QUEUE_SMS)
		public Queue smsQueue(){
			return new Queue(QUEUE_SMS);
		}
	 
		//ROUTINGKEY_EMAIL队列绑定交换机，指定routingKey
		@Bean
		public Binding bindingEmail(@Qualifier(QUEUE_EMAIL) Queue queue,
									@Qualifier(EXCHANGE_NAME) Exchange exchange){
			return BindingBuilder.bind(queue).to(exchange).with(ROUTINGKEY_EMAIL).noargs();
		}
		//ROUTINGKEY_SMS队列绑定交换机，指定routingKey
		@Bean
		public Binding bindingSMS(@Qualifier(QUEUE_SMS) Queue queue,
								  @Qualifier(EXCHANGE_NAME) Exchange exchange){
			return BindingBuilder.bind(queue).to(exchange).with(ROUTINGKEY_SMS).noargs();
		}
	 
	}
	
	生产者：
	
	@SpringBootTest
	@RunWith(SpringRunner.class)
	public class Send {
	 
		@Autowired
		RabbitTemplate rabbitTemplate;
		
		@Test
		public void sendMsgByTopics(){
	 
			/**
			 * 参数：
			 * 1、交换机名称
			 * 2、routingKey
			 * 3、消息内容
			 */
			for (int i=0;i<5;i++){
				String message = "恭喜您，注册成功！userid="+i;
				rabbitTemplate.convertAndSend(RabbitmqConfig.EXCHANGE_NAME,"topic.sms.email",message);
				System.out.println(" [x] Sent '" + message + "'");
			}
	 
		}
	}

	消费者：
	
	@Component
	public class ReceiveHandler {
	 
		//监听邮件队列
		@RabbitListener(bindings = @QueueBinding(
				value = @Queue(value = "queue_email", durable = "true"),
				exchange = @Exchange(
						value = "topic.exchange",
						ignoreDeclarationExceptions = "true",
						type = ExchangeTypes.TOPIC
				),
				key = {"topic.#.email.#","email.*"}))
		public void rece_email(String msg){
			System.out.println(" [邮件服务] received : " + msg + "!");
		}
	 
		//监听短信队列
		@RabbitListener(bindings = @QueueBinding(
				value = @Queue(value = "queue_sms", durable = "true"),
				exchange = @Exchange(
						value = "topic.exchange",
						ignoreDeclarationExceptions = "true",
						type = ExchangeTypes.TOPIC
				),
				key = {"topic.#.sms.#"}))
		public void rece_sms(String msg){
			System.out.println(" [短信服务] received : " + msg + "!");
		}
	}
	- 属性说明： 

	- @Componet：类上的注解，注册到Spring容器

	- @RabbitListener：方法上的注解，声明这个方法是一个消费者方法，需要指定下面的属性：

	- bindings：指定绑定关系，可以有多个。值是@QueueBinding的数组。@QueueBinding包含下面属性：

	- value：这个消费者关联的队列。值是@Queue，代表一个队列

	- exchange：队列所绑定的交换机，值是@Exchange类型

	- key：队列和交换机绑定的RoutingKey，可指定多个