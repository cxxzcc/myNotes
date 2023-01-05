关系型数据库    

* 复杂查询
* 事务

非关系型数据库

* 键值 redis
* 文档 MongoDB
* 搜索引擎  es
* 列式  HBase     降低系统I/O,适合分布式文件系统
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

# Oracle

## 架构

![image-20221222111254931](C:\Users\cch\AppData\Roaming\Typora\typora-user-images\image-20221222111254931.png)

### sql执行流程

![image-20221222105711385](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221222105805.png)

Oracle中采用了共享池来判断SQL语句是否存在缓存和执行计划，通过这一步骤我们可以知道应该采用硬解析还是软解析。

1. 语法检查：检查SQL拼写是否正确，如果不正确，Oracle会报语法错误。
2. 语义检查：检查SQL中的访问对象是否存在。比如我们在写SELECT语句的时候，列名写错了，系统就会提示错误。语法检查和语义检查的作用是保证SQL语句没有错误。
3. 权限检查：看用户是否具备访问该数据的权限。
4. 共享池检查：共享池(Shared Pool)是一块内存池，最主要的作用是==缓存SQL语句和该语句的执行计划。==
   Oracle通过检查共享池是否存在SQL语句的执行计划，来判断进行软解析，还是硬解析。那软解析和硬解析又该怎么理解呢？
   在共享池中，Oracle首先对SQL语句进行Hash运算，然后根据Hash值在库缓存(Library Cache)中查找，如果存在SQL语句的执行计划，就直接拿来执行，直接进入“执行器”的环节，这就是软解析。
   如果没有找到SQL语句和执行计划，Oracle就需要创建解析树进行解析，生成执行计划，进入“优化器”这个步骤，这就是硬解析。
5. 优化器：优化器中就是要进行硬解析，也就是决定怎么做，比如创建解析树，生成执行计划。
6. 执行器：当有了解析树和执行计划之后，就知道了SQL该怎么被执行，这样就可以在执行器中执行语句了。

共享池是Oracle中的术语，包括了==库缓存，数据字典缓冲区==等。我们上面已经讲到了库缓存区，它主要缓存SQL语句和执行计划。而数据字典缓冲区存储的是Oracle中的对象定义，比如表、视图、索引等对象。当对SQL语句进行解析的时候，如果需要相关的数据，会从数据字典缓冲区中提取。

库缓存这一个步骤，决定了SQL语句是否需要进行硬解析。为了提升SQL的执行效率，我们应该尽量避免硬解析，因为在SQL的执行过程中，创建解析树，生成执行计划是很消耗资源的。

你可能会问，如何避免硬解析，尽量使用软解析呢？在Oracle中，==绑定变量==是它的一大特色。绑定变量就是在SQL语句中使用变量，通过不同的变量取值来改变SQL的执行结果。这样做的好处是能提升软解析的可能性，不足之处在于可能会导致生成的执行计划不够优化，因此是否需要绑定变量还需要视情况而定。

```sql
SQL>select * from player where player_id 10001;
#使用绑定变量，如：
SQL>select * from player where player_id :player_id;
```

这两个查询语句的效率在Oracle中是完全不同的。如果你在查询player_id=10001之后，还会查询10002、10003之类的数据，那么每一次查询都会创建一个新的查询解析。而第二种方式使用了绑定变量，那么在第一次查询之后，在共享池中就会存在这类查询的执行计划，也就是软解析。
因此，我们可以通过使用==绑定变量来减少硬解析，减少Oracle的解析工作量==。这种方式也有缺点，使用动态SQL的方式，因为参数不同，会导致SQL的执行效率不同，同时SQL优化也会比较困难。

Oracle和MySQL在进行SQL的查询上面有软件实现层面的差异。Oracle提出了共享池的概念，通过共享池来判断是进行软解析，还是硬解析。









# mysql

[官方文档](https://dev.mysql.com/doc/refman/8.0/)



## 架构

### 用户与权限

```sql
CREATE USER 'zhang3' IDENTIFIED BY '123123';#默认host是%
CREATE USER 'kangshifu'@'localhost' IDENTIFIED BY '123456';

UPDATE mysq1.user SET USER='114'WHERE USER='wang5';
FLUSH PRIVILEGES;

DROP USER li4;#默认删除host为%的用户
DROP USER kangshifu'@'localhost';
```

修改自己密码

1. 使用ALTER USER命令来修改当前用户密码
   用户可以使用ALTER命令来修改自身密码，如下语句代表修改当前登录用户的密码。基本语法如下：

   ```sql
   ALTER USER USER() IDENTIFIED BY 'new_password';
   ```

2. 使用SET语句来修改当前用户密码
   使用root用户登录MySQL后，可以使用SET语句来修改密码，具体SQL语句如下：

   ```sql
   SET PASSWORD='new_password';
   ```

修改其他用户密码

1. 使用ALTER语句来修改普通用户的密码
   可以使用ALTER USERT语句来修改普通用户的密码。基本语法形式如下：

   ```sql
   ALTER USER user [IDENTIFIED BY'新密码']
   [,user[IDENTIFIED IBY'新密码']]；
   
   ALTER USER 'kangshifu'@'localhost' IDENTIFIED BY 'HelloWorld_123';
   #其中，user参数表示新用户的账户，由用户名和主机名构成；“IDENTIFIED BY”关键字用来设直密码
   ```


2. 使用SET命令来修改普通用户的密码
      使用root用户登录到MySQL服务器后，可以使用SET语句来修改普通用户的密码。SET语句的代码如下：
      SET 

   ```sql
   PASSWORD FOR 'username'@'hostname' = 'new_password';
   ```

   其中，username参数是普通用户的用户名；hostname参数是普通用户的主机名；new_password是新密码。

#### 权限

```sql
表权限
'Selct','Insert','Update','Delete','Create','Drop','Grant','References','Index','Alter'
列权限
'Select','Insert','Update','References'
过程权限
'Execute,'Alter Routine','Grant'
```

##### 增

```sql
GRANT 权限1，权限2，权限n ON 数据库名称.表名称 TO 用户名@用户地址 [IDENTIFIED BY'密码口令']；
#该权限如果发现没有该用户，则会直接新建一个用户。
#授予通过网络方式登录的joe用户，对所有库所有表的全部权限，密码设为123。注意这里唯独不包括grant的权限
GRANT ALL PRIVILEGES ON *.* To joe@'%' IDENTIFIED BY 123';
ALL PRIVILEGES #是表示所有权限，你也可以使用SELECT、UPDATE等权限。
#如果需要赋予包括GRANTE的权限，添加参数“WITH GRANT OPTION”这个选项即可
```

##### 查

```sql
#查看当前用户权限
SHOW GRANTS;
#或
SHOW GRANTS FOR CURRENT_USER:
#或
SHOW GRANTS FOR CURRENT_USER();
#查看某用户的全局权限
SHOW GRANTS FOR'user'@'主机地址'；
```

##### 删

```sql
#收回权限命令
REVOKE 权限1，权限2，权限n ON 数据库名称.表名称 FROM 用户名@用户地址	
```

##### 权限表

```
user			用户账号及权限信息
global grants	动态全局授权
db				数据库层级的权限
tables priv		表层级的权限
columns priv	列层级的权限
procs priv		存储的过程和函数权限
proxies priv	代理用户的权限
```

#### 角色

```sql
#新建角色
CREATE ROLE role_name'[@host_name'][role_name'[@host_name']]...
#给角色权限
GRANT privileges ON table_name To role_name'[@host_name'];
#查看角色权限
SHOW GRANTS FOR 'role_name'
#删除角色权限
REVOKE privileges ON tablename FROM 'rolename'
#删除角色
DROP ROLE role [role2]...

#给用户赋予角色
#角色创建并授权后，要赋给用户并处于激活状态才能发挥作用。
GRANT role [role2,...]To user [,user2,...];

#显示当前会话的角色
SELECT CURRENT_ROLE();


#激活角色
SET DEFAULT ROLE ALL TO 'kangshifu'@'localhost';
#或
SET GLOBAL activate_all_roles_on_login=ON;

#撤销用户角色
REVOKE role FROM user

#强制角色是给每个创建账户的默认角色，不需要手动设置。强制角色无法被REVOKE或者DROP。
#方式1：服务启动前设置
[mysqld]
mandatory_roles='role1,role2@localhost,r3@%.atguigu.com'
#方式2：运行时设置
SET PERSIST mandatory_-roles='role1,role2@1 ocalhost,r3@%,example.com';#系统重启后仍然有效
SET GLOBAL mandatory-.roles='role1,role2@1 ocalhost,r3@%.example.com';#系统重启后失效
```

#### 配置文件

##### 格式

与在命令行中指定启动选项不同的是，配置文件中的启动选项被划分为若干个组，每个组有一个组名，用中括号[]扩起来，像这样：

```properties
[server]
(具体的启动选项.·.)
[mysqld]
(具体的启动选项.··)
[mysqld_safe]
(具体的启动选项.·)
[client]
(具体的启动选项.·)
[mysql]
(具体的启动选项.·)
[mysqladmin]
(具体的启动选项.··)
```

##### 启动命令与选项组

配置文件中不同的选项组是给不同的启动命令使用的。

```properties
mysqld		启动服务器		[mysqld]、[server]
mysqld_safe	启动服务器		[mysqld]、[server]、[mysq1ld_safe]
mysq1.server 启动服务器		[mysqld]、[server]、[mysql.server]
mysql		启动客户端		[mysql]、[client]
mysqladmin	启动客户端		[mysqladmin][client]
mysqldump	启动客户端		[mysqldump][client]

#同一个配置文件中多个组的优先级 同一个配置最后一个生效
```


比如，在/etc/mysql/my.cnf这个配置文件中添加一些内容：
心英)

### 默认数据库

* mysql
  MySQL系统自带的核心数据库，它存储了MySQL的用户账户和权限信息，一些存储过程、事件的定义信息，
  一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。
* information_schema
  MySQL系统自带的数据库，这个数据库保存着MySQL服务器维护的所有其他数据库的信息，比如有哪些表、哪
  些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时候
  也称之为元数据。在系统数据库information_schema中提供了一些以innodb_sys开头的表，用于表示内
  部系统表。

* performance_schema
  MySQL系统自带的数据库，这个数据库里主要保存MySQL服务器运行过程中的一些状态信息，可以用来监控
  MySQL服务的各类性能指标。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存
  的使用情况等信息。
* sys
  MySQL系统自带的数据库，这个数据库主要是通过视图的形式把information_schema和
  performance_schema结合起来，帮助系统管理员和开发人员监控MySQL的技术性能。

### 逻辑架构

![image-20221221164603217](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20221221164603217.png)

![image-20210914162434653](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210914162434653.png)



**1.连接层**

最上层是一些客户端和连接服务，包含本地sock通信和大多数基于客户端/服务端工具实现的类似于tcp/ip的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

**2.服务层**

第二层架构主要完成大多少的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化及部分内置函数-的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定查询表的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存。如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

* SQL Interface:SQL接▣
  接收用户的SQL命令，并且返回用户需要查询的结果。比如SELECT...FROM就是调用SQL Interface
  MySQL支持DML(数据操作语言)、DDL(数据定义语言)、存储过程、视图、触发器、自定义函数等多种SQL语言接口

* Parser:解析器

  * 在解析器中对SQL语句进行语法分析、语义分析。将SQL语句分解成数据结构，并将这个结构传递到后续步骤，以后SQL语句的传递和处理就是基于这个结构的。如果在分解构成中遇到错误，那么就说明这个SQL语句是不合理的。

  * 在SQL命令传递到解析器的时候会被解析器验证和解析，并为其创建语法树，并根据数据字典丰富查询语法树，会验证该客户端是否具有执行该查询的权限。创建好语法树后，MySQL:还会对SQL查询进行语法上的优化，进行查询重写。

* Optimizer:查询优化器

  * SQL语句在语法解析之后、查询之前会使用查询优化器确定SQL语句的执行路径，生成一个执行计划。

  * 这个执行计划表明应该使用哪些索引进行查询（全表检索还是使用索引检索），表之间的连接顺序如何，最后会按照执行计划中的步骤调用存储引擎提供的方法来真正的执行查询，并将查询结果返回给用户。

  * 它使用“选取-投影-连接”策略进行查询。例如：

    ```sql
    SELECT id,name FROM student WHERE gender =''
    #这个SELECT查询先根据WHERE语句进行选取，而不是将表全部查询出来以后再进行gender过滤：
    #这个SELECT查询先根据id和name进行属性投影，而不是将属性全部取出以后再进行过滤，将这两个查询条件连接起来生成最终查询结果。
    ```

* Caches&Buffers:查询缓存组件

  * MySQL内部维持着一些Cache和Buffer,比如Query Cache用来缓存一条SELECT语句的执行结果，如果能够
    在其中找到对应的查询结果，那么就不必再进行查询解析、优化和执行的整个过程了，直接将结果反馈给客户端。
  * 这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等。
  * 这个查询缓存可以在不同客户端之间共享。
  * 从MySQL5.7.20开始，不推荐使用查询缓存，并在==MySQL8.0中删除==

**3.引擎层**

存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过APl与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取。后面介绍MyISAM和InnoDB

==插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。==

**4.存储层**

数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互、

#### sql执行流程

![image-20221221170933783](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20221221170933783.png)

1. 查询缓存 (命中率低8.0移除)

   ```sql
   #query_.cache_type有3个值g代表关闭查询缓存OFF,1代表开启DN,2(DEMAND) DEMAND,代表当sql语句中有SQL_CACHE关键词时才缓存
   query_cache_type=2
   
   select SQL_CACHE from test where ID=5: #使用缓存  
   select SQL_NO_CACHE from test where ID=5:#不使用
   
   #监控查询缓存的命中率：
   show status like'%Qcache%';
   ```

2. 解析器：在解棉器中对SQL语句进行语法分析、语义分析。生成语法树

   词法分析

   语法分析

3. 优化器：在优化器中会确定SQL语句的执行路径，比如是根据全表检索，还是根据索引检索等。

   可以分为逻辑查询优化阶段和物理查询优化阶段

   * 物理查询优化则是通过索引和表连接方式等技术来进行优化，这里重点需要掌握索引的使用。
   * 逻辑查询优化就是通过SQL等价变换提升查询效率，直白一点就是说，换一种查询写法执行效率可能更高。

4. 执行器

#### sql执行原理


```SQL
# 查看是否开启计划。开启它可以让MySQL收集在SQL执行时所使用的资源情况
select @@profiling;
show variables like 'profiling';

#profiling=o代表关闭，我们需要把profiling打开，即设置为1：
set profiling=1;I
#Profiling功能由MySQL会话变量：profiling控制。默认是oFF(关闭状态)。


#查看执行过程
show profiles
show profile for [num]

show profile cpu,block io for query 7;

Syntax:
SHOW PROFILE [type [, type]...
[FOR QUERY n]
[LIMIT row_count [OFFSET offset]]

type:{
ALL --一显示所有参数的开销信息
|BLOCK IO --显示IO的相关开销
|CONTEXT SWITCHES--上下文切换相关开销
|CPU--显示CPU相关开销信息
|IPC--显示发送和接收相关开销
|MEMORY--显示内存相关开销信息
| PAGE FAULTS--显示页面错误相关开销信息
|SOURCE--显示和Source_function,Source_file,Source_line相关的开销信息
|SWAPS--显示交换次数相关的开销信息
}
```

### 数据库缓冲池(buffer pool)

InnoDB存储擎是以页为单位来管理存储空间, DBMS申请占用内存来作为数据缓冲池，在真正访问页面之前，需要把在磁盘上的页缓存到内存中的Buffer Pool之后才可以访问。

这样做的好处是可以让磁盘活动最小化，从而减少与磁盘直接进行I/O的时间。

![image-20221222112041166](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221222112102.png)

缓存池的重要性：
InnoDB以页的形式存储。
loDB存储引擎在处理客户端的请求时，当需要访问某个页的数据时，就会把完整的页的数据全部加载到内存中，
也就是说即使我们只需要访问一个页的一条记录，那也需要先把整个页的数据加载到内存中。将整个页加载到内
存中后就可以进行读写访问了，在进行完读写访问之后并不着急把该页对应的内存空间释放掉，而是将其缓存起
来，这样将来有请求再次访问该页面时，就可以省去磁盘IO的开销了。

缓存原则:
“位置*频次”这个原则，可以帮我们对I/O访问效率进行优化。 优先对使用频次高的热数据进行加载

缓冲池的预读特性:
预读
缓冲池的作用就是提升1/O效率，而我们进行读取数据的时候存在一个“局部性原理”， 也就是说我们使用了- -些
数据，大概率还会使用它周围的一些数据 ，因此采用“预读’ 的机制提前加载，可以减少未来可能的磁盘1/0操作。

#### 原理

![image-20221223110629681](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221223110649.png)

如果我们执行SQL语句的时候更新了缓存池中的数据，那么这些数据会马上同步到磁盘上吗?
修改时，先改缓冲池的数据，然后数据库会以==一定的频率刷新到磁盘上==。不是每次更新，都会立刻进行磁盘回写。

缓冲池会采用一种叫做==checkpoint==的机制将数据回写到磁盘上，这样做的好处就是提升了数据库的整体性能。

比如，当缓冲池不够用时，需要释放掉一些不常用的页， 此时就可以强行采用checkpoint的方式，将不常用的脏页回写到磁盘上，然后再从缓冲池中将这些页释放掉。这里脏页(dirty page)指的是缓冲池中被修改过的页，与磁盘上的数据页不-致。

#### 查看/设置

```sql
#MyISAM缓存索引，不缓存数据
key_ buffer_size，
#InnoDB
innodb_buffer_pool_sife 

show variables like 'innodb_ buffer_ pool_ size';
set global innodb_ buffer_ pool_ size = 268435456 ;

```

#### 多实例

Buffer Pool本质是InnoDB向操作系统申请的一块连续的内存空间,在多线程环境下，访问Buffer Pool中的数据都需要加锁处理。在Buffer Pool特别大而且多线程并发访问特别高的情况下，单Buffer Pool可能会影响请求的处理速度。所以在Buffer Fpol特别大的时候，我们可以把它们拆分成若干个小的Buffer Pool ，每个Buffer Pool都称为一个实例，它们都是独立的，独立的去申请内存空间，独立的管理各种链表。所以在多线程并发访问时并不会相互影响，从而提高并发处理能力。

我们可以在服务器启动的时候通过设置innodb_ buffer pool_ instances的值来修改Buffer Pool实例的个数，

```properties
[server]
innodb_ buffer_ pool _ instances = 2
```

不过也不是说Buffer Pool实例创建的越多越好，分别管理各个Buffer Pool也 是需要性能开销的，InnoDB规定:
当innodb_ buffer._pool size的值小于1G的时候设置多个实例是无效的，InnoDB会默认把innodb_ buffer pool instances 的值修改为1。而我们鼓励在Buffer Pool大于或等于1G的时候设置多个Buffer Pool实例。

### 存储引擎

==存储引擎就是指表的类型==

1. 如果表b采用InnoDB,data\a中会产生1个或者2个文件：
   * b.frm:描述表结构文件，字段长度等
   * 如果采用系统表空间模式的，数据信息和索引信息都存储在ibdata1中
   * 如果采用独立表空间存储模式，data a中还会产生b.ibd文件（存储数据信息和索引信息）
     此外：
     1. MySQL5.7中会在data/a的目录下生成db.opt文件用于保存数据库的相关配置。比如：字符集、比较规则。而MySQL8.0不再提供db.opt文件。
     2. MySQL8.0中不再单独提供b.frm,而是合并在b.ibd文件中。

2. 如果表b采用MyISAM,data\a中会产生3个文件：
   * MySQL5.7中：b.frm:描述表结构文件，字段长度等。MySQL8.0中b.xXx.sdi:描述表结构文件，字段长度等
   * b.MYD(MYData):数据信息文件，存储数据信息（如果采用独立表存储模式）        
   * b.MYI(MYIndex):存放索引信息文件

```sql
show engines
```

![image-20221226100713317](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221226100728.png)

```sql
#查看默认的存储引擎:
show variables like '%storage_engine%';
#或
SELECT @@default_ storage_ engine;

#修改
SET DEFAULT STORAGE_ ENGINE=My]JSAM; 
#或者修改my.cnf文件:
default-stor age- engine=MyISAM 
#重启服务
systemctl restart mysqld. service

#改表
alter table emp4 engine = myisam;
```

|          | myisam                     | innodb                                                       |
| -------- | -------------------------- | ------------------------------------------------------------ |
| 主外键   | 不支持                     | 支持                                                         |
| 事务     | 不支持                     | 支持                                                         |
| 行表锁   | 表锁                       | 行锁   适合高并发的操作                                      |
| 缓存     | 只缓存索引，不缓存真实数据 | 缓存索引和真实数据对内存要求较高，而且内存大小对性能有决定性的影响 |
| 表空间   | 小                         | 大                                                           |
| 关注点   | 性能                       | 事务                                                         |
| 默认安装 | y                          | y                                                            |

#### 引擎种类

##### InnoDB

具备外键支持功能的事务存储引擎

* MySQL从3.23.34a开始就包含InnoDB存储引擎。大于 等于5.5之后，默认采用InnoDB引擎。
* InnoDB是MySQL的==默认事务型引擎==，它被设计用来处理大量的短期(short-lived)事务。 可以确保事务的完整提交(Commit)和回滚(Rttback)。
* 除了增加和查询外，还需要更新、删除操作，那么，应优先选择InnoDB存储弓|擎。
* 除非有非常特别的原因需要使用其他的存储引擎，否则应该优先考虑InnoDB引擎。
* 数据文件结构: (在《第02章MySQL数据目录》章节已讲)
  * 表名.frm存储表结构(MySQL8.0时， 合并在表名.ibd中)
  * 表名.ibd存储数据和索引
* InnoDB是==为处理巨大数据量的最大性能设计。==
  。在以前的版本中，字典数据以元数据文件、非事务表等来存储。现在这些元数据文件被删除了。比
  如: .frm， .par, .trn， .isl, . db. opt等都在MySQL8.0中不存在了。
* 对比MyISAM的存储引擎，InnoDB==写的处理效率差一 些== ，并且会占用更多的磁盘空间以保存数据和索引。
* MyISAM只缓存索引，不缓存真实数据; InnoDB不仅缓存索引还要缓存真实数据，==对内存要求较高==,而且内存大小对性能有决定性的影响。

##### MyISAM

主要的非事务处理存储引擎

* MyISAM提供了大量的特性，包括全文索引、压缩、空间函数(GIS)等，但MyISAM==不支持事务、行级锁、外键,==
  有一个毫无疑问的缺陷就是崩溃后无法安全恢复。
* 5.5之前默认的存储引擎
* 优势是==访问的速度快==,对事务完整性没有要求或者以SELECT、INSERT为主的应用
* 针对数据统计有额外的常数存储。故而count(*)的查询效率很高
* 数据文件结构: (在《第02章MySQL数据目录》章节已讲)
  * 表名.frm存储表结构
  * 表名.MYD存储数据(MYData)
  * 表名.MYI存储索引(MYIndex)
* 应用场景:只读应用或者以读为主的业务

##### Archive

用于数据存档

* archive是==归档==的意思，仅仅支持==插入和查询==两种功能(行被插入后不能再修改)。
* 在MySQL5.5以后 支持索引功能。
* 拥有很好的压缩机制，使用==zlib压缩库==，在记录请求的时候实时的进行压缩，经常被用来作为仓库使用。
* 创建ARCHIVE表时，存储引擎会创建名称以表名开头的文件。数据文件的扩展名为. ARZ。
* 根据英文的测试结论来看，同样数据量下，Archive表比MyISAM表要小大约75%， 比支持事务处理的InnoDB表
  小大约83%。
* ARCHIVE存储引擎采用了==行级锁==。该ARCHIVE引擎支持AUTO_INCREMENT列属性。AUTO_ INCREMENT列可以具有唯一索引或非唯一 索引。尝试在任何其他列上创建索引会导致错误。
* Archive表适合日志和数据采集(档案)类应用;适合存储大量的独立的作为历史记录的数据。拥有很高的插入速度，但是对查询的支持较差。

##### Blackhole

丢弃写操作，读操作会返回空内容

* Blackhole弓|擎没有实现任何存储机制，它会丢弃所有插入的数据，不做任何保存。
* 但服务器会记录Blackhole表的日志，所以可以用于复制数据到备库，或者简单地记录到日志。但这种应用方式会碰到很多问题，因此并不推荐。

##### CSV

以逗号分隔各个数据项

* CSV引擎可以将普通的CSV文件作为MySQL的表来处理，但不支持索引。
* CSV引擎可以作为一种数据交换的机制,非常有用。
* CSV存储的数据直接可以在操作系统里，用文本编辑器，或者excel读取。
* 对于数据的快速导入、导出是有明显优势的。

##### Memory

置于内存的表
Memory采用的逻辑介质是内存，响应速 度很快，但是当mysqld守护进程崩溃的时候==数据会丢失==。另外，要求存储的数据是数据长度不变的格式，比如，Blob和Text类型的数据不可用(长度不固定的)。 

* Memory同时支持==哈希(HASH) 索引和B+树索引。==
  * 哈希索引相等的比较快，但是对于范围的比较慢很多。
  * ==默认使用哈希(HASH)索引==，其速度要比使用B型树(BTREE) 索引快。
  * 如果希望使用B树索引，可以在创建索弓时选择使用。
* Memory表至少比MyISAM表要 快一个 数量级。
* MEMORY表的==大小是受到限制的。==表的大小主要取决于两个参数，分别是max_ rows和max_ heap. table_ size。其中，max_ rows可以在创建表时指定; max_ heap_ table_ size的大小默认为16MB,可以按需要进行扩大。
* 数据文件与索引文件分开存储。
  * 每个基于MEMORY存储弓|擎的表实际对应一个磁盘文件，该文件的文件名与表名相同，类型为frm 类型,该文件中只存储表的结构，而其==数据文件都是存储在内存中的。==
  * 这样有利于数据的快速处理，提供整个表的处理效率。
* 缺点:其数据易丢失，生命周期短。基于这个缺陷，选择MEMORY存储弓|擎时需要特别小心。

使用Memory存储引擎的场景:
1. 目标数据比较小，而且非常频繁的进行访问，在内存中存放数据，如果太大的数据会造成内存溢出。可以通过参数max_ heap_ table_ size 控制Memory表的大小，限制Memory表的最 大的大小。
2. 如果数据是临时的，而且必须立即可用得到，那么就可以放在内存中。
3. 存储在Memory表中的数据如果突然间丢失的话也没有太大的关系。

##### Federated

访问远程表
Federated引擎是访问其他MySQL服务器的一=一个代理,尽管该引擎看起来提供了一种很好的 跨服务器的灵活性，但也经常带来问题，因此默认是禁用的。

##### Menige

管理多个MyISAM表构成的表集合

##### NDB

MySQL 集群专用存储引擎
也叫做NDB Cluster存储引擎，主要用于==MySQL Cluster==分布式集群环境，类似于Oracle的RAC集群。



## 索引和调优

索引(Index) 是帮助MySQL高效获取数据的数据结构。

索引是在存储引擎中实现的，因此每种存储弓擎的索引不一定完全相同， 并且每种存储引擎不一定支持所有索引类型。同时，存储引擎可以定义每个表的最大索引数和最大索引长度。所有存储引擎支持每个表至少16个索引，总索引长度至少为256字节。有些存储引擎支持更多的索引数和更大的索引长度。

**优点**

1. 提高数据检索的效率，降低数据库的I0成本
2. 唯一索引， 可以保证数据库表中每一行数据的唯一性。
3. 在实现数据的参考完整性方面，可以加速表和表之间的连接。换句话说，对于有依赖关系的子表和父表联合查询时，可以提高查询速度。
4. 在使用分组和排序子句进行数据查询时，可以显著减少查询中分组和排序的时间，降低了CPU的消耗。

**缺点**

1. 创建索引和维护索引要耗费时间，并且随着数据量的增加，所耗费的时间也会增加。
2. 索引需要占磁盘空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，存储在磁盘 上，如果有大量的索引，索引文件就可能比数据文件更快达到最大文件尺寸。
3. 虽然索引大大提高了查询速度，同时却会降低更新表的速度。当对表中的数据进行增加、删除和修改的时候，索引也要动态地维护，这样就降低了数据的维护速度。

索引可以提高查询的速度，但是会影响插入记录的速度。这种情况下，最好的办法是先删除表中的索引，然后插入数据，插入完成后再创建索引。
### 索引
#### 分类
MySQL的索引包括普通索引、唯一性索引、 全文索引、单列索引、多列索引和空间索引等。
* 从功能逻辑上说，索引主要有4种，分别是普通索引、唯一索引、主键索引、全文索引。
* 按照物理实现方式，索引可以分为2种:聚簇索引和非聚簇索引。
* 按照作用字段个数进行划分，分成单列索引和联合索引。

1. 普通索引
	不附加任何限制条件
	
2. 唯一索引
	使用UNIQUE参数设置索引
	
3. 主键索引
	NOT NULL+UNIQUE, 表里最多只有一个主键索引。
	这是由主键索弓|的物理实现方式决定的，因为数据存储在文件中只能按照一种顺序进行存储。
	
4. 单列索引
	在表中的单个字段上创建索引
	
5. 联合索引
	多个字段组合，创建一个索引。遵循最左前缀
	
6. 全文索引
	全文索引(也称全文检索)是目前搜索引擎使用的一种关键技术。它能够利用==分词技术==等多种算法智能分析出文本文字中关键词的频率和重要性，然后按照一定的算法规则智能地筛选出我们想要的搜索结果。全文索引非常适合大型数据集，对于小的数据集，它的用处比较小。
	使用参数==FULLTEXT==可以设置索引为全文索引。在定义索引的列上支持值的全文查找，允许在这些索引列中插入重复值和空值。全文索引只能创建在CHAR、VARCHAR 或TEXT类型及其系列类型的字段上，查询数据量较大的字符串类型的字段时，使用全文索引可以提高查询速度。例如，表student的字段information是TEXT类型，该字段包含了很多文字信息。在字段information. 上建立全文索引后，可以提高查询字段information的速度。
	全文索引典型的有两种类型:自然语言的全文索引和布尔全文索引。
	随着大数据时代的到来，关系型数据库应对全文索弓的需求已力不从心，逐渐被solr、ElasticSearch 等专门的搜索引擎所替代。
	
	```sql
	--全文索引用match+against方式查询 比like快 精度可能有问题
	SELECT * FROM papers WHERE MATCH( title，content) AGAINST (' 查询字符串');
	
	```
	
	
	
7. 空间索引
	使用参数SPATIAL可以设置索引为空间索引。空间索引只能建立在空间数据类型上，这样可以提高系统获取空间数据的效率。MySQL中的空间数据类型包括 GEOMETRY, POINT、LINESTRING 和POLYGON等。目前只有MyISAM存储弓擎支持空间检索，而且索引的字段不能为空值。

小结:不同的存储引擎支持的索引类型也不-样
InnoDB:支持B-tree、Full-text 等索引，不支持Hash索引;
MyISAM :支持B-trpe、Flltext 等索引，不支持Hash索引;
Memory :支持B-tree、Hash 等索引，不支持Full-text索引;
NDB :支持Hash索引，不支持B-tree、Full-text 等索引; 
Archive :不支持B-tree、Hash、 Full-text 等索引;

#### 操作
```sql
--1.create table
CREATE TABLE table_name [col_name data_type]
[UNIQUE | FULLTEXT | SPATIAL] [INDEX | KEY] [index_name] (col_name [length]) [ASC | DESC]

--UNIQUE、 FULLTEXT 和SPATIAL为可选参数，分别表示唯一索引、全文索引和空间索引，
--INDEX与KEY为同义词，两者的作用相同，用来指定创建索引;
--index_name 指定索引的名称，为可选参数，如果不指定，那么MySQL默认coL name为索引名;
--col name为需要创建索引的字段列，该列必须从数据表中定义的多个列中选择;
--length 为可选参数，表示索引的长度，只有字符串类型的字段才能指定索引长度;
--ASC 或DESC指定升序或者降序的索引值存储。

--2
ALTER TABLE table_name ADD [UNIQUE | FULLTEXT | SPATIAL] [INDEX | KEY]
[index_name] (col_name[length],...) [ASC | DESC]
--3
CREATE [ UNIQUE | FULLTEXT | SPATIAL ] INDEX index_ name
ON table_name (col_name[length],...) [ASC | DESC]

--查看
SHOW CREATE TABLE bopk; 
SHOW INDEX FROM book;

--删除
ALTER TABLE table_name DROP INDEX index_name
DROP INDEX index_name ON table_name ;
```

#### 8.0新
##### 支持降序索引
降序索引以降序存储键值。虽然在语法上，从MySQL 4版本开始就经支持降序索弓|的语法了，但实际上该DESC定义是被忽略的，直到MySQL 8.x版本才开始真正支持降序索引(仅限于InnoDB存储引擎)。

MySQL在8.0版本之前创建的仍然是升序索引，使用时进行反向扫描，这大大降低了数据库的效率。在某些场景下，降序索引意义重大。例如，如果一个查询， 需要对多个列进行排序，且顺序要求不一致， 那么使用降序索引将会避免数据库使用额外的文件排序操作，从而提高性能。

##### 隐藏索引
在MySQL 5.7版本及之前，只能通过显式的方式删除索引。此时，如果发现删除索引后出现错误，又只能通过显式创建索引的方式将删除的索引创建回来。如果数据表中的数据量非常大，或者数据表本身比较大，这种操作就会消耗系统过多的资源，操作成本非常高。

