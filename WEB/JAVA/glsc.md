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

[[Elasticsearch]]















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

1. 添加线程组
2. 添加 HTTP 请求
3. 添加监听器
4. 启动压测&查看



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

## 分布式锁

1.  基于mysql关系型实现
2.  基于redis非关系型数据实现
3.  基于zookeeper/etcd实现



mysql
* 悲观锁
    select ... for update
* 乐观锁乐观读取数据时不锁，更新时检查是否数据已经被更新过，如果是则取消当前更新进行重试。






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

### Redisson


https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95

#### 简介&整合

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

#### Redisson看门狗-解决死锁

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
        //需判断是否是当前线程持有锁
        if(redissonLock.isLocked() && redissonLock.isHeldByCurrentThread()){
            lock.unlock();
        }
    }
    return "hello";
}
```

```java
lock.tryLock()//等待时间内获取锁
lock.getFairLock()//公平锁 有序取锁
```

进入到 `Redisson` Lock 源码

1. 进入 `Lock` 的实现 发现 他调用的也是 `lock` 方法参数  时间为 -1
	![image-20210427204138000](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204138000.png)
2. 再次进入 `lock` 方法
	发现他调用了 tryAcquire
	![image-20210427204156621](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204156621.png)
3. 进入 tryAcquire
	![image-20210427204212298](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204212298.png)
4. 里头调用了 tryAcquireAsync
	这里判断 laseTime != -1 就与刚刚的第一步传入的值有关系
	![image-20210427204243709](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204243709.png)
5. 进入到 `tryLockInnerAsync` 方法
	![image-20210427204300643](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204300643.png)
	锁不存在-加锁 锁存在+1 存在非本线程返回ttl
6. `internalLockLeaseTime` 这个变量是锁的默认时间
	这个变量在构造的时候就赋初始值
	![image-20210427204317552](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204317552.png)
7. 最后查看 `lockWatchdogTimeout` 变量
	也就是30秒的时间
	![image-20210427204330633](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427204330633.png)
8. watchdog
	![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230329150641.png)
	![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230329151015.png)
	![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230329151058.png)
	![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230329151109.png)
9. unlock
	![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230329151158.png)


#### Reidsson-读写锁

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

#### Redisson-闭锁测试

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

#### Redisson-信号量测试

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

#### 多机版

* redlock实现 已弃用 
	五台master实现
* MultiLock 多重锁


```java
@ConfigurationProperties(prefix = "spring.redis", ignoreUnknownFields = false)
@Data
public class RedisProperties {
    private int database;
    /**
     * 等待节点回复命令的时间。该时间从命令发送成功时开始计时
     */
    private int timeout;
    private String password;
    private String mode;
    /**
     * 池配置
     */
    private RedisPoolProperties pool;
    /**
     * 单机信息配置
     */
    private RedisSingleProperties single;
}

@Data
public class RedisSingleProperties {
    private  String address1;
    private  String address2;
    private  String address3;
}
@Data
public class RedisPoolProperties {
    private int maxIdle;
    private int minIdle;
    private int maxActive;
    private int maxWait;
    private int connTimeout;
    private int soTimeout;
    /*
     * 池大小
     */
    private  int size;
}

@Configuration
@EnableConfigurationProperties(RedisProperties.class)
public class CacheConfiguration {

    @Autowired
    RedisProperties redisProperties;

    @Bean
    RedissonClient redissonClient1() {
        Config config = new Config();
        String node = redisProperties.getSingle().getAddress1();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redisProperties.getPool().getConnTimeout())
                .setConnectionPoolSize(redisProperties.getPool().getSize())
                .setConnectionMinimumIdleSize(redisProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redisProperties.getPassword())) {
            serverConfig.setPassword(redisProperties.getPassword());
        }
        return Redisson.create(config);
    }

    @Bean
    RedissonClient redissonClient2() {
        Config config = new Config();
        String node = redisProperties.getSingle().getAddress2();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redisProperties.getPool().getConnTimeout())
                .setConnectionPoolSize(redisProperties.getPool().getSize())
                .setConnectionMinimumIdleSize(redisProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redisProperties.getPassword())) {
            serverConfig.setPassword(redisProperties.getPassword());
        }
        return Redisson.create(config);
    }

    @Bean
    RedissonClient redissonClient3() {
        Config config = new Config();
        String node = redisProperties.getSingle().getAddress3();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redisProperties.getPool().getConnTimeout())
                .setConnectionPoolSize(redisProperties.getPool().getSize())
                .setConnectionMinimumIdleSize(redisProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redisProperties.getPassword())) {
            serverConfig.setPassword(redisProperties.getPassword());
        }
        return Redisson.create(config);
    }
}

@RestController
@Slf4j
public class RedLockController {

    public static final String CACHE_KEY_REDLOCK = "ATGUIGU_REDLOCK";

    @Autowired
    RedissonClient redissonClient1;

    @Autowired
    RedissonClient redissonClient2;

    @Autowired
    RedissonClient redissonClient3;

    boolean isLockBoolean;

    @GetMapping(value = "/multiLock")
    public String getMultiLock() throws InterruptedException
    {
        String uuid =  IdUtil.simpleUUID();
        String uuidValue = uuid+":"+Thread.currentThread().getId();

        RLock lock1 = redissonClient1.getLock(CACHE_KEY_REDLOCK);
        RLock lock2 = redissonClient2.getLock(CACHE_KEY_REDLOCK);
        RLock lock3 = redissonClient3.getLock(CACHE_KEY_REDLOCK);

        RedissonMultiLock redLock = new RedissonMultiLock(lock1, lock2, lock3);
        redLock.lock();
        try
        {
            System.out.println(uuidValue+"\t"+"---come in biz multiLock");
            try { TimeUnit.SECONDS.sleep(30); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(uuidValue+"\t"+"---task is over multiLock");
        } catch (Exception e) {
            e.printStackTrace();
            log.error("multiLock exception ",e);
        } finally {
            redLock.unlock();
            log.info("释放分布式锁成功key:{}", CACHE_KEY_REDLOCK);
        }

        return "multiLock task is over  "+uuidValue;
    }

}
```





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









[[RabbitMQ]]



[[kafka]]





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

8. **队列削峰**
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