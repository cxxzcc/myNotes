## kafka

Kafka是一个分布式的基于发布/订阅模式的消息队列

Kafka是一个开源的分布式事件流平台



两种模式

* 点对点
* 发布/订阅
  * 一个topic多个partitiotn
  * 配合分区, 提出消费者组, 组内每个消费者并行消费
  * 分区会有多个副本
  * ZK记录leader    2.8.0之后不需要ZK

![image-20220315174437588](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220315174437588.png)



### 入门

#### 基本概念

•       Broker : 和AMQP里协议的概念一样， 就是消息中间件所在的服务器

•       Topic(主题) : 每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）

•       Partition(分区) : Partition是物理上的概念，体现在磁盘上面，每个Topic包含一个或多个Partition.

•       Producer : 负责发布消息到Kafka broker

•       Consumer : 消息消费者，向Kafka broker读取消息的客户端。

•       Consumer Group（消费者群组） : 每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。

•       offset 偏移量： 是kafka用来确定消息是否被消费过的标识，在kafka内部体现就是一个递增的数字 



#### 配置

server.properties

```properties
#服务唯一标识,不能重复
broker.id=0
#修改log路径
log.dirs=/tmp/kafka-logs
# 组测ZK 到kafka目录下
zookeeper.connect=aaa:2181,bbb:2181/kafka
```

配置环境变量

```shell
vim /etc/profile.d/my_env.sh

#KAFKA HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin


source /etc/profile
```

#### 启动

```shell
bin/kafka-server-start.sh -daemon config/server.properties
```

```shell
#! /bin/bash
case $1 in
"start")
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "--- 启动$i kafka ---"
		ssh $i "/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties
	done
;;
"stop")
	for i in hadoop102 hadoop103 hadoop104 
	do
		echo "--- 停止$i kafka ---"
		ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh "
	done
;;
esac
```

#### 命令

kafka-topics.sh

```shell
--bootstrap-server <String: server toconnect to> 连接的 Kafka Broker主机名称和端口号
--topic <String: topic>    操作的topic名称
--create创建主题。
--delete删除主题。
--alter修改主题。
--list查看所有主题。
--describe查看主题详细描述。
--partitions <Integer: #of partitions>  设置分区数。
--replication-factor<Integer: replication factor>设置分区副本。
--config <String: name=value>更新系统默认的配置。
```

kafka-console-producer.sh

```shell
bin/kafka-console-producer.sh --bootstrap-server hadoop102:9092 --topic first
```

kafka-console-consumer.sh

```shell
kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first
--from-beginning #从头消费数据
```

#### kafka优缺点

优点: 基于磁盘的数据存储 高伸缩性 高性能 应用场景 :  收集指标和日志  提交日志  流处理

缺点: 运维难度大 偶尔有数据混乱的情况 对zookeeper强依赖 多副本模式下对带宽有一定要求

#### server 参数解释

```
log.dirs: 日志文件存储地址， 可以设置多个

num.recovery.threads.per.data.dir：用来读取日志文件的线程数量，对应每一个log.dirs 若此参数为2 log.dirs 为2个目录 那么就会有4个线程来读取

auto.create.topics.enable:是否自动创建tiopic

num.partitions: 创建topic的时候自动创建多少个分区 (可以在创建topic的时候手动指定)

log.retention.hours: 日志文件保留时间 超时即删除

log.retention.bytes: 日志文件最大大小

log.segment.bytes: 当日志文件达到一定大小时，开辟新的文件来存储(分片存储)

log.segment.ms: 同上 只是当达到一定时间时 开辟新的文件

message.max.bytes: broker能接收的最大消息大小(单条) 默认1M
```

#### kafka基本命令

```shell
##列出所有主题 
kafka-topics.bat --zookeeper localhost:2181/kafka --list

##列出所有主题的详细信息 
kafka-topics.bat --zookeeper localhost:2181/kafka --describe

##创建主题 主题名 my-topic，1副本，8分区 
kafka-topics.bat --zookeeper localhost:2181/kafka --create --replication-factor 1 --partitions 8 --topic my-topic

##增加分区，注意：分区无法被删除 
kafka-topics.bat --zookeeper localhost:2181/kafka --alter --topic my-topic --partitions 16

##删除主题 
kafka-topics.bat --zookeeper localhost:2181/kafka --delete --topic my-topic

##列出消费者群组（仅Linux） 
kafka-topics.sh --new-consumer --bootstrap-server localhost:9092/kafka --list

##列出消费者群组详细信息（仅Linux） 
kafka-topics.sh --new-consumer --bootstrap-server localhost:9092/kafka --describe --group 群组名 

```



### 生产者

#### 参数详解

