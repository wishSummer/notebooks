# mqtt
		mqtt是一种基于*发布/订阅*模式的’轻量级‘通讯协议，构建于TCP/IP协议之上。mqtt的优势在于用极少的代码和有限的带宽，为远程设备提供实时可靠的消息服务。在物联网中很常见。
		### mqtt协议特点
			- mqtt使用的是发布/订阅消息模式; Publisher --> mqtt-server --> Subscriber 消息分发机制，由mqtt-server将Publisher发布的message根据Topic分发给Subscriber。
			- QoS（Quality of Service levels）
				- QoS 0 这一级别会发生消息丢失或重复，消息发布依赖于底层TCP/IP网络。
				- QoS 1 消息将至少传送一次给订阅者，可能会重复传输。
				- QoS 2 保证消息仅传送到目的地一次。为此，带有唯一消息ID的消息会存储两次，首先来自发送者，然后是接收者。QoS 级别 2 在网络中具有最高的开销，因为在发送方和接收方之间需要两个流。
		### mqtt数据包结构
			- 固定头（Fixed header），存在于所有MQTT数据包中，表示数据包类型及数据包的分组类标识；
			- 可变头（Variable header），存在于部分MQTT数据包中，数据包类型决定了可变头是否存在及其具体内容；
			- 消息体（Payload），存在于部分MQTT数据包中，表示客户端收到的具体内容；
				- CONNECT，消息体内容主要是：客户端的ClientID、订阅的Topic、Message以及用户名和密码
				- SUBSCRIBE，消息体内容是一系列的要订阅的主题以及QoS。
				- SUBACK，消息体内容是服务器对于SUBSCRIBE所申请的主题及QoS进行确认和回复。
				- UNSUBSCRIBE，消息体内容是要订阅的主题。