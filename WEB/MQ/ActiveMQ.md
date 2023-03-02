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









