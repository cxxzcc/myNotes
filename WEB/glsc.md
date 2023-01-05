nginx

拷贝原先默认的 conf

修改

![image-20211118112432103](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211118112432103.png)

 配置 UpStream

![image-20211118112450473](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211118112450473.png)

nginx代理丢失请求头信息

proxy_set_header Host  $host





使用nginx实现负载平衡的最简单配置如下,官网地址：https://nginx.org/en/docs/http/load_balancing.html



```http
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
```

gateway

```properties
- id: gulimall_host_route
  uri: lb://gulimall-product
  predicates:
    - Host=**.gulimall.com
```





# 检索

## 检索业务实现

### 检索查询参数

```java
/**
 * 封装页面所有可能传递过来的查询条件
 * @author gcq
 * @Create 2020-11-02
 */
@Data
public class SearchParam {

    /**
     * 页面传递过来的全文匹配关键字
     */
    private String keyword;
    /**
     * 三级分类id
     */
    private Long catalog3Id;
    /**
     * sort=saleCout_asc/desc
     * sort=skuPrice_asc/desc
     * sort=hotScore_asc/desc
     * 排序条件
     */
    private String sort;

    /**
     * hasStock(是否有货) skuPrice区间，brandId、catalog3Id、attrs
     */
    /**
     * 是否显示有货
     */
    private Integer hasStock = 0;
    /**
     * 价格区间查询
     */
    private String skuPrice;
    /**
     * 按照品牌进行查询，可以多选
     */
    private List<Long> brandId;
    /**
     * 按照属性进行筛选
     */
    private List<String> attrs;
    /**
     * 页码
     */
    private Integer pageNum = 1;

}
```

### 检索返回结果

```java
/**
 * 查询结果返回
 * @author gcq
 * @Create 2020-11-02
 */
@Data
public class SearchResult {
    /**
     * 查询到所有商品的商品信息
     */
    private List<SkuEsModel> products;
    /**
     * 以下是分页信息
     * 当前页码
     */
    private Integer pageNum;
    /**
     * 总共记录数
     */
    private Long total;
    /**
     * 总页码
     */
    private Integer totalPages;
    /**
     * 当前查询到的结果，所有设计的品牌
     */
    private List<BrandVo> brands;
    /**
     * 当前查询结果，所有涉及到的分类
     */
    private List<CatalogVo> catalogs;
    /**
     * 当前查询到的结果，所有涉及到的所有属性
     */
    private List<AttrVo> attrs;
    /**
     * 页码
     */
    private List<Integer> pageNavs;

    //==================以上是要返回给页面的所有信息
    @Data
    public static class BrandVo {
        /**
         * 品牌id
         */
        private Long brandId;
        /**
         * 品牌名字
         */
        private String brandName;
        /**
         * 品牌图片
         */
        private String brandImg;
    }
    @Data
    public static class CatalogVo {
        /**
         * 分类id
         */
        private Long catalogId;
        /**
         * 品牌名字
         */
        private String CatalogName;
    }

    @Data
    public static class AttrVo {
        /**
         * 属性id
         */
        private Long attrId;

        /**
         * 属性名字
         */
        private String attrName;
        /**
         * 属性值
         */
        private List<String> attrValue;
    }

}
```

















# 压测

内存泄漏、并发与同步

有效的压力测试系统将应用以下这些关键条件：重复、并发、量级、随机变化



## 性能指标

**响应时间（Response Time:RT）**
		响应时间指用户从客户端发起一个请求开始，到客户端接收到从服务器端返回的响应结束，整个过程所耗费的时间。

**HPS（Hits Per Second）:每秒点击次数，单位是次/秒。**【不是特别重要】
**TPS(Transaction per Second）:系统每秒处理交易数，单位是笔/秒。**
**Qps(Query per Second）:系统每秒处理查询次数，单位是次/秒。**
		对于互联网业务中，如果某些业务有且仅有一个请求连接，那么TPS=QPS=HPS，一般情况下用 TPS来衡量整个业务流程，用QPS来衡量接口查询次数，用HPS来表示对服务器单击请求。
**无论TPS、QPS、HPS,此指标是衡量系统处理能力非常重要的指标，越大越好，根据经验，一般情况下:**
		金融行业:1000TPS~5000OTPS，不包括互联网化的活动
		保险行业:100TPS~10000OTPS，不包括互联网化的活动
		制造行业:10TPS~5000TPS
		互联网电子商务:1000OTPS~1000000TPS
		互联网中型网站:1000TPS~50000TPS
		互联网小型网站:50OTPS~10000TPS

**最大响应时间（MaxResponse Time）**指用户发出请求或者指令到系统做出反应(响应)的最大时间。

**最少响应时间(Mininum ResponseTime）**指用户发出请求或者指令到系统做出反应(响应）的最少时间。
**90%响应时间（90%Response Time）**是指所有用户的响应时间进行排序，第90%的响应时间。

**从外部看，性能测试主要关注如下三个指标**
		**吞吐量**:每秒钟系统能够处理的请求数、任务数。
		**响应时间**:服务处理一个请求或一个任务的耗时。
		**错误率**。一批请求中结果出错的请求所占比例。

## JMeter

压力测试工具：Apache ab、Gatling、JMeter

https://jmeter.apache.org/download_jmeter.cgi 

 [apache-jmeter-5.3.zip](https://mirrors.tuna.tsinghua.edu.cn/apache//jmeter/binaries/apache-jmeter-5.3.zip)





### JMeterAddress Already in use 错误解决

```
windows本身提供的端口访问机制的问题。
Windows提供给 TCP/IP链接的端口为1024-5000，并且要四分钟来循环回收他们。就导致
我们在短时间内跑大量的请求时将端口占满了。

1.cmd中，用regedit命令打开注册表
2.在HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters下
	1.右击parameters，添加一个新的 DWORD，名字为MaxUserPort和TCPTimedWaitDelay
	2.然后双击MaxUserPort，输入数值数据为65534，基数选择十进制（如果是分布式运
行的话，控制机器和负载机器都需要这样操作哦）
	3.然后双击TCPTimedWaitDelay，输入数值数据为30，基数选择十进制（如果是分布式运
行的话，控制机器和负载机器都需要这样操作哦）
	4．修改配置完毕之后记得重启机器才会生效
https://support.microsoft.com/zh-cn/help/196271/when-you-try-to-connect-from-tcp-ports-greater-than-5000-you-receive-t

```

## 优化

影响性能考虑点包括:
	数据库、应用程序、中间件（ tomact、gateway、Nginx、）、网络（带宽）和操作系统等方面

首先考虑自己的应用属于CPU密集型还是Io密集型
	CPU：计算、排序、过滤、整合【集群】
	IO：网络、磁盘、数据库、redis【内存+缓存+固态+提高网卡的传输效率】



nginx动静分离

```nginx
location /static {
    root   /usr/share/nginx/html;
}
```



## 性能监控

![image-20211116150152816](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211116150152816.png)

### jconsole&jvisualvm



运行：正在运行的线程

休眠：sleep

等待：wait

驻留：线程池里面的空闲线程

监视：组赛的线程、正在等待锁



cmd输入jvisualvm
监控垃圾回收，安装插件
	如果不能安装插件，点击检查最新版本异常：
	1）进：https://visualvm.github.io/pluginscenters.html
	2）java -version	查看版本1.8.0_171
	3）找到对应版本：https://visualvm.github.io/uc/8u131/updates.xml.gz
	4）粘贴到设置
安装Visual GC



### 监控指标

SQL 耗时越小越好、一般情况下微妙级别

命中率越高越好、一般情况下不能低于95%

锁等待次数越低越好、等待时间越短越好

1. 中间件越多，性能损失越大【损失在网络交互】
   优化：1）优化中间件【吞吐量、响应时间】
   			2）网线、网卡、传输协议
2. 业务
   优化：1）DB（MySql优化【索引、服务器优化】）
   			2）模板的渲染速度（打开缓存）【CPU性能】
   			3）静态资源：动静分离
   			4）堆【内存爆满把应用挤下线】

#### 中间件指标

| 压测内容                                                     | 压测线程数 | 吞吐量/s          | 90%响应时间 | 99%响应时间 |
| :----------------------------------------------------------- | ---------- | ----------------- | ----------- | ----------- |
| Nginx                                                        | 50         | 2521              | 4           | 599         |
| Gateway                                                      | 50         | 2812              | 31          | 55          |
| 简单服务                                                     | 50         | 5624              | 20          | 73          |
| 首页渲染                                                     | 50         | 178(db,thymeleaf) | 442         | 729         |
| 首页渲染（开缓存）                                           | 50         | 214               | 380         | 710         |
| 首页渲染（thymeleaf开缓存+优化数据库+日志级别：error）       | 50         | 480               | 159         | 253         |
| 三级分类数据获取localhost:10000/index/catalog.json           | 50         | 2(db)             | 26311       | 27335       |
| 三级分类数据获取（加索引）                                   | 50         | 5                 | 9597        | 10176       |
| 三级分类数据获取（优化业务逻辑(一次性查询)+加索引+堆内存）   | 50         | 65                | 1150        | 1849        |
| 三级分类数据获取（优化业务逻辑(一次性查询)+加索引+堆内存+redis缓存） | 50         | 390               | 155         | 296         |
| 三级分类数据获取（优化业务逻辑(一次性查询)+加索引+堆内存+redis缓存+分布式锁） | 50         | 313               | 212         | 355         |
| 首页渲染（全量数据获取）  localhost:10000【废弃】            | 50         | 13(静态资源)      | 4916        | 6954        |
| 首页渲染（全量数据获取+动静分离）gulimall.com                | 50         | 8.2               | 8514        | 13435       |
| 首页渲染（全量数据获取+动静分离+堆优化）gulimall.com         | 50         | 8                 | 8311        | 13411       |
| Nginx+Gateway                                                | 50         |                   |             |             |
| Gateway+简单服务                                             | 50         | 1180              | 80          | 142         |
| 全链路(nginx+gateway+简单服务)                               | 50         | 532               | 126         | 226         |







































# 缓存与分布式锁

## 1.1 缓存

### 1.1.1 缓存使用

为了系统性能的提升，我们一般都会将部分数据放入缓存中，加速访问，而 db 承担数据落盘工作

**哪些数据适合放入缓存？**

- **即时性、数据一致性要求不高的**
- **访问量大且更新频率不高的数据（读多、写少）**

举例：电商类应用、商品分类，商品列表等适合缓存并加一个失效时间（根据数据更新频率来定）后台如果发布一个商品、买家需要 5 分钟才能看到新商品一般还是可以接受的



注意：在开发中，凡是放到缓存中的数据我们都应该制定过期时间，使其可以在系统即使没有主动更新数据也能自动触发数据加载的流程，避免业务奔溃导致的数据永久不一致的问题





本地缓存

```
实现：直接给service类定义一个Map
弊端：
	1、集群下的本地缓存不共享，存在于jvm中【并且负载均衡到新的机器后会重新查询】

	2、数据一致性：如果一台机器修改了数据库+缓存，但是集群下其他机器的缓存未修改

所以分布式情况下不使用本地缓存
```



### 1.1.2 整合 redis 作为缓存

#### 引入依赖

https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/using-boot-build-systems.html#using-boot-starter

```xml
<!--引入redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <!--不加载自身的 lettuce-->
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!--jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

```yml
Spring:
  redis:
    host: 192.168.56.10
    port: 6379
```

RedisAutoConfig.java

#### 测试

```java
@Autowired
StringRedisTemplate stringRedisTemplate;

@Test
public void testStringRedisTemplate() {

    stringRedisTemplate.opsForValue().set("hello","world_" + UUID.randomUUID().toString());
    String hello = stringRedisTemplate.opsForValue().get("hello");
    System.out.println("之前保存的数据是：" + hello);
}
```

#### 优化

```java
/**
 * TODO 产生堆外内存溢出 OutOfDirectMemoryError
 * 1、SpringBoot2.0以后默认使用 Lettuce作为操作redis的客户端，它使用 netty进行网络通信
 * 2、lettuce 的bug导致netty堆外内存溢出，-Xmx300m netty 如果没有指定堆内存移除，默认使用 -Xmx300m
 *      可以通过-Dio.netty.maxDirectMemory 进行设置
 *   解决方案 不能使用 -Dio.netty.maxDirectMemory调大内存
 *   1、升级 lettuce客户端，【2.3.2已解决】【lettuce使用netty吞吐量很大】2、 切换使用jedis
 *   redisTemplate:
 *   lettuce、jedis 操作redis的底层客户端，Spring再次封装二者
 * @return
 */
@Override
public Map<String, List<Catelog2Vo>> getCatelogJson() {

    // 给缓存中放 json 字符串、拿出的是 json 字符串，还要逆转为能用的对象类型【序列化和反序列化】

    // 1、加入缓存逻辑，缓存中放的数据是 json 字符串
    // JSON 跨语言，跨平台兼容
    String catelogJSON = redisTemplate.opsForValue().get("catelogJSON");
    if (StringUtils.isEmpty(catelogJSON)) {
        // 2、缓存没有，从数据库中查询
        Map<String, List<Catelog2Vo>> catelogJsonFromDb = getCatelogJsonFromDb();
        // 3、查询到数据，将数据转成 JSON 后放入缓存中
        String s = JSON.toJSONString(catelogJsonFromDb);
        redisTemplate.opsForValue().set("catelogJSON",s);
        return catelogJsonFromDb;
    }
    // 转换为我们指定的对象
    Map<String, List<Catelog2Vo>> result = JSON.parseObject(catelogJSON, new TypeReference<Map<String, List<Catelog2Vo>>>() {});

    return result;
}
```

#### RedisTemplate底层原理

```
Lettuce和Jedis是redis的客户端，RedisTemplate是对Lettuce和Jedis的再一层封装
RedisAutoConfiguration自动配置类，会导入Lettuce和Jedis的配置类
JedisConfiguration.java类会给容器放一个@Bean：：JedisConnectionFactory
```



### 1.1.3 缓存失效

#### 高并发下缓存失效问题 

**缓存穿透:**
指查询一个一定不存在的数据，由于缓存是不命中，将去查询数据库，但是数据库也无此记录,我们没有将这次查询的nul写入缓存,这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义
**风险:**
利用不存在的数据进行攻击，数据库瞬时压力增大，最终导致崩溃
**解决:**
null结果缓存，并加入短暂过期时间 【可能导致redis里面全是null，黑客每次都生成一个唯一的UUID】

布隆过滤器【不放行不存在的查询  id=UUID】【过滤器存储mysql中所有的id号】



**缓存雪崩:**
缓存雪崩是指在我们设置缓存时key采用了相同的过期时间，导致缓存在某一时刻同时失效， 请求全部转发到DB，DB瞬时压力过重雪崩。
**解决:**
原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。



**缓存击穿:**
对于一些设置了过期时间的key,如果这些key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。
如果这个key在大量请求同时进来前正好失效，那么所有对这个key的数据查询都落到db,我们称为缓存击穿。
**解决:**
加锁



**总结**

1、空结果缓存:解决缓存穿透
2、设置过期时间(加随机值) :解决缓存雪崩
3、加锁:解决缓存击穿

## 1.2 分布式锁

### 1.2.1 分布式锁原理与应用

#### 1.2.1.1 分布式锁基本原理

我们可以同时去一个地方“占坑”，如果占到，就执行逻辑。否则就必须等待，直到释放锁。“占坑' "可以去redis, 可以去数据库，可以去任何大家都能访问的地方。等待可以**自旋**的方式。

#### 1.2.1.2 分布式锁演进 - 阶段一

![image-20210427165846442](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427171355228.png)

```java
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock", "0");//redis NX操作 占锁
if (lock) {
    // 加锁成功..执行业务
    Map<String,List<Catelog2Vo>> dataFromDb = getDataFromDB();
    redisTemplate.delete("lock"); // 删除锁
    return dataFromDb;
} else {
    // 加锁失败，重试 synchronized()
    // 休眠100ms重试
    return getCatelogJsonFromDbWithRedisLock();
}
```

#### 1.2.1.3 分布式锁演进 - 阶段二

![image-20210427171048959](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427171355228.png)

```java
Boolean lock = redisTemplate.opsForValue().setIfAbsent()
    if (lock) {
        // 加锁成功..执行业务
        // 设置过期时间
        redisTemplate.expire("lock",30,TimeUnit.SECONDS);
        Map<String,List<Catelog2Vo>> dataFromDb = getDataFromDB();
        redisTemplate.delete("lock"); // 删除锁
        return dataFromDb;
    } else {
        // 加锁失败，重试 synchronized()
        // 休眠100ms重试
        return getCatelogJsonFromDbWithRedisLock();
    }
```

#### 1.2.1.4 分布式锁演进 - 阶段三

![image-20210427171355228](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427171355228.png)

```java
// 设置值同时设置过期时间  redis EX
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock","111",300,TimeUnit.SECONDS);
if (lock) {
    // 加锁成功..执行业务
    // 设置过期时间,必须和加锁是同步的，原子的
    redisTemplate.expire("lock",30,TimeUnit.SECONDS);
    Map<String,List<Catelog2Vo>> dataFromDb = getDataFromDB();
    redisTemplate.delete("lock"); // 删除锁
    return dataFromDb;
} else {
    // 加锁失败，重试 synchronized()
    // 休眠100ms重试
    return getCatelogJsonFromDbWithRedisLock();
}
```

#### 1.2.1.5 分布式锁演进 - 阶段四

![image-20210427171732570](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427171732570.png)

```java
String uuid = UUID.randomUUID().toString();
// 设置值同时设置过期时间
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock",uuid,300,TimeUnit.SECONDS);
if (lock) {
    // 加锁成功..执行业务
    // 设置过期时间,必须和加锁是同步的，原子的
    //            redisTemplate.expire("lock",30,TimeUnit.SECONDS);
    Map<String,List<Catelog2Vo>> dataFromDb = getDataFromDB();
    //            String lockValue = redisTemplate.opsForValue().get("lock");
    //            if (lockValue.equals(uuid)) {
    //                // 删除我自己的锁
    //                redisTemplate.delete("lock"); // 删除锁
    //            }
    // 通过使用lua脚本进行原子性删除
    String script = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
    //删除锁
    Long lock1 = redisTemplate.execute(new DefaultRedisScript<Long>(script, Long.class), Arrays.asList("lock"), uuid);

    return dataFromDb;
} else {
    // 加锁失败，重试 synchronized()
    // 休眠100ms重试
    return getCatelogJsonFromDbWithRedisLock();
}
```

#### 1.2.1.6 分布式锁演进 - 阶段五 最终模式

![image-20210427172602266](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427172602266.png)

```java
String uuid = UUID.randomUUID().toString();
// 设置值同时设置过期时间
Boolean lock = redisTemplate.opsForValue().setIfAbsent("lock",uuid,300,TimeUnit.SECONDS);
if (lock) {
    System.out.println("获取分布式锁成功");
    // 加锁成功..执行业务
    // 设置过期时间,必须和加锁是同步的，原子的
    //            redisTemplate.expire("lock",30,TimeUnit.SECONDS);
    Map<String,List<Catelog2Vo>> dataFromDb;
    //            String lockValue = redisTemplate.opsForValue().get("lock");
    //            if (lockValue.equals(uuid)) {
    //                // 删除我自己的锁
    //                redisTemplate.delete("lock"); // 删除锁
    //            }
    try {
        dataFromDb = getDataFromDB();
    } finally {
        String script = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
        //删除锁
        Long lock1 = redisTemplate.execute(new DefaultRedisScript<Long>(script, Long.class), Arrays.asList("lock"), uuid);
    }
    return dataFromDb;
} else {
    // 加锁失败，重试 synchronized()
    // 休眠200ms重试
    System.out.println("获取分布式锁失败，等待重试");
    try { TimeUnit.MILLISECONDS.sleep(200); } catch (InterruptedException e) { e.printStackTrace(); }
    return getCatelogJsonFromDbWithRedisLock();
}
```

### 1.2.2 分布式锁 - Redisson



https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95

#### 1.2.2.1、简介&整合

官网文档上详细说明了 不推荐使用 setnx来实现分布式锁，应该参考 the Redlock algorithm 的实现

https://redis.io/topics/distlock

https://github.com/redisson/redisson

```xml
<!--以后使用 redisson 作为分布锁，分布式对象等功能-->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.12.0</version>
</dependency>


<!--也可使用springboot的方式：https://github.com/redisson/redisson/tree/master/redisson-spring-boot-starter-->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.13.3</version>
</dependency>
```

```properties
# common spring boot settings

spring.redis.database=
spring.redis.host=
spring.redis.port=
spring.redis.password=
spring.redis.ssl=
spring.redis.timeout=
spring.redis.cluster.nodes=
spring.redis.sentinel.master=
spring.redis.sentinel.nodes=

# Redisson settings

#path to config - redisson.yaml
spring.redis.redisson.config=classpath:redisson.yaml
```

```java
@Configuration//配置连接 单机
public class MyRedissonConfig {
    @Bean(destroyMethod = "shutdown")
    public RedissonClient redisson() {
        Config config = new Config();
        // 集群模式
//        config.useClusterServers()
//                .addNodeAddress("127.0.0.1:7004", "127.0.0.1:7001");
        config.useSingleServer().setAddress("redis://127.0.0.1:6379");
        Redisson.create(config);
        return Redisson.create(config);
    }
}
```

#### 1.2.2.2 测试&Redisson-Lock看门狗原理-Redisson如何解决死锁

```java
@Autowired
RedissonClient redisson;

@RequestMapping("/hello")
@ResponseBody
public String hello(){
    // 1、获取一把锁，只要锁得名字一样，就是同一把锁
    RLock lock = redission.getLock("my-lock");

    // 2、加锁
    lock.lock(); // 阻塞式等待，默认加的锁都是30s时间
    // 1、锁的自动续期，如果业务超长，运行期间自动给锁续上新的30s，不用担心业务时间长，锁自动过期后被删掉
    // 2、加锁的业务只要运行完成，就不会给当前锁续期，即使不手动解锁，锁默认会在30s以后自动删除

    lock.lock(10, TimeUnit.SECONDS); //10s 后自动删除
    //问题 lock.lock(10, TimeUnit.SECONDS) 在锁时间到了后，不会自动续期
    // 1、如果我们传递了锁的超时时间，就发送给 redis 执行脚本，进行占锁，默认超时就是我们指定的时间
    // 2、如果我们为指定锁的超时时间，就是用 30 * 1000 LockWatchchdogTimeout看门狗的默认时间、
    //      只要占锁成功，就会启动一个定时任务，【重新给锁设置过期时间，新的过期时间就是看门狗的默认时间】,每隔10s就自动续期
    //      internalLockLeaseTime【看门狗时间】 /3,10s

    //最佳实践
    // 1、lock.lock(10, TimeUnit.SECONDS);省掉了整个续期操作，手动解锁

    try {
        System.out.println("加锁成功，执行业务..." + Thread.currentThread().getId());
        Thread.sleep(3000);
    } catch (Exception e) {

    } finally {
        // 解锁 将设解锁代码没有运行，reidsson会不会出现死锁
        System.out.println("释放锁...." + Thread.currentThread().getId());
        lock.unlock();
    }
    return "hello";
}
```

```java
lock.tryLock()//等待时间内获取锁
lock.getFairLock()//公平锁 有序取锁
```

进入到 `Redisson` Lock 源码

1、进入 `Lock` 的实现 发现 他调用的也是 `lock` 方法参数  时间为 -1

![image-20210427204138000](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204138000.png)

2、再次进入 `lock` 方法

发现他调用了 tryAcquire

![image-20210427204156621](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204156621.png)

3、进入 tryAcquire

![image-20210427204212298](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204212298.png)

4、里头调用了 tryAcquireAsync

这里判断 laseTime != -1 就与刚刚的第一步传入的值有关系

![image-20210427204243709](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204243709.png)

5、进入到 `tryLockInnerAsync` 方法

![image-20210427204300643](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204300643.png)

6、`internalLockLeaseTime` 这个变量是锁的默认时间

这个变量在构造的时候就赋初始值

![image-20210427204317552](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204317552.png)

7、最后查看 `lockWatchdogTimeout` 变量

也就是30秒的时间

![image-20210427204330633](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204330633.png)

#### 1.2.2.3 Reidsson - 读写锁

```java
/**
     * 保证一定能读取到最新数据，修改期间，写锁是一个排他锁（互斥锁，独享锁）读锁是一个共享锁
     * 写锁没释放读锁就必须等待
     * 读 + 读 相当于无锁，并发读，只会在 reids中记录好，所有当前的读锁，他们都会同时加锁成功
     * 写 + 读 等待写锁释放
     * 写 + 写 阻塞方式
     * 读 + 写 有读锁，写也需要等待
     * 只要有写的存在，都必须等待
     * @return String
     */
@RequestMapping("/write")
@ResponseBody
public String writeValue() {

    RReadWriteLock lock = redission.getReadWriteLock("rw_lock");
    String s = "";
    RLock rLock = lock.writeLock();
    try {
        // 1、改数据加写锁，读数据加读锁
        rLock.lock();
        System.out.println("写锁加锁成功..." + Thread.currentThread().getId());
        s = UUID.randomUUID().toString();
        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
        redisTemplate.opsForValue().set("writeValue",s);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        rLock.unlock();
        System.out.println("写锁释放..." + Thread.currentThread().getId());
    }
    return s;
}

@RequestMapping("/read")
@ResponseBody
public String readValue() {
    RReadWriteLock lock = redission.getReadWriteLock("rw_lock");
    RLock rLock = lock.readLock();
    String s = "";
    rLock.lock();
    try {
        System.out.println("读锁加锁成功..." + Thread.currentThread().getId());
        s = (String) redisTemplate.opsForValue().get("writeValue");
        try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        rLock.unlock();
        System.out.println("读锁释放..." + Thread.currentThread().getId());
    }
    return s;
}
```

![image-20210429093410321](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210429093410321.png)

#### 1.2.2.4 Redisson - 闭锁测试

```java
/**
 * 放假锁门
 * 1班没人了
 * 5个班级走完，我们可以锁们了
 * @return
 */
@GetMapping("/lockDoor")
@ResponseBody
public String lockDoor() throws InterruptedException {
    RCountDownLatch door = redission.getCountDownLatch("door");
    door.trySetCount(5);
    door.await();//等待闭锁都完成

    return "放假了....";
}
@GetMapping("/gogogo/{id}")
@ResponseBody
public String gogogo(@PathVariable("id") Long id) {
    RCountDownLatch door = redission.getCountDownLatch("door");
    door.countDown();// 计数器减一

    return id + "班的人走完了.....";
}
```

await()等待闭锁完成

countDown() 把计数器减掉后 await就会放行

#### 1.2.2.5 Redisson - 信号量测试

限流，所有服务上来了去获取一个信号量，一个一个放行

```java
/**
 * 车库停车
 * 3车位
 * @return
 */
@GetMapping("/park")
@ResponseBody
public String park() throws InterruptedException {
    RSemaphore park = redission.getSemaphore("park");
    boolean b = park.tryAcquire();//获取一个信号，获取一个值，占用一个车位

    return "ok=" + b;
}

@GetMapping("/go")
@ResponseBody
public String go() {
    RSemaphore park = redission.getSemaphore("park");

    park.release(); //释放一个车位

    return "ok";
}
```

类似 JUC 中的 Semaphore 

### 1.2.3 缓存数据一致性

就是缓存中的数据如何与数据库保持一致

#### **1.2.3.1 缓存数据一致性 - 双写模式**

![image-20210429094814099](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210429094814099.png)

两个线程写 最终只有一个线程写成功，后写成功的会把之前写的数据给覆盖，这就会造成脏数据

#### 1.2.3.2 缓存数据一致性 - 失效模式

![image-20210429094909045](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210429094909045.png)

三个连接 

一号连接 写数据库 然后删缓存

二号连接 写数据库时网络连接慢，还没有写入成功

三号链接 直接读取数据，读到的是一号连接写入的数据，此时 二号链接写入数据成功并删除了缓存，三号开始更新缓存发现更新的是二号的缓存

#### 1.2.3.3 缓存数据一致性解决方案

无论是双写模式还是失效模式，都会导致缓存不一致的问题，即多个实例同时更新会出事，怎么办？

- 1、如果是用户维度数据（订单数据、用户数据），这并发几率很小，几乎不用考虑这个问题，缓存数据加上过期时间，每隔一段时间触发读的主动更新即可
- 2、如果是菜单，商品介绍等基础数据，也可以去使用 canal 订阅，binlog 的方式
- 3、缓存数据 + 过期时间也足够解决大部分业务对缓存的要求
- 4、通过加锁保证并发读写，写写的时候按照顺序排好队，读读无所谓，所以适合读写锁，（业务不关心脏数据，允许临时脏数据可忽略）

总结:

- 我们能放入缓存的数据本来就不应该是实时性、一致性要求超高的。所以缓存数据的时候加上过期时间，保证每天拿到当前的最新值即可
- 我们不应该过度设计，增加系统的复杂性
- 遇到实时性、一致性要求高的数据，就应该查数据库，即使慢点



#### **缓存数据一致性-Canal**

```
原理：操作数据库，canal获得binlog二进制日志，根据日志去修改缓存，不用再调用修改缓存的代码

好处：减少了代码量

弊端：增加了一个中间件

作用：canal分析 各种表的binlog，生成一个用户推荐表，所以每个人的首页都是不一样的。
```

![image-20210429100311802](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210429100311802.png)

### 1.2.4.5 Spring Cache

https://docs.spring.io/spring-framework/docs/5.2.10.RELEASE/spring-framework-reference/integration.html#cache-strategie

#### 1.2.4.1 简介

- Spring 从3.1开始定义了 `org.springframework.cache.Cache` 和 `org.sprngframework.cache.CacheManager` 接口睐统一不同的缓存技术
- 并支持使用 `JCache`（JSR-107）注解简化我们的开发
- Cache 接口为缓存的组件规范定义，包含缓存的各种操作集合 `Cache` 接口下 Spring 提供了各种 XXXCache的实现，如 `RedisCache`、`EhCache`,`ConcrrentMapCache`等等，
- 每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过;如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
- 使用 `Sprng` 缓存抽象时我们需要关注的点有以下两点
  - 1、确定方法需要被缓存以及他们的的缓存策略
  - 2、从缓存中读取之前缓存存储的数据

#### 1.2.4.2 基础概念

从3.1版本开始，Spring框架就支持透明地向现有Spring应用程序添加缓存。与事务支持类似，缓存抽象允许在对代码影响最小的情况下一致地使用各种缓存解决方案。从Spring 4.1开始，缓存抽象在JSR-107注释和更多定制选项的支持下得到了显著扩展。

```java
 /**
 *  8、整合SpringCache简化缓存开发
 *      1、引入依赖
 *          spring-boot-starter-cache
 *      2、写配置
 *          1、自动配置了那些
 *              CacheAutoConfiguration会导入 RedisCacheConfiguration
 *              自动配置好了缓存管理器，RedisCacheManager
 *          2、配置使用redis作为缓存
 *          Spring.cache.type=redis
 *		3, 测试
 *			主启动 @EnableCaching/配置类
 *
 *       4、原理
 *       CacheAutoConfiguration ->RedisCacheConfiguration ->
 *       自动配置了 RedisCacheManager ->初始化所有的缓存 -> 每个缓存决定使用什么配置
 *       ->如果redisCacheConfiguration有就用已有的，没有就用默认的
 *       ->想改缓存的配置，只需要把容器中放一个 RedisCacheConfiguration 即可
 *       ->就会应用到当前 RedisCacheManager管理所有缓存分区中
 */
```

#### 1.2.4.3 注解

```java
@Cacheable: 触发将数据保存到缓存的操作，如果缓存中有，方法不调用
@CacheEvict:  触发将数据从缓存删除的操作    失效模式
@CachePut: 不影响方法执行更新缓存    双写模式
@Caching: 组合以上多个操作   【实现双写+失效模式】
@CacheConfig: 在类级别共享缓存的相同配置
```

#### 1.2.4.4 配置

```java
package com.atguigu.gulimall.product.config;

import org.springframework.boot.autoconfigure.cache.CacheProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@EnableConfigurationProperties(CacheProperties.class)
@EnableCaching
@Configuration
public class MyCacheConfig {

    /**
     * 配置文件中的东西没有用上
     * 1、原来的配置文件绑定的配置类是这样子的
     *      @ConfigurationProperties(prefix = "Spring.cache")
     * 2、要让他生效
     *      @EnableConfigurationProperties(CacheProperties.class)
     * @param cacheProperties
     * @return
     */
    @Bean
    RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        // 设置key的序列化
        config = config.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
        // 设置value序列化 ->JackSon
        config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        CacheProperties.Redis redisProperties = cacheProperties.getRedis();
        if (redisProperties.getTimeToLive() != null) {
            config = config.entryTtl(redisProperties.getTimeToLive());
        }
        if (redisProperties.getKeyPrefix() != null) {
            config = config.prefixKeysWith(redisProperties.getKeyPrefix());
        }
        if (!redisProperties.isCacheNullValues()) {
            config = config.disableCachingNullValues();
        }
        if (!redisProperties.isUseKeyPrefix()) {
            config = config.disableKeyPrefix();
        }
        return config;
    }

}
```

```yml
Spring:
  cache:
    type: redis
    redis:
      time-to-live: 3600000           # 过期时间
      key-prefix: CACHE_              # key前缀
      use-key-prefix: true            # 是否使用写入redis前缀
      cache-null-values: true         # 是否允许缓存空值   防止缓存穿透
```

#### 1.2.4.5 demo

缓存保存

```java
/**
     * 1、每一个需要缓存的数据我们都需要指定放到那个名字的缓存【缓存分区的划分【按照业务类型划分】】
     * 2、@Cacheable({"category"})
     *      代表当前方法的结果需要缓存，如果缓存中有，方法不调用
     *      如果缓存中没有，调用方法，最后将方法的结果放入缓存
     * 3、默认行为:
     *      1、如果缓存中有，方法不用调用
     *      2、key默自动生成，缓存的名字:SimpleKey[](自动生成的key值)
     *      3、缓存中value的值，默认使用jdk序列化，将序列化后的数据存到redis
     *      3、默认的过期时间，-1
     *
     *    自定义操作
     *      1、指定缓存使用的key     key属性指定，接收一个SpEl
     	SpEL表达式可以参照官网 Avaliable Caching SpEL Evaluation Context
     *      2、指定缓存数据的存活时间  配置文件中修改ttl
     * 			  cache:
     *  			redis:
     * 				  time-to-live: 
     *      3、将数据保存为json格式
     * @return
     */
    @Cacheable(value = {"category"},key = "#root.method.name")
    @Override
    public List<CategoryEntity> getLevel1Categorys() {
        long l = System.currentTimeMillis();
        // parent_cid为0则是一级目录
        List<CategoryEntity> categoryEntities = baseMapper.selectList(new QueryWrapper<CategoryEntity>().eq("parent_cid", 0));
        System.out.println("耗费时间：" + (System.currentTimeMillis() - l));
        return categoryEntities;
    }
```

缓存删除

```java
/**
     * 级联更新所有的关联数据
     * @CacheEvict 失效模式
     * 1、同时进行多种缓存操作 @Caching
     * 2、指定删除某个分区下的所有数据 @CacheEvict(value = {"category"},allEntries = true)
     * 3、存储同一类型的数据，都可以指定成同一分区，分区名默认就是缓存的前缀
     *
     * @param category
     */
    @Caching(evict = {
            @CacheEvict(value = {"category"},key = "'getLevel1Categorys'"),
            @CacheEvict(value = {"category"},key = "'getCatelogJson'")
    })
//    @CacheEvict(value = {"category"},allEntries = true)
    @Transactional
    @Override
    public void updateCascate(CategoryEntity category) {
        // 更新自己表对象
        this.updateById(category);
        // 更新关联表对象
        categoryBrandRelationService.updateCategory(category.getCatId(), category.getName());
    }
```

#### 1.2.3.6 Spring-Cache不足

1、读模式：

 * 缓存穿透:查询一个null数据，解决 缓存空数据：ache-null-values=true/【布隆过滤器】

 * 缓存击穿:大量并发进来同时查询一个正好过期的数据，解决:加锁 ？ 默认是无加锁 

   ```java
   @Cacheable(value = {"category"},key = "#root.method.name",sync = true) //本地锁
   ```

 * 缓存雪崩:大量的key同时过期，解决：加上随机时间，Spring-cache-redis-time-to-live

2、写模式：（缓存与数据库库不一致）

 * 1、读写加锁
 * 2、引入canal，感知到MySQL的更新去更新数据库
 * 3、读多写多，直接去数据库查询就行

总结：
 * 常规数据（读多写少，即时性，一致性要求不高的数据）完全可以使用SpringCache 写模式（ 只要缓存数据有过期时间就足够了）
   
 * 特殊数据：特殊设计

原理：
 *          CacheManager(RedisManager) -> Cache(RedisCache) ->Cache负责缓存的读写





















#  多线程&异步 

## 1.1 CompletableFuture 异步编排

### 1.1.1 创建异步对象

CompletableFuture 提供了四个静态方法来创建一个异步操作

```java
CompletableFuture.runAsync(Runnable runnable);
CompletableFuture.runAsync(Runnable runnable, Executor executor);
CompletableFuture.supplyAsync(Supplier<U> supplier);
CompletableFuture.supplyAsync(Supplier<U> supplier, Executor executor);
```

1. runXxx 都是没有返回结果的，supplyXxxx都是可以获取返回结果的

2. 可以传入自定义的线程池，否则就是用默认的线程池

3. 根据方法的返回类型来判断是否该方法是否有返回类型

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    System.out.println("main....start.....");
    CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
        System.out.println("当前线程：" + Thread.currentThread().getId());
        int i = 10 / 2;
        System.out.println("运行结果：" + i);
    }, executor);

    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
        System.out.println("当前线程：" + Thread.currentThread().getId());
        int i = 10 / 2;
        System.out.println("运行结果：" + i);
        return i;
    }, executor);
    Integer integer = future.get();

    System.out.println("main....stop....." + integer);
}
```

### 1.1.2 计算完成时回调方法

```java
whenCompleteAsync()  
whenComplete()  
exceptionally() 
```

whenComplete 可以处理正常和异常的计算结果，exceptionally 处理异常情况

whenComplete 和 whenCompleteAsync 的区别

​		whenComplete ：是执行当前任务的线程继续执行 whencomplete 的任务

​		whenCompleteAsync： 是执行把 whenCompleteAsync 这个任务继续提交给线程池来进行执行

**方法不以 Async 结尾，意味着 Action 使用相同的线程执行，而 Async 可能会使用其他线程执行（如何是使用相同的线程池，也可能会被同一个线程选中执行）**

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    System.out.println("当前线程：" + Thread.currentThread().getId());
    int i = 10 / 0;
    System.out.println("运行结果：" + i);
    return i;
}, executor).whenComplete((res,exception) ->{
    // 虽然能得到异常信息，但是没法修改返回的数据
    System.out.println("异步任务成功完成了...结果是：" +res + "异常是：" + exception);
}).exceptionally(throwable -> {
    // 可以感知到异常，同时返回默认值
    return 10;
});
```

### 1.13 handle 方法

```java
handleAsync()
handle()
```

和 complete 一样，可以对结果做最后的处理（可处理异常），可改变返回值

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    System.out.println("当前线程：" + Thread.currentThread().getId());
    int i = 10 / 2;
    System.out.println("运行结果：" + i);
    return i;
}, executor).handle((res,thr) ->{
    if (res != null ) {
        return res * 2;
    }
    if (thr != null) {
        return 0;
    }
    return 0;
});
```

### 1.1.4 线程串行化方法

```java
thenRunAsync()
thenRun()

thenAccept()
thenAcceptAsync()
    
thenApply()
thenApplyAsync()
```

thenApply 方法：当一个线程依赖另一个线程时，获取上一个任务返回的结果，并返回当前任务的返回值

thenAccept方法：消费处理结果，接受任务处理结果，并消费处理，无返回结果

thenRun 方法：只要上面任务执行完成，就开始执行 thenRun ,只是处理完任务后，执行 thenRun的后续操作

带有 Async 默认是异步执行的，同之前，

以上都要前置任务完成

```java
/**
* 线程串行化，
* 1、thenRun:不能获取到上一步的执行结果，无返回值
* .thenRunAsync(() ->{
*             System.out.println("任务2启动了....");
*         },executor);
* 2、能接受上一步结果，但是无返回值 thenAcceptAsync
* 3、thenApplyAsync 能收受上一步结果，有返回值
*
*/
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    System.out.println("当前线程：" + Thread.currentThread().getId());
    int i = 10 / 2;
    System.out.println("运行结果：" + i);
    return i;
}, executor).thenApplyAsync(res -> {
    System.out.println("任务2启动了..." + res);
    return "Hello " + res;
}, executor);
String s = future.get();

System.out.println("main....stop....." + s);
```

### 1.1.5 两任务组合 - 都要完成

```java
thenCombine()
thenCombineAsync()

thenAcceptBoth()
thenAcceptBothAsync()

runAfterBoth()
runAfterBothAsync()
```

两个任务必须都完成，触发该任务

thenCombine: 组合两个 future，获取两个 future的返回结果，并返回当前任务的返回值

thenAccpetBoth: 组合两个 future，获取两个 future 任务的返回结果，然后处理任务，没有返回值

runAfterBoth:组合 两个 future，不需要获取 future 的结果，只需要两个 future处理完成任务后，处理该任务，

```java
/**
         * 两个都完成
         */
CompletableFuture<Integer> future01 = CompletableFuture.supplyAsync(() -> {
    System.out.println("任务1当前线程：" + Thread.currentThread().getId());
    int i = 10 / 4;
    System.out.println("任务1结束：" + i);
    return i;
}, executor);

CompletableFuture<String> future02 = CompletableFuture.supplyAsync(() -> {
    System.out.println("任务2当前线程：" + Thread.currentThread().getId());
    System.out.println("任务2结束：");
    return "Hello";
}, executor);

// f1 和 f2 执行完成后在执行这个
//        future01.runAfterBothAsync(future02,() -> {
//            System.out.println("任务3开始");
//        },executor);

// 返回f1 和 f2 的运行结果
//        future01.thenAcceptBothAsync(future02,(f1,f2) -> {
//            System.out.println("任务3开始....之前的结果:" + f1 + "==>" + f2);
//        },executor);

// f1 和 f2 单独定义返回结果
CompletableFuture<String> future = future01.thenCombineAsync(future02, (f1, f2) -> {
    return f1 + ":" + f2 + "-> Haha";
}, executor);

System.out.println("main....end....." + future.get());
```

### 1.1.6 两任务组合 - 一个完成

```java
applyToEither
applyToEitherAsync

acceptEither
acceptEitherAsync

runAfterEither
runAfterEitherAsync
```

当两个任务中，任意一个future 任务完成时，执行任务

**applyToEither**;两个任务有一个执行完成，获取它的返回值，处理任务并有新的返回值

**acceptEither**: 两个任务有一个执行完成，获取它的返回值，处理任务，没有新的返回值

**runAfterEither**:两个任务有一个执行完成，不需要获取 future 的结果，处理任务，也没有返回值

```java
/**
         * 两个任务，只要有一个完成，我们就执行任务
         * runAfterEnitherAsync：不感知结果，自己没有返回值
         * acceptEitherAsync：感知结果，自己没有返回值
         *  applyToEitherAsync：感知结果，自己有返回值
         */
//        future01.runAfterEitherAsync(future02,() ->{
//            System.out.println("任务3开始...之前的结果:");
//        },executor);

//        future01.acceptEitherAsync(future02,(res) -> {
//            System.out.println("任务3开始...之前的结果:" + res);
//        },executor);

CompletableFuture<String> future = future01.applyToEitherAsync(future02, res -> {
    System.out.println("任务3开始...之前的结果：" + res);
    return res.toString() + "->哈哈";
}, executor);
```

### 1.1.7 多任务组合

```java
allOf
anyOf
```

allOf：等待所有任务完成

anyOf:只要有一个任务完成

```java
CompletableFuture<String> futureImg = CompletableFuture.supplyAsync(() -> {
    System.out.println("查询商品的图片信息");
    return "hello.jpg";
});

CompletableFuture<String> futureAttr = CompletableFuture.supplyAsync(() -> {
    System.out.println("查询商品的属性");
    return "黑色+256G";
});

CompletableFuture<String> futureDesc = CompletableFuture.supplyAsync(() -> {
    try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
    System.out.println("查询商品介绍");
    return "华为";
});

// 等待全部执行完
//        CompletableFuture<Void> allOf = CompletableFuture.allOf(futureImg, futureAttr, futureDesc);
//        allOf.get();

// 只需要有一个执行完
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(futureImg, futureAttr, futureDesc);
anyOf.get();
System.out.println("main....end....." + anyOf.get())
```























# Session

![image-20210607171605279](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210607171605279.png)

1. 不能跨不同域名共享

2. 不同服务不能共享

3. 同服务多份不能共享

## 解决方案1-session复制

![image-20210607172008759](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210607172008759.png)

## 解决方案2-客户端存储

![image-20210607172231696](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210607172231696.png)

## 解决方案3-hash一致性

![image-20210607172255088](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210607172255088.png)



## 解决方案4-统一存储

![image-20210607172330969](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210607172330969.png)

## SpringSession

```xml
<!-- 整合Spring-seesion  -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

```yaml
spring: 
  session:
    store-type: redis
# session存活时间
server:
  servlet:
    session:
      timeout: 30m
      

#配置redis链接地址，端口，密码等等。。。。。
spring.redis.host=localhost # Redis server host.
spring.redis.password= # Login password of the redis server.
spring.redis .port=6379 # Redis server port.
```

主启动@EnableRedisHttpSession

```java
@Configuration
public class GlMallSessionConfig {

	@Bean
	public CookieSerializer cookieSerializer(){
		DefaultCookieSerializer cookieSerializer = new DefaultCookieSerializer();
		// 明确的指定Cookie的作用域
		cookieSerializer.setDomainName("glmall.com");
		cookieSerializer.setCookieName("FIRESESSION");
		return cookieSerializer;
	}

	/**
	 * 自定义序列化机制
	 * 这里方法名必须是：springSessionDefaultRedisSerializer
	 */
	@Bean
	public RedisSerializer<Object> springSessionDefaultRedisSerializer(){
		return new GenericJackson2JsonRedisSerializer();
	}
}
```





**核心原理**

@EnableRedisHttpSession导入RedisHttpSessionConfiguration配置

1. 给容器中添加了一个组件

   ​     sessionRepository = 》RedisOperationsSessionRepository redis 操作 session.  session的增删改查封装类

2. SessionRepositoryFilter==>:session存储过滤器，每个请求过来必须经过Filter
   
   1. 创建的时候，就自动从容器中获取到了SessionRepostiory

 * 2、原始的request,response都被包装了 SessionRepositoryRequestWrapper、SessionRepositoryResponseWrapper

 * 3、以后获取session.request.getSession()

   SessionRepositoryResponseWrapper

 * 4、wrappedRequest.getSession() ==>SessionRepository

   ==**装饰者模式**==

 * spring-redis的相关功能: 自动延期

    	执行session相关操作后，redis里面存储的时间也会刷新



核心源码是：

- `SessionRepositoryFilter` 类下面的 `doFilterInternal` 方法

- 及那个 `request`、`response` 包装成 `SessionRepositoryRequestWrapper`

![image-20210608160450184](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210608160450184.png)



# 认证服务&单点登录&社交登录

## 认证服务



```java
@Configuration
public class GulimallWebConfig implements WebMvcConfigurer {

    /**
     * 视图映射:发送一个请求，直接跳转到一个页面
     */
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {

        // registry.addViewController("/login.html").setViewName("login");
        registry.addViewController("/reg.html").setViewName("reg");
    }
}
```



### 前端验证码倒计时

```js
$(function () {
    /**
     * 验证码发送
     */
    $("#sendCode").click(function () {
        //判断是否有该样式
        if ($(this).hasClass("disabled")) {
            // 正在倒计时
        } else {
            // 发送验证码
            $.get("/sms/sendCode?phone=" + $("#phoneNum").val(), function (data) {
                if (data.code != 0) {
                    alert(data.msg)
                }
            })
            timeoutChangeStyle();
        }
    })
})
// 60秒
var num = 60;
function timeoutChangeStyle() {
    // 先添加样式，防止重复点击
    $("#sendCode").attr("class", "disabled")
    // 到达0秒后 重置时间，去除样式
    if (num == 0) {
        $("#sendCode").text("发送验证码")
        num = 60;
        // 时间到达后清除样式
        $("#sendCode").attr("class", "");
    } else {
        var str = num + "s 后再次发送"
        $("#sendCode").text(str);
        setTimeout("timeoutChangeStyle()", 1000);
    }
    num--;
}
```

### 整合短信验证码

阿里云的短信服务进行开通验证

appCode 

使用 Java 测试短信是否能进行发送

​	服务商提供的**接口地址**，**请求参数**都不同，请参考服务商提供的测试代码

```java
@Test
public void contextLoads() {
   String host = "http://dingxin.market.alicloudapi.com";
	    String path = "/dx/sendSms";
	    String method = "POST";
	    String appcode = "你自己的AppCode";
	    Map<String, String> headers = new HashMap<String, String>();
	    //最后在header中的格式(中间是英文空格)为Authorization:APPCODE 83359fd73fe94948385f570e3c139105
	    headers.put("Authorization", "APPCODE " + appcode);
	    Map<String, String> querys = new HashMap<String, String>();
	    querys.put("mobile", "159xxxx9999");
	    querys.put("param", "code:1234");
	    querys.put("tpl_id", "TP1711063");
	    Map<String, String> bodys = new HashMap<String, String>();


	    try {
	    	/**
	    	* 重要提示如下:
	    	* HttpUtils请从
	    	* https://github.com/aliyun/api-gateway-demo-sign-java/blob/master/src/main/java/com/aliyun/api/gateway/demo/util/HttpUtils.java
	    	* 下载
	    	*
	    	* 相应的依赖请参照
	    	* https://github.com/aliyun/api-gateway-demo-sign-java/blob/master/pom.xml
	    	*/
	    	HttpResponse response = HttpUtils.doPost(host, path, method, headers, querys, bodys);
	    	System.out.println(response.toString());
	    	//获取response的body
	    	//System.out.println(EntityUtils.toString(response.getEntity()));
	    } catch (Exception e) {
	    	e.printStackTrace();
	    }
}
```

需要导入对应工具类，参照注释就行

### 验证码防刷校验

用户要是一直提交验证码

- 前台：限制一分钟后提交

- 后台：存入redis 如果有就返回

```java
/**
 * 发送短信验证码
 * @param phone 手机号
 * @return
 */
@GetMapping("/sms/sendCode")
@ResponseBody
public R sendCode(@RequestParam("phone") String phone) {
    // TODO 1、接口防刷
    // 先从redis中拿取
    String redisCode = redisTemplate.opsForValue().get(AuthServerConstant.SMS_CODE_CACHE_PREFIX + phone);
    if(!StringUtils.isEmpty(redisCode)) {
        // 拆分
        long l = Long.parseLong(redisCode.split("_")[1]);
        // 当前系统事件减去之前验证码存入的事件 小于60000毫秒=60秒
        if (System.currentTimeMillis() -l < 60000) {
            // 60秒内不能再发
            R.error(BizCodeEnume.SMS_CODE_EXCEPTION.getCode(),BizCodeEnume.SMS_CODE_EXCEPTION.getMsg());
        }
    }
    // 2、验证码的再次效验
    // 数据存入 =》redis key-phone value - code sms:code:131xxxxx - >45678
    String code = UUID.randomUUID().toString().substring(0,5).toUpperCase();
    // 拼接验证码
    String substring = code+"_" + System.currentTimeMillis();
    // redis缓存验证码 防止同一个phone在60秒内发出多次验证吗
    redisTemplate.opsForValue().set(AuthServerConstant.SMS_CODE_CACHE_PREFIX+phone,substring,10, TimeUnit.MINUTES);

    // 调用第三方服务发送验证码
    thirdPartFeignService.sendCode(phone,code);
    return R.ok();
}
```

### 注册

####  vo

- 使用到了 JSR303校验

```java
/**
 * 注册数据封装Vo
 * @author gcq
 * @Create 2020-11-09
 */
@Data
public class UserRegistVo {
    @NotEmpty(message = "用户名必须提交")
    @Length(min = 6,max = 18,message = "用户名必须是6-18位字符")
    private String userName;

    @NotEmpty(message = "密码必须填写")
    @Length(min = 6,max = 18,message = "密码必须是6-18位字符")
    private String password;

    @NotEmpty(message = "手机号码必须提交")
    @Pattern(regexp = "^[1]([3-9])[0-9]{9}$",message = "手机格式不正确")
    private String phone;

    @NotEmpty(message = "验证码必须填写")
    private String code;
}
```

#### 数据校验

```java
/**
 * //TODO 重定向携带数据，利用session原理，将数据放在session中，
 * 只要跳转到下一个页面取出这个数据，session中的数据就会删掉
 * //TODO分布式下 session 的问题
 * RedirectAttributes redirectAttributes 重定向携带数据
 * redirectAttributes.addFlashAttribute("errors", errors); 只能取一次
 * @param vo 数据传输对象
 * @param result 用于验证参数
 * @param redirectAttributes 数据重定向
 * @return
 */
@PostMapping("/regist")
public String regist(@Valid UserRegistVo vo, BindingResult result,
                     RedirectAttributes redirectAttributes) {
    // 校验是否通过
    if (result.hasErrors()) {
        // 拿到错误信息转换成Map
        Map<String, String> errors = result.getFieldErrors().stream().collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage));
        //用一次的属性
        redirectAttributes.addFlashAttribute("errors",errors);
        // 校验出错，转发到注册页
        return "redirect:http://auth.gulimall.com/reg.html";
    }

    // 将传递过来的验证码 与 存redis中的验证码进行比较
    String code = vo.getCode();
    String s = redisTemplate.opsForValue().get(AuthServerConstant.SMS_CODE_CACHE_PREFIX + vo.getPhone());
    if (!StringUtils.isEmpty(s)) {
        // 验证码和redis中的一致
        if(code.equals(s.split("_")[0])) {
            // 删除验证码：令牌机制
            redisTemplate.delete(AuthServerConstant.SMS_CODE_CACHE_PREFIX + vo.getPhone());
            // 调用远程服务，真正注册
            R r = memberFeignService.regist(vo);
            if (r.getCode() == 0) {
                // 远程调用注册服务成功
                return "redirect:http://auth.gulimall.com/login.html";
            } else {
                Map<String, String> errors = new HashMap<>();
                errors.put("msg",r.getData(new TypeReference<String>(){}));
                redirectAttributes.addFlashAttribute("errors", errors);
                return "redirect:http://auth.gulimall.com/reg.html";
            }
        } else {
            Map<String, String> errors = new HashMap<>();
            errors.put("code", "验证码错误");
            redirectAttributes.addFlashAttribute("code", "验证码错误");
            // 校验出错，转发到注册页
            return "redirect:http://auth.gulimall.com/reg.html";
        }
    } else {
        Map<String, String> errors = new HashMap<>();
        errors.put("code", "验证码错误");
        redirectAttributes.addFlashAttribute("code", "验证码错误");
        // 校验出错，转发到注册页
        return "redirect:http://auth.gulimall.com/reg.html";
    }
}
```

#### 异常机制

- 用户注册单独抽出了一个服务

Controller

```java
/**
 * 注册
 * @param registVo
 * @return
 */
@PostMapping("/regist")
public R regist(@RequestBody MemberRegistVo registVo) {
    try {
        memberService.regist(registVo);
    } catch (PhoneExsitException e) {
        // 返回对应的异常信息
       return R.error(BizCodeEnume.PHONE_EXIST_EXCEPTION.getCode(),BizCodeEnume.PHONE_EXIST_EXCEPTION.getMsg());
    } catch (UserNameExistException e) {
        return R.error(BizCodeEnume.USER_EXIST_EXCEPTION.getCode(),BizCodeEnume.USER_EXIST_EXCEPTION.getMsg());
    }
    return R.ok();
}
```

```java
@Override
public void regist(MemberRegistVo registVo) {
    MemberDao memberDao = this.baseMapper;
    MemberEntity entity = new MemberEntity();

    // 设置默认等级
    MemberLevelEntity memberLevelEntity = memberLevelDao.getDefaultLevel();
    entity.setLevelId(memberLevelEntity.getId());


    // 检查手机号和用户名是否唯一
    checkPhoneUnique(registVo.getPhone());
    checkUserNameUnique(registVo.getUserName());

    entity.setMobile(registVo.getPhone());
    entity.setUsername(registVo.getUserName());

    //密码要加密存储
    BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    String encode = passwordEncoder.encode(registVo.getPassword());
    entity.setPassword(encode);

    memberDao.insert(entity);
}

@Override
public void checkPhoneUnique(String phone) throws PhoneExsitException {
    MemberDao memberDao = this.baseMapper;
    Integer mobile = memberDao.selectCount(new QueryWrapper<MemberEntity>().eq("mobile", phone));
    if (mobile > 0) {
        throw new PhoneExsitException();
    }
}

@Override
public void checkUserNameUnique(String username) throws UserNameExistException {
    MemberDao memberDao = this.baseMapper;
    Integer count = memberDao.selectCount(new QueryWrapper<MemberEntity>().eq("username", username));
    if (count > 0) {
        throw new PhoneExsitException();
    }
}
```

### 密码加密存储

MD5：信息摘要算法，只要一个字节发生变化，结果值就会变化

MD5&MD5盐值加密

1. MD5

   Message Digest algorithm 5，信息摘要算法

   * 压缩性:任意长度的数据，算出的MD5值长度都是固定的。

   * 容易计算:从原数据计算出MD5值很容易。

   * 抗修改性:对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。

   * 强抗碰撞:想找到两个不同的数据，使它们具有相同的MD5值，是非常困难的。百度网盘秒传功能

   * 不可逆

2. 加盐:【明文相同，盐值不同密文也不同，增加了彩虹表的难度】
   * 通过生成随机数与MD5生成字符串进行组合
   * 数据库同时存储MD5值与salt值。验证正确性时使用salt进行MD5即可 

```java
import org.apache.commons.codec.digest.DigestUtils;
@Test
public void contextLoads() {
    // MD5不能直接进行密码的加密存储
    String s = DigestUtils.md5Hex("123456 ");
    System.out.println(s);
    // 盐值加密
    Md5Crypt.md5Crypt("123456".getBytes(), "$1$qqq");
}

//Spring提供了一个类【不需要在数据库中存储盐值，盐值直接拼接在了密文中。】
public class Test {
    public static void main(String[] args) {
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        String encode = bCryptPasswordEncoder.encode("123456");
        String encode2 = bCryptPasswordEncoder.encode("123456");
        System.out.println(encode);
        System.out.println(encode2);

        System.out.println(bCryptPasswordEncoder.matches("123456", "$2a$10$IOF07aBAbdNQsZnAWxVJzOxSAHexp/pUNhQC7MwDwOG7mpZaRdrHC"));
        System.out.println(bCryptPasswordEncoder.matches("123456", "$2a$10$gVqMI4WFNXIwmURQW50V.O0ZyNKVKkljhLTxl/D1Re.daF9XlohzG"));
    }
}
```

验证密码【明文与密文直接匹配】
		boolean matche = bCryptPasswordEncoder.matches(vo.getPassword(), "密文xx");
原理：

1. 相同的明文123456，生成的密文不一样【1、盐值不同导致密文不同】
2. 盐值被拼在了密文中，所以先根据密文解析出盐，然后再 明文+盐 加密与密文匹配









## 单点登录

【社交登录、SpringSession+扩大子域   只是单系统分布式集群的登录】

多系统-单点登录

​	1、一处登录处处登录

​	2、一处退出处处退出

### 什么是SSO

单点登录(SingleSignOn，SSO)，就是通过用户的一次性鉴别登录。当用户在身份认证服务器上登录一次以后，即可获得访问单点登录系统中其他关联系统和应用软件的权限，同时这种实现是不需要管理员对用户的登录状态或其他信息进行修改的，这意味着在多个应用系统中，用户只需一次登录就可以访问所有相互信任的应用系统。这种方式减少了由登录产生的时间消耗，辅助了用户管理，是目前比较流行的

### 单点登录框架 & 原理演示

 https://gitee.com/xuxueli0323/xxl-sso

XXL-SSO 是一个分布式单点登录框架。只需要登录一次就可以访问所有相互信任的应用系统。 拥有"轻量级、分布式、跨域、Cookie+Token均支持、Web+APP均支持"等特性。现已开放源代码，开箱即用。

修改配置文件 ->进行：mvn clean package -Dmaven.skip.test=true  

xxl-sso-server:

- 8080/xxl-sso-server

- windows host文件添加：
  - ssoserver.com 登陆验证服务器
  - client1.com 客户端1  java -jar --server.port=8081
  - client2.com 客户端2 java -jar --server.port=8082

先启动xxl-sso-server 然后启动client1

只要 `client1` 登录成功 `client2` 就不用进行登录直接登录成功



1. 中央认证服务器; ssoserver. com
2. 其他系统，想要登录去ssoserver.com登录，登录成功跳转回来
3. 只要有一个登录，其他都不用登录
4. 全系统统一一个sso-sessionid



server登录返回cookie

client访问带上cookie

### 代码测试

#### sso-client

```java
/**
 * @author gcq
 * @Create 2020-11-12
 */
@Controller
public class HelloController {

    @Value("${sso.server.url}")
    private String ssoServerUrl;

    /**
     * 无需登录就可以访问
     * @return
     */
    @ResponseBody
    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }

    /**
     * 需要验证的连接
     * @param model
     * @param token 只要是ssoserver登陆成功回来就会带上
     * @return
     */
    @GetMapping("/employees")
    public String employees(Model model, HttpSession session,
                            @RequestParam(value="token",required = false) String token) {
        if (!StringUtils.isEmpty(token)) {
            // 去ssoserver登录成功调回来就会带上
            //TODO 1、去ssoserver获取当前token真正对应的用户信息
            RestTemplate restTemplate = new RestTemplate();
            // 使用restTemplate进行远程请求
            ResponseEntity<String> forEntity = restTemplate.getForEntity("http://ssoserver.com:8080/userInfo?token=" + token, String.class);
            // 拿到数据
            String body = forEntity.getBody();
            // 设置到session中
            session.setAttribute("loginUser",body);
        }
        Object loginUser = session.getAttribute("loginUser");
        if (loginUser == null ){
            // 没有登录重定向到登陆页面，并带上当前地址
            return "redirect:" + ssoServerUrl + "?redirect_url=http://client1.com:8081/employees";
        } else {
            List<String> emps = new ArrayList<>();
            emps.add("张三");
            emps.add("李四");
            model.addAttribute("emps",emps);
            return "list";
        }

    }
}
```

#### sso-server

```java
/**
 * @author gcq
 * @Create 2020-11-12
 */
@Controller
public class LoginController {

    @Autowired
    StringRedisTemplate redisTemplate;

    /**
     * 根据token从redis中查询用户信息
     * @param token
     * @return
     */
    @ResponseBody
    @GetMapping("/userInfo")
    public String userInfo(@RequestParam("token") String token) {
        String s = redisTemplate.opsForValue().get(token);
        return s;
    }

    @GetMapping("login.html")
    public String login(@RequestParam("redirect_url") String url, Model model,
                        @CookieValue(value = "sso_token",required = false)String sso_token) {
        if (!StringUtils.isEmpty(sso_token)) {
            //说明有人之前登录过，给浏览器留下了痕迹
            return "redirect:" + url + "?token=" + sso_token;
        }
        // 添加url到model地址中，在前端页面进行取出
        model.addAttribute("url",url);
        return "login";
    }

    /**
     * 登录
     * @param username
     * @param password
     * @param url client端带过来的地址
     * @return
     */
    @PostMapping("/doLogin")
    public String doLogin(@RequestParam("username") String username,
                          @RequestParam("password") String password,
                          @RequestParam("url") String url,
                          HttpServletResponse response){
        // 账号密码不为空 
        if (!StringUtils.isEmpty(username) && !StringUtils.isEmpty(password)) {
            // 登陆成功
            // 把登录成功的用户存起来
            String uuid = UUID.randomUUID().toString().replace("-","");
            redisTemplate.opsForValue().set(uuid,username);
            // 将uuid存入cookie
            Cookie token = new Cookie("sso_token",uuid);
            response.addCookie(token);
            // 保存到cookie
            return "redirect:" + url + "?token=" + uuid;
        }
        // 登录失败，展示登录页
        return "login";
    }
}
```































## 社交登录



### OAuth2

- **OAuth：**OAuth（开放授权）是一个开放标准，允许用户授权第三方网站访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方网站或分享他们的数据的内容

- **OAuth2.0：**对于用户相关的 OpenAPI（例如获取用户信息，动态同步，照片，日志，分享等），为了保存用户数据的安全和隐私，第三方网站访问用户数据前都需要显示向用户授权

- 流程

  ![image-20211112160220431](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211112160220431.png)

### 微博登录准备工作

1. 进入微博开放平台\
2. 登录微博，进入微连接，选择网站接入
3. 选择立即接入
4. 创建自己的应用
5. 开发阶段使用
6. 进入高级信息
7. 添加测试账号
8. [进入文档](https://open.weibo.com/wiki/%E6%8E%88%E6%9D%83%E6%9C%BA%E5%88%B6%E8%AF%B4%E6%98%8E)

### 代码实现

1. 点击微博登录后，跳转到微博授权页面

2. 登录成功返回code

3. 带code发请求 获取accessToken 使用accessToken可以获取用户信息

#### 授权后回调，code换AccessToken

```java
/**
     * 回调接口
     * @param code
     * @return
     * @throws Exception
     */
@GetMapping("/oauth2.0/weibo/success")
public String weibo(@RequestParam("code") String code) throws Exception {
    // 1、根据code换取accessToken
    Map<String, String> map = new HashMap<>();
    map.put("client_id", "1133714539");
    map.put("client_secret", "f22eb330342e7f8797a7dbe173bd9424");
    map.put("grant_type", "authorization_code");
    map.put("redirect_uri", "http://auth.gulimall.com/oauth2.0/weibo/success");
    map.put("code", code);


    HttpResponse response = HttpUtils.doPost("https://api.weibo.com",
                                             "/oauth2/access_token",
                                             "post",
                                             new HashMap<>(),
                                             map,
                                             new HashMap<>());

    // 状态码为200请求成功
    if (response.getStatusLine().getStatusCode() == 200 ){
        // 获取到了accessToken
        String json = EntityUtils.toString(response.getEntity());
        SocialUser socialUser = JSON.parseObject(json, SocialUser.class);
        R r = memberFeignService.OAuthlogin(socialUser);
        if (r.getCode() == 0) {
            MemberRespVo data = r.getData("data", new TypeReference<MemberRespVo>() {
            });
            log.info("登录成功:用户:{}",data.toString());

            // 2、登录成功跳转到首页
            return "redirect:http://gulimall.com";
        } else {
            // 注册失败
            return "redirect:http://auth.gulimall.com/login.html";
        }
    } else {
        // 请求失败
        // 注册失败
        return "redirect:http://auth.gulimall.com/login.html";
    }

    // 2、登录成功跳转到首页
    return "redirect:http://gulimall.com";
}
```

#### 用AccessToken 取信息

```java
@Override
public MemberEntity login(SocialUser vo) {
    // 登录和注册合并逻辑
    String uid = vo.getUid();
    MemberDao memberDao = this.baseMapper;
    // 根据社交用户的uuid查询
    MemberEntity memberEntity = memberDao.selectOne(new QueryWrapper<MemberEntity>()
            .eq("social_uid", uid));
    // 能查询到该用户
    if (memberEntity != null ){
        // 更新对应值
        MemberEntity update = new MemberEntity();
        update.setId(memberEntity.getId());
        update.setAccessToken(vo.getAccess_token());
        update.setExpiresIn(vo.getExpires_in());

        memberDao.updateById(update);

        memberEntity.setAccessToken(vo.getAccess_token());
        memberEntity.setExpiresIn(vo.getExpires_in());
        return memberEntity;
    } else {
        // 2、没有查询到当前社交用户对应的记录就需要注册一个
        MemberEntity regist = new MemberEntity();
        try {
            Map<String,String> query = new HashMap<>();
            // 设置请求参数
            query.put("access_token",vo.getAccess_token());
            query.put("uid",vo.getUid());
            // 发送get请求获取社交用户信息
            HttpResponse response = HttpUtils.doGet("https://api.weibo.com/",
                    "2/users/show.json",
                    "get",
                    new HashMap<>(),
                    query);
            // 状态码为200 说明请求成功
            if (response.getStatusLine().getStatusCode() == 200){
                // 将返回结果转换成json
                String json = EntityUtils.toString(response.getEntity());
                // 利用fastjson将请求返回的json转换为对象
                JSONObject jsonObject = JSON.parseObject(json);
                // 拿到需要的值
                String name = jsonObject.getString("name");
                String gender = jsonObject.getString("gender");
                //.. 拿到多个信息
                regist.setNickname(name);
                regist.setGender("m".equals(gender) ? 1 : 0);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 设置社交用户相关信息
        regist.setSocialUid(vo.getUid());
        regist.setAccessToken(vo.getAccess_token());
        regist.setExpiresIn(vo.getExpires_in());
        memberDao.insert(regist);
        return regist;
    }
}
```

































# MQ

异步, 消息, 解耦, 削峰



1、两个重要概念

1. 消息代理（message broker）即安装MQ的服务器

2. 目的地（destination）
   目的地表示生产者发送消息给消息代理之后，是存储到消息代理中哪一个具体的目的地（队列或主题）

   1. 队列（queue）：点对点通信【1发，n接，一个信息只会被一个接受者消费】

   2. 主题（topic）：发布/订阅【多发+多订。多个订阅者会同时接收到】
      所以后面的订单 死信路由要用topic，因为是集群的 多个发送者

     这里的queue和topic指的是交换机Exchange的类型	



JMS (Java Message Service) JAVA消息服务:【基于JAVA API，例如JDBC】
	-基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

AMQP(Advanced Message Queuing Protocol)
	-高级消息队列协议，也是一个消息代理的规范，兼容JMS
	-RabbitMQ是AMQP的实现

![image-20211110104537047](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211110104537047.png)



Spring支持
	spring-jms提供了对JMS的支持
	spring-rabbit提供了对AMQP的支持
	需要ConnectionFactory的实现来连接消息代理
	提供JmsTemplate、RabbitTemplate来发送消息
	JmsListener(JMS).@RabbitListener(AMQP）注解在方法上监听消息代理发布的消息
	-EnableJms、@EnableRabbit开启支持



Spring Boot自动配置
	-JmsAutoConfiguration
	-RabbitAutoConfiguration



市面的MQ产品

	- ActiveMQ、RabbitMQ、RocketMQ、Kafka

![image-20210913170103564](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210913170103564.png)









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
两个通配符:符号 # 和符号 *    #匹配0个或多个单词，*匹配一个单词。

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





# 购物车

## 需求

**登录状态**下将商品添加到购物车**【登陆购物车/在线购物车】**

redis

**未登录状态**下将商品添加到购物车**【游客购物车/离线购物车】**

放入 localstorage

cookie 

WebSQL

放入 redis(采用)



用户可以使用购物车一起结算下单

用户可以**查询自己的购物车**

用户可以在**购物中修改购买商品的数量**

用户可以在**购物车中删除商品**

**选中补不选中商品**

在购物车中**展示商品优惠信息**

提示购物车商品价格变化



购物车数据结构  redis 使用hash

基本字段包括

```json
{skuId:123123,
check:true,
title:"apple phone",
defaultImage:'',
price:4999,
count:1,
totalPrice:4999
skuSaleVo:{}},{...}
```

## ThreadLocal 身份验证

同线程共享数据

本质上是Map<Thread, object>

```java
/**
     * 浏览器有一个cookie：user-key,标识用户身份，一个月后过期
     * 如果第一次使用jd的购物车功能，都会给一个临时的用户身份
     * 浏览器以后保存，每次访问都会带上这个cookie
     *
     * 登录 session 有
     * 没登录，按照cookie中的user-key来做
     * 第一次：如果没有临时用户，帮忙创建一个临时用户 -->使用拦截器
```

```java
@Configuration
public class GulimallWebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new CartInterceptor())//注册拦截器
                .addPathPatterns("/**");
    }
}
```

```java
**
 * 在执行目标方法之前，判断用户的登录状态.并封装传递给controller目标请求
        */
public class CartInterceptor implements HandlerInterceptor {
    public static ThreadLocal<UserInfoTo> toThreadLocal = new ThreadLocal<>();

    /***
     * 拦截所有请求给ThreadLocal封装UserInfoTo对象
     * 1、从session中获取MemberResponseVo != null，登录状态，为UserInfoTo设置Id
     * 2、从request中获取cookie，找到user-key的value，
     * 目标方法执行之前：在ThreadLocal中存入用户信息【同一个线程共享数据】
     * 从session中获取数据【使用session需要cookie中的GULISESSION 值】
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        UserInfoTo userInfoTo = new UserInfoTo();
        HttpSession session = request.getSession();
        //获得当前登录用户的信息
        MemberResponseVo memberResponseVo = (MemberResponseVo) session.getAttribute(AuthServerConstant.LOGIN_USER);
        if (memberResponseVo != null) {
            //用户登录了
            userInfoTo.setUserId(memberResponseVo.getId());
        }
        Cookie[] cookies = request.getCookies();
        if (cookies != null && cookies.length > 0) {
            for (Cookie cookie : cookies) {
                //user-key
                String name = cookie.getName();
                if (name.equals(CartConstant.TEMP_USER_COOKIE_NAME)) {
                    userInfoTo.setUserKey(cookie.getValue());
                    // 标识客户端已经存储了 user-key
                    userInfoTo.setTempUser(true);
                }
            }
        }
        //如果没有临时用户一定分配一个临时用户
        if (StringUtils.isEmpty(userInfoTo.getUserKey())) {
            String uuid = UUID.randomUUID().toString();
            userInfoTo.setUserKey(uuid);
        }
        //目标方法执行之前
        toThreadLocal.set(userInfoTo);
        return true;
    }


    /**
     * 业务执行之后，让浏览器保存临时用户user-key
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 获取当前用户的值
        UserInfoTo userInfoTo = toThreadLocal.get();
        // 1、判断是否登录；2、判断是否创建user-token的cookie
        if (userInfoTo != null && !userInfoTo.isTempUser()) {
            //创建一个cookie
            Cookie cookie = new Cookie(CartConstant.TEMP_USER_COOKIE_NAME, userInfoTo.getUserKey());
            cookie.setDomain("gulimall.com");
            cookie.setMaxAge(CartConstant.TEMP_USER_COOKIE_TIMEOUT);
            response.addCookie(cookie);
        }
    }
}
```

```java
UserInfoTo userInfoTo = CartInterceptor.toThreadLocal.get();//同线程获取
```



## 添加购物车幂等性

**重定向到其他的页面**

```java
 /**
     * 添加商品到购物车
     * attributes.addFlashAttribute():将数据放在session中，可以在页面中取出，但是只能取一次
     * attributes.addAttribute():将数据放在url后面
     * @return
     */
    @GetMapping(value = "/addCartItem")
    public String addCartItem(@RequestParam("skuId") Long skuId,
                              @RequestParam("num") Integer num,
                              RedirectAttributes attributes) throws ExecutionException, InterruptedException {
        cartService.addToCart(skuId,num);
        attributes.addAttribute("skuId", skuId);// 给重定向请求用的【参数会拼接在下面请求之后】【转发会在请求域中】
        return "redirect:http://cart.gulimall.com/addToCartSuccessPage.html";
    }

    /**
     * 跳转到添加购物车成功页面【防止重复提交】
     */
    @GetMapping(value = "/addToCartSuccessPage.html")
    public String addToCartSuccessPage(@RequestParam("skuId") Long skuId,
                                       Model model) {
        //重定向到成功页面。再次查询购物车数据即可
        CartItemVo cartItemVo = cartService.getCartItem(skuId);
        model.addAttribute("cartItem",cartItemVo);
        return "success";
    }
```









# 订单

电商系列涉及到 3 流，分别为信息流、资金流、物流，而订单系统作为中枢将三者有机的集合起来

订单模块是电商系统的枢纽，在订单这个模块上获取多个模块的数据和信息，同时对这些信息进行加工处理后流向下个环节，这一系列就构成了订单的信息疏通

1. 信息流：商品信息、优惠信息
2. 资金流：退款、付款
3. 物流：发送、退货



## 订单服务

### 订单构成

#### 1、用户信息

用户信息包括是用户账号、用户等级、用户的收货地址、收货人、收货人电话、用户账号需要绑定手机号码，但是用户绑定的手机号码不一定是收货信息上的电话。用户可以添加多个收货信息，用户等级信息可以用来和促销系统进行匹配，获取商品折扣，同时用户等级还可以获取积分的奖励等

#### 2、订单基础信息

订单基础信息是订单流转的核心，其包括订单类型，父/子订单、订单编号、订单状态、订单流转时间等

1. 订单类型包括实体订单和虚拟订单商品等，这个根据商城商品和服务类型进行区分
2. 同时订单都需要做父子订单处理，之前在初创公司一直只有一个订单，没有做父子订单处理后期需
3. 要进行拆单的时候就比较麻烦，尤其是多商户商场，和不同仓库商品的时候，父子订单就是为后期做拆单准备的。
4. 订单编号不多说了，需要强调的一点是父子订单都需要有订单编号，需要完善的时候可以对订单编号的每个字段进行统一定义和诠释。
5. 订单状态记录订单每次流转过程，后面会对订单状态进行单独的说明。
6. 订单流转时间需要记录下单时间，支付时间，发货时间，结束时间/关闭时间等等



#### 3、商品信息

商品信息从商品库中获取商品的SKU信息、图片、名称、属性规格、商品单价、商户信息等，从用户

下单行为记录的用户下单数量，商品合计价格等



#### 4、优惠信息

优惠信息记录用户参与过的优惠活动，包括优惠促销活动，比如满减、满赠、秒杀等，用户使用的优

惠卷信息，优惠卷满足条件的优惠卷需要展示出来，另外虚拟币抵扣信息等进行记录

为什么把优惠信息单独拿出来而不放在支付信息里面呢?

因为优惠信息只是记录用户使用的条目，而支付信息需要加入数据进行计算，所以做为区分。



#### 5、支付信息

支付流水单号，这个流水单号是在唤起网关支付后支付通道返回给电商业务平台的支付流水号，财务

通过订单号和流水单号与支付通道进行对账使用。

支付方式用户使用的支付方式，比如微信支付、支付宝支付、钱包支付、快捷支付等。支付方式有时候可能有两个一-余额支付+第三方支付。

商品总金额，每个商品加总后的金额:运费，物流产生的费用;优惠总金额，包括促销活动的优惠金额，

优惠券优惠金额，虚拟积分或者虛拟币抵扣的金額，会员折扣的金额等之和;实付金额，用户实际需要

付款的金额。

**用户实付金额=商品总金额+运费 - 优惠总金额**



#### 6、物流信息

物流信息包括配送方式，物流公司，物流单号，物流状态，物流状态可以通过第三方接口来
获取和向用户展示物流每个状态节点。



### 订单状态

##### 1、待付款

用户提交订单后，订单进行预下单，目前主流电商网站都会唤起支付，便于用户快速完成支
付，需要注意的是待付款状态下可以对库存进行锁定，锁定库存需要配置支付超时时间，超
时后将自动取消订单，订单变更关闭状态。

##### 2、已付款/代发货

用户完成订单支付，订单系统需要记录支付时间，支付流水单号便于对账，订单下放到WMS系统，仓库进行调动、配货、分拣，出库等操作

##### 3、待收货/已发货

仓库将商品出库后，订单进入物流环节，订单系统需要同步物流信息，便于用户实时熟悉商品的物流状态

##### 4、已完成

用户确认收货后吗，订单交易完成，后续支付则进行计算，如果订单存在问题进入售后状态

##### 5、已取消

付款之前取消订单，包括超时未付款或用户商户取消订单都会产生这种订单状态

##### 6、售后中

用户在付款后申请退款，或商家发货后用户申请退货



售后也同样存在各种状态，当发起售后申请后生成售后订单，售后订单状态为待审核，等待

商家审核，商家审核通过后订单状态变更为待退货，等待用户将商品寄回，商家收货后订单

状态更新为待退款状态，退款到用户原账户后订单状态更新为售后成功。



### 订单流程

订单流程是指从订单产生到完成整个流转的过程，从而行程了-套标准流程规则。而不同的产品类型或业务类型在系统中的流程会千差万别，比如上面提到的线上实物订单和虚拟订单的流程，线上实物订单与020订单等，所以需要根据不同的类型进行构建订单流程。不管类型如何订单都包括正向流程和逆向流程，对应的场景就是购买商品和退换货流程，正向流程就是一一个正常的网购步骤:

订单生成>支付订单->卖家发货一确认收货>交易成功。

而每个步骤的背后，订单是如何在多系统之间交互流转的，

![image-20211111154503621](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211111154503621.png)



1、订单创建与支付

1. 订单创建前需要预览订单，选择收货信息等
2. 订单创建需要锁定库存，库存有才可创建，否则不能创建
3. 订单创建后超时未支付需要解锁库存
4. 支付成功后，需要进行拆单，根据商品打包方式，所在仓库，物流等进行拆单
5. 支付的每笔流水都需要记录，以待查账
6. 订单创建，支付成功等状态都需要给MQ发送消息，方便其他系统感知订阅

2、逆向流程

1. 修改订单，用户没有提交订单，可以对订单一些信息进行修改，比如配送信息，优惠信息，及其他一些订单可修改范围的内容，此时只需对数据进行变更即可。
2. 订单取消**，用户主动取消订单和用户超时未支付**，两种情况下订单都会取消订单，而超时情况是系统自动关闭订单，所以在订单支付的响应机制上面要做支付的





































# 事务

## 1.1 本地事务

**原子性、一致性、隔离性、持久性**

### 1.1 事务的隔离级别

- **READ UNCOMMITEED(读未提交)**
  - 读到未提交事务的数据，即脏读
- **READ COMMITTED（读提交）**
  - 一个事物可以读取另一个已提交的事务，多次读取会造成不一样的结果，此现象称为不可重复读，复读问题，Oracle 和SQL Server 的默认隔离级别
- **REPEATABLE READ（可重复读）**
  -  MySQL 默认，在同一个事物中，SELECT 的结果是事务开始时时间点的状态，因此，同样的 SELECT 操作读到的结果会是一致
  - 会有幻读,MySQL的 InnoDB 引擎可以通过 next-key locks 机制 来避免幻读
- **SERIALIZABLE（序列化）**
  - 事务串行执行，mysql的innodb会给读隐式加读共享锁, 避免脏读 ,幻读, 不可重复读



越大的隔离级别，并发能力越低

### 1.2 事务的传播行为

1. **PROPAGATION_ REQUIRED**: 如果当前没有事务，就创建一个新事务， 如果当前存在事务，就加入该事务，该设置是最常用的设置。
2. PROPAGATION_ SUPPORTS: 支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
3. PROPAGATION _MANDATORY:支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。_
4. **PROPAGATION_ REQUIRES_ NEW**:创建新事务，无论当前存不存在事务，都创建新事务。
5. PROPAGATION NOT_ SUPPORTED: 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
6. PROPAGATION_ NEVER: 以非事务方式执行，如果当前存在事务，则抛出异常。
7. PROPAGATION NESTED: 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_ REQUIRED 类似的操作。



==同一个对象内事务方法互调默认失效, 原因 绕过了代理对象 事务使用代理对象来控制的==

方法a、b、c都在同一个service里面，事务传播行为不生效，共享一个事务

解决：使用代理对象来调用事务方法

1. 引入 spring-boot-starter-aop 它帮我们引入了aspectj

2. 主启动@EnableAspectJAutoProxy(exposeProxy = true) [对外暴露代理对象] 开启aspectj动态代理功能 而不是jdk默认的动态代理 即使没有接口也可以创建动态代理

3. 本类互调用代理对象      (强转)AopContext.currentProxy()

   ```java
   @EnableTransactionManagement(proxyTargetClass= true)
   //手动开启事务管理
   ```

## 1.2 分布式事务

### 1.2.1 CAP 定理

CAP 原则又称 CAP 定理指的是在一个分布式系统中

- **一致性（Consistency）**
  - 在分布式系统中所有数据备份，在同一时刻是否是同样的值，（等同于所有节点访问同一份最新数据的副本）
- **可用性（Avaliability）**
  - 在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求，（对数据更新具备高可用性）
- **分区容错性（Partition tolerance）**
  - 大多数分布式系统都分布在多个子网络，每个自网络叫做一个区（partition）。分区容错性的意思是，区间通信可能失败，比如，一台服务器放在中国另一台服务器放在美国，这就是两个区，它们之间可能无法通信

CAP 的原则是，这三个要素最多只能满足两个点，**不可能三者兼顾**



一般来说，分区容错无法避免，因此我们可以认为 CAP 的 P 总是成立，CAP 定理告诉我们 剩下的 C 和 A 无法同时做到

分布式实现一致性的 raft 算法

http://thesecretlivesofdata.com/raft/



**2、面临的问题**

对于大多数互联网应用的场景、主机众多、部署分散，而且集群规模越来越大，所以节点故障，网络故障是常态，而且要保证服务可用性达到99.999%，即保证P 和 A,舍弃C



### 1.2.2 BASE 理论

是对CAP的延申，思想即是无法做到强一致性（CAP的一致性就是强一致性），但可以采用适当的弱一致性，即**最终一致性**

BASE 是指

- 基本可用（Basically Avaliable）

  - 基本可用是指分布式系统中在出现故障的时候，允许损失部分可用性（列入响应时间，功能上的可用性）允许损失部分可用性。需要注意的是基本可用不等价于系统不可用
  - 响应时间上的损失，正常情况下搜索引擎需要在0.5秒之内返回给用户相应的查询结果，但由于出现故障（比如系统部分机房断电断网故障），查询的结果响应时间增加到了1~2秒
  - 功能上的损失，购物网站双十一购物高峰，为了保证系统的稳定性，部分消费者会被引入到一个降级页面

- **软状态（Soft State）**

  - 软状态是指允许 系统存在中间状态，而该中间状态不会影响系统整体可用性。分布式存储中一般一份数据会有 多个副本，允许不同副本同步的延时就是软状态的体现。mysglreplication的异步复制也是一种体现。

- **最终一致性( Eventual Consistency)**

  - 最终致性是指系统中的所有数据副本经过一定时间后，最终能够达到一 致的状态。弱一致性和强一致性相反，最终一致性是弱一致性的一种特殊情况。

    

**强一致性、弱一致性、最终一致性**



强一致性：更新后的数据后续访问能看到【本地事务，立即回滚】
弱一致性：容忍部分或全部访问不到（数据不一致）【软状态，存在不一致的数据】
最终一致性：一段时间后能更新到最新数据【经过一段时间后回滚】



### 1.2.3 分布式事务的几种方案

#### 1.2.3.1 2PC 模式

数据库支持的 2PC[2 phase commit 二阶提交]，又叫 XA Transactions

MySQL 从5.5版本开始支持，Sql Server 2005 开始支持，oracle7 开始支持

其中，XA 是一个两阶段提交协议，该协议分为两个阶段：

第一阶段：事务协调器要求每个涉及到事务的数据库预提交（precommit）此操作，并反应是否可以提交

第二阶段：事务协调器要求每个数据库提交数据

其中，如果有任何一个数据库否认这次提交，那么所有数据库都会要求回滚他们在此事务中的那部分信息

![image-20210426145239641](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210426145239641.png)

- XA协议比较简单，而且一旦商业数据库实现了XA协议，使用分布式事务的成本也比较低。
- **XA性能不理想**，特别是在交易下单链路，往往并发量很高，XA无法满足高并发场景
- XA目前在商业数据库支持的比较理想，**在mysql数据库中支持的不太理想**，mysgl的XA实现，没有记录prepare阶段日志，主备切换回导致主库与备库数据不一致。
- 许多nosgI也没有支持XA，这让XA的应用场景变得非常狭隘。
- 也有3PC,引入了超时机制(无论协调者还是参与者，在向对方发送请求后，若长时间未收到回应则做出相应处理)



#### 1.2.3.2柔性事务 - TCC 事务补偿方案

刚性事务:遵循ACID原则，强一致性。
柔性事务:遵循BASE理论，最终一致性;
与刚性事务不同，柔性事务允许一定时间内，不同节点的数据不一致，但要求最终一致。

![image-20210426150137500](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210426150137500.png)

一阶段 prepare 行为:调用自定义的 prepare 逻辑。
二阶段 commit 行为:调用自定义的 commit 逻辑。
二阶段 rollback行为:调用自定义的 rollback 逻辑。
所谓TCC模式，是指支持把自定义的分支事务纳入到全局事务的管理中。



#### 1.2.3.3 柔性事务 - 最大努力通知型方案

按规律进行通知，**不保证数据定能通知成功， 但会提供可查询操作接口进行核对**。这种方案主要用在与第三方系统通讯时，比如:调用微信或支付宝支付后的支付结果通知。这种方案也是结合MQ进行实现，例如:通过MQ发送http请求，设置最大通知次数。达到通知次数后即不再通知。

案例:银行通知、商户通知等(各大交易业务平台间的商户通知:多次通知、查询校对、对账文件)，支付宝的支付成功异步回调

#### 1.2.3.4 柔性事务 - ==可靠信息== + 最终一致性方案（异步通知型）

实现:业务处理服务在业务事务提交之前，向实时消息服务请求发送消息，实时消息服务只记录消息数据，而不是真正的发送。业务处理服务在业务事务提交之后，向实时消息服务确认发送。只有在得到确认发送指令后，实时消息服务才会真正发送。






































































# 幂等性

**同一操作发起的一次请求或者多次请求的结果一致**



需要防止的情况

- 多次点击
- 回退再提交
- 微服务互相调用, 网络问题, 请求失败, feign触发重试
- 其他



## 1.1 解决方案

### 1.1.1 token机制

1、服务端提供了发送token的接口。我们在分析业务的时候，哪些业务是存在幂等问题的，就必须在执行业务前，先去获取token,服务器会把token保存到redis中。

2、然后调用业务接口请求时，把token携带过去，一般放在请求头部。
3、服务器判断token是否存在redis中，存在表示第一次请求，然后删除token,继续执行业务。
4、如果判断token不存在redis 中，就表示是重复操作，直接返回重复标记给client, 这样就保证了业务代码，不被重复执行。



**危险性:**
1、先删除token还是后删除token;

1. 先删除可能导致，业务确实没有执行，重试还带上之前token, 由于防重设计导致，请求还是不能执行。
2. 后删除 可能导致，业务处理成功，但是服务闪断，出现超时，没有删除token,别人继续重试，导致业务被执行两遍
3. 我们最好设计为先删除token,如果业务调用失败，就重新获取token再次请求。

2、Token 获取、比较和删除必须是原子性

1. redis.get(token)、 token.equals 、redis.del(token)如果这两个操作不是原子， 可能导致，高并发下，都get到同样的数据，判断都成功，继续业务并发执行

2. 可以在 redis使用lua脚本完成这个操作

   ```lua
   if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end
   ```

### 1.1.2 各种锁

#### 1.1.2.1 数据库悲观锁

select * from xxXX where id = 1 for update;
悲观锁使用时一般伴随事务起使用，数据锁定时间可能会很长，需要根据实际情况选用。
另外要注意的是，id字段一定是主键或者唯一索引，不然可能造成锁表的结果，处理起来会非常麻烦。

#### 1.1.2.2 数据库乐观锁

这种方法适合在更新的场景中，
update t_ goods set count = count -1，version = version+ 1 where good_ id=2 and version= 1

根据version版本，也就是在操作库存前先获取当前商品的version版本号，然后操作的时候带上此version号。我们梳理下，我们第一次操作库存时，得到version为1，调用库存服务version变成了2;但返回给订单服务出现了问题，订单服务又一次发起调用库存服务，当订单服务传如的version还是1，再执行上面的sgl语句时，就不会执行;因为version已经变为2了，where 条件就不成立。这样就保证了不管调用几次，只会真正的处理一次。

乐观锁主要使用于处理读多写少的问题

#### 1.1.2.3 业务层分布式锁

如果多个机器可能在同一时间同时处理相同的数据，比如多台机器定时任务都拿到了相同数据处理，我们就可以加分布式锁，锁定此数据，处理完成后释放锁。获取到锁的必须先判断这个数据是否被处理过。

### 1.1.3 各种唯一约束

#### 1.1.3.1 数据库唯一约束

插入数据，应该按照唯一索引进行插入，比如订单号，相同的订单就不可能有两条记录插入。我们在数据库层面防止重复。

这个机制是利用了数据库的主键唯约束的特性， 解决了在insert 场景时幂等问题。 但主键的要求不是自增的主键，这样就需要业务生成全局唯一的主键。

如果是分库分表场景下，路由规则要保证相同请求下，落地在同一个数据库和同一表中，要不然数据库主键约束就不起效果了，因为是不同的数据库和表主键不相关。

#### 1.1.3.2 redis set防重

很多数据需要处理，只能被处理一次，比如我们可以计算数据的MD5将其放入redis的set,每次处理数据，先看这个MD5是否已经存在，存在就不处理。

### 1.1.4 防重表

使用订单号orderNo做为去重表的唯一索引， 把唯一索引插入去重表，再进行业务操作，且他们在同一个事务中。这个保证了重复请求时，因为去重表有唯一约束，导致请求失败，避免了幂等问题。

这里要注意的是，去重表和业务表应该在同一库中，这样就保证了在同一个事务，即使业务操作失败了，也会把去重表的数据回滚。这个很好的保证了数据一致性。

之前说的redis防重也算

### 1.1.5 全局请求唯一id

调用接口时，生成一个唯一id, redis 将数据保存到集合中(去重)，存在即处理过。
可以使用nginx设置每一个请求的唯一id;

```
proxy_set_header X-Request-id $request_ id;
```

# 支付

## 支付宝

### 进入蚂蚁金服开放平台

https://open.alipay.com/platform/home.htm

下载支付宝官方Demo，进行配置和测试

文档地址：

https://openhome.alipay.com/docCenter/docCenter.htm 创建应用对应文档

https://opendocs.alipay.com/open/200/105304 网页移动应用文档

https://opendocs.alipay.com/open/54/cyz7do 相关Demo



创建应用

### 使用沙箱进行测试

https://openhome.alipay.com/platform/appDaily.htm?tab=info

### 公钥、私钥、加密、签名和验签

RSA算法

![image-20210607143403871](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210607143403871.png)

### 支付宝支付流程

https://opendocs.alipay.com/open/270/105898

1. 引导用户进入到支付宝页面

2. 用户点击付款

3. 付款成功后跳转到成功页面

### 订单整合支付宝

#### 支付

```xml
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
    <version>4.10.111.ALL</version>
</dependency>
```

AlipayTemplate.java 和 payVo.java

```java
@ConfigurationProperties(prefix = "alipay")
@Component
@Data
public class AlipayTemplate {

    // 应用ID,您的APPID，收款账号既是您的APPID对应支付宝账号
    public String app_id;

    // 商户私钥，您的PKCS8格式RSA2私钥
    public String merchant_private_key;

    // 支付宝公钥,查看地址：https://openhome.alipay.com/platform/keyManage.htm 对应APPID下的支付宝公钥。
    public String alipay_public_key;

    // 服务器[异步通知]页面路径  需http://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问
    // 支付宝会悄悄的给我们发送一个请求，告诉我们支付成功的信息
    public String notify_url;

    // 页面跳转同步通知页面路径 需http://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问
    //同步通知，支付成功，一般跳转到成功页
    public String return_url;

    // 签名方式
    private  String sign_type;

    // 字符编码格式
    private  String charset;

    //订单超时时间
    private String timeout = "1m";

    // 支付宝网关； https://openapi.alipaydev.com/gateway.do
    public String gatewayUrl;

    public  String pay(PayVo vo) throws AlipayApiException {

        //AlipayClient alipayClient = new DefaultAlipayClient(AlipayTemplate.gatewayUrl, AlipayTemplate.app_id, AlipayTemplate.merchant_private_key, "json", AlipayTemplate.charset, AlipayTemplate.alipay_public_key, AlipayTemplate.sign_type);
        //1、根据支付宝的配置生成一个支付客户端
        AlipayClient alipayClient = new DefaultAlipayClient(gatewayUrl,
                                                            app_id, merchant_private_key, "json",
                                                            charset, alipay_public_key, sign_type);

        //2、创建一个支付请求 //设置请求参数
        AlipayTradePagePayRequest alipayRequest = new AlipayTradePagePayRequest();
        alipayRequest.setReturnUrl(return_url);
        alipayRequest.setNotifyUrl(notify_url);

        //商户订单号，商户网站订单系统中唯一订单号，必填
        String out_trade_no = vo.getOut_trade_no();
        //付款金额，必填
        String total_amount = vo.getTotal_amount();
        //订单名称，必填
        String subject = vo.getSubject();
        //商品描述，可空
        String body = vo.getBody();

        alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\","
                                    + "\"total_amount\":\""+ total_amount +"\","
                                    + "\"subject\":\""+ subject +"\","
                                    + "\"body\":\""+ body +"\","
                                    + "\"timeout_express\":\""+timeout+"\","
                                    + "\"product_code\":\"FAST_INSTANT_TRADE_PAY\"}");

        String result = alipayClient.pageExecute(alipayRequest).getBody();

        //会收到支付宝的响应，响应的是一个页面，只要浏览器显示这个页面，就会自动来到支付宝的收银台页面
        System.out.println("支付宝的响应："+result);

        return result;

    }
}
@Data
public class PayVo {
    private String out_trade_no; // 商户订单号 必填
    private String subject; // 订单名称 必填
    private String total_amount;  // 付款金额 必填
    private String body; // 商品描述 可空
}
```

```properties
#支付宝相关的配置
alipay.app_id=2021000118641369
alipay.merchant_private_key= #商户私钥
alipay.alipay_public_key= #支付宝公钥
alipay.notify_url=http://5kargw.natappfree.cc/payed/notify
alipay.return_url=http://member.gulimall.com/memberOrder.html
alipay.sign_type=RSA2
alipay.charset=utf-8
alipay.gatewayUrl=https://openapi.alipaydev.com/gateway.do
```



```java
@Slf4j
@Controller
public class PayWebController {

    @Autowired
    private AlipayTemplate alipayTemplate;

    @Autowired
    private OrderService orderService;

    @Autowired
    private BestPayService bestPayService;

    @Resource
    private WxPayConfig wxPayConfig;

    /**
     * 用户下单:支付宝支付
     * 1、让支付页让浏览器展示
     * 2、支付成功以后，跳转到用户的订单列表页
     * @param orderSn
     * @return
     * @throws AlipayApiException
     */
    @ResponseBody
    @GetMapping(value = "/aliPayOrder",produces = "text/html")
    public String aliPayOrder(@RequestParam("orderSn") String orderSn) throws AlipayApiException {

        PayVo payVo = orderService.getOrderPay(orderSn);
        // 支付宝返回一个页面【支付宝账户登录的html页面】
        String pay = alipayTemplate.pay(payVo);
        System.out.println(pay);
        return pay;
    }


    /**
     * 微信支付
     * @param orderSn
     * @return
     */
    @GetMapping(value = "/weixinPayOrder")
    public String weixinPayOrder(@RequestParam("orderSn") String orderSn, Model model) {

        OrderEntity orderInfo = orderService.getOrderByOrderSn(orderSn);

        if (orderInfo == null) {
            throw new RuntimeException("订单不存在");
        }

        PayRequest request = new PayRequest();
        request.setOrderName("4559066-最好的支付sdk");
        request.setOrderId(orderInfo.getOrderSn());
        request.setOrderAmount(0.01);
        request.setPayTypeEnum(WXPAY_NATIVE);

        PayResponse payResponse = bestPayService.pay(request);
        payResponse.setOrderId(orderInfo.getOrderSn());
        log.info("发起支付 response={}", payResponse);

        //传入前台的二维码路径生成支付二维码
        model.addAttribute("codeUrl",payResponse.getCodeUrl());
        model.addAttribute("orderId",payResponse.getOrderId());
        model.addAttribute("returnUrl",wxPayConfig.getReturnUrl());

        return "createForWxNative";
    }


    //根据订单号查询订单状态的API
    @GetMapping(value = "/queryByOrderId")
    @ResponseBody
    public OrderEntity queryByOrderId(@RequestParam("orderId") String orderId) {
        log.info("查询支付记录...");
        return orderService.getOrderByOrderSn(orderId);
    }
}
```

#### 支付成功同步跳转

```java
//alipay.return_url=http://member.gulimall.com/memberOrder.html
@Controller
public class MemberWebController {

    @Autowired
    private OrderFeignService orderFeignService;

    @GetMapping(value = "/memberOrder.html")
    public String memberOrderPage(@RequestParam(value = "pageNum",required = false,defaultValue = "0") Integer pageNum,
                                  Model model, HttpServletRequest request) {

        //获取到支付宝给我们转来的所有请求数据
        //request,验证签名


        //查出当前登录用户的所有订单列表数据
        Map<String,Object> page = new HashMap<>();
        page.put("page",pageNum.toString());

        //远程查询订单服务订单数据
        R orderInfo = orderFeignService.listWithItem(page);
        System.out.println(JSON.toJSONString(orderInfo));
        model.addAttribute("orders",orderInfo);

        return "orderList";
    }

}

@FeignClient("gulimall-order")
public interface OrderFeignService {

    /**
     * 分页查询当前登录用户的所有订单信息
     */
    @PostMapping("/order/order/listWithItem")
    R listWithItem(@RequestBody Map<String, Object> params);

}

/**
     * 分页查询当前登录用户的所有订单信息
     */
@PostMapping("/listWithItem")
//@RequiresPermissions("order:order:list")
public R listWithItem(@RequestBody Map<String, Object> params){
    PageUtils page = orderService.queryPageWithItem(params);

    return R.ok().put("page", page);
}

/**
     * 查询当前用户所有订单数据
     */
@Override
public PageUtils queryPageWithItem(Map<String, Object> params) {

    MemberResponseVo memberResponseVo = LoginUserInterceptor.loginUser.get();

    IPage<OrderEntity> page = this.page(
        new Query<OrderEntity>().getPage(params),
        new QueryWrapper<OrderEntity>()
        .eq("member_id",memberResponseVo.getId()).orderByDesc("create_time")
    );

    //遍历所有订单集合
    List<OrderEntity> orderEntityList = page.getRecords().stream().map(order -> {
        //根据订单号查询订单项里的数据
        List<OrderItemEntity> orderItemEntities = orderItemService.list(new QueryWrapper<OrderItemEntity>()
                                                                        .eq("order_sn", order.getOrderSn()));
        order.setOrderItemEntityList(orderItemEntities);
        return order;
    }).collect(Collectors.toList());

    page.setRecords(orderEntityList);

    return new PageUtils(page);
}
```

#### 支付成功异步通知

支付宝 分布式事务，使用的是最大努力通知方案



```nginx
#修改nginx配置文件 适应测设环境 
location /payed/ {
    proxy_set_header Host order.gulimall.com;
    proxy_pass http://gulimall;
}
#server_name 配置上内网穿透的host：iua4i6.natappfree.cc
server_name  gulimall.com *.gulimall.com iua4i6.natappfree.cc;
```

登录拦截器放行/payed/notify请求

```java
//支付成功异步跳转的地址+内网穿透
//alipay.notify_url=http://5kargw.natappfree.cc/payed/notify
@RestController
public class OrderPayedListener {

    @Autowired
    private OrderService orderService;

    @Autowired
    private AlipayTemplate alipayTemplate;

   @PostMapping(value = "/payed/notify")
    public String handleAlipayed(PayAsyncVo asyncVo, HttpServletRequest request) throws AlipayApiException, UnsupportedEncodingException {
        // 只要收到支付宝的异步通知，返回 success 支付宝便不再通知
        // 获取支付宝POST过来反馈信息
        //验签
        boolean signVerified = isSignVerified(request);

        if (signVerified) {
            System.out.println("签名验证成功...");
            //去修改订单状态
            String result = orderService.handlePayResult(asyncVo);
            return result;
        } else {
            System.out.println("签名验证失败...");
            return "error";
        }
    }

    @PostMapping(value = "/pay/notify")
    public String asyncNotify(@RequestBody String notifyData) {
        //异步通知结果
        return orderService.asyncNotify(notifyData);
    }

}
```

```java
@ToString
@Data
public class PayAsyncVo {
    private String gmt_create;
    private String charset;
    private String gmt_payment;
    private Date notify_time;
    private String subject;
    private String sign;
    private String buyer_id;//支付者的id
    private String body;//订单的信息
    private String invoice_amount;//支付金额
    private String version;
    private String notify_id;//通知id
    private String fund_bill_list;
    private String notify_type;//通知类型； trade_status_sync
    private String out_trade_no;//订单号
    private String total_amount;//支付的总额
    private String trade_status;//交易状态  TRADE_SUCCESS
    private String trade_no;//流水号
    private String auth_app_id;//
    private String receipt_amount;//商家收到的款
    private String point_amount;//
    private String app_id;//应用id
    private String buyer_pay_amount;//最终支付的金额
    private String sign_type;//签名类型
    private String seller_id;//商家的id
}
```

##### 验单

```java
private boolean isSignVerified(HttpServletRequest request) throws AlipayApiException {
    Map<String, String> params = new HashMap<>();
    Map<String, String[]> requestParams = request.getParameterMap();
    for (String name : requestParams.keySet()) {
        String[] values = requestParams.get(name);
        String valueStr = "";
        for (int i = 0; i < values.length; i++) {
            valueStr = (i == values.length - 1) ? valueStr + values[i]
                : valueStr + values[i] + ",";
        }
        //乱码解决，这段代码在出现乱码时使用
        // valueStr = new String(valueStr.getBytes("ISO-8859-1"), "utf-8");
        params.put(name, valueStr);
    }

    boolean signVerified = AlipaySignature.rsaCheckV1(params, alipayTemplate.getAlipay_public_key(),
                                                      alipayTemplate.getCharset(), alipayTemplate.getSign_type()); //调用SDK验证签名
    return signVerified;
}
```

#### 收单

* 自动收单功能解决。只要一段时间不支付，就不能支付了

  ```java
  //发送请求的时候带上超时时间：
  //private String timeout = "1m";
  @ConfigurationProperties(prefix = "alipay")
  @Component
  @Data
  public class AlipayTemplate {
  
      // 应用ID,您的APPID，收款账号既是您的APPID对应支付宝账号
      public String app_id;
  
      // 商户私钥，您的PKCS8格式RSA2私钥
      public String merchant_private_key;
  
      // 支付宝公钥,查看地址：https://openhome.alipay.com/platform/keyManage.htm 对应APPID下的支付宝公钥。
      public String alipay_public_key;
  
      // 服务器[异步通知]页面路径  需http://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问
      // 支付宝会悄悄的给我们发送一个请求，告诉我们支付成功的信息
      public String notify_url;
  
      // 页面跳转同步通知页面路径 需http://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问
      //同步通知，支付成功，一般跳转到成功页
      public String return_url;
  
      // 签名方式
      private  String sign_type;
  
      // 字符编码格式
      private  String charset;
  
      //订单超时时间
      private String timeout = "1m";
  
      // 支付宝网关； https://openapi.alipaydev.com/gateway.do
      public String gatewayUrl;
  
      public  String pay(PayVo vo) throws AlipayApiException {
          //AlipayClient alipayClient = new DefaultAlipayClient(AlipayTemplate.gatewayUrl, AlipayTemplate.app_id, AlipayTemplate.merchant_private_key, "json", AlipayTemplate.charset, AlipayTemplate.alipay_public_key, AlipayTemplate.sign_type);
          //1、根据支付宝的配置生成一个支付客户端
          AlipayClient alipayClient = new DefaultAlipayClient(gatewayUrl,
                  app_id, merchant_private_key, "json",
                  charset, alipay_public_key, sign_type);
  
          //2、创建一个支付请求 //设置请求参数
          AlipayTradePagePayRequest alipayRequest = new AlipayTradePagePayRequest();
          alipayRequest.setReturnUrl(return_url);
          alipayRequest.setNotifyUrl(notify_url);
  
          //商户订单号，商户网站订单系统中唯一订单号，必填
          String out_trade_no = vo.getOut_trade_no();
          //付款金额，必填
          String total_amount = vo.getTotal_amount();
          //订单名称，必填
          String subject = vo.getSubject();
          //商品描述，可空
          String body = vo.getBody();
  
          alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\","
                  + "\"total_amount\":\""+ total_amount +"\","
                  + "\"subject\":\""+ subject +"\","
                  + "\"body\":\""+ body +"\","
                  + "\"timeout_express\":\""+timeout+"\","
                  + "\"product_code\":\"FAST_INSTANT_TRADE_PAY\"}");
  
          String result = alipayClient.pageExecute(alipayRequest).getBody();
  
          //会收到支付宝的响应，响应的是一个页面，只要浏览器显示这个页面，就会自动来到支付宝的收银台页面
          System.out.println("支付宝响应：登录页面的代码\n"+result);
  
          return result;
      }
  }
  ```

* 手动收单

  ```java
  //设置请求参数
  AlipayTradeCloseRequest alipayRequest = new AlipayTradeCloseRequest();
  //商户订单号，商户网站订单系统中唯一订单号
  String out_trade_no = new String(request.getParameter("WIDTCout_trade_no").getBytes("ISO-8859-1"),"UTF-8");
  //支付宝交易号
  String trade_no = new String(request.getParameter("WIDTCtrade_no").getBytes("ISO-8859-1"),"UTF-8");
  //请二选一设置
  
  alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\"," +"\"trade_no\":\""+ trade_no +"\"}");
  
  //请求
  String result = alipayClient.execute(alipayRequest).getBody();
  ```

* 网络阻塞问题，订单支付成功的异步通知一直不到达
  查询订单列表时，ajax获取当前未支付的订单状态，查询订单状态时，再获取一下支付宝此订单的状态

* 其他各种问题
  定时任务 每天晚上闲时下载支付宝对账单，——进行对账









## 内网穿透

#### 1、简介

内网穿透功能可以允许我们使用外网的网址来访问主机;
正常的外网需要访问我们项目的流程是:

1、买服务器并且有公网固定IP

2、买域名映射到服务器的IP

3、域名需要进行备案和审核

#### 2、使用场景

1、开发测试(微信、支付宝)

2、智慧互联

3、远程控制

4、私有云

#### 3、内网穿透常用的软件

1、natapp:https://natapp.cn/
2、续断: www.zhexi.tech
3、花生壳: https://www.oray.com/



# 秒杀&定时/异步任务

瞬间高并发

限流 + 异步 + 缓存（页面静态化） + 独立部署



限流方式：

1、前端限流，一些高并发的网站直接在前端页面开始限流，列如：小米的验证码

2、nginx限流，直接负载部分请求到错误的静态页面，令牌算法、漏斗算法

3、网关限流、限流的过滤器

4、代码中使用分布式信号量

5、rabbitmq限流（能者多劳 channel.basicQos(1)）保证发挥服务器的所用性能



## 秒杀（高并发）系统关注的问题

1. **服务单一职责**

   秒杀服务即使自己扛不住压力，挂掉，不要影响别人

2. **秒杀链接加密**

   防止恶意攻击模拟秒杀请求，1000次/s攻击
   防止连接暴露，自己工作人员，提前秒杀商品

3. **库存预热 + 快速扣减** 

   提前把数据加载缓存中，

4. **动静分离**

   nginx做好动静分离，保证秒杀和商品详情页的动态请求才打到后端的服务集群

   使用 CDN 网络，分担服务器压力

5. **恶意请求拦截**

   识别非法攻击的请求并进行拦截	，网关层

6. **流量错峰**

   使用各种手段，将流量分担到更大宽度的时间点，比如验证码，加入购物车

7.  **限流 & 熔断 & 降级**

   前端限流 + 后端限流

   限制次数，限制总量，快速失败降级运行，熔断隔离防止雪崩

8. **队列消峰**

   1万个请求，每个1000件被秒杀，双11所有秒杀成功的请求，进入队列，慢慢创建订单，扣减库存即可

## 秒杀实现

秒杀的两种实现，我们采用了第二种

1. 第一种：加入购物车，跳转订单确认页，提交秒杀订单，信号量扣减，扣减成功后支付【还是订单系统】
       	弊端：可能导致各系统流量都很大【购物车服务、商品服务、订单服务(订单确认页)】
       	优点：业务是统一的，数据模型是一致的，秒杀业务只是价格不同

   ![image-20211109104955811](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211109104955811.png)

2. 第二种：超高并发可选择这个
       	1）定时任务上架 活动时，就已经将库存锁定了。例如DB一次性锁定400
       	2）然后直接将用户、订单号、商品  直接发送给mq，然后直接返回秒杀成功，进入地址确认、支付确认页
       	3）创建订单异步执行，由订单服务监听队列
       	优点：返回页面没有操作DB，没有远程调用

   ![image-20211109105031993](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211109105031993.png)

   发送订单创建消息到mq：销峰

   ![image-20211109105109752](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211109105109752.png)



定时上架秒杀场次至redis



```java
//mq监听队列创建订单：
@Slf4j
@Component
@RabbitListener(queues = "order.seckill.order.queue")
public class OrderSeckillListener {

    @Autowired
    private OrderService orderService;

    @RabbitHandler
    public void listener(SeckillOrderTo orderTo, Channel channel, Message message) throws IOException {
        log.info("准备创建秒杀单的详细信息...");
        try {
            orderService.createSeckillOrder(orderTo);
            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
        } catch (Exception e) {
            channel.basicReject(message.getMessageProperties().getDeliveryTag(),true);
        }
    }
}
```







## 定时任务

```
多种实现方式：
	1、Timer
	2、线程池
	3、mq的延时队列
	4、QUARTZ【搭配cron表达式使用】
	5、spring框架的定时任务，可以整合QUARTZ
		springboot默认定时任务框架不是QUARTZ，如果需要使用引入即可
```

### cron表达式

语法：秒 分 时 日 月 周 年（Spring不支持年）

http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/crontrigger.html

![image-20211108134305458](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211108134305458.png)

特殊字符：

- `*`（ *“所有值”*）-用于选择字段中的所有值。例如，分钟字段中的“ ***** ”表示*“每分钟”*。
- `？`（ *“无特定值”*）-当您需要在允许使用字符的两个字段之一中指定某个内容而在另一个不允许的字段中指定某些内容时很有用。例如，如果我希望在某个月的某个特定日期（例如，第10天）触发触发器，但不在乎一周中的哪一天发生，则将“ 10”设置为月字段，以及“？” 在“星期几”字段中。请参阅下面的示例以进行澄清。
- `--`用于指定范围。例如，小时字段中的“ 10-12”表示*“小时10、11和12”*。
- `，` -用于指定其他值。例如，“星期几”字段中的“ MON，WED，FRI”表示*“星期一，星期三和星期五的日子”*。
- `/` -用于指定增量。例如，秒字段中的“ 0/15”表示*“秒*0、15、30*和45”*。秒字段中的“ 5/15”表示*“秒*5、20、35*和50”*。您还可以在“ **”字符**后指定“ /” **-在这种情况下，“** ”等效于在“ /”之前使用“ 0”。每月日期字段中的“ 1/3”表示*“从该月的第一天开始每三天触发一次”*。
- `L`（ *“最后一个”*）-在允许使用的两个字段中都有不同的含义。例如，“月”字段中的值“ L”表示*“*月*的最后一天”，即*非January年的1月31日，2月28日。如果单独用于星期几字段，则仅表示“ 7”或“ SAT”。但是，如果在星期几字段中使用另一个值，则表示*“该月的最后xxx天”*，例如“ 6L”表示*“该月的最后一个星期五”*。您还可以指定与该月最后一天的偏移量，例如“ L-3”，这表示日历月的倒数第三天。 *使用“ L”选项时，不要指定列表或值的范围很重要，因为这样会导致混淆/意外结果。*
- `W`（ *“工作日”*）-用于指定最接近给定日期的工作日（星期一至星期五）。例如，如果您要指定“ 15W”作为“月日”字段的值，则含义是： *“离月15日最近的工作日”*。因此，如果15号是星期六，则触发器将在14号星期五触发。如果15日是星期日，则触发器将在16日星期一触发。如果15号是星期二，那么它将在15号星期二触发。但是，如果您将“ 1W”指定为月份的值，并且第一个是星期六，则触发器将在第3个星期一触发，因为它不会“跳过”一个月日的边界。仅当月份中的某天是一天，而不是范围或天数列表时，才可以指定“ W”字符。

> “ L”和“ W”字符也可以在“月日”字段中组合以产生“ LW”，这表示*“每月的最后一个工作日” *。

- `＃` -用于指定每月的第“ XXX”天。例如，“星期几”字段中的值“ 6＃3”表示*“该月的第三个星期五”*（第6天=星期五，“＃3” =该*月的第三个星期五*）。其他示例：“ 2＃1” =该月的第一个星期一，“ 4＃5” =该月的第五个星期三。请注意，如果您指定“＃5”，并且该月的指定星期几中没有5个，则该月将不会触发。

> 法定字符以及月份和星期几的名称不区分大小写。`MON` 与`mon`相同。

### SpringBoot定时任务&异步任务

```java
/**
 * 定时任务
 *      1、@EnableScheduling 开启定时任务
 *      2、Scheduled 开启一个定时任务
 *      3、自动配置类 TaskSchedulingAutoConfiguration 属性绑定在TaskExecutionProperties
 *
 * 异步任务
 *      1、@EnableAsync 开启异步任务功能
 *      2、@Async 给希望异步执行的方法上标注
 *      3、自动配置类 TaskExecutionAutoConfiguration
 *
 */
@Slf4j
@Component
@EnableAsync // 启用Spring异步任务支持
@EnableScheduling // 启用Spring的计划任务执行功能
public class HelloSchedule {

    /**
     * 1、Spring中6位组成，不允许第7位的年
     * 2、在周几的位置，1-7代表周一到周日：MON-SUN
     * 3、定时任务不应该阻塞，默认是阻塞的
     *      1、可以让业务以异步的方式运行，自己提交到线程池
     *          CompletableFuture.runAsync(() -> {
     *              xxxService.hello();
     *          })
     *      2、支持定时任务线程池，设置 TaskSchedulingProperties
     *          spring.task.scheduling.pool.size=5
     *      3、让定时任务异步执行
     *          异步执行
     *     解决：使用异步任务 + 定时任务来完成定时任务不阻塞的功能
     */
//    @Async 异步
//    @Scheduled(cron = "* * * * * ?")
//    public void hello() {
//        log.info("hello...");
//    }
}
```

### 缓存秒杀信息--分布式幂等性

分布式锁 + 判断

```java
@Slf4j
@Service
public class SeckillScheduled {

    @Autowired
    private SeckillService seckillService;

    @Autowired
    private RedissonClient redissonClient;

    //秒杀商品上架功能的锁
    private final String upload_lock = "seckill:upload:lock";

    // @Scheduled(cron = "*/5 * * * * ? ")
    @Scheduled(cron = "0 0 1/1 * * ? ")
    public void uploadSeckillSkuLatest3Days() {
        //1、重复上架无需处理
        log.info("上架秒杀的商品...");

        //分布式锁
        RLock lock = redissonClient.getLock(upload_lock);
        try {
            //加锁
            lock.lock(10, TimeUnit.SECONDS);
            seckillService.uploadSeckillSkuLatest3Days();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
/**
     * 缓存秒杀活动所关联的商品信息
     */
private void saveSessionSkuInfo(List<SeckillSessionWithSkusVo> sessions) {

    sessions.stream().forEach(session -> {
        //准备hash操作，绑定hash
        BoundHashOperations<String, Object, Object> operations = redisTemplate.boundHashOps(SECKILL_CHARE_PREFIX);
        session.getRelationSkus().stream().forEach(seckillSkuVo -> {
            //生成随机码
            String token = UUID.randomUUID().toString().replace("-", "");
            String redisKey = seckillSkuVo.getPromotionSessionId().toString() + "-" + seckillSkuVo.getSkuId().toString();
            // 幂等性1：判断商品是否存在
            // 幂等性2：信号量也是幂等性，不能重复上架修改信号量
            // 但是代码不要判断信号量是否存在，token每次都是新创建的 key不唯一导致信号量上传多次
            if (!operations.hasKey(redisKey)) {

                //缓存我们商品信息
                SeckillSkuRedisTo redisTo = new SeckillSkuRedisTo();
                Long skuId = seckillSkuVo.getSkuId();
                //1、先查询sku的基本信息，调用远程服务
                R info = productFeignService.getSkuInfo(skuId);
                if (info.getCode() == 0) {
                    SkuInfoVo skuInfo = info.getData("skuInfo",new TypeReference<SkuInfoVo>(){});
                    redisTo.setSkuInfo(skuInfo);
                }

                //2、sku的秒杀信息
                BeanUtils.copyProperties(seckillSkuVo,redisTo);

                //3、设置当前商品的秒杀时间信息
                redisTo.setStartTime(session.getStartTime().getTime());
                redisTo.setEndTime(session.getEndTime().getTime());

                //4、设置商品的随机码（防止恶意攻击）
                redisTo.setRandomCode(token);

                //序列化json格式存入Redis中
                String seckillValue = JSON.toJSONString(redisTo);
                operations.put(seckillSkuVo.getPromotionSessionId().toString() + "-" + seckillSkuVo.getSkuId().toString(),seckillValue);

                //如果当前这个场次的商品库存信息已经上架就不需要上架
                //5、使用库存作为分布式Redisson信号量（限流）
                // 使用库存作为分布式信号量
                RSemaphore semaphore = redissonClient.getSemaphore(SKU_STOCK_SEMAPHORE + token);
                // 商品可以秒杀的数量作为信号量
                semaphore.trySetPermits(seckillSkuVo.getSeckillCount());
            }
        });
    });
}
```

































































# 集群篇

## devops

微服务，服务自治。
DevOps: Development和Operations的组合


* DevOps看作开发(软件工程)、技术运营和质量保障 (QA) 三者的交集。
* 突出重视软件开发人员和运维人员的沟通合作，通过自动化流程来使得软件构建、测试、发布更加快捷、频繁和可靠。
* DevOps希望做到的是软件产品交付过程中IT工具链的打通，使得各个团队减少时间损耗，更加高效地协同工作。专家们总结出了下面这个DevOps能力图，良好的闭环可以大大增加整体的产出。

### CI&CD

#### 持续集成(Continuous Integration)

* 持续集成是指软件个人研发的部分向软件整体部分交付，频繁进行集成以便更快地发现其中的错误。持续集成“源自于极限编程(XP) ，是XP最初的12种实践之一。
* CI需要具备这些:
  * 全面的自动化测试。这是实践持续集成&持续部署的基础，同时，选择合适的自动化测试工具也极其重要;
  * 灵活的基础设施。容器，虚拟机的存在让开发人员和QA人员不必再大费周折;
  * 版本控制工具。如Git, CVS, SVN等;自动化的构建和软件发布流程的工具，如Jenkins, flow.ci;
  * 反馈机制。如构建/测试的失败， 可以快速地反馈到相关负责人，以尽快解决达到一个更稳定的版本。

#### 持续交付(Continuous Delivery)

持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的「类生产环境J(production-like environments)中。持续交付优先于整个产品生命周期的软件部署，建立在高水平自动化持续集成之上。
灰度发布。
持续交付和持续集成的优点非常相似:

* 快速发布。能够应对业务需求，并更快地实现软件价值。
* 编码->测试->.上线->交付的频繁迭代周期缩短，同时获得迅速反馈;

* 高质量的软件发布标准。整个交付过程标准化、可重复、可靠，
* 整个交付过程进度可视化，方便团队人员了解项目成熟度;
* 更先进的团队协作方式。从需求分析、产品的用户体验到交互设计、开发、测试、运维等角色密切协作，相比于传统的瀑布式软件团队，更少浪费。

#### 持续部署(Continuous Deployment)

持续部署是指当交付的代码通过评审之后，自动部署到生产环境中。持续部署是持续交付的最高阶段。这意味着，所有通过了一系列的自动化测试的改动都将自动部署到生产环境。它也可以被称为"Continuous Release"。

“开发人员提交代码，持续集成服务器获取代码，执行单元测试，根据测试结果决定是否部署到预演环境，如果成功部署到预演环境，进行整体验收测试，如果测试通过，自动部署到产品环境，全程自动化高效运转。”

持续部署主要好处是，可以相对独立地部署新的功能，并能快速地收集真实用户的反馈。

## 集群

### 集群的基础形式

#### 主从式

主从复制，同步方式

主从调度，控制方式

#### 分片式

数据分片存储

片区之间备份

#### 选主式

为了容灾选主
为了调度