从MySQL 8.x开始支持隐藏索引(invisible indexes) ，只需要将待删除的索引设置为隐藏索引，使查询优化器不再使用这个索引(即使使用force index (强制使用索引) ,优化器也不会使用该索引)， 确认将索弓|设置为隐藏索引后系统不受任何响应，就可以彻底删除索引。这 种通过先将索引设置为隐藏索引，再删除索引的方式就是软删除。
同时，如果你想验证某个索引|删除之后的查询性能影响，就可以暂时先隐藏该索引。
主键不能被设置为隐藏索引。

索引默认是可见的，在使用CREATE TABLE, CREATE INDEX或者ALTER TABLE等语句时可以通过VISIBLE或者INVISIBLE关键词设置索弓的可见性。
```sql
ALTER TABLE tablename ALTER INDEX index_ name INVISIBLE; #切换成隐藏索引
ALTER TABLE tablename ALTER INDEX index_ name VISIBLE; #切换成非隐藏索引

--使隐藏索引对查询优化器可见
--默认关闭
select @@optimizer_switch \G
set session optimizer_switch="use_invisible_indexes=on"
```
当索引|被隐藏时，它的内容仍然是和正常索引-样实时更新的。不用可删除
#### 设计原则
##### 适合

1. 字段有唯一性限制
2. 频繁作为WHERE条件的字段
3. 经常GROUP BY和ORDER BY的列
4. UPDATE、 DELETE的WHERE条件列
5. .DISTINCT字段需要创建索引
6. 多表JOIN连接操作时，创建索引注意事项
  连接表的数量尽量不要超过3
  对WHERE 条件创建索引
  对用于连接的字段创建索引 且数据类型一致
7. 使用列的类型小的创建索引
8. 使用字符串前缀创建索引
   ```sql
   select count( distinct address) / count(*) from shop;
   通过不同长度去计算，与全表的选择性对比: 90%以上即可
   公式:
   count(distinct 1eft(列名， 索引长度))/count(*)
   例如:
   select count(distinct left(addrless, 10)) / count(*) as sub10, --截取前10个字符的选择度
   count(distinct left(address, 15)) / count(*) as sub11， --截取前15个字符的选择度
   count(distinct left(address,20)) / count(*) as sub12, --截取前20个字符的选择度
   count( distinct left(address,25)) / count(*) as sub13 --截取前25个字符的选择度
   from shop;
   ```
9. 区分度高(散列性高)的列适合作为索引
	可以使用公式select count(distinct a)/count() from t1计算区分度，越接近1越好，般超过 33% 就算是比较高效的索引了。
10. 使用最频繁的列放到联合索引的左侧
11. 在多个字段都要创建索引的情况下，联合索引优于单值索引

**限制索引数目**
单表不超过6个

##### 不合适
1. where使用不到
2. 数据量小
3. 重复太多的列
4. 避免对经常更新的表创建过多的索引
5. 无序的列
6. 删除不再使用或者很少使用的索引
7. 不要定义冗余或重复的索引

#### 失效

 1. 全值匹配
 2. 最佳左前缀 
 3. 主键有序
 4. 计算、函数、类型转换(自动或手动)导致索引失效
 5. 范围条件右边索引列的失效
 6. != <>
 7. is null / is not null
 8. like %左边
 9. OR前后存在非索引的列，索引失效
 10. 字符集不一致

### InnodB

索引按照物理实现方式，索引可以分为2种:聚簇(聚集)和非聚簇(非聚集)索引。我们也把非聚集索引称为二级索引或者辅助索引。

#### 聚簇索引

聚簇索引并不是一种单独的索引类型，而是一种数据存储方式(所有的用户记录都存储在了叶子节点) ,也就是所谓的索引即数据，数据即索引。

特点

1. 使用记录主键值的大小进行记录和页的排序，这包括三个方面的含义:
   * 页内的记录是按照主键的大小顺序排成一个单向链表。
   * 各个存放用户记录的页也是根据页中用户记录的主键大小顺序排成一 个双向链表。
   * 存放目录项记录的页分为不同的层次，在同一层次中的页也是根据页中目录项记录的主键大小顺序排成一个双向链表。

2. B+树的 叶子节点存储的是完整的用户记录。
所谓完整的用户记录，就是指这个记录中存储了所有列的值(包括隐藏列)。

具有这两种特性的B+树称为聚簇索引,所有完整的用户记录都存放在这个聚簇索引的叶子节点处。这种聚簇索弓|并不需要我们在MySQL语句中显式的使用INDEX语句去创建，InnoDB 存储引擎会自动的为我们创建聚簇索引。

**优点:**

* 数据访问更快，因为聚簇索引|将索引和数据保存在同一个B+树中，因此从聚簇索引中获取数据比非聚簇索引更快
* 聚簇索引对于主键的排序查找和范围查找速度非常快
* 按照聚簇索引排列顺序，查询显示一定范围数据的时候，由于数据都是紧密相连，数据库不用从多个数据块中提取数据，所以节省了大量的io操作。

**缺点**:

* 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式，否则将会出现页分裂，严重影响性能。因此，对于InnoDB表，我们一般都会定义-个自增的ID列为主键
* 更新主键的代价很高，因为将会导致被更新的行移动。因此，对于InnoDB表，我们一般定义主键为不可更新
* 二级索引访问需要两次索引查找，第-次找到主键值，第二次根据主键值找到行数据

限制:

* 对于MySQL数据库目前只有InnoDB支持聚簇索引，而MyISAM并不支持聚簇索引。
* 由于数据物理存储排序方式只能有一种，所以每个MySQL的表只能有一个聚簇索引。一般情况下就是该表的主键。
* 如果没有定义主键，Innodb会选择非空的唯一 索引代替。如果没有这样的索引，Innodb会隐式的定义-一个主键来作为聚簇索引。
* 为了充分利用聚簇索引的聚簇的特性，所以innodb表的主键列尽量选用有序的顺序id，而不建议用无序的id,比如UUID、MD5、HASH、 字符串列作为主键无法保证数据的顺序增长。

#### 二级索引 (辅助索引、 非聚簇索引)

回表
先查二级索引再查聚簇索引

1. 聚簇索引的叶子节点存储的就是我们的数据记录，非聚簇索引的叶子节点存储的是数据位置。非聚簇索引不会影响数据表的物理存储顺序。

2. -个表只能有一个聚簇索引，因为只能有一种排序存储的方式，但可以有多个非聚簇索引，也就是多个索引目录提供数据检索。
3. 使用聚簇索引的时候，数据的查询效率高，但如果对数据进行插入，删除，更新等操作，效率会比非聚簇索引低。

##### 联合索引



#### InnoDB的B+树索引的注意事项

1. 根页面位置万年不动

   B+树的形成过程是这样的:

   * 每当为某个表创建一 个B+树索引(聚簇索引不是人为创建的，默认就有)的时候，都会为这个索引创建一 个根节点页面。最开始表中没有数据的时候，每个B+树索引|对应的根节点中既没有用户记录，也没有目录项记录。

   * 随后向表中插入用户记录时，先把用户记录存储到这个根节点中。

   * 当根节点中的可用空间用完时继续插入记录，此时会将根节点中的所有记录复制到一一个新分配的页，比如页a中，然后对这个新页进行==页分裂==的操作，得到另一 个新页，比如页b。这时新插入的记录根据键值(也就是聚簇索引|中的主键值，二级索引中对应的索引列的值)的大小就会被分配到页a或者页b中，而根节点便升级为存储目录项记录的页。

     这个过程特别注意的是:一个B+树索引的根节点自诞生之日起，便不会再移动。这样只要我们对某个表建立一个索引，那么它的根节点的页号便会被记录到某个地方，然后凡是InnoDB存储引擎需要用到这个索引的时候，都会从那个固定的地方取出根节点的页号，从而来访问这个索引。

2. 内节点中目录项记录的唯一性

   二级索引可能有重复值, 内节点会带主键存储

3. 一个页面最少存储2条记录

#### 存储结构

==页16kb   磁盘和内存之间交互的基本单位==

```sql
show variables like' %innodb_ page_ size%' ;
```

SQL Server中页的大小为8KB，而在Oracle中我们用术语“块”(Block) 来代表“页”，Oralce 支持的块大小为2KB，4KB， 8KB， 16KB， 32KB 和64KB。

##### 页的上层结构

![](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221226164848.png)

区(Extent) 是比页大-级的存储结构，在InnoDB存储引擎中，-个区会分配==64个连续的页==。因为InnoDB中的页大小默认是16KB,所以一个区的大小是64*16KB= 1MB。

段(Segment) 由一个或多个区组成，区在文件系统是一个连续分配的空间(在InnoDB中是连续的64个页) ,不过在段中不要求区与区之间是相邻的。==段是 数据库中的分配单位，不同类型的数据库对象以不同的段形式存在。==当我们创建数据表、索引的时候，就会相应创建对应的段，比如创建一张表时会创建一个表段， 创建一个索引时会创建一个索引段。

表空间(Tablespace) 是一个逻辑容器， 表空间存储的对象是段，在一个表空间中可以有一 个或多个段，但是一个段只能属于一个表空间。数据库由一个或多个表空间组成，表空间从管理上可以划分为==系统表空间、用户表空间、撤销表空间、临时表空间==等。

###### 区
B+树的每一层中的页都会形成一个双向链表， 如果是以页为单位来分配存储空间的话，双向链表相邻的两个页之间的物理位置可能离得非常远。我们介绍B+树索引|的适用场景的时候特别提到范围查询只需要定位到最左边的记录和最右边的记录，然后沿着双向链表一直扫描就可以了，而如果链表中相邻的两个页物理位置离得非常远，就是所谓的随机I/0。再一次强调，磁盘的速度和内存的速度差了好几个数量级，随机I/0是 非常慢的，所以我们应该尽量让链表中相邻的页的物理位置也相邻，这样进行范围查询的时候才可以使用所谓的顺序I/0。

引入区的概念，一个区就是在物理位置上连续的64个页。因为InnoDB中的页大小默认是16KB，所以一个区的大小是64* 16KB=1MB。在表中数据量大的时候，为某个索引分配空间的时候就不再按照页为单位分配了，而是按照区为单位分配，甚至在表中的数据特别多的时候，可以-次性分配多个连续的区。虽然可能造成- 点点空间的浪费(数据不足以填充满整个区) ,但是从性能角度看，可以消除很多的随机/O，功大于过!

区大体上可以分为4种类型:
* 空闲的区(FREE) :现在还没有用到这个区中的任何页面。
* 有剩余空间的碎片区(FREE FRAG) :表示碎片区中还有 可用的页面。
* 没有剩余空间的碎片区(FULL FRAG) :表示碎片区中的所有页面都被使用，没有空闲页面。
* 附属于某个段的区(FSEG):每一个索引都可以分为叶子节点段和非叶子节点段。
处于FREE、FREE FRAG 以及FULL_FRAG这三种状态的区都是独立的，直属于表空间。而处于FSEG状态的区是附属于某个段的



###### 段
对于范围查询，其实是对B+树叶子节点中的记录进行顺序扫描，而如果不区分叶子节点和非叶子节点，统统把节点代表的页面放到申请到的区中的话，进行范围扫描的效果就大打折扣了。所以InnoDB对B+树的叶子节点和非叶子节点进行了区别对待，也就是说叶子节点有自己独有的区，非叶子节点也有自己独有的区。存放叶子节点的区的集合就算是一个段( segment)，存放非叶子节点的区的集合也算是一个段。 也就是说一个索引会生成2个段，一个叶子节点段，-个非叶子节点段。

除了索引的叶子节点段和非叶子节点段之外，InnoDB中还有为存储一一些特殊的数据而定义的段， 比如回滚段。所以，常见的段有数据段、索引段、回滚段。 

数据段即为B+树的叶子节点，索引段即为B+树的非叶子节点。
在InnoDB存储引擎中，对段的管理都是由引|擎自身所完成，DBA不能也没有必要对其进行控制。这从-定程度上简化了DBA对于段的管理。
段其实不对应表空间中某一个连续的物理区域，而是一个逻辑上的概念，由若千个零散的页面以及-一些完整的区组成。
###### 碎片区
为了考虑以完整的区为单位分配给某个段对于数据量较小的表太浪费存储空间的这种情况，InnoDB提出了一个碎片(fragment)区的概念。在一个碎片区中，并不是所有的页都是为了存储同一个段的数据而存在的，而是碎片区中的页可以用于不同的目的，比如有些页用于段A,有些页用于段B，有些页甚至哪个段都不属于。碎片区直属于表空间，并不属于任何一个段。
* 在刚开始向表中插入数据的时候，段是从某个碎片区以单个页面为单位来分配存储空间的。
* 当某个段已经占用了32个碎片区页面之后，就会申请以完整的区为单位来分配存储空间。

###### 表空间
表空间可以看做是InnoDB存储弓擎逻辑结构的最高层，所有的数据都存放在表空间中。
表空间是一个逻辑容器，表空间存储的对象是段，在一个表空间中可以有一个或多个段，但是一个段只能属于一个表空间。表空间数据库由一个或多个表空间组成，表空间从管理上可以划分为系统表空间(System
tablespace)、独立表空间(File-per-table tablespace)、撤销表 空间(Undo Tablespace)和临时表空间(Temporary Tablespace)等。

1. 独立表空间
	独立表空间，即每张表有一个独立的表空间，也就是数据和索引|信息都会保存在自己的表空间中。独立的表空间(即:单表)可以在不同的数据库之间进行迁移。

	空间可以回收(DROP TABLE操作可自动回收表空间;其他情况，表空间不能自己回收)。如果对于统计分析或是日志表，删除大量数据后可以通过: alter table TableName engine=innodb; 回收不用的空间。对于使用独立表空间的表，不管怎么删除，表空间的碎片不会太严重的影响性能，而且还有机会处理。

	一个新建的表对应的. ibd文件只占用了96K，才6个页面大小(MySQL5.7中) ,因为一开始表空间占用的空间很小，因为表里边都没有数据。不过别忘了这些.ibd文件是自扩展的，随着表中数据的增多，表空间对应的文件也逐渐增大。
	```sql
	show variables like 'innodb_file_per_.table';
	```
1. 系统表空间
	结构和独立表空间基本类似，只不过由于整个MySQL进程只有一个系统表空间， 在系统表空间中会额外记录一些有关整个系统信息的页面
	![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227150652.png)
	这些系统表也被称为数据字典，它们都是以B+树的形式保存在系统表空间的某些页面中，其中sYS. _TABLES、SYS_ COLUMNS、SYS. INDEXES、SYS. FIELDS这四个表尤其重要，称之为基本系统表(basic system tables)



##### 页的内部结构

常见的有数据页(保存B+树节点)、 系统页、Undo页和事务数据页等

数据页的16KB大小的存储空间被划分为七个部分，分别是文件头(File Header)、页头(Page Header)、最大最小记录(Infimum+supremum) 、用户记录(User Records)、空闲空间(Free Space)、页目录(Page Directory)和文件尾(File Tailer)。

![](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221226165902.png)

![](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221226170031.png)

###### file header & file trailer

header 

描述各种页的通用信息。(比如页的编号、 其上一页、下一页是谁等) 大小: 38字节

![](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221226170435.png)

* FIL_ PAGE OFFSET (4字节)
  每一个页都有一个单独的页号，就跟你的身份证号码一样，InnoDB通过页号可以==唯一定位==一个页。

* FIL PAGE TYPE (2字节)  这个代表当前页的类型。![](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221226170634.png)

* FIL PAGE PREV (4字节)和FIL PAGE NEXT (4字节)
  InnoDB都是以页为单位存放数据的，如果数据分散到多个不连续的页中存储的话需要把这些页关联起来，FIL_ _PAGE_ PREV和FIL_ _PAGE_ NEXT就分 别代表本页的上一个和下一个页的页号。这样通过建立一个双向链表把许许多多的页就都串联起来了，保证这些页
  之间不需要是物理上的连续，而是逻辑上的连续。

* FIL_ PAGE SPACE OR_ CHKSUM (4字节)       代表当前页面的校验和( checksum)。

  内存写入磁盘页, 校验头尾校验和是否一致,  防止发生意外数据错误

* FIL PAGE LSN (8字节)
  页面被最后修改时对应的日志序列位置(英文名是: Log Sequence Number)

File Trailer (文件尾部) (8字节)

前4个字节代表页的校验和:
这个部分是和FileHeader中的校验和相对应的。
后4个字节代表页面被最后修改时对应的日志序列位置(LSN):
这个部分也是为了校验页的完整性的，如果首部和尾部的LSN值校验不成功的话，就说明同步过程出现了问题。



###### Free Space (空闲空间)

我们自己存储的记录会按照指定的行格式存储到UserRecords部分。但是在一开始生成页的时候，其实并没有User Records这个部分，每当我们插入一条记录，都会从FreeSpace部分，也就是尚未使用的存储空间中申请一个记录大小的空间划分到UserRecords部分，当Free Space部分的空间全部被User Records部分替代掉之后，也就意味着这个页使用完了，如果还有新的记录插入的话，就需要去申请新的页了。

###### User Records (用户记录)

User Records中的这些记录按照指定的行格式-条一条摆在User Records部分，相互之间形成单链表。

###### Infimum Supremum (最小最大记录)
比较主键大小
5字节大小的记录头信息和8字节大小的一个固定的部分组成
![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227101800.png)
这两条记录不是我们自己定义的记录，所以它们并不存放在页的User Records部分，他们被单独放在一个称为Infimum + Supremum的部分
![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227102035.png)

###### Page Directory (页目录)
在页中，记录是以单向链表的形式进行存储的。单向链表的特点就是插入、删除非常方便但是检索效率不高，最差的情况下需要遍历链表上的所有节点才能完成检索。因此在页结构中专门设计了页目录这个模块，专门给记录做一个目录，通过二分查找法的方式进行检索，提升效率。

1. 将所有的记录分成几个组，这些记录包括最小记录和最大记录，但不包括标记为“已删除”的记录。
2. 第1组，也就是最小记录所在的分组只有1个记录;最后一组，就是最大记录所在的分组，会有1-8条记录;其余的组记录数量在4-8条之间。
	除了第1组(最小记录所在组)以外，其余组的记录数会尽量平分。
3. 在每个组中最后一条记录的头信息中会存储该组一共有多少条记录，作为n_ owned字段。
4. 页目录用来存储每组最后一条记录的地址偏移量，这些地址偏移量会按照先后顺序存储起来，每组的地址偏移量也被称之为槽(slot) ，每个槽相当于指针指向了不同组的最后一个记录。

###### Page Header (页面头部)
占用固定的56个字节，专门存储各种状态信息。

![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227103945.png)
假如新插入的一条记录的主键值比上一条记录的主键值大，我们说这条记录的插入方向是右边，反之则是左边。用来表示最后一条记录插入方向的状态就是PAGE_ DIRECTION。

假设连续几次插入新记录的方向都是一致的，InnoDB会 把沿着同一个方向插入记录的条数记下来，这个条数就用PAGE_ N_ DIRECTION这个状态表示。当然，如果最后一条记录的插入方向改变了的话，这个状态的值会被清零重新统计。

##### 数据页的角度看B+树查询


1. B+树是如何进行记录检索的?
	从B+树的根开始，逐层检索，直到找到叶子节点，也就是找到对应的数据页为止，将数据页加载到内存中，页目录中的槽(slot) 采用二分查找的方式先找到一个粗略的记录分组，然后再在分组中通过链表遍历的方式查找记录。
2. 普通索引和唯一索引在查询效率上有什么不同?
	在检索效率上基本上没有差别。


##### InnoDB行格式(或记录格式)

磁盘上的存放方式也被称为行格式或者‘记录格式。
InnoDB存储引擎设计了4种行格式，分别是Compact'、、Redundant'、‘Dynamic' 和、Compressedi

```sql
SELECT @@innodb_ default_ row_ format;
--也可以使用如下语法查看具体表使用的行格式:
SHOW TABLE STATUS like '表名'\G

--在创建或修改表的语句中指定行格式:
CREATE TABLE表名(列的信息) ROW_ FORMAT= 行格式名称
ALTER TABLE表名ROW_ FORMAT= 行格式名称
```


###### COMPACT行格式

在MySQL 5.1版本中，默认为Compact行格式。一条完整的记录其实可以被分为记录的额外信息和记录的真实数据两大部分。
![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227105151.png)

* 变长字段长度列表
	MySQL支持一些变 长的数据类型，比如VARCHAR(M)、VARBINARY(M)、 TEXT类型，BLOB类型，这些数据类型修饰列称为变长字段，变长字段中存储多少字节的数据不是固定的，所以我们在存储真实数据的时候需要顺便把这些数据占用的字节数也存起来。在Compact行格式中，把所有变长字段的真实数据占用的==字节长度都存放在记录的开头部位==从而形成一个变长字段长度列表。
	这里面存储的变长长度和字段顺序是反过来的。
* NULL值列表
	Compact行格式会把可以为NULL的列统一管 理起来，存在--个标记为NULL值 列表中。如果表中没有允许存储NULL的列，则NULL值列表也不存在了
	1 null 0 非null
* 记录头信息
	* delete_mask    0 没删除 1 已删除  真删除会重排序性能消耗  删除的记录会维护一个垃圾链表 该空间为可重用空间
	* min_ rec mask
	  B+树的每层非叶子节点中的最小记录都会添加该标记，min_rec__mask值为1
	* record_ type
	  这个属性表示当前记录的类型，一.共有4种类型的记录:
	  0:表示普通记录
	  1:表示B+树非叶节点记录
	  2:表示最小记录
	  3:表示最大记录
	* heap_ no
	  这个属性表示当前记录在本页中的位置
	  MySQL会自动给每个页里加了两个记录，称为伪记录或虚拟记录。这两个伪记录一个代表最小记录，一个代表最大记录
	  最小记录和最大记录的heap__no值分别是0和1,也就是说它们的位置最靠前。
	* n_owned
		页目录中每个组中最后- - 条记录的头信息中会存储该组一共有多少条记录，作为n_ owned 字段。
	* next_record
		下一条记录的真实数据的地址偏移量
* 记录真实数据![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227110014.png)
	DB_ _ROW_ ID、 DB_ TRX_ ID、 DB_ ROLL PTR。
	一个表没有手动定义主键，则会选取一个Unique键作为主键如果连Unique键都没有定义的话，则会为表默认添加一个名为row_ id的隐藏列作为主键。所以row_ id是在没有自定义主键以及Unique键的情况下才会存在的。
###### Dynamic和Compressed行格式
行溢出
InnoDB存储引擎可以将一条记录中的某些数据存储在真正的数据页面之外。

页是16KB，也就是16384字节， 而一个VARCHAR(M)类型的列就最多可以存储65533个字节，这样就可能出现一个页存放不了-条记录， 这种现象称为行溢出。
在Compact和Reduntant行格式中，对于占用存储空间非常大的列，在记录的真实数据处只会存储该列的一部分数据，把剩余的数据分散存储在几个其他的页中进行分页存储，然后记录的真实数据处用20个字节存储指向这些页的地址(当然这20个字节中还包括这些分散在其他页面中的数据的占用的字节数)，从而可以找到剩余数据所在的页。

在MySQL 8.0中，默认行格式就是Dynamic, 
Dynamic、 Compressed行 格式和Compact行格式挺像，只不过在处理行溢出数据时有分歧:
* Compressed和Dynamic两种记录格式对于存放在BLOB中的数据采用了完全的行溢出的方式。在数据页中只存放20个字节的指针(溢出页的地址)，实际的数据都存放在Off Page (溢出页)中。
* Compact和Redundant两种格式会在记录的真实数据处存储-部分数据(存放768个前缀字节)。

Compressed行记录格式的另-个功能就是， 存储在其中的行数据会以zlib的算法进行 压缩，因此对于BLOB、TEXT、VARCHAR这类大长度类型的数据能够进行非常有效的存储

###### Redundant行格式
Redundant是MySQL 5.0版本之前InnoDB的行记录存储方式，MySQL 5.0支持Redundant是为了兼容之前版本的页格式。
![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227111649.png)

从上图可以看到，不同于Compact行记录格式，Redundant行格 式的首部是一个字段长度偏移列表，同样是按照列的顺序逆序放置的。

注意Compact行格式的开头是变长字段长度列表，而Redundant行格式的开头是字段长度偏移列表，与变长字段长度列表有两处不同:
* 少了“变长”两个字: Redundant行 格式会把该条记录中所有列(包括隐藏列)的长度信息都按照逆序存储到字段长度偏移列表
* 多了“偏移”两个字:这意味着计算列值长度的方式不像Compact行格式那么直观它是采用两个相邻数值的差值来计算各个列值的长度。

不同于Compact行格式，Redundant行 格式中的记录头信息固定占用6个字节(48位)每位的含义见下表。
![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221227111911.png)

与Compact行格 式的记录头信息对比来看，有两处不同:
* Redundant行格式多了n_ field和1byte_ _offs_ flag这两 个属性。
* Redundant行 格式没有record_ type这 个属性。
其中，n_ fields: 代表一行中列的数量， 占用10位， 这也很好地解释了为什么MySQL-个行支持最多的列为1023。另一个值为1byte_ _offs_ flags, 该值定义了偏移列表占用1个字节还是2个字节。当它的值为1时，表明使用1个字节存储。当它的值为0时，表明使用2个字节存储。











### MyISAM

| 索引/存储引擎 | MyISAM | InnoDB | Memory |
| ------------- | ------ | ------ | ------ |
| B-Tree索引    | 支持   | 支持   | 支持   |

即使多个存储引擎支持同一种类型的索引，但是他们的实现原理也是不同的。Innodb和MyISAM默认的索引是Btree索引;而Memory默认的索引|是Hash索引。

MyISAM引擎使用B+Tree作为索弓|结构，叶子节点的data域存放的是==数据记录的地址==。

MyISAM将索引和数据分开存储:

* 将表中的记录按照记录的插入顺序单独存储在一个文件中， 称之为数据文件。这个文件并不划分为若干个数据页，有多少记录就往这个文件中塞多少记录就成了。由于在插入数据的时候并没有刻意按照主键大小排序，所以我们并不能在这些数据上使用二分法进行查找。
* 使用MyISAM存储弓|擎的表会把索引|信息另外存储到一个称为索引文件的另一个文件中。MyISAM 会单独为表的主键创建一个索引， 只不过在索的叶子节点中存储的不是完整的用户记录，而是主键值+数据记录地址的组合。

#### MyISAM与InnoDB对比

MylSAM的索引方式都是“非聚簇”的，与InnoDB包含1个聚簇索引是不同的。 区别:

1. 在InnoDB存储弓|擎中，我们只需要根据主键值对聚簇索引进行一次查找就能找到对应的记录，而在MyISAM中却需要进行一次回表操作，意味着MyISAM中建立的索引相当于全部都是二级索引。
2. InnoDB的数据文件本身就是索引文件，而MyISAM索引文件和数据文件是 分离的，索引文件仅保存数据记录的地址。
3. InnoDB的非聚簇索引data域存储相应记录主键的值，而MyISAM索引记录的是地址。换句话说，InnoDB的所有非聚簇索引都引用主键作为data域。
4. MyISAM的回表操作是十分快速的，因为是拿着地址偏移量直接到文件中取数据的，反观InnoDB是通过获取主键之后再去聚簇索引里找记录，虽然说也不慢，但还是比不上直接用地址去访问。
5. InnoDB要求表必须有主键( MyISAM可以没有)。如果没有显式指定，则MySQL系统会自动选择一个可以非空且唯一标识数据记录的列作为主键。如果不存在这种列，则MySQL 自动为InnoDB表生成一个隐含字段作为主键 ,这个字段长度为6个字节，类型为长整型。

### 数据结构

磁盘的I/0 操作次数对索引的使用效率至关重要。
#### Hash

Memory存储引擎

加速查找速度的数据结构，常见的有两类:

* 树，例如平衡二叉搜索树，查询/插入/修改/删除的平均时间复杂度都是0(log2N) ;
* 哈希，例如HashMap, 查询/插入修改/删除的平均时间复杂度都是0(1) ;

缺点

1. Hash 索引仅能满足(=) (<>) 和IN查询。如果进行范围查询，哈希型的索引，时间复杂度会退化为0(n);而树型的“有序”特性，依然能够保持0(log2N) 的高效率。
2. Hash 索弓|还有一个缺陷，数据的存储是没有顺序的，在ORDER BY的情况下，使用Hash索弓|还需要对数据重新排序。
3. 对于联合索引|的情况，Hash 值是将联合索引键合并后一起来计算的，无法对单独的一一个键或者几个索引键进行查询。
4. 对于等值查询来说，通常Hash索引|的效率更高，不过也存在一种情况， 就是索引列的重复值如果很多，效率就会降低。这是因为遇到Hash冲突时，需要遍历桶中的行指针来进行比较，找到查询的关键字，非常耗时。所以，Hash 索弓通常不会用到重复值多的列上，比如列为性别、年龄的情况等。



InnoDB本身不支持Hash索引，但是提供自适应Hash索引(Adaptive Hash Index)。什么情况下才会使用自适应Hash索引呢?如果某个数据经常被访问，当满足一定条件的时候， 就会将这个数据页的地址存放到Hash表中。这样下次查询的时候，就可以直接找到这个页面的所在位置。这样让B+树也具备了Hash索引的优点。

```sql
show variables like' %adaptive_ hash_ index' ;
```

#### 二叉搜索树

如果我们利用二叉树作为索引结构，那么磁盘的I0次数和索弓|树的高度是相关的。

1. 二叉搜索树的特点
   * 一个节点只能有两个子节点，也就是一个节点度不能超过2
   * 左子节点<本节点;右子节点>=本节点，比我大的向右，比我小的向左

#### AVL树

为了解决上面二叉查找树退化成链表的问题，人们提出了平衡二叉搜索树(Balanced Binary Tree), 又称为AVL树(有别于AVL算法)，它在二叉搜索树的基础上增加了约束，具有以下性质:
它是- 棵空树或它的左右两个子树的高度差的绝对值不超过1,并且左右两个子树都是一棵平衡=叉树。

常见的平衡二二叉树有很多种，包括了平衡二叉搜索树、红黑树、 数堆、伸展树 。

#### B-Tree

B树的英文是Balance Tree,也就是==多路平衡查找树==。简写为B-Tree (注意横杠表示这两个单词连起来的意思，不是减号)。它的高度远小于平衡二叉树的高度。

B树相比于平衡二叉树来说磁盘I/0操作要少，在数据查询中比平衡=叉树效率要高。所以只要树的高度足够低，I0次数足够少，就可以提高查询性能。
1. B树在插入和删除 节点的时候如果导致树不平衡，就通过自动调整节点的位置来保持树的自平衡。
2. 关键字集合分布在整棵树中，==即叶子节点和非叶子节点都存放数据==。搜索有可能在非叶子节点结束
3. 其搜索性能等价于在关键字全集内做- -次二分查找。

#### B+Tree

B+树也是一种多路搜索树，基于B树做出了改进。相比于B-Tree, B+Tree适合文件索引 系统。

B+树和B树的差异
1. 有k个孩子的节点就有k个关键字。也就是孩子数量=关键字数，而B树中，孩子数量=关键字数+1。
2. 非叶子节点的关键字也会同时存在在子节点中，并且是在子节点中所有关键字的最大(或最小)。
3. 非叶子节点仅用于索引，不保存数据记录，跟记录有关的信息都放在叶子节点中。而B树中，==非叶子节点既保存索引，也保存数据记录。==
4. 所有关键字都在叶子节点出现，叶子节点构成一个有序链表， 而且叶子节点本身按照关键字的大小从小到大顺序链接。

![123](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221226163347.png)

B+树查询效率更稳定。

B+树查询效率更高。

在查询范围上，B+树的效率也比B树高。

#### R树

R-Tree在MySQL很少使用，==仅支持geometry数据类型，==支持该类型的存储弓|擎只有myisam、bdb、 innodb、ndb、archive几种。 

举个R树在现实领域中能够解决的例子:查找20英里以内所有的餐厅。如果没有R树你会怎么解决? -般情况下我们会押餐厅的坐标(x,y)分为两个字段存放在数据库中，-个字段记录经度，另-一个字段记录纬度。这样的话我们就需要遍历所有的餐厅获取其位置信息，然后计算是否满足要求。如果一个地区有100家餐厅的话，我们就要进行100次位置计算操作了,如果应用到谷歌、百度地图这种超大数据库中，这种方法便必定不可行了。R树就很好的解决了这种高维空间搜索问题。它把B树的思想很好的扩展到了多维空间，采用了B树分割空间的思想，并在添加、删除操作时采用合并、分解结点的方法，保证树的平衡性。因此，R树就是一棵用来存储高维数据的平衡树。相对于B-Tree, R-Tree的优势在于范围查找。














### 性能分析工具

#### 数据库服务器优化步骤
* 周期性波动    加缓存  更改缓存失效策略
* 非周期性波动
	1. 开慢查询
	2. explan  show profiling
	3. sql等待时间长  调优服务器参数
* sql查询达到瓶颈
	* 主从   读写分离
	* 分库分表

#### 查看性能参数
```sql
SHOW [ GLOBAL | SESSION] STATUS LIKE ' 参数';
```
* Connections: 连接MySQL服务器的次数。
* Uptime: MySQL服务器的上线时间。
* Slow_ queries:慢查询的次数。
* Innodb_ rows_ read: Select查询返回的行数
* Innodb_ rows_ inserted: 执行INSERT操作插入的行数
* Innodb_ rows_ updated:执行UPDATE操作更新的行数
* Innodb_ rows_deleted:执行DELETE操作删除的行数
* Com_ select: 查询操作的次数。
* Com_insert: 插入操作的次数。对于批量插入的INSERT操作，只累加一次。
* Com_ update: 更新操作的次数。
* Cam_ delete: 删除操作的次数。