```
acks:
 至少要多少个分区副本接收到了消息返回确认消息 一般是 0:只要消息发送出去了就确认(不管是否失败) 1:只要 有一个broker接收到了消息 就返回 all： 所有集群副本都接收到了消息确认 当然 2 3 4 5 这种数字都可以， 就是具体多少台机器接收到了消息返回， 但是一般这种情况很少用到

buffer.memory： 
 生产者缓存在本地的消息大小 ： 如果生产者在生产消息的速度过快 快过了往 broker发送消息的速度 那么就会出现buffer.memory不足的问题 默认值为32M 注意 单位是byte 大概3355000左右

max.block.ms:
 生产者获取kafka元数据(集群数据，服务器数据等) 等待时间 ： 当因网络原因导致客户端与服务器通讯时等待的时间超过此值时 会抛出一个TimeOutExctption 默认值为 60000ms

retries：
 设置生产者生产消息失败后重试的次数 默认值 3次

retry.backoff.ms:
 设置生产者每次重试的间隔 默认 100ms

batch.size:
 生产者批次发送消息的大小 默认16k 注意单位还是byte

linger.ms:
 生产者生产消息后等待多少毫秒发送到broker 与batch.size 谁先到达就根据谁 默认值为0

compression.type：
 kafka在压缩数据时使用的压缩算法 可选参数有:none、gzip、snappy none即不压缩 gzip,和snappy压缩算法之间取舍的话 gzip压缩率比较高 系统cpu占用比较大 但是带来的好处是 网络带宽占用少， snappy压缩比没有gzip高 cpu占用率不是很高 性能也还行， 如果网络带宽比较紧张的话 可以选择gzip 一般推荐snappy

client.id:
 一个标识， 可以用来标识消息来自哪， 不影响kafka消息生产 

max.in.flight.requests.per.connection：
 指定kafka一次发送请求在得到服务器回应之前,可发送的消息数量


```



#### 发送原理

在消息发送的过程中，涉及到了两个线程main线程和Sender线程。在main线程中创建了一个双端队列RecordAccumulator.

main 线程将消息发送给RecordAccumulator,Sender线程不断从RecordAccumulator中拉取消息发送到Kafka Broker。

![image-20220316165118877](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220316165118877.png)

#### 异步发送

消息发送到队列即返回

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.1.0</version>
</dependency>
```

```java
import  org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class aaa {
    public static void main(String[] args) {
        //配置连接
        final Properties properties = new Properties();
        final Object put = properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        // 指定k v 序列化类型
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        //创建连接
        final KafkaProducer<String, String> kafkaProducer = new KafkaProducer<String, String>(properties);

        //发送
        kafkaProducer.send(new ProducerRecord<>("first", "aaa"));
		//带返回发送
        kafkaProducer.send(new ProducerRecord<>("first", "aaa"), new Callback() {
            @Override
            public void onCompletion(RecordMetadata recordMetadata, Exception e) {

                if (e != null) {
                    System.out.println("主题" +recordMetadata.topic()+
                                       "分区"+ recordMetadata.partition());
                }
            }
        });
        kafkaProducer.close(); 
    }
}
```

#### 同步发送

发送到broker 再接收下一批数据

```java
//同步发送
kafkaProducer.send(new ProducerRecord<>("first", "aaa")).get();
```

#### 分区

1. ==便于合理使用存储资源==，每个Partition在一个Broker上存储，可以把海量的数据按照分区切割成一块块数据存储在多台Broker.上。合理控制分区的任务，可以实现==负载均衡==的效果。
2. ==提高并行度==，生产者可以以分区为单位发送数据;消费者可以以分区为单位进行消费数据。

![image-20220317171716058](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220317171716058.png)

##### 分区策略

```java
DefaultPartitioner

