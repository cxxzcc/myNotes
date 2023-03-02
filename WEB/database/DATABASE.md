关系型数据库
[[MYSQL]]
[[Oracle]] 

* 复杂查询
* 事务

非关系型数据库

* 键值 [[redis]]
* 文档 [[MongoDB]]
* 搜索引擎  [[Elasticsearch]]
* 列式  [[HBase]]     降低系统I/O,适合分布式文件系统
* 图  





# SQL

## DDL 数据定义语言

用来定义数据库对象：数据库，表，列等
```shell
create/alter/drop/rename/truncate
```

## DML数据操作语言

用来对数据库中表的数据进行增删改
```
insert/delete/update/select
```

```
DESCRIBE 表名   显示表结构
delete from 表名; -- 不推荐使用。有多少条记录就会执行多少次删除操作
TRUNCATE TABLE 表名; -- 推荐使用，效率更高 先删除表，然后再创建一张一样的表。
```

## DQL数据查询语言









## DCL数据控制语言

用来定义数据库的访问权限和安全级别，及创建用户
```
commit/rollback/savepoint/grant/revoke
```









# TOOLS

## DataX

https://github.com/alibaba/DataX

阿里巴巴开源的一个异构数据源离线同步工具，致力于实现包括关系型数据库(MySQL、Oracle 等)、HDFS、Hive、ODPS、HBase、FTP 等各种异构数据源之间稳定高效的数据同步功能。 

![image-20211013102649063](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211013102649063.png)

- Reader：Reader为数据采集模块，负责采集数据源的数据，将数据发送给Framework。
- Writer： Writer为数据写入模块，负责不断向Framework取数据，并将数据写入到目的端。
- Framework：Framework用于连接reader和writer，作为两者的数据传输通道，并处理缓冲，流控，并发，数据转换等核心技术问题。

![image-20211013102743353](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211013102743353.png)

Job：单个作业的管理节点，负责数据清理、子任务划分、TaskGroup监控管理。
Task：由Job切分而来，是DataX作业的最小单元，每个Task负责一部分数据的同步工作。
Schedule：将Task组成TaskGroup，单个TaskGroup的并发数量为5。
TaskGroup：负责启动Task。



举例来说，用户提交了一个DataX 作业，并且配置了20 个并发，目的是将一个100 张分表的mysql 数据同步到odps 里面。 DataX 的调度决策思路是：

1. DataXJob 根据分库分表切分成了100 个Task。
2. 根据20 个并发，DataX 计算共需要分配4 个TaskGroup。
3. 4 个TaskGroup 平分切分好的100 个Task，每一个TaskGroup 负责以5 个并发共计运行25 个Task。

### 与 Sqoop 的对比

![image-20211013103257614](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211013103257614.png)


## MyCat
数据库中间件

中间件：连接软件组件和应用的计算机软件，以便于软件各部件之间的沟通
数据库中间件：连接java应用程序和数据库

为什么要用Mycat？
	Java与数据库紧耦合。 
	高访问量高并发对数据库的压力。 
	读写请求数据不一致

① Cobar属于阿里B2B事业群，始于2008年，在阿里服役3年多，接管3000+个MySQL数据库的schema, 集群日处理在线SQL请求50亿次以上。由于Cobar发起人的离职，Cobar停止维护。 
② Mycat是开源社区在阿里cobar基础上进行二次开发，解决了cobar存在的问题，并且加入了许多新 的功能在其中。青出于蓝而胜于蓝。 
③ OneProxy基于MySQL官方的proxy思想利用c进行开发的，OneProxy是一款商业收费的中间件。舍 弃了一些功能，专注在性能和稳定性上。 
④ kingshard由小团队用go语言开发，还需要发展，需要不断完善。 
⑤ Vitess是Youtube生产在使用，架构很复杂。不支持MySQL原生协议，使用需要大量改造成本。 
⑥ Atlas是360团队基于mysql proxy改写，功能还需完善，高并发下不稳定。 
⑦ MaxScale是mariadb（MySQL原作者维护的一个版本） 研发的中间件 
⑧ MySQLRoute是MySQL官方Oracle公司发布的中间件

[http://www.mycat.io/](http://www.mycat.io/)
[http://www.mycat.org.cn/](http://www.mycat.org.cn/)

作用
* 读写分离
* 数据分片 垂直拆分（分库）、水平拆分（分表）、垂直+水平拆分（分库分表）
* 多数据源整合












