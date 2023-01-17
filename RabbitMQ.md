## RabbitMQ



### 概念

* **Message  消息**

  由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key，(路由键)、priority (相对于其他消息的优先权)、delivery-mode (指出该消息可能需要持久性存储)等。

* **Publisher 消息的生产者**

* **Exchange** 

  交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
  Exchange有4种类型: direct(默认), fanout, topic, 和headers,不同类型的Exchange转发消息的策略有所区别

* **Queue**

  逍息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息-直在队列里面，等待消费者连接到这个队列将其取走。

* **Binding**

  绑定，用于消息队列和交换器之间的关联。-个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

  Exchange和Queue的绑定可以是多对多的关系。

* **Connection 网络连接，比如一个TCP连接。**

* **Channel**

  信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP连接。


* **Consumer消息的消费者**
* **Virtual Host**
  虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个vhost本质上就是一个 mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是AMQP概念的基础，必须在连接时指定,
  RabbitMQ默认的vhost是/
* **Broker 表示消息队列服务器实体**

![image-20211110134600888](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211110134600888.png)

![image-20211110134908226](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211110134908226.png)

1、publisher发送message，消息头 + 消息体，发送到Message Broker消息代理服务器上
2、Exchange绑定queue，并确定route key
3、exchange接收消息，根据消息头中的路由键与自己绑定的queue的路由键匹配，根据exchange的类型使用不同的匹配规则将消息发送到 满足需求的queue上
4、消费者监听队列，建立一条连接【连接上开辟了多条信道】
	1）不需要每个请求建立一条连接
	2）不需要与每个队列建立连接
	3）一条 信道 可监听一条 队列

长连接的好处【能实时感知消费者断开，然后保存消息不至于丢失。】

### 安装

```shell
docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management

#自动重启：
docker update rabbitmq --restart=always

4369,25672(Erlang发现&集群端口)
5672,5671(AMQP端口)
15672(web管理后台端口)
61613,61614(STOMP协议端口)
1883,8883(MQTT协议端口)
https://www.rabbitmq.com/networking.html 

登录管理页面
http://192.168.56.10:15672/
guest
guest
```

### 运行机制

**AMQP 中的消息路由**

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP中增加了 **Exchange** 和 **Binding** 的角色 生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送给那个队列



**Exchange 类型**

Exchange 分发消息时根据类型的不同分发策略有区别，目前共四种类型：**direct、tanout、topic、headers** 

header匹配AMQP消息的 header 而不是路由键，headers 交换器和 direct 交换器完全一致，但性能差能多，目前几乎用不到了，所以直接看另外三种类型



**direct**

根据路由键精确匹配  【单播模式】

**fanout**

直接发送到所有队列   【广播模式，不区分路由键】最快

**topic**

发到部分队列

它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。
两个通配符:符号 # 和符号 *    # 匹配0个或多个单词，* 匹配一个单词。

### spring boot整合

```xml
<!--rabbitmq-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: 192.168.56.10
    port: 5672
#    虚拟主机
    virtual-host: /
#    开启发送端抵达队列确认【发送端确认机制+本地事务表】
    publisher-returns: true
#    开启发送确认【发送端确认机制+本地事务表】
    publisher-confirm-type: correlated
#    只要抵达队列，优先回调return confirm
    template:
      mandatory: true
#    使用手动确认模式，关闭自动确认【消息丢失】
    listener:
      simple:
        acknowledge-mode: manual
```

```java
@EnableRabbit 加在启动类上【发送消息可以不需要这个注解，监听消息必须使用这个注解】

RabbitAutoConfiguration 生效，给容器自动配置了很多类
	RabbitTemplate、AmqpAdmin、CachingConnectionFactory、RabbitMessagingTemplate  

接收消息注解：
@RabbitListener(queues={"hello-java-queue"})
@RabbitHandler
```

#### AmqpAdmin

创建Exchanger、Queue、Binding

```java
@Test
void createExchange() {
    // 创建交换机
    amqpAdmin.declareExchange(new DirectExchange("hello-java-exchange", true 是否持久化, false));
    log.info("Exchange创建[{}]成功", "hello-java-exchange");
}

@Test
void createQueue() {
    amqpAdmin.declareQueue(new Queue("hello-java-queue", true 持久化, false 排他, false));
    log.info("Queue创建[{}]成功", "hello-java-queue");
}

@Test
void createBinding() {
    // 创建绑定
    // String destination【目的地，队列name或 交换机name(如果作为路由的话)】
    // Binding.DestinationType destinationType【目的地类型 queue还是exchange(路由)】
    // String exchange【交换机】
    // String routingKey【路由键】
    // @Nullable Map<String, Object> arguments【自定义参数】
    amqpAdmin.declareBinding(new Binding("hello-java-queue", Binding.DestinationType.QUEUE,"hello-java-exchange", "hello.java", null));
    log.info("Binding创建[{}]成功", "hello-java-binding");
}
```

