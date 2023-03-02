ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现

主要特点：
1. 多种语言和协议编写客户端。语言: Java, C, C++, C#, Ruby, Perl, Python, PHP。
应用协议: OpenWire,Stomp REST,WS Notification,XMPP,AMQP
2. 完全支持JMS1.1和J2EE 1.4规范 (持久化,XA消息,事务)
3. 对Spring的支持,ActiveMQ可以很容易内嵌到使用Spring的系统里面去,而且也支持Spring2.0的特性
4. 通过了常见J2EE服务器(如 Geronimo,JBoss 4, GlassFish,WebLogic)的测试,其中通过JCA 1.5 			resource adaptors的配置,可以让ActiveMQ可以自动的部署到任何兼容J2EE 1.4 商业服务器上
5. 支持多种传送协议:in-VM,TCP,SSL,NIO,UDP,JGroups,JXTA
6. 支持通过JDBC和journal提供高速的消息持久化
7. 从设计上保证了高性能的集群,客户端-服务器,点对点
8. 支持Ajax
9. 支持与Axis的整合
10. 可以很容易得调用内嵌JMS provider,进行测试


消息的传递有两种类型
	点对点
	发布/订阅模式

JMS定义了五种不同的消息正文格式
	StreamMessage -- Java原始值的数据流
	MapMessage -- 一套名称-值对
	TextMessage -- 一个字符串对象
	ObjectMessage -- 一个序列化的 Java对象
	BytesMessage -- 一个字节的数据流

安装
```shell
使用bin目录下的activemq命令启动：
./activemq start
./activemq start
./activemq status
```
进入管理后台：
http://192.168.25.168:8161/admin
用户名：admin
密码：admin

405问题：
修改hosts文件，配置机器名和127.0.0.1的映射关系。
机器名：/etc/sysconfig/network文件中定义了机器名：
重新启动Activemq的服务


## Queue
### producer
```java

@Test
public void testQueueProducer() throws Exception {
    // 第一步：创建ConnectionFactory对象，需要指定服务端ip及端口号。
    //brokerURL服务器的ip及端口号
    ConnectionFactory connectionFactory = 
    new ActiveMQConnectionFactory("tcp://192.168.25.168:61616");
    // 第二步：使用ConnectionFactory对象创建一个Connection对象。
    Connection connection = connectionFactory.createConnection();
    // 第三步：开启连接，调用Connection对象的start方法。
    connection.start();
    // 第四步：使用Connection对象创建一个Session对象。
    //第一个参数：是否开启事务。true：开启事务，第二个参数忽略。
    //第二个参数：当第一个参数为false时，才有意义。消息的应答模式。
    //1、自动应答2、手动应答。一般是自动应答。
    Session session = 
    connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    // 第五步：使用Session对象创建一个Destination对象（topic、queue），
    //此处创建一个Queue对象。
    //参数：队列的名称。
    Queue queue = session.createQueue("test-queue");
    // 第六步：使用Session对象创建一个Producer对象。
    MessageProducer producer = session.createProducer(queue);
    // 第七步：创建一个Message对象，创建一个TextMessage对象。
    /*TextMessage message = new ActiveMQTextMessage();
    message.setText("hello activeMq,this is my first test.");*/
    TextMessage textMessage = 
    session.createTextMessage("hello activeMq,this is my first test.");
    // 第八步：使用Producer对象发送消息。
    producer.send(textMessage);
    // 第九步：关闭资源。
    producer.close();
    session.close();
    connection.close();
}

```

### Consumer

