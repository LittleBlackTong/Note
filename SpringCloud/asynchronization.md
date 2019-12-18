# 异步 和 MQ
### 一、异步的常见形态

- 通知
- 请求/异步响应
- 消息 （一对多，消息被感兴趣的服务消费）

### 二、MQ应用场景
- 异步处理
- 流量削峰
- 日志处理
- 应用解耦

### 三、RabbitMq的基本使用
1.生产者 添加依赖

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-amqp</artifactId>
</dependency>
```
2.声明 amqpTemplate 

```
@Autowired
private AmqpTemplate amqpTemplate;
```
3.调用 convertAndSend 方法生产消息

```
amqpTemplate.convertAndSend("pushExchange","like",System.currentTimeMillis()+":like");

```
> 其中 `pushexchange` 是队列交换器 `like` 是绑定队列的 routing key


4.消费者 添加依赖

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-amqp</artifactId>
</dependency>
```

5.声明 processMessage 方法进行消息消费

```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-amqp</artifactId>
</dependency>
```

6.声明 processMessage 方法进行消息消费

```
@RabbitListener(bindings = @QueueBinding(
	exchange = @Exchange("pushExchange"), //声明交换器
	key = "comment", //交换器与路由绑定的 routing key
	value = @Queue("commentQueue") //声明队列的名称
    ))
public void processQueueComments(String message) {
	log.info("commentReceiver message is {}", message);
}
```
