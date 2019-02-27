代码片段

基本对象

```java
//获取Channel
Channel channel = connection.createChannel();

//声明队列
channel.queueDeclare(QUEUE_NAME, false, false, false, null);

//声明Exchange
channel.exchangeDeclare(EXCHANGE_NAME, "direct");//direct
channel.exchangeDeclare(EXCHANGE_NAME, "topic");//topic
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");//分发



```

需要说明的是Queue,Exchange,定义一次就可以了，所以我们在spring里面定义。



```java
//发送消息
1.发送到哪个交换机上面 2.routkingKey消息类型 3.其他参数 4.消息正文
channel.basicPublish(EXCHANGE_NAME, "goods.delete", null, msgString.getBytes());
```
```java
//接受消息
channel.baskCosume(QUEUE_NAME, autoAck,consumer);
```
```java
//消费者的队列，绑定交换机
1.这个消费者将使用哪个队列 2.队列绑定到哪个交换机 3.接受的消息类型。
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "goods.update");
```

收获
1. 队列声明一次就行了，消费者和生产者其中之一声明了队列也就可以了。
2. Channel是Connection内部的逻辑连接。


名词解释
1. QoS(Quality of Service,服务质量)

2. AMQP协议是基于tcp/ip的协议。