```java
@Test
public void testQueueConsumer() throws Exception {
    // 第一步：创建一个ConnectionFactory对象。
    ConnectionFactory connectionFactory = 
    new ActiveMQConnectionFactory("tcp://192.168.25.168:61616");
    // 第二步：从ConnectionFactory对象中获得一个Connection对象。
    Connection connection = connectionFactory.createConnection();
    // 第三步：开启连接。调用Connection对象的start方法。
    connection.start();
    // 第四步：使用Connection对象创建一个Session对象。
    Session session = 
    connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    // 第五步：使用Session对象创建一个Destination对象。
    // 和发送端保持一致queue，并且队列的名称一致。
    Queue queue = session.createQueue("test-queue");
    // 第六步：使用Session对象创建一个Consumer对象。
    MessageConsumer consumer = session.createConsumer(queue);
    // 第七步：接收消息。
    consumer.setMessageListener(new MessageListener() {
        @Override
        public void onMessage(Message message) {
            try {
                TextMessage textMessage = (TextMessage) message;
                String text = null;
                //取消息的内容
                text = textMessage.getText();
                // 第八步：打印消息。
                System.out.println(text);
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    });
    //等待键盘输入
    System.in.read();
    // 第九步：关闭资源
    consumer.close();
    session.close();
    connection.close();
}

```

## Topic

### Producer
```java

@Test
public void testTopicProducer() throws Exception {
    // 第一步：创建ConnectionFactory对象，需要指定服务端ip及端口号。
    // brokerURL服务器的ip及端口号
    ConnectionFactory connectionFactory = 
    new ActiveMQConnectionFactory("tcp://192.168.25.168:61616");
    // 第二步：使用ConnectionFactory对象创建一个Connection对象。
    Connection connection = connectionFactory.createConnection();
    // 第三步：开启连接，调用Connection对象的start方法。
    connection.start();
    // 第四步：使用Connection对象创建一个Session对象。
    // 第一个参数：是否开启事务。true：开启事务，第二个参数忽略。
    // 第二个参数：当第一个参数为false时，才有意义。
    // 消息的应答模式。1、自动应答2、手动应答。一般是自动应答。
    Session session = 
    connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    // 第五步：使用Session对象创建一个Destination对象（topic、queue），
    // 此处创建一个topic对象。
    // 参数：话题的名称。
    Topic topic = session.createTopic("test-topic");
    // 第六步：使用Session对象创建一个Producer对象。
    MessageProducer producer = session.createProducer(topic);
    // 第七步：创建一个Message对象，创建一个TextMessage对象。
    /*
     * TextMessage message = new ActiveMQTextMessage(); message.setText(
     * "hello activeMq,this is my first test.");
     */
    TextMessage textMessage = 
    session.createTextMessage("hello activeMq,this is my topic test");
    // 第八步：使用Producer对象发送消息。
    producer.send(textMessage);
    // 第九步：关闭资源。
    producer.close();
    session.close();
    connection.close();
}

```
### Consumer
```java

@Test
public void testTopicConsumer() throws Exception {
    // 第一步：创建一个ConnectionFactory对象。
    ConnectionFactory connectionFactory = 
    new ActiveMQConnectionFactory("tcp://192.168.25.168:61616");
    // 第二步：从ConnectionFactory对象中获得一个Connection对象。
    Connection connection = connectionFactory.createConnection();
    // 第三步：开启连接。调用Connection对象的start方法。
    connection.start();
    // 第四步：使用Connection对象创建一个Session对象。
    Session session = 
    connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    // 第五步：使用Session对象创建一个Destination对象。
    //和发送端保持一致topic，并且话题的名称一致。
    Topic topic = session.createTopic("test-topic");
    // 第六步：使用Session对象创建一个Consumer对象。
    MessageConsumer consumer = session.createConsumer(topic);
    // 第七步：接收消息。
    consumer.setMessageListener(new MessageListener() {
     
        @Override
        public void onMessage(Message message) {
            try {
                TextMessage textMessage = (TextMessage) message;
                String text = null;
                // 取消息的内容
                text = textMessage.getText();
                // 第八步：打印消息。
                System.out.println(text);
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    });
    System.out.println("topic的消费端03。。。。。");
    // 等待键盘输入
    System.in.read();
    // 第九步：关闭资源
    consumer.close();
    session.close();
    connection.close();
}

```