##### 统计SQL查询成本: last_query_cost
成本即SQL语句所需要读取的页的数量

1. 位置决定效率。如果页就在数据库缓冲池中，那么效率是最高的，否则还需要从内存或者磁盘中进行读取，当然针对单个页的读取来说，如果页存在于内存中，会比在磁盘中读取效率高很多。
2. 批量决定效率 。如果我们从磁盘中对单一页进行随机读，那么效率是很低的(差不多10ms)， 而采用顺序读取的方式，批量对页进行读取，平均-页的读取效率就会提升很多，甚至要快于单个页面在内存中的随机读取。

所以说，遇到I/O并不用担心，方法找对了，效率还是很高的。我们首先要考虑数据存放的位置，如果是经
常使用的数据就要尽量放到缓冲池中，其次我们可以充分利用磁盘的吞吐能力，-次性批量读取数据，这样单个页的读取效率也就得到了提升。

#### 慢查询日志
long_ query_ time  默认为10，默认不开启,  不调优不开启

min_ examined_ row_ limit. 查询扫描过的最少记录数。默认是0
这个变量和查询执行时间，共同组成了判别一个查询是否是慢查询的条件。如果查询
扫描过的记录数大于等于这个变量的值，并且查询执行时间超过long_ query_ time 的值，那么，这个查询就被记录到慢查询日志中;反之，则不被记录到慢查询日志中。
```sql
show variables like '%slow_query_log';
set global slow_query_log= 'ON';
--永久设置的方式。
--修改my.cnf文件[mysqld]下增加或修改参数long_query_time、slow_query_log和slow_query_log_file后，然后重启MySQL服务器。
[mysq1d]
slow_query_log=ON #开启慢查询日志的开关
slow_query_log_file=/var/lib/mysql/atguigu-slow.1og
#慢查询日志的目录和文件名信息
long_query_Time=3 #设置慢查询的阙值为3秒，超出此设定值的SQL即被记录到慢查询日志
log_output=FILE
--如果不指定存储路径，慢查询日志将默认存储到MySQL数据库的数据文件夹下。如果不指定默认为hostname-slow.log

--查询当前系统中有多少条慢查询记录
SHOW GLOBAL STATUS LIKE '%slow_queries%' ;

--关闭 配置文件注掉
SET GLOBAL slow_query_1og=off;
```

##### 分析工具mysqldumpshow
```shell
#重新生成日志文件
mysqladmin -uroot -p flush-logs slow 

mysqldumpshow --help

#得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow. log
#得到访问次数最多的10个SQL
mysqldumpslow -s C -t 10 /var/lib/mysql/atguigu-slow. log
#得到按照时间排序的前10条里面含有左连接的查询语句
mysq1dumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow .1og

```






#### 执行成本: SHOW PROFILE
默认关闭
```sql
show variables like' profiling' ;
set profiling ='ON' ;

show profiles
show profile for [num]

show profile cpu,block io for query 7;

Syntax:
SHOW PROFILE [type [, type]...
[FOR QUERY n]
[LIMIT row_count [OFFSET offset]]

type:{
ALL --一显示所有参数的开销信息
|BLOCK IO --显示IO的相关开销
|CONTEXT SWITCHES--上下文切换相关开销
|CPU--显示CPU相关开销信息
|IPC--显示发送和接收相关开销
|MEMORY--显示内存相关开销信息
| PAGE FAULTS--显示页面错误相关开销信息
|SOURCE--显示和Source_function,Source_file,Source_line相关的开销信息
|SWAPS--显示交换次数相关的开销信息
}
```
日常开发需注意的结论:
1. converting HEAP to MyISAM:查询结果太大，内存不够，数据往磁盘上搬了。
2. Creating tmp table :创建临时表。先拷贝数据到临时表，用完后再删除临时表。
3. Copying to tmp table on disk :把内存中临时表复制到磁盘上，警惕!
4. locked。
如果在show profile诊断结果中出现了以上4条结果中的任何一条，则sql语句需要优化。

SHOW PROFILE命令将被弃用，我们可以从information_ schema 中的profiling数据表进行查看。

#### EXPLAIN
EXPLAIN/DESCRIBE
可以做

- 表的读取顺序
- 数据读取操作的操作类型
- 哪些索引可以使用
- 哪些索引被实际使用
- 表之间的引用
- 每张表有多少行被优化器查询

| 列名                  | 描述                                                   |
| --------------------- | ------------------------------------------------------ |
| id                    | 在一个大的查询语句中每个SELECT关键字都对应一个唯一的id |
| select_type           | SELECT关键字对应的那个查询的类型                       |
| table                 | 表名                                                   |
| partitions            | 匹配的分区信息                                         |
| ==**type**==          | 针对单表的访问方法                                     |
| **==possible_keys==** | 可能用到的索引                                         |
| ==**key**==           | 实际上使用的索引                                       |
| ==**key_len**==       | 实际使用到的索引长度                                   |
| ref                   | 当使用索引列等值查询时，与索引列进行等值匹配的对象信息 |
| rows                  | 预估的需要读取的记录条数                               |
| filtered              | 某个表经过搜索条件过滤后剩余记录条数的百分比           |
| ==**Extra**==         | 一些额外的信息                                         |

![image-20210914154252170](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210914154252170.png)


##### id

select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的==顺序==  一个select对应一个id
 
- id相同，执行顺序由上至下
- id不同，id值越大优先级越高，越先被执行
- id越少越好

衍生=DERIVED

##### select_type

查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询

| select_type  | 简介                                                         |
| ------------ | ------------------------------------------------------------ |
| SIMPLE       | 简单的select 查询，查询中不包含子查询或者UNION               |
| PRIMARY      | 查询中若包含任何复杂的子部分，最外层查询则被标记             |
| SUBQUERY     | 在SELECT或WHERE列表中包含了子查询                            |
| DERIVED      | 在FROM列表中包含的子查询被标记为DERIVED（衍生）MySQL会递归执行这些子查询，把结果放在临时表里 |
| UNION        | 若第二个SELECT出现在UNION之后，则被标记为UNION                                                                                      若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED |
| UNION RESULT | 从UNION表获取结果的SELECT                                    |

##### type

![image-20210914155042014](C:\Users\cch\AppData\Roaming\Typora\typora-user-images\image-20210914155042014.png)

访问类型

==从最好到最差依次是==

```sql
system > const > eq_ref > ref > range > index > ALL
--或
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
```

一般来说，得保证查询至少达到range级别，最好能达到ref

| type   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| system | 表只有一行记录（等于系统表）,这是const类型的特列，平时不会出现，这个也可以忽略不计 |
| const  | 表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快如将主键置于where列表中，MySQL就能将该查询转换为一个常量 |
| eq_ref | 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描 |
| ref    | 非唯一性索引扫描，返回匹配某个单独值的所有行.本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体 |
| range  | 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引, 一般就是在你的where语句中出现了between、<、>、in等的查询, 这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引 |
| index  | Full Index Scan与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。(也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的） |
| ALL    | 全表扫描                                                     |

##### possible_keys
可能用的索引，一个或多个。

查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

##### key
实际使用索引

查询中若使用了覆盖索引，则该索引仅出现在key列表中

##### key_len
表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。==在不损失精确性的情况下，长度越短越好==

key_len显示的值为索引字段的最大可能长度，==并非实际使用长度==，即key_len是根据表定义计算而得，不是通过表内检索出的

##### ref
显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值
##### rows
根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数
##### filtered
条件过滤后剩余记录条数的百分比
单表查询，filtered列的值没什么意义，更关注在连接查询中驱动表对应的执行计划记录的filtered值， 它决定了被驱动表要执行的次数(即: rows * filtered)

##### Extra

额外信息

| extra                            | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| ==**Using filesort**==           | 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取, MySQL中无法利用索引完成的排序操作称为”文件排序” |
| ==**Using temporary**==          | 使用临时表    常见于order by 和group by                      |
| ==**USING index**==              | 使用了覆盖索引（Covering Index) 如果同时出现using where,表明索引被用来执行索引键值的查找；如果没有同时出现using where,表明索引用来读取数据而非执行查找动作 |
| **Using where**                  | 使用了where过滤                                              |
|Index Condition Pushdown|  有些搜索条件中虽然出现了索引列，但却不能使用到索引   |
| **using join buffer**            | 使用了连接缓存                                               |
| **impossible where**             | where总是false，不能获取任何元组                             |
| **select tables optimized away** | 在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT()操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。|
| **distinct**                     | 优化distinct                                                 |

覆盖索引：不用回表即可获取查询数据    ==查询列要被所建的索引覆盖。==

##### 输出格式
* 传统格式   explain + sql
* JSON格式  explain format=json + sql    显示信息最全  查询成本
	* read_cost   io成本   检测rows x (1 - filter)条记录的cpu成本
	* eval_ cost   rows * filter 成本。
	* prefix_ cost   read_ cost + eval_ cost
	* data_ read_per_join   需要读取的数据量。
* TREE格式    主要根据查询的各个部分之间的关系和各部分的执行顺序来描述如何查寻
* 可视化输出    mysql workbench 

##### SHOW WARNINGS
EXPLAIN后，用SHOW WARNINGS 查看扩展信息

* level
* code
* message

Code值为1003时， Message 展示查询优化器重写后的语句

#### trace
分析优化器执行计划
0PTIMIZER_ TRACE 可以跟踪优化器做出的各种决策(比如访问表的方法、各种开销计算、各种转换等)，并将跟踪结果记录到INFORMATION_ SCHEMA.0PTIMIZER_TRACE表中。
默认关闭。开启trace, 并设置格式为JSON,同时设置trace最大能够使用的内存大小，避免解析过程中因
为默认内存过小而不能够完整展示。
```sql
SET optimizer_trace="enabled=on", end_markers_in_json=on;
set optimizer_trace_max_mem_size= 1000000 ;

--开启后，可分析如下语句:
●SELECT
●INSERT
●REPLACE
●UPDATE
●DELETE
●EXPLAIN
●SET
●DECLARE
●CASE
●IF
●RETURN
●CALL 
```

1. query 查询语句
2. trace  query字段对应语句的跟踪信息
3. MISSING_BYTES_BEYOND_MAX_MEM_SIZE:0 跟踪信息过长时，被截断的跟踪信息的字节数。
	丢失的超出最大容量的字节
4. INSUFFICIENT_PRIVILEGES: 0   缺失权限
	执行跟踪语句的用户是否有查看对象的权限。当不具有权限时，该列信息为1且TRACE字段为空，一般在调用带有SQL SECURITY DEFINER的 视图或者是存储过程的情况下，会出现此问题。

#### 监控分析视图-sys schema

降低查询per formance_ schema的复杂度， 生产不要频繁用

1. 主机相关:以host _summary开头，主要汇总了I0延迟的信息。
2. Innodb相关:以innodb开头， 汇总了innodb buffer信息和事务等待innodb锁的信息。
3. 1/o相关:以io开头， 汇总了等待I/O、I/0使用量情况。
4. 内存使用情况:以memory开头，从主机、线程、事件等角度展示内存的使用情况
5. 连接与会话信息: processlist和session相关视图， 总结了会话相关信息。
6. 表相关:以schema_ _table开头的视图，展示了表的统计信息。
7. 索引信息:统计了索引的使用情况，包含冗余索引和未使用的索引情况。
8. 语句相关:以statement开头， 包含执行全表扫描、使用临时表、排序等的语句信息。
9. 用户相关:以user开头的视图，统计了用户使用的文件/O、执行语句统计信息。
10. 等待事件相关信息:以wait开头， 展示等待事件的延迟情况。

```sql
索引情况
#1.查询冗余索引
select * from sys.schema_redundant_indexes ;
#2.查询未使用过的索引
select * from sys.schema_unused_indexes;
#3.查询索引的使用情况
select index_name, rows_selected, rows_inserted, rows_updated, rows_deleted
from sys.schema_index_statistics where table schema= 'dbname' ;

表相关
# 1.查询表的访问量
select table_schema,table_name,sum(io_read_requests+io_write_requests) as io from
sys.schema_table_statistics group by table_schema,table_name order by io desc;
# 2.查询占用bufferpool较多的表:
select object_schema,object_name , allocated, data
from sys.innodb_buffer_stats_by_table order by allocated limit 10;
# 3.查看表的全表扫描情况
select * from sys.statements_with_full_table_scans where db='dbname';

语句相关
#1.监控SQL执行的频率
select db,exec_count,query from sys.statement_analysis
order by exec_count desc;
#2.监控使用了排序的SQL
select db,exec_count,first_seen,last_seen,query
from sys.statements_with_sorting limit 1;
#3.监控使用了临时表或者磁盘临时表的SQL
select db,exec_count,tmp_tables,tmp_disk_tables,query
from sys.statement_analysis where tmp_tables>0 or tmp_disk_tables >0
order by (tmp_tables+tmp_disk_tables)desc; 

io相关
#1.查看消耗磁盘I0的文件
select file,avg_read,avg_write,avg_read+avg_write as avg_io
from sys.io_global_by_file_by_bytes order by avg_read limit 10;

Innodb相关
#1.行锁阻塞情况
select from sys.innodb_lock_waits;
```


















### 优化
#### 关联查询优化
* 小结果集驱动大结果集
* 驱动表匹配的条件增加索引|(减少内层表的循环匹配次数)
* 增大join buffer size的大小(-次缓存的数据越多，那么内层包的扫表次数就越少)
* 减少驱动表不必要的字段查询(字段越少，join buffer 所缓存的数据就越多)

```sql
--block_nested_loop
show variables1ike'%optimizer_switch%' #查看block_nested_loop状态。默认是开启的。
--join_buffer_size
驱动表能不能一次加载完，要看join buffer能不能存储所有的数据，默认情况下join_buffer_size=256k。
show variables1ike'%join_buffer_size%' 
```

1. Simple Nested-Loop Join (简单嵌套循环连接)
2. Index Nested-Loop Join (索引 嵌套循环连接)   使用索引
3. Block Nested-Loop Join (块嵌套循环连接)   使用join buffer

##### hash join
MySQL8.0.2废弃BNLJ, 引入了hash join默认都会使用hash join
* Nested Loop:
	对于被连接的数据子集较小的情况，Nested Loop:是个较好的选择。
* Hash Join是做大数据集连接时的常用方式，优化器使用两个表中较小（相对较小）的表利用Join Key在内存中建立散列表，然后扫描较大的表并探测散列表，找出与Hash表匹配的行。
	* 这种方式适用于较小的表完全可以放于内存中的情况，这样总成本就是访问两个表的成本之和。
	* 在表很大的情况下并不能完全放入内存，这时优化器会将它分割成若干不同的分区，不能放入内存的部分就把该分区写入磁盘的临时段，此时要求有较大的临时段从而尽量提高I/O的性能。
	* 它能够很好的工作于没有索引的大表和并行查询的环境中，并提供最好的性能。大多数人都说它是Join的重型升降机。Hash Join,只能应用于等值连接（如WHERE A.CoL1=B.COL2),这是由Hash的特点决定的。
![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20221228173459.png)

#### 子查询优化

子查询的执行效率不高
1. 执行子查询时，内层查询语句的查询结果建立临时表 ，然后外层查询语句从临时表中查询记录。查询完毕后，再撤销这些临时表。这样会消耗过多的CPU和IO资源，产生大量的慢查询。
2. 临时表没有索引
3. 结果集越大的子查询，对性能影响越大。
用join替代子查询

> 尽量不要使用NOT IN或者NOT EXISTS,用LEFT JOIN xxx ON xx WHERE xx IS NULL替代

#### 排序优化

1. SQL 中，可以在WHERE子句和ORDER BY子句中使用索引，目的是在WHERE子句中避免全表扫描，在ORDER BY子句避免使用FileSort 排序。当然，某些情况下全表扫描，或者FileSort排序不一定比索引慢。但总的来说，我们还是要避免，以提高查询效率。
2. 尽量使用Index完成ORDER BY排序。如果WHERE和ORDER BY后面是相同的列就使用单索弓|列;如果不同就使用联合索引。
3. 无法使用Index时，需要对FileSort方式进行调优。


##### FileSort
1. 双路排序(慢)
	* MySQL 4.1之前是使用双路排序，扫两次磁盘，读取行指针和order by列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出
	* 从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。取一批数据，要对磁盘进行两次扫描，众所周知，IO是很耗时的，所以在mysql4.1之后，出现了第二种改进的算法，就是单路排序。
2. 单路排序(快)
	* 从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机I0变成了顺序I0，但是它会使用更多的空间，因为它把每一行都保存在内存中了。
	* 在sort_ buffer中， 单路比多路要多占用空间
```sql
1.尝试提高sort_buffer_size   默认1mb
SHOW VARIABLES LIKE '%sort_buffer_size%';
2.尝试提高max_length_for_sort_data     提高这个参数，会增加用改进算法的概率。
SHOW VARIABLES LIKE' %max_length_for_sort_data%';     #默认1024字节
--但是如果设的太高，数据总容量超出sort_ _buffer_ size的概率就增大，明显症状是高的磁盘I/0活动和低的处理器使用率。如果需要返回的列的总长度大于max_ length. _for sort. _data, 使用双路算法，否则使用单路算法。1024-8192字节之间调整


```
1. Order by时select*是一个大忌。最好只Query需要的字段。原因:
	* 当Query的字段大小总和小于max. length_ for_ sort_ data，而且排序字段不是TEXT|BLOB类型时，会用改进后的算法- -单路排序，否则用老算法- -多路排序。
	* 两种算法的数据都有可能超出sort_ buffer size的容量，超出之后，会创建tmp文件进行合并排序，导致多次I/O，但是用单路排序算法的风险会更大一一些， 所以要提高sort_ .buffer. size 

#### GROUP BY优化

* group by使用索引的原则几乎跟order by一致，group by即使没有过滤条件用到索引，也可以直接使用索引。
* group by先排序再分组，遵照索引建的最佳左前缀法则
* 当无法使用索引列，增大max_length_for_sort_data和sort_buffer_size参数的设置
* where效率高于having,能写在where限定的条件就不要写在having中了
* 减少使用order by,和业务沟通能不排序就不排序，或将排序放到程序端去做。Order by、group by、.distinct这些语句较为耗费cPU,数据库的cPU资源是极其宝贵的。
* 包含了order by、group by、distinct这些查询的语句，where条件过滤出来的结果集请保持在1000行以内，否则SQL会很慢。

#### 分页优化

```sql
在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。
EXPLAIN SELECT * 
FROM student t,(SELECT id FROM student ORDER BY id LIMIT 2000000,10) a
WHERE t.id a.id;

该方案适用于主键自增的表，可以把Limi t查询转换成某个位置的查询。
EXPLAIN SELECT FROM student WHERE id 2000000 LIMIT 10;
```
#### 覆盖索引

不用回表即可获取查询数据    ==查询列要被所建的索引覆盖。==

1. 避免回表, 减少IO
2. 随机IO改成顺序IO
3. != 和 like 可用覆盖索引优化
#### 索引下推ICP
5.6中新特性，是一种在存储引擎层使用索引过滤数据的优化方式。
在回表前尽可能使用where后的条件, 在二级索引上尽可能过滤数据

* 默认开启。可以通过设置系统变量optimizer-switch控制：index_condition_pushdown
```sql
#关闭索引下推
SET optimizer_switch 'index_condition_pushdown=off';
#打开索引下推
SET optimizer_switch 'index_condition_pushdown=on';
```
* 当使用索引条件下推时，EXPLAIN语句输出结果中Extra列内容显示为Using index coundition。

ICP的使用条件
1. 如果表访问的类型为range、ref、eq_ref和ref_or_null可以使用lcP
2. ICP可以用于InnoDB和MyISAM表，包括分区表InnoDB和MyISAM表
3. 对于InnoDB表，ICP仅用于二级索引。ICP的目标是减少全行读取次数，从而减少I/O操作。
4. 当SQL使用覆盖索引时，不支持ICP。因为这种情况下使用ICP不会减少I/O。
5. 相关子查询的条件不能使用ICP
#### 其他优化策略
* exists和in
	* 小结果集驱动大结果集
* COUNT( * )与COUNT(具体字段)效率
	* count(1)和count( * )没有区别
	* MyISAM  O(1)复杂度  每个表维护一个row_count值
	* InnoDB 全表扫描 O(n)复杂度
		* COUNT(具体字段)使用二级索引  自动找占用空间小的二级索引进行全表扫描
*  select(* )
	* 解析的过程中，会通过查询数据字典将"* "按序转换成所有列名，这会大大的耗费资源和时间。
	* 无法使用覆盖索引
* LIMIT 1
	* 全表扫描 加上可以优化
	* 走唯一索引 不用加
* commit
	* 尽量多使用COMMIT,这样程序的性能得到提高
	* COMMIT所释放的资源:
		* 回滚段上用于恢复数据的信息
		* 被程序语句获得的锁
		* redo / undo log buffer中的空间
		* 管理上述3种资源中的内部花费
#### 主键设计

##### 自增ID的问题

1. 可靠性不高
	存在自增ID回溯的问题，这个问题直到最新版本的MySQL 8.0才修复。
2. 安全性不高
	对外暴露的接口可以非常容易猜测对应的信息。比如: /User/1/这样的接口， 可以非常容易猜测用户ID的值为多少，总用户数量有多少，也可以非常容易地通过接口进行数据的爬取。
3. 性能差
	自增ID的性能较差，需要在数据库服务器端生成。
4. 交互多
	业务还需要额外执行一-次类似last_ insert. id()的函数才能知道刚才插入的自增值，这需要多-次的网络交互。在海量并发的系统中，多1条SQL，就多-次性能上的开销。
5. 局部唯-性
	只在当前数据库实例中唯一， 而不是全局唯一， 在任意服务器间都是唯一的。对于目前分布式系统来说，这简直就是噩梦。

##### 推荐主键设计
非核心业务:对应表的主键自增ID,如告警、日志、监控等信息。
核心业务:主键设计至少应该是全局唯一-且是单调递增。全局唯一保证在各 系统之间都是唯一-的， 单调递增是希望插入时不影响数据库性能。

###### UUID
全局唯一, 占用36字节，数据无序，插入性能差。
UUID =时间 (间隔100ns)+UUID版本(16字节) - 时钟序列(4字节)-MAC地址(12字节)

若将时间高低位互换，则时间就是单调递增的了，也就变得单调递增了。MySQL 8.0可以更换时间低位和时间高位的存储方式，这样UUID就是有序的UUID了。
MySQL 8.0解决了UUID存在的空间占用的问题，除去了UUID字符串中无意义的"-"字符串，并且将字符串用二进制类型保存，这样存储空间降低为了16字节。
可以通过MySQL8.0提供的uuid_ to_ bin函数实现上述功能，同样的，MySQL也提供 了bin. to_ uuid函数进行转化:
```sql
SET @uuid = UUID();
SELECT @uuid, uuid_to_bin(@uuid), uuid_to_bin(@uuid,TRUE);
```

### 数据库设计规范

#### 范式
在关系型数据库中，关于数据表设计的基本原则、规则就称为范式。
Normal Form  简称NF

目前关系型数据库有六种常见范式，按照范式级别，从低到高分别是:
第-范式(1NF) 、第二范式(2NF)、第三范式(3NF) 、巴斯科德范式(BCNF) 、第四范式(4NF)和第五范式(5NF, 又称完美范式)。
数据库的范式设计越高阶，冗余度就越低，同时高阶的范式-定符合低阶范式的要求，满足最低要求的范式是第-范式(1NF) 。在第一范式的基础上进一步满足更多规范要求的称为第二 范式(2NF)， 其余范式以次类推。
-般来说，在关系型数据库设计中，最高也就遵循到BCNF，普遍还是3NF。但也不绝对，有时候为了提高某
些查询性能，我们还需要破坏范式规则，也就是反规范化。

* 超键:能唯一标识元组的属性集叫做超键。
* 候选键:如果超键不包括多余的属性，那么这个超键就是候选键。
* 主键:用户可以从候选键中选择一 个作为主键。
* 外键:如果数据表R1中的某属性集不是R1的主键，而是另一一个数据表R2的主键，那么这个属性集就是数据表R1的外键。
* 主属性:包含在任一候选键中的属性称为主属性。
* 非主属性:与主属性相对，指的是不包含在任何一个候选键中的属性。

通常，我们也将候选键称之为“码”，把主键也称为“主码”。因为键可能是由多个属性组成的，针对单个属性，我们还可以用主属性和非主属性来进行区分。


优点  消除数据冗余，第三范式(3NF) 通常被认为在性能、扩展性和数据完整性方面达到了最好的平衡
缺点  降低查询效率。查询关联多张表，代价昂贵，可能使一些索引策略无效


##### 第一范式(1st NF)

表中每个字段的值必须具有原子性

##### 第二范式(2nd NF)
第二范式要求，在满足第一范式的基础上， 还要满足数据表里的每一条数据记录， 都是可唯一标识的。而且所有非主键字段，都必须完全依赖主键，不能只依赖主键的一部分。如果知道主键的所有属性的值，就可以检索到任何元组(行)的任何属性的任何值。(要求中的主键， 其实可以拓展替换为候选键)。

##### 第三范式(3rd NF)
第三范式是在第二范式的基础上，确保数据表中的每一个非主键字段都和主键字段直接相关，也就是说，要求数据表中的所有非主键字段不能依赖于其他非主键字段。
该规则的意思是所有非主键属性之间不能有依赖关系，必须相互独立。


##### 反范式
增加冗余字段来提高数据库的读性能。

* 这个冗余字段不需要经常进行修改;
* 这个冗余字段查询的时候不可或缺。

数据仓库 历史数据

##### BCNF(巴斯范式)
3NF的基础上进行了改进，提出了巴斯范式(BCNF) ,， 也叫做巴斯.科德范式(Boyce-Codd Normal Form)。BCNF被认为没有新的设计规范加入，只是对第三范式中设计规范要求更强，使得数据库冗余度更小。

若一个关系达到了第三范式，并且它只有一个候选键，或者它的每个候选键都是单属性，则该关系自然达到BC范式

-般来说，一个数据库设计符合3NF或BCNF就可以了。

##### 第四范式
多值依赖的概念：
* 多值依赖即属性之间的一对多关系，记为K→→A。
* 函数依赖事实上是单值依赖，所以不能表达属性值之间的一对多关系。
* 平凡的多值依赖：全集U=K+A,一个K可以对应于多个A,即K→→A。此时整个表就是一组一对多关系。
* 非平凡的多值依赖：全集U=K+A+B,一个K可以对应于多个A,也可以对应于多个B,A与B互相独立，即K→→A,K→→B。整个表有多组一对多关系，且有：“一”部分是相同的属性集合，“多”部分是互相独立的属性集合。
第四范式即在满足巴斯-科德范式(BCNF)的基础上，消除非平凡且非函数依赖的多值依赖（即把同一表内的多对多关系删除)。

##### 第五范式、域键范式
除了第四范式外，我们还有更高级的第五范式（又称完美范式）和域键范式(DKNF)。
在满足第四范式(4NF)的基础上，消除不是由候选键所蕴含的连接依赖。如果关系模式中的每一个连接依赖均由的候选键所隐含，则称此关系模式符合第五范式。
函数依赖是多值依赖的一种特殊的情况，而多值依赖实际上是连接依赖的一种特殊情况。但连接依赖不像函数依赖和多值依赖可以由语义直接导出，而是在关系连接运算时才反映出来。存在连接依赖的关系模式仍可能遇到数据冗余及插入、修改、删除异常等问题。
第五范式处理的是无损连接问题，这个范式基本没有实际意义，因为无损连接很少出现，而且难以察觉。而域键范式试图定义一个终极范式，该范式考虑所有的依赖和约束类型，但是实用价值也是最小的，只存在理论研究中。









#### ER模型
实体关系模型

* 实体，可以看做是数据对象。矩形
	* 强实体是指不依赖于其他实体的实体
	* 弱实体是指对另一个实体有很强的依赖关系的实体
* 属性，则是指实体的特性。椭圆形
* 关系，则是指实体之间的联系。菱形

#### 数据表的设计原则
1. 数据表的个数越少越好
2. 数据表中的字段个数越少越好
3. 数据表中联合主键的字段个数越少越好
4. 使用主键和外键越多越好

#### PowerDesigner



### 数据库调优策略
定位问题
* 用户的反馈(主要)
* 日志分析(主要)
* 服务器资源使用监控
* 数据库内部状况监控  在数据库的监控中，活动会话(Active Session) 监控是一个重要的指标。
* 其它  也可以对事务、锁等待等进行监控，

#### 调优的维度和步骤

* 选择适合的DBMS
* 优化表设计
* 优化逻辑查询  SQL语句进行等价重写
* 优化物理查询  使用索引
* 使用Redis或Memcached作为缓存
* 库级优化
	* 读写分离
	* 数据分片 分库分表

#### 优化MySQL服务器

##### 硬件
1. 大内存
2. 高速磁盘
3. 合理分布磁盘IO
4. 配置多处理器  
##### 参数
 * innodb_buffer_pool_size:  InnoDB类型的表和索引的最大缓存。它不仅仅缓存索引数据，还会缓存表的数据。这个值越大，查询的速度就会越快。但是这个值太大会影响操作系统的性能。
* key_buffer_size：表示索引缓冲区的大小。索引缓冲区是所有的线程共享。增加索引缓冲区可以得到更好处理的索引（对所有读和多重写）。当然，这个值不是越大越好，它的大小取决于内存的大小。如果这个值太大，就会导致操作系统频繁换页，也会降低系统性能。对于内存在4GB左右的服务器该参数可设置为256M或384M。
* table_cache：表示同时打开的表的个数。这个值越大，能够同时打开的表的个数越多。物理内存越大，设置就越大。默认为2402，调到512-1024最哇。这个值不是越大越好，因为同时打开的表太多会影响操作系统的性能。
* query_cache_size:表示查询缓冲区的大小。可以通过在MySQL控制台观察，如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况，就要增加Query_cache_size的值；如果Qcache_hits的值非常大，则表明查询缓冲使用非常频繁，如果该值较小反而会影响效率，那么可以考虑不用查询缓存；Qcache_.free_blocks,如果该值非常大，则表明缓冲区中碎片很多。MySQL8.0之后失效。该参数需要和query_cache_type配合使用。
* query_cache_type的值是o时，所有的查询都不使用查询缓存区。但是query_.cache_.type=0并不会导致MySQL释放query_.cache_size所配置的缓存区内存。
	* 当query_cache_type=1时，所有的查询都将使用查询缓存区，除非在查询语句中指定SQL_NO_CACHE,如SELECT SQL_NO_CACHE FROM tbl_name.
	* 当query_cache_type=2时，只有在查询语句中使用SQL_CACHE关键字，查询才会使用查询缓存区。使用查询缓存区可以提高查询的速度，这种方式只适用于修改操作少且经常执行相同的查询操作的情况。
* sort_buffer_size：表示每个需要进行排序的线程分配的缓冲区的大小。增加这个参数的值可以提高ORDER BY或GR0UP BY操作的速度。默认数值是2097144字节（约2MB)。对于内存在4GB左右的服务器推荐设置为6-8M,如果有100个连接，那么实际分配的总共排序缓冲区大小为100×6=600MB。
* join_buffer_size=8M：表示联合查询操作所能使用的缓冲区大小，和sot_buffer_size一样，该参数对应的分配内存也是每个连接独享。
* read_buffer_size：表示每个线程连续扫描时为扫描的每个表分配的缓冲区的大小（字节）。当线程从表中连续读取记录时需要用到这个缓冲区。SET SESSION read_buffer_size=n可以临时设置该参数的值。默认为64K,可以设置为4M。
* innodb_flush_log_at_trx_commit：表示何时将缓冲区的数据写入日志文件，并且将日志文件写入磁盘中。该参数对于innoDB引擎非常重要。该参数有3个值，分别为0、1和2。该参数的默认值为1。
	* 值为0时，表示每秒1次的频率将数据写入日志文件并将日志文件写入磁盘。每个事务的commit并不会触发前面的任何操作。该模式速度最快，但不太安全，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失。
	* 值为1时，表示每次提交事务时将数据写入日志文件并将日志文件写入磁盘进行同步。该模式是最安全的，但也是最慢的一种方式。因为每次事务提交或事务外的指令都需要把日志写入(fus)硬盘。
	* 值为2时，表示每次提交事务时将数据写入日志文件，每隔1秒将日志文件写入磁盘。该模式速度较快，也比0安全，只有在操作系统崩溃或者系统断电的情况下，上一秒钟所有事务数据才可能丢失。
* innodb_log_buffer_size:这是InnoDB存储引擎的事务日志所使用的缓冲区。为了提高性能，也是先将信息写入Innodb Log Buffer中，当满足innodb_flush_log_trx_commit参数所设置的相应条件（或者日志缓冲区写满)之后，才会将日志写到文件（或者同步到磁盘）中。
* max_connections：表示允许连接到MySQL数据库的最大数量，默认值是151。如果状态变量connectionerrors_.max_connections不为零，并且一直增长，则说明不断有连接请求因数据库连接数已达到允许最大值而失败，这是可以考虑增大max_connections的值。在Linux平台下，性能好的服务器，支持500-1000个连接不是难事，需要根据服务器性能进行评估设定。这个连接数不是越大越好，因为这些连接会浪费内存的资源。过多的连接可能会导致MySQL服务器僵死。
* back_log:用于控制MySQL监听TCP端口时设置的积压请求栈大小。如果MySq的连接数达到max_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back_log,如果等待连接的数量超过back_log,将不被授予连接资源，将会报错。5.6.6版本之前默认值为50，之后的版本默认为50+(max_connections/5),对于Linux系统推荐设置为小于512的整数，但最大不超过900。如果需要数据库在较短的时间内处理大量连接请求，可以考虑适当增大back_log的值。
* thread_cache_size：线程池缓存线程数量的大小，当客户端断开连接后将当前线程缓存起来，当在接到新的连接请求时快速响应无需创建新的线程。这尤其对那些使用短连接的应用程序来说可以极大的提高创建连接的效率。那么为了提高性能可以增大该参数的值。默认为60，可以设置为120。
	可以通过如下几个MySQL状态值来适当调整线程池的大小：
	mysq1>show global status like 'Thread%';
