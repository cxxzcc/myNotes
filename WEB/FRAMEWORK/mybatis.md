# mybatis

## 批量插入

### 方式一

http://www.mybatis.org/mybatis-dynamic-sql/docs/insert.html   **Batch Insert Support**  推荐性能好

```java
SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH);
try {
    SimpleTableMapper mapper = session.getMapper(SimpleTableMapper.class);
    List<SimpleTableRecord> records = getRecordsToInsert(); // not shown
 
    BatchInsert<SimpleTableRecord> batchInsert = insert(records)
            .into(simpleTable)
            .map(id).toProperty("id")
            .map(firstName).toProperty("firstName")
            .map(lastName).toProperty("lastName")
            .map(birthDate).toProperty("birthDate")
            .map(employed).toProperty("employed")
            .map(occupation).toProperty("occupation")
            .build()
            .render(RenderingStrategy.MYBATIS3);
 
    batchInsert.insertStatements().stream().forEach(mapper::insert);
 
    session.commit();
} finally {
    session.close();
}
```

### foreach性能一般 20-50



在MySql Docs中也提到过这个trick，如果要优化插入速度时，可以将许多小型操作组合到一个大型操作中。理想情况下，这样可以在单个连接中一次性发送许多新行的数据，并将所有索引更新和一致性检查延迟到最后才进行。

```xml
<insert id="batchInsert" parameterType="java.util.List">
    insert into USER (id, name) values
    <foreach collection="list" item="model" index="index" separator=","> 
        (#{model.id}, #{model.name})
    </foreach>
</insert>
```

## sql注入

- sql注入发生的时间，sql注入发生的阶段在sql==预编译阶段==，当编译完成的sql不会产生sql注入

### 采用jdbc操作数据时候

```java
String sql = "update ft_proposal set id = "+id;
PreparedStatement prepareStatement = conn.prepareStatement(sql);
prepareStatement.executeUpdate();

//preparedStatement 预编译对象会对传入sql进行预编译，
//那么当传入id 字符串为 “update ft_proposal set id = 3;drop table ft_proposal;” 这种情况下就会导致sql注入删除ft_proposal这张表。

//如何防止sql注入，首先将要执行sql进行预编译，然后在将占位符进行替换
String sql = "update ft_proposal set id = ?"
PreparedStatement ps = conn.preparedStatement(sql);
ps.setString(1,"2");
ps.executeUpdate();

//处理使用预编译语句之外，另一种实现方式可以采用存储过程，存储过程其实也是预编译的，存储过程是sql语句的集合，将所有预编译的sql 语句编译完成后，存储在数据库上，但是由于存储过程比较死板一般不采用这种方式进行处理
```

### **mybatis是如何处理sql注入的**

* #{} 先预编译后替换

  **mybatis底层实现了预编译**，**底层通过prepareStatement预编译实现类对当前传入的sql进行了预编译，这样就可以防止sql注入了**。

* ${} 先替换后预编译



**#{}和${}**

#{}：相当于预处理中的占位符？。

​	\#{}里面的参数表示接收java输入参数的名称。

​	\#{}可以接受HashMap、简单类型、POJO类型的参数。

​	当接受简单类型的参数时，#{}里面可以是value，也可以是其他。

​	\#{}可以防止SQL注入。

${}：相当于拼接SQL串，对传入的值不做任何解释的原样输出。

​	${}会引起SQL注入，所以要谨慎使用。

​	${}可以接受HashMap、简单类型、POJO类型的参数。

​	当接受简单类型的参数时，${}里面只能是value。







# mybatisPlus

官方地址: http://mp.baomidou.com 

代码发布地址: 

Github: https://github.com/baomidou/mybatis-plus 

Gitee: https://gitee.com/baomidou/mybatis-plus 

文档发布地址: http://mp.baomidou.com/#/?id=%E7%AE%80%E4%BB%8B



```xml
@TableName
全局的 MP 配置: <property name="tablePrefix" value="tbl_"></property>
@TableField
全局的 MP 配置: <property name="dbColumnUnderline" value="true"></property>
@TableId
全局的 MP 配置: <property name="idType" value="0"></property>
支持主键自增的数据库插入数据获取主键值
```

 MP 启动注入 SQL 原理分析

* employeeMapper 的本质 org.apache.ibatis.binding.MapperProxy 

* MapperProxy 中 sqlSession –>SqlSessionFactory