#### RabbitTemplate

发送消息【1、交换机；2、路由键；3、消息】

```java
@Autowired
RabbitTemplate rabbitTemplate;
@Test
void sendMsg() {
    rabbitTemplate.convertAndSend("hello-java-exchange", "hello.java", "hello world");
}
```

对象发送要实现序列化接口

```java
//使用json格式的序列化器
//否则使用jdk的序列化器
@Configuration
public class MyRabbitConfig {
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

#### 接收消息

```java
//必须使用@EnableRabbit
//监听方法必须放在@Component中
//@RabbitListener(queues={"hello-java-queue"})类+方法
//@RabbitHandler：标在方法上【作用：重载处理不同类型的数据】

/**
     * 参数
     *  message 原生消息
     *  T<发送的消息类型>
     *  channel   通道
     *  
     *  多监听 一消费
     *  消费一个 再消费下一个
     */
@RabbitListener(queues = "")
public void listen(Message message, OrderReturnReasonEntity a ,Channel channel){
    message.getMessageProperties();//消息头
    message.getBody();//消息体

}
```

### 消息确认机制 - 可靠到达

- 保证消息不丢失，可靠抵达，可以使用事务消息，性能下降250倍，为此引入确认机制
- **publisher** confirmCallback 确认模式
- **publisher** returnCallback 未投递到 queue 退回
- **consumer** ack 机制

![image-20211110153616505](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211110153616505.png)



发送端确认机制： 两个：
	publisher confirmCallback 确认模式【如果投递到Broker了，回调confirmCallback方法】
	publisher returnCallback未投递到queue 退回模式【如果没有投递到queue，调用returnCallback】

消费端确认机制：【消费者收到消息，给服务器发送确认，服务器删除该消息】
	consumer ack机制（消息确认机制）【让Broker知道哪些消息被消费者正确消费了（如果没有则重新投递）】

1. 默认是自动确认的，只要消息接收到，客户端会自动确认，服务端会移除这个消息
   	BUG：消息丢失，消费者监听队列【所有消息会一次性发送到通道，所以自动确认宕机会导致消息丢失】

2. 手动确认：处理一个确认一个【否则是未签收状态，服务器宕机则会重新进入ready状态不会丢失】
   	参数1：消息下标，参数2：是否批量签收
      	签收：channel.basicAck(message.getMessageProperties().getDeliverTag(), false);

#### 发送端

```java
@Configuration
public class MyRabbitConfig {
    // @Autowired 不要自动注入，会循环依赖
    private RabbitTemplate rabbitTemplate;

    @Primary
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        this.rabbitTemplate = rabbitTemplate;
        rabbitTemplate.setMessageConverter(messageConverter());
        initRabbitTemplate();
        return rabbitTemplate;
    }

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    /**
     * 定制RabbitTemplate
     * 1、服务收到消息就会回调
     *      1、spring.rabbitmq.publisher-confirms: true
     *      2、设置确认回调
     * 2、消息正确抵达队列就会进行回调
     *      1、spring.rabbitmq.publisher-returns: true
     *         spring.rabbitmq.template.mandatory: true
     *      2、设置确认回调ReturnCallback
     *
     * 3、消费端确认(保证每个消息都被正确消费，此时才可以broker删除这个消息)
     *
     */
    // @PostConstruct  //MyRabbitConfig对象创建完成以后，执行这个方法
    public void initRabbitTemplate() {

        /**
         * 确认回调
         * 1、只要消息抵达Broker就ack=true
         * correlationData：当前消息的唯一关联数据(这个是消息的唯一id)
         * ack：消息是否成功收到
         * cause：失败的原因
         */
        // 消息到达broker
        rabbitTemplate.setConfirmCallback((correlationData,ack,cause) -> {
            System.out.println("confirm...correlationData["+correlationData+"]==>ack:["+ack+"]==>cause:["+cause+"]");
        });


        /**
         * 只要消息没有投递给指定的队列，就触发这个失败回调
         * message：投递失败的消息详细信息
         * replyCode：回复的状态码
         * replyText：回复的文本内容
         * exchange：当时这个消息发给哪个交换机
         * routingKey：当时这个消息用哪个路邮键
         */
        // 消息未到达队列
        rabbitTemplate.setReturnCallback((message,replyCode,replyText,exchange,routingKey) -> {
            // 需要修改数据库 消息的状态【后期定期重发消息】
            System.out.println("Fail Message["+message+"]==>replyCode["+replyCode+"]" +
                               "==>replyText["+replyText+"]==>exchange["+exchange+"]==>routingKey["+routingKey+"]");
        });
    }
}
```

#### 接收端

```java
channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);//接收
channel.basicReject(message.getMessageProperties().getDeliveryTag(),true);//拒绝接受 false丢弃/true重新入队
channel.basicNack(message.getMessageProperties().getDeliveryTag(),是否批量,false丢弃/true重新入队)//拒绝接受
    