* wait_ timeout :指定-个请求的最大连接时间，对于4GB左右内存的服务器可以设置为5-10。
* interactive_ timeout :表示服务器在关闭连接前等待行动的秒数。
#### 优化数据库结构
* 拆分表:冷热数据分离
* 增加中间表  需要经常联合查询的表，可以建立中间表以提高查询效率。通过建立中间表，把需要经常联合查询的数据插入中间表中，然后将原来的联合查询改为对中间表的查询，以此来提高查询效率。
	一致性
* 冗余字段
* 优化数据类型   优先选择符合存储需要的最小的数据类型。
	* 整型INT。非负型(如自增ID、 整型IP)，用无符号整型UNSIGNED
	* 既可以使用文本类型也可以使用整数类型的字段，要选择使用整数类型
	* 避免使用TEXT、BLOB数据类型
	* 避免使用ENUM类型  使用TINYINT来代替ENUM类型。
	* 使用TIMESTAMP存储时间
	* 用DECIMAL代替FLOAT和DOUBLE存储精确浮点数
* 插入优化
	* myisam
		* 禁用索引
			禁用ALTER TABLE table_ name DISABLE KEYS;
			开启ALTER TABLE table name ENABLE KEYS;
		* 禁用唯一性检查
			禁用SET UNIQUE_ CHECKS=0;
			开启SET UNIQUE _ CHECKS=1 ;
		* 批量插入 
		* 使用LOAD DATA INFILE批量导入
	* innodb
		* 禁用唯一性检查
			禁用SET UNIQUE_ CHECKS=0;
			开启SET UNIQUE _ CHECKS=1 ;
		* 禁用外键检查
			禁用SET foreign_key_checks=0;
			开启SET foreign_key_checks=1 ;
		* 禁用自动提交 
			set autocommit=0 ;
			set autocommit=1 ;
* 非空约束
* 分析表、检查表与优化表
	```sql
	分析表
	ANALYZE [ LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [,tbl_name]..
	
	检查表
	CHECK TABLE tb1_name[,tbl_name ]...[option]...
	option = {QUICK | FAST| MEDIUM| EXTENDED| CHANGED}
	QUICK:不扫描行，不检查错误的连接。
	FAST :只检查没有被正确关闭的表。
	CHANGED :只检查上次检查后被更改的表和没有被正确关闭的表。
	MEDIUM: 扫描行，以验证被删除的连接是有效的。也可以计算各行的关键字校验和，并使用计算出的校验和验证这一点。
	EXTENDED :对每行的所有关键字进行一个全面的关键字查找。这可以确保表是100%-致的，但是花的时间较长。
	option只对MyISAM类型的表有效，对InnoDB类型的表无效 
	
	优化表
	只能优化表中的VARCHAR、BLOB或TEXT类型的字段。一个表使用了这些字段的数据类型，若已经删除了表的一大部分数据，或者已经对含有可变长度行的表(含有VARCHAR、 BLOB或TEXT列的表) 进行了很多更新，则应使用OPTIMIZE TABLE来重新利用末使用的空间，并整理数据文件的碎片。
	OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_ name [,tbl_name]...
	或者
	mysqlcheck -o DatabaseName TableName -U root -p******
	mysqlcheck是Linux中的rompt, -o是代表Optimize。
	```
	使用分析表、检查表与优化表的过程中，数据库系统会自动对表加-一个只读锁。能够分析InnoDB和MyISAM类型的表，但是不能作用于视图。
	cardinality (区分度)，该值统计了表中某一键所在的列不重复的值的个数。该值越接近表中的总行数，则在表连接查询或者索引查询时，就越优先被优化器选择使用

#### 大表优化
1. 限定查询范围   禁止不带任何限制数据范围条件的查询语句。
2. 读写分离
3. 垂直拆分
4. 水平拆分
	* 客户端代理:分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。 当当网的Sharding-JDBC、阿里的TDDL是两种比较常用的实现。
	* 中间件代理:在应用和数据中间加了一 个代理层。分片逻辑统-维护在中间件服务中。我们现在谈的Mycat, 360的Atlas、 网易的DDB等等都是这种架构的实现。

#### 其他
* 服务器语句超时处理
	在MySQL 8.0中可以设置服务器语句超时的限制，单位可以达到亳秒级别。当中断的执行语句超过设置的毫秒数后，服务器将终止查询影响不大的事务或连接，然后将错误报给客户端。
	设置服务器语句超时的限制，可以通过设置系统变量^ MAX_ EXECUTION TIME来实现。 默认情况下，
	MAX_EXECUTION_TIME的值为0，代表没有时间限制。
* 创建全局通用表空间
	MySQL 8.0使用CREATE TABLESPACE 语句来创建一个全局通用表空间。全局表空间可以被所有的数据库的表共享，而且相比于独享表空间，使用手动创建共享表空间可以节约元数据方面的内存。可以在创建表的时候，指定属于哪个表空间，也可以对已有表进行表空间修改等。
* MySQL 8.0新特性:隐藏索引对调优的帮助



## 事务
### 概述
ACID
* 原子性
* 一致性
* 隔离性
* 持久性

```sql
# 显示事务
begin /
start transaction read only /read write/with consistent snapshot #一致性读

release savepoint [savepoint_name] #删除保存点  
savepoint [savepoint_name]  #保存点
rollback to [savepoint_name] #回滚到保存点

# 隐式事务
show variables like 'autocommit';  

set autocommit = false;#DML有效DDL无效
#或使用显示事务不会自动提交

#completion_type
show variables like '%completion_type%';  
set completion_type = 0 # 默认 提交一个事务 下一个还要开启
set completion_type = 1 # 提交事务开启一个同隔离级别的事务
set completion_type = 2 # 提交事务断开连接
```

#### 分类
* 扁平事务
* 带保存点的扁平事务
* 链事务   事务链  子事务提交释放不需要的数据对象
* 嵌套事务
* 分布式事务

#### 隔离级别
* 脏写Dirty Write
	A改B未提交数据
* 脏读Dirty Read
	A读B未提交数据
* 不可重复读 Non-Repeatable Read
	A读B改A再读 A读的数据不一致
* 幻读Phantom
	A读B插 A再读数据多了

* read uncommitted：读未提交   问题：脏读、不可重复读、幻读
* read committed：读已提交 （Oracle默认） 问题：不可重复读、幻读
* repeatable read：可重复读 （MySQL默认） 问题：幻读
* serializable：串行化   可以解决所有的问题
隔离级别从小到大安全性越来越高，但是效率越来越低

```sql
set global / session  transaction isolation level
set global / session  transaction_isolation = 
# read committed  
# read uncommitted  
# repeatable read  
# serializable
```

### 事务日志
* 事务的隔离性由锁机制实现。
* 而事务的原子性、一致性和持久性由事务的redo日志和undo日志来保证。
	* REDO LOG称为重做日志，提供再写入操作，恢复提交事务修改的页操作，用来保证事务的持久性。
	* UND0LOG称为回滚日志，回滚行记录到某个特定版本，用来保证事务的原子性、一致性。

REDO和UNDO都可以视为是一种恢复操作
* redo log:是存储引擎层(innodb)生成的日志，记录的是"物理级别"上的页修改操作，比如页号xxx、偏移量yyy 写入了'zzz'数据。主要为了保证数据的可靠性；
* undo log:是存储引擎层(innodb)生成的日志，记录的是逻辑操作日志，比如对某一行数据进行了INSERT语句操作，那么undo log就记录一条与之相反的DELETE操作。主要用于事务的回滚(undo log记录的是每个修改操作的逆操作)和一致性非锁定读(undo log回滚行记录到某种特定的版本--MVCC,即多版本并发控制)。

#### REDO log (持久性)
InnoDB页为单位存储 读取时从磁盘加载到Buffer pool 更新的脏页以一定频率落盘( checkpoint机制 )

将修改的数据记录到日志文件 --- redo log

InnoDB引擎的事务采用了WAL技术(Write-Ahead Logging),这种技术的思想就是先写日志，再写磁盘

好处
* redo日志降低了刷盘频率
* redo日志占用的空间非常小
* 顺序写磁盘
* 事务执行过程中，redo log不断记录
	redo log跟bin log的区别
	redo log是存储引擎层产生的，而bin log是数据库层产生的。
	假设一个事务，对表做10万行的记录插入，在这个过程中，一直不断的往redo log)顺序记录，而bin log不会记录，直到这个事务提交，才会一次写入到bin log.文件中。

##### 组成

* 重做日志的缓冲(redo log buffer),保存在内存中，是易失的。
	在服务器启动时就向操作系统申请了一大片称之为redo log buffer的连续内存空间，翻译成中文就是redo日志缓冲区。这片内存空间被划分成若干个连续的redo log block。一个redo log block占用512字节大小。
```sql
#默认16M  最大4096M 最小1M
show variables like 'innodb_log_buffer_size';
```
* 重做日志文件(redo log file),保存在硬盘中，是持久的。
	其中的ib_logfile0和ib_logfile1即为REDO日志。


注意，redo log buffer刷盘到redo log file的过程并不是真正的刷到磁盘中去，只是刷入到文件系统缓存(page cache)中去（这是现代操作系统为了提高文件写入效率做的一个优化），真正的写入会交给系统自己来决定（比如page cache足够大了)。那么对于InnoDB来说就存在一个问题，如果交给系统来同步，同样如果系统宕机，那么数据也丢失了（虽然整个系统宕机的概率还是比较小的）。

针对这种情况，InnoDB给出 innodb_flush_log_at_trx_commit 参数，该参数控制commit提交事务时，如何
将redo log buffer中的日志刷新到redo log file中。它支持三种策略：
* 0：表示每次事务提交时不进行刷盘操作。（系统默认master thread每隔1s进行一次重做日志的同步）
* 1：表示每次事务提交时都将进行同步，刷盘操作（默认值）
* 2：表示每次事务提交时都只把redo log buffer内容写入page cache,不进行同步。由os自己决定什么时候同步到磁盘文件。

###### redo log buffer

Mini-Transaction
MySQL把对底层页面中的一次原子访问的过程称之为一个Mini-Transaction,简称mtr,比如，向某个索引对
应的B+树中插入一条记录的过程就是一个Mini-Transaction。一个所谓的mtr可以包含一组redo日志，在进行崩溃恢复时这一组redo日志作为一个不可分割的整体。
一个事务可以包含若干条语句，每一条语句其实是由若干个mtr组成，每一个mtr又可以包含若干条redo日志

redo log block
一个redo log block;是由日志头、日志体、日志尾组成。日志头占用12字节，日志尾占用8字节，所以一个block.真正能存储的数据就是512-12-8=492字节。512对应机械硬盘一个扇区

###### redo log file

* innodb_log_group_home_dir:指定redo log文件组所在的路径，默认值为./，表示在数据库的数据目录下。MySQL的默认数据目录（var/lib/mysql)下默认有两个名为ib_logfile0和ib_logfile1的文件，
* innodb_log_files_in_group:指明redo log file的个数，命名方式如：ib_logfile0,iblogfile1.....iblogfilen。默认2个，最大100个。
* innodb_log_file_size:单个redo log文件设置大小，默认值为48M。最大值为512G,注意最大值指的是整个redo log系列文件之和，即(innodb_log_files_in_group* innodb_log_fle_size)不能大于最大值512G。

###### 日志文件组
循环写入日志文件组

checkpoint
* write pos是当前记录的位置，一边写一边后移
* checkpoint是当前要擦除的位置，也是往后推移,  一边落库一边后移

#### UNDO log(原子性)
作用
* 回滚
* MVVC
	InnoDB存储引擎中MVCC的实现是通过undo来完成。当用户读取一行记录时，若该记录已经被其他事务占用，当前事务可以通过undo读取之前的行版本信息，以此实现非锁定读取。

undo的存储结构
1. 回滚段与undo页
	InnoDB对undo log的管理采用段的方式，也就是回滚段(rollback segment)。每个回滚段记录了1024个undo log segment,而在每个undo log segment段中进行undo页的申请。
	* 在InnoDB1.1版本之前（不包括1.1版本），只有一个rollback segment,因此支持同时在线的事务限制为1024。虽然对绝大多数的应用来说都已经够用。
	* 从1.1版本开始InnoDB支持最大128个rollback segment,故其支持同时在线的事务限制提高到了128 * 1024。
	* 这些rollback segment都存储于共享表空间ibdata中
	* innodb_undo_directory:设置rollback segment.文件所在的路径。这意味着rollback segment可以存放在共享表空间以外的位置，即可以设置为独立表空间。该参数的默认值为“./”，表示当前InnoDB存储引擎的目录。
	* innodb-undo_logs：设置rollback segment的个数，默认值为128。在InnoDB1.2版本中，该参数用来替换之前版本的参数innodb_rollback_segments。
	* innodb_undo_tablespaces：设置构成rollback segment文件的数量，这样rollback segmenti可以较为平均地分布在多个文件中。设置该参数后，会在路径innodb_undo_directory看到undo为前缀的文件，该文件就代表rollback segment文件
	* undo log相关参数一般很少改动。

undo页的重用
当事务提交时，并不会立刻删除undo页。因为重用，所以这个undo页可能混杂着其他事务的undo log。undo logi在commit后，会被放到一个链表中，然后判断undo页的使用空间是否小于3/4,如果小于3/4的话，则表示当前的undo页可以被重用，那么它就不会被回收，其他事务的undo log可以记录在当前undo页的后面。由于undo log是离散的，所以清理对应的磁盘空间时，效率不高。

1. 回滚段与事务
	1. 每个事务只会使用一个回滚段，一个回滚段在同一时刻可能会服务于多个事务。
	2. 当一个事务开始的时候，会制定一个回滚段，在事务进行的过程中，当数据被修改时，原始的数据会被复制到回滚段。
	3. 在回滚段中，事务会不断填充盘区，直到事务结束或所有的空间被用完。如果当前的盘区不够用，事务会在段中请求扩展下一个盘区，如果所有已分配的盘区都被用完，事务会覆盖最初的盘区或者在回滚段允许的情况下扩展新的盘区来使用。
	4. 回滚段存在于undo表空间中，在数据库中可以存在多个undo表空间，但同一时刻只能使用一个undo表空间。
		show variables like innodb_undo_tablespaces';
		undo log的数量，最少为2，undo log的truncate操作有purge协调线程发起。在truncate某个undo log表空间的过程中，保证有一个可用的undo log可用。
	5. 当事务提交时，InnoDB存储引擎会做以下两件事情：
		* 将undo log放入列表中，以供之后的purge操作
		* 判断undo log所在的页是否可以重用，若可以分配给下个事务使用
2. 回滚段中的数据分类
	1. 未提交的回滚数据(uncommitted undo information)：该数据所关联的事务并未提交，用于实现读一致性，所以该数据不能被其他事务的数据覆盖。
	2. 己经提交但未过期的回滚数据(committed undo information):该数据关联的事务已经提交，但是仍受到undo retention参数的保持时间的影响。
	3. 事务已经提交并过期的数据(expired undo information):事务已经提交，而且数据保存时间已经超过undo retention参数指定的时间，属于已经过期的数据。当回滚段满了之后，会优先覆盖"事务已经提交并过期的数据"。

事务提交后并不能马上删除undo log及undo log所在的页。这是因为可能还有其他事务需要通过undo log来得到行记录之前的版本。故事务提交时将undo log)放入一个链表中，是否可以最终删除undo log及undo log所在页由purge线程来判断。
##### 类型
* insert undo log
	insert undo log是指在insert操作中产生的undo log。因为insert操作的记录，只对事务本身可见，对其他事务不可见（这是事务隔离性的要求），故该undo log可以在事务提交后直接册删除。不需要进行purge操作。
* update undo log
	update undo logi记录的是对lelete和update操作产生的undo log。.该undo logi可能需要提供MVCC机制，因此不能在事务提交时就进行删除。提交时放入undo log链表，等待purge线程进行最后的删除。

purge线程两个主要作用是：清理undo页和清除page里面带有Delete_.Bit标识的数据行。在InnoDB中，事务中的Deletej操作实际上并不是真正的删除掉数据行，而是一种Delete Markj操作，在记录上标识Delete_Bit,而不删除记录。是一种"假删除"，只是做了个标记，真正的删除工作需要后台puge线程去完成。

### 锁
隔离性
#### 数据操作类型分
##### 读锁-共享锁
```sql
select ... LOCK IN SHARE MODE
select ... FOR SHARE #8.0
```

##### 写锁-排他锁
```sql
select ... FOR UPDATE
```
在5.7及之前的版本，SELECT.FOR UPDATE,如果获取不到锁，会一直等待，直到innodb_lock_wait_timeout超时。在8.O版本中，SELECT..FOR UPDATE,SELECT ..FOR SHARE添加NOWAIT、SKIP LOCKED语法，跳过锁等待，或者跳过锁定。
* 通过添加NOWAIT、SKIP LOCKED语法，能够立即返回。如果查询的行已经加锁：
	* 那么NOWAIT会立即披错返回
	* 而SKIP LOCKED也会立即返回，只是返回的结果中不包含被锁定的行。

#### 粒度分
##### 表锁
###### 读写锁
* LOCK TABLES t READ:InnoDB存储引擎会对表t加表级别的S锁。
* LOCK TABLES t WRITE:InnoDB存储引擎会对表t加表级别的X锁。

```sql
show open tables where in_use > 0  #查锁住的表
```

###### 意向锁 intention lock
InnoDB支持多粒度锁(multiple granularity locking),它允许行级锁与表级锁共存，而意向锁就是其中的一种表锁。
1. 意向锁的存在是为了协调行锁和表锁的关系，支持多粒度（表锁与行锁）的锁并存。
2. 意向锁是一种不与行级锁冲突表级锁，这一点非常重要。
3. 表明“某个事务正在某些行持有了锁或该事务准备去持有锁”

意向锁分为两种：
* 意向共享锁(intention shared lock,IS):事务有意向对表中的某些行加共享锁(S锁)
	事务要获取某些行的S锁，必须先获得表的IS锁。
	SELECT column FROM table...LOCK IN SHARE MODE
* 意向排他锁(intention exclusive lock,IX)：事务有意向对表中的某些行加排他锁(X锁)
	事务要获取某些行的X锁，必须先获得表的工X锁。
	SELECT column FROM table...FOR UPDATE;

即：意向锁是由存储引擎自己维护的，用户无法手动操作意向锁，在为数据行加共享/排他锁之前InooDB会先获取该数据行所在数据表的对应意向锁。

给更大一级别的空间示意里面是否已经上过锁。
==给某一行数据加上了排它锁，数据库会自动给更大一级的空间，比如数据页或数据表加上意向锁，告诉其他人这个数据页或数据表已经有人上过排它锁了，==这样当其他人想要获取数据表排它锁的时候，只需要了解是否有人已经获取了这个数据表的意向排他锁即可。
* 如果事务想要获得数据表中某些记录的共享锁，就需要在数据上添加意向共享锁。
* 如果事务想要获得数据表中某些记录的排他锁，就需要在数据表上添加意向排他锁。
这时，意向锁会告诉其他事务已经有人锁定了表中的某些记录。

1. InnoDB支持多粒度锁，特定场景下，行级锁可以与表级锁共存。
2. 意向锁之间互不排斥，但除了S与S兼容外，意向锁会与共享锁/排他锁互斥。
3. X,IS是表级锁，不会和行级的X,S锁发生冲突。只会和表级的X,S发生冲突。
4. 意向锁在保证并发性的前提下，实现了行锁和表锁共存且满足事务隔离性的要求。

###### 自增锁(AUTO-INC锁)
为某个列添加AUTO_INCREMENT属性

插入数据的方式总共分为三类，分别是“Simple inserts”,“Bulk inserts”和“Mixed-mode inserts”。

1. “Simple inserts”(简单插入)
	可以预先确定要插入的行数（当语句被初始处理时）的语句。包括没有嵌套子查询的单行和多行INSERT...VALUES()和REPLACE语句。比如我们上面举的例子就属于该类插入，已经确定要插入的行数。
1. “Bulk inserts'”(批量插入)
	事先不知道要插入的行数（和所需自动递增值的数量）的语句。比如INSERT,··SELECT,REPLACE..SELECT和LOAD DATA语句，但不包括纯INSERT。InnoDB在每处理一行，为AUTO_INCREMENT列分配一个新值。
3. “Mixed-mode inserts'”(混合模式插入)
	这些是“Simple inserts"”语句但是指定部分新行的自动递增值。例如INSERT INTO teacher(id,name)VALUES(1,'a'),(NULL,'b'),(5,'c'),(NULL,'d');只是指定了部分id的值。另一种类型的“混合模式插入”是INSERT···ON DUPLICATE KEY UPDATE。

对于上面数据插入的案例，MySQL中采用了自增锁的方式来实现，AUTO-INC锁是当向使用含有AUTO_INCREMENT列的表中插入数据时需要获取的一种特殊的表级锁，在执行插入语句时就在表级别加一个AUTO-INC锁，然后为每条待插入记录的AUTO_INCREMENT修饰的列分配递增的值，在该语句执行结束后，再把AUTO-INC锁释放掉。一个事务在持有AUTO-NC锁的过程中，其他事务的插入语句都要被阻塞，可以保证一个语句中分配的递增值是连续的。也正因为此，其并发性显然并不高，当我们向一个有AUTO_INCREMENT关键字的主键插入值的时候，每条语句都要对这个表锁进行竞争，这样的并发潜力其实是很低下的，所以innodb通过innodb_autoinc_lock_mode的不同取值来提供不同的锁定机制，来显著提高SQL语句的可伸缩性和性能。

innodb_autoinc_lock_mode有三种取值，分别对应与不同锁定模式：
* 0   传统”锁定模式
	表级锁 限制并发能力
* 1   连续"锁定模式
	在MySQL8.0之前，连续锁定模式是默认的。
	在这个模式下，“bulk inserts'”仍然使用AUTO-INC表级锁，并保持到语句结束。这适用于所有INSERT.SELECT, REPLACE..SELECT和LOAD DATA语句。同一时刻只有一个语句可以持有AUTO-INC锁。

	对于“Simple inserts'”(要插入的行数事先已知)，则通过在mutex(轻量锁)的控制下获得所需数量的自动递增值来避免表级AUTO-INC锁，它只在分配过程的持续时间内保持，而不是直到语句完成。不使用表级AUTO-NC锁，除非AUTO-INc锁由另一个事务保持。如果另一个事务保持AUTO-INc锁，则“Simple inserts”等待AUTO-INc锁，如同它是一个“bulk inserts”。
* 2   交错"锁定模式
	从MySQL8.0开始，交错锁模式是默认设置。
	在这种锁定模式下，所有类INSERT语句都不会使用表级AUTO-INC锁，并且可以同时执行多个语句。这是最快和最可扩展的锁定模式，但是当使用基于语句的复制或恢复方案时，从二进制日志重播SQL语句时，这是不安全的。

	在此锁定模式下，自动递增值保证在所有并发执行的所有类型的iset语句中是唯一且单调递增的。但是，由于多个语句可以同时生成数字（即，跨语句交叉编号），为任何给定语句插入的行生成的值可能不是连续的。

	如果执行的语句是“simple inserts”,其中要插入的行数已提前知道，除了“Mixed-mode inserts”之外，为单个语句生成的数字不会有间隙。然而，当执行“bulk inserts”时，在由任何给定语句分配的自动递增值中可能存在间隙。

###### 元数据锁(MDL锁)
MySQL5.5引入了meta data lock,简称MDL锁，属于表锁范畴。MDL的作用是，保证读写的正确性。比如，如果一个查询正在遍历一个表中的数据，而执行期间另一个线程对这个表结构做变更，增加了一列，那么查询线程拿到的结果跟表结构对不上，肯定是不行的。

因此，==当对一个表做增删改查操作的时候，加MDL读锁；当要对表做结构变更操作的时候，加MDL写锁。==
读锁之间不互斥，因此你可以有多个线程同时对一张表增删改查。读写锁之间、写锁之间是互斥的，用来保证变更表结构操作的安全性，解决了DML和DDL操作之间的一致性问题。不需要显式使用，在访问一个表的时候会被自动加上。

##### 页锁
页锁的开销介于表锁和行锁之间，会出现死锁。锁定粒度介于表锁和行锁之间，并发度一般。

每个层级的锁数量是有限制的，因为锁会占用内存空间，锁空间的大小是有限的。当某个层级的锁数量超过了这个层级的阈值时，就会进行锁升级。锁升级就是用更大粒度的锁替代多个更小粒度的锁，比如InnoDB中行锁升级为表锁，这样做的好处是占用的锁空间降低了，但同时数据的并发度也下降了。

##### 行锁
存储引擎层实现
优点：锁定粒度小，发生锁冲突概率低，可以实现的并发度高。
缺点：对于锁的开销比较大，加锁会比较慢，容易出现死锁情况。

###### 记录锁
LOCK_REC_NOT_GAP    加载一条记录上
记录读写锁


###### 间隙锁 Gap Locks
MySQL在REPEATABLE READ隔离级别下是可以解决幻读问题的，解决方案有两种，可以使用MVCC方案解决，也可以采用加锁方案解决。但是在使用加锁方案解决时有个大问题，就是事务在第一次执行读取操作时，那些幻影记录尚不存在，我们无法给这些幻影记录加上记录锁。InnoDB提出了一种称之为Gap Locks的锁，官方的类型名称为：LOCK_GAP,我们可以简称为gap锁。

gap锁的提出仅仅是为了防止插入幻影记录而提出的。虽然有共享gap锁和独占gap锁这样的说法，但是它们起到的作用是相同的。而且如果对一条记录加了gap锁（不论是共享gap锁还是独占gap锁），并不会限制其他事务对这条记录加记录锁或者继续加gap锁。

想要锁(20，+∞)   这时候我们在讲数据页时介绍的两条伪记录派上用场了：
* Infimum记录，表示该页面中最小的记录
* Supremum记录，表示该页面中最大的记录。
20到Supremum记录加上一个gap锁，

会有死锁问题

###### 临键锁(Next-Key Locks)
有时候我们既想锁住某条记录，又想阻止其他事务在该记录前边的间隙插入新记录，所以InnoDB就提出了一种称之为Next-Key Locks的锁，官方的类型名称为：LOCK_ORDINARY,我们也可以简称为next-key锁。Next-Key Locks是在存储引擎innodb、事务级别在可重复读的情况下使用的数据库锁，innodb默认的锁就是Next-Key locks。

next-key锁的本质就是一个记录锁和一个gap锁的合体，它既能保护该条记录，又能阻止别的事务将新记录插入被保护记录前边的间隙。

###### 插入意向锁(Insert Intention Locks)
我们说一个事务在插入一条记录时需要判断一下插入位置是不是被别的事务加了gap锁(next-key锁也包，含gap锁)，如果有的话，插入操作需要等待，直到拥有gap锁的那个事务提交。但是InnoDB规定事务在等待的时候也需要在内存中生成一个锁结构，表明有事务想在某个间隙中插入新记录，但是现在在等待。IoDB就把这种类型的锁命名为Insert Intention Locks,官方的类型名称为：LOCK_INSERT_INTENTION,我们称为插入意向锁。插入意向锁是一种Gap锁，不是意向锁，在insert操作时产生。

插入意向锁是在插入一条记录行前，由INSERT操作产生的一种间隙锁。该锁用以表示插入意向，当多个事务在同一区间(gap)插入位置不同的多条数据时，事务之间不需要互相等待。假设存在两条值分别为4和7的记录，两个不同的事务分别试图插入值为5和6的两条记录，每个事务在获取插入行上独占的（排他）锁前，都会获取(4,7)之间的间隙锁，但是因为数据行之间并不冲突，所以两个事务之间并不会产生冲突（阻塞等待）。

总结来说，插入意向锁的特性可以分成两部分：
* 插入意向锁是一种特殊的间隙锁一一间隙锁可以锁定开区间内的部分记录。
* 插入意向锁之间互不排斥，所以即使多个事务在同一区间插入多条记录，只要记录本身（主键、唯一索引)不冲突，那么事务之间就不会出现冲突等待。

注意，虽然插入意向锁中含有意向锁三个字，但是它并不属于意向锁而属于间隙锁，因为意向锁是表锁而插入意向锁是行锁。

插入意向锁并不会阻止别的事务继续获取该记录上任何类型的锁。

#### 态度分

1. 悲观锁(Pessimistic Locking)
	行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁，当其他线程想要访问数据时，都需要阻塞挂起。
2. 乐观锁(Optimistic Locking)
	通过程序来实现。在程序上，我们可以采用版本号机制或者CAS机制实现。乐观锁适用于多读的应用类型，
	1. 乐观锁的版本号机制
		版本字段version,第一次读的时候，会获取version字段的取值。然后对数据进行更新或删除操作时，会执行UPDATE··SET version=version+1 WHERE version=version。此时如果已经有事务对这条数据进行了更改，修改就不会成功。
		这种方式类似我们熟悉的SVN、CVS版本管理系统，当我们修改了代码进行提交时，首先会检查当前版本号与服务器上的版本号是否一致，如果一致就可以直接提交，如果不一致就需要更新服务器上的最新代码，然后再进行提交。
	2. 乐观锁的时间戳机制
		时间戳和版本号机制一样，也是在更新提交的时候，将当前数据的时间戳和更新之前取得的时间戳进行比较，如果两者一致则更新成功，否则就是版本冲突。

#### 加锁的方式分

##### 隐式锁
一个事务在执行INSERT操作时，如果即将插入的间隙已经被其他事务加了gap锁，那么本次INSERT操作会阻塞，并且当前事务会在该间隙上加一个插入意向锁，否则一般情况下INSERT操作是不加锁的。

那如果一个事务首先插入了一条记录（此时并没有在内存生产与该记录关联的锁结构），然后另一个事务：
* 立即使用SELECT·,·LOCK IN SHARE MODE语句读取这条记录，也就是要获取这条记录的S锁，或者使用SELECT,.·FOR UPDATE语句读取这条记录，也就是要获取这条记录的X锁，怎么办？
	如果允许这种情况的发生，那么可能产生脏读问题。
* 立即修改这条记录，也就是要获取这条记录的X锁，怎么办？
	如果允许这种情况的发生，那么可能产生脏写问题。

这时候我们前边提过的事务d又要起作用了。我们把聚簇索引和二级索引中的记录分开看一下：
* 情景一：对于聚簇索引记录来说，有一个trx_Id隐藏列，该隐藏列记录着最后改动该记录的事务Id。那么如果在当前事务中新插入一条聚簇索引记录后，该记录的trx_Id隐藏列代表的的就是当前事务的事务Id,如果其他事务此时想对该记录添加S锁或者X锁时，首先会看一下该记录的tr×_Id隐藏列代表的事务是否是当前的活跃事务，如果是的话，那么就帮助当前事务创建一个X锁（也就是为当前事务创建一个锁结构is-waiting属性是false),然后自己进入等待状态（也就是为自己也创建一个锁结构，is_waiting属性是true)。
* 情景二：对于二级索引记录来说，本身并没有trx_id隐藏列，但是在二级索引页面的Page Header部分有一个PAGE_MAX_TRX_ID属性，该属性代表对该页面做改动的最大的事务id,如果PAGE_MAX_TRX_ID属性值小于当前最小的活跃事务Id,那么说明对该页面做修改的事务都已经提交了，否则就需要在页面中定位到对应的二级索引记录，然后回表找到它对应的聚簇索引记录，然后再重复情景一的做法。

即：一个事务对新插入的记录可以不显式的加锁（生成一个锁结构），但是由于事务Id的存在，相当于加了一个隐式锁。别的事务在对这条记录加S锁或者X锁时，由于隐式锁的存在，会先帮助当前事务生成一个锁结构，然后自己再生成一个锁结构后进入等待状态。隐式锁是一种==延迟加锁==的机制，从而来减少加锁的数量。
隐式锁在实际内存对象中并不含有这个锁信息。只有当产生锁等待时，隐式锁转化为显式锁。

##### 显式锁
通过特定的语句进行加锁，我们一般称之为显示加锁

#### 全局锁

全局锁就是对整个数据库实例加锁。当你需要让整个库处于只读状态的时候，可以使用这个命令，之后其他线程的以下语句会被阻塞：数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句。全局锁的典型使用场景是：做全库逻辑备份。

全局锁的命令：
Flush tables with read lock

#### 死锁

1. 等待，直到超时(innodb_lock_wait_timeout=:50s)。
	即当两个事务互相等待时，当一个事务等待时间超过设置的阈值时，就将其回滚，另外事务继续进行。这种方法简单有效，在innodb中，参数innodb_lock_wait_timeout用来设置超时时间。
	缺点：对于在线服务来说，这个等待时间往往是无法接受的。
	那将此值修改短一些，比如1s,0.1s是否合适？不合适，容易误伤到普通的锁等待。