kafkaProducer.send(new ProducerRecord<>("topic", 1,"k","v")).get();
```

1. 指明partition的情况下，直接将指明的值作为partition值.
2. 没有指明partition值但有key的情况下，将key的hash值与topic的partition数进行取余得到partition值;
3. 既没有partition值又没有key值的情况下，Kafka采用Sticky Partition (黏性分区器)，会随机选择一个分区， 并尽可能一直使用该分区，待该分区的batch已满或者已完成，Kafka再随机一个分区进行使用(和上一次的分区不同)。

##### 自定义分区

```java
public class config implements Partitioner {
    @Override
    public int partition(String topic, Object k, byte[] bytes, Object v, byte[] bytes1, Cluster cluster) {
        if (v.toString().contains("aaa")) {
            return 0;
        }
        //返回partition
        return 1;
    }
    @Override
    public void close() {}
    @Override
    public void configure(Map<String, ?> map) {}
}
//使用自定义分区
properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, "com.itl.iap.auth.myClass.juc.config");
```

#### 提高吞吐量

```
batch.size:批次大小,默认16k
linger.ms:等待时间，修改为5- 100ms
compression.type:压缩snappy
RecordAccumulator:缓冲区大小，修改为64m
```

```java
properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);//缓冲区
properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);//批次
properties.put(ProducerConfig.LINGER_MS_CONFIG, 5);//延迟
properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");//压缩 gzip/snappy/lz4/ztsd
```

#### 数据可靠

ack应答级别

- 0  不回答       可能丢失数据
- 1   leader收到回答    可能丢失数据
- -1/all  leader和isr队列所有节点收到回答  可能重复数据

Leader维护了一个动态的in-sync replica set (ISR) ，意为和Leader保持同步的Follower+Leader集合(leader: 0，isr:0,1,2)。

如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值由replica.lag.time.max.ms参数设定，默认30s。

例如2超时，(leader:0, isr:0,1)。这样就不用等长期联系不上或者已经故障的节点。

如果分区副本设置为1个，或者ISR里应答的最小副本数量( min.insync.replicas 默认为1)设置为1，和ack=1的效 果是一样的，仍然有丢数的风险(leader: 0， isr:0)


==数据完全可靠条件=== ACK级别设置为-1 +分区副本大于等于2 + ISR里应答的最小副本数量大于等于2



在生产环境中，acks=0很少使用; 

acks=1， 一般用于传输普通日志，允许丢个别数据;

acks=-1, 一般用于传输和钱相关的数据，对可靠性要求比较高的场景。

```java
properties.put(ProducerConfig.ACKS_CONFIG, 1);
properties.put(ProducerConfig.RETRIES_CONFIG, 1);//重试次数
```

至少一次(At Least Once) = ACK级别设置为-1 +分区副本大于等于2 + ISR里应答的最小副本数量大于等于2
最多一次(AtMostOnce)=ACK级别设置为0

##### 数据幂等性

精确一次(Exactly Once) = 幂等性+至少一次( ack=-1+分区副本数>=2 + ISR最小副本数量>=2)。
重复数据的判断标准:具有<PID, Partition, SeqNumber> >相同主键的消息提交时，Broker只 会持久化一条。其中PID是Kafka每次重启都会分配一一个新的; Partition 表示分区号; Sequence Number是单调自增的

所以幂等性只能保证的是在==单分区单会话内不重复。==

开启幂等性

开启参数enable.idempotence默认为true，false 关闭。

##### 生产者事务

![image-20220317201356824](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220317201356824.png)

```java
//指定事务id
properties.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transaction-id-01");