//multiple：批量签收，requeue：是否重新入队
```



#### 可靠抵达 - ConfirmCallback

```properties
spring.rabbitmq.publisher-confirms=true  
#开启发送端抵达服务器确认【发送端确认机制+本地事务表】
```

在创建 `connectionFactory` 的时候设置 PublisherConfirms(true) 选项，开启 `confirmcallback`。

`CorrelationData` 用来表示当前消息唯一性

消息只要被 broker 接收到就会执行 confirmCallback,如果 cluster 模式，需要所有 broker 接收到才会调用 confirmCallback

被 broker 接收到只能表示 message 已经到达服务器，并不能保证消息一定会被投递到目标 queue 里，所以需要用到接下来的 returnCallback

#### 可靠抵达 - ReturnCallback

```properties
spring.rabbitmq.publisher-retuns=true
#开启发送到队列确认【发送端确认机制+本地事务表】
spring.rabbitmq.template.mandatory=true
#只要抵达队列，优先回调return confirm
```

confirm 模式只能保证消息到达 broker，不能保证消息准确投递到目标 queue 里。在有些模式业务场景下，我们需要保证消息一定要投递到目标 queue 里，此时就需要用到 return 退回模式

这样如果未能投递到目标 queue 里将调用 returnCallback，可以记录下详细到投递数据，定期的巡检或者自动纠错都需要这些数据

#### 可靠抵达 - Ack 消息确认机制

- 消费者获取到消息，成功处理，可以回复Ack给Broker
  - basic.ack 用于肯定确认：broker 将移除此消息
  - basic.nack 用于否定确认：可以指定 beoker 是否丢弃此消息，可以批量
  - basic.reject用于否定确认，同上，但不能批量
- 默认，消息被消费者收到，就会从broker的queue中移除
- 消费者收到消息，默认自动ack，但是如果无法确定此消息是否被处理完成，或者成功处理，我们可以开启手动ack模式
  - 消息处理成功，ack()，接受下一条消息，此消息broker就会移除
  - 消息处理失败，nack()/reject() 重新发送给其他人进行处理，或者容错处理后ack
  - 消息一直没有调用ack/nack方法，brocker认为此消息正在被处理，不会投递给别人，此时客户端断开，消息不会被broker移除，会投递给别人

```yaml
spring:
  rabbitmq:
#    使用手动确认模式，关闭自动确认【消息丢失】
    listener:
      simple:
        acknowledge-mode: manual
```

#### 最终解决方案

确认机制+本地事务表 

1. 发送消息的时候生成消息ID，然后在回调方法里面修改数据库里消息的状态    
2. 定时扫描数据库消息的状态，没有成功的重新投递一次
3. 消费消息时使用手动签收机制【不要使用自动签收】

### 延时队列【定时任务】

**场景:**

比如未付款的订单，超过一定时间后，系统自动取消订单并释放占有物品

**常用解决方案：**

​	Spring的schedule 定时任务轮询数据库

​	**缺点：**消耗系统内存，增加了数据库的压力，存在较大的时间误差

​	**解决：**rabbitmq的消息TTL和死信Exchange结合

![image-20211110162519666](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211110162519666.png)



#### 消息的TTL（Time To Live）

- 消息的TTL 就是**消息的存活时间**，
- RabbitMQ可以对**队列**还有**消息**分别设置TTL
  - 对队列设置就是没有消费者连着的保持时间，**也可以对每一个消息单独的设置，超过了这个时间我们可以认为这个消息他死了，称之为死信**
  - 如果队列设置了，消息也设置了，那么会**取小**，所以一个消息如果被路由到不同的队列中，这个消息死亡时间有可能不一样的（不同队列设置），这里讲的是单个TTL 因为他是实现延时任务的关键，可以通过**设置消息的 expiration 字段或者 x-message-ttl** 来设置时间两者是一样的效果

#### Dead Letter Exchange（DLX）

- 一个消息在满足如下条件下，会进**死信路由**，记住这里是路由不是队列，一个路由可以对应很多队列，（什么是死信）
  - 一个消息被Consumer拒收了，并且reject方法的参数里requeue是false。也就是说不被再次放在队列里，被其他消费者使用。(basic.reject/ basic.nack) 
  - requeue= false上面的消息的TTL到了，消息过期了。
  - 队列的长度限制满了。排在前面的消息会被丢弃或者扔到死信路由上
- Dead Letter Exchange其实就是一种普通的exchange, 和创建其他exchange没有两样。只是在某一个设置 Dead Letter Exchange的队列中有消息过期了自动触发消息的转发，发送到Dead Letter Exchange中去。
- 我们既可以控制消息在一段时间后变成死信， 又可以控制变成死信的消息被路由到某一个指定的交换机， 结合C者，其实就可以实现一个延时队列



* 实现一  队列设置ttl  推荐
* 实现二  消息设置ttl  

![image-20211111104118657](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211111104118657.png)

升级

![image-20211111104134624](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211111104134624.png)

#### 代码实现

SpringBoot可以使用@Bean 来初始化Queue、exchange、Biding等

```java
/**
 * 监听队列信息
 * @param orderEntity
 */