2. 使用死锁检测进行死锁处理
	方式1检测死锁太过被动，innodb还提供了wait-for graph算法来主动进行死锁检测，每当加锁请求无法立即满足需要并进入等待时，wait-for graph:算法都会被触发。
	这是一种较为主动的死锁检测机制，要求数据库保存锁的信息链表和事务等待链表两部分信息。

	一旦检测到回路、有死锁，这时候InnoDB存储引擎会选择回滚undo量最小的事务，让其他事务继续执行(`innodb_deadlock_detect:=on`表示开启这个逻辑)。

	缺点：每个新的被阻塞的线程，都要判断是不是由于自己的加入导致了死锁，这个操作时间复杂度是O(n)。如果100个并发线程同时更新同一行，意味着要检测100* 100=1万次，1万个线程就会有1千万次检测。
	1. 关闭死锁检测，但意味着可能会出现大量的超时，会导致业务有损。
	2. 控制并发访问的数量。比如在中间件中实现对于相同行的更新，在进入引擎之前排队，这样在nnoDB内部就不会有大量的死锁检测工作。
	3. 可以考虑通过将一行改成逻辑上的多行来减少锁冲突。

如何避免死锁
* 合理设计索引，使业务SQL尽可能通过索引定位更少的行，减少锁竞争。
* 调整业务逻辑SQL执行顺序，避免update/delete长时间特有锁的SQL在事务前面.
* 避免大事务，尽量将大事务拆成多个小事务来处理，小事务缩短锁定资源的时间，发生锁冲突的几率也更小。
* 在并发比较高的系统中，不要显式加锁，特别是是在事务里显式加锁。如select..for update语句，如果是在事务里运行了start transaction或设置了autocommit等于0，那么就会锁定所查找到的记录。
* 降低隔离级别。如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从R调整为RC,可以避免掉很多因为gap锁造成的死锁。

#### 锁的内存结构

对不同记录加锁时，如果符合下边这些条件的记录会放到一个锁结构中。
* 在同一个事务中进行加锁操作
* 被加锁的记录在同一个页面中
* 加锁的类型是一样的
* 等待状态是一样的

InnoDB存储擎中的锁结构如下：
![](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230104143823.png)

1. 锁所在的事务信息：
	不论是表锁还是行锁，都是在事务执行过程中生成的，哪个事务生成了这个锁结构，这里就记录这个事务的信息。
	此锁所在的事务信息在内存结构中只是一个指针，通过指针可以找到内存中关于该事务的更多信息，比方说事务id等。
2. 索引信息：
	对于行锁来说，需要记录一下加锁的记录是属于哪个索引的。这里也是一个指针。
3. 表锁/行锁信息：
	表锁结构和行锁结构在这个位置的内容是不同的：
	* 表锁：
		记载着是对哪个表加的锁，还有其他的一些信息。
	* 行锁：
		记载了三个重要的信息：
		* Space ID:记录所在表空间。
		* Page Number:记录所在页号。
		* n_bits：对于行锁来说，一条记录就对应着一个比特位，一个页面中包含很多记录，用不同的比特位来区分到底是哪一条记录加了锁。为此在行锁结构的末尾放置了一堆比特位，这个nbits属性代表使用了多少比特位。nbits的值一般都比页面中记录条数多一些。主要是为了之后在页面中插入了新记录后也不至于重新分配锁结构
4. type_mode
	这是一个32位的数，被分成了lock_mode、lock_type和rec_lock_type三个部分，如图所示：
	![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230104160306.png)
	* 锁的模式(lock_ mode)，占用低4位，可选的值如下:
		* LOCK Is (十进制的0) :表示共享意向锁，也就是IS锁。
		* LOCK_ IX (十进制的1) :表示独占意向锁，也就是IX锁。
		* LOCK_ S (十进制的2) :表示共享锁，也就是S锁。
		* LOCK_ _X (十进制的3) :表示独占锁，也就是X锁。
		* LOCK_ AUTO_INC (十进制的4) :表示AUTO-INC锁。
		在InnoDB存储引擎中，LOCK_IS, LOCK_IX, LOCK_AUTO_INC都算是表级锁的模式，LOCK_S和LOCK_X既可以算是表级锁的模式，也可以是行级锁的模式。
	* 锁的类型(lock_type)，占用第5 ~8位，不过现阶段只有第5位和第6位被使用:
		* LOCK_TABLE (十进制的16) ，也就是当第5个比特位置为1时，表示表级锁。
		* LOCK_REC (十进制的32)，也就是当第6个比特位置为1时，表示行级锁。
	* 行锁的具体类型( rec_lock_type) ，使用其余的位来表示。只有在lock_ type的值为LOCK_ REC时，也就是只有在该锁为行级锁时，才会被细分为更多的类型: 
		* LOCK_ ORDINARY (十进制的0) :表示next-key锁。
		* LOCK_ GAP (十进制的512) :也就是当第10个比特位置为1时，表示gap锁。
		* LOCK_ REC_NOT_GAP (十进制的1024) :也就是当第11个比特位置为1时，表示正经记录锁。
		* LOCK_INSERT_INTENTION (十进制的2048) :也就是当第12个比特位置为1时，表示插入意向锁。其他的类型:还有一一些不常用的类型我们就不多说了。
	* is_waiting属性呢?基于内存空间的节省，所以把is_waiting 属性放到了type_mode 这个32位的数字中:
		* LOCK_ WAIT (十进制的256) :当第9个比特位置为1时表示is. waiting为true，也就是当前事务尚未获取到锁，处在等待状态;当这个比特位为0时，表示is_ waiting 为false ,也就是当前事务获取锁成功。
5. 其他信息:
	为了更好的管理系统运行过程中生成的各种锁结构而设计了各种哈希表和链表。
6. 一堆比特位:
	如果是行锁结构的话，在该结构末尾还放置了一堆比特位， 比特位的数量是由上边提到的n_ bits 属性表示的。InnoDB数据页中的每条记录在记录头信息中都包含一个 heap- no属性，伪记录Infimum的heap_ no值为0，Supremum的heap- no值为1，之后每插入-条记录，heap- no值就增1。锁结构最后的一堆比特位就对应着一个页面中的记录，-个比特位映射一个heap_ no,即一个比特位映射到页内的一条记录。
#### 锁监控
```sql
show status like '%innodb_row_lock%'
Innodb_row_lock_current_waits   #当前锁
Innodb_row_lock_time             #总时长
Innodb_row_lock_time_avg
Innodb_row_lock_time_max
Innodb_row_lock_waits             #总等待


information_schema.INNODB_TRX  
information_schema.INNODB_LOCK_WAITS  
information_schema.INNODB_LOCKS
#8.0新
performance_schema.data_locks
performance_schema.data_lock_waits
```

### MVCC
多版本并发控制

MVCC (Multiversion Concurrency Control)，多版本并发控制。顾名思义，MVCC 是通过数据行的多个版本管理来实现数据库的并发控制。这项技术使得在InnoDB的事务隔离级别下执行一致性读 操作有了保证。换言之，就是为了查询一些正在被另一个事务更新的行，并且可以看到它们被更新之前的值，这样在做查询的时候就不用等待另一个事务释放锁。

MVCC没有正式的标准，在不同的DBMS中MVCC的实现方式可能是不同的，也不是普遍使用的(大家可以参考相关的DBMS文档)。这里讲解InnoDB中MVCC的实现机制(MySQL其它的存储引擎并不支持它)。

==MVCC可以不采用锁机制，而是通过乐观锁的方式来解决不可重复读和幻读问题==
它可以在大多数情况下替代行级锁，降低系统的开销。

* 解决读写之间阻塞的问题
* 降低了死锁的概率。



#### 快照读与当前读
MVCC在MySQL InnoDB中的实现主要是为了提高数据库并发性能，用更好的方式去处理读一写冲突，做到即使有读写冲突时，也能做到不加锁，非阻塞并发读，而这个读指的就是快照读，而非当前读。当前读实际上是一种加锁的操作，是悲观锁的实现。而MVCC本质是采用乐观锁思想的一种方式。

##### 快照读
快照读又叫一致性读，读取的是快照数据。不加锁的简单的SELECT都属于快照读，即不加锁的非阻塞读;

之所以出现快照读的情况，是基于提高并发性能的考虑，快照读的实现是基于MVCC，它在很多情况下，避免了加锁操作，降低了开销。

既然是基于多版本，那么快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本。

快照读的前提是隔离级别不是串行级别，串行级别下的快照读会退化成当前读。

##### 当前读
当前读读取的是记录的最新版本(最新数据，而不是历史版本的数据)，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。加锁的SELECT,或者对数据进行增删改都会进行当前读。

#### 原理 ReadView


隐藏字段、Undo Log版本链,   ReadView
* trx_id:  记录事务id
* roll_ pointer : 指向undo日志中修改前版本

历史记录在undo log ReadView决定读哪个

ReadView就是事务A在使用MVCC机制进行快照读操作时产生的读视图。当事务启动时，会生成数据库系统当前的一个快照，InnoDB为每个事务构造了-个数组，用来记录并维护系统当前活跃事务的ID (“活跃’ 指的就是，启动了但还没提交)。

##### 设计思路
使用READ COMMITTED 和REPEATABLE READ‘ 隔离级别的事务，都必须保证读到已经提交了的事务修改过的记录。假如另一个事务已经修改了记录但是尚未提交，是不能直接读取最新版本的记录的，核心问题就是需要判断一下版本链中的哪个版本是当前事务可见的，这是ReadView要解决的主要问题。

ReadView中主要包含4个比较重要的内容，分别如下: 
1. creator_trx_id,创建这个Read View的事务ID。
	说明:只有在对表中的记录做改动时(执行INSERT、 DELETE、 UPDATE这些语句时) 才会为事务分配事务id,否则在一个只读事务 中的事务id值都默认为0。
2. trx_ids，表示在生成ReadView时当前系统中活跃的读写事务的事务id列表。
3. up_limit_id,活跃的事务中最小的事务ID。
4. low_limit_id,表示生成ReadView时系统中应该分配给下一个事务的id值。low_limit_id 是系统最大的事务id值，这里要注意是系统中的事务id,需要区别于正在活跃的事务ID。

注意: low_limit_id并不是trx_ ids中的最大值，事务id是递增分配的。比如，现在有id为1， 2, 3这三个事
务，之后id为3的事务提交了。那么一个新的读事务在生成ReadView时，trx_ jids就包括1和2，up_ limit id
的值就是1，low_ limit_ id的值就是4。

##### 规则

1. trx_id = creator_trx_id 直接访问
2. trx_id < up_limit_id    此记录在ReadView生成前已提交  直接访问
3. trx_id >= low_limit_id  此记录在ReadView生成后开启事务 不能访问
4. up_limit_id < trx_id < low_limit_id    判断trx_id是否在trx_ids中 
	* 在 事务活跃 不能访问
	* 不在 可以访问

##### MVCC整体操作流程

1. 首先获取事务自己的版本号，也就是事务ID;
2. 获取ReadView;
3. 查询得到的数据，然后与ReadView中的事务版本号进行比较;
4. 如果不符合ReadView规则，就需要从Undo Log中获取历史快照;
5.  最后返回符合规则的数据。

隔离级别为读已提交(Read Committed) 时，-个事务中的每一 次SELECT查询都会重新获取一次ReadView。

当隔离级别为可重复读的时候，就避免了不可重复读这是因为一个事务只在第一次 SELECT的时候会获取一次Read View,而后面所有的SELECT都会复用这个Read View


## 日志与备份

* **慢查询日志**:记录所有执行时间超过long_query_time的所有查询， 方便我们对查询进行优化。
* **通用查询日志**:记录所有连接的起始时间和终止时间，以及连接发送给数据库服务器的所有指令，对我们复原操作的实际场景、发现问题，甚至是对数据库操作的审计都有很大的帮助。
* **错误日志**:记录MySQL服务的启动、运行或停l止MySQL服务时出现的问题， 方便我们了解服务器的状态，从而对服务器进行维护。
* **二进制日志**:记录所有更改数据的语句，可以用于主从服务器之间的数据同步，以及服务器遇到故障时数据的无损失恢复。
* **中继日志**:用于主从服务器架构中，从服务器用来存放主服务器二进制日志内容的一个中间文件。从服务器通过读取中继日志的内容，来同步主服务器上的操作。
* **数据定义语句日志**:记录数据定义语句执行的元数据操作。

弊端
* 影响性能
* 占用磁盘

### 通用查询日志(general query log)
通用查询日志用来记录用户的所有操作
```sql
show variables like '%general%'

#临时开启
set global general_log = off;  
set global general_log_file =

#修改my.cnf或者my.ini配置文件来设置
[mysqld]
general_log=ON
general_log_file= [path[filename]]


#刷新通用日志
mysqladmin -uroot -p flush-logs
```

### error log
默认开启无法关闭
默认linux mysql.log  mac hostname.err   改路径文件名在my.cnf/my.ini中改log-error=
```shell
install -omysql -gmysql -m0644 /dev/nu1l /var /log/mysqld. log
mysqladmin -uroot -p flush-logs
#刷新日志

show variables like '%log_error%';
```
### bin log
记录所有变更操作
* 数据恢复
* 数据复制

```sql
show variables like '%log_bin%';

#修改MySQL的my.cnf或my.ini文件可以设置二进制日志的相关参数：
[mysqld]
#启用二进制日志
log-bin=atguigu-bin
binlog_expire_logs_seconds=600 #秒 默认30天
max_binlog_size=100M   #最大和默认1G 不能严格控制文件大小


show BINARY logs

show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset, ] row_ count] ;
●IN 'log_ name':指定要查询的binlog文件名(不指定就是第一个binlog文件)
●FROM pos:指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
●LIMIT [offset] :偏移量(不指定就是0)
●row_ count :查询总条数(不指定就是所有行)
```

```shell
mysqlbinlog -v --base64-output=DECODE-ROWS + 文件  #查看bin log
#可查看参数帮助
mysqlbin1og --no-defaults --help
#查看最后100行
mysqlbinlog --no-defaults --base64-output=decode-rows -vv atguigu-bin. 00002 |tail 100
#根据position查找 
mysqlbinlog --no-defaults --base64-output=decode-rows -vv atguigu-bin.00002 | grep -A 20 '4939002'
```


格式
* Statement
	每一条会修改数据的sqI都会记录在binlog中。
	优点:不需要记录每一行的变化，减少了binlog日志量，节约了I0， 提高性能。
* Row
	5.1.5版本的MySQL才开始支持row level的复制，它不记录sql语句上下文相关信息，仅保存哪条记录被修改。
	优点: row level的日志内容会非常清楚的记录下每一行数据修改的细节。 而且不会出现某些特定情况下的存储过程，或function,以及trigger的调用和触发无法被正确复 制的问题。
* Mixed
	从5.1.8版本开始，MySQL提供了Mixed格式，实际上就是Statement与Row的结合。

#### 数据恢复

```sql
flush logs
```

```shell
/usr/bin/mysqlbinlog --start-position=464 --stop-position=1308 --database=atguigu14
/var/lib/mysql/binlog/atguigu-bin.00005 | /usr/bin/mysql -uroot -pabc123 -v atguigu14
#数据恢复 基于show binlog events

#数据恢复 基于mysqlbinlog 时间
/usr/bin/mysqlbinlog --start-datetime="2222-12-22 12:12:12" --stop-datetime="2222-12-22 12:21:22" --database=atguigu14
/var/lib/mysql/binlog/atguigu-bin.00005 | /usr/bin/mysql -uroot -pabc123 -v atguigu14
```

#### 删除
```sql
#删部分
PURGE {MASTER| BINARY} LOGS TO ' 指定日志文件名'
PURGE {MASTER| BINARY} LOGS BEFORE ' 指定日期'

#删除全部
reset master
```












## mysql安装

```shell
ps -ef|grep mysql   #查询所安装程序
#查询已安装
rpm -ivh MySQL-server-5.5.48-1.1inux2.6.i386.rpm
#查询命令：
rpm -qa|grep mysql
#删除命令：
rpm -eRPM软件包名（该名字是上一个命令查出来的名字）

mysqladmin --version
cat /etc/group|grep mysq1
cat /etc/passwd|grep mysq1

#启动
service mysql start
service mysql stop

top
free
iostat
vmstat

#设置开机自启动
chkconfig mysql on
chkconfig -list|grep mysql
#查看开机自启服务
ntsysv

```

| 路径                          | 解释                      | 备注                             |
| ----------------------------- | ------------------------- | -------------------------------- |
| /var/lib/mysq/                | mysql数据库文件的存放路径 | /var/lib/mysql/atguigu.cloud.pid |
| /usr/share/mysql    etc/mysql | 配置文件目录              | mysql.server命令及配置文件       |
| /usr/bin    /usr/sbin         | 相关命令目录              | mysqladmin mysqldump等命令       |
| /etc/init.d/mysql             | 启停相关脚本              |                                  |

### 配置文件

主要配置文件

- 二进制日志    log-bin       主从复制
- 错误日志log-error            默认是关闭的，记录严重的警告和错误信息，每次启动和关闭的详细信息等
- 查询日志      默认关闭，记录查询的sql语句，如果开启会减低mysql的整体性能
- 数据文件      /var/lib/mysq/

- - - - frm文件   存放表结构
      - myd文件  表数据
      - myi文件    表索引

- 如何配置 

- - - windows   my.ini文件
    - Linux       /etc/my.cnf文件

```shell
ls -lF|grep ^d  #正则查d开头的目录

#复制配置文件
cp   /usr/share/mysql/my-huge.cnf     /etc/my.cnf  //5.5
cp  /usr/share/mysql/my-default.cnf   /etc/my.cnf   //5.6
#查看字符集
show variables like'character%'；
show variables like'%char%；
#修改字符集 （建库前修改）
vim 文件名     //修改
o             下面插入一行
esc  -> :w 保存但不退出
        :wq 保存并退出
        :q 退出
        :q! 强制退出，不保存
        :e! 放弃所有修改，从上次保存文件开始再编辑命令历史
```

![image-20210914163254582](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210914163254582.png)

## mysql调优

1. 选择最合适的字段属性：类型、长度、是否允许NULL等；尽量量把字段设为not null
2. 要尽量避免全表扫描，⾸先应考虑在 where 及 order by 涉及的列上建立索引。
3. 应尽量避免在 where 子句中对字段进行 null 值判断、使用!= 或 <> 操作符，否则将导致引擎放弃使用索引而进行全表扫描
4. 应尽量避免在 where 子句中使用 or 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描
5. in 和 not in 也要慎用，否则会导致全表扫描
6. 模糊查询也将导致全表扫描，若要提高效率，可以考虑字段建立前置索引或用全文检索；
7. 如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。
8. 应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。
9. 不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
10. 在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
11. 不要写一些没有意义的查询，如需要生成一个空表结构：
12. Update 语句，如果只更更改1、2个字段，不要Update全部字段，否则频繁调用会引起明显的性能消耗，同时带来⼤大量日志。
13. 对于多张大数据量（这⾥几百条就算⼤大了了）的表JOIN，要先分页再JOIN，否则逻辑读会很高，性能很差。
14. select count(*) from table；这样不不带任何条件的count会引起全表扫描，并且没有任何业务意义，是一定要杜绝的。
15. 索引并不不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为
    insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况⽽而定。一个表的索引数最好不不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。
16. 应尽可能的避免更新 clustered 索引数据列列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列列，那么需要考虑是否应将该索引建为 clustered 索引。
17. 尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
18. 尽可能的使⽤用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。
19. 任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。
20. 尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。

22. 避免频繁创建和删除临时表，以减少系统表资源的消耗。临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件， 最好使用导出表。
23. 在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，
    以提⾼高速度；如果数据量不⼤，为了缓和系统表的资源，应先create table，然后insert。
24. 如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样
    可以避免系统表的较长时间锁定。
25. 尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万⾏，那么就应该考虑改写。
26. 使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效。
27. 与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时 间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。
28. 在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。
29. 尽量避免大事务操作，提高系统并发能力。
30. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。



## mysql优化

### 索引

==是高效获取数据的数据结构==

==排好序的快速查找数据结构==

我们平常所说的索引，如果没有特别指明，都是指B树(多路搜索树，并不一定是二叉的)结构组织的索引。其中聚集索引，次要索引，覆盖索引，复合索引,前缀索引，唯一索引默认都是使用B+树索引，统称索引。
自然，除了B+树这种类型的索引之外，还有哈稀索引(hash index)等。



**优势**

提高select速度, 减少io和cpu消耗

**劣势**

额外占用空间, 增删改减慢, 需要优化

#### mysql索引分类

##### 单值索引 

一个索引一列, 一个表可有多个

##### 唯一索引

索引列唯一,可以有null

##### 复合索引

一个索引多列

#### mysql索引结构

1. B树（多路搜索树，并不一定是二叉的）
2. 哈稀索引（hash index）
3. full-text全文索引
4. R-Tree索引

#### 基本语法

```sql
--新建
CREATE [UNIQUE] INDEX indexName ON mytable(columnname(length));+
ALTER mytable ADD [UNIQUE] INDEX [indexName]ON(columnname(length))
--删
DROP INDEX [indexName]ON mytable; 
--查
SHOW INDEX FROM table_name\G

ALTER TABLE tbl_name ADD PRIMARY KEY（column_list）
--该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL

ALTER TABLE tl_name ADD UNIQUE index_name（column_list）
--这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）

ALTER TABLE tbl_name ADD INDEX index_name（column_list）
--添加普通索引，索引值可出现多次。

ALTER TABLE tbL_name ADD FULLTEXT index_name（column_list）
--该语句指定了索引为FULLTEXT，用于全文索引。
```

#### **创建索引条件**

1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段应该创建索引
3. 查询中与其它表关联的字段，外键关系建立索引
4. 频繁更新的字段不适合创建索引e因为每次更新不单单是更新了记示
5. Where条件里用不到的字段不创建索引
6. 单键/组合索引的选择问题，who？（在高并发下倾向创建组合索引）
7. 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
8. 查询中统计或者分组字段

**不建情况**

1. 表记录太少
2. 经常增删改的表
3. 重复过多的列

#### 索引失效

尽可能全值匹配/最佳左前缀

1. 索引列上做任何操作（计算、函数、（自动or手动）类型转换）

2. 多列索引顺序不符
3. 范围判断之后失效
4. select *      （使用覆盖索引） 
5. !=和<>会失效 
6. is null / is not null
7. like（'%abc'）        (==使用覆盖索引解决==)
8. 少用or
9. ==varchar不写单引号==    （索引失效行锁变表锁）

#### 回表

```shell
	表tbl有a,b,c三个字段，其中a是主键，b上建了索引，然后编写sql语句SELECT * FROM tbl WHERE a=1这样不会产生回表，因为所有的数据在a的索引树中均能找到SELECT * FROM tbl WHERE b=1这样就会产生回表，因为where条件是b字段，那么会去b的索引树里查找数据，但b的索引里面只有a,b两个字段的值，没有c，那么这个查询为了取到c字段，就要取出主键a的值，然后去a的索引树去找c字段的数据。查了两个索引树，这就叫回表。索引覆盖就是查这个索引能查到你所需要的所有数据，不需要去另外的数据结构去查。其实就是不用回表。怎么避免？不是必须的字段就不要出现在SELECT里面。或者b,c建联合索引。但具体情况要具体分析，索引字段多了，存储和插入数据时的消耗会更大。这是个平衡问题


InnoDB有两大类索引：
聚集索引(clustered index)
普通索引(secondary index)

 

InnoDB聚集索引和普通索引有什么差异？
InnoDB聚集索引的叶子节点存储行记录，因此， InnoDB必须要有，且只有一个聚集索引：
（1）如果表定义了PK，则PK就是聚集索引；
（2）如果表没有定义PK，则第一个not NULL unique列是聚集索引；
（3）否则，InnoDB会创建一个隐藏的row-id作为聚集索引；

先查p索引再去查聚集索引
```



### 性能分析

#### mysql常见瓶颈

CPU:CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
I0:磁盘I/O瓶颈发生在装入数据远大于内存容量的时候
服务器硬件的性能瓶颈: top,free, iostat和vmstat来查看系统的性能状态

#### mysql查询优化器

**mysql query optimizer**
### 查询优化

1. 慢查询的开启并捕获 
2. explain+慢SQL分析
3. show profile查询SQL在Mysq1服务器里面的执行细节和生命周期情况
4. SQL数据库服务器的参数调优。
5. 分库分表

```sql
PROCEDURE ANALYSE() --自动分析字段和其实际的数据，并会给你一些有用的建议。表中有实际的数据，建议才会变得有用
SELECT * FROM table PROCEDURE analyse(4);
```

==小结果集驱动大结果集==

优先优化内层循环

被驱动表join字段加索引  左连接 索引加右表  右连接  索引加左表

设置JoinBuffer

#### order by 优化

* 使用index排序, 避免FileSort排序

  index利用索引效率高

  * order by使用索引最左前列
  * where和order by 条件列组合满足索引最左前列

* FileSort  分为两种算法: 双路排序  单路排序
  * 双路排序  : mysql4.1之前使用   读两次磁盘
  * 单路排序  : 读一次 , 占空间大 效率快

##### 优化策略

增大sort_buffer_size

增大max_length_for_sort_data



1.Order by时select*是一个大忌只Query需要的字段，这点非常重要。在这里的影响是：

​		1.1当Query的字段大小总和小于max_length_for_sort_data而且排序字段不是TEXTIBLOB类型时，会用改进后的算法——单路排序，否则用老算法——多路排序。

​		1.2两种算法的数据都有可能超出sort_buffer的容量，超出之后，会创建tmp文件进行合并排序，导致多次I/O,但是用单路排序算法的风险会更大一些，所以要提高sort_buffer_size.

2.尝试提高sort_buffer_size

不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数	是针对每个进程的

3.尝试提高max_length_for_sort_data

提高这个参数，会增加用改进算法的概率。但是如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘I/O活动和低的处理器使用率.

#### group by 优化

group by实质是先排序后进行分组，遵照索引建的最佳左前缀
当无法使用索引列，增大max_ Jength_ for, sort. ,data参数的设置+增大sort_ buffer size参数的设置
where高于having,能写在where限定的条件就不要去having限定了。

### 慢查询

```sql
SHOW VARIABLES LIKE '%slow_query_log%'; 
set global slow_query_log=1  --开启  只对当前数据库生效，重启失效

set global long_query_time=3; --设置阈值--默认10s
SHOW VARIABLES LIKE 'long_query_time%';--查看阈值

--修改后需要新开会话
SHOW VARIABLES LIKE 'long_query_time%'
--或
SHOW global VARIABLES LIKE 'long_query_time'
```

永久生效修改my.cnf配置文件

修改my.cnf文件，[mysqld]下增加或修改参数

slow_query_log 和slow_query_log_file后，然后重启MySQL服务器。也即将如下两行配置进my.cnf文件

slow_query_log=1

slow_query_log_file=/var/lib/mysql/atguigu-slow.log

#### **mysqldumpslow**  日志查看工具

```shell
#得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log
#得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log
#得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" / var / lib / mysql / atguigu-slow . log
#另外建议在使用这些命令时结合|和more使用，否则有可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log | more
```

```shell
s:是表示按照何种方式排序；
c:访问次数
:锁定时间
r:返回记录
t:查询时间
al:平均锁定时间
ar:平均返回记录数
at:平均查询时间
t:即为返回前面多少条的数据；
g:后边搭配一个正则匹配模式，大小写不敏感的；
```

### **批量插入**

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

### **show profile**

分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量

默认情况下，参数处于关闭状态，并保存最近15次的运行结果

```sql
Show variables like 'profiling'
set profiling=on;

--查看
show profiles
show profile cpu,block io for query--上一步前面的问题SQL数字号码；
```

```sql
type:
|ALL--显示所有的开销信息
|BLOCK IO--显示块IO相关开销
|CONTEXT SWITCHES--上下文切换相关开销
|CPU--显示CPU相关开销信息
|IPC--显示发送和接收相关开销信息
|MEMPRY--显示内存相关开销信息
|PAGE FAULTS--显示页面错误相关开销信息
|SOURCE--显示和Source_function,Source_file,Source_line相关的开销信息
|SWAPS--显示交换次数相关开销的信息
```

bad 

```sql
converting HEAP to MyISAM查询结果太大，内存都不够用了往磁盘上搬了。
Creating tmp table 创建临时表   拷贝数据到临时表   用完再删除
Copying to tmp table on disk 把内存中临时表复制到磁盘，危险！!!
locked
```

### **全局查询日志**

永远不要在生产环境开启这个功能

```shell
在mysql的my.cnf中，设置如下：
#开启
general_log=1
#记录日志文件的路径
general_log_file=/path/logfile
#输出格式
log_output=FILE
```

命令启用

```sql
set global general_log=1;
set global log_output='TABLE';
查看sql
select*from mysql.general_log;
```

### 锁机制



#### 分类

粒度分  行锁  表锁  页锁

操作分   读锁  写锁



#### 表锁(偏读)

偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低





#### 行锁(偏写)

InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

InnoDB与MyISAM的最大不同有两点:一是支持事务(TRANSACTION) ;二是采用了行级锁

MyISAM在执行查询语句（SELECT)前，会自动给涉及的所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁。

读锁阻塞写

写锁堵塞读和写

Myisam的读写锁调度是写优先，这也是myisam不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞

```sql
--常看当前数据库的事务隔离级别：
show variables like'tx_isolation';

--锁一行
select xxx...for update读定某一行后，其它的操作会被阻塞，直到锁定行的会话提交commit
```

```sql
lock table 表名字 read(write),表名字2 read(write),其它；
unlock tables;
--看锁
show open tables;
show status like 'table%';

Table_locks_immediate:产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1;
Table_locks_waited:出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1),此值高则说明存在着较严重的表级锁争用情况；
```

```sql
show status like 'innodb_row_lock%';
--查看行锁
Innodb_row_lock_current_waits:当前正在等待锁定的数量；
Innodb_row_lock_time:从系统启动到现在锁定总时间长度工
Innodb_row_lock_time_avg:每次等待所花平均时间；
Innodb_row_lock_time_max:从系统启动到现在等待最常的一次所花的时间；
Innodb_row_lock_waits:系统启动后到现在总共等待的次数；
--对于5个状态变量，比较重要的主要
Innodb_row_lock_time_avg(等待平均时长)
Innodb_row_lock_waits（等待总次数）
Innodb_row_lock_time(等待总时长）
```

**什么是间隙锁**

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，

InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。

**危害**

因为Query执行过程中通过过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。

间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害

|                | session1                   | session2                       |
| -------------- | -------------------------- | ------------------------------ |
| session1加读锁 | 可读加锁表不可读其他无锁表 | 可读加锁表可读可更新其他无锁表 |
|                | 不可更新加锁表             | 更新加锁表会阻塞               |
|                | 释放锁                     | 更新操作完成                   |

|                | session1         | session2       |
| -------------- | ---------------- | -------------- |
| session1加写锁 | 可读可更新加锁表 | 查询加锁表阻塞 |
|                | 释放锁           | 查询操作完成   |



#### 页锁

开销和加锁时间界于表锁和行锁之间;会出现死锁;锁定粒度界于表锁和行锁之间，并发度一般。

#### 优化建议

尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。
合理设计索引，尽量缩小锁的范围
尽可能较少检索条件，避免间隙锁
尽量控制事务大小，减少锁定资源量和时间长度
尽可能低级别事务隔离

## 主从复制







## 其他版本mysql

![image-20210915111320638](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210915111320638.png)

Percona为MySQL数据库服务器进行了改进，在功能和性能上较MySQL有着很显著的提升。该版本提升了在高负载情况下的InnoDB的性能、为DBA提供一些非常有用的性能诊断工具；另外有更多的参数和命令来控制服务器行为。

该公司新建了一款存储引擎叫xtradb完全可以替代innodb，并且在性能和并发上做得更好，

阿里巴巴大部分mysql数据库其实使用的percona的原型加以修改。

AliSql+AliRedis

```shell
#镜像地址：https://hub.docker.com/_/percona/
#拉取镜像
docker pull percona:5.7.23
#创建容器
docker create --name percona -v /data/mysql-data:/var/lib/mysql -p 3306:3306 
-e MYSQL_ROOT_PASSWORD=root percona:5.7.23

#参数解释：
--name：percona 指定是容器的名称
-v：/data/mysql-data:/var/lib/mysql 
将主机目录/data/mysql-data挂载到容器的/var/lib/mysql上

-p：33306:3306设置端口映射，主机端口是33306，容器内部端口3306
-e：MYSQL_ROOT_PASSWORD=root设置容器参数，设置root用户的密码为root
percona:5.7.23：镜像名:版本

#启动容器
docker start percona
```



# Elasticsearch
[官网](https://www.elastic.co/cn/)

一个开源的高扩展的分布式全文搜索引擎



The Elastic Stack, 包括 Elasticsearch、Kibana、Beats 和 Logstash（也称为 ELK Stack）。能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。Elaticsearch是整个Elastic Stack技术栈的核心。它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。



官网文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

官网中文：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

社区中文：http://doc.codingdict.com/elasticsearch/



腾讯云使用Kafka或Logstash作为日志采集，存储到es集群，使用可视化界面快速定位日志异常

ELK快速定位日志
	1）使用LogStash作为日志收集的汇聚节点
	2）使用ES集群作为存储
	3）使用Kibana作为可视化



## Elasticsearch And Solr

都是基于Lucene搭建的，可以独立部署启动的搜索引擎服务软件

![image-20211031124211229](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211031124211229.png)



* 与Solr相比，Elasticsearch易于安装且非常轻巧。此外，你可以在几分钟内安装并运行Elasticsearch。但是，如果Elasticsearch管理不当，这种易于部署和使用可能会成为一个问题。基于JSON的配置很简单，但如果要为文件中的每个配置指定注释，那么它不适合您。总的来说，如果你的应用使用的是JSON，那么Elasticsearch是一个更好的选择。否则，请使用Solr，因为它的schema.xml和solrconfig.xml都有很好的文档记录。

*  Solr拥有更大，更成熟的用户，开发者和贡献者社区。ES虽拥有的规模较小但活跃的用户社区以及不断增长的贡献者社区。

* Solr贡献者和提交者来自许多不同的组织，而Elasticsearch提交者来自单个公司。

*  Solr更成熟，但ES增长迅速，更稳定。

* Solr是一个非常有据可查的产品，具有清晰的示例和API用例场景。 Elasticsearch的文档组织良好，但它缺乏好的示例和清晰的配置说明。

那么，到底是Solr还是Elasticsearch

*  由于易于使用，Elasticsearch在新开发者中更受欢迎。一个下载和一个命令就可以启动一切。

* 如果除了搜索文本之外还需要它来处理分析查询，Elasticsearch是更好的选择

* 如果需要分布式索引，则需要选择Elasticsearch。对于需要良好可伸缩性和以及性能分布式环境，Elasticsearch是更好的选择。

*  Elasticsearch在开源日志管理用例中占据主导地位，许多组织在Elasticsearch中索引它们的日志以使其可搜索。

* 如果你喜欢监控和指标，那么请使用Elasticsearch，因为相对于Solr，Elasticsearch暴露了更多的关键指标



## 安装

https://www.elastic.co/cn/downloads/past-releases#elasticsearch



windows版 elasticsearch.bat启动es服务

9300端口为Elasticsearch集群间组件的通信端口，9200端口为浏览器访问的http协议RESTful端口

http://localhost:9200测试结果

如果系统配置JAVA_HOME，es使用系统默认JDK，否则使用自带的JDK，一般建议使用系统配置

空间不足修改config/jvm.options 堆内存



docker

```shell
mkdir -p /mydata/elasticsearch/config # 用来存放配置文件
mkdir -p /mydata/elasticsearch/data  # 数据
echo "http.host: 0.0.0.0" >/mydata/elasticsearch/config/elasticsearch.yml # 允许任何机器访问
chmod -R 777 /mydata/elasticsearch/ ## 设置elasticsearch文件可读写权限

docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v  /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2 


docker update elasticsearch --restart=always #开机自启

#Kibana 
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.56.10:9200 -p 5601:5601 -d kibana:7.4.2

#http://192.168.56.10:9200 改成自己Elasticsearch上的地址
```



## 入门

Elasticsearch是面向文档型数据库，一条数据在这里就是一个文档

![image-20211031130342892](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211031130342892.png)

Elasticsearch 7.X中, Type的概念已经被删除



==倒排索引 (将关键字作为索引)==



### http操作

#### _cat

```
GET /_cat/nodes：查看所有节点
GET /_cat/health：查看 es 健康状况
GET /_cat/master：查看主节点
GET /_cat/incices：查看所有索引 show databases;
```



#### 索引操作

```shell
#创建索引
#向ES服务器发PUT请求 http://127.0.0.1:9200/shopping
{
    "acknowledged"【响应结果】: true, # true操作成功
    "shards_acknowledged"【分片结果】: true, # 分片操作成功
    "index"【索引名称】: "shopping"
}
#注意：创建索引库的分片数默认1片，在7.0.0之前的Elasticsearch版本中，默认5片

#查看索引
#向ES服务器发GET请求 ：http://127.0.0.1:9200/_cat/indices?v
#_cat表示查看的意思，indices表示索引  查看当前ES服务器中的所有索引
#返回结果如下
```

| 表头           | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| health         | 当前服务器健康状态：  **green**(集群完整) **yellow**(单点正常、集群不完整)  red(单点不正常) |
| status         | 索引打开、关闭状态                                           |
| index          | 索引名                                                       |
| uuid           | 索引统一编号                                                 |
| pri            | 主分片数量                                                   |
| rep            | 副本数量                                                     |
| docs.count     | 可用文档数量                                                 |
| docs.deleted   | 文档删除状态（逻辑删除）                                     |
| store.size     | 主分片和副分片整体占空间大小                                 |
| pri.store.size | 主分片占空间大小                                             |

```shell
#查看单个索引
#向ES服务器发GET请求 ：http://127.0.0.1:9200/shopping
{
    "shopping"【索引名】: {        
        "aliases"【别名】: {},
        "mappings"【映射】: {},
        "settings"【设置】: {
            "index"【设置 - 索引】: {
                "creation_date"【设置 - 索引 - 创建时间】: "1614265373911",
                "number_of_shards"【设置 - 索引 - 主分片数量】: "1",
                "number_of_replicas"【设置 - 索引 - 副分片数量】: "1",
                "uuid"【设置 - 索引 - 唯一标识】: "eI5wemRERTumxGCc1bAk2A",
                "version"【设置 - 索引 - 版本】: {
                    "created": "7080099"
                },
                "provided_name"【设置 - 索引 - 名称】: "shopping"
            }
        }
    }
}

#删除索引
#向ES服务器发DELETE请求 ：http://127.0.0.1:9200/shopping
```

#### 文档操作

##### 创建文档

```shell
向ES服务器发POST请求 ：http://127.0.0.1:9200/shopping/_doc
{
    "title":"小米手机",
    "category":"小米",
    "images":"http://www.gulixueyuan.com/xm.jpg",
    "price":3999.00
}

#返回
{
    "_index"【索引】: "shopping",
    "_type"【类型-文档】: "_doc",
    "_id"【唯一标识】: "Xhsa2ncBlvF_7lxyCE9G", #可以类比为MySQL中的主键，随机生成
    "_version"【版本】: 1,
    "result"【结果】: "created", #这里的create表示创建成功
    "_shards"【分片】: {
        "total"【分片 - 总数】: 2,
        "successful"【分片 - 成功】: 1,
        "failed"【分片 - 失败】: 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
#上面的数据创建后，由于没有指定数据唯一性标识（ID），默认情况下，ES服务器会随机生成一个。
#如果想要自定义唯一性标识，需要在创建时指定：http://127.0.0.1:9200/shopping/_doc/1
#如果增加数据时明确数据主键，那么请求方式也可以为PUT
```

##### 查看文档

```shell
#GET请求 ：http://127.0.0.1:9200/shopping/_doc/1
#返回
{
    "_index"【索引】: "shopping",
    "_type"【文档类型】: "_doc",
    "_id": "1",
    "_version": 2,
    "_seq_no": 2,#并发控制字段，每次更新就会+1，用来做乐观锁
    "_primary_term": 2,  #同上，主分片重新分配，如重启，就会变化
    "found"【查询结果】: true, # true表示查找到，false表示未查找到
    "_source"【文档源信息】: {
        "title": "华为手机",
        "category": "华为",
        "images": "http://www.gulixueyuan.com/hw.jpg",
        "price": 4999.00
    }
}

#GET请求 ：http://127.0.0.1:9200/shopping/_search  查询全部
#更新携带：?if_seq_no=4&if_primary_term=1  乐观锁
```

##### 修改文档

```shell
#POST请求 ：http://127.0.0.1:9200/shopping/_doc/1  会覆盖
{
    "title":"华为手机",
    "category":"华为",
    "images":"http://www.gulixueyuan.com/hw.jpg",
    "price":4999.00
}

#返回
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version"【版本】: 2,
    "result"【结果】: "updated", # updated表示数据被更新
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 2
}
-------------------------------------
POST customer/external/1/_update
{
	"doc":{
		"name":"John Doew"
	}
}
或者
POST customer/external/1
{
	"name":"John Doe2"
}
或者
PUT customer/external/1
{
	"name":"jack"
}

#不同：Post  _update操作会对比源文档数据，数据如果一样就不进行任何操作
#	Post/PUT操作总会将数据重新保存并增加 version 版本

#对于大并发更新，不带update
#对于大并发查询并偶尔更新，带update 对比更新，重新计算分配规则
```

##### 修改字段

```shell
#POST请求 ：http://127.0.0.1:9200/shopping/_update/1
{ 
  "doc": {
    "price":3000.00
  } 
}
```

##### 删除文档

```shell
#DELETE请求 ：http://127.0.0.1:9200/shopping/_doc/1 逻辑删除

#返回
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "1",
    "_version"【版本】: 4, #对数据的操作，都会更新版本
    "result"【结果】: "deleted", # deleted表示数据被标记为删除
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 4,
    "_primary_term": 2
}
```

##### 条件删除文档

```shell
#POST请求 ：http://127.0.0.1:9200/shopping/_delete_by_query
{
  "query":{
    "match":{
      "price":4000.00
    }
  }
}
#返回
{
    "took"【耗时】: 175,
    "timed_out"【是否超时】: false,
    "total"【总数】: 2,
    "deleted"【删除数量】: 2,
    "batches": 1,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1.0,
    "throttled_until_millis": 0,
    "failures": []
}
```

#### 批量bulk 

```shell
#语法
{action:{metadata}}\n
{requeestBody}\n
{action:{metadata}}\n
{requesetbod }\n

POST customer/external/_bulk
{"index":{"_id":"1"}}
{"name":"John Doe"}
{"index":{"_id":"2"}}
{"name":"John Doe"}

#复杂实例
POST /_bulk
{"delete":{"_index":"website","_type":"blog","_id":"123"}}
{"create":{"_index":"website","_type":"blog","_id":"123"}}
{"title":"my first blog post"}
{"index":{"_index":"website","_type":"blog"}}
{"title":"my second blog post"}
{"update":{"_index":"website","_type":"blog","_id":"123"}}
{"doc":{"title":"my updated blog post"}}
```

bulk 按顺序执行所有的action (动作)。如果一一个单个的动作因任何原因而失败，它将继续处理它后面剩余的动作。当bulkAPI 返回时，它将提供每个动作的状态(与发送的顺序相同)



样本测试数据

https://github.com/elastic/elasticsearch/edit/master/docs/src/test/resources/accounts.json

#### [映射操作](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/mapping-types.html)

映射，类似于数据库(database)中的表结构(table)。创建数据库表需要设置字段名称，类型，长度，约束等；索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射(mapping)



ES 7.x 

URL 中的 type 参数 可选，比如索引一个文档不再要求提供文档类型

ES 8.X

不在支持 URL 中的 type 参数

```shell
#创建映射 / 添加新映射
#PUT请求 ：http://127.0.0.1:9200/student/_mapping
{
  "properties": {
    "name":{
      "type": "text",
      "index": true
    },
    "sex":{
      "type": "text",
      "index": false
    },
    "age":{
      "type": "long",
      "index": false
    }
  }
}
#映射数据说明
字段名：任意填写，下面指定许多属性，例如：title、subtitle、images、price
type：类型，Elasticsearch中支持的数据类型非常丰富，说几个关键的：
	String类型，又分两种：
		text：可分词
		keyword：不可分词，数据会作为完整字段进行匹配
	Numerical：数值类型，分两类
		基本数据类型：long、integer、short、byte、double、float、half_float
		浮点数的高精度类型：scaled_float
	Date：日期类型
	Array：数组类型
	Object：对象
index：是否索引，默认为true，也就是说你不进行任何配置，所有字段都会被索引。
	true：字段会被索引，则可以用来进行搜索
	false：字段不会被索引，不能用来搜索
store：是否将数据进行独立存储，默认为false
	原始的文本会存储在_source里面，默认情况下其他提取出来的字段都不是独立存储的，是从_source里面提取出来的。当然	 你也可以独立的存储某个字段，只要设置"store": true即可，获取独立存储的字段要比从_source中解析快得多，但是也	会占用更多的空间，所以要根据实际业务需求来设置。
analyzer：分词器，这里的ik_max_word即使用ik分词器,后面会有专门的章节学习


#查看映射
#GET请求 ：http://127.0.0.1:9200/student/_mapping

#索引映射关联
#PUT请求 ：http://127.0.0.1:9200/student1
{
  "settings": {},
  "mappings": {
	  "properties": {
		"name":{
		  "type": "text",
		  "index": true
		  
		},
		"sex":{
		  "type": "text",
		  "index": false
		},
		"age":{
		  "type": "long",
		  "index": false
		}
	  }
  }
}

#对于已经存在的映射字段，我们不能更新，更新必须创建新的索引进行数据迁移


#数据迁移
#先创 new_twitter 的正确映射，然乎使用如下方式进行数据迁移
POST _reindex [固定写法]
{
  "source":{
    "index":"twitter"
  },
  "dest":{
    "index":"new_twitter"
  }
}
## 将旧索引的 type 下的数据进行迁移
POST _reindex
{
  "source": {
    "index":"twitter",
    "type":"tweet"
  },
  "dest":{
    "index":"twweets"
  }
}
```

参考官网：https://www.elastic.co/guide/en/elasticsearch/reference/7.10/mapping-types.html

参数映射规则：https://www.elastic.co/guide/en/elasticsearch/reference/7.10/mapping-params.html#mapping-params

#### 高级查询

ES 支持两种基本方式检索:

- 一个是通过使用 REST request URL,发送搜索参数，(uri + 检索参数)
- 另一个是通过使用 REST request bod 来发送他们，(uri + 请求体)



DSL （domain-specifig langurage 领域特定语言）， Query DSL 





```json
GET /bank/_search
{
    "query": { #query 定义如何查询
    	"match_all": {}  # match_all 查询类型【代表查询所有的所有】
    	"match": {
    		"address": "mill lane"  #字符串分词匹配   非字符串精确匹配   查询结果评分排序
    		"address.keyword": "mill lane" #keyword精确匹配
		} ,
		"match_phrase": { "address": "mill lane" }#不分词匹配
		"multi_match": {
      		"query": "mill",
      		"fields": ["address","city"]  #两个字段任意一个包含 多字段匹配 会分词
    	}
	},  
	"sort": [
    	{ "account_number": "asc" }
	],
	"from": 10,
	"size": 10
	"_source": ["firstname","lastname"] #返回部分字段
}
```





##### 查询所有文档

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match_all": {}
  }
}
# "query"：这里的query代表一个查询对象，里面可以有不同的查询属性
# "match_all"：查询类型，例如：match_all(代表查询所有)， match，term ， range 等等
# {查询条件}：查询条件会根据类型的不同，写法也有差异

#返回
{
  "took【查询花费时间，单位毫秒】" : 1116,
  "timed_out【是否超时】" : false,
  "_shards【分片信息】" : {
    "total【总数】" : 1,
    "successful【成功】" : 1,
    "skipped【忽略】" : 0,
    "failed【失败】" : 0
  },
  "hits【搜索命中结果】" : {
     "total"【搜索条件匹配的文档总数】: {
         "value"【总命中计数的值】: 3,
         "relation"【计数规则】: "eq" # eq 表示计数准确， gte表示计数不准确
     },
    "max_score【匹配度分值】" : 1.0,
    "hits【命中结果集合】" : [
       。。。
      }
    ]
  }
}
```

##### 匹配查询

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search   
#match匹配类型查询，会把查询条件进行分词，然后进行查询，多个词条之间是or的关系
{
  "query": {
    "match": {
        "name":"zhangsan"
    }
  }
}

```

##### 字段匹配查询

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
#multi_match与match类似，不同的是它可以在多个字段中查询
{
  "query": {
    "multi_match": {
        "query": "zhangsan",
        "fields": ["name","nickname"]
    }
  }
}
```

##### term

和 match 一样，匹配某个属性的值，全文检索字段用 match，其他非text字段匹配用 term

```shell
#term查询，精确的关键词匹配查询，不对查询条件进行分词
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "term": {
      "name": {
        "value": "zhangsan"
      }
    }
  }
}
```

##### 多关键字精确查询

```shell
#terms 查询和 term 查询一样，但它允许你指定多值进行匹配
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "terms": {
      "name": ["zhangsan","lisi"]
    }
  }
}
```

##### 指定查询字段

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
#_source过滤字段
{
  "_source": ["name","nickname"],  
  "query": {
    "terms": {
      "nickname": ["zhangsan"]
    }
  }
}
```

#####  过滤字段

```shell
#includes：来指定想要显示的字段
#excludes：来指定不想要显示的字段
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "_source": {
    "includes": ["name","nickname"]
    "excludes": ["name","nickname"]
  },  
  "query": {
    "terms": {
      "nickname": ["zhangsan"]
    }
  }
}
```

##### 组合查询-bool

复合语句可以合并 任何 其他嵌套语句，包括复合语句





should: 达到会增加相关文档的评分，并不会改变查询的结果，如果 query 中只有 should 且只有一种匹配规则，那么 should的条件就会被作为默认匹配条件而区别查询结果

使用`minimum_should_match`，至少匹配一项`should`子句

```shell
#`bool`把各种其它查询通过`must`（必须 ）、`must_not`（必须不）、`should`（应该）的方式进行组合
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "zhangsan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "age": "40"
          }
        }
      ],
      "should": [
        {
          "match": {
            "sex": "男"
          }
        }
      ]
      "filter": {  #不会产生相关得分
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

#####  范围查询

```shell
#range 查询找出那些落在指定区间内的数字或者时间  gt> lt< gte>= lte<=
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 35
      }
    }
  }
}
```

##### 模糊查询

```shell
#编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数。这些更改可以包括：
#更改字符（box → fox）
#删除字符（black → lack）
#插入字符（sic → sick）
#转置两个相邻字符（act → cat）
#为了找到相似的术语，fuzzy查询会在指定的编辑距离内创建一组搜索词的所有可能的变体或扩展。然后查询返回每个扩展的完全匹配。
#通过fuzziness修改编辑距离。一般使用默认值AUTO，根据术语的长度生成编辑距离。

#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "zhangsan"
      }
    }
  }
}
#或
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "zhangsan",
		"fuzziness": 2
      }
    }
  }
}
```

##### 单字段排序

```shell
#sort 可以让我们按照不同的字段进行排序，并且通过order指定排序的方式。desc降序，asc升序
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match": {
        "name":"zhangsan"
    }
  },
  "sort": [{
    "age": {
        "order":"desc"
    }
  }]
}

```

##### 多字段排序

```shell
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },
    {
      "_score":{
        "order": "desc"
      }
    }
  ]
}

```



##### 高亮查询

```shell
#使用match查询的同时，加上一个highlight属性
#pre_tags：前置标签
#post_tags：后置标签
#fields：需要高亮的字段
#title：这里声明title字段需要高亮，后面可以为这个字段设置特有配置，也可以空

#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match": {
      "name": "zhangsan"
    }
  },
  "highlight": {
    "pre_tags": "<font color='red'>",
    "post_tags": "</font>",
    "fields": {
      "name": {}
    }
  }
}
```

##### 分页查询

```shell
#from：当前页的起始索引，默认从0开始。 from = (pageNum - 1) * size
#size：每页显示多少条 
#GET请求 ：http://127.0.0.1:9200/student/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "from": 0,
  "size": 2
}
```

##### 聚合查询 -aggregations

聚合允许使用者对es文档进行统计分析，类似与关系型数据库中的group by，当然还有很多其他的聚合，例如取最大值、平均值等等

聚合提供了从数据分组和提取数据的能力，最简单的聚合方法大致等于 SQL GROUP BY 和 SQL 聚合函数，在 ES 中，你有执行搜索返回 hits （命中结果） 并且同时返回聚合结果，把一个响应中的所有 hits（命中结果）分隔开的能力，这是非常强大有效的，你可以执行查询和多个聚合，并且在一个使用中得到各自的（任何一个的）返回结果，使用一次简洁简化的 API 来避免网络往返



```shell
#GET请求 ：http://127.0.0.1:9200/student/_search 
#对某个字段取最大值max
{
    "aggs":{
      "max_age":{
        "max":{"field":"age"}
        "min":{"field":"age"}
        "sum":{"field":"age"}
      }
    },
    "size":0
}
#min
{
    "aggs":{
      "min_age":{
        "min":{"field":"age"}
      }
    },
    "size":0
}
#sum
{
    "aggs":{
      "sum_age":{
        "sum":{"field":"age"}
      }
    },
    "size":0
}
#avg
{
    "aggs":{
      "avg_age":{
        "avg":{"field":"age"}
      }
    },
    "size":0
}
#去重
{
    "aggs":{
      "distinct_age":{
        "cardinality":{"field":"age"}
      }
    },
    "size":0
}
#State聚合  一次性返回count，max，min，avg和sum五个指标
{
    "aggs":{
      "stats_age":{
        "stats":{"field":"age"}
      }
    },
    "size":0
}
```

**搜索address中包含mill的所有人的年龄分布以及平均年龄**

```http
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg":{
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg":{
      "avg": {
        "field": "balance"
      }
    }
    
  },
  "size":0
}
```

**按照年龄聚合，并且请求这些年龄段的这些人的平均薪资**

```java
GET bank/_search
{
  "query":{
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      },
      "aggs": {
        "ageAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

**查出所有年龄分布，并且这些年龄段中M的平均薪资和 F 的平均薪资以及这个年龄段的总体平均薪资**

```java
GET /bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "genderAgg": {
          "terms": {
            "field": "gender.keyword",
            "size": 10
          },
          "aggs": {
            "balanceAvg": {
              "avg": {
                "field": "balance"
                }
            }
          }
        }
      }
    }
  }
}
```

##### 桶聚合查询

```shell
#桶聚和相当于sql中的group by语句
#terms聚合，分组统计

#GET请求 ：http://127.0.0.1:9200/student/_search
{
    "aggs":{
      "age_groupby":{
        "terms":{"field":"age"}
      }
    },
    "size":0
} 

#terms分组下再进行聚合
{
    "aggs":{
      "age_groupby":{
        "terms":{"field":"age"}
        "aggs":{
          "sum_age":{
            "sum":{"filed":"age"}
          }
        }
      }
    },
    "size":0
}
```

### java api操作

```xml
<dependencies>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch的客户端 -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.8.0</version>
    </dependency>
    <!-- elasticsearch依赖2.x的log4j -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
</dependency>
	<dependency>
		<groupId>com.fasterxml.jackson.core</groupId>
		<artifactId>jackson-databind</artifactId>
		<version>2.9.9</version>
	</dependency>
    <!-- junit单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

#### 客户端对象

```
// 创建客户端对象
RestHighLevelClient client = new RestHighLevelClient(
		RestClient.builder(new HttpHost("localhost", 9200, "http"))
);

...

// 关闭客户端连接
client.close();
```

#### 索引操作

##### 创建索引

```java
// 创建索引 - 请求对象
CreateIndexRequest request = new CreateIndexRequest("user");
// 发送请求，获取响应
CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
boolean acknowledged = response.isAcknowledged();
// 响应状态
System.out.println("操作状态 = " + acknowledged);
```

##### 查看索引

```java
// 查询索引 - 请求对象
GetIndexRequest request = new GetIndexRequest("user");
// 发送请求，获取响应
GetIndexResponse response = client.indices().get(request, RequestOptions.DEFAULT);
System.out.println("aliases:"+response.getAliases());
System.out.println("mappings:"+response.getMappings());
System.out.println("settings:"+response.getSettings());
```

##### 删除索引

```java
// 删除索引 - 请求对象
DeleteIndexRequest request = new DeleteIndexRequest("user");
// 发送请求，获取响应
AcknowledgedResponse response = client.indices().delete(request, RequestOptions.DEFAULT);
// 操作结果
System.out.println("操作结果 ： " + response.isAcknowledged());
```

#### 文档操作

##### 增

```java
@Data
class User {                         
    private String name;             
    private Integer age;             
    private String sex; 
}

// 新增文档 - 请求对象
IndexRequest request = new IndexRequest();
// 设置索引及唯一性标识
request.index("user").id("1001");
// 创建数据对象
User user = new User();
user.setName("zhangsan");
user.setAge(30);
user.setSex("男");
ObjectMapper objectMapper = new ObjectMapper();
String productJson = objectMapper.writeValueAsString(user);
// 添加文档数据，数据格式为JSON格式
request.source(productJson,XContentType.JSON);
// 客户端发送请求，获取响应对象
IndexResponse response = client.index(request, RequestOptions.DEFAULT);
////3.打印结果信息
System.out.println("_index:" + response.getIndex());
System.out.println("_id:" + response.getId());
System.out.println("_result:" + response.getResult());  

```

##### 删

```java
//创建请求对象
DeleteRequest request = new DeleteRequest().index("user").id("1");
//客户端发送请求，获取响应对象
DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
//打印信息
System.out.println(response.toString());
```

##### 改

```java
// 修改文档 - 请求对象
UpdateRequest request = new UpdateRequest();
// 配置修改参数
request.index("user").id("1001");
// 设置请求体，对数据进行修改
request.doc(XContentType.JSON, "sex", "女");
// 客户端发送请求，获取响应对象
UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
System.out.println("_index:" + response.getIndex());
System.out.println("_id:" + response.getId());
System.out.println("_result:" + response.getResult());
```

##### 查

```java
//1.创建请求对象
GetRequest request = new GetRequest().index("user").id("1001");
//2.客户端发送请求，获取响应对象
GetResponse response = client.get(request, RequestOptions.DEFAULT);
////3.打印结果信息
System.out.println("_index:" + response.getIndex());
System.out.println("_type:" + response.getType());
System.out.println("_id:" + response.getId());
System.out.println("source:" + response.getSourceAsString());
```

##### 批量

批量新增

```java
//创建批量新增请求对象
BulkRequest request = new BulkRequest();
request.add(new IndexRequest().index("user").id("1001").source(XContentType.JSON, "name", "zhangsan"));
request.add(new IndexRequest().index("user").id("1002").source(XContentType.JSON, "name", "lisi"));
request.add(new IndexRequest().index("user").id("1003").source(XContentType.JSON, "name", "wangwu"));
//客户端发送请求，获取响应对象
BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
//打印结果信息
System.out.println("took:" + responses.getTook());
System.out.println("items:" + responses.getItems());
```

批量删除

```java
//创建批量删除请求对象
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest().index("user").id("1001"));
request.add(new DeleteRequest().index("user").id("1002"));
request.add(new DeleteRequest().index("user").id("1003"));
//客户端发送请求，获取响应对象
BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
//打印结果信息
System.out.println("took:" + responses.getTook());
System.out.println("items:" + responses.getItems());
```

#### 高级查询

##### 查询所有索引数据

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
// 查询所有数据
sourceBuilder.query(QueryBuilders.matchAllQuery());
request.source(sourceBuilder);

SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### term查询，查询条件为关键字

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.termQuery("age", "30"));
request.source(sourceBuilder);

SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 分页

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());

// 分页查询
// 当前页其实索引(第一条数据的顺序号)，from
sourceBuilder.from(0);
// 每页显示多少条size
sourceBuilder.size(2);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");

```

##### 排序

```java
// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());

// 排序
sourceBuilder.sort("age", SortOrder.ASC);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");

```

##### 过滤字段

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.query(QueryBuilders.matchAllQuery());

//查询字段过滤
String[] excludes = {};
String[] includes = {"name", "age"};
sourceBuilder.fetchSource(includes, excludes);

request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 组合查询

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
// 必须包含
boolQueryBuilder.must(QueryBuilders.matchQuery("age", "30"));
// 一定不含
boolQueryBuilder.mustNot(QueryBuilders.matchQuery("name", "zhangsan"));
// 可能包含
boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "男"));

sourceBuilder.query(boolQueryBuilder);
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 范围查询

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("age");
// 大于等于
rangeQuery.gte("30");
// 小于等于
rangeQuery.lte("40");

sourceBuilder.query(rangeQuery);
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");

```

##### 模糊查询

```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("student");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

sourceBuilder.query(QueryBuilders.fuzzyQuery("name","zhangsan").fuzziness(Fuzziness.ONE));
request.source(sourceBuilder);
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
// 查询匹配
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut());
System.out.println("total:" + hits.getTotalHits());
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
for (SearchHit hit : hits) {
	//输出每条查询的结果信息
	System.out.println(hit.getSourceAsString());
}
System.out.println("<<========");
```

##### 高亮查询

```java
// 高亮查询
SearchRequest request = new SearchRequest().indices("student");
//2.创建查询请求体构建器
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
//构建查询方式：高亮查询
TermsQueryBuilder termsQueryBuilder = QueryBuilders.termsQuery("name","zhangsan");
//设置查询方式
sourceBuilder.query(termsQueryBuilder);
//构建高亮字段
HighlightBuilder highlightBuilder = new HighlightBuilder();
highlightBuilder.preTags("<font color='red'>");//设置标签前缀
highlightBuilder.postTags("</font>");//设置标签后缀
highlightBuilder.field("name");//设置高亮字段
//设置高亮构建对象
sourceBuilder.highlighter(highlightBuilder);
//设置请求体
request.source(sourceBuilder);
//3.客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

//4.打印响应结果
SearchHits hits = response.getHits();
System.out.println("took::"+response.getTook());
System.out.println("time_out::"+response.isTimedOut());
System.out.println("total::"+hits.getTotalHits());
System.out.println("max_score::"+hits.getMaxScore());
System.out.println("hits::::>>");
for (SearchHit hit : hits) {
	String sourceAsString = hit.getSourceAsString();
	System.out.println(sourceAsString);
	//打印高亮结果
	Map<String, HighlightField> highlightFields = hit.getHighlightFields();
	System.out.println(highlightFields);
}
System.out.println("<<::::");

```

##### 聚合查询

```java
SearchRequest request = new SearchRequest().indices("student");
//max age
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.max("maxAge").field("age"));
//设置请求体
request.source(sourceBuilder);
//3.客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

//4.打印响应结果
SearchHits hits = response.getHits();
System.out.println(response);

```

```java
SearchRequest request = new SearchRequest().indices("student");
//分组
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.terms("age_groupby").field("age"));

//设置请求体
request.source(sourceBuilder);
//3.客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);

//4.打印响应结果
SearchHits hits = response.getHits();
System.out.println(response);

```

### kibana操作

#### 索引

```shell
#创建索引
# PUT 索引名称(小写)
PUT test_ index

# PUT索引
# 增加配置↓JSON格式的主体内容
PUT test_ index 1
"aliases": {
	"test1": {}
}

# HEAD索引HTTP状态码: 200， 404
head test_ index1

#查询索引
# GET索引名称
GET test_ index
#查询所有索引
GET _cat/indices

#删除索引
# delete 索引名称
DELETE test_ index_ 1
 #kibana 不许修改索引
```

#### 文档

```shell
# 创建文档(索引数据)。增加唯一性标识(手动， 自动)
PUT test_ doc
PUT test_ doc/_ doc/1001
{
	"id" : 1001,
	"name" : "zhangsan",
	"age" : 30
} 
POST test. _doc/_ doc
{
"id" : 1002,
"name" : "lisi",
"age" : 40
}

#查询文档
GET test_doc/_doc/1001
#查询索引中所有的文档数据
GET test doc/_ search
#修改数据
PUT/POST test_doc/_doc/1001
{
"id" : 10011,
"name" : "zhangsan1"，
"age" : 300,
"tel": 123121 }

#删除数据
DELETE test_ doc/_ doc/1002|

```





## 环境

### 集群

大于等于2个节点就可以看做集群。一般出于高性能及高可用方面来考虑集群中节点数量都是3个以上

一个Elasticsearch集群有一个唯一的名字标识，这个名字默认就是”elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

#### Windows集群

1. 创建elasticsearch-cluster文件夹，在内部复制三个elasticsearch服务

2. 修改集群文件目录中每个节点的 config/elasticsearch.yml配置文件

   node1

   ```yaml
   #节点1的配置信息：
   #集群名称，节点之间要保持一致
   cluster.name: my-elasticsearch
   #节点名称，集群内要唯一
   node.name: node-1001
   node.master: true
   node.data: true
    
   #ip地址
   network.host: localhost
   #http端口
   http.port: 1001
   #tcp监听端口
   transport.tcp.port: 9301
   
   #discovery.seed_hosts: ["localhost:9301", "localhost:9302","localhost:9303"]
   #discovery.zen.fd.ping_timeout: 1m
   #discovery.zen.fd.ping_retries: 5
    
   #集群内的可以被选为主节点的节点列表
   #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
    
   #跨域配置
   #action.destructive_requires_name: true
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   
   ```

   node2

   ```yaml
   #节点2的配置信息：
   #集群名称，节点之间要保持一致
   cluster.name: my-elasticsearch
   #节点名称，集群内要唯一
   node.name: node-1002
   node.master: true
   node.data: true
    
   #ip地址
   network.host: localhost
   #http端口
   http.port: 1002
   #tcp监听端口
   transport.tcp.port: 9302
   
   discovery.seed_hosts: ["localhost:9301"]
   discovery.zen.fd.ping_timeout: 1m
   discovery.zen.fd.ping_retries: 5
    
   #集群内的可以被选为主节点的节点列表
   #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
    
   #跨域配置
   #action.destructive_requires_name: true
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   
   ```

   node3

   ```yaml
   #节点3的配置信息：
   #集群名称，节点之间要保持一致
   cluster.name: my-elasticsearch
   #节点名称，集群内要唯一
   node.name: node-1003
   node.master: true
   node.data: true
    
   #ip地址
   network.host: localhost
   #http端口
   http.port: 1003
   #tcp监听端口
   transport.tcp.port: 9303
   #候选主节点的地址，在开启服务后可以被选为主节点 
   discovery.seed_hosts: ["localhost:9301", "localhost:9302"]
   discovery.zen.fd.ping_timeout: 1m
   discovery.zen.fd.ping_retries: 5
    
   #集群内的可以被选为主节点的节点列表
   #cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
    
   #跨域配置
   #action.destructive_requires_name: true
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   
   ```

3. 启动前先删除每个节点中的data目录中所有内容（如果存在）

   双击执行 bin/elasticsearch.bat    启动后，会自动加入指定名称的集群

4. 测试  

   GET 请求 ：http://127.0.0.1:1001/_cluster/health

   GET 请求 ：http://127.0.0.1:1002/_cluster/health

   GET 请求 ：http://127.0.0.1:1003/_cluster/health

   status  

   ​	green 主分片副分片都正常

   ​	yellow 主分片正常 副分片有不正常的

   ​	red 主分片不正常

#### linux 集群

```shell
# 解压缩
tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/module
# 改名
mv elasticsearch-7.8.0 es-cluster

#因为安全问题，Elasticsearch不允许root用户直接运行，所以要在每个节点中创建新用户，在root用户中创建新用户 
useradd es #新增es用户
passwd es #为es用户设置密码

userdel -r es #如果错了，可以删除再加
chown -R es:es /opt/module/es-cluster #文件夹所有者

```

修改/opt/module/es/config/elasticsearch.yml文件，分发文件

```yaml
# 加入如下配置
#集群名称
cluster.name: cluster-es
#节点名称，每个节点的名称不能重复
node.name: node-1
#ip地址，每个节点的地址不能重复
network.host: linux1
#是不是有资格主节点
node.master: true
node.data: true
http.port: 9200
# head 插件需要这打开这两个配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["node-1"]
#es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["linux1:9300","linux2:9300","linux3:9300"]
gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
#集群内同时启动的数据任务个数，默认是2个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数，默认4个
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认4个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
```

修改/etc/security/limits.conf ，分发文件

```yaml
# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
```

修改/etc/security/limits.d/20-nproc.conf，分发文件

```yaml
\# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
\* hard nproc 4096
\# 注：* 带表Linux所有用户名称
```

修改/etc/sysctl.conf

```yaml
\# 在文件中增加下面内容
vm.max_map_count=655360
```

重新加载

```shell
sysctl -p 
```

启动

```shell
cd /opt/module/es-cluster
#启动
bin/elasticsearch
#后台启动
bin/elasticsearch -d
```

### linux 单机

https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-8-0

```shell
# 解压缩
tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz -C /opt/module
# 改名
mv elasticsearch-7.8.0 es

#因为安全问题，Elasticsearch不允许root用户直接运行，所以要创建新用户，在root用户中创建新用户 
useradd es #新增es用户
passwd es #为es用户设置密码

userdel -r es #如果错了，可以删除再加
chown -R es:es /opt/module/es #文件夹所有者

#修改/opt/module/es/config/elasticsearch.yml文件
# 加入如下配置
cluster.name: elasticsearch
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"]

#修改/etc/security/limits.conf 
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536

#修改/etc/security/limits.d/20-nproc.conf
# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
# 操作系统级别对每个用户创建的进程数的限制
* hard nproc 4096
# 注：* 带表Linux所有用户名称

#修改/etc/sysctl.conf
# 在文件中增加下面内容
# 一个进程可以拥有的VMA(虚拟内存区域)的数量,默认值为65536
vm.max_map_count=655360

#重新加载
sysctl -p

cd /opt/module/es/
#启动
bin/elasticsearch
#后台启动
bin/elasticsearch -d

#暂时关闭防火墙
systemctl stop firewalld

#永久关闭防火墙
systemctl enable firewalld.service #打开防火墙永久性生效，重启后不会复原
systemctl disable firewalld.service #关闭防火墙，永久性生效，重启后不会复原
```

### kibana

https://artifacts.elastic.co/downloads/kibana/kibana-7.8.0-windows-x86_64.zip

修改config/kibana.yml文件

```yaml
# 默认端口
server.port: 5601
# ES服务器的地址
elasticsearch.hosts: ["http://localhost:9200"]
# 索引名
kibana.index: ".kibana"
# 支持中文
i18n.locale: "zh-CN"
```

Windows环境下执行bin/kibana.bat文件

## 进阶

### 核心概念

* 索引 (名字小写字母)

  **Elasticsearch索引的精髓：一切设计都是为了提高搜索的性能**

* 类型

  一个索引中，你可以定义一种或多种类型

  7.X 默认不再支持自定义索引类型（默认类型为：_doc）

* 文档

  一个文档是一个可被索引的基础信息单元，也就是一条数据

* 字段（Field）

  相当于是数据表的字段，对文档数据根据不同属性进行的分类标识

* 映射

  mapping是处理数据的方式和规则方面做一些限制，如：某个字段的数据类型、默认值、分析器、是否被索引等等。这些都是映射里面可以设置的，其它就是处理ES里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好

* 分片（Shards）

  一个索引可以存储超出单个节点硬件限制的大量数据。比如，一个具有10亿文档数据的索引占据1TB的磁盘空间，而任一节点都可能没有这样大的磁盘空间。或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，每一份就称之为分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。

  分片很重要，主要有两方面的原因：

  1. 允许你水平分割 / 扩展你的内容容量。

  2. 允许你在分片之上进行分布式的、并行的操作，进而提高性能/吞吐量。

  至于一个分片怎样分布，它的文档怎样聚合和搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的，无需过分关心。

  **被混淆的概念是，一个 Lucene 索引 我们在 Elasticsearch 称作 分片 。 一个 Elasticsearch 索引 是分片的集合。 当 Elasticsearch 在索引中搜索的时候， 他发送查询到每一个属于索引的分片(Lucene 索引)，然后合并每个分片的结果到一个全局的结果集**

* 副本（Replicas）

  Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片(副本)。

  * 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。

  * 扩展你的搜索量/吞吐量，因为搜索可以在所有的副本上并行运行。

  总之，每个索引可以被分成多个分片。一个索引也可以被复制0次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。默认情况下，Elasticsearch中的每个索引被分片1个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有1个主分片和另外1个复制分片（1个完全拷贝），这样的话每个索引总共就有2个分片，我们需要根据索引需要确定分片个数。

* 分配（Allocation）

  将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。这个过程是由master节点完成的

### 架构

一个运行中的 Elasticsearch 实例称为一个节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为主节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。



### 分布式集群

#### 单节点集群

分配3个主分片和一份副本（每个主分片拥有一个副本分片）

```json
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

通过elasticsearch-head插件查看集群情况

集群健康值:yellow( 3 of 6 ) : 表示当前集群的全部主分片都正常运行，但是副本分片没有全部处在正常状态

3个主分片正常

3个副本分片都是 Unassigned —— 它们都没有被分配到任何节点。 在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据。

当前我们的集群是正常运行的，但是在硬件故障时有丢失数据的风险

#### 故障转移

增加一个节点  主分片和副本分片都被分配

3个副本分片被分配到新节点

#### 水平扩容

启动第三个节点，我们的集群将会拥有三个节点的集群 : 为了分散负载而对分片进行重新分配

扩容超过6个节点时

在运行中的集群上是可以动态调整副本分片数目的，按需伸缩集群。把副本数从默认的 1 增加到 2

####  应对故障

选举一个新的主节点

新的主节点立即将这些分片在 Node 2 和 Node 3 上对应的副本分片提升为主分片

此时集群的状态将会为 yellow



重新启动 Node 1 ，集群可以将缺失的副本分片再次进行分配，那么集群的状态也将恢复成之前的状态。 如果 Node 1 依然拥有着之前的分片，它将尝试去重用它们，同时仅从主分片复制发生了修改的数据文件。和之前的集群相比，只是Master节点切换了

### 路由计算

创建文档时，应当被存储在哪个分片

![image-20211101165540472](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211101165540472.png)

routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。 routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards （主分片的数量）后得到余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是我们所寻求的文档所在分片的位置



这就解释了为什么我们要在创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。

所有的文档 API（ get 、 index 、 delete 、 bulk 、 update 以及 mget ）都接受一个叫做 routing 的路由参数 ，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。

### 分片控制

我们可以发送请求到集群中的任一节点。 每个节点都有能力处理任意请求。 每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上。 在下面的例子中，将所有的请求发送到 Node 1，我们将其称为 **协调节点**(coordinating node) 。

**当发送请求的时候，** **为了扩展负载，更好的做法是轮询集群中所有的节点。**

#### 写流程

新建、索引和删除 请求都是 写 操作， 必须在主分片上面完成之后才能被复制到相关的副本分片



* 客户端向 Node 1 发送请求
* 通过id找到主分片
* 主分片执行, 副本同步 , 完成返回客户端



有一些可选的请求参数允许您影响这个过程，可能以数据安全为代价提升性能。这些选项很少使用，因为Elasticsearch已经很快，但是为了完整起见，请参考下面表格：

| 参数        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| consistency | consistency，即一致性。在默认设置下，即使仅仅是在试图执行一个_写_操作之前，主分片都会要求 必须要有 规定数量(quorum)（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态，才会去执行_写_操作(其中分片副本可以是主分片或者副本分片)。这是为了避免在发生网络分区故障（network partition）的时候进行_写_操作，进而导致数据不一致。_规定数量_即：  **int( (primary  + number_of_replicas) / 2 ) + 1**  consistency 参数的值可以设为 one （只要主分片状态 ok 就允许执行_写_操作）,all（必须要主分片和所有副本分片的状态没问题才允许执行_写_操作）, 或 quorum 。默认值为  quorum , 即大多数的分片副本状态没问题就允许执行_写_操作。  注意，规定数量 的计算公式中 number_of_replicas 指的是在索引设置中的设定副本分片数，而不是指当前处理活动状态的副本分片数。如果你的索引设置中指定了当前索引拥有三个副本分片，那规定数量的计算结果即：  **int( (primary  + 3 replicas) / 2 ) + 1 = 3**  如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，也因此您将无法索引和删除任何文档。 |
| timeout     | 如果没有足够的副本分片会发生什么？ Elasticsearch会等待，希望更多的分片出现。默认情况下，它最多等待1分钟。 如果你需要，你可以使用 timeout 参数 使它更早终止： 100 100毫秒，30s 是30秒。 |

新索引默认有 1 个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当 number_of_replicas 大于1的时候，规定数量才会执行

#### 读流程

在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。在文档被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报告文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的



#### 更新流程



* 从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试本步骤，超过 retry_on_conflict 次后放弃。
* 更新成功,同步副本,返回

当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。请记住，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。 如果Elasticsearch仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档。

#### 多文档操作流程

mget 和 bulk API 的模式类似于单文档模式。区别在于协调节点知道每个文档存在于哪个分片中。它将整个多文档请求分解成 每个分片 的多文档请求，并且将这些请求并行转发到每个参与节点。

协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端

**用单个 mget** **请求取回多个文档所需的步骤顺序**:

* 客户端向 Node 1 发送 mget 请求。

* Node 1 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复， Node 1 构建响应并将其返回给客户端。

可以对 docs 数组中每个文档设置 routing 参数



**bulk API**， 允许在单个批量请求中执行多个创建、索引、删除和更新请求

bulk API 按如下步骤顺序执行：

*  客户端向 Node 1 发送 bulk 请求。

* Node 1 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。

* 主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。 一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

### 分片原理

#### 倒排索引

正向索引，就是搜索引擎会将待搜索的文件都对应一个文件ID，搜索时将这个ID和搜索关键字进行对应，形成K-V对，然后对关键字进行统计计数

倒排索引, 把文件ID对应到关键词的映射转换为关键词到文件ID的映射，每个关键词都对应着一系列的文件，这些文件中都出现这个关键词。



分词和标准化的过程称为**分析**

这非常重要。你只能搜索在索引中出现的词条，所以索引文本和查询字符串必须标准化为相同的格式。

#### 文档搜索

早期的全文检索会为整个文档集合建立一个很大的倒排索引并将其写入到磁盘。 一旦新的索引就绪，旧的就会被其替换，这样最近的变化便可以被检索到。

倒排索引被写入磁盘后是 不可改变 的:它永远不会修改。

不变性有重要的价值：

* 不需要锁。如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。

* 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升。

* 其它缓存(像filter缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化。

* 写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和 需要被缓存到内存的索引的使用量。

当然，一个不变的索引也有不好的地方。主要事实是它是不可变的! 你不能修改它。如果你需要让一个新的文档 可被搜索，你需要重建整个索引。这要么对一个索引所能包含的数据量造成了很大的限制，要么对索引可被更新的频率造成了很大的限制。



#### 动态更新索引

保留不变性的前提下实现倒排索引的更新

答案是: 用更多的索引。通过增加新的补充索引来反映新近的修改，而不是直接重写整个倒排索引。每一个倒排索引都会被轮流查询到，从最早的开始查询完后再对结果进行合并。

Elasticsearch 基于 Lucene, 这个java库引入了**按段搜索**的概念。 每一 段 本身都是一个倒排索引， 但索引在 Lucene 中除表示所有段的集合外， 还增加了提交点的概念 — 一个列出了所有已知段的文件



这种方式可以用相对较低的成本将新文档添加到索引。

段是不可改变的，所以既不能从把文档从旧的段中移除，也不能修改旧的段来进行反映文档的更新。 取而代之的是，每个提交点会包含一个 .del 文件，文件中会列出这些被删除文档的段信息。

当一个文档被 “删除” 时，它实际上只是在 .del 文件中被 标记 删除。一个被标记删除的文档仍然可以被查询匹配到， 但它会在最终结果被返回前从结果集中移除。

文档更新也是类似的操作方式：当一个文档被更新时，旧版本文档被标记删除，文档的新版本被索引到一个新的段中。 可能两个版本的文档都会被一个查询匹配到，但被删除的那个旧版本文档在结果集返回前就已经被移除。



#### 近实时搜索

随着按段（per-segment）搜索的发展，一个新的文档从索引到可被搜索的延迟显著降低了。新文档在几分钟之内即可被检索，但这样还是不够快。磁盘在这里成为了瓶颈。提交（Commiting）一个新的段到磁盘需要一个 fsync 来确保段被物理性地写入磁盘，这样在断电的时候就不会丢失数据。 但是 fsync 操作代价很大; 如果每次索引一个文档都去执行一次的话会造成很大的性能问题。

我们需要的是一个更轻量的方式来使一个文档可被搜索，这意味着 fsync 要从整个过程中被移除。在Elasticsearch和磁盘之间是文件系统缓存。 像之前描述的一样， 在内存索引缓冲区中的文档会被写入到一个新的段中。 但是这里新段会被先写入到文件系统缓存—这一步代价会比较低，稍后再被刷新到磁盘—这一步代价比较高。不过只要文件已经在缓存中， 就可以像其它文件一样被打开和读取了。



在 Elasticsearch 中，写入和打开一个新段的轻量的过程叫做 refresh 。 默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 Elasticsearch 是 近 实时搜索: 文档的变化并不是立即对搜索可见，但会在一秒之内变为可见。

这些行为可能会对新用户造成困惑: 他们索引了一个文档然后尝试搜索它，但却没有搜到。这个问题的解决办法是用 refresh API 执行一次手动刷新: /users/_refresh

尽管刷新是比提交轻量很多的操作，它还是会有性能开销。当写测试的时候， 手动刷新很有用，但是不要在生产环境下每次索引一个文档都去手动刷新。 相反，你的应用需要意识到 Elasticsearch 的近实时的性质，并接受它的不足。

并不是所有的情况都需要每秒刷新。可能你正在使用 Elasticsearch 索引大量的日志文件， 你可能想优化索引速度而不是近实时搜索， 可以通过设置 refresh_interval ， 降低每个索引的刷新频率

```
{
  "settings": {
    "refresh_interval": "30s" 
  }
}
```

refresh_interval 可以在既存索引上进行动态更新。 在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来

```
# 关闭自动刷新
PUT /users/_settings
{ "refresh_interval": -1 } 

# 每一秒刷新
PUT /users/_settings
{ "refresh_interval": "1s" } 
```

#### 持久化变更

如果没有用 fsync 把数据从文件系统缓存刷（flush）到硬盘，我们不能保证数据在断电甚至是程序正常退出之后依然存在。为了保证 Elasticsearch 的可靠性，需要确保数据变化被持久化到磁盘。在 动态更新索引，我们说一次完整的提交会将段刷到磁盘，并写入一个包含所有段列表的提交点。Elasticsearch 在启动或重新打开一个索引的过程中使用这个提交点来判断哪些段隶属于当前分片。

即使通过每秒刷新（refresh）实现了近实时搜索，我们仍然需要经常进行完整提交来确保能从失败中恢复。但在两次提交之间发生变化的文档怎么办？我们也不希望丢失掉这些数据。Elasticsearch 增加了一个 translog ，或者叫事务日志，在每一次对 Elasticsearch 进行操作时均进行了日志记录



整个流程如下：

1. 一个文档被索引之后，就会被添加到内存缓冲区，并且追加到了 translog

2. 刷新（refresh）使分片每秒被刷新（refresh）一次：

   * 这些在内存缓冲区的文档被写入到一个新的段中，且没有进行 fsync 操作。

   * 这个段被打开，使其可被搜索

   * 内存缓冲区被清空

3.  这个进程继续工作，更多的文档被添加到内存缓冲区和追加到事务日志

4. 每隔一段时间—例如 translog 变得越来越大—索引被刷新（flush）；一个新的 translog 被创建，并且一个全量提交被执行

   * 所有在内存缓冲区的文档都被写入一个新的段。

   * 缓冲区被清空。

   * 一个提交点被写入硬盘。

   * 文件系统缓存通过 fsync 被刷新（flush）。

   * 老的 translog 被删除



translog 提供所有还没有被刷到磁盘的操作的一个持久化纪录。当 Elasticsearch 启动的时候， 它会从磁盘中使用最后一个提交点去恢复已知的段，并且会重放 translog 中所有在最后一次提交后发生的变更操作。

translog 也被用来提供实时 CRUD 。当你试着通过ID查询、更新、删除一个文档，它会在尝试从相应的段中检索之前， 首先检查 translog 任何最近的变更。这意味着它总是能够实时地获取到文档的最新版本

执行一个提交并且截断 translog 的行为在 Elasticsearch 被称作一次 flush

分片每30分钟被自动刷新（flush），或者在 translog 太大的时候也会刷新

你很少需要自己手动执行 flush 操作；通常情况下，自动刷新就足够了。这就是说，在重启节点或关闭索引之前执行 flush 有益于你的索引。当 Elasticsearch 尝试恢复或重新打开一个索引， 它需要重放 translog 中所有的操作，所以如果日志越短，恢复越快。

translog 的目的是保证操作不会丢失，在文件被 fsync 到磁盘前，被写入的文件在重启之后就会丢失。默认 translog 是每 5 秒被 fsync 刷新到硬盘， 或者在每次写请求完成之后执行(e.g. index, delete, update, bulk)。这个过程在主分片和复制分片都会发生。最终， 基本上，这意味着在整个请求被 fsync 到主分片和复制分片的translog之前，你的客户端不会得到一个 200 OK 响应。

在每次请求后都执行一个 fsync 会带来一些性能损失，尽管实践表明这种损失相对较小（特别是bulk导入，它在一次请求中平摊了大量文档的开销）。

但是对于一些大容量的偶尔丢失几秒数据问题也并不严重的集群，使用异步的 fsync 还是比较有益的。比如，写入的数据被缓存到内存中，再每5秒执行一次 fsync 。如果你决定使用异步 translog 的话，你需要 保证 在发生crash时，丢失掉 sync_interval 时间段的数据也无所谓。请在决定前知晓这个特性。如果你不确定这个行为的后果，最好是使用默认的参数（ "index.translog.durability": "request" ）来避免数据丢失

#### 段合并

由于自动刷新流程每秒会创建一个新的段 ，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。 每一个段都会消耗文件句柄、内存和cpu运行周期。更重要的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。



Elasticsearch通过在后台进行段合并来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段

被删除的文档（或被更新文档的旧版本）不会被拷贝到新的大段中



合并大的段需要消耗大量的I/O和CPU资源，如果任其发展会影响搜索性能。Elasticsearch在默认情况下会对合并流程进行资源限制，所以搜索仍然 有足够的资源很好地执行



### 文档分析

分析 包含下面的过程：

* 将一块文本分成适合于倒排索引的独立的 词条

*  将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 recall

分析器执行上面的工作。分析器实际上是将三个功能封装到了一个包里：

* 字符过滤器

  首先，字符串按顺序通过每个 字符过滤器 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 and

* 分词器

  其次，字符串被 分词器 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条

* Token过滤器

  最后，词条按顺序通过每个 token 过滤器 。这个过程可能会改变词条（例如，小写化 Quick ），删除词条（例如， 像 a， and， the 等无用词），或者增加词条（例如，像 jump 和 leap 这种同义词）

#### 内置分析器

Elasticsearch还附带了可以直接使用的预包装的分析器



"Set the shape to semi-transparent by calling set_trans(5)"

* 标准分析器

  标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 Unicode 联盟 定义的 单词边界 划分文本。删除绝大部分标点。最后，将词条小写。它会产生：

  ​	set, the, shape, to, semi, transparent, by, calling, set_trans, 5

* 简单分析器

  简单分析器在任何不是字母的地方分隔文本，将词条小写。它会产生：

  ​	set, the, shape, to, semi, transparent, by, calling, set, trans

* 空格分析器

  空格分析器在空格的地方划分文本。它会产生：

  ​	Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

* 语言分析器

  特定语言分析器可用于 很多语言。它们可以考虑指定语言的特点。例如， 英语 分析器附带了一组英语无用词（常用单词，例如 and 或者 the ，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 词干 。

  英语 分词器会产生下面的词条：

  ​	set, shape, semi, transpar, call, set_tran, 5

  ​	注意看 transparent、 calling 和 set_trans 已经变为词根格式



#### 分析器使用场景

当我们 索引 一个文档，它的全文域被分析成词条以用来创建倒排索引。 但是，当我们在全文域 搜索 的时候，我们需要将查询字符串通过 相同的分析过程 ，以保证我们搜索的词条格式与索引中的词条格式一致。

全文查询，理解每个域是如何定义的，因此它们可以做正确的事：

*  当你查询一个 全文 域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。

* 当你查询一个 精确值 域时，不会分析查询字符串，而是搜索你指定的精确值

#### 测试分析器

可以使用 analyze API 来看文本是如何被分析的。在消息体里，指定分析器和要分析的文本

```shell
GET http://localhost:9200/_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}

#结果中每个元素代表一个单独的词条：
#token 是实际存储到索引中的词条。 position 指明词条在原始文本中出现的位置。 start_offset 和 end_offset 指明字符在原始字符串中的位置
{
   "tokens": [
      {
         "token":        "text",
         "start_offset": 0,
         "end_offset":   4,
         "type":         "<ALPHANUM>",
         "position":     1
      },
      {
         "token":        "to",
         "start_offset": 5,
         "end_offset":   7,
         "type":         "<ALPHANUM>",
         "position":     2
      },
      {
         "token":        "analyze",
         "start_offset": 8,
         "end_offset":   15,
         "type":         "<ALPHANUM>",
         "position":     3
      }
   ]
}

```

#### IK分词器

IK中文分词器，下载地址为: [https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.8.0](https://github.com/medcl/elasticsearch-analysis-ik)

下载与 es对应的版本

安装后拷贝到 plugins 目录下

```shell
# GET http://localhost:9200/_analyze
{
	"text":"测试单词",
	"analyzer":"ik_max_word"
}

#ik_max_word：会将文本做最细粒度的拆分
#ik_smart：会将文本做最粗粒度的拆分
```

**扩展词汇**

首先进入ES根目录中的plugins文件夹下的ik文件夹，进入config目录，创建custom.dic文件，写入弗雷尔卓德。同时打开IKAnalyzer.cfg.xml文件，将新建的custom.dic配置其中，重启ES服务器

**自定义词库**

修改 /usr/share/elasticsearch/plugins/ik/config/中的 IKAnalyzer.cfg.xml

/usr/share/elasticsearch/plugins/ik/config

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 使用nginx-->
         <entry key="remote_ext_dict">http://192.168.56.10/es/fenci.txt</entry>
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>

```



#### 自定义分析器

Elasticsearch真正的强大之处在于，你可以通过在一个适合你的特定数据的设置之中组合字符过滤器、分词器、词汇单元过滤器来创建自定义的分析器



**字符过滤器**

字符过滤器 用来 整理 一个尚未被分词的字符串。例如，如果我们的文本是HTML格式的，它会包含像 <p> 或者 <div> 这样的HTML标签，这些标签是我们不想索引的。我们可以使用 html清除 字符过滤器 来移除掉所有的HTML标签，并且像把 &Aacute; 转换为相对应的Unicode字符 Á 这样，转换HTML实体。一个分析器可能有0个或者多个字符过滤器。



 **分词器**

一个分析器 必须 有一个唯一的分词器。 分词器把字符串分解成单个词条或者词汇单元。 标准 分析器里使用的 标准 分词器 把一个字符串根据单词边界分解成单个词条，并且移除掉大部分的标点符号，然而还有其他不同行为的分词器存在。

例如， 关键词 分词器 完整地输出 接收到的同样的字符串，并不做任何分词。 空格 分词器 只根据空格分割文本 。 正则 分词器 根据匹配正则表达式来分割文本 。



**词单元过滤器**

经过分词，作为结果的 词单元流 会按照指定的顺序通过指定的词单元过滤器 。

词单元过滤器可以修改、添加或者移除词单元。我们已经提到过 lowercase 和 stop 词过滤器 ，但是在 Elasticsearch 里面还有很多可供选择的词单元过滤器。 词干过滤器 把单词 遏制 为 词干。 ascii_folding 过滤器移除变音符，把一个像 "très" 这样的词转换为 "tres" 。 ngram 和 edge_ngram 词单元过滤器 可以产生 适合用于部分匹配或者自动补全的词单元



创建自定义的分析器

```shell
# PUT http://localhost:9200/my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
#测试
# GET http://127.0.0.1:9200/my_index/_analyze
{
    "text":"The quick & brown fox",
    "analyzer": "my_analyzer"
}
#返回
{
    "tokens": [
        {
            "token": "quick",
            "start_offset": 4,
            "end_offset": 9,
            "type": "<ALPHANUM>",
            "position": 1
        },
        {
            "token": "and",
            "start_offset": 10,
            "end_offset": 11,
            "type": "<ALPHANUM>",
            "position": 2
        },
        {
            "token": "brown",
            "start_offset": 12,
            "end_offset": 17,
            "type": "<ALPHANUM>",
            "position": 3
        },
        {
            "token": "fox",
            "start_offset": 18,
            "end_offset": 21,
            "type": "<ALPHANUM>",
            "position": 4
        }
    ]
}

```

### 文档处理

#### 文档冲突

当我们使用 index API 更新文档 ，可以一次性读取原始文档，做我们的修改，然后重新索引 整个文档 。 最近的索引请求将获胜：无论最后哪一个文档被索引，都将被唯一存储在 Elasticsearch 中。如果其他人同时更改这个文档，他们的更改将丢失。

很多时候这是没有问题的。也许我们的主数据存储是一个关系型数据库，我们只是将数据复制到 Elasticsearch 中并使其可被搜索。 也许两个人同时更改相同的文档的几率很小。或者对于我们的业务来说偶尔丢失更改并不是很严重的问题。

但有时丢失了一个变更就是 非常严重的 。试想我们使用 Elasticsearch 存储我们网上商城商品库存的数量， 每次我们卖一个商品的时候，我们在 Elasticsearch 中将库存数量减少。有一天，管理层决定做一次促销。突然地，我们一秒要卖好几个商品。 假设有两个 web 程序并行运行，每一个都同时处理所有商品的销售



在数据库领域中，有两种方法通常被用来确保并发更新时变更不会丢失：

**悲观并发控制**

这种方法被关系型数据库广泛使用，它假定有变更冲突可能发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。

**乐观并发控制**

Elasticsearch 中使用的这种方法假定冲突是不可能发生的，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户

#### 乐观并发控制

Elasticsearch 是分布式的。当文档创建、更新或删除时， 新版本的文档必须复制到集群中的其他节点。Elasticsearch 也是异步和并发的，这意味着这些复制请求被并行发送，并且到达目的地时也许 顺序是乱的 。 Elasticsearch 需要一种方法确保文档的旧版本不会覆盖新的版本。

当我们之前讨论 index ， GET 和 delete 请求时，我们指出每个文档都有一个 _version （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 version 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

我们可以利用 version 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 version 号来达到这个目的。 如果该版本不是当前版本号，我们的请求将会失败。

老的版本es使用version，但是新版本不支持了，会报下面的错误，提示我们用if_seq_no和if_primary_term

#### 外部系统版本控制

一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索， 这意味着主数据库的所有更改发生时都需要被复制到 Elasticsearch ，如果多个进程负责这一数据同步，你可能遇到类似于之前描述的并发问题。

如果你的主数据库已经有了版本号 — 或一个能作为版本号的字段值比如 timestamp — 那么你就可以在 Elasticsearch 中通过增加 version_type=external 到查询字符串的方式重用这些相同的版本号， 版本号必须是大于零的整数， 且小于 9.2E+18 — 一个 Java 中 long 类型的正值。

外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同， Elasticsearch 不是检查当前 _version 和请求中指定的版本号是否相同， 而是检查当前 _version 是否 小于 指定的版本号。 如果请求成功，外部的版本号作为文档的新 _version 进行存储。

外部版本号不仅在索引和删除请求是可以指定，而且在 创建 新文档时也可以指定。

## 集成



### Elasticsearch - Rest - client

1. 9300：TCP

   Spring-data-elasticsearch:transport-api.jar

   SpringBoot版本不同，`transport-api.jar` 不同，不能适配 es 版本

   7.x 已经不在适合使用，8 以后就要废弃

2. 9200：HTTP

   JestClient 非官方，更新慢

   RestTemplate:默认发送 HTTP 请求，ES很多操作都需要自己封装、麻烦

   HttpClient：同上

   **Elasticsearch - Rest - Client：官方RestClient，封装了 ES 操作，API层次分明**

最终选择 Elasticsearch - Rest - Client （elasticsearch - rest - high - level - client）

### SpringBoot 整合

```xml
<!-- 导入es的 rest-high-level-client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.4.2</version>
</dependency>
```

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-maven.html

```java
 * 1、导入配置
 * 2、编写配置，给容器注入一个RestHighLevelClient
 * 3、参照API 官网进行开发
 */
@Configuration
public class GulimallElasticsearchConfig {


    public static final RequestOptions COMMON_OPTIONS;
    static {
        RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();
//        builder.addHeader("Authorization", "Bearer " + TOKEN);
//        builder.setHttpAsyncResponseConsumerFactory(
//                new HttpAsyncResponseConsumerFactory
//                        .HeapBufferedResponseConsumerFactory(30 * 1024 * 1024 * 1024));
        COMMON_OPTIONS = builder.build();
    }



    @Bean
    public RestHighLevelClient esRestClient() {
        RestClientBuilder builder = null;
        builder = RestClient.builder(new HttpHost("192.168.56.10", 9200, "http"));

        RestHighLevelClient client = new RestHighLevelClient(builder);
//        RestHighLevelClient client = new RestHighLevelClient(
//                RestClient.builder(
//                        new HttpHost("localhost", 9200, "http"),
//                        new HttpHost("localhost", 9201, "http")));
        return client;
    }

}
```

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-initialization.html

```java
@Autowired
private RestHighLevelClient client;
/**
 * 添加或者更新
 * @throws IOException
 */
@Test
public void indexData() throws IOException {
    IndexRequest indexRequest = new IndexRequest("users");
    User user = new User();
    user.setAge(19);
    user.setGender("男");
    user.setUserName("张三");
    String jsonString = JSON.toJSONString(user);
    indexRequest.source(jsonString,XContentType.JSON);

    // 执行操作
    IndexResponse index = client.index(indexRequest,GulimallElasticsearchConfig.COMMON_OPTIONS);

    // 提取有用的响应数据
    System.out.println(index);
}

@Test
public void searchTest() throws IOException {
    // 1、创建检索请求
    SearchRequest searchRequest = new SearchRequest();
    // 指定索引
    searchRequest.indices("bank");
    // 指定 DSL，检索条件
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    sourceBuilder.query(QueryBuilders.matchQuery("address", "mill"));

    //1、2 按照年龄值分布进行聚合
    TermsAggregationBuilder aggAvg = AggregationBuilders.terms("ageAgg").field("age").size(10);
    sourceBuilder.aggregation(aggAvg);

    //1、3 计算平均薪资
    AvgAggregationBuilder balanceAvg = AggregationBuilders.avg("balanceAvg").field("balance");
    sourceBuilder.aggregation(balanceAvg);


    System.out.println("检索条件" + sourceBuilder.toString());

    searchRequest.source(sourceBuilder);

    // 2、执行检索
    SearchResponse searchResponse = client.search(searchRequest, GulimallElasticsearchConfig.COMMON_OPTIONS);

    // 3、分析结果
    System.out.println(searchResponse.toString());

    // 4、拿到命中得结果
    SearchHits hits = searchResponse.getHits();
    // 5、搜索请求的匹配
    SearchHit[] searchHits = hits.getHits();
    // 6、进行遍历
    for (SearchHit hit : searchHits) {
        // 7、拿到完整结果字符串
        String sourceAsString = hit.getSourceAsString();
        // 8、转换成实体类
        Accout accout = JSON.parseObject(sourceAsString, Accout.class);
        System.out.println("account:" + accout );
    }

    // 9、拿到聚合
    Aggregations aggregations = searchResponse.getAggregations();
    //        for (Aggregation aggregation : aggregations) {
    //
    //        }
    // 10、通过先前名字拿到对应聚合
    Terms ageAgg1 = aggregations.get("ageAgg");
    for (Terms.Bucket bucket : ageAgg1.getBuckets()) {
        // 11、拿到结果
        String keyAsString = bucket.getKeyAsString();
        System.out.println("年龄:" + keyAsString);
        long docCount = bucket.getDocCount();
        System.out.println("个数：" + docCount);
    }
    Avg balanceAvg1 = aggregations.get("balanceAvg");
    System.out.println("平均薪资：" + balanceAvg1.getValue());
    System.out.println(searchResponse.toString());
}
```

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html#java-rest-high-search-request-optional





### Spring Data

一个用于简化数据库、非关系型数据库、索引库访问，并支持云服务的开源框架

https://spring.io/projects/spring-data

 Spring Data Elasticsearch

https://spring.io/projects/spring-data-elasticsearch

目前最新springboot对应Elasticsearch7.6.2，Spring boot2.3.x一般可以兼容Elasticsearch7.x



#### demo

##### 搭建

pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.6.RELEASE</version>
        <relativePath/>
    </parent>


    <groupId>com.atguigu.es</groupId>
    <artifactId>springdata-elasticsearch</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
        </dependency>
    </dependencies>
</project>
```

application.properties

```properties
# es服务地址
elasticsearch.host=127.0.0.1
# es服务端口
elasticsearch.port=9200
# 配置日志级别,开启debug日志
logging.level.com.atguigu.es=debug
```

数据实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Product {
    private Long id;//商品唯一标识
    private String title;//商品名称
    private String category;//分类名称
    private Double price;//商品价格
    private String images;//图片地址

}
```

配置类

* ElasticsearchRestTemplate是spring-data-elasticsearch项目中的一个类，和其他spring项目中的template类似。

* 在新版的spring-data-elasticsearch中，ElasticsearchRestTemplate代替了原来的ElasticsearchTemplate。

* 原因是ElasticsearchTemplate基于TransportClient，TransportClient即将在8.x以后的版本中移除。所以，我们推荐使用ElasticsearchRestTemplate。

* ElasticsearchRestTemplate基于RestHighLevelClient客户端的。需要自定义配置类，继承AbstractElasticsearchConfiguration，并实现elasticsearchClient()抽象方法，创建RestHighLevelClient对象。

```java
@ConfigurationProperties(prefix = "elasticsearch")
@Configuration
@Data
public class ElasticsearchConfig extends AbstractElasticsearchConfiguration {
    private String host ;
    private Integer port ;

    //重写父类方法
    @Override
    public RestHighLevelClient elasticsearchClient() {
        RestClientBuilder builder = RestClient.builder(new HttpHost(host, port));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);
        return restHighLevelClient;
    }
}
```

dao

```java
@Repository
public interface ProductDao extends ElasticsearchRepository<Product,Long> {

}
```

 实体类映射操作

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Document(indexName = "shopping", shards = 3, replicas = 1)
public class Product {
    //必须有id,这里的id是全局唯一的标识，等同于es中的"_id"
    @Id
    private Long id;//商品唯一标识
    /**
     * type : 字段数据类型
     * analyzer : 分词器类型
     * index : 是否索引(默认:true)
     * Keyword : 短语,不进行分词
     */
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String title;//商品名称
    @Field(type = FieldType.Keyword)
    private String category;//分类名称
    @Field(type = FieldType.Double)
    private Double price;//商品价格
    @Field(type = FieldType.Keyword, index = false)
    private String images;//图片地址
}
```

##### 测试

索引操作

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringDataESIndexTest {
    //注入ElasticsearchRestTemplate
    @Autowired
    private ElasticsearchRestTemplate elasticsearchRestTemplate;

    //创建索引并增加映射配置
    @Test
    public void createIndex(){
        //创建索引，系统初始化会自动创建索引
        System.out.println("创建索引");
    }

    @Test
    public void deleteIndex(){
        //创建索引，系统初始化会自动创建索引
        boolean flg = elasticsearchRestTemplate.deleteIndex(Product.class);
        System.out.println("删除索引 = " + flg);
    }
}
```

文档操作

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringDataESProductDaoTest {
    @Autowired
    private ProductDao productDao;

    /**
     * 新增
     */
    @Test
    public void save(){
        Product product = new Product();
        product.setId(2L);
        product.setTitle("华为手机");
        product.setCategory("手机");
        product.setPrice(2999.0);
        product.setImages("http://www.atguigu/hw.jpg");
        productDao.save(product);
    }

    //修改
    @Test
    public void update(){
        Product product = new Product();
        product.setId(1L);
        product.setTitle("小米2手机");
        product.setCategory("手机");
        product.setPrice(9999.0);
        product.setImages("http://www.atguigu/xm.jpg");
        productDao.save(product);
    }

    //根据id查询
    @Test
    public void findById(){
        Product product = productDao.findById(1L).get();
        System.out.println(product);
    }

    //查询所有
    @Test
    public void findAll(){
        Iterable<Product> products = productDao.findAll();
        for (Product product : products) {
            System.out.println(product);
        }
    }

    //删除
    @Test
    public void delete(){
        Product product = new Product();
        product.setId(1L);
        productDao.delete(product);
    }

    //批量新增
    @Test
    public void saveAll(){
        List<Product> productList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Product product = new Product();
            product.setId(Long.valueOf(i));
            product.setTitle("["+i+"]小米手机");
            product.setCategory("手机");
            product.setPrice(1999.0+i);
            product.setImages("http://www.atguigu/xm.jpg");
            productList.add(product);
        }
        productDao.saveAll(productList);
    }

    //分页查询
    @Test
    public void findByPageable(){
        //设置排序(排序方式，正序还是倒序，排序的id)
        Sort sort = Sort.by(Sort.Direction.DESC,"id");
        int currentPage=0;//当前页，第一页从0开始，1表示第二页
        int pageSize = 5;//每页显示多少条
        //设置查询分页
        PageRequest pageRequest = PageRequest.of(currentPage, pageSize,sort);
        //分页查询
        Page<Product> productPage = productDao.findAll(pageRequest);
        for (Product Product : productPage.getContent()) {
            System.out.println(Product);
        }
    }
}

```

文档搜索

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringDataESSearchTest {
    @Autowired
    private ProductDao productDao;

    /**
     * term查询
     * search(termQueryBuilder) 调用搜索方法，参数查询构建器对象
     */
    @Test
    public void termQuery(){
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", "小米");
        Iterable<Product> products = productDao.search(termQueryBuilder);
        for (Product product : products) {
            System.out.println(product);
        }
    }

    /**
     * term查询加分页
     */
    @Test
    public void termQueryByPage(){
        int currentPage= 0 ;
        int pageSize = 5;
        //设置查询分页
        PageRequest pageRequest = PageRequest.of(currentPage, pageSize);
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", "小米");
        Iterable<Product> products = productDao.search(termQueryBuilder,pageRequest);
        for (Product product : products) {
            System.out.println(product);
        }
    }
}
```

### Spark Streaming集成

Spark Streaming是Spark core API的扩展，支持实时数据流的处理，并且具有可扩展，高吞吐量，容错的特点。 数据可以从许多来源获取，如Kafka，Flume，Kinesis或TCP sockets，并且可以使用复杂的算法进行处理，这些算法使用诸如map，reduce，join和window等高级函数表示。 最后，处理后的数据可以推送到文件系统，数据库等。 实际上，您可以将Spark的机器学习和图形处理算法应用于数据流



#### demo

创建Maven项目 pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.es</groupId>
    <artifactId>sparkstreaming-elasticsearch</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_2.12</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.8.0</version>
        </dependency>
        <!-- elasticsearch的客户端 -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.8.0</version>
        </dependency>
        <!-- elasticsearch依赖2.x的log4j -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>com.fasterxml.jackson.core</groupId>-->
<!--            <artifactId>jackson-databind</artifactId>-->
<!--            <version>2.11.1</version>-->
<!--        </dependency>-->
<!--        &lt;!&ndash; junit单元测试 &ndash;&gt;-->
<!--        <dependency>-->
<!--            <groupId>junit</groupId>-->
<!--            <artifactId>junit</artifactId>-->
<!--            <version>4.12</version>-->
<!--        </dependency>-->
    </dependencies>
</project>

```

```java
object SparkStreamingESTest {

    def main(args: Array[String]): Unit = {
        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("ESTest")
        val ssc = new StreamingContext(sparkConf, Seconds(3))

        val ds: ReceiverInputDStream[String] = ssc.socketTextStream("localhost", 9999)

        ds.foreachRDD(
            rdd => {
                println("*************** " + new Date())
                rdd.foreach(
                    data => {
                        val client = new RestHighLevelClient(
                            RestClient.builder(new HttpHost("localhost", 9200, "http"))
                        );
                        // 新增文档 - 请求对象
                        val request = new IndexRequest();
                        // 设置索引及唯一性标识
                        val ss = data.split(" ")
                        println("ss = " + ss.mkString(","))
                        request.index("sparkstreaming").id(ss(0));
                        val productJson =
                            s"""
                              | { "data":"${ss(1)}" }
                              |""".stripMargin;
                        // 添加文档数据，数据格式为JSON格式
                        request.source(productJson,XContentType.JSON);
                        // 客户端发送请求，获取响应对象
                        val response = client.index(request, RequestOptions.DEFAULT);
                        System.out.println("_index:" + response.getIndex());
                        System.out.println("_id:" + response.getId());
                        System.out.println("_result:" + response.getResult());

                        client.close()
                    }
                )
            }
        )
        ssc.start()
        ssc.awaitTermination()
    }
}
```

### Flink集成

Apache Spark是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。

Apache Spark掀开了内存计算的先河，以内存作为赌注，赢得了内存计算的飞速发展。但是在其火热的同时，开发人员发现，在Spark中，计算框架普遍存在的缺点和不足依然没有完全解决，而这些问题随着5G时代的来临以及决策者对实时数据分析结果的迫切需要而凸显的更加明显

* 数据精准一次性处理（Exactly-Once）

*  乱序数据，迟到数据

* 低延迟，高吞吐，准确性

*  容错性

Apache Flink是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算。在Spark火热的同时，也默默地发展自己，并尝试着解决其他计算框架的问题。

慢慢地，随着这些问题的解决，Flink慢慢被绝大数程序员所熟知并进行大力推广，阿里公司在2015年改进Flink，并创建了内部分支Blink，目前服务于阿里集团内部搜索、推荐、广告和蚂蚁等大量核心实时业务。

#### demo

Maven项目pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.es</groupId>
    <artifactId>flink-elasticsearch</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-scala_2.12</artifactId>
            <version>1.12.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_2.12</artifactId>
            <version>1.12.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_2.12</artifactId>
            <version>1.12.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-elasticsearch7_2.11</artifactId>
            <version>1.12.0</version>
        </dependency>

        <!-- jackson -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.11.1</version>
        </dependency>
    </dependencies>
</project>

```

```java
public class FlinkElasticsearchSinkTest {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> source = env.socketTextStream("localhost", 9999);

        List<HttpHost> httpHosts = new ArrayList<>();
        httpHosts.add(new HttpHost("127.0.0.1", 9200, "http"));
        //httpHosts.add(new HttpHost("10.2.3.1", 9200, "http"));

// use a ElasticsearchSink.Builder to create an ElasticsearchSink
        ElasticsearchSink.Builder<String> esSinkBuilder = new ElasticsearchSink.Builder<>(
                httpHosts,
                new ElasticsearchSinkFunction<String>() {
                    public IndexRequest createIndexRequest(String element) {
                        Map<String, String> json = new HashMap<>();
                        json.put("data", element);

                        return Requests.indexRequest()
                                .index("my-index")
                                //.type("my-type")
                                .source(json);
                    }

                    @Override
                    public void process(String element, RuntimeContext ctx, RequestIndexer indexer) {
                        indexer.add(createIndexRequest(element));
                    }
                }
        );

// configuration for the bulk requests; this instructs the sink to emit after every element, otherwise they would be buffered
        esSinkBuilder.setBulkFlushMaxActions(1);

// provide a RestClientFactory for custom configuration on the internally created REST client
//        esSinkBuilder.setRestClientFactory(
//                restClientBuilder -> {
//                    restClientBuilder.setDefaultHeaders(...)
//                    restClientBuilder.setMaxRetryTimeoutMillis(...)
//                    restClientBuilder.setPathPrefix(...)
//                    restClientBuilder.setHttpClientConfigCallback(...)
//                }
//        );

        source.addSink(esSinkBuilder.build());

        env.execute("flink-es");
    }
}

```

## 优化

### 硬件选择

Elasticsearch的基础是 Lucene，所有的索引和文档数据是存储在本地的磁盘中，具体的路径可在 ES 的配置文件../config/elasticsearch.yml中配置path.data



1. 使用 SSD
2. 使用 RAID 0。条带化 RAID 会提高磁盘 I/O，代价显然就是当一块硬盘故障时整个就故障了。不要使用镜像或者奇偶校验 RAID 因为副本已经提供了这个功能。
3. 另外，使用多块硬盘，并允许 Elasticsearch 通过多个 path.data 目录配置把数据条带化分配到它们上面。
4. 不要使用远程挂载的存储，比如 NFS 或者 SMB/CIFS。这个引入的延迟对性能来说完全是背道而驰的。

### 分片策略

#### 合理设置分片数

分片和副本的设计为 ES 提供了支持分布式和故障转移的特性，但并不意味着分片和副本是可以无限分配的。而且索引的分片完成分配后由于索引的路由机制，我们是不能重新修改分片数的。

一个分片并不是没有代价的

* 一个分片的底层即为一个 Lucene 索引，会消耗一定文件句柄、内存、以及 CPU 运转。

* 每一个搜索请求都需要命中索引中的每一个分片，如果每一个分片都处于不同的节点还好， 但如果多个分片都需要在同一个节点上竞争使用相同的资源就有些糟糕了。 

*  用于计算相关度的词项统计信息是基于分片的。如果有许多分片，每一个都只有很少的数据会导致很低的相关度。



一个业务索引具体需要分配多少分片可能需要架构师和技术人员对业务的增长有个预先的判断，横向扩展应当分阶段进行。为下一阶段准备好足够的资源。 只有当你进入到下一个阶段，你才有时间思考需要作出哪些改变来达到这个阶段。一般来说，我们遵循一些原则：

* 控制每个分片占用的硬盘容量不超过ES的最大JVM的堆空间设置（一般设置不超过32G，参考下文的JVM设置原则），因此，如果索引的总容量在500G左右，那分片大小在16个左右即可；当然，最好同时考虑原则2。

* 考虑一下node数量，一般一个节点有时候就是一台物理机，如果分片数过多，大大超过了节点数，很可能会导致一个节点上存在多个分片，一旦该节点故障，即使保持了1个以上的副本，同样有可能会导致数据丢失，集群无法恢复。所以， 一般都设置分片数不超过节点数的3倍。

*  主分片，副本和节点最大数之间数量，我们分配的时候可以参考以下关系：节点数<=主分片数*（副本数+1）

#### 推迟分片分配

对于节点瞬时中断的问题，默认情况，集群会等待一分钟来查看节点是否会重新加入，如果这个节点在此期间重新加入，重新加入的节点会保持其现有的分片数据，不会触发新的分片分配。这样就可以减少 ES 在自动再平衡可用分片时所带来的极大开销。

通过修改参数 delayed_timeout ，可以延长再均衡的时间，可以全局设置也可以在索引级别进行修改:

```shell
PUT /_all/_settings 
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m" 
  }
}
```

### 路由选择

当我们查询文档的时候，Elasticsearch 如何知道一个文档应该存放到哪个分片中呢？它其实是通过下面这个公式来计算出来:

shard = hash(routing) % number_of_primary_shards

routing 默认值是文档的 id，也可以采用自定义值，比如用户 id。

**不带 routing** **查询**

在查询的时候因为不知道要查询的数据具体在哪个分片上，所以整个过程分为 2 个步骤

* 分发：请求到达协调节点后，协调节点将查询请求分发到每个分片上。

*  聚合: 协调节点搜集到每个分片上查询结果，在将查询的结果进行排序，之后给用户返回结果。

**带 routing** **查询**

查询的时候，可以直接根据 routing 信息定位到某个分配查询，不需要查询所有的分配，经过协调节点排序。

向上面自定义的用户查询，如果 routing 设置为 userid 的话，就可以直接查询出数据来，效率提升很多。

### 写入速度优化

ES的默认配置，是综合了数据可靠性、写入速度、搜索实时性等因素。实际使用时，我们需要根据公司要求，进行偏向性的优化。

针对于搜索性能要求不高，但是对写入要求较高的场景，我们需要尽可能的选择恰当写优化策略。综合来说，可以考虑以下几个方面来提升写索引的性能：

* 加大 Translog Flush ，目的是降低 Iops、Writeblock。

* 增加 Index Refresh 间隔，目的是减少 Segment Merge 的次数。
* 调整 Bulk 线程池和队列。
* 优化节点间的任务分布。
* 优化 Lucene 层的索引建立，目的是降低 CPU 及 IO。

#### 批量数据提交

ES 提供了 Bulk API 支持批量操作，当我们有大量的写任务时，可以使用 Bulk 来进行批量写入。

通用的策略如下：Bulk 默认设置批量提交的数据量不能超过 100M。数据条数一般是根据文档的大小和服务器性能而定的，但是单次批处理的数据大小应从 5MB～15MB 逐渐增加，当性能没有提升时，把这个数据量作为最大值

#### 优化存储设备

ES 是一种密集使用磁盘的应用，在段合并的时候会频繁操作磁盘，所以对磁盘要求较高，当磁盘速度提升之后，集群的整体性能会大幅度提高

#### 合理使用合并

Lucene 以段的形式存储数据。当有新的数据写入索引时，Lucene 就会自动创建一个新的段。

随着数据量的变化，段的数量会越来越多，消耗的多文件句柄数及 CPU 就越多，查询效率就会下降。

由于 Lucene 段合并的计算量庞大，会消耗大量的 I/O，所以 ES 默认采用较保守的策略，让后台定期进行段合并

#### 减少Refresh的次数

Lucene 在新增数据时，采用了延迟写入的策略，默认情况下索引的 refresh_interval 为 1 秒。

Lucene 将待写入的数据先写到内存中，超过 1 秒（默认）时就会触发一次 Refresh，然后 Refresh 会把内存中的的数据刷新到操作系统的文件缓存系统中。

如果我们对搜索的实效性要求不高，可以将 Refresh 周期延长，例如 30 秒。

这样还可以有效地减少段刷新次数，但这同时意味着需要消耗更多的Heap内存

#### 加大Flush设置

Flush 的主要目的是把文件缓存系统中的段持久化到硬盘，当 Translog 的数据量达到 512MB 或者 30 分钟时，会触发一次 Flush。

index.translog.flush_threshold_size 参数的默认值是 512MB，我们进行修改。

增加参数值意味着文件缓存系统中可能需要存储更多的数据，所以我们需要为操作系统的文件缓存系统留下足够的空间

#### 减少副本的数量

ES 为了保证集群的可用性，提供了 Replicas（副本）支持，然而每个副本也会执行分析、索引及可能的合并过程，所以 Replicas 的数量会严重影响写索引的效率。

当写索引时，需要把写入的数据都同步到副本节点，副本节点越多，写索引的效率就越慢。

如果我们需要大批量进行写入操作，可以先禁止 Replica 复制，设置 index.number_of_replicas: 0 关闭副本。在写入完成后，Replica 修改回正常的状态

### 内存设置

ES 默认安装后设置的内存是 1GB，对于任何一个现实业务来说，这个设置都太小了。

如果是通过解压安装的 ES，则在 ES 安装文件中包含一个 jvm.option 文件，添加如下命令来设置 ES 的堆大小，Xms 表示堆的初始大小，Xmx 表示可分配的最大内存，都是 1GB。

确保 Xmx 和 Xms 的大小是相同的，其目的是为了能够在 Java 垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源，可以减轻伸缩堆大小带来的压力。



ES 堆内存的分配需要满足以下两个原则：

* 不要超过物理内存的 50%：Lucene 的设计目的是把底层 OS 里的数据缓存到内存中。

  Lucene 的段是分别存储到单个文件中的，这些文件都是不会变化的，所以很利于缓存，同时操作系   统也会把这些段文件缓存起来，以便更快的访问。

  如果我们设置的堆内存过大，Lucene 可用的内存将会减少，就会严重影响降低 Lucene 的全文本查  询性能。

* 堆内存的大小最好不要超过 32GB：在 Java 中，所有对象都分配在堆上，然后有一个 Klass Pointer 指针指向它的类元数据。

  这个指针在 64 位的操作系统上为 64 位，64 位的操作系统可以使用更多的内存（2^64）。在 32 位  的系统上为 32 位，32 位的操作系统的最大寻址空间为 4GB（2^32）。

  但是 64 位的指针意味着更大的浪费，因为你的指针本身大了。浪费内存不算，更糟糕的是，更大的   指针在主内存和缓存器（例如 LLC, L1等）之间移动数据的时候，会占用更多的带宽。

  

最终我们都会采用 31 G 设置

  -Xms 31g

  -Xmx 31g

假设你有个机器有 128 GB 的内存，你可以创建两个节点，每个节点内存分配不超过 32 GB。 也就是说不超过 64 GB 内存给 ES 的堆内存，剩下的超过 64 GB 的内存给 Lucene

### 重要配置

| 参数名                             | 参数值        | 说明                                                         |
| ---------------------------------- | ------------- | ------------------------------------------------------------ |
| cluster.name                       | elasticsearch | 配置 ES 的集群名称，默认值是ES，建议改成与所存数据相关的名称，ES 会自动发现在同一网段下的集群名称相同的节点 |
| node.name                          | node-1        | 集群中的节点名，在同一个集群中不能重复。节点的名称一旦设置，就不能再改变了。当然，也可以设置成服务器的主机名称，例如 node.name:${HOSTNAME}。 |
| node.master                        | true          | 指定该节点是否有资格被选举成为 Master 节点，默认是  True，如果被设置为 True，则只是有资格成为 Master 节点，具体能否成为  Master 节点，需要通过选举产生。 |
| node.data                          | true          | 指定该节点是否存储索引数据，默认为 True。数据的增、删、改、查都是在 Data 节点完成的。 |
| index.number_of_shards             | 1             | 设置都索引分片个数，默认是 1 片。也可以在创建索引时设置该值，具体设置为多大都值要根据数据量的大小来定。如果数据量不大，则设置成 1 时效率最高 |
| index.number_of_replicas           | 1             | 设置默认的索引副本个数，默认为 1 个。副本数越多，集群的可用性越好，但是写索引时需要同步的数据越多。 |
| transport.tcp.compress             | true          | 设置在节点间传输数据时是否压缩，默认为 False，不压缩         |
| discovery.zen.minimum_master_nodes | 1             | 设置在选举 Master 节点时需要参与的最少的候选主节点数，默认为 1。如果使用默认值，则当网络不稳定时有可能会出现脑裂。  合理的数值为(master_eligible_nodes/2)+1，其中 master_eligible_nodes 表示集群中的候选主节点数 |
| discovery.zen.ping.timeout         | 3s            | 设置在集群中自动发现其他节点时 Ping 连接的超时时间，默认为  3 秒。  在较差的网络环境下需要设置得大一点，防止因误判该节点的存活状态而导致分片的转移 |

## 面试题

### 为什么要使用Elasticsearch?

使用ES做一个全文索引，提高查询速度。

### Elasticsearch的master选举流程？

* Elasticsearch的选主是ZenDiscovery模块负责的，主要包含Ping（节点之间通过这个RPC来发现彼此）和Unicast（单播模块包含一个主机列表以控制哪些节点需要ping通）这两部分
* 对所有可以成为master的节点（node.master: true）根据nodeId字典排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个（第0位）节点，暂且认为它是master节点。
* 如果对某个节点的投票数达到一定的值（可以成为master节点数n/2+1）并且该节点自己也选举自己，那这个节点就是master。否则重新选举一直到满足上述条件。
* master节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data节点可以关闭http功能。

### Elasticsearch集群脑裂问题？

“脑裂”问题可能的成因:

* 网络问题
* 节点负载：主节点的角色既为master又为data，访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点。
* 内存回收：data节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应。

脑裂问题解决方案：
* 减少误判：discovery.zen.ping_timeout节点状态的响应时间，默认为3s，可以适当调大
* 选举触发: discovery.zen.minimum_master_nodes:1
    该参数是用于控制选举行为发生的最小集群主节点数量。当备选主节点的个数大于等于该参数的值，	且备选主节点中有该参数个节点认为主节点挂了，进行选举。官方建议为（n/2）+1，n为主节点个数	（即有资格成为主节点的节点个数）
* 角色分离：即master节点与data节点分离，限制角色
    主节点配置为：node.master: true node.data: false
    从节点配置为：node.master: false node.data: true

### Elasticsearch索引文档的流程？

* 协调节点默认使用文档ID参与计算（也支持通过routing），以便为路由提供合适的分片：
  shard = hash(document_id) % (num_of_primary_shards)
* 当分片所在的节点接收到来自协调节点的请求后，会将请求写入到Memory Buffer，然后定时（默认是每隔1秒）写入到Filesystem Cache，这个从Memory Buffer到Filesystem Cache的过程就叫做refresh；
* 当然在某些情况下，存在Momery Buffer和Filesystem Cache的数据可能会丢失，ES是通过translog的机制来保证数据的可靠性的。其实现机制是接收到请求后，同时也会写入到translog中，当Filesystem cache中的数据写入到磁盘中时，才会清除掉，这个过程叫做flush；
* 在flush过程中，内存中的缓冲将被清除，内容被写入一个新段，段的fsync将创建一个新的提交点，并将内容刷新到磁盘，旧的translog将被删除并开始一个新的translog。
* flush触发的时机是定时触发（默认30分钟）或者translog变得太大（默认为512M）时；  



### Elasticsearch更新和删除文档的流程？

* 删除和更新也都是写操作，但是Elasticsearch中的文档是不可变的，因此不能被删除或者改动以展示其变更；
* 磁盘上的每个段都有一个相应的.del文件。当删除请求发送后，文档并没有真的被删除，而是在.del文件中被标记为删除。该文档依然能匹配查询，但是会在结果中被过滤掉。当段合并时，在.del文件中被标记为删除的文档将不会被写入新段。
* 在新的文档被创建时，Elasticsearch会为该文档指定一个版本号，当执行更新时，旧版本的文档在.del文件中被标记为删除，新版本的文档被索引到一个新段。旧版本的文档依然能匹配查询，但是会在结果中被过滤掉。 

### Elasticsearch搜索的流程？

* 搜索被执行成一个两阶段过程，我们称之为 Query Then Fetch；
* 在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 每个分片在本地执行搜索并构建一个匹配文档的大小为 from + size 的优先队列。PS：在搜索的时候是会查询Filesystem Cache的，但是有部分数据还在Memory Buffer，所以搜索是近实时的。
* 每个分片返回各自优先队列中 所有文档的 ID 和排序值 给协调节点，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。
* 接下来就是取回阶段，协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。每个分片加载并丰富文档，如果有需要的话，接着返回文档给协调节点。一旦所有的文档都被取回了，协调节点返回结果给客户端。
* Query Then Fetch的搜索类型在文档相关性打分的时候参考的是本分片的数据，这样在文档数量较少的时候可能不够准确，DFS Query Then Fetch增加了一个预查询的处理，询问Term和Document frequency，这个评分更准确，但是性能会变差。 

### Elasticsearch 在部署时，对 Linux 的设置有哪些优化方法？

* 64 GB 内存/32 GB /16 GB    少于8 GB 不行。
* 更快的 CPUs 和更多的核心之间选择，选择更多的核心更好。多个内核提供的额外并发远胜过稍微快一点点的时钟频率。
* SSD
* 避免集群跨越多个数据中心。绝对要避免集群跨越大的地理距离。
* 请确保运行你应用程序的 JVM 和服务器的 JVM 是完全一样的。 在 Elasticsearch 的几个地方，使用 Java 的本地序列化。
* 通过设置gateway.recover_after_nodes、gateway.expected_nodes、gateway.recover_after_time可以在集群重启的时候避免过多的分片交换，这可能会让数据恢复从数个小时缩短为几秒钟。
* Elasticsearch 默认被配置为使用单播发现，以防止节点无意中加入集群。只有在同一台机器上运行的节点才会自动组成集群。最好使用单播代替组播。
* 不要随意修改垃圾回收器（CMS）和各个线程池的大小。
* 把你的内存的（少于）一半给 Lucene（但不要超过 32 GB！），通过ES_HEAP_SIZE 环境变量设置。
* 内存交换到磁盘对服务器性能来说是致命的。如果内存交换到磁盘上，一个 100 微秒的操作可能变成 10 毫秒。 再想想那么多 10 微秒的操作时延累加起来。 不难看出 swapping 对于性能是多么可怕。
* Lucene 使用了大量的文件。同时，Elasticsearch 在节点和 HTTP 客户端之间进行通信也使用了大量的套接字。 所有这一切都需要足够的文件描述符。你应该增加你的文件描述符，设置一个很大的值，如 64,000。



补充：索引阶段性能提升方法

* 使用批量请求并调整其大小：每次批量数据 5–15 MB 大是个不错的起始点。
* 使用 SSD

* 段和合并：Elasticsearch 默认值是 20 MB/s，对机械磁盘应该是个不错的设置。如果你用的是 SSD，可以考虑提高到 100–200 MB/s。如果你在做批量导入，完全不在意搜索，你可以彻底关掉合并限流。另外还可以增加 index.translog.flush_threshold_size 设置，从默认的 512 MB 到更大一些的值，比如 1 GB，这可以在一次清空触发的时候在事务日志里积累出更大的段。

* 如果你的搜索结果不需要近实时的准确度，考虑把每个索引的index.refresh_interval 改到30s。
* 如果你在做大批量导入，考虑通过设置index.number_of_replicas: 0 关闭副本。

### GC方面，在使用Elasticsearch时要注意什么？

* 倒排词典的索引需要常驻内存，无法GC，需要监控data node上segment memory增长趋势。
* 各类缓存，field cache, filter cache, indexing cache, bulk queue等等，要设置合理的大小，并且要应该根据最坏的情况来看heap是否够用，也就是各类缓存全部占满的时候，还有heap空间可以分配给其他任务吗？避免采用clear cache等“自欺欺人”的方式来释放内存。
* 避免返回大量结果集的搜索与聚合。确实需要大量拉取数据的场景，可以采用scan & scroll api来实现。
* cluster stats驻留内存并无法水平扩展，超大规模集群可以考虑分拆成多个集群通过tribe node连接。
* 想知道heap够不够，必须结合实际应用场景，并对集群的heap使用情况做持续的监控。

### Elasticsearch对于大数据量（上亿量级）的聚合如何实现？

  Elasticsearch 提供的首个近似聚合是cardinality 度量。它提供一个字段的基数，即该字段的distinct或者unique值的数目。它是基于HLL算法的。HLL 会先对我们的输入作哈希运算，然后根据哈希运算的结果中的 bits 做概率估算从而得到基数。其特点是：可配置的精度，用来控制内存的使用（更精确 ＝ 更多内存）；小的数据集精度是非常高的；我们可以通过配置参数，来设置去重需要的固定内存使用量。无论数千还是数十亿的唯一值，内存使用量只与你配置的精确度相关 



### 在并发情况下，Elasticsearch如果保证读写一致？

* 通过版本号使用乐观并发控制
* 另外对于写操作，一致性级别支持quorum/one/all，默认为quorum，即只有当大多数分片可用时才允许写操作。但即使大多数可用，也可能存在因为网络等原因导致写入副本失败，这样该副本被认为故障，分片将会在一个不同的节点上重建。
* 对于读操作，可以设置replication为sync(默认)，这使得操作在主分片和副本分片都完成后才会返回；如果设置replication为async时，也可以通过设置搜索请求参数_preference为primary来查询主分片，确保文档是最新版本。 

### 如何监控 Elasticsearch 集群状态？

elasticsearch-head插件
通过 Kibana 监控 Elasticsearch。你可以实时查看你的集群健康状态和性能，也可以分析过去的集群、索引和节点指标

### 是否了解字典树？

常用字典数据结构如下所示:

![image-20211102162051333](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20211102162051333.png)

==字典树又称单词查找树，Trie树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计，排序和保存大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：利用字符串的公共前缀来减少查询时间，最大限度地减少无谓的字符串比较，查询效率比哈希树高。==

* Trie的核心思想是空间换时间，利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。它有3个基本性质:
  * 根节点不包含字符，除根节点外每一个节点都只包含一个字符。
  * 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串。
  * 每个节点的所有子节点包含的字符都不相同。
* 对于中文的字典树，每个节点的子节点用一个哈希表存储，这样就不用浪费太大的空间，而且查询速度上可以保留哈希的复杂度O(1)。

### Elasticsearch中的集群、节点、索引、文档、类型是什么？

* 集群是一个或多个节点（服务器）的集合，它们共同保存您的整个数据，并提供跨所有节点的联合索引和搜索功能。群集由唯一名称标识，默认情况下为“elasticsearch”。此名称很重要，因为如果节点设置为按名称加入群集，则该节点只能是群集的一部分。
* 节点是属于集群一部分的单个服务器。它存储数据并参与群集索引和搜索功能。
* 索引就像关系数据库中的“数据库”。它有一个定义多种类型的映射。索引是逻辑名称空间，映射到一个或多个主分片，并且可以有零个或多个副本分片。 MySQL =>数据库 Elasticsearch =>索引
* 文档类似于关系数据库中的一行。不同之处在于索引中的每个文档可以具有不同的结构（字段），但是对于通用字段应该具有相同的数据类型。 MySQL => Databases => Tables => Columns / Rows Elasticsearch => Indices => Types =>具有属性的文档
* 类型是索引的逻辑类别/分区，其语义完全取决于用户。

### Elasticsearch中的倒排索引是什么

ES中的倒排索引其实就是lucene的倒排索引，区别于传统的正向索引，倒排索引会再存储数据时将关键词和数据进行关联，保存到倒排表中，然后查询时，将查询内容进行分词后在倒排表中进行查询，最后匹配数据即可。



## Elastic 8.0版

jdk17

### 安装

![image-20220727174353950](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174353950.png)

![image-20220727174506498](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174506498.png)

![image-20220727174551671](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174551671.png)

![image-20220727174745713](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174745713.png)

![image-20220727174824367](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727174824367.png)

![image-20220727180418576](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727180418576.png)

![image-20220727180759983](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727180759983.png)

![image-20220727180906870](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20220727180906870.png)



























































# 工具

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