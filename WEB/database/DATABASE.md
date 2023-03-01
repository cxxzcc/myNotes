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

```shell
create/alter/drop/rename/truncate
```



## DML数据操作语言

```
insert/delete/update/select
```

```
DESCRIBE 表名   显示表结构
```

### 运算符

```
+-* /(DIV)  %(MOD)

#比较运算符
=  <=> (可以对null判断)  <> != 
IS NULL / IS NOT NULL / ISNULL
GREATEST(最大) LEAST(最小)
in/not in
like
REGEXP/RLIKE 正则表达式

逻辑运算符
not/!
and/&&
or/||
xor 异或 只满足一个
```

### 分页

```
limit a,b
limit b offset a (8.0新)
```









## DCL数据控制语言 

```
commit/rollback/savepoint/grant/revoke
```









## 批量插入

```sql
--只对当前数据库生效，重启失效
show variables like 'log_bin_trust_function_creators';
set global log_bin_trust_function_creators=1;

--随机字符串
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
    DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyZABCDEFJHIJKLMNOPQRSTUVWXYZ';
    DECLARE return_str VARCHAR(255)DEFAULT";
    DECLARE i INT DEFAULT 0;
    WHILEi<nDO
    SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1);
    SETi=i+1;
    END WHILE;
    RETURN return_str;
END $$ I
```

```sql
--用于随机产生部门编号
DELIMITER $$
CREATE FUNCTION rand_num()
RETURNS INT(5)
BEGIN
    DECLARE i INT DEFAULT 0;
    SETi=FLOOR(100+RAND()*10);
RETURN i;
END$$I
--假如要删除
#drop function rand_num;

--批量插入
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
#set autocommit=0把autocommit设置成0
    SET autocommit = 0;
    REPEAT
    SETi=i+1;
    INSERT INTO emp (empno, ename ,job ,mgr ,hiredate ,sal ,comm ,deptno ) VALUES ((START+i)
    ,rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num();
    UNTILi=max_num
END REPEAT;
COMMIT;
END $$
```

```sql
--执行存储过程，往dept表添加随机数据
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
    DECLARE i INT DEFAULT 0;
    SET autocommit =0;
    REPEAT
    SETi=i+1;
    INSERT INTO dept (deptno ,dname,loc ) VALUES ((START+i) ,rand_string(10),rand_string(8);
    UNTILi=max_num
    END REPEAT;
	COMMIT ; 
END $$
                                                  
--调用
DELIMITER ;
CALL insert_dept(100,10); 
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