@RabbitListener(queues = "order.release.order.queue")
public void listener(OrderEntity orderEntity, Channel channel, Message message) throws IOException {
    System.out.println("收到过期的订单信息，准备关闭订单" + orderEntity.getOrderSn());
    // 确认接收到消息，不批量接收
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}

/**
 * 容器中的 Binding、Queue、exchange 都会自动创建，(RabbitMQ没有的情况下)
 * 如果存在，不会覆盖。所以如果创建错了要删除，重启服务
 */
@Bean
public Queue orderDelayQueue(){
    // 特殊参数
    Map<String,Object> map = new HashMap<>();
    // 设置交换器
    map.put("x-dead-letter-exchange", "order-event-exchange");
    // 路由键
    map.put("x-dead-letter-routing-key","order.release.order");
    // 消息过期时间
    map.put("x-message-ttl",60000);
    Queue queue = new Queue("order.delay.queue", true, false, false,map);
    return queue;
}

/**
 * 创建队列
 * @return
 */
@Bean
public Queue orderReleaseOrderQueue() {
    Queue queue = new Queue("order.release.order.queue", true, false, false);
    return queue;
}

/**
 * 创建交换机
 * @return
 */
@Bean
public Exchange orderEventExchange() {
    return new TopicExchange("order-event-exchange",true,false);
}

/**
 * 绑定关系 将delay.queue和event-exchange进行绑定
 * @return
 */
@Bean
public Binding orderCreateOrderBingding(){
        return new Binding("order.delay.queue",
                Binding.DestinationType.QUEUE,
                "order-event-exchange",
                "order.create.order",
                null);
}

/**
 * 将 release.queue 和 event-exchange进行绑定
 * @return
 */
@Bean
public Binding orderReleaseOrderBinding(){
    return new Binding("order.release.order.queue",
            Binding.DestinationType.QUEUE,
            "order-event-exchange",
            "order.release.order",
            null);
}
```

### 消息可靠性

#### 消息丢失

- **消息发送出去，由于网络问题没有抵达服务器**
  - 做好容错方法(try-Catch) ，发送消息可能会网络失败，失败后要有重试机制，可记录到数据库，采用定期扫描重发的方式
  - 做好日志记录，每个消息状态是否都被服务器收到都应该记录
  - 做好定期重发，如果消息没有发送成功，定期去数据库扫描未成功的消息进行重发
- **消息抵达Broker, Broker要将消息写入磁盘(持久化)才算成功。此时Broker尚未持久化完成，宕机。**
  - publisher也必须加入确认回调机制，确认成功的消息，修改数据库消息状态。
- **自动ACK的状态下。消费者收到消息，但没来得及消息然后宕机**
  - 一定开启手动ACK，消费成功才移除，失败或者没来得及处理就noAck并重新入队

#### 消息重复

- **消息消费成功，事务已经提交，ack时，机器宕机。导致没有ack成功，Broker的消息重新由unack变为ready,并发送给其他消费者**
- **消息消费失败，由于重试机制，自动又将消息发送出去**
- **成功消费，ack时宕机，消息由unack变为ready, Broker又重新发送**
  - 消费者的业务消费接口应该设计为**幂等性**的。比如扣库存有工作单的状态标志
  - 使用**防重表**(redis/mysql) ，发送消息每一 个都有业务的唯一 标识，处理过就不用处理
  - rabbitMQ的每一个消息都有redelivered字段， 可以获取**是否是被重新投递过来的**，而不是第一次投递过来的

#### 消息积压

- **消费者积压**
- **消费者消费能力不足积压**
- **发送者发送流量太大**
  - 上线更多消费者，进行正常消费
  - 上线专门的队列消费服务，将消息先批量取出来，记录数据库，离线慢慢处理