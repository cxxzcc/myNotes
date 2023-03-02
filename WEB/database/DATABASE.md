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