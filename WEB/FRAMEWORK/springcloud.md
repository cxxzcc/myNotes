https://www.springcloud.cc/

# spring cloud

https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md中文

https://github.com/spring-projects/spring-boot/releases/

https://github.com/spring-projects/spring-cloud

SpringCloud和SpringBoot之间的依赖关系

https://spring.io/projects/spring-cloud#overview

https://start.spring.io/actuator/info

SpringCloud第二季定稿版(截止2020.2.15)

cloud-Hoxton.SR1  

boot-2.2.2.RELEASE 

cloud alibaba-2.1.0.RELEASE

![image-20210420174359756](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210420174359756.png)



## 1.0 项目构建

### 1.0.1 maven

Maven 使用dependencyManagement 元素来管理依赖版本号

使用pom.xml 中的dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。
Maven 会沿着父子层次向上走，直到找到一个拥有dependencyManagement 元素的项目，然后它就会使用这个

1. **dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。**

2. 如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;

3. 如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。. 

```xml
<build><!-- maven中跳过单元测试 -->
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <skip>true</skip>
            </configuration>
        </plugin>
    </plugins>
</build>
```

或者IDEA工具支持(推荐)单击闪电图标

### 1.0.2 mysql

com.mysql.jdbc.Driver和mysql-connector-java 5一起用。
com.mysql.cj.jdbc.Driver和mysql-connector-java 6 一起用。


com.mysql.cj.jdbc.Driver是mysql-connector-java 6 中的特性，相比mysql-connector-java 5 多了一个时区：serverTimezone，把数据源配置的驱动改一下就好了

```yml
server:
  port: 8001
spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包 com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 123456
mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
```

### 1.0.3 热部署Devtools

1. Adding devtools to your project

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
       <scope>runtime</scope>
       <optional>true</optional>
   </dependency>
   ```

2. Adding plugin to your pom.xml

   ```xml
   下段配置我们粘贴进聚合父类总工程的pom.xml里
   <build>
       <finalName>你自己的工程名字</finalName>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
               <configuration>
                   <fork>true</fork>
                   <addResources>true</addResources>
               </configuration>
           </plugin>
       </plugins>
   </build>
   ```

3. Enabling automatic build

   ![image-20210422155748005](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210422155748005.png)

4. Update the value of

   ctr1 +shift+A1t +/

   ![image-20210422155834942](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210422155834942.png)

5. 重启IDEA

### 1.0.4 RestTemplate

https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

(url, requestMap, ResponseBean.class)这三个参数分别代表 
REST请求地址、请求参数、HTTP响应转换被转换成的对象类型

```java
@Configuration
public class ApplicationContextConfig{
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

## 1.1 Eureka

Eureka包含两个组件：Eureka Server和Eureka Client

Eureka Server提供服务注册服务

EurekaClient通过注册中心进行访问   连接到 Eureka Server并维持心跳连接

### 1.1.1 服务注册中心

```xml
以前的老版本（当前使用2018）
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
现在新版本（当前使用2020.2）
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
    #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

主启动@EnableEurekaServer

### 1.1.2 服务

```xml
现在新版本,当前使用
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 123456

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka


mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
```

主启动@EnableEurekaClient

### 1.1.3 集群

**EurekaServer集群环境构建步骤**

C:\Windows\System32\drivers\etc路径下的hosts文件

修改映射配置添加进hosts文件

```
127.0.0.1  eureka7001.com
127.0.0.1  eureka7002.com
```

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/
```

```yml
server:
  port: 7002
eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

将微服务发布到上面2台Eureka集群配置中

```yml
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
```

**服务提供者集群环境构建**

修改端口

```yml
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
```

负载均衡

```java
@Configuration
public class ApplicationContextBean{
    @Bean
    @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

### 1.1.4 actuator微服务信息完善

主机名称:服务名称修改

访问信息有IP信息提示

```yml
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
      #defaultZone: http://localhost:7001/eureka  # 单机版
  instance:
    instance-id: payment8001    #主机名称:服务名称修改
    prefer-ip-address: true     #访问路径可以显示IP地址
```

### 1.1.5 服务发现Discovery

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息

```java
@Resource
private DiscoveryClient discoveryClient;
@GetMapping(value = "/payment/discovery")
public Object discovery(){
    List<String> services = discoveryClient.getServices();
    for (String element : services) {
        System.out.println(element);
    }

    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    for (ServiceInstance element : instances) {
        System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t" + element.getUri());
    }
    return this.discoveryClient;
}

```

主启动@EnableDiscoveryClient

### 1.1.6 Eureka自我保护

保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，
Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。



自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。



某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存

属于CAP里面的AP分支

**禁止自我保护**

注册中心

```yml
eureka:
  instance:
    hostname: eureka7001.com
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      #defaultZone: http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      defaultZone: http://eureka7001.com:7001/eureka
  server:
    #关闭自我保护机制，保证不可用服务被及时踢除
       enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

服务

```yml
eureka:
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      # cluster version
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      # singleton version
      defaultZone: http://eureka7001.com:7001/eureka
#心跳检测与续约时间
#开发时设置小些，保证服务关闭后注册中心能即使剔除服务
  instance:
  #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
  #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

## 1.2 Zookeeper

zookeeper是一个分布式协调工具，可以实现注册中心功能

关闭Linux服务器防火墙后启动zookeeper服务器

### 1.2.1 demo 服务提供者

```xml
<!-- SpringBoot整合zookeeper客户端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

```yml
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004
#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 192.168.111.144:2181
```

主启动@EnableDiscoveryClient

解决zookeeper版本jar包冲突问题  与服务器版本一致

```xml
<!-- SpringBoot整合zookeeper客户端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
    <!--先排除自带的zookeeper3.5.3-->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--添加zookeeper3.4.9版本-->
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency>
```

### 1.2.2 入门

1. Zookeeper：一个Leader，多个Follower组成的集群
2. 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。适合奇数安装。 
3. 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。 
4. 更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。 
5. 数据更新原子性，一次数据更新要么成功，要么失败。 
6. 实时性，在一定时间范围内，Client能读到最新数据




数据结构 
> ZooKeeper 数据模型的结构与 Unix 文件系统很类似，整体上可以看作是一棵树，每个 节点称做一个 ZNode。每一个 ZNode 默认能够存储 1MB 的数据，每个 ZNode 都可以通过 其路径唯一标识


应用
* 统一命名服务  域名
* 统一配置管理
* 统一集群管理
* 服务器节点动态上下线
* 软负载均衡

### 1.2.3 安装
[zookeeper.apache.org](https://zookeeper.apache.org/)

1. 本地安装
	* jdk
	* 拷贝 apache-zookeeper-3.5.7-bin.tar.gz 安装包到 Linux 系统下
	* tar -zxvf apache-zookeeper-3.5.7- bin.tar.gz -C /opt/module/
	* mv apache-zookeeper-3.5.7 -bin/ zookeeper-3.5.7
	* mv zoo_sample.cfg zoo.cfg
	* vim zoo.cfg
		dataDir=/opt/module/zookeeper-3.5.7/zkData
	* mkdir zkData
	* 启动
		* bin/zkServer.sh start
		* bin/zkServer.sh status
		* bin/zkCli.sh 启动client
		* quit 
		* bin/zkServer.sh stop
2. 配置文件zoo.cfg
	* tickTime = 2000：通信心跳时间 服务器与客户端心跳时间 ms
	* initLimit = 10：Leader-Follower初始通信时限
	* syncLimit = 5：LF同步通信时限
	* dataDir：保存Zookeeper中的数据
	* clientPort = 2181：客户端连接端口
3. 集群安装



















## 1.3 Consul

https://www.consul.io/intro/index.html

https://www.springcloud.cc/spring-cloud-consul.html中文

开源的分布式服务发现和配置管理系统，由 HashiCorp 公司用 Go 语言开发。

提供了微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。

它具有很多优点。包括： 基于 raft 协议，比较简洁； 支持健康检查, 同时支持 HTTP 和 DNS 协议 支持跨数据中心的 WAN 集群 提供图形界面 跨平台，支持 Linux、Mac、Windows

作用

1. 服务发现  提供HTTP和DNS两种发现方式。
2. 健康监测  支持多种方式，HTTP、TCP、Docker、Shell脚本定制化监控
3. KV存储  Key、Value的存储方式
4. 多数据中心   Consul支持多数据中心
5. 可视化Web界面

### 1.3.1 安装

https://www.consul.io/downloads.html

https://learn.hashicorp.com/consul/getting-started/install.html

```shell
consul --version  #查看版本
#使用开发模式启动
consul agent -dev
http://localhost:8500
```

### 1.3.2 demo

```xml
<!--SpringCloud consul-server -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
<!-- SpringBoot整合Web组件 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yml
###consul服务端口号
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
####consul注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname: 127.0.0.1
        service-name: ${spring.application.name}
```

主启动@EnableDiscoveryClient

## 1.4 Ribbon负载均衡服务调用

基于Netflix Ribbon实现的一套客户端负载均衡的工具

https://github.com/Netflix/ribbon/wiki/Getting-Started

Ribbon目前也进入维护模式   未来替换方案LoadBlancer



LB负载均衡(Load Balance)是什么
简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。
常见的负载均衡有软件Nginx，LVS，硬件 F5等。



Ribbon本地负载均衡客户端 VS Nginx服务端负载均衡区别
 Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的。

 Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。



**集中式LB**

即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；



**进程内LB**

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。



负载均衡+RestTemplate调用



Ribbon在工作时分成两步
第一步先选择 EurekaServer ,它优先选择在同一个区域内负载较少的server.
第二步再根据用户指定的策略，在从server取到的服务注册列表中选择一个地址。
其中Ribbon提供了多种策略：比如轮询、随机和根据响应时间加权。



### 1.4.1 RestTemplate

https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

getForObject方法

返回对象为响应体中数据转化成的对象，基本上可以理解为Json

getForEntity方法

返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等

```java
@Resource
private RestTemplate restTemplate;
restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class)
restTemplate.getForEntity(PAYMENT_URL+"/payment/get/"+id,CommonResult.class)
```

postForObject/postForEntity

```java
@Bean
@LoadBalanced
public RestTemplate getRestTemplate(){
	return new RestTemplate();
}
```

### 1.4.2 Ribbon核心组件IRule

IRule：根据特定算法中从服务列表中选取一个要访问的服务

1. com.netflix.loadbalancer.RoundRobinRule 轮询
2. com.netflix.loadbalancer.RandomRule 随机
3. com.netflix.loadbalancer.RetryRule 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
4. WeightedResponseTimeRule 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
5. BestAvailableRule 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
6. AvailabilityFilteringRule 先过滤掉故障实例，再选择并发较小的实例
7. ZoneAvoidanceRule 默认规则,复合判断server所在区域的性能和server的可用性选择服务器

**替换**

官方文档明确给出了警告：
这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，
否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。

```java
package com.atguigu.myrule;//新建包

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MySelfRule
{
    @Bean
    public IRule myRule()
    {
        return new RandomRule();//定义为随机
    }
}
```

主启动添加

```java
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration=MySelfRule.class)
```

Ribbon负载均衡算法

负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标  ，每次服务重启动后rest接口计数从1开始。

**手写**

1. ApplicationContextBean去掉注解@LoadBalanced

2. LoadBalancer接口

   ```java
   public interface LoadBalancer{
       ServiceInstance instances(List<ServiceInstance> serviceInstances);
   }
   ```

3. MyLB

   ```java
   @Component
   public class MyLB implements LoadBalancer{
       private AtomicInteger atomicInteger = new AtomicInteger(0);
   
       public final int getAndIncrement(){
           int current;
           int next;
           do{
               current = this.atomicInteger.get();
               next = current >= 2147483647 ? 0 : current + 1;
           } while(!this.atomicInteger.compareAndSet(current, next));
           System.out.println("*****next: "+next);
           return next;
       }
       @Override
       public ServiceInstance instances(List<ServiceInstance> serviceInstances){
           int index = getAndIncrement() % serviceInstances.size();
           return serviceInstances.get(index);
       }
   }
   ```

4. OrderController

   ```java
   @Resource
   private DiscoveryClient discoveryClient;
   @Resource
   private LoadBalancer loadBalancer;
   @GetMapping("/consumer/payment/lb")
   public String getPaymentLB(){
           List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
   
           if(instances == null || instances.size()<=0) {
               return null;
           }
           ServiceInstance serviceInstance = loadBalancer.instances(instances);
           URI uri = serviceInstance.getUri();
   
           return restTemplate.getForObject(uri+"/payment/lb",String.class);
       }
   
   ```

## 1.5 OpenFeign服务接口调用

https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/#spring-cloud-openfeign

https://github.com/spring-cloud/spring-cloud-openfeign

声明式WebService客户端

使用方法是定义一个服务接口然后在上面添加注解。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡



Feign集成了Ribbon

**Feign和OpenFeign两者区别**

Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端
Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务

OpenFeign是Spring Cloud 在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。

### 1.5.1 demo

```xml
<!--openfeign-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