kafkaProducer.initTransactions();
kafkaProducer.beginTransaction();
kafkaProducer.commitTransaction();
kafkaProducer.abortTransaction();//终止事务
//在事务内提交已经消费的偏移量(主要用于消费者)
kafkaProducer.sendOffsetsToTransaction();
```

##### 数据有序

生产消费有序

单分区有序

多分区要排序

1. kafka在1.x版本之前保证数据单分区有序，条件如下:
   max.in.f1ight.requests.per.connection=1 (不需要考虑是否开幂等性)。
2. kafka在1.x及 以后版本保证数据单分区有序，条件如下:
   1. 未开启幂等性
      max.in.flight.requests.per.connection需要设置为1。.
   2. 开启幂等性
      max.in.flight.requests.per.connection需要设置小于等于5。

原因说明:因为在kafka1.x以后，启用幂等后，kafka服 务端会缓存producer发来的最近5个request的元数据,故无论如何，都可以保证最近5个request的数据都是有序的。

![image-20220317202355958](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220317202355958.png)



### Broker

#### ZK

![image-20220317202743567](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220317202743567.png)

#### 原理

![image-20220317203015903](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220317203015903.png)

#### 新增删除节点

1. 新增

   ```shell
   #创建一个要均衡的主题
   vim topics-to-move.json
   
   {
   	"topics": [
   		{"topic": "first"}	
   	],
   	"version": 1
   }
   
   #生成计划
   bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --topics-to-move-json-file topics-to-move.json --broker-list "0, 1,2,3" --generatee
   
   vim increase-replication-factor.json
   #将上一步返回的计划复制粘贴进json文件
   
   #执行计划
   bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --reassignment-json-file increase-replication-factor.json --execute
   
   #验证
   bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --reassignment-json-file increase-replication-factor.json --verify
   
   ```

2. 删除   减少计划数即可

   ```shell
   #创建一个要均衡的主题
   vim topics-to-move.json
   
   {
   	"topics": [
   		{"topic": "first"}	
   	],
   	"version": 1
   }
   
   #生成计划
   bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --topics-to-move-json-file topics-to-move.json --broker-list "0,1,2" --generatee
   
   vim increase-replication-factor.json
   #将上一步返回的计划复制粘贴进json文件
   
   #执行计划
   bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --reassignment-json-file increase-replication-factor.json --execute
   
   #验证
   bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --reassignment-json-file increase-replication-factor.json --verify
   
   ```


#### 副本

提高可靠性

Kafka默认副本1个，生产环境一般配置为 2个，保证数据可靠性;太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率。

Kafka 中副本分为:
Leader和Follower。 Kafka 生产者只会把数据发往Leader,然后Follower找Leader进行同步数据。

Kafka 分区中的所有副本统称为AR ( Assigned Repllicas)。
AR=ISR+OSR

**ISR**，表示和Leader保持同步的Follower集合。如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。 该时间阈值由replica.lag.time.max.ms参数设定，默认30s。Leader 发生故障之后，就会从ISR中选举新的Leader
**OSR**,表示Follower与Leader副本同步时，延迟过多的副本

##### leader选举流程

![image-20220719142740422](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719142740422.png)



##### Follower故障处理细节

![image-20220719144110443](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719144110443.png)

##### Leader故障处理细节

![image-20220719144247672](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719144247672.png)

##### 分区副本分配

尽量均衡

###### 手动调整副本分配

![image-20220719145036459](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719145036459.png)

![image-20220719145203676](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719145203676.png)

##### Leader Partition自动平衡

![image-20220719145721458](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719145721458.png)



生产可设为false 或比例调大  减少再平衡次数 减少消耗

##### 增加副本

![image-20220719145939809](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719145939809.png)



#### 文件存储

![image-20220719150230490](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719150230490.png)



![image-20220719150533644](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719150533644.png)



![image-20220719150746135](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719150746135.png)

##### 文件清除

Kafka中默认的日志保存时间为7天，可以通过调整如下参数修改保存时间。
log,retention.hours, 最低优先级小时，默认7天。
log.retention.minutes， 分钟。
log.retention.nts,最高优先级毫秒。
log.retention. check.interval.ms，负责设置检查周期，默认5分钟。。

那么日志一旦超过了设置的时间，怎么处理呢?
Kafka中提供的日志清理策略有delete和compact两种



###### delete

日志删除:将过期数据删除
log.cleanup.policy = delete所有数据启用删除策略
(1)基于时间:默认打开。以segment中所有记录中的最大时间戳作为该文件时间戳。
(2)基于大小:默认关闭。超过设置的所有日志总大小，删除最早的segment。

log.retention.bytes,默认等于-1，表示无穷大。

###### compact日志压缩

![image-20220719151503588](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719151503588.png)



##### ==高效读写==

1. ==分布式集群，加分区技术，并行度高==

2. 读数据采用==稀疏索引==，可以快速定位要消费的数据

3. ==顺序写磁盘==
   Kafka的producer生产数据，要写入到log文件中，写的过程是==一直追加到文件末端==，为顺序写。官网有数据表明，同样的磁盘，顺序写能到600M/s，而随机写只有100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。

4. ==页缓存+零拷贝技术==

   ![image-20220719152248453](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719152248453.png)

   

### 消费者

#### 消费者参数

```
fetch.min.bytes：
 该属性指定了消费者从服务器获取记录的最小字节数。broker 在收到消费者的数据请求时，如果可用的数据量小于 fetch.min.bytes 指定的大小，那么它会等到有足够的可用数据时才把它返回给消费者。这样可以降低消费者和 broker 的工作负载，因为它们在主题不是很活跃的时候（或者一天里的低谷时段）就不需要来来回回地处理消息。如果没有很多可用数据，但消费者的 CPU 使用率却很高，那么就需要把该属性的值设得比默认值大。如果消费者的数量比较多，把该属性的值设置得大一点可以降低 broker 的工作负载。 默认值为1 byte

fetch.max.wait.ms
 我们通过 fetch.min.bytes 告诉 Kafka，等到有足够的数据时才把它返回给消费者。而 feth.max.wait.ms 则用于指定 broker 的等待时间，默认是如果没有足够的数据流入Kafka，消费者获取最小数据量的要求就得不到满足，最终导致 500ms 的延迟。如果 fetch.max.wait.ms 被设为 100ms，并且 fetch.min.bytes 被设为 1MB，那么 Kafka 在收到消费者的请求后，要么返回 1MB 数据，要么在 100ms 后返回所有可用的数据，就看哪个条件先得到满足。 默认值为500ms

max.partition.fetch.bytes
 该属性指定了服务器从每个分区里返回给消费者的最大字节数。默认值是 1MB

session.timeout.ms 和heartbeat.interval.ms
session.timeout.ms : 
 消费者多久没有发送心跳给服务器服务器则认为消费者死亡/退出消费者组 默认值:10000ms
heartbeat.interval.ms :
 消费者往kafka服务器发送心跳的间隔 一般设置为session.timeout.ms的三分之一 默认值:3000ms

auto.offset.reset:
 当消费者本地没有对应分区的offset时 会根据此参数做不同的处理 默认值为:latest

earliest 
 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费 

latest 
 当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据 

none 
 topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常

enable.auto.commit
 该属性指定了消费者是否自动提交偏移量，默认值是 true。为了尽量避免出现重复数据和数据丢失，可以把它设为 false，由自己控制何时提交偏移量。如果把它设为 true，还可以通过配置 auto.commit.interval.ms 属性来控制提交的频率。

partition.assignment.strategy
 PartitionAssignor 根据给定的消费者和主题，决定哪些分区应该被分配给哪个消费者。Kafka 有两个默认的分配策略。 