* SqlSessionFacotry 中 → Configuration→ MappedStatements 

  每一个 mappedStatement 都表示 Mapper 接口中的一个方法与 Mapper 映射文件 中的一个 SQL。 MP 在启动就会挨个分析 xxxMapper 中的方法，并且将对应的 SQL 语句处理好，保 存到 configuration 对象中的 mappedStatements 中.

* Configuration： MyBatis 或者 MP 全局配置对象 

* MappedStatement：一个 MappedStatement 对象对应 Mapper 配置文件中的一个 select/update/insert/delete 节点，主要描述的是一条 SQL 语句 

* SqlMethod : 枚举对象 ，MP 支持的 SQL 方法 

* TableInfo：数据库表反射信息 ，可以获取到数据库表相关的信息 

* SqlSource: SQL 语句处理对象

* MapperBuilderAssistant： 用于缓存、SQL 参数、查询方剂结果集处理等. 通过 MapperBuilderAssistant 将每一个 mappedStatement  添加到 configuration 中的 mappedstatements 中

## ActiveRecord(活动记录)

Active Record(活动记录)，是一种领域模型模式，特点是一个模型类对应关系型数据库中的 一个表，而模型类的一个实例对应表中的一行记录。



使用 AR 模式 

实体类继承 Model 类且实现主键指定方法

```java
@TableName("tbl_employee")
public class Employee extends Model<Employee>{
    // .. fields
    // .. getter and setter
    @Override
    protected Serializable pkVal() {
        return this.id;
    }
```

## 代码生成器

模板引擎 MP 的代码生成器默认使用的是 Apache 的 Velocity 模板，当然也可以更换为别的模板 技术，例如 freemarker。此处不做过多的介绍。 需要加入 Apache Velocity 的依赖

```xml
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.0</version>
</dependency>
```

加入 slf4j ,查看日志输出信息

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.7</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.7</version>
</dependency>
```

```java
@Test
public void testGenerator() {
    //全局配置
    GlobalConfig config = new GlobalConfig();
    config.setActiveRecord(true) //是否支持AR模式
        .setAuthor("weiyunhui") //作者
        .setOutputDir("D:\\workspace_my\\mp03\\src\\main\\java") 
        //生成路径
        .setFileOverride(true)//文件覆盖
        .setServiceName("%sService") //设置生成的service接口名首字母是否为I
        .setIdType(IdType.AUTO) //主键策略
        ;
    //数据源配置
    DataSourceConfig dsConfig = new DataSourceConfig();
    dsConfig.setDbType(DbType.MYSQL)
        .setUrl("jdbc:mysql://localhost:3306/javaEE_0228")
        .setDriverName("com.mysql.jdbc.Driver")
        .setUsername("root")
        .setPassword("1234");
    //策略配置
    StrategyConfig stConfig = new StrategyConfig();
    stConfig.setCapitalMode(true) // 全局大写命名
        .setDbColumnUnderline(true) //表名 字段名 是否使用下滑线命名
        .setNaming(NamingStrategy.underline_to_camel) // 数据库表映射到实体的命名策略
        .setInclude("tbl_employee") //生成的表
        .setTablePrefix("tbl_"); // 表前缀
    //包名策略
    PackageConfig pkConfig = new PackageConfig();
    pkConfig.setParent("com.atguigu.mp")
        .setController("controller")
        .setEntity("beans")
        .setService("service");
    AutoGenerator ag = new
        AutoGenerator().setGlobalConfig(config)
        .setDataSource(dsConfig)
        .setStrategy(stConfig)
        .setPackageInfo(pkConfig);
    ag.execute();
}
```

## 插件

Mybatis 插件机制简介 

1. 插件机制: Mybatis 通过插件(Interceptor) 可以做到拦截四大对象相关方法的执行,根据需求，完成相关数据的动态改变。 Executor StatementHandler ParameterHandler ResultSetHandler 
2. 插件原理 四大对象的每个对象在创建时，都会执行 interceptorChain.pluginAll()，会经过每个插 件对象的 plugin()方法，目的是为当前的四大对象创建代理。代理对象就可以拦截到四 大对象相关方法的执行，因为要执行四大对象的方法需要经过代理.

分页插件  

com.baomidou.mybatisplus.plugins.PaginationInterceptor



执行分析插件 

1. com.baomidou.mybatisplus.plugins.SqlExplainInterceptor 
2. SQL 执行分析拦截器，只支持 MySQL5.6.3 以上版本
3. 该插件的作用是分析 DELETE UPDATE 语句,防止小白 或者恶意进行 DELETE UPDATE 全表操作 
4. 只建议在开发环境中使用，不建议在生产环境使用 
5. 在插件的底层 通过 SQL 语句分析命令:Explain 分析当前的 SQL 语句， 根据结果集中的 Extra 列来断定当前是否全表操作。

性能分析插件 

1. com.baomidou.mybatisplus.plugins.PerformanceInterceptor 
2. 性能分析拦截器，用于输出每条 SQL 语句及其执行时间
3. SQL 性能执行分析,开发环境使用，超过指定时间，停止运行。有助于发现问题

乐观锁插件 

1. com.baomidou.mybatisplus.plugins.OptimisticLockerInterceptor 
2. 如果想实现如下需求: 当要更新一条记录的时候，希望这条记录没有被别人更新 
3. 乐观锁的实现原理: 取出记录时，获取当前 version 2  更新时，带上这个 version 2  执行更新时， set version = yourVersion+1 where version = yourVersion 如果 version 不对，就更新失败 
4. @Version 用于注解实体字段，必须要有。



## 自定义全局操作

根据 MybatisPlus 的 AutoSqlInjector 可以自定义各种你想要的 sql ,注入到全局中，相当于自 定义 Mybatisplus 自动注入的方法。 之前需要在 xml 中进行配置的 SQL 语句，现在通过扩展 AutoSqlInjector 在加载 mybatis 环境 时就注入

AutoSqlInjector  

1. 在 Mapper 接口中定义相关的 CRUD 方法 
2. 扩展 AutoSqlInjector inject 方法，实现 Mapper 接口中方法要注入的 SQL 
3. 在 MP 全局策略中，配置 自定义注入器

自定义注入器的应用之 逻辑删除

1. com.baomidou.mybatisplus.mapper.LogicSqlInjector 
2. logicDeleteValue 逻辑删除全局值
3.  logicNotDeleteValue 逻辑未删除全局值 
4. 在 POJO 的逻辑删除字段 添加 @TableLogic 注解 
5. 会在 mp 自带查询和更新方法的 sql 后面，追加『逻辑删除字段』=『LogicNotDeleteValue 默认值』 删除方法: deleteById()和其他 delete 方法, 底层 SQL 调用的是 update tbl_xxx  set 『逻辑删除字段』=『logicDeleteValue 默认值』

## 公共字段自动填充

元数据处理器接口 

com.baomidou.mybatisplus.mapper.MetaObjectHandler 

insertFill(MetaObject metaObject)  

updateFill(MetaObject metaObject) 

metaobject: 元对象. 是 Mybatis 提供的一个用于更加方便，更加优雅的访问对象的属性, 给对象的属性设置值 的一个对象. 还会用于包装对象. 支持对 Object 、Map、Collection 等对象进行包装 

本质上 metaObject 获取对象的属性值或者是给对象的属性设置值，最终是要 通过 Reflector 获取到属性的对应方法的 Invoker, 最终 invoke.



步骤 

1. 注解填充字段 @TableFile(fill = FieldFill.INSERT) 查看 FieldFill 

2. 自定义公共字段填充处理器 

   ```java
   /**
    * 自定义公共字段填充处理器
    */
   public class MyMetaObjectHandler extends MetaObjectHandler {
       /**
   	 * 插入操作 自动填充
   	 */
       @Override
       public void insertFill(MetaObject metaObject) {
           //获取到需要被填充的字段的值
           Object fieldValue = getFieldValByName("name", metaObject);
           if(fieldValue == null) {
               System.out.println("*******插入操作 满足填充条件*********");
               setFieldValByName("name", "weiyunhui", metaObject);
           }
       }
       /**
   	 * 修改操作 自动填充
   	 */
       @Override
       public void updateFill(MetaObject metaObject) {
           Object fieldValue = getFieldValByName("name", metaObject);
           if(fieldValue == null) {
               System.out.println("*******修改操作 满足填充条件*********");
               setFieldValByName("name", "weiyh", metaObject);
           }
       }
   }
   ```

   

3. MP 全局注入 自定义公共字段填充处理器



## Oracle

### 主键 Sequence

1. 实体类配置主键 Sequence @KeySequence(value=”序列名”，clazz=xxx.class 主键属性类 型) 
2. 全局 MP 主键生成策略为 IdType.INPUT  
3. 全局 MP 中配置 Oracle 主键 Sequence com.baomidou.mybatisplus.incrementer.OracleKeyGenerator
4. 可以将@keySequence 定义在父类中，可实现多个子类对应的多个表公用一个 Sequence



