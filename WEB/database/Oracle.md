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

## 基础
oracle 数据库与其它数据库产品的区别：表和其它的数据库对象都是存储在用户下的
```sql
--创建表空间
create tablespace itheima
datafile 'c:\itheima.dbf'
size 100m
autoextend on  //自动扩展
next 10m;  //每次10m
--删除表空间
drop tablespace itheima;

--创建用户
create user itheima
identified by itheima   //密码
default tablespace itheima;

--给用户授权
--oracle数据库中常用角色
connect--连接角色，基本角色
resource--开发者角色
dba--超级管理员角色
--给itheima用户授予dba角色
grant dba to itheima;

---创建一个person表
create table person(
       pid number(20),
       pname varchar2(10)
);

---修改表结构
---添加一列
alter table person add (gender number(1));
---修改列类型
alter table person modify gender char(1);
---修改列名称
alter table person rename column gender to sex;
---删除一列
alter table person drop column sex;

--删除表中全部记录
delete from person;
--删除表结构
drop table person;
--先删除表，再次创建表。效果等同于删除表中全部记录。
--在数据量大的情况下，尤其在表中带有索引的情况下，该操作效率高。
--索引可以提供查询效率，但是会影响增删改效率。
truncate table person;

----序列不真的属于任何一张表，但是可以逻辑和表做绑定。
----序列：默认从1开始，依次递增，主要用来给主键赋值使用。
----dual：虚表，只是为了补全语法，没有任何意义。
create sequence s_person;
select s_person.nextval from dual;

insert into person (pid, pname) values (s_person.nextval, '小明');

----scott用户，密码tiger。
--解锁scott用户
alter user scott account unlock;
--解锁scott用户的密码【此句也可以用来重置密码】
alter user scott identified by tiger;

单行函数：作用于一行，返回一个值。
upper('yes')//变大写
lower('YES')//变小写
round(56.16, -2)//四舍五入 ， 保留几位
trunc(56.16, -1)//截取   ，  截到那位
mod(10, 3)//取余
nvl()//空值转换

---日期转字符串
select to_char(sysdate, 'fm yyyy-mm-dd hh24:mi:ss') from dual;
---字符串转日期
select to_date('2018-6-7 16:39:50', 'fm yyyy-mm-dd hh24:mi:ss') from dual;

---oracle中专用外连接
select *
from emp e, dept d
where e.deptno(+) = d.deptno;

排序操作会影响rownum的顺序排序操作会影响rownum的顺序
rownum行号不能写上大于一个正数。
select * from(
    select rownum rn, tt.* from(
          select * from emp order by sal desc
    ) tt where rownum<11
) where rn>5

ORACLE 的 NULL 只能用 IS 或 IS NOT 进行比较，而不能用 = 、!= 、<> 进行比较

```

### 条件表达式
```sql
---条件表达式的通用写法，mysql和oracle通用
---给emp表中员工起中文名
select e.ename, 
       case e.ename
         when 'SMITH' then '曹贼'
           when 'ALLEN' then '大耳贼'
             when 'WARD' then '诸葛小儿'
               --else '无名'
                 end
from emp e;
---判断emp表中员工工资，如果高于3000显示高收入，如果高于1500低于3000显示中等收入，
-----其余显示低收入
select e.sal, 
       case 
         when e.sal>3000 then '高收入'
           when e.sal>1500 then '中等收入'
               else '低收入'
                 end
from emp e;
----oracle中除了起别名，都用单引号。
----oracle专用条件表达式
select e.ename, 
        decode(e.ename,
          'SMITH',  '曹贼',
            'ALLEN',  '大耳贼',
              'WARD',  '诸葛小儿',
                '无名') "中文名"             
from emp e;
```

### 视图
```sql
---创建视图【必须有dba权限】
create view v_emp as select ename, job from emp;
---查询视图
select * from v_emp;
---修改视图[不推荐]
update v_emp set job='CLERK' where ename='ALLEN';
---创建只读视图
create view v_emp1 as select ename, job from emp with read only;
```
视图的作用
	视图可以屏蔽掉一些敏感字段。
	保证总部和分部数据及时统一。
### 索引
```sql
查看索引在建在那表、列：
   select * from user_indexes;
   select * from user_ind_columns;

---单列索引
---创建单列索引
create index idx_ename on emp(ename);
---单列索引触发规则，条件必须是索引列中的原始值。
---单行函数，模糊查询，都会影响索引的触发。
select * from emp where ename='SCOTT'

---复合索引
---创建复合索引
create index idx_enamejob on emp(ename, job);
---复合索引中第一列为优先检索列
---如果要触发复合索引，必须包含有优先检索列中的原始值。
select * from emp where ename='SCOTT' and job='xx';---触发复合索引
select * from emp where ename='SCOTT' or job='xx';---不触发索引
select * from emp where ename='SCOTT';---触发单列索引


```
索引碎片问题：由于对基表做DML操作，导致索引表块的自动更改操作，尤其是基表的delete操作会引起index表的index_entries的逻辑删除，注意只有当一个索引块中的全部index_entry都被删除了，才会把这个索引块删除，索引对基表的delete、insert操作都会产生索引碎片问题。
 
在Oracle文档里并没有清晰的给出索引碎片的量化标准,Oracle建议通过Segment Advisor(段顾问）解决表和索引的碎片问题，如果你想自行解决，可以通过查看index_stats视图，当以下三种情形之一发生时，说明积累的碎片应该整理了（仅供参考）。
1.HEIGHT >=4   
2 PCT_USED< 50%   
3 DEL_LF_ROWS/LF_ROWS>0.2
```sql
分析索引：
analyze index ind_1 validate structure;
select name,HEIGHT,PCT_USED,DEL_LF_ROWS/LF_ROWS from index_stats;
整理索引
alter index ind_1 rebuild [online] [tablespace name];
```






### 锁和事务
原子性,一致性,隔离性,持久性(oracle主要是靠 'rudo' 日志)

共享锁与排他锁
只有有事物才会产生锁，保证数据的完整性和正确性

锁类型：
* DML锁（data locks，数据锁），用于保护数据的完整性。  TX(行级锁),TM(表级锁)，我们日常所使用的DML操作就会产生事物和锁。
* DDL锁（dictionary locks，数据字典锁），用于保护数据库对象的结构，如表、索引等的结构定义。
* SYSTEM锁（internal locks and latches），保护数据库的内部结构

```sql
登录的用户需要使用sysdba形式：conn system/tiger@orcl as sysdba;
查看事务：select * from v$transaction;

查看锁：select * from v$lock;



加锁
select * from emp1 where deptno = 10 for update;
select * from emp1 where empno = 7782 for update nowait;  不等待
select * from emp1 where empno = 7782 for update wait 5;  等几秒
select * from emp1 where job= 'CLERK' for update skip locked;  跳过被锁记录

如果这个锁占用的时间太长，我们可以通过管理员杀掉session用户。
首先要找到是哪个sid占用了太长时间，查看v$lock表
然后根据v$lock表的SID，去v$session里面去找到，进行kill操作。
 select sid, serial# from v$session where sid = 170;
 alter system kill session 'sid,serial';

```

TX :一种锁
TM :五种: RS.RX、s.SRX、X
	mode : 23456
ROW SHARE
行共享(RS)，允许其他用户同时更新其他行，允许其他用户同时加共享锁，不允许有独占(排他性质)的锁
ROW EXCLUSIVE 行排他(RX)， 允许其他用户同时更新其他行，只允许其他用户同时加行共享锁或者行排他锁
SHARE共享(S)，不允许其他用户同时更新任何行，只允许其他用户同时加共享锁或者行共享锁
SHARE ROW EXCLUSIV(SRX)共享行排他， 不允许其他用户同时更新其他行，只允许其他用户同时加行共享锁
EXCLUSIVE (&)排他，其他用户禁止更新任何行，禁止其他用户同时加任何排他锁。

```sql
sql语句                      加锁模式                           许可其他用户的加锁模式
select * from table_ name    无                               RS,RX,S,SRX,X
insert, update, delete (DMI操作)  RX                               RS,RX
select * from table_name for update  RX                              RS,RX
lock table table_name in rou share mode  RS                        RS,RX,S,SRX
lock table table_name in rou exclusive mode RX                       RS,RX
lock table table_name in share mode          S                      RS,S
lock table table_name in share rov exclusive mode  SRX                   RS
lock table table_name in exclusive mode           X                    无
```







## pl/sql
Procedure Language/SQL
是对sql语言的扩展，使得sql语言具有过程化编程的特性。
比一般的过程化编程语言，更加灵活高效。
主要用来编写存储过程和存储函数等
```sql
declare
    说明部分 （变量说明，游标申明，例外说明 〕
begin
    语句序列 （DML 语句〕…
exception
    例外处理语句
End;
--常量定义：married constant boolean:=true

---声明方法
---赋值操作可以使用:=也可以使用into查询语句赋值
declare
    i number(2) := 10;
    s varchar2(10) := '小明';
    ena emp.ename%type;---引用型变量
    emprow emp%rowtype;---记录型变量
begin
    dbms_output.put_line(i);
    dbms_output.put_line(s);
    select ename into ena from emp where empno = 7788;
    dbms_output.put_line(ena);
    select * into emprow from emp where empno = 7788;
    dbms_output.put_line(emprow.ename || '的工作为：' || emprow.job);
end;

```

### 条件
```sql
语法 1：
    IF 条件 THEN 语句 1;
        语句 2;
        END IF;
语法 2：
    IF 条件 THEN 语句序列 1；
        ELSE 语句序列 2；
        END IF；
语法 3：
    IF 条件 THEN 语句;
    ELSIF 语句 THEN 语句;
    ELSE 语句;
    END IF;

---pl/sql中的if判断
---输入小于18的数字，输出未成年
---输入大于18小于40的数字，输出中年人
---输入大于40的数字，输出老年人
declare
  i number(3) := &ii; //输入 & + 任意变量名
begin
  if i<18 then
    dbms_output.put_line('未成年');
  elsif i<40 then
    dbms_output.put_line('中年人');
  else
    dbms_output.put_line('老年人');
  end if;
end;
```
### 循环
```sql
---pl/sql中的loop循环
---用三种方式输出1到10是个数字
---while循环
declare
  i number(2) := 1;
begin
  while i<11 loop
     dbms_output.put_line(i);
     i := i+1;
  end loop;  
end;
---exit循环
declare
  i number(2) := 1;
begin
  loop
    exit when i>10;
    dbms_output.put_line(i);
    i := i+1;
  end loop;
end;
---for循环
declare
begin
  for i in 1..10 loop
     dbms_output.put_line(i);  
  end loop;
end;
```

### 游标
```sql
---游标：可以存放多个对象，多行记录。
---输出emp表中所有员工的姓名
declare
  cursor c1 is select * from emp;
  emprow emp%rowtype;
begin
  open c1;
     loop
         fetch c1 into emprow;
         exit when c1%notfound;
         dbms_output.put_line(emprow.ename);
     end loop;
  close c1; 
end;

-----给指定部门员工涨工资
declare
  cursor c2(eno emp.deptno%type) 
  is select empno from emp where deptno = eno;
  en emp.empno%type;
begin
  open c2(10);
     loop
        fetch c2 into en;
        exit when c2%notfound;
        update emp set sal=sal+100 where empno=en;
        commit;
     end loop;  
  close c2;
end;
```
### 存储过程

存储过程：存储过程就是提前已经编译好的一段pl/sql语言，放置在数据库端可以直接被调用。这一段pl/sql一般都是固定步骤的业务。

```sql
create [or replace] PROCEDURE 过程名[(参数名 in/out 数据类型)]
AS
begin
    PLSQL 子程序体；
End;
或者
create [or replace] PROCEDURE 过程名[(参数名 in/out 数据类型)]
is
begin
    PLSQL 子程序体；
End 过程名; 

----给指定员工涨100块钱
create or replace procedure p1(eno emp.empno%type)
is

begin
   update emp set sal=sal+100 where empno = eno;
   commit;
end;


---out类型参数如何使用
---使用存储过程来算年薪
create or replace procedure p_yearsal(eno emp.empno%type, yearsal out number)
is
   s number(10);
   c emp.comm%type;
begin
   select sal*12, nvl(comm, 0) into s, c from emp where empno = eno;
   yearsal := s+c;
end;

---测试p_yearsal
declare
  yearsal number(10);
begin
  p_yearsal(7788, yearsal);
  dbms_output.put_line(yearsal);
end;


```

### 存储函数
```sql
create or replace function 函数名(Name in type, Name in type, ...) return 数据类型 is
    结果变量 数据类型;
begin
    return(结果变量);
end 函数名;

----通过存储函数实现计算指定员工的年薪
----存储过程和存储函数的参数都不能带长度
----存储函数的返回值类型不能带长度
create or replace function f_yearsal(eno emp.empno%type) return number
is
  s number(10);     
begin
  select sal*12+nvl(comm, 0) into s from emp where empno = eno;
  return s;
end;

----测试f_yearsal
----存储函数在调用的时候，返回值需要接收。
declare
  s number(10); 
begin
  s := f_yearsal(7788);
  dbms_output.put_line(s);
end;

in和out类型参数的区别
凡是涉及到into查询语句赋值或者:=赋值操作的参数，都必须使用out来修饰。

存储过程和存储函数的区别
	语法区别：关键字不一样，
		存储函数比存储过程多了两个return。
	本质区别：存储函数有返回值，而存储过程没有返回值。
		如果存储过程想实现有返回值的业务，我们就必须使用out类型的参数。
		即便是存储过程使用了out类型的参数，起本质也不是真的有了返回值，
		而是在存储过程内部给out类型参数赋值，在执行完毕后，我们直接拿到输出类型参数的值。

----使用存储函数有返回值的特性，来自定义函数
----使用存储函数来实现提供一个部门编号，输出一个部门名称。
create or replace function fdna(dno dept.deptno%type) return dept.dname%type
is
  dna dept.dname%type;
begin
  select dname into dna from dept where deptno = dno;
  return dna;
end;
---使用fdna存储函数来实现案例需求：查询出员工姓名，员工所在部门名称。
select e.ename, fdna(e.deptno)
from emp e;


```

### 触发器

就是制定一个规则，==增删改==操作时，只要满足该规则，自动触发，无需调用。
语句级触发器：不包含有for each row的触发器。
行级触发器：包含有for each row的就是行级触发器。
加for each row是为了使用:old或者:new对象或者一行记录。
```sql
CREATE [or REPLACE] TRIGGER 触发器名
    {BEFORE | AFTER}
    {DELETE | INSERT | UPDATE [OF 列名]}
    ON 表名
    [FOR EACH ROW [WHEN(条件) ] ]
begin
    PLSQL 块
End 触发器名


---语句级触发器
----插入一条记录，输出一个新员工入职
create or replace trigger t1
after
insert
on person
declare

begin
  dbms_output.put_line('一个新员工入职');
end;
---触发t1
insert into person values (1, '小红');
commit;


---语句级触发器
----插入一条记录，输出一个新员工入职
create or replace trigger t1
after
insert
on person
declare

begin
  dbms_output.put_line('一个新员工入职');
end;
---触发t1
insert into person values (1, '小红');
commit;


----触发器实现主键自增。【行级触发器】
---分析：在用户做插入操作的之前，拿到即将插入的数据，
------给该数据中的主键列赋值。
create or replace trigger auid
before
insert
on person
for each row
declare

begin
  select s_person.nextval into :new.pid from dual;
end;

```
### javaDemo
```java
public class OracleDemo {

    @Test
    public void javaCallOracle() throws Exception {
        //加载数据库驱动
        Class.forName("oracle.jdbc.driver.OracleDriver");
        //得到Connection连接
        Connection connection = DriverManager.getConnection("jdbc:oracle:thin:@192.168.88.6:1521:orcl", "itheima", "itheima");
        //得到预编译的Statement对象
        PreparedStatement pstm = connection.prepareStatement("select * from emp where empno = ?");
        //给参数赋值
        pstm.setObject(1, 7788);
        //执行数据库查询操作
        ResultSet rs = pstm.executeQuery();
        //输出结果
        while(rs.next()){
            System.out.println(rs.getString("ename"));
        }
        //释放资源
        rs.close();
        pstm.close();
        connection.close();
    }

    /**
     * java调用存储过程
     * {?= call <procedure-name>[(<arg1>,<arg2>, ...)]}   调用存储函数使用
     *  {call <procedure-name>[(<arg1>,<arg2>, ...)]}   调用存储过程使用
     * @throws Exception
     */
    @Test
    public void javaCallProcedure() throws Exception {
        //加载数据库驱动
        Class.forName("oracle.jdbc.driver.OracleDriver");
        //得到Connection连接
        Connection connection = DriverManager.getConnection("jdbc:oracle:thin:@192.168.88.6:1521:orcl", "itheima", "itheima");
        //得到预编译的Statement对象
        CallableStatement pstm = connection.prepareCall("{call p_yearsal(?, ?)}");
        //给参数赋值
        pstm.setObject(1, 7788);
        pstm.registerOutParameter(2, OracleTypes.NUMBER);
        //执行数据库查询操作
        pstm.execute();
        //输出结果[第二个参数]
        System.out.println(pstm.getObject(2));
        //释放资源
        pstm.close();
        connection.close();
    }


    /**
     * java调用存储函数
     * {?= call <procedure-name>[(<arg1>,<arg2>, ...)]}   调用存储函数使用
     *  {call <procedure-name>[(<arg1>,<arg2>, ...)]}   调用存储过程使用
     * @throws Exception
     */
    @Test
    public void javaCallFunction() throws Exception {
        //加载数据库驱动
        Class.forName("oracle.jdbc.driver.OracleDriver");
        //得到Connection连接
        Connection connection = DriverManager.getConnection("jdbc:oracle:thin:@192.168.88.6:1521:orcl", "itheima", "itheima");
        //得到预编译的Statement对象
        CallableStatement pstm = connection.prepareCall("{?= call f_yearsal(?)}");
        //给参数赋值
        pstm.setObject(2, 7788);
        pstm.registerOutParameter(1, OracleTypes.NUMBER);
        //执行数据库查询操作
        pstm.execute();
        //输出结果[第一个参数]
        System.out.println(pstm.getObject(1));
        //释放资源
        pstm.close();
        connection.close();
    }
}
```



### 批量插入

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