•	 Range：该策略会把主题的若干个连续的分区分配给消费者。假设消费者 C1 和消费者 C2 同时订阅了主题 T1 和主题 T2，并且每个主题有 3 个分区。那么消费者 C1 有可能分配到这两个主题的分区 0 和分区 1，而消费者 C2 分配到这两个主题的分区2。因为每个主题拥有奇数个分区，而分配是在主题内独立完成的，第一个消费者最后分配到比第二个消费者更多的分区。只要使用了 Range 策略，而且分区数量无法被消费者数量整除，就会出现这种情况。
•	 RoundRobin：该策略把主题的所有分区逐个分配给消费者。如果使用 RoundRobin 策略来给消费者 C1 和消费者 C2 分配分区，那么消费者 C1 将分到主题 T1 的分区 0 和分区 2 以及主题 T2 的分区 1，消费者 C2 将分配到主题 T1 的分区 1 以及主题 T2 的分区 0 和分区 2。一般来说，如果所有消费者都订阅相同的主题（这种情况很常见），RoundRobin 策略会给所有消费者分配相同数量的分区（或最多就差一个分区）。

max.poll.records
  单次调用 poll() 方法最多能够返回的记录条数 ,默认值 500

receive.buffer.bytes 和 send.buffer.bytes
 receive.buffer.bytes 默认值 64k 单位 bytes
 send.buffer.bytes 默认值 128k 单位 bytes
 这两个参数分别指定了 TCP socket 接收和发送数据包的缓冲区大小。如果它们被设为 -1
  

```

![image-20220719162408397](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719162408397.png)

![image-20220719162727389](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719162727389.png)



#### 消费者组

Consumer Group (CG) :消费者组，由多个consumer组成。形成- - 个消费者组的条件，是所有消费者的groupid相同。
●消费者组内每个消费者负责消费不同分区的数据，一个分区只能由-一个组内消费者消费。
●消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑.上的一个订阅者。

●消费者超过主题分区数量，一部分消费者就会闲置，不会接收任何消息。

![image-20220719163639304](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719163639304.png)

![image-20220719163843688](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719163843688.png)



#### 消费

注意:在消费者API代码中必须配置消费者组id。命令行启动消费者不填写消费者组id会被自动填写随机的消费者组id

```java
//配置连接
final Properties properties = new Properties();
final Object put = properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
//反序列化
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
//配置消费者组id
properties. put (ConsumerConfig.GROUP_ID_CONFIG, "test");

//创建消费者
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);

//订阅主题first
ArrayList<String> topics = new ArrayList<>();
topics. add("first");
kafkaConsumer . subscribe (topics);

// 2订阅主题对应的分区
ArrayList<TopicPartition> topicPartitions = new ArrayList<>();
topicPartitions . add(new TopicPartition(  "first", 0));
kafkaConsumer . assign(topicPartitions);

//3消费数据
while (true){
    ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration. ofSeconds(1));
    for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
        System. out.println(consumerRecord);
    }
}
```

#### 分区的分配以及再平衡

![image-20220719165946084](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719165946084.png)



##### Range

![image-20220719170156857](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719170156857.png)

##### RoundRobin

![image-20220719170503779](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719170503779.png)

```java
//设置分区 分配策略
properties.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG, "org.apache.kafka.clients.consumer.RoundRobinAssignor");
```

##### Sticky以及再平衡

粘性分区定义:可以理解为分配的结果带有“粘性的”。即在执行一次新的分配之前，考虑上一次分配的结果，尽量少的调整分配的变动，可以节省大量的开销

粘性分区是Kafka 从0.11.x 版本开始引入这种分配策略，首先会尽量均衡的放置分区到消费者.上面，在出现同一消费者组内消费者出现问题的时候，会尽量保持原有分配的分区不变化。。

#### offset位移

![image-20220719171407648](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719171407648.png)

_consumer_ offsets 主题里面采用key和value 的方式存储数据。key是group.id+topic+分区号，value 就是当前offset 的值。每隔一段时间，kafka 内部会对这个topic 进行compact，也就是每个group.id+topic+分区号就保留最新数据



(1)在配置文件config/consumer.properties 中添加配置exclude.internal.topics =false默认是true,表示不能消费系统主题。为了查看该系统主题数据，所以该参数修改为false

![image-20220719171737260](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719171737260.png)

##### 自动提交

![image-20220719171953500](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719171953500.png)

```java
//自动提交
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
//提交时间间隔
properties.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 1000);
```

##### 手动提交

![image-20220719172432218](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719172432218.png)

```java
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
kafkaConsumer.commitAsync();
kafkaConsumer.commitSync();
```

##### 指定offset

![image-20220719172856143](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719172856143.png)



```java
//指定位置进行消费
Set<TopicPartition> assignment = kafkaConsumer.assignment();
//指定消费的offset
for (TopicPartition topicPartition : assignment) {
    kafkaConsumer.seek(topicPartition, 100);
}

//希望把时间转换为对应的offset
HashMap<TopicPartition, Long> topicPartitionLongHashMap = new HashMap<>();
//封装对应集合
for (TopicPartition topicPartition : assignment) {
    topicPartitionLongHashMap.put(topicPartition, System.currentTimeMillis() - 1 * 24 * 3600 * 1000);
}
Map<TopicPartition, OffsetAndTimestamp> topicPartitionOffsetAndTimestampMap = kafkaConsumer.offsetsForTimes(topicPartitionLongHashMap);
//指定消费的offset
for (TopicPartition topicPartition : assignment) {
    OffsetAndTimestamp offsetAndTimestamp = topicPartitionOffsetAndTimestampMap.get(topicPartition);
    kafkaConsumer.seek(topicPartition, offsetAndTimestamp.offset());
}
```

###### 漏消费和重复消费

![image-20220719174114822](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719174114822.png)



![image-20220719174158928](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719174158928.png)

###### 数据积压

![image-20220719174402355](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719174402355.png)

### Kafka-Eagle监控

Kafka-Eagle https://www.kafka-eagle.org/

Kafka-Eagle的安装依赖于MySQL，MySQL主要用来存储可视化展示的数据。

### Kafka-Kraft模式

![image-20220719175243802](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719175243802.png)





### 整合

#### flume

![image-20220719175850854](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719175850854.png)

![image-20220719180026678](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719180026678.png)

![image-20220719180205159](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719180205159.png)

![image-20220719180217490](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719180217490.png)

![image-20220719180406987](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719180406987.png)

![image-20220719180530535](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719180530535.png)

![image-20220719180607805](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719180607805.png)

#### flink

![image-20220719182421112](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719182421112.png)

![image-20220719182320873](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719182320873.png)

![image-20220719182453744](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719182453744.png)

![image-20220719182546953](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719182546953.png)







#### spring boot

<img src="https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181112348.png" alt="image-20220719181112348" />

![image-20220719181234318](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181234318.png)



![image-20220719181306153](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181306153.png)

![image-20220719181357039](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181357039.png)

#### spak

![image-20220719181536315](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181536315.png)

![image-20220719181612556](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181612556.png)



![image-20220719181757583](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181757583.png)

![image-20220719181830017](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719181830017.png)

![image-20220719182201819](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220719182201819.png)



### 调优

#### 硬件选择

1亿条/天 日志

1k条/s

高峰 = 平时 * 20或40  ->20m/s-40m/s

##### 服务器

服务器台数 = 2*( 生产峰值 *副本/100)

##### 硬盘

kafka顺序读写    机械和固态顺序读写速度差不多

##### 内存

kafka 堆内存 (内部配置) 10-15g + 页缓存 (分区数/4/服务器数)

##### CPU

num.io.threads=8负责写磁盘的线程数，整个参数值要占总核数的50%
num.replica.fetchers= 1副本拉取线程数，这个参数占总核数的50%的1/3。
num.network.threads=3数 据传输线程数，这个参数占总核数的50%的2/3。建议32个cpu core

##### 网络

大于峰值*8

#### 生产者

read-only: 重新启动更新
per broker :动态更新
each broker群集范围：可以动态更新为群集范围内的默认值



```shell
bootstrap.servers  服务器地址
key.serializer和value .serializer   kv序列化
buffer. memory     RecordAccumulator缓冲区总大小，默认32m。
batch.size
缓冲区一批数据最大值，默认16k。 适当增加该值，可以提高吞吐量，但是如果该值设置太大，会导致数据传输延迟增加。

linger.ms
如果数据迟迟未达到batch.size, sender 等待linge.time 之后就会发送数据。单位ms，默认值是0ms，表示没有延迟。生产环境建议该值大小为5-100ms之间。

acks
0:生产者发送过来的数据，不需要等数据落盘应答。
1:生产者发送过来的数据，Leader收到数据后应答。
-1 (all) :生产者发送过来的数据，Leader+和isr队列里面的所有节点收齐数据后应答。默认值是-1, -1和all是等价的。

maxin.flight.requests.per.connectione|允许最多没有返回ack的次数，默认为5，开启基等性要保证该值是1-5

retries-
当消息发送出现错误的时候,系统会重发消息。retries表示重试次数。默认是int最大值，2147483647。 
如果设置了重试，还想保证消息的有序性，需要设置_MAX_ IN FLIGHT_ REQUESTS PER CONNECTION=1
否则在重试此失败消息的时候，其他的消息可能发送成功了。