主启动@EnableFeignClients

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService{
    @GetMapping(value = "/payment/get/{id}")
    CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}
```

### 1.5.2 OpenFeign超时控制

```yml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

### 1.5.3 OpenFeign日志打印功能

日志级别

1. NONE：默认的，不显示任何日志；

2. BASIC：仅记录请求方法、URL、响应状态码及执行时间；

3. HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息；

4. FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据。

配置日志bean

```java
@Configuration
public class FeignConfig{
    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

```yml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.atguigu.springcloud.service.PaymentFeignService: debug
```

### 远程请求头丢失问题

```java
/**
 * feign拦截器功能
 * 解决feign 远程请求头丢失问题
 **/
@Configuration
public class GuliFeignConfig {
    @Bean("requestInterceptor")
    public RequestInterceptor requestInterceptor() {
        RequestInterceptor requestInterceptor = new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                //1、使用RequestContextHolder拿到 /toTrade请求的request 请求数据
                // spring提供的
                ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                if (requestAttributes != null) {
                    //老请求
                    HttpServletRequest request = requestAttributes.getRequest();
                    if (request != null) {
                        //2、同步请求头的数据（主要是cookie）
                        //把老请求的cookie值放到新请求上来，进行一个同步
                        String cookie = request.getHeader("Cookie");
                        template.header("Cookie", cookie);
                    }
                }
            }
        };
        return requestInterceptor;
    }

}

```

### 异步请求头丢失

因为使用ThreadLocal共享数据

```java
//主线程   获取当前线程请求头信息(解决Feign异步调用丢失请求头问题)
RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
//其他线程   再设置：就可以在拦截器内获取了
RequestContextHolder.setRequestAttributes(requestAttributes);
```





## 1.6 Hystrix断路器

处理分布式系统的延迟和容错的开源库

https://github.com/Netflix/Hystrix/wiki/How-To-Use

Hystrix官宣，停更进维

### 1.6.1 Hystrix重要概念

服务降级   服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback

​	哪些情况会出发降级

1. 程序运行异常
2. 超时
3. 服务熔断触发服务降级
4. 线程池/信号量打满也会导致服务降级

服务熔断   达到最大服务访问后，直接拒绝访问，调用服务降级的方法并返回友好提示

服务限流  秒杀高并发等操作

### 1.6.2 服务降级

**生产端**

当前服务不可用了，做服务降级，兜底的方案都是paymentInfo_TimeOutHandler

```java
@HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")
    })
    public String paymentInfo_TimeOut(Integer id)
    {
        int second = 5;
        try { TimeUnit.SECONDS.sleep(second); } catch (InterruptedException e) { e.printStackTrace(); }
        return "线程池:"+Thread.currentThread().getName()+"paymentInfo_TimeOut,id: "+id+"\t"+"O(∩_∩)O，耗费秒: "+second;
    }
    public String paymentInfo_TimeOutHandler(Integer id){
        return "/(ㄒoㄒ)/调用支付接口超时或异常：\t"+ "\t当前线程池名字" + Thread.currentThread().getName();
    }

```

主启动+@EnableCircuitBreaker

**消费端**

```yml
feign:
  hystrix:
    enabled: true
```

主启动@EnableHystrix

```java
@GetMapping("/consumer/payment/hystrix/timeout/{id}")
@HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
        @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
})
public String paymentInfo_TimeOut(@PathVariable("id") Integer id)
{
    String result = paymentHystrixService.paymentInfo_TimeOut(id);
    return result;
}
public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id)
{
    return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
}
```

**统一配置**

```java
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
public class PaymentHystirxController{
	@GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand //加了@DefaultProperties属性注解，并且没有写具体方法名字，就用统一全局的
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }
    public String payment_Global_FallbackMethod(){
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }
}
```

Feign+fallback

```java
@Component //必须加 //必须加 //必须加
public class PaymentFallbackService implements PaymentFeignClientService{
    @Override
    public String getPaymentInfo(Integer id){
        return "服务调用失败，提示来自：cloud-consumer-feign-order80";
    }
}
```

```yml
feign:
  hystrix:
   enabled: true #在Feign中开启Hystrix
```

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)
public interface PaymentFeignClientService{
    @GetMapping("/payment/hystrix/{id}")
    public String getPaymentInfo(@PathVariable("id") Integer id);
}
```

### 1.6.3 服务熔断

https://martinfowler.com/bliki/CircuitBreaker.html

熔断机制概述
熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，
会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。
<font color="red">当检测到该节点微服务调用响应正常后，恢复调用链路</font>

当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是@HystrixCommand。

```java
 //=====服务熔断
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),//是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),//请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), //时间窗口期
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"),//失败率达到多少后跳闸
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if (id < 0) {
            throw new RuntimeException("******id 不能负数");
        }
        String serialNumber = IdUtil.simpleUUID();
        return Thread.currentThread().getName() + "\t" + "调用成功，流水号: " + serialNumber;
    }

    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " + id;
    }
```

**熔断类型**

1. 熔断打开  请求不再进行调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间)，当打开时长达到所设时钟则进入半熔断状态
2. 熔断关闭  熔断关闭不会对服务进行熔断
3. 熔断半开  部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

**涉及到断路器的三个重要参数：快照时间窗、请求总数阀值、错误百分比阀值。**

1. 快照时间窗：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。

2. 请求总数阀值：在快照时间窗内，必须满足请求总数阀值才有资格熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用次数不足20次，即使所有的请求都超时或其他原因失败，断路器都不会打开。

3. 错误百分比阀值：当请求总数在快照时间窗内超过了阀值，比如发生了30次调用，如果在这30次调用中，有15次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%阀值情况下，这时候就会将断路器打开。

**断路器开启或者关闭的条件**

1. 当满足一定的阀值的时候（默认10秒内超过20个请求次数）

2. 当失败率达到一定的时候（默认10秒内超过50%的请求失败）

3. 到达以上阀值，断路器将会开启

4. 当开启的时候，所有请求都不会进行转发

5. 一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。重复4和5



当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑，当休眠时间窗到期，断路器将进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将继续闭合，主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时。

```java
//========================All配置
@HystrixCommand(fallbackMethod = "str_fallbackMethod",
        groupKey = "strGroupCommand",
        commandKey = "strCommand",
        threadPoolKey = "strThreadPool",

        commandProperties = {
                // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
                @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
                // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
                @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
                // 配置命令执行的超时时间
                @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
                // 是否启用超时时间
                @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
                // 执行超时的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
                // 执行被取消的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
                // 允许回调方法执行的最大并发数
                @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
                // 服务降级是否启用，是否执行回调函数
                @HystrixProperty(name = "fallback.enabled", value = "true"),
                // 是否启用断路器
                @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
                // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，
                // 如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
                @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
                // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过
                // circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50,
                // 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
                @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
                // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，
                // 会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，
                // 如果成功就设置为 "关闭" 状态。
                @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
                // 断路器强制打开
                @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
                // 断路器强制关闭
                @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
                // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
                @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
                // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据
                // 设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
                // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
                @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
                // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。
                @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
                // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
                @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
                // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
                @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
                // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
                // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
                // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
                @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
                // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。
                @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
                // 是否开启请求缓存
                @HystrixProperty(name = "requestCache.enabled", value = "true"),
                // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
                @HystrixProperty(name = "requestLog.enabled", value = "true"),
        },
        threadPoolProperties = {
                // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
                @HystrixProperty(name = "coreSize", value = "10"),
                // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，
                // 否则将使用 LinkedBlockingQueue 实现的队列。
                @HystrixProperty(name = "maxQueueSize", value = "-1"),
                // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
                // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue
                // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
                @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
        }
)
public String strConsumer() {
    return "hello 2020";
}
public String str_fallbackMethod(){
    return "*****fall back str_fallbackMethod";
}
```

### 1.6.4 hystrix工作流程

https://github.com/Netflix/Hystrix/wiki/How-it-Works

![image-20210421105711927](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210421105711927.png)

1. 创建 HystrixCommand（用在依赖的服务返回单个操作结果的时候） 或 HystrixObserableCommand（用在依赖的服务返回多个操作结果的时候） 对象。

2. 命令执行。其中 HystrixComand 实现了下面前两种执行方式；而 HystrixObservableCommand 实现了后两种执行方式：execute()：同步执行，从依赖的服务返回一个单一的结果对象， 或是在发生错误的时候抛出异常。queue()：异步执行， 直接返回 一个Future对象， 其中包含了服务执行结束时要返回的单一结果对象。observe()：返回 Observable 对象，它代表了操作的多个结果，它是一个 Hot Obserable（不论 "事件源" 是否有 "订阅者"，都会在创建后对事件进行发布，所以对于 Hot Observable 的每一个 "订阅者" 都有可能是从 "事件源" 的中途开始的，并可能只是看到了整个操作的局部过程）。toObservable()： 同样会返回 Observable 对象，也代表了操作的多个结果，但它返回的是一个Cold Observable（没有 "订阅者" 的时候并不会发布事件，而是进行等待，直到有 "订阅者" 之后才发布事件，所以对于 Cold Observable 的订阅者，它可以保证从一开始看到整个操作的全部过程）

3. 若当前命令的请求缓存功能是被启用的， 并且该命令缓存命中， 那么缓存的结果会立即以 Observable 对象的形式 返回。

4. 检查断路器是否为打开状态。如果断路器是打开的，那么Hystrix不会执行命令，而是转接到 fallback 处理逻辑（第 8 步）；如果断路器是关闭的，检查是否有可用资源来执行命令（第 5 步）。

5. 线程池/请求队列/信号量是否占满。如果命令依赖服务的专有线程池和请求队列，或者信号量（不使用线程池的时候）已经被占满， 那么 Hystrix 也不会执行命令， 而是转接到 fallback 处理逻辑（第8步）。

6. Hystrix 会根据我们编写的方法来决定采取什么样的方式去请求依赖服务。HystrixCommand.run() ：返回一个单一的结果，或者抛出异常。HystrixObservableCommand.construct()： 返回一个Observable 对象来发射多个结果，或通过 onError 发送错误通知。

7. Hystrix会将 "成功"、"失败"、"拒绝"、"超时" 等信息报告给断路器， 而断路器会维护一组计数器来统计这些数据。断路器会使用这些统计数据来决定是否要将断路器打开，来对某个依赖服务的请求进行 "熔断/短路"。

8. 当命令执行失败的时候， Hystrix 会进入 fallback 尝试回退处理， 我们通常也称该操作为 "服务降级"。而能够引起服务降级处理的情况有下面几种：第4步： 当前命令处于"熔断/短路"状态，断路器是打开的时候。第5步： 当前命令的线程池、 请求队列或 者信号量被占满的时候。第6步：HystrixObservableCommand.construct() 或 HystrixCommand.run() 抛出异常的时候。

9. 当Hystrix命令执行成功之后， 它会将处理结果直接返回或是以Observable 的形式返回。

tips：如果我们没有为命令实现降级逻辑或者在降级处理逻辑中抛出了异常， Hystrix 依然会返回一个 Observable 对象， 但是它不会发射任何结果数据， 而是通过 onError 方法通知命令立即中断请求，并通过onError()方法将引起命令失败的异常发送给调用者。

### 1.6.5 服务监控hystrixDashboard

新建module

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

主启动@EnableHystrixDashboard

监控module

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

新版本Hystrix需要在主启动类MainAppHystrix8001中指定监控路径

```java
/**
 *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
 *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
 *只要在自己的项目里配置上下面的servlet就可以了
 */
@Bean
public ServletRegistrationBean getServlet() {
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}

```

填写监控地址  http://localhost:8001/hystrix.stream

1：Delay：该参数用来控制服务器上轮询监控信息的延迟时间，默认为2000毫秒，可以通过配置该属性来降低客户端的网络和CPU消耗。

2：Title：该参数对应了头部标题Hystrix Stream之后的内容，默认会使用具体监控实例的URL，可以通过配置该信息来展示更合适的标

![image-20210421111554289](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210421111554289.png)	

![image-20210421111515110](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210421111515110.png)

## 1.7 zuul路由网关

https://github.com/Netflix/zuul/wiki/Getting-Started

https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.1.RELEASE/reference/html/#router-and-filter-zuul

基于JVM路由和服务端的负载均衡器  提供动态路由、监视、弹性、安全性等功能的边缘服务

代理+路由+过滤三大功能

作用

1. 路由
2. 过滤
3. 负载均衡
4. 灰度发布

### 1.7.1 demo

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-zuul-gateway

eureka:
  client:
    service-url:
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true 
```

hosts修改   127.0.0.1  myzuul.com

主启动类   @EnableZuulProxy

测试    

```xml
zuul映射配置+注册中心注册后对外暴露的服务名称+rest调用地址

http://myzuul.com:9527/cloud-provider-payment/paymentInfo
```

由于Zuul自动集成了Ribbon和Hystrix，所以Zuul天生就有负载均衡和服务容错能力

```yml
zuul:
  prefix: /atguigu  #设置统一公共前缀
  ignored-services: cloud-provider-payment  #忽略默认路由配置  多个可以用"*"
  routes: # 路由映射配置
    mypayment.serviceId: cloud-provider-payment
    mypayment.path: /weixin/**
    mysms.serviceId: cloud-provider-sms
    mysms.path: /mysms/**
```

### 1.7.2 查看路由信息

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yml
# 开启查看路由的端点
management:
  endpoints:
    web:
      exposure:
        include: 'routes' 
```

http://localhost:9527/actuator/routes

### 1.7.3 过滤器

ZuulFilter

过滤类型

1. pre：在请求被路由到目标服务前执行，比如权限校验、打印日志等功能；
2. routing：在请求被路由到目标服务时执行
3. post：在请求被路由到目标服务后执行，比如给目标服务的响应添加头信息，收集统计数据等功能；
4. error：请求在其他阶段发生错误时执行。

过滤顺序  数字小的先执行

过滤是否开启    shouldFilter方法为true走

执行逻辑   自己的业务逻辑

demo 前置过滤器，用于在请求路由到目标服务前打印请求日志

```java
@Component
@Slf4j
public class PreLogFilter extends ZuulFilter{
    @Override
    public String filterType(){
        return "pre";
    }
    @Override
    public int filterOrder(){
        return 1;
    }
    @Override
    public boolean shouldFilter(){
        return true;
    }
    @Override
    public Object run() throws ZuulException{
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();
        String host = request.getRemoteHost();
        String method = request.getMethod();
        String uri = request.getRequestURI();
        //log.info("=====> Remote host:{},method:{},uri:{}", host, method, uri);
        System.out.println("********"+new Date().getTime());
        return null;
    }
}
```

```yml
zuul:
  PreLogFilter:
    pre:
      disable: true#开关，YML配置
```







## 1.8 Gateway新一代网关

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

上一代zuul 1.X https://github.com/Netflix/zuul/wiki

Gateway是在Spring生态系统之上构建的API网关服务，基于Spring 5，Spring Boot 2和 Project Reactor等技术。
Gateway旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能， 例如：安全，监控/指标，熔断、限流、重试等

<font color='red'>使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架</font>

**作用**

1. 反向代理
2. 鉴权
3. 流量控制
4. 熔断
5. 日志监控

**Spring Cloud Gateway 具有如下特性：**

- 基于Spring Framework 5, Project Reactor 和 Spring Boot 2.0 进行构建；
- 动态路由：能够匹配任何请求属性；
- 可以对路由指定 Predicate（断言）和 Filter（过滤器）；
- 集成Hystrix的断路器功能；
- 集成 Spring Cloud 服务发现功能；
- 易于编写的 Predicate（断言）和 Filter（过滤器）；
- 请求限流功能；
- 支持路径重写。

**Spring Cloud Gateway 与 Zuul的区别**

在SpringCloud Finchley 正式版之前，Spring Cloud 推荐的网关是 Netflix 提供的Zuul：

1. Zuul 1.x，是一个基于阻塞 I/ O 的 API Gateway

2. Zuul 1.x 基于Servlet 2. 5使用阻塞架构它不支持任何长连接(如 WebSocket) Zuul 的设计模式和Nginx较像，每次 I/ O 操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是Nginx 用C++ 实现，Zuul 用 Java 实现，而 JVM 本身会有第一次加载较慢的情况，使得Zuul 的性能相对较差。

3. Zuul 2.x理念更先进，想基于Netty非阻塞和支持长连接，但SpringCloud目前还没有整合。 Zuul 2.x的性能较 Zuul 1.x 有较大提升。在性能方面，根据官方提供的基准测试， Spring Cloud Gateway 的 RPS（每秒请求数）是Zuul 的 1. 6 倍。

4. Spring Cloud Gateway 建立 在 Spring Framework 5、 Project Reactor 和 Spring Boot 2 之上， 使用非阻塞 API。

5. Spring Cloud Gateway 还 支持 WebSocket， 并且与Spring紧密集成拥有更好的开发体验

   

**Zuul1.x模型**

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。

servlet由servlet container进行生命周期管理。
container启动时构造servlet对象并调用servlet init()进行初始化；
container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用service()。
container关闭时调用servlet destory()销毁servlet；

![image-20210421114855630](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210421114855630.png)

上述模式的缺点：
servlet是一个简单的网络IO模型，当请求进入servlet container时，servlet container就会为其绑定一个线程，在并发不高的场景下这种模型是适用的。但是一旦高并发(比如抽风用jemeter压)，线程数量就会上涨，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间。在一些简单业务场景下，不希望为每个request分配一个线程，只需要1个或几个线程就能应对极大并发的请求，这种业务场景下servlet模型没有优势

所以Zuul 1.X是基于servlet之上的一个阻塞式处理模型，即spring实现了处理所有request请求的一个servlet（DispatcherServlet）并由该servlet阻塞式处理处理。所以Springcloud Zuul无法摆脱servlet模型的弊端



**WebFlux**

https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-new-framework

传统的Web框架，比如说：struts2，springmvc等都是基于Servlet API与Servlet容器基础之上运行的。
但是
<font color='red'>在Servlet3.1之后有了异步非阻塞</font>的支持。而WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程（Spring5必须让你使用java8）

Spring WebFlux 是 Spring 5.0 引入的新的响应式框架，区别于 Spring MVC，它不需要依赖Servlet API，它是完全异步非阻塞的，并且基于 Reactor 来实现响应式流规范。

### 1.8.1 三大核心概念

1. Route(路由)   路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

2. Predicate(断言)     参考的是Java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数)，如果请求与断言相匹配则进行路由

3. Filter(过滤)    指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。
predicate就是我们的匹配条件；
filter，就可以理解为一个无所不能的拦截器。有了这两个元素，再加上目标uri，就可以实现一个具体的路由了

### 1.8.2 Gateway工作流程

![image-20210421134724114](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210421134724114.png)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。

Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

Filter在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，
在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

核心逻辑 : <font color="red">路由转发+执行过滤器链</font>

### 1.8.3 demo

```xml
<!--gateway-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

```java
主启动
@SpringBootApplication
@EnableEurekaClient
```

Gateway网关路由有两种配置方式：

1. yml

2. 代码中注入RouteLocator的Bean

   ```java
   @Configuration
   public class GateWayConfig{
       /**
        * 配置了一个id为route-name的路由规则，
        * 当访问地址 http://localhost:9527/guonei时会自动转发到地址：http://news.baidu.com/guonei
        * @param builder
        * @return
        */
       @Bean
       public RouteLocator customRouteLocator(RouteLocatorBuilder builder){
           RouteLocatorBuilder.Builder routes = builder.routes();
   
           routes.route("path_route_atguigu", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
   
           return routes.build();
   
       }
       @Bean
       public RouteLocator customRouteLocator2(RouteLocatorBuilder builder){
           RouteLocatorBuilder.Builder routes = builder.routes();
           routes.route("path_route_atguigu2", r -> r.path("/guoji").uri("http://news.baidu.com/guoji")).build();
           return routes.build();
       }
   }
   ```

### 1.8.4 通过微服务名实现动态路由

默认情况下Gateway会根据注册中心注册的服务列表，
以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

需要注意的是uri的协议为lb，表示启用Gateway的负载均衡功能。

lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri

### 1.8.5 Predicate的使用

Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。
Spring Cloud Gateway包括许多内置的Route Predicate工厂。所有这些Predicate都与HTTP请求的不同属性匹配。多个Route Predicate工厂可以进行组合

Spring Cloud Gateway 创建 Route 对象时， 使用 RoutePredicateFactory 创建 Predicate 对象，Predicate 对象可以赋值给 Route。 Spring Cloud Gateway 包含许多内置的Route Predicate Factories。

所有这些谓词都匹配HTTP请求的不同属性。多种谓词工厂可以组合，并通过逻辑and



1. After Route Predicate

   ```java
   ZonedDateTime zbj = ZonedDateTime.now(); // 默认时区
   System.out.println(zbj);
   ```

   ```yml
   uri: lb://cloud-payment-service #匹配后提供服务的路由地址
   predicates:
       - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
       - After=2020-02-05T15:10:03.685+08:00[Asia/Shanghai]         # 断言，路径相匹配的进行路由
   ```

2. Before Route Predicate

   ```yml
   - Before=2020-02-05T15:10:03.685+08:00[Asia/Shanghai]         # 断言，路径相匹配的进行路由
   ```

3. Between Route Predicate

   ```yml
   - Between=2020-02-02T17:45:06.206+08:00[Asia/Shanghai],2020-03-25T18:59:06.206+08:00[Asia/Shanghai]
   ```

4. Cookie Route Predicate 

   ```yml
   - Cookie=username,zzyy
   #Cookie Route Predicate需要两个参数，一个是 Cookie name ,一个是正则表达式。
   #路由规则会通过获取对应的 Cookie name 值和正则表达式去匹配
   ```

5. Header Route Predicate

   ```yml
   - Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
   #两个参数：一个是属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行
   ```

6. Host Route Predicate

   ```yml
   #Host Route Predicate 接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用.号作为分隔符。
   它通过参数中的主机地址作为匹配规则。
   - Host=**.atguigu.com
   ```

7. Method Route Predicate

   ```yml
   - Method=GET
   ```

8. Path Route Predicate

   ```yml
   - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
   ```

9. Query Route Predicate

   ```yml
   #支持传入两个参数，一个是属性名，一个为属性值，属性值可以是正则表达式
   - Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
   ```

### 1.8.6 Filter的使用

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。

种类

1. GatewayFilter

2. GlobalFilter

生命周期   pre    post

常用的GatewayFilter   AddRequestParameter   demo

```yml
 routes:
   filters:
            - AddRequestParameter=X-Request-Id,1024 #过滤器工厂会在匹配的请求头加上一对请求头，名称为X-Request-Id值为1024
```

**自定义过滤器**

全局日志记录, 统一网关鉴权...

implements GlobalFilter,Ordered

```java
@Component //必须加，必须加，必须加
public class MyLogGateWayFilter implements GlobalFilter,Ordered{
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
        System.out.println("time:"+new Date()+"\t 执行了自定义的全局过滤器: "+"MyLogGateWayFilter"+"hello");
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            System.out.println("****用户名为null，无法登录");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
    @Override
    public int getOrder(){
        return 0;
    }
}

```

## 1.9 SpringCloud Config分布式配置中心

https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/

SpringCloud Config分为服务端和客户端两部分。

服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

作用

1. 集中管理配置文件
2. 分环境部署
3. 运行期间动态调整配置
4. 配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
5. 将配置信息以REST接口的形式暴露

与GitHub整合配置

由于SpringCloud Config默认使用Git来存储配置文件(也有其它方式,比如支持SVN和本地文件)，
但最推荐的还是Git，而且使用的是http/https访问的形式

### 1.9.1 Config服务端配置与测试

在GitHub上新建一个名为springcloud-config的新Repository

clone 刚新建的git地址

新建Module模块cloud-config-center-3344 它即为Cloud的配置中心模块cloudConfig Center

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

```yml
server:
  port: 3344
spring:
  application:
    name:  cloud-config-center #注册进Eureka服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: git@github.com:zzyybs/springcloud-config.git #GitHub上面的git仓库名字
        ####搜索目录
          search-paths:
            - springcloud-config
      ####读取分支
      label: master
#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

主启动@EnableConfigServer

windows下修改hosts文件，增加映射   127.0.0.1  config-3344.com

**配置读取规则**

1. /{label}/{application}-{profile}.yml

   ```yml
   http://config-3344.com:3344/master/config-dev.yml    #master分支
   http://config-3344.com:3344/dev/config-dev.yml       #dev分支
   ```

2. /{application}-{profile}.yml

   ```yml
   http://config-3344.com:3344/config-dev.yml    #读默认分支
   ```

3. /{application}/{profile}[/{label}]

   ```yml
   http://config-3344.com:3344/config/dev/master
   
   /{name}-{profiles}.yml 
   /{label}-{name}-{profiles}.yml
   label：分支(branch)
   name ：服务名
   profiles：环境(dev/test/prod)
   ```

### 1.9.2 Config客户端配置与测试

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

 Spring Cloud会创建一个“Bootstrap Context”，作为Spring应用的`Application Context`的父上下文。初始化的时候，`Bootstrap Context`负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。

`Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。

`Bootstrap context`和`Application Context`有着不同的约定，所以新增了一个`bootstrap.yml`文件，保证`Bootstrap Context`和`Application Context`配置的分离。

bootstrap.yml

```yml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://localhost:3344 #配置中心地址k

#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
```

### 1.9.3 Config客户端之动态刷新

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

controller加@RefreshScope

发送Post请求刷新     curl -X POST "http://localhost:3355/actuator/refresh"

## 1.10 SpringCloud Bus消息总线

Spring Cloud Bus 配合 Spring Cloud Config 使用可以实现配置的动态刷新。

Bus支持两种消息代理：RabbitMQ 和 Kafka

Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了Java的事件处理机制和消息中间件的功能。
Spring Cloud Bus能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道



什么是总线
在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。



基本原理
ConfigClient实例都监听MQ中同一个topic(默认是springCloudBus)。当一个服务刷新数据的时候，它会把这个信息放入到Topic中，这样其它监听同一Topic的服务就能得到通知，然后去更新自身的配置

### 1.10.1 RabbitMQ环境配置

安装Erlang  http://erlang.org/download/otp_win64_21.3.exe

安装RabbitMQ   https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/rabbitmq-server-3.7.14.exe

进入RabbitMQ安装目录下的sbin目录

输入命令启动管理功能   rabbitmq-plugins enable rabbitmq_management

访问RabbitMQ   http://localhost:15672/    guest guest

### 1.10.2 动态刷新全局广播

设计思想

1. 利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置

2. 利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，而刷新所有客户端的配置

2架构显然更加适合，1不适合的原因如下

- 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新的职责。
- 破坏了微服务各节点的对等性。
- 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改



配置中心服务端添加消息总线支持

```xml
<!--添加消息总线RabbitMQ支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

```yml
#rabbitmq相关配置
rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
##rabbitmq相关配置,暴露bus刷新配置的端点
management:
  endpoints: #暴露bus刷新配置的端点
    web:
      exposure:
        include: 'bus-refresh'
```

客户端添加消息总线支持

```xml
<!--添加消息总线RabbitMQ支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

```yml
#rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

一次发送，处处生效   curl -X POST "http://localhost:3344/actuator/bus-refresh"

### 1.10.3 动态刷新定点通知

公式：http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}

/bus/refresh请求不再发送到具体的服务实例上，而是发给config server并通过destination参数类指定需要更新配置的服务或实例

destination = moduleName+port

demo  curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"

## 1.11 SpringCloud Stream消息驱动

https://spring.io/projects/spring-cloud-stream#overview

https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.1.RELEASE/reference/html/

中文指导手册 https://m.wang1314.com/doc/webapp/topic/20971999.html

 Spring Cloud Stream 是一个构建消息驱动微服务的框架。

应用程序通过 inputs 或者 outputs 来与 Spring Cloud Stream中binder对象交互。
通过我们配置来binding(绑定) ，而 Spring Cloud Stream 的 binder对象负责与消息中间件交互。
所以，我们只需要搞清楚如何与 Spring Cloud Stream 交互就可以方便使用消息驱动的方式。

通过使用Spring Integration来连接消息代理中间件以实现消息事件驱动。
Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布-订阅、消费组、分区的三个核心概念。

**目前仅支持RabbitMQ、Kafka。**



屏蔽底层消息中间件的差异,降低切换成本，统一消息的编程模型

### 1.11.1 设计思想

标准MQ

- 生产者/消费者之间靠消息媒介传递信息内容  Message
- 消息必须走特定的通道   消息通道MessageChannel
- 消息通道里的消息如何被消费呢，谁负责收发处理   消息通道MessageChannel的子接口SubscribableChannel，由MessageHandler消息处理器所订阅

Cloud Stream

通过定义绑定器Binder作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离。

两种类型

- INPUT对应于消费者
- OUTPUT对应于生产者

Stream中的消息通信方式遵循了发布-订阅模式  Topic主题进行广播----在RabbitMQ就是Exchange在Kakfa中就是Topic



**Spring Cloud Stream标准流程**

Binder   连接中间件，屏蔽差异

Channel   通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置

Source和Sink   简单的可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入。

常用api和注解

![-](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210422163025527.png)

### 1.11.2 消息驱动之生产者

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

```yml
spring:
  application:
    name: cloud-stream-provider
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          output: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
```

```java
@EnableBinding(Source.class) // 可以理解为是一个消息的发送管道的定义
public class MessageProviderImpl implements IMessageProvider{
    @Resource
    private MessageChannel output; // 消息的发送管道

    @Override
    public String send(){
        String serial = UUID.randomUUID().toString();
        this.output.send(MessageBuilder.withPayload(serial).build()); // 创建并发送消息
        System.out.println("***serial: "+serial);
        return serial;
    }
}
```

### 1.11.3消费者 

```yml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
```

```java
@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListener{
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
        System.out.println("消费者1号，------->接收到的消息：" + message.getPayload()+"\t port: "+serverPort);
    }
}
```

### 1.11.4 分组和持久化

不同组是可以全面消费的(重复消费)，
同一组内会发生竞争关系，只有其中一个可以消费。

```yml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称，在分析具体源代码的时候会进行说明
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
       binder: defaultRabbit # 设置要绑定的消息服务的具体设置
             group: atguiguA  #<----------------添加即分组和持久化---------------
```

## 1.12 SpringCloud Sleuth分布式请求链路跟踪

链路追踪组件有Google的Dapper，Twitter的Zipkin，以及阿里的Eagleeye（鹰眼）等



https://github.com/spring-cloud/spring-cloud-sleuth

Spring Cloud Sleuth提供了一套完整的服务跟踪的解决方案

在分布式系统中提供追踪解决方案并且兼容支持了zipkin 可视化

1. **zipkin**      zipkin-server-2.12.9-exec.jar  

   SpringCloud从F版起已不需要自己构建Zipkin Server了，只需调用jar包即可

   https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

   运行jar    http://localhost:9411/zipkin/

   一条链路通过Trace Id唯一标识，Span标识发起的请求信息，各span通过parent id 关联起来

   * Trace:类似于树结构的Span集合，表示一条调用链路，存在唯一标识

     Trace(跟踪):一系列Span组成的一个树状结构。请求一个微服务系统的AP接口，这个AP接口，需要调用多个微服务，调用每个微服务都会产生一个新的Span，所有由这个请求产生的Span组成了这个Traceo

   * span:表示调用链路来源，通俗的理解span就是一次请求信息

     Span（跨度），基本工作单元，发送一个远程调度任务就会产生一个Span，Span是一个64位ID唯一标识的，Trace是用另一个64位ID唯一标识的，Span还有其他数据信息，比如摘要、时间戳事件、Span的ID、以及进度ID。

   * Annotation(标注）:用来及时记录一个事件的，一些核心注解用来定义一个请求的开始和结束。这些注解包括以下:
     	cs - Client Sent-客户端发送一个请求，这个注解描述了这个Span 的开始
       	sr-Server Received -服务端获得请求并准备开始处理它,如果将其5r减去cs时间戳便可得到网络传输的时间。
       	ss- Server Sent（服务端发送响应）–该注解表明请求处理的完成(当请求返回客户端)，如果ss的时间戳减去sr时间戳，就可以得到服务器请求的时间。
       	cr-Client Received（客户端接收响应）-此时Span的结束，如果cr,的时间戳减去cs时间戳便可以得到整个请求所消耗的时间。
     官方文档:
     https://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.1.3.RELEASE/single/spring-cloud-sleuth.html

2. 服务提供者

   ```xml
   <!--包含了sleuth+zipkin-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zipkin</artifactId>
   </dependency>
   ```

   ```yml
   spring:
     zipkin:
       base-url: http://localhost:9411 #zipkin服务器
       discovery-client-enabled: false #关闭服务发现，否则spring cloud会把zipkin的url当做服务名称
       sender:
         type: web  #设置使用http的方式传输数据
     sleuth:
       sampler:
         #采样率值介于 0 到 1 之间，1 则表示全部采集
        probability: 1
   ```

3. 服务消费者

   ```xml
   <!--包含了sleuth+zipkin-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zipkin</artifactId>
   </dependency>
   ```

   ```yml
   spring:
     zipkin:
       base-url: http://localhost:9411 #zipkin服务器
       discovery-client-enabled: false #关闭服务发现，否则spring cloud会把zipkin的url当做服务名称
       sender:
         type: web  #设置使用http的方式传输数据
     sleuth:
       sampler:
         #采样率值介于 0 到 1 之间，1 则表示全部采集
        probability: 1
   ```

3. ```shell
   #安装zipkin服务器
   docker run -d -p 9411:9411 openzipkin/zipkin
   ```

   查看超过100ms的请求，是否可以优化
   结合zipkin找到慢请求，该降级的降级，该限流的限流

4. 数据持久化

   使用es

   ```shell
   docker run --env STORAGE_TYPE=elasticsearch --env ES_HOSTS=192.168.56.10:9200 openzipkin/zipkin-dependencies
   ```

   

# spring cloud alibaba

https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md

https://spring.io/projects/spring-cloud-alibaba#overview

## 1.1 Nacos

https://github.com/alibaba/Nacos

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery

https://nacos.io/zh-cn/index.html

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_config

支持负载均衡 ( 集成ribbon )

### 1.1.1各种注册中心对比

![image-20210419091816588](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419091816588.png)

![image-20210419091838325](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419091838325.png)

<font color="red">Nacos 支持AP和CP模式的切换</font> 

<font color="red">C是所有节点在同一时间看到的数据是一致的；而A的定义是所有的请求都会收到响应。</font>



C:Consistency（强一致性）A:Availability（可用性）P:Partition tolerance（分区容错性）

CAP理论关注粒度是数据，而不是整体系统设计的策略 CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，
因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：
CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。
AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些



何时选择使用何种模式？
一般来说，
如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式。当前主流的服务如 Spring cloud 和 Dubbo 服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

如果需要在服务级别编辑或者存储配置信息，那么 CP 是必须，K8S服务和DNS服务则适用于CP模式。
CP模式下则支持注册持久化实例，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。

```cmd
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
```

### 1.1.2 Nacos作为配置中心和注册中心

#### 1.1.2.1 基础配置

<font color="red">bootstrap优先级高于application</font>

bootstrap

```yml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
```

application

```yml
spring:
  profiles:
    active: dev # 表示开发环境
```

```java
@EnableDiscoveryClient
```

```java
@RefreshScope 
//在控制器类加入@RefreshScope注解使当前类下的配置支持Nacos的动态刷新功能。
//通过Spring Cloud原生注解@RefreshScope 实现配置自动更新:
```

Nacos中的dataid的组成格式及与SpringBoot配置文件中的匹配规则

https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html

```yml
#${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
#nacos-config-client-dev.yaml
```

配置文件的历史版本默认保留30天，一键回滚，会触发配置更新

#### 1.1.2.2 分类配置

namespace用于区分部署环境，Group和DataID逻辑上区分两个目标对象

![image-20210419185913671](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419185913671.png)



默认情况：
Namespace=public，Group=DEFAULT_GROUP, 默认Cluster是DEFAULT

Nacos默认的命名空间是public，Namespace主要用来实现隔离。
比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。

Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组里面去

Service就是微服务；一个Service可以包含多个Cluster（集群），Nacos默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分。
比方说为了容灾，将Service微服务分别部署在了杭州机房和广州机房，
这时就可以给杭州机房的Service微服务起一个集群名称（HZ），
给广州机房的Service微服务起一个集群名称（GZ），还可以尽量让同一个机房的微服务互相调用，以提升性能。

最后是Instance，就是微服务的实例。

```yml
spring:
  profiles:
    active: demo   #dataid = wms-iow-provider-demo.yml
  application:
    name: wms-iow-provider #设置项目名称
  cloud:
  nacos:
    discovery:
      server-addr: ${NACOS_HOST:localhost:8848} 
      namespace: ${NAME_SPACE:mesdev}
    config:
      server-addr: ${NACOS_HOST:localhost:8848}
      file-extension: yml 
      namespace: ${NAME_SPACE:mesdev}
      group: ${group-id}   
```

### 1.1.3 Nacos集群和持久化配置

https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

https://nacos.io/zh-cn/docs/deployment.html

![image-20210419193625255](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419193625255.png)

#### 1.1.3.1 nacos持久化

Nacos默认自带的是嵌入式数据库derby

https://github.com/alibaba/nacos/blob/develop/config/pom.xml

derby到mysql切换配置步骤

1. nacos-server-1.1.4\nacos\conf目录下找到sql脚本     建库运行

2. nacos-server-1.1.4\nacos\conf目录下找到application.properties  修改mysql连接

#### 1.1.3.2 Linux版Nacos+MySQL生产环境配置

1个Nginx+3个nacos注册中心+1个mysql

https://github.com/alibaba/nacos/releases/tag/1.1.4

1. Linux服务器上mysql数据库配置 建库建表
2. application.properties 配置 mysql连接
3. 配置nacos/conf/cluster.conf   3台nacos的不同服务端口号写入    这个IP不能写127.0.0.1，必须是Linux命令hostname -i能够识别的IP

![image-20210419195104841](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419195104841.png)

4. 编辑Nacos的启动脚本startup.sh，使它能够接受不同的启动端口

   ![image-20210419195827557](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419195827557.png)

5. Nginx的配置，由它作为负载均衡器
   
   1. 修改nginx/conf/nginx.conf配置文件
   
      ![image-20210419201003177](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419201003177.png)
   
   2. 按照指定启动
   
      ```cmd
      /usr/local/nginx/sbin 
      . /nginx -c /usr/local/nginx/conf/nginx.conf
      ```

6. 测试通过nginx访问nacos   http://192.168.111.144:1111/nacos/#/login

7. 微服务cloudalibaba-provider-payment9002启动注册进nacos集群

   ```yml
    server-addr: 192.168.111.144:1111
   ```

## 1.2 Sentinel

https://github.com/alibaba/Sentinel

https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D

https://github.com/alibaba/Sentinel/releases

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel

![image-20210420170314658](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210420170314658.png)

降级分类

* 调用方的熔断保护：feign.sentinel.enable=true
* 调用方手动指定远程服务的降级策略。远程服务被降级处理。触发我们的熔断回调方法
* 超大浏览的时候，必须牺牲一些远程服务。在服务的提供方（远程服务）指定降级策略；
  提供方是在运行，但是不允许自己的业务逻辑，返回的是默认的降级数据（限流的数据）

自定义降级资源

1. ```java
   try (Entry entry = SphU.entry("seckillSkus")) {
       //业务逻辑
   } catch(Exception e) {}
   ```

2. @SentinelResource

3. 使用dashboard

   ```java
   /**
    * @Description: 自定义阻塞返回方法
    **/
   @Component
   public class PigxUrlBlockHandler implements BlockExceptionHandler {
   
       @Override
       public void handle(HttpServletRequest request, HttpServletResponse response, BlockException ex) throws IOException {
           R error = R.error(BizCodeEnume.TO_MANY_REQUEST.getCode(), BizCodeEnume.TO_MANY_REQUEST.getMsg());
           response.setCharacterEncoding("UTF-8");
           response.setContentType("application/json");
           response.getWriter().write(JSON.toJSONString(error));
       }
   }
   ```

   







### 1.2.1 demo

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>

<!--springboot 收集健康状况信息，提供给sentinel使用-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Sentinel采用懒加载

```yml
sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719
        
#actuator：暴露所有端点给sentinel监控产生图表
management:
  endpoints:
    web:
      exposure:
        include: *        
```

```shell
java -jar sentinel-dashboard-1.7.1.jar --server.port=8333 #启动dashboard
```



### 1.2.2 流控规则

![image-20210420144831478](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210420144831478.png)

![image-20210420144813812](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210420144813812.png)

#### 1.2.2.1流控模式

1. 直接(默认)   直接->快速失败
2. 关联            A关联的资源B达到阀值，限流A
3. 链路

#### 1.2.2.2 流控效果

1. 直接->快速失败(默认的流控处理)   DefaultController

2. 预热     阈值除以coldFactor(默认值为3),经过预热时长后才会达到阈值

   https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6

   WarmUpController   应用秒杀系统

3. 排队等待 RateLimiterController

   https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6

### 1.2.3 降级规则

https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7

Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。

当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

Sentinel的断路器<font color="red">没有半开状态</font>

半开的状态系统自动去检测是否请求有异常，没有异常就关闭断路器恢复使用，有异常则继续打开断路器不可用。具体可以参考Hystrix

#### 1.2.3.1 RT（平均响应时间，秒级）

​      平均响应时间   超出阈值  且   在时间窗口内即1s通过的请求>=5，两个条件同时满足后触发降级
​      窗口期过后关闭断路器
​      RT最大4900ms（更大的需要通过-Dcsp.sentinel.statistic.max.rt=XXXX才能生效）

#### 1.2.3.2 异常比列（秒级）

​    QPS >= 5 且异常比例（秒级统计）超过阈值时，触发降级；时间窗口结束后，关闭降级

#### 1.2.3.3 异常数（分钟级）

​     异常数（分钟统计）超过阈值时，触发降级；时间窗口结束后，关闭降级

​     <font color="red">时间窗口一定要大于等于60秒</font>

### 1.2.4 热点key限流

https://github.com/alibaba/Sentinel/wiki/%E7%83%AD%E7%82%B9%E5%8F%82%E6%95%B0%E9%99%90%E6%B5%81

热点即经常访问的数据，很多时候我们希望统计或者限制某个热点数据中访问频次最高的TopN数据，并对其访问进行限流或者其它操作

源码 BlockException

```java
@SentinelResource(value = "testHotKey",blockHandler = "dealHandler_testHotKey")
//访问某个参数超过阈值 走dealHandler_testHotKey方法
```

#### 1.2.4.1参数例外项    

p1参数当它是某个特殊值时，它的限流值和平时不一样

**热点参数的注意点，参数必须是基本类型或者String**

### 1.2.5 系统规则

https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%87%AA%E9%80%82%E5%BA%94%E9%99%90%E6%B5%81

系统层面限流

### 1.2.6 @SentinelResource

注解方式不支持private方法   

```java
 	@GetMapping("/byResource")
    @SentinelResource(value = "byResource",blockHandler = "handleException"
                      ,fallback="异常返回方法")
    public CommonResult byResource(){
        return new CommonResult(200,"OK",new Payment(2020L,"serial001"));
    }
    public CommonResult handleException(BlockException exception){
        return new CommonResult(444,exception.getClass().getCanonicalName()+"\t 服务不可用");
    }
```

自定义限流处理

```java
public class CustomerBlockHandler {
    public static CommonResult handlerException(BlockException exception) {
        return new CommonResult(4444, "按客戶自定义,global handlerException----1");
    }

    public static CommonResult handlerException2(BlockException exception) {
        return new CommonResult(4444, "按客戶自定义,global handlerException----2");
    }
}
```

```java
 @GetMapping("/rateLimit/customerBlockHandler")
    @SentinelResource(value = "customerBlockHandler",
            blockHandlerClass = CustomerBlockHandler.class,
            blockHandler = "handlerException2")
    public CommonResult customerBlockHandler() {
        return new CommonResult(200, "按客戶自定义", new Payment(2020L, "serial003"));
    }
```

Sentinel主要有三个核心Api

1. SphU定义资源
2. Tracer定义统计
3. ContextUtil定义了上下文

### 1.2.7 服务熔断功能

sentinel整合ribbon+openFeign+fallback

fallback管运行异常

blockHandler管配置违规

blockHandler 和 fallback 都配置，则被限流降级而抛出 BlockException 时只会进入 blockHandler 处理逻辑。

#### 1.2.7.1 ribbon

```java
@SentinelResource(value = "fallback", fallback = "handlerFallback", 
                  blockHandler = "blockHandler",
            exceptionsToIgnore = {IllegalArgumentException.class})
    public CommonResult<Payment> fallback(@PathVariable Long id) {}

 //本例是fallback
public CommonResult handlerFallback(@PathVariable Long id, Throwable e) {
        Payment payment = new Payment(id, "null");
        return new CommonResult<>(444, "兜底异常handlerFallback,exception内容  " + e.getMessage(), payment);
    }

//本例是blockHandler
public CommonResult blockHandler(@PathVariable Long id, BlockException blockException) {
        Payment payment = new Payment(id, "null");
        return new CommonResult<>(445, "blockHandler-sentinel限流,无此流水: blockException  " + blockException.getMessage(), payment);
    }
```

```java
//忽略属性 exceptionsToIgnore 异常会直接抛出
@RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback", 
                      fallback = "handlerFallback", 
                      blockHandler = "blockHandler",
            exceptionsToIgnore = {IllegalArgumentException.class})
    public CommonResult<Payment> fallback(@PathVariable Long id)
    {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/"+id,CommonResult.class,id);
        if (id == 4) {
            throw new IllegalArgumentException ("非法参数异常....");
        }else if (result.getData() == null) {
            throw new NullPointerException ("NullPointerException,该ID没有对应记录");
        }
        return result;
    }
```

#### 1.2.7.2 openFeign

```yml
# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true  
```

主启动添加@EnableFeignClients启动Feign的功能

```java
@FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
```

### 1.2.8 规则持久化

持久化进Nacos保存

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

```yml
spring:
  cloud:
    sentinel:
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: ${spring.application.name}
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

添加Nacos业务规则配置   json   dataid与xml中相同

```json
[
    {
        "resource": "/rateLimit/byUrl",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
resource：资源名称；
limitApp：来源应用；
grade：阈值类型，0表示线程数，1表示QPS；
count：单机阈值；
strategy：流控模式，0表示直接，1表示关联，2表示链路；
controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；
clusterMode：是否集群。
```

### 1.2.9网关流控

https://github.com/alibaba/Sentinel/wiki/%E7%BD%91%E5%85%B3%E9%99%90%E6%B5%81

spring cloud gateway

```xml
<!--网关流控-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>
```

```properties
#设置回调状态码
spring.cloud.sentinel.scg.fallback.response-status=400
```

```java
/**
 * @Description: 自定义阻塞返回方法
 **/
@Component
public class SentinelGatewayConfig {

    public SentinelGatewayConfig() {
        GatewayCallbackManager.setBlockHandler(new BlockRequestHandler(){
            // 网关限流了请求，就会调用此回调
            @Override
            public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
                R error = R.error(BizCodeEnume.TO_MANY_REQUEST.getCode(), BizCodeEnume.TO_MANY_REQUEST.getMsg());
                String errJson = JSON.toJSONString(error);
                return ServerResponse.ok().body(Mono.just(errJson), String.class);
            }
        });
    }
}
```





## 1.3 Seata

分布式事务问题

一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用

Seata--开源的分布式事务解决方案

多种模式：AT、TCC、SAGA 和 XA



AT模式：Auto Transiaction：自动事务模式，根据回滚日志表自动回滚
TCC模式：就是根据自己手写的事务补偿方法 来回滚

http://seata.io/zh-cn/

https://github.com/seata/seata/releases

### 1.3.1  Seata简介

#### 1.3.1.1 分布式事务处理过程的一ID+三组件模型

**Transaction ID XID**  全局唯一的事务ID

3组件

- **Transaction Coordinator (TC)**	

  事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚；

- **Transaction Manager (TM)**    

  控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议；

- **Resource Manager (RM)**   

  控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚

#### 1.3.1.2 分布式事务过程

1. TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的 XID；
2. XID 在微服务调用链路的上下文中传播；
3. RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖；
4. TM 向 TC 发起针对 XID 的全局提交或回滚决议；
5. TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求。

![image-20210414161435405](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20210414161801.png)

### 1.3.2 Seata-Server安装

http://seata.io/zh-cn/

https://github.com/seata/seata/releases

#### 1.3.2.1 .zip解压到指定目录并修改conf目录下的file.conf配置文件

先备份原始file.conf文件

主要修改：自定义事务组名称+事务日志存储模式为db+数据库连接信息

service模块

```java
 service {
 
  vgroup_mapping.my_test_tx_group = "fsp_tx_group"//自定义事务组名称

  default.grouplist = "127.0.0.1:8091"
  enableDegrade = false
  disable = false
  max.commit.retry.timeout = "-1"
  max.rollback.retry.timeout = "-1"
}
```

store模块

```java
 
## transaction log store
store {
  ## store mode: file、db
  mode = "db"		//事务日志存储模式为db
 
  ## file store
  file {
    dir = "sessionStore"
 
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    max-branch-session-size = 16384
    # globe session size , if exceeded throws exceptions
    max-global-session-size = 512
    # file buffer size , if exceeded allocate new buffer
    file-write-buffer-cache-size = 16384
    # when recover batch read size
    session.reload.read_size = 100
    # async, sync
    flush-disk-mode = async
  }
 
  ## database store
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"//数据库连接信息
    user = "root"
    password = "你自己密码"
    min-conn = 1
    max-conn = 3
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
```

#### 1.3.2.2 数据库新建库seata 建表

建表db_store.sql在\seata-server-0.9.0\seata\conf目录里面

#### 1.3.2.3修改\seata\conf目录下的registry.conf配置文件

```java
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"
 
  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
```

指明注册中心为nacos，及修改nacos连接信息

#### 1.3.2.4 启动Nacos seata-server

### 1.3.3 demo订单/库存/账户业务

下订单->减库存->扣余额->改(订单)状态

#### 1.3.3.1 数据库准备

3个库分别建对应的回滚日志表

\seata-server-0.9.0\seata\conf目录下的db_undo_log.sql

![image-20210414170017505](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210414170017505.png)



#### 1.3.3.2新建订单Order-Module库存Storage-Module账户Account-Module

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>mscloud03</artifactId>
        <groupId>com.atguigu.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>seata-order-service2001</artifactId>

    <dependencies>
        <!--nacos-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--seata-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>seata-all</artifactId>
                    <groupId>io.seata</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.seata</groupId>
            <artifactId>seata-all</artifactId>
            <version>0.9.0</version>
        </dependency>
        <!--feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--web-actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--mysql-druid-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.37</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

```yml
server:
  port: 2001

spring:
  application:
    name: seata-order-service
  cloud:
    alibaba:
      seata:
        #自定义事务组名称需要与seata-server中的对应
        tx-service-group: fsp_tx_group
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_order
    username: root
    password: 123456

feign:
  hystrix:
    enabled: false

logging:
  level:
    io:
      seata: info
mybatis:
  mapperLocations: classpath:mapper/*.xml
```

```java
/**
 * @auther zzyy
 * @create 2020-02-26 16:24
 * 使用Seata对数据源进行代理
 */
@Configuration
public class DataSourceProxyConfig {

    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }

}			
```

```java
@GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
```

### 1.3.4 原理

TC( 即seata服务)/TM/RM三大组件

- TM 开启分布式事务（TM 向 TC注册全局事务记录）

- 按业务场景，编排数据库、服务等事务内资源（RM 向 TC 汇报资源准备状态 ）

- TM 结束分布式事务，事务一阶段结束（TM 通知 TC 提交/回滚分布式事务）；
- TC 汇总事务信息，决定分布式事务是提交还是回滚；

- TC 通知所有 RM 提交/回滚 资源，事务二阶段结束。


#### 1.3.4.1 AT模式

- 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源。
- 二阶段：
  - 提交异步化，非常快速地完成。
  - 回滚通过一阶段的回滚日志进行反向补偿。

在一阶段，Seata 会拦截“业务 SQL”

1. 解析 SQL 语义，找到“业务 SQL”要更新的业务数据，在业务数据被更新前，将其保存成“before image”，

2. 执行“业务 SQL”更新业务数据，在业务数据更新之后，其保存成“after image”，最后生成行锁。

以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子

![image-20210416162639716](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210416162639716.png)

二阶段如是顺利提交的话，
因为“业务 SQL”在一阶段已经提交至数据库，所以Seata框架只需将一阶段保存的快照数据和行锁删掉，完成数据清理即可。

二阶段回滚：

二阶段如果是回滚的话，Seata 就需要回滚一阶段已经执行的“业务 SQL”，还原业务数据。

回滚方式便是用“before image”还原业务数据；但在还原前要首先要校验脏写，对比“数据库当前业务数据”和 “after image”，

如果两份数据完全一致就说明没有脏写，可以还原业务数据，如果不一致就说明有脏写，出现脏写就需要转人工处理。



### 1.3.5 Seata  步骤 - AT

http://seata.io/zh-cn/docs/overview/what-is-seata.html

1. 每一个微服务必须创建undo_log

2. 安装事务协调器：https://github.com/seata/seata/releases    1.0.0版本

3. 解压并启动seata-server

   registry.conf 注册中心配置 修改registry type=nacos

4. 导入依赖 

   ```xml
   <!--seata 分布式事务-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
   </dependency>
   ```

5. 各数据库建表

6. 每个想要使用分布式事务的微服务都要用seata DataSourceProxy代理自己的数据源

   ```java
   @Configuration
   public class MySeataConfig {
   
       @Autowired
       DataSourceProperties dataSourceProperties;
   
       /**
        * 需要将 DataSourceProxy 设置为主数据源，否则事务无法回滚
        */
       @Bean
       public DataSource dataSource(DataSourceProperties dataSourceProperties) {
           HikariDataSource dataSource = dataSourceProperties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
           if (StringUtils.hasText(dataSourceProperties.getName())) {
               dataSource.setPoolName(dataSourceProperties.getName());
           }
           return new DataSourceProxy(dataSource);
       }
   }
   ```

7. 给微服务复制两个配置文件file.conf，registry.conf

   修改file.conf：vgroup_mapping.{当前应用的名字}-fescar-service-group

8. 启动测试分布式事务

9. 给分布式大事务入口标注 @GlobalTransactional

10. 每一个远程的小事务用 @Transactional





