retry.backoff.ms-两次重试之间的时间间隔，默认是100ms
enable. idempotence是否开启暴等性，默认true,开启基等性
compression.type生产者发送的所有数据的压缩方式。默认是none ,，也就是不压缩。
支持压缩类型:none、gzip、 snappy、 lz4 和zstd。
```

##### 提高吞吐量

buffer.memory
RecordAccumulator缓冲区总大小，默认32m
batch.size
缓冲区一批数据最大值，默认16k。 适当增加该值，可以提高吞吐量，但是如果该值设置太大，会导致数据传输延迟增加
linger.ms
如果数据迟迟未达到batch.size， sender 等待linge.time之后就会发送数据。单位ms,默认值是0ms,表示没有延迟。生产环境建议该值大小为5-100ms之间。
compression.type-
生产者发送的所有数据的压缩方式。默认是none,，也就是不压缩。.支持压缩类型: none、 gzip、 snappy、 lz4 和zstd。

##### 数据可靠

至少一次(At Least Once) = ACK级别设置为-1 +分区副本大于等于2 +ISR里应答的最小副本数量大于等于2

##### 幂等性

enable.idempotence是否开启暴等性，默认true，表示开启幂等性。

###### 事务

![image-20220721214451339](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220721214451339.png)

##### 数据有序

单分区内，有序(有条件的，不能乱序)
多分区，分区与分区间无序;

##### 数据乱序

enable.idempotencee
是否开启暴等性，默认true,表示开启暴等性。。
max.in.flight.requests.per.connection-
允许最多没有返回ack的次数，默认为5，开启暴等性要保证该值是1-5 的数字。

#### broker

```
replica.lag.time.max.ms
ISR中,如果Follower长时间未向Leader发送通信请求或同步数据,则该Follower将被踢出ISR。该时间阈值，默认30s

auto.leader.rebalance.enable
默认是true。自动 Leader Partition平衡。建议关闭。

leader.imbalance.per. broker.percentage
默认是10%。每个broker允许的不平衡的leader的比率。如果每个broker 超过了这个值，控制器会触发leader的平衡

leader. imbalance.check.interval.seconds-
默认值300秒。检查leader负载是否平衡的间隔时间

log.segment.bytese-
Kafka中log 日志是分成-块块存储的，此配置是指log日志划分成块的大小，默认值1G

log.index. intrval.bytes-
默认4kb, kafka里面每当写入了4kb大小的日志(.log) ,然后就往index文件里面记录一一个索引

logretention.hourse
Kafka中数据保存的时间，默认7天。
log.retention.minutes-
Kafka中数据保存的时间，分钟级别，默认关闭。
log.retention.ms-
Kafka中数据保存的时间，毫秒级别，默认关闭。
log.retention.check.interval.ms
检查数据是否保存超时的间隔，默认是5分钟。

logretention.bytes-
默认等于-1，表示无穷大。超过设置的所有日志总大小，删除最早的segment。

log.cleanup.policy-
默认是delete,表示所有数据启用删除策略;如果设置值为compact， 表示所有数据启用压缩策略。

num. io.threads
默认是8。负责写磁盘的线程数。整个参数值要占总核数的50%。
num.replica.fetchers
默认是1。副本拉取线程数，这个参数占总核数的50%的1/32
num.network. threads←
默认是3。数据传输线程数，这个参数占总核数的50%的2/3。

log.fush.interval.messages-
强制页缓存刷写到磁盘的条数,默认是long的最大值，9223372036854775807。 一般不建议修改，交给系统自己管理。
log.flush.interval.mse-
每隔多久，刷数据到磁盘，默认是null。一般不建议修改，交给系统自己管理


```

加减节点

![image-20220721215936109](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220721215936109.png)

![image-20220721220012657](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220721220012657.png)



![image-20220721220025429](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220721220025429.png)

![image-20220721220048819](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220721220048819.png)

##### 负载均衡

![image-20220721220144547](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220721220144547.png)

##### 自动创建主题

![image-20220721220239816](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220721220239816.png)









#### 消费者

```
bootstrap.servers-向Kafka集群建立初始连接用到的host/port 列表。
key.deserializer和value.deserializer指定接收消息的key和value的反序列化类型。要写全类名。

group.id
标记消费者所属的消费者组

enable.auto.commit
默认值为true,消费者会自动周期性地向服务器提交偏移量

auto.commit.interval.ms
如果设置了enable.auto.commit 的值为true，则该值定 义了消费者偏移量向Kafka提交的频率，默认5s

autoffset.reset
当Kafka中没有初始偏移量或当前偏移量在服务器中不存在(如，数据被删除了)，该如何处理? 
earliest:自动重置偏移量到最早的偏移量。
latest: 默认，自动重置偏移量为最新的偏移量。
none:如果逍费组原来的(previous) 偏移量不存在，则向消费者抛异常。
anything:向消费者抛异常。

offsets.topic.num,partitions_
consumer_ offsets 的分区数，默认是50个分区。不建议修改

heartbeat. interval.ms
Kafka消费者和coordinator之间的心跳时间，默认3s。。该条目的值必须小于session.timeout.ms ，也不应该高于
session.timeout.ms的1/3。不建议修改。

session.timeout.ms
Kafka消费者和coordinator之间连接超时时间，默认45s.超过该值，该消费者被移除，消费者组执行再平衡。

max.poll.interval.ms
消费者处理消息的最大时长，默认是5分钟。超过该值，该消费者被移除，消费者组执行再平衡。

fetch.max.bytes-
默认Default: 52428800 (50m)。消费者获取服务器端一~批消息最大的字节数。如果服务器端批次的数据大于该值
(50m)仍然可以拉取回来这批数据，因此，这不是一一个绝对最大值。-批次的大小受message.max.bytes ( broker
config) or max.message.bytes (topic config)影响。<

max.poll.records -次poll拉取数据返回消息的最大条数，默认是500条。

```

##### 消费者再平衡

```
heartbeat.interval.ms-
Kafka消费者和coordinator之间的心跳时间，默认3s。
该条目的值必须小于sesson.timeout.ms，也不应该高于”session.timeout.ms的1/3。

session.timeout.ms-
Kafka消费者和coordinator之间连接超时时间，默认45s。超过该值，该消费者被移除，消费者组执行再平衡。

max. poll.interval.ms<
消费者处理消息的最大时长，默认是5分钟。超过该值，该消费者被移除，消费者组执行再平衡。(

partition. assignment. strategy-
消费者分区分配策略，默认策略是Range +CooperativeSticky. Kafka 可以同时使用多个分区分配策略。
可以选择的策略包括: Range、 RoundRobin、 Sticky 、CooperativeSticky
```

![image-20220725115218843](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220725115218843.png)

##### 提高吞吐量

![image-20220725115308057](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220725115308057.png)

#### 总体调优

1. 提升生产吞吐量<

   1. buffer.memory: 发送消息的缓冲区大小，默认值是32m,可以增加到64m。

   2. batch.size:默认是16k。如果batch设置太小，会导致频繁网络请求，吞吐量下降; .如果batch太大，会导致一条消息需要等待很久才能被发送出去，增加网络延时

   3.  linger.ms, 这个值默认是0，意思就是消息必须立即被发送。一般设置 一个 5-100毫秒。如果linger.ms设置的太小，会导致频繁网络请求，吞吐量下降;如果linger.ms太长，会导致一条消息需要等待很久才能被发送出去，增加网络延时

   4. compression.type: 默认是none，不压缩，但是也可以使用lz4压缩，效率还是不错的，压缩之后可以减小数据量，提升吞吐量，但是会加大producer端的CPU开销

2. 增加分区
3. 消费者提高吞吐量←
   1. 调整fetch.max.bytes大小，默认是50m。
   2. 调整max.poll.records大小，默认是500条。
4. 增加下游消费者处理能力

##### 数据精准一次

1. 生产者角度。
   acks设置为-1 (acks=-1) 。
   暴等性(enable.idempotence= true) +事务。
2. broker 服务端角度
   分区副本大于等于 2 (--replication-factor2) 。
   ISR里应答的最小副本数量大于等于2 (min.insync.replicas =2)。
3. 消费者
   事务 +手动提交offset (enable.auto.commit= false) 。。
   消费者输出的目的地必须支持事务(MySQL、 Kafka) 。

##### 合理设置分区数

1. 创建-一个只有1个分区的topic
2. 测试这个topic的producer吞吐量和consumer吞吐量。
3. 假设他们的值分别是Tp和Tc，单位可以是MB/s。
4. 然后假设总的目标吞吐量是Tt,那么分区数=Tt/min (Tp，Tc)。
   例如: producer 吞吐量= 20m/s; consumer 吞吐量= 50m/s，期望吞吐量100m/s;
   分区数= 100/20=5分区
   分区数一般设置为: 3-10 个
   分区数不是越多越好，也不是越少越好，需要搭建完集群，进行压测，再灵活调整分区个数。

##### 单条日志大于1m

```
message.max.bytes-默认1m, broker 端接收每个批次消息最大值。

max.request.size
默认1m,生产者发往broker每个请求消息最大值。针对topic级别设置消息体的大小。

replica.fetch.max.bytes-默认1m,副本同步数据，每个批次消息最大值。

fetch.max.bytes-
默认Default: 52428800 (50m)。消费者获取服务器端一-批消息最大的字节数。如果服务器端-批次的数据大于该值
(50m)仍然可以拉取回来这批数据，因此，这不是一一个绝对最大值。一批次的大小受message.max.bytes (broker config)
or max.message.bytes (topic config)影响。


```

##### 服务器挂了

在生产环境中，如果某个Kafka节点挂掉。
正常处理办法:。

1. 先尝试重启
2. 如果重启不行，考虑增加内存、增加CPU、网络带宽。
3. 如果将kafka 整个节点误删除，如果副本数大于等于2,可以按照服役新节点的方式重新服役一个新节点， 并执行负载均衡。

##### 压测

用Kafka官方自带的脚本，对Kafka进行压测。

* 生产者压测: kafka producer-perf test.sh
* 消费者压测: kafka-consumer-perf-test.sh

###### 生产者

![image-20220725192125695](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220725192125695.png)

![image-20220725192543996](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220725192543996.png)

###### 消费者

![image-20220725192633392](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220725192633392.png)

![image-20220725192825038](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220725192825038.png)