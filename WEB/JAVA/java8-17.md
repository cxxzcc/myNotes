[https://openjdk.java.net/projects/jdk9/](https://openjdk.java.net/projects/jdk9/)
[https://openjdk.java.net/projects/jdk/10/](https://openjdk.java.net/projects/jdk/10/)
[https://openjdk.java.net/projects/jdk/11/](https://openjdk.java.net/projects/jdk/11/)
[https://openjdk.java.net/projects/jdk/12/](https://openjdk.java.net/projects/jdk/12/)
[https://openjdk.java.net/projects/jdk/13/](https://openjdk.java.net/projects/jdk/13/)
[https://openjdk.java.net/projects/jdk/14/](https://openjdk.java.net/projects/jdk/14/)
[https://openjdk.java.net/projects/jdk/15/](https://openjdk.java.net/projects/jdk/15/)
[https://openjdk.java.net/projects/jdk/16/](https://openjdk.java.net/projects/jdk/16/)
[https://openjdk.java.net/projects/jdk/17/](https://openjdk.java.net/projects/jdk/17/)

新特性关注点

- 角度一：语法层面
   lambda表达式，switch，自动装箱和拆箱、enum、接口中的静态方法、默认方法、私有方法
- 角度二：API层面
   Stream API、新的日期时间的API、Optional、String、集合框架
- 角度三：底层优化  Java   C/C++
  JVM的优化、元空间、GC、GC的组合、GC的参数、js的执行引擎、集合底层的实现

# java8(LTS)

## stream
新建流
```java
1.集合
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);  
Stream<Integer> stream = integerList.stream();

2.数组
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);  
Stream<Integer> stream = integerList.stream();

3.数值
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);  
Stream<Integer> stream = integerList.stream();

4.文件
Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())

5.函数生成 提供了iterate和generate两个静态方法从函数中生成流

Stream<Integer> stream = Stream.iterate(0, n -> n + 2).limit(5);
接受两个参数，初始化值，函数操作，无限流，通过`limit`方法对流进行了截断，只生成5个偶数

Stream<Double> stream = Stream.generate(Math::random).limit(5);
一个参数，方法参数类型为`Supplier<T>`，无限流，因此通过`limit`对流进行了截断

```

使用数值流可以避免计算过程中拆箱装箱，提高性能。`Stream API`提供了`mapToInt`、`mapToDouble`、`mapToLong`三种方式将对象流【即`Stream<T>`】转换成对应的数值流，同时提供了`boxed`方法将数值流转换为对象流

操作
```java
1.中间操作
filter
distinct
limit
skip
map 将接受的元素映射成另外一个元素
flatMap 将一个流中的每个值都转换为另一个流
	Arrays.asList("Hello", "World").stream().map(x -> x.split("")).flatMap(Arrays::stream)

allMatch 匹配所有
allMatch 匹配所有
noneMatch 全部不匹配

2.终端操作
count 统计出流中元素个数 collect(counting()) 
findFirst 查找第一个
findAny 随机查找一个
reduce  将流中的元素组合起来
	int sum = integerList.stream().reduce(0, (a, b) -> (a + b));
	int sum = integerList.stream().reduce(0, Integer::sum);
	两个参数，初始值，`BinaryOperator<T> accumulator` `reduce`还有一个没有初始化值的重载方法
min/max
	OptionalInt min = menu.stream().mapToInt(Dish::getCalories).min();  
	OptionalInt max = menu.stream().mapToInt(Dish::getCalories).max();
	Optional<Integer> min = 
	menu.stream().map(Dish::getCalories).min(Integer::compareTo);
	Optional<Integer> max = 
	menu.stream().map(Dish::getCalories).max(Integer::compareTo);
minBy/maxBy
	Optional<Integer> min = 
	menu.stream().map(Dish::getCalories).collect(minBy(Integer::compareTo));
	Optional<Integer> max = 
	menu.stream().map(Dish::getCalories).collect(maxBy(Integer::compareTo));
	Optional<Integer> min = menu.stream().map(Dish::getCalories).reduce(Integer::min);  
	Optional<Integer> max = menu.stream().map(Dish::getCalories).reduce(Integer::max);
sum
	int sum = menu.stream().collect(summingInt(Dish::getCalories));
	int sum = menu.stream().map(Dish::getCalories).reduce(0, Integer::sum);
	int sum = menu.stream().mapToInt(Dish::getCalories).sum();
avg
	double average = menu.stream().collect(averagingInt(Dish::getCalories));
summarizingInt同时求总和、平均值、最大值、最小值
	IntSummaryStatistics intSummaryStatistics = 
	menu.stream().collect(summarizingInt(Dish::getCalories));
	double average = intSummaryStatistics.getAverage();  //获取平均值
	int min = intSummaryStatistics.getMin();  //获取最小值
	int max = intSummaryStatistics.getMax();  //获取最大值
	long sum = intSummaryStatistics.getSum();  //获取总和

joining 拼接流中的元素
	String result=menu.stream().map(Dish::getName).collect(Collectors.joining(", "));

groupingBy 分组
	Map<Type, List<Dish>> result=dishList.stream().collect(groupingBy(Dish::getType));
嵌套使用`groupingBy`进行多级分类
	Map<Type, List<Dish>> result = menu.stream().collect(groupingBy(Dish::getType,  
        groupingBy(dish -> {  
            if (dish.getCalories() <= 400) return CaloricLevel.DIET;  
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;  
                else return CaloricLevel.FAT;  
        })));
 
partitioningBy 进行分区  分为两组
Map<Boolean, List<Dish>> result = menu.stream().collect(partitioningBy(Dish :: isVegetarian))
Map<Boolean, List<Dish>> result = menu.stream().collect(groupingBy(Dish :: isVegetarian))
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);  
Map<Boolean, List<Integer>> result = integerList.stream().collect(partitioningBy(i -> i < 3));



```






# java9




* 模块化系统
*  jShell 命令
*  多版本兼容 jar 包 
*  接口的私有方法 
* 钻石操作符的使用升级 
* 语法改进：try 语句 
* 下划线使用限制 
* String 存储结构变更 
* 便利的集合特性：of() 
* 增强的 Stream API 
* 多分辨率图像 API 
* 全新的 HTTP 客户端 API 
* Deprecated 的相关 API 
* 智能 Java 编译工具 
* 统一的 JVM 日志系统 
* javadoc 的 HTML 5 支持 
* Javascript 引擎升级：Nashorn 
* java 的动态编译器



文档

https://docs.oracle.com/javase/9/

http://www.oracle.com/technetwork/java/javase/documentation/jdk9-doc-downloads-3850606.html

## 模块化系统

模块(module)的概念，其实就是package外再裹一层， 也就是说，用模块来管理各个package，通过声明某个package暴露， 不声明默认就是隐藏。因此，模块化使得代码组织上更安全，因 为它可以指定哪些部分可以暴露，哪些部分隐藏



模块独立、化繁为简 

模块化（以 Java 平台模块系统的形式）将 JDK 分成一组模块，可 以在编译时，运行时或者构建时进行组合



实现目标 

* 主要目的在于减少内存的开销
* 只须必要模块，而非全部jdk模块，可简化各种类库和大型应用的 开发和维护
* 改进 Java SE 平台，使其可以适应不同大小的计算设备 
* 改进其安全性，可维护性，提高性能

### demo

模块将由通常的类和新的模块声明文件（module-info.java）组成。 该文件是位于 java 代码结构的顶层，该模块描述符明确地定义了我们 的模块需要什么依赖关系，以及哪些模块被外部使用。在 exports 子 句中未提及的所有包默认情况下将封装在模块中，不能在外部使用。

module-info.java

```java
module java9demo {
 requires java9test;
 requires java.logging;
 requires junit;
}
//requires：指明对其它模块的依赖。
```

```java
module java9test {
//package we export
 exports com.atguigu.bean;
}//exports：控制着哪些包可以被其它模块访问到。所有不被导出的包默认都被封装在模块里面
```

## REPL 工具： jShell

交互式编程环境 REPL (read - evaluate - print - loop)



是 tab 键进行自动补全和自动添加分号

没有受检异常（编译时异常）

```cmd
/!
重新运行上一个片段 -- 请参阅 /help rerun
/-<n>
重新运行以前的第 n 个片段 -- 请参阅 /help rerun
/<id>
按 ID 或 ID 范围重新运行片段 -- 参见 /help rerun
/?
获取有关使用 jshell 工具的信息
/drop
删除源条目
/edit
编辑源条目
/env
查看或更改评估上下文
/exit
退出 jshell 工具
/help
获取有关使用 jshell 工具的信息
/history
您键入的内容的历史记录
/imports
列出导入的项
/list
列出您键入的源
/methods
列出已声明方法及其签名
/open
打开文件作为源输入
/reload
重置和重放相关历史记录 -- 当前历史记录或上一个历史记录 (-restore)
/reset
重置 jshell 工具
/save
将片段源保存到文件
/set
设置配置信息
/types
列出类型声明
/vars
列出已声明变量及其值
<再次按 Tab 可查看完整文档>
jshell> /
```

## 多版本兼容 jar 包

新版本覆盖旧版本同名类,  新用新旧用旧

## 语法

### 接口的私有方法

jdk7 全局常量(public static final)抽象方法public abstract

jdk8 静态方法默认方法

jdk9 私有方法



<font color="red"> 面试题:抽象类和接口的异同? </font>

1. 二者的定义: a. 声明的方式b. 内部的结构(jdk 7 ;jdk 8 ; jdk 9)
2. 共同点:不能实例化;以多态的方式使用
3. 不同点:单继承;多实现

### 钻石操作符(Diamond Operator)使用升级

匿名实现类共同使用钻石操作符（diamond operator）

```java
Set<String> set = new HashSet<>(){};
```

### try 语句

java 8 中，可以实现资源的自动关闭，但是要求执行后必须关闭的所 有资源必须在 try 子句中初始化，否则编译不通过。如下例所示：

```java
try(InputStreamReader reader = new InputStreamReader(System.in)){
}catch (IOException e){
    e.printStackTrace();
}
```

java 9 中，用资源语句编写 try 将更容易，我们可以在 try 子句中使用 已经初始化过的资源，此时的资源是 **final 的**

```java
InputStreamReader reader = new InputStreamReader(System.in);
OutputStreamWriter writer = new OutputStreamWriter(System.out);
try(reader;writer){
    //reader 是 final 的，不可再被赋值
    // reader = null;
}catch (IOException e){
    e.printStackTrace();
}
```

### UnderScore(下划线)使用的限制

在 java 8 中，标识符可以独立使用“_”来命名：

在 java 9 中规定“_”不再可以单独命名标识符了

## api

### String 存储结构变更

String 再也不用 char[] 来存储啦，改成了 byte[] 加上编码标 记，节约了一些空间



String: jdk 8及之前:底层使用char[]存储: jdk 9 :底层使用byte[] (encoding flag)
StringBuffer:jdk 8及之前:底层使用char[]存储; jdk 9 :底层使用byte[]
StringBuilder:jdk 8及之前:底层使用char[]存储; jdk 9 :底层使用byte[]
String:不可变的字符序列;
StringBuffer:可变的字符序列;线程安全的，效率低;
StringBuilder:可变的字符序列;线程不安全的，效率高(jdk 5.0)

### 集合工厂方法：快速创建只读集合

```java
List<String> list = Collections.unmodifiableList(Arrays.asList("a", "b", "c"));
Set<String> set = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("a", "b", "c")));
//如下操作不适用于 jdk 8 及之前版本,适用于 jdk 9
Map<String,Integer> map = Collections.unmodifiableMap(new HashMap<>(){{
 put("a", 1);
 put("b", 2);
 put("c", 3);
}});
map.forEach((k,v) -> System.out.println(k + ":" + v));

//java9
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map1 = Map.of("Tom", 12, "Jerry", 21, "Lilei", 33, "HanMeimei", 18);
```

### Stream API

dropWhile, takeWhile, ofNullable，还有个 iterate 方法的 新重载方法

```java
//takeWhile 返回从开头开始的尽量多的元素
//dropWhile 的行为与 takeWhile 相反，返回剩余的元素。
List<Integer> list = Arrays.asList(45,43,76,87,42,77,90,73,67,88);
list.stream().takeWhile(x -> x < 50).forEach(System.out::println);

List<Integer> list = Arrays.asList(45,43,76,87,42,77,90,73,67,88);
list.stream().dropWhile(x -> x < 50).forEach(System.out::println);

//ofNullable()可以包含一个非空元素，也可以创建一个空 Stream。
Stream<Object> stream1 = Stream.ofNullable(null);
System.out.println(stream1.count());//0

//iterator()
//原来的控制终止方式：
Stream.iterate(1,i -> i + 1).limit(10).forEach(System.out::println);
//现在的终止方式：
Stream.iterate(1,i -> i < 100,i -> i + 1).forEach(System.out::println);

//Optional 转成 stream
optional.stream()
```

### 多分辨率图像 API

* java.awt.image 包下

* 将不同分辨率的图像封装到一张（多分辨率的）图像中，作为它 的变体
* 获取这个图像的所有变体 
* 获取特定分辨率的图像变体-表示一张已知分辨率单位为 DPI 的特 定尺寸大小的逻辑图像，并且这张图像是最佳的变体。
* 基于当前屏幕分辨率大小和运用的图像转换算法， java.awt.Graphics 类可以从接口 MultiResolutionImage 获取所需的 变体。
* MultiResolutionImage 的基础实现是 java.awt.image.BaseMultiResolutionImage。

### 全新的 HTTP 客户端 API

HTTP/1.1和HTTP/2的主要区别是如何在客户端和服务器之间构建 和传输数据。HTTP/1.1 依赖于请求/响应周期。 HTTP/2 允许服务器 “push”数据：它可以发送比客户端请求更多的数据。 这使得它可 以优先处理并发送对于首先加载网页至关重要的数据。

Java 9 提供对 WebSocket 和 HTTP/2 的支持

全新的 HTTP 客户端 API 可以从 jdk.incubator.httpclient 模块中获取。 因为在默认情况下，这个模块是不能根据 classpath 获取的，需要使 用 add modules 命令选项配置这个模块，将这个模块添加到 classpath 中

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder(URI.create("http://www.atguigu.com")).GET().build();
HttpResponse<String> response = client.send(req, 
HttpResponse.BodyHandler.asString());
System.out.println(response.statusCode());
System.out.println(response.version().name());
System.out.println(response.body());
```

### Deprecated 的相关 API

Java 9 废弃或者移除了几个不常用的功能。其中最主要的是 Applet API，现在是标记为废弃的。随着对安全要求的提高，主流浏 览器已经取消对 Java 浏览器插件的支持。HTML5 的出现也进一步加 速了它的消亡。开发者现在可以使用像 Java Web Start 这样的技术来 代替 Applet，它可以实现从浏览器启动应用程序或者安装应用程序。 同时，appletviewer 工具也被标记为废弃

## 其他

### 智能 Java 编译工具

用于在多核处理器情况下提升 JDK 的编译速度

### 统一的 JVM 日志系统

对所有的 JVM 组件引入一个单一的系统， 这些 JVM 组件支持细粒度的和易配置的 JVM 日志。

### javadoc 的 HTML 5 支持

### Javascript 引擎升级：Nashorn

​	Nashorn 项目在 JDK 9 中得到改进，它为 Java 提供轻量级的 Javascript 运行时。Nashorn 项目跟随 Netscape 的 Rhino 项目，目 的是为了在 Java 中实现一个高性能但轻量级的 Javascript 运行时。 Nashorn 项目使得 Java 应用能够嵌入 Javascript。它在 JDK 8 中为 Java 提供一个 Javascript 引擎。 

​	JDK 9 包含一个用来解析 Nashorn 的 ECMAScript 语法树的 API。这个 API 使得 IDE 和服务端框架不需要依赖 Nashorn 项目的 内部实现类，就能够分析 ECMAScript 代码。

### java 的动态编译器

​		JIT（Just-in-time）编译器可以在运行时将热点编译成本地代码， 速度很快。但是 Java 项目现在变得很大很复杂，因此 JIT 编译器需 要花费较长时间才能热身完，而且有些 Java 方法还没法编译，性能 方面也会下降。AoT 编译就是为了解决这些问题而生的。 

​		在 JDK 9 中， AOT（JEP 295: Ahead-of-Time Compilation）作为实 验特性被引入进来，开发者可以利用新的 jaotc 工具将重点代码转换 成类似类库一样的文件。虽然仍处于试验阶段，但这个功能使得 Java  应用在被虚拟机启动之前能够先将 Java 类编译为原生代码。此功能 旨在改进小型和大型应用程序的启动时间，同时对峰值性能的影响很 小











# java10

# java11(LTS)

ZGC

并行的 Full GC，快速的 CardTable 扫描，自适应的堆占用比例调整（IHOP），在并发标记阶段的类型卸载等等

**云计算时代的监控、诊断和** **Profiling** **能力**，这个是相比 ZGC 更具生产实践意义的特性。

**Flight Recorder（JFR）**是 Oracle 刚刚开源的强大特性

* JFR 是一套集成进入 JDK、JVM 内部的事件机制框架，通过良好架构和设计的框架，硬件层面的极致优化，生产环境的广泛验证，它可以做到极致的可靠和低开销。在 SPECjbb2015 等基准测试中，JFR 的性能开销最大不超过 1%，所以，工程师可以基本没有心理负担地在大规模分布式的生产系统使用，这意味着，我们既可以随时主动开启 JFR 进行特定诊断，也可以让系统长期运行 JFR，用以在复杂环境中进行“After-the-fact”分析。

* 在保证低开销的基础上，JFR 提供的能力可以应用在对锁竞争、阻塞、延迟，JVM GC、SafePoint 等领域，进行非常细粒度分析。甚至深入 JIT Compiler 内部，全面把握热点方法、内联、逆优化等等。JFR 提供了标准的 Java、C++ 等扩展 API，可以与各种层面的应用进行定制、集成，为复杂的企业应用栈或者复杂的分布式应用，提供 All-in-One 解决方案。



 **JEP 331: Low-Overhead Heap Profiling**

通过获取对象分配细节，为 JDK 补足了对象分配诊断方面的一些短板，工程师可以通过 JVMTI 使用这个能力增强自身的工具。

从 Java 类库发展的角度来看，JDK 11 最大的进步也是两个方面：

第一, HTTP/2 Client API，新的 HTTP API 提供了对 HTTP/2 等业界前沿标准的支持，精简而又友好的 API 接口，与主流开源 API（如，Apache HttpClient， Jetty， OkHttp 等）对等甚至更高的性能。与此同时它是 JDK 在 Reactive-Stream 方面的第一个生产实践，广泛使用了 Java Flow API 等，终于让 Java 标准 HTTP 类库在扩展能力等方面，满足了现代互联网的需求。

第二，就是安全类库、标准等方面的大范围升级，其中特别是 JEP 332: Transport Layer Security (TLS) 1.3，除了在安全领域的重要价值，它还是中国安全专家范学雷所领导的 JDK 项目，完全不同于以往的修修补补，是个非常大规模的工程。

除此之外，JDK 还在逐渐进行瘦身工作，或者偿还 JVM、Java 规范等历史欠账，例如

335: Deprecate the Nashorn JavaScript Engine

它进一步明确了 Graal 很有可能将成为 JVM 向前演进的核心选择，Java-on-Java 正在一步步的成为现实。

## JShell

java9引入了jshell这个交互性工具，让Java也可以像脚本语言一样来运行，可以从控制台启动 jshell ，在 jshell 中直接输入表达式并查看其执行结果。当需要测试一个方法的运行效果，或是快速的对表达式进行求值时，jshell 都非常实用。

除了表达式之外，还可以创建 Java 类和方法。jshell 也有基本的代码完成功能。

```shell
/help
```

## Dynamic Class-File Constants类文件新添的一种结构

Java的类型文件格式将被拓展，支持一种新的常量池格式：CONSTANT_Dynamic，加载CONSTANT_Dynamic会将创建委托给bootstrap方法。

目标

其目标是降低开发新形式的可实现类文件约束带来的成本和干扰。

## 局部变量类型推断（var ”关键字”）

var javastack = "javastack";

System.out.println(javastack);

在声明隐式类型的lambda表达式的形参时允许使用var

使用var的好处是在使用lambda表达式时给参数加上注解

(@Deprecated var x, @Nullable var y) -> x.process(y);

注意点 : 
	1) var a; 这样不可以, 因为无法推断.
	2) 类的属性的数据类型不可以使用var.

## 新API

### 集合

自 Java 9 开始，Jdk 里面为集合（List/ Set/ Map）都添加了 of 和 copyOf 方法，它们两个都用来创建不可变的集合

```java
// 快速把数据变成集合的方法
List<String> list1 = Arrays.asList("aa", "yyy", "zzz", "123");
//list1.add("ppp"); // 是一个不可以添加元素的集合

var list = List.of("Java", "Python", "C");
var copy = List.copyOf(list);
System.out.println(list == copy); // true

var list = new ArrayList<String>();
var copy = List.copyOf(list);
System.out.println(list == copy); // false
//示例1和2代码差不多，为什么一个为true,一个为false?

```

copyOf 方法会先判断来源集合是不是 AbstractImmutableList 类型的，如果是，就直接返回，如果不是，则调用 of 创建一个新的集合。

示例2因为用的 new 创建的集合，不属于不可变 AbstractImmutableList 类的子类，所以 copyOf 方法又创建了一个新的实例，所以为false.

注意：使用of和copyOf创建的集合为不可变集合，不能进行添加、删除、替换、排序等操作，不然会报 java.lang.UnsupportedOperationException 异常。

上面演示了 List 的 of 和 copyOf 方法，Set 和 Map 接口都有。

### stream

Java 9 开始对 Stream 增加了以下 4 个新方法。

 

```java
//增加单个参数构造方法，可为null
Stream.ofNullable(null).count(); // 0

//加 takeWhile 和 dropWhile 方法
Stream.of(1, 2, 3, 2, 1)
.takeWhile(n -> n < 3)
.collect(Collectors.toList()); // [1, 2]
//始计算，当 n < 3 时就截止。

Stream.of(1, 2, 3, 2, 1)
.dropWhile(n -> n < 3)
.collect(Collectors.toList()); // [3, 2, 1]
//和上面的相反，一旦 n < 3 不成立就开始计算。

//terate重载
// iterate 方法的新重载方法，可以让你提供一个 Predicate (判断条件)来指定什么时候结束迭代。
// 流的迭代, 创建流, 无限
Stream<Integer> stream1 = Stream.iterate(1, t -> (2 * t) + 1);
stream1.limit(10).forEach(System.out::println);

// 有限的迭代
Stream<Integer> stream2 = Stream.iterate(1, t -> t < 1000, t -> (2 * t) + 1);
stream2.forEach(System.out::println);
```

### 字符串

```java
//判断字符串是否为空白
"  \t  \r\n ".isBlank(); // true
//去除首尾空白
"  \t  \r\n Javastack ".strip(); // "Javastack" 去重字符串首尾的空白, 包括英文和其他所有语言中的空白字符
"  \t  \r\n Javastack ".trim();//去重字符串首尾的空白字符, 只能去除码值小于等于32的空白字符
//去除尾部空格
" Javastack ".stripTrailing(); // " Javastack"
//去除首部空格
" Javastack ".stripLeading(); // "Javastack "
//复制字符串
"Java".repeat(3);// "JavaJavaJava"
//行数统计
"A\nB\nC".lines().count(); // 3
"A\nB\nC".lines().forEach(System.out::println);
```

### Optional

现在可以很方便的将一个 Optional 转换成一个 Stream, 或者当一个空 Optional 时给它一个替代的。

```java
Optional.of("javastack").orElseThrow(); // javastack
Optional.of("javastack").stream().count(); // 1
Optional.ofNullable(null)
.or(() -> Optional.of("javastack"))
.get(); // javastack
```

### InputStream

```java
//InputStream 终于有了一个非常有用的方法：transferTo，可以用来将数据直接传输到 OutputStream，这是在处理原始数据流时非常常见的一种用法，如下示例。
var classLoader = ClassLoader.getSystemClassLoader();
var inputStream = classLoader.getResourceAsStream("javastack.txt");
var javastack = File.createTempFile("javastack2", "txt");
try (var outputStream = new FileOutputStream(javastack)) {
    inputStream.transferTo(outputStream);
}
```

### HTTP Client API

支持同步和异步

```java
//来看一下 HTTP Client 的用法：
var request = HttpRequest.newBuilder()
.uri(URI.create("https://javastack.cn"))
.GET()
.build();
var client = HttpClient.newHttpClient();
// 同步
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
// 异步
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
.thenApply(HttpResponse::body)
.thenAccept(System.out::println);
//上面的 .GET() 可以省略，默认请求方式为 Get！
```

## 移除

移除项

​	移除了com.sun.awt.AWTUtilities

​	移除了sun.misc.Unsafe.defineClass，

​	使用java.lang.invoke.MethodHandles.Lookup.defineClass来替代

​	移除了Thread.destroy()以及 Thread.stop(Throwable)方法

​	移除了sun.nio.ch.disableSystemWideOverlappingFileLockCheck、sun.locale.formatasdefault属性

​	移除了jdk.snmp模块

​	移除了javafx，openjdk估计是从java10版本就移除了，oracle jdk10还尚未移除javafx，而java11版本则oracle的jdk版本	也移除了javafx

​	移除了Java Mission Control，从JDK中移除之后，需要自己单独下载

​	移除了这些Root Certificates ：Baltimore Cybertrust Code Signing CA，SECOM ，AOL and Swisscom

废弃项

​	-XX+AggressiveOpts选项

​	-XX:+UnlockCommercialFeatures

​	-XX:+LogCommercialFeatures选项也不再需要



### 在java11中移除了不太使用的JavaEE模块和CORBA技术

CORBA来自于二十世纪九十年代，Oracle说，现在用CORBA开发现代Java应用程序已经没有意义了，维护CORBA的成本已经超过了保留它带来的好处。

但是删除CORBA将使得那些依赖于JDK提供部分CORBA API的CORBA实现无法运行。目前还没有第三方CORBA版本，也不确定是否会有第三方愿意接手CORBA API的维护工作。

在java11中将java9标记废弃的Java EE及CORBA模块移除掉，具体如下：

1. xml相关的，

   java.xml.ws, 

   java.xml.bind，

   java.xml.ws，

   java.xml.ws.annotation，

   jdk.xml.bind，

   jdk.xml.ws被移除，

只剩下java.xml，java.xml.crypto,jdk.xml.dom这几个模块；

2. java.corba，

   java.se.ee，

   java.activation，

   java.transaction被移除，

但是java11新增一个java.transaction.xa模块



### JEP : 335 : Deprecate the Nashorn JavaScript Engine

废除Nashorn javascript引擎，在后续版本准备移除掉，有需要的可以考虑使用GraalVM



### JEP : 336 : Deprecate the Pack200 Tools and API

Java5中带了一个压缩工具:Pack200，这个工具能对普通的jar文件进行高效压缩。其 实现原理是根据Java类特有的结构，合并常数 池，去掉无用信息等来实现对java类的高效压缩。由于是专门对Java类进行压缩的，所以对普通文件的压缩和普通压缩软件没有什么两样，但是对于Jar 文件却能轻易达到10-40%的压缩率。这在Java应用部署中很有用，尤其对于移动Java计算，能够大大减小代码下载量。

Java5中还提供了这一技术的API接口，你可以将其嵌入到你的程序中使用。使用的方法很简单，下面的短短几行代码即可以实现jar的压缩和解压：

压缩

Packer packer=Pack200.newPacker(); 

OutputStream output=new BufferedOutputStream(new FileOutputStream(outfile)); 

packer.pack(new JarFile(jarFile), output); 

output.close(); 

解压

Unpacker unpacker=Pack200.newUnpacker(); 

output=new JarOutputStream(new FileOutputStream(jarFile)); 

unpacker.unpack(pack200File, output); 

output.close(); 

Pack200的压缩和解压缩速度是比较快的，而且压缩率也是很惊人的，在我是使用 的包4.46MB压缩后成了1.44MB（0.322%），而且随着包的越大压缩率会根据明显，据说如果jar包都是class类可以压缩到1/9的大 小。其实JavaWebStart还有很多功能，例如可以按不同的jar包进行lazy下载和 单独更新，设置可以根据jar中的类变动进行class粒度的下载。

但是在java11中废除了pack200以及unpack200工具以及java.util.jar中的Pack200 API。因为Pack200主要是用来压缩jar包的工具，由于网络下载速度的提升以及java9引入模块化系统之后不再依赖Pack200，因此这个版本将其移除掉。

## 更简化的编译运行程序

JEP 330 : 增强java启动器支持运行单个java源代码文件的程序.

注意点 :

1. 执行源文件中的第一个类, 第一个类必须包含主方法

2. 并且不可以使用别源文件中的自定义类, 本文件中的自定义类是可以使用的.

一个命令编译运行源代码

看下面的代码。

// 编译

javac Javastack.java

// 运行

java Javastack

在我们的认知里面，要运行一个 Java 源代码必须先编译，再运行，两步执行动作。而在未来的 Java 11 版本中，通过一个 java 命令就直接搞定了，如以下所示。不会生成class文件

java Javastack.java

## Unicode 10

Unicode 10 增加了8518个字符, 总计达到了136690个字符. 并且增加了4个脚本.同时还有56个新的emoji表情符号.



## Epsilon垃圾收集器

A NoOp Garbage Collector

JDK上对这个特性的描述是: 开发一个处理内存分配但不实现任何实际内存回收机制的GC, 一旦可用堆内存用完, JVM就会退出.

如果有System.gc()调用, 实际上什么也不会发生(这种场景下和-XX:+DisableExplicitGC效果一样), 因为没有内存回收, 这个实现可能会警告用户尝试强制GC是徒劳.

用法 : -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC

使用这个选项的原因 :

提供完全被动的GC实现, 具有有限的分配限制和尽可能低的延迟开销,但代价是内存占用和内存吞吐量.

众所周知, java实现可广泛选择高度可配置的GC实现. 各种可用的收集器最终满足不同的需求, 即使它们的可配置性使它们的功能相交. 有时更容易维护单独的实现, 而不是在现有GC实现上堆积另一个配置选项.

主要用途如下 :

* 性能测试(它可以帮助过滤掉GC引起的性能假象)
* 内存压力测试(例如,知道测试用例 应该分配不超过1GB的内存, 我们可以使用-Xmx1g –XX:+UseEpsilonGC, 如果程序有问题, 则程序会崩溃)
* 非常短的JOB任务(对象这种任务, 接受GC清理堆那都是浪费空间)
* VM接口测试
* Last-drop 延迟&吞吐改进

## ZGC

GC暂停时间不会超过10ms

既能处理几百兆的小堆, 也能处理几个T的大堆(OMG)

和G1相比, 应用吞吐能力不会下降超过15%

为未来的GC功能和利用colord指针以及Load barriers优化奠定基础

初始只支持64位系统

ZGC的设计目标是：支持TB级内存容量，暂停时间低（<10ms），对整个程序吞吐量的影响小于15%。 将来还可以扩展实现机制，以支持不少令人兴奋的功能，例如多层堆（即热对象置于DRAM和冷对象置于NVMe闪存），或压缩堆。

 

GC是java主要优势之一. 然而, 当GC停顿太长, 就会开始影响应用的响应时间.消除或者减少GC停顿时长, java将对更广泛的应用场景是一个更有吸引力的平台. 此外, 现代系统中可用内存不断增长,用户和程序员希望JVM能够以高效的方式充分利用这些内存, 并且无需长时间的GC暂停时间.

STW – stop the world



ZGC是一个并发, 基于region, 压缩型的垃圾收集器, 只有root扫描阶段会STW, 因此GC停顿时间不会随着堆的增长和存活对象的增长而变长.

ZGC : avg 1.091ms max:1.681

G1  : avg 156.806 max:543.846

 

用法 : -XX:+UnlockExperimentalVMOptions –XX:+UseZGC, 因为ZGC还处于实验阶段, 所以需要通过JVM参数来解锁这个特性

##  完全支持Linux容器（包括Docker）

许多运行在Java虚拟机中的应用程序（包括Apache Spark和Kafka等数据服务以及传统的企业应用程序）都可以在Docker容器中运行。但是在Docker容器中运行Java应用程序一直存在一个问题，那就是在容器中运行JVM程序在设置内存大小和CPU使用率后，会导致应用程序的性能下降。这是因为Java应用程序没有意识到它正在容器中运行。随着Java 10的发布，这个问题总算得以解决，JVM现在可以识别由容器控制组（cgroups）设置的约束。可以在容器中使用内存和CPU约束来直接管理Java应用程序，其中包括：

遵守容器中设置的内存限制
在容器中设置可用的CPU
在容器中设置CPU约束
Java 10的这个改进在Docker for Mac、Docker for Windows以及Docker Enterprise Edition等环境均有效。

容器的内存限制
在Java 9之前，JVM无法识别容器使用标志设置的内存限制和CPU限制。而在Java 10中，内存限制会自动被识别并强制执行。

Java将服务器类机定义为具有2个CPU和2GB内存，以及默认堆大小为物理内存的1/4。例如，Docker企业版安装设置为2GB内存和4个CPU的环境，我们可以比较在这个Docker容器上运行Java 8和Java 10的区别。

首先，对于Java 8：

docker container run -it -m512 --entrypoint bash openjdk:latest
$ docker-java-home/bin/java -XX:+PrintFlagsFinal -version | grep MaxHeapSize
uintx MaxHeapSize                              := 524288000                          {product}
openjdk version "1.8.0_162"
1
2
3
4
最大堆大小为512M或Docker EE安装设置的2GB的1/4，而不是容器上设置的512M限制。

相比之下，在Java 10上运行相同的命令表明，容器中设置的内存限制与预期的128M非常接近：

docker container run -it -m512M --entrypoint bash openjdk:10-jdk
$ docker-java-home/bin/java -XX:+PrintFlagsFinal -version | grep MaxHeapSize
size_t MaxHeapSize                              = 134217728                          {product} {ergonomic}
openjdk version "10" 2018-03-20
1
2
3
4
设置可用的CPU
默认情况下，每个容器对主机CPU周期的访问是无限的。可以设置各种约束来限制给定容器对主机CPU周期的访问。Java 10可以识别这些限制：

docker container run -it --cpus 2 openjdk:10-jdk
jshell> Runtime.getRuntime().availableProcessors()
$1 ==> 2
1
2
3
分配给Docker EE的所有CPU会获得相同比例的CPU周期。这个比例可以通过修改容器的CPU share权重来调整，而CPU share权重与其它所有运行在容器中的权重相关。此比例仅适用于正在运行的CPU密集型的进程。当某个容器中的任务空闲时，其他容器可以使用余下的CPU时间。实际的CPU时间的数量取决于系统上运行的容器的数量。这些可以在Java 10中设置：

docker container run -it --cpu-shares 2048 openjdk:10-jdk
jshell> Runtime.getRuntime().availableProcessors()
$1 ==> 2
1
2
3
cpuset约束设置了哪些CPU允许在Java 10中执行。

docker run -it --cpuset-cpus="1,2,3" openjdk:10-jdk
jshell> Runtime.getRuntime().availableProcessors()
$1 ==> 3
1
2
3
分配内存和CPU
使用Java 10，可以使用容器设置来估算部署应用程序所需的内存和CPU的分配。我们假设已经确定了容器中运行的每个进程的内存堆和CPU需求，并设置了JAVA_OPTS配置。例如，如果有一个跨10个节点分布的应用程序，其中五个节点每个需要512Mb的内存和1024个CPU-shares，另外五个节点每个需要256Mb和512个CPU-shares。

请注意，1个CPU share比例由1024表示。

对于内存，应用程序至少需要分配5Gb。

512Mb × 5 = 2.56Gb
256Mb × 5 = 1.28Gb
该应用程序需要8个CPU才能高效运行。

1024 x 5 = 5个CPU
512 x 5 = 3个CPU
最佳实践是建议分析应用程序以确定运行在JVM中的每个进程实际需要多少内存和分配多少CPU。但是，Java 10消除了这种猜测，可以通过调整容器大小以防止Java应用程序出现内存不足的错误以及分配足够的CPU来处理工作负载。

## 支持G1上的并行完全垃圾收集

对于 G1 GC，相比于 JDK 8，升级到 JDK 11 即可免费享受到：并行的 Full GC，快速的 CardTable 扫描，自适应的堆占用比例调整（IHOP），在并发标记阶段的类型卸载等等。这些都是针对 G1 的不断增强，其中串行 Full GC 等甚至是曾经被广泛诟病的短板，你会发现 GC 配置和调优在 JDK11 中越来越方便。

## Low-Overhead Heap Profiling免费的低耗能飞行记录仪和堆分析仪。

通过JVMTI的SampledObjectAlloc回调提供了一个开销低的heap分析方式

提供一个低开销的, 为了排错java应用问题, 以及JVM问题的数据收集框架, 希望达到的目标如下 :

  提供用于生产和消费数据作为事件的API

  提供缓存机制和二进制数据格式

  允许事件配置和事件过滤

  提供OS,JVM和JDK库的事件

## 实现RFC7539中指定的ChaCha20和Poly1305两种加密算法, 代替RC4

实现 RFC 7539的ChaCha20 and ChaCha20-Poly1305加密算法

 

RFC7748定义的秘钥协商方案更高效, 更安全. JDK增加两个新的接口

```java
XECPublicKey 和 XECPrivateKey
KeyPairGenerator kpg = KeyPairGenerator.getInstance(“XDH”);
NamedParameterSpec paramSpec = new NamedParameterSpec(“X25519”);
kpg.initialize(paramSpec);
KeyPair kp = kgp.generateKeyPair();

KeyFactory kf = KeyFactory.getInstance(“XDH”);
BigInteger u = new BigInteger(“xxx”);
XECPublicKeySpec pubSpec = new XECPublicKeySpec(paramSpec, u);
PublicKey pubKey = kf.generatePublic(pubSpec);

KeyAgreement ka = KeyAgreement.getInstance(“XDH”);
ka.init(kp.getPrivate());
ka.doPhase(pubKey, true);
byte[] secret = ka.generateSecret();
```

## Java Flight Recorder

Flight Recorder源自飞机的黑盒子

 

Flight Recorder以前是商业版的特性，在java11当中开源出来，它可以导出事件到文件中，之后可以用Java Mission Control来分析。可以在应用启动时配置java -XX:StartFlightRecording，或者在应用启动之后，使用jcmd来录制，比如

$ jcmd < pid> JFR.start
$ jcmd < pid> JFR.dump filename=recording.jfr
$ jcmd < pid> JFR.stop

 

是 Oracle 刚刚开源的强大特性。我们知道在生产系统进行不同角度的 Profiling，有各种工具、框架，但是能力范围、可靠性、开销等，大都差强人意，要么能力不全面，要么开销太大，甚至不可靠可能导致 Java 应用进程宕机。

而 JFR 是一套集成进入 JDK、JVM 内部的事件机制框架，通过良好架构和设计的框架，硬件层面的极致优化，生产环境的广泛验证，它可以做到极致的可靠和低开销。在 SPECjbb2015 等基准测试中，JFR 的性能开销最大不超过 1%，所以，工程师可以基本没有心理负担地在大规模分布式的生产系统使用，这意味着，我们既可以随时主动开启 JFR 进行特定诊断，也可以让系统长期运行 JFR，用以在复杂环境中进行“After-the-fact”分析。还需要苦恼重现随机问题吗？JFR 让问题简化了很多。

在保证低开销的基础上，JFR 提供的能力也令人眼前一亮，例如：我们无需 BCI 就可以进行 Object Allocation Profiling，终于不用担心 BTrace 之类把进程搞挂了。对锁竞争、阻塞、延迟，JVM GC、SafePoint 等领域，进行非常细粒度分析。甚至深入 JIT Compiler 内部，全面把握热点方法、内联、逆优化等等。JFR 提供了标准的 Java、C++ 等扩展 API，可以与各种层面的应用进行定制、集成，为复杂的企业应用栈或者复杂的分布式应用，提供 All-in-One 解决方案。而这一切都是内建在 JDK 和 JVM 内部的，并不需要额外的依赖，开箱即用。

 

















# java12

## switch表达式（预览）

### 传统switch的弊端

* 匹配是自上而下的，如果忘记写break, 后面的case语句不论匹配与否都会执行； 
* 所有的case语句共用一个块范围，在不同的case语句定义的变量名不能重复； 
* 不能在一个case里写多个执行结果一致的条件； 
* 整个switch不能作为表达式返回值；

### 新

* 省去了 break 语句
* 多个 case 合并到一行
* 不能混用 -> 和 : 

```java
switch(fruit){
    case PEAR -> System.out.println(4);
    case APPLE,MANGO,GRAPE -> System.out.println(5);
    case ORANGE,PAPAYA -> System.out.println(6);
    default -> throw new IllegalStateException("No Such Fruit:" + fruit);
};
int numberOfLetters = switch(fruit){
    case PEAR -> 4;
    case APPLE,MANGO,GRAPE -> 5;
    case ORANGE,PAPAYA -> 6;
    default -> throw new IllegalStateException("No Such Fruit:" + fruit);
};
```

##  Shenandoah GC：低停顿时间的GC（预览）

Shenandoah GC 主要目标是 99.9% 的暂停小于 10ms，暂停与堆大小无关等

### 工作原理 

从原理的角度，我们可以参考该项目官方的示意图，其内存结构与 G1 非常相似，都是将内存划分为类似棋盘的 region。整体流程与 G1 也是比较相似的，最大的区别在于实现了并发的 疏散(Evacuation) 环节，引入的 Brooks Forwarding Pointer 技术使得 GC 在移动对象时，对象引用仍然可以访问。



这种对于低延迟的保证，也是以消耗 CPU 等计算资源为代价的

```shell
-XX:+AlwaysPreTouch：使用所有可用的内存分页，减少系统运行停顿，为避免运行时性能损失。

-Xmx == -Xmsv：设置初始堆大小与最大值一致，可以减轻伸缩堆大小带来的压力，与 AlwaysPreTouch 参数配
合使用，在启动时提交所有内存，避免在最终使用中出现系统停顿。

-XX:+ UseTransparentHugePages：能够大大提高大堆的性能，同时建议在 Linux 上使用时将
/sys/kernel/mm/transparent_hugepage/enabled 和
/sys/kernel/mm/transparent_hugepage/defragv 设置为：madvise，同时与 AlwaysPreTouch 一起使
用时，init 和 shutdownv 速度会更快，因为它将使用更大的页面进行预处理。

-XX:+UseNUMA：虽然 Shenandoah 尚未明确支持 NUMA（Non-Uniform Memory Access），但最好启用此功
能以在多插槽主机上启用 NUMA 交错。与 AlwaysPreTouch 相结合，它提供了比默认配置更好的性能。

-XX:+DisableExplicitGC：忽略代码中的 System.gc() 调用。当用户在代码中调用 System.gc() 时会强制
Shenandoah 执行 STW Full GC ，应禁用它以防止执行此操作，另外还可以使用 
-XX:+ExplicitGCInvokesConcurrent，在 调用 System.gc() 时执行 CMS GC 而不是 Full GC，建议在有
System.gc() 调用的情况下使用。

不过目前 Shenandoah 垃圾回收器还被标记为实验项目，如果要使用Shenandoah GC需要编译时--with-jvmfeatures选项带有shenandoahgc，然后启动时使用参数
-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC
```

##  JVM 常量 API

Java 12 中引入 JVM 常量 API，用来更容易地对关键类文件 (key class-file) 和运行时构件（artefact）的名义描述 (nominal description) 进行建模，特别是对那些从常量池加载的常量，这是一项非常技术性的变化，能够以更简 单、标准的方式处理可加载常量。 

具体来说就是java.base模块新增了java.lang.constant包（而非 java.lang.invoke.constant ）。包中定义了一系列基 于值的符号引用（JVMS 5.1）类型，它们能够描述每种可加载常量。



引入了ConstantDesc接口及Constable接口

**此 API 对于操作类和方法 的工具很有帮助。**

```java
private static void testDescribeConstable() {
    System.out.println("======test java 12 describeConstable======");
    String name = "尚硅谷Java高级工程师";
    Optional<String> optional = name.describeConstable();
    System.out.println(optional.get());
}
```

## 微基准测试套件

JMH，即Java Microbenchmark Harness，是专门用于代码微基准测试的工具套件。何谓Micro Benchmark呢？简 单的来说就是基于方法层面的基准测试，精度可以达到微秒级。当你定位到热点方法，希望进一步优化方法性能的时 候，就可以使用JMH对优化的结果进行量化的分析。

应用场景： 

* 想准确的知道某个方法需要执行多长时间，以及执行时间和输入之间的相关性；
* 对比接口不同实现在给定条件下的吞吐量；
* 查看多少百分比的请求在多长时间内完成；

JMH的使用

需要Maven环境，JMH的源代码以及官方提供的Sample就是使用Maven进行项目管理的， github上也有使用gradle的例子可自行搜索参考。使用mvn命令行创建一个JMH工程

```shell
mvn archetype:generate \
-DinteractiveMode=false \
-DarchetypeGroupId=org.openjdk.jmh \
-DarchetypeArtifactId=jmh-java-benchmark-archetype \
-DgroupId=co.speedar.infra \
-DartifactId=jmh-test \
-Dversion=1.0
```

在现有Maven项目中使用JMH，只需要把生成出来的两个依赖以及shade插件拷贝到项目的pom中即可

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>0.7.1</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>0.7.1</version>
    <scope>provided</scope>
</dependency>
...
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <finalName>microbenchmarks</finalName>
                <transformers>
                    <transformer    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>org.openjdk.jmh.Main</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 新特性的说明

 Java 12 中添加一套新的基本的微基准测试套件（microbenchmarks suite），此功能为JDK源代码添加了一套微基准 测试（大约100个），**简化了现有微基准测试的运行和新基准测试的创建过程**。使开发人员可以轻松运行现有的微基 准测试并创建新的基准测试，其目标在于提供一个稳定且优化过的基准。 它基于Java Microbenchmark Harness（JMH），可以轻松测试JDK性能，支持JMH更新。 

微基准套件与 JDK 源代码位于同一个目录中，并且在构建后将生成单个 jar 文件。但它是一个单独的项目，在支持构 建期间不会执行，以方便开发人员和其他对构建微基准套件不感兴趣的人在构建时花费比较少的构建时间。 

要构建微基准套件，用户需要运行命令：make build-microbenchmark， 类似的命令还有：make test TEST="micro:java.lang.invoke" 将使用默认设置运行 java.lang.invoke 相关的微基准测试。

## 只保留一个 AArch64 实现

Java 12 中将删除由 Oracle 提供的 arm64端口相关的所有源码，即删除目录 open/src/hotspot/cpu/arm 中关于 64-bit 的这套实现，只保留其中有关 32-bit ARM端口的实现，余下目录的 open/src/hotspot/cpu/aarch64 代码 部分就成了 AArch64 的默认实现。

## 默认生成类数据共享(CDS)归档文件

类数据共享机制 (Class Data Sharing ，简称 CDS) 的概念，通过把一些核心 类在每个JVM间共享，每个JVM只需要装载自己的应用类即可。好处是：启动时间减少了，另外核心类是共享的，所 以JVM的内存占用也减少了。

JDK 12之前，想要利用CDS的用户，即使仅使用JDK中提供的默认类列表，也必须 java -Xshare:dump 作为额外的步 骤来运行。 

Java 12 针对 64 位平台下的 JDK 构建过程进行了增强改进，使其默认生成类数据共享（CDS）归档，以进一步达到 改进应用程序的启动时间的目的，同时也避免了需要手动运行：java -Xshare:dump 的需要，修改后的 JDK 将在 ${JAVA_HOME}/lib/server 目录中生成一份名为classes.jsa的默认archive文件(大概有18M)方便大家使用。 

当然如果需要，也可以添加其他 GC 参数，来调整堆大小等，以获得更优的内存分布情况，同时用户也可以像之前一 样创建自定义的 CDS 存档文件。

## 可中断的 G1 Mixed GC

当 G1 垃圾回收器的回收超过暂停时间的目标，则能中止垃圾回收过程。

## 增强G1，自动返回未用堆内存给操作系统

在空闲时自动将 Java 堆内存返还给操作系统

G1 垃圾收集器将在应用程序不活动期间定期生成或持续循环检查整体 Java 堆使用情况，以便 G1 垃圾收集器能够更及时的将 Java 堆中不使用内存部分返还给操作系统。对于长时间处于空闲 状态的应用程序，此项改进将使 JVM 的内存利用率更加高效

JDK12的这个特性新增了两个参数分别是G1 PeriodicGCInterval及G1 PeriodicGCSystemLoadThreshold，设置为0 的话，表示禁用。如果应用程序为非活动状态，在下面两种情况任何一个描述下，G1 回收器会触发定期垃圾收集： 

* 自上次垃圾回收完成以来已超过 G1PeriodicGCInterval ( milliseconds )， 并且此时没有正在进行的垃圾回收 任务。如果 G1PeriodicGCInterval 值为零表示禁用快速回收内存的定期垃圾收集。 
* 应用所在主机系统上执行方法 getloadavg()，默认一分钟内系统返回的平均负载值低于 G1PeriodicGCSystemLoadThreshold指定的阈值，则触发full GC或者concurrent GC( 如果开启 G1PeriodicGCInvokesConcurrent )，GC之后Java heap size会被重写调整，然后多余的内存将会归还给操作 系统。如果 G1PeriodicGCSystemLoadThreshold 值为零，则此条件不生效。 如果不满足上述条件中的任何一个，则取消当期的定期垃圾回收。等一个 G1PeriodicGCInterval 时间周期后，将重 新考虑是否执行定期垃圾回收



如 果定期垃圾收集严重影响程序执行，则需要考虑整个系统 CPU 负载，或让用户禁用定期垃圾收集。

## 其他

### 支持unicode 11

### 支持压缩数字格式化

```java
var cnf = NumberFormat.getCompactNumberInstance(Locale.CHINA,
NumberFormat.Style.SHORT);
System.out.println(cnf.format(1_0000));
System.out.println(cnf.format(1_9200));
System.out.println(cnf.format(1_000_000));
System.out.println(cnf.format(1L << 30));
System.out.println(cnf.format(1L << 40));
System.out.println(cnf.format(1L << 50));
/**
1万
2万
100万
11亿
1兆
1126兆
**/
```

### String新增方法

transform

```java
var result = "foo".transform(input -> input + " bar");
System.out.println(result); // foo bar
var result = "foo".transform(input -> input + " bar").transform(String::toUpperCase)
System.out.println(result); // FOO BAR
```

indent调整String实例的缩进

```java
String result = "Java\n Python\nC++".indent(3);
System.out.println(result);
/**
   Java
	Python
   C++
```

### Files新增mismatch方法

```java
FileWriter fileWriter = new FileWriter("tmp\\a.txt");
fileWriter.write("a");
fileWriter.write("b");
fileWriter.write("c");
fileWriter.close();
FileWriter fileWriterB = new FileWriter("tmp\\b.txt");
fileWriterB.write("a");
fileWriterB.write("1");
fileWriterB.write("c");
fileWriterB.close();
System.out.println(Files.mismatch(Path.of("tmp/a.txt"),Path.of("tmp/b.txt")));//1
```

# java13

## switch表达式（预览）

引入了yield语句，用于返回值

```java
int i = switch (x) {
    case "1":
        yield 1;
    case "2":
        yield 2;
    default:
        yield 3;
}
int i = switch (x) {
    case "1" -> 1;
    case "2" -> 2;
    default -> {
        yield 3;
    }
};
```

## 文本块（预览）

在字符串常量池

编译器会去除多余空格(开头与"""对其 结尾都去掉)

```java
String newQuery = """
    select employee_id,last_name,salary,department_id
    from employees
    where department_id in (40,50,60)
    order by department_id asc
    """;
    String code =
    """
        String text = \"""
        	A text block inside a text block
        \""";
    """;
```

## 动态CDS档案（动态类数据共享归档）

CDS，是java 12的特性了，可以让不同 Java 进程之间共享一份类元数据，减少内存占用，它还能加快应用的启动速 度。而JDK13的这个特性支持在Java application执行之后进行动态archive。存档类将包括默认的基础层CDS存档 中不存在的所有已加载的应用程序和库类。也就是说，在Java 13中再使用AppCDS的时候，就不再需要这么复杂了。 

该提案处于目标阶段，旨在提高AppCDS的可用性，并消除用户进行试运行以创建每个应用程序的类列表的需要。 

使用示例：

```bash
# JVM退出时动态创建共享归档文件：导出jsa
java -XX:ArchiveClassesAtExit=hello.jsa -cp hello.jar Hello
# 用动态创建的共享归档文件运行应用:使用jsa
java -XX:SharedArchiveFile=hello.jsa -cp hello.jar Hello
```

## ZGC:取消使用未使用的内存

GC释放的内存会还给操作系统

## 重新实现旧版套接字API

全新实现的 NioSocketImpl 来替换JDK1.0的PlainSocketImpl。

* 它便于维护和调试，与 NewI/O (NIO) 使用相同的 JDK 内部结构，因此不需要使用系统本地代码。 
* 它与现有的缓冲区缓存机制集成在一起，这样就不需要为 I/O 使用线程栈。 
* 它使用 java.util.concurrent 锁，而不是 synchronized 同步方法，增强了并发能力。 
* 新的实现是Java 13中的默认实现，但是旧的实现还没有删除，可以通过设置系统属性 jdk.net.usePlainSocketImpl来切换到旧版本。

##  其他

增加项 

* 添加FileSystems.newFileSystem(Path, Map) Method 
* 新的java.nio.ByteBuffer Bulk get/put Methods Transfer Bytes Without Regard to Buffer Position 
* 支持Unicode 12.1 
* 添加-XX:SoftMaxHeapSize Flag，目前仅仅对ZGC起作用 
* ZGC的最大heap大小增大到16TB







































































# java14

## 语法

### instanceof

```java
//举例1：
if(obj instanceof String str){ //新特性：省去了强制类型转换的过程
    System.out.println(str.contains("Java"));
}else{
    System.out.println("非String类型");
}
public boolean equals(Object o){
    return o instanceof Monitor other && model.equals(other.model) && price == other.price;
}
```

### NullPointerException

改进了NullPointerException的可读性，能更准确地给出null变量的信息

-XX:+ShowCodeDetailsInExceptionMessages开启

### Record类型

当你用record 声明一个类时，该类将自动拥有以下功能：

* 获取成员变量的简单方法，以上面代码为例 name() 和 partner() 。注意区别 于我们平常getter的写法
* 一个 equals 方法的实现，执行比较时会比较该类的所有成员属性
* 重写 hashCode
* 一个可以打印该类所有成员属性的 toString 方法。
* 请注意只会有一个构造方法。

还可以在Record声明的类中定义静态字段、静态方法、构造器或实例方法。

不能在Record声明的类中定义实例字段；类不能声明为abstract；不能声明显式的父类等。

```java
public record Person(String name,Person partner) {
    //还可以声明静态的属性、静态的方法、构造器、实例方法
    public static String nation;
    public static String showNation(){
        return nation;
    }
    public Person(String name){
        this(name,null);
    }
    public String getNameInUpperCase(){
        return name.toUpperCase();
    }
    //不可以声明非静态的属性
//    private int id;//报错
}
//不可以将record定义的类声明为abstract的
//abstract record Order(){
//
//}

//不可以给record定义的类声明显式的父类（非Record类）
//record Order() extends Thread{
//
//}
```

### switch表达式

```java
//jdk12
enum Week {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}
Week day = Week.FRIDAY;
switch (day){
    case MONDAY,TUESDAY,WEDNESDAY -> System.out.println(1);
    case THURSDAY -> System.out.println(2);
    case FRIDAY,SATURDAY -> System.out.println(3);
    case SUNDAY -> System.out.println(4);
    default -> throw new IllegalStateException("What day is today?" + day);
}
//使用变量接收switch表达式的值
int num = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY -> 1;
    case THURSDAY -> 2;
    case FRIDAY, SATURDAY -> 3;
    case SUNDAY -> 4;
    default -> throw new IllegalStateException("What day is today?" + day);
};
System.out.println(num);

//jdk13新特性：引用了yield关键字，用于返回指定的数据，结束switch结构
String x = "3";
int num = switch (x){
    case "1" -> 1;
    case "2" -> 2;
    case "3" -> 3;
    default -> {
        System.out.println("default...");
        yield 4;
    }
};
System.out.println(x);
String x = "3";
int num = switch (x){
    case "1":yield 1;
    case "2":yield 2;
    case "3":yield 3;
    default: yield 4;
};
System.out.println(num);
```

### 文本块

```java
//jdk13新特性：
String sql1 = """
    SELECT id,NAME,email
    FROM customers
    WHERE id > 4
    ORDER BY email DESC
    """;

//jdk14
// \:取消换行操作
// \s:表示一个空格
String sql2 = """
    SELECT id,NAME,email \
    FROM customers\s\
    WHERE id > 4 \
    ORDER BY email DESC
    """; 
```

## 底层

### 弃用ParallelScavenge和SerialOld GC组合

### 删除CMS垃圾回收器

CMS的弊端 ：

1. 会产生内存碎片，导致并发清除后，用户线程可用的空间不足。 
2. 既然强调了并发（Concurrent），CMS收集器对CPU资源非常敏感
3. CMS 收集器无法处理浮动垃圾

### ZGC

任意堆内存大小低延迟

使用了读屏障、染色指针和内存多重映射等技术来 实现可并发的标记-压缩算法的，

以低延迟为首要目标的一款垃圾收集器



现在mac或Windows上也能使用ZGC了，示例如下： -XX:+UnlockExperimentalVMOptions -XX:+UseZGC

















# java15

历史版本的主要新特性

<font color="red">JDK 5</font>: enum、泛型、自动装箱与拆箱、可变参数、增强循环等
JDK 6: 支持脚本语言、JDBC4.0API
JDK 7: 支持try-with-resources、switch语句块增加String支持、NIO2.0包
<font color="red">JDK 8</font>: lambda表达式、Stream API、新的日期时间的API、方法引用、构造器引用
JDK 9: 模块化系统、jshell
JDK 10: 局部变量的类型推断
<font color="red">JDK 11</font>: ZGC的引入、Epsilon GC
JDK 12: switch表达式、Shenandoah GC、增强G1
JDK 13: switch表达式引入yield、文本块
JDK 14: instanceof模式识别、Records、弃用Parallel Scavenge+Serial GC组合、删除CMS GC



孵化器模块（Incubator/孵化版/实验版）  大概率移除

预览特性（Preview/预览版）                        小概率移除







**新特性概述**

http://openjdk.java.net/projects/jdk/15/

14项主要的增强/更改，包括一个孵化器模块，三个预览功能，两个不推荐使用的功能以及两个删除功能。

## 主要新特性

1. **Sealed Classes(Preview)密封类**

   密封的类和接口限制继承或实现

   使用修饰符sealed，您可以将一个类声明为密封类。密封的类使用reserved关键字permits列出可以直接扩展它的类。子类可以是最终的，非密封的或密封的

   支持模式匹配的前提

   ```java
   public sealed class Person permits Teacher,Student,Worker{ } //人
   final class Teacher extends Person { }//教师
   sealed class Student extends Person permits MiddleSchoolStudent,GraduateStudent{ } //学生
   final class MiddleSchoolStudent extends Student { }  //中学生
   final class GraduateStudent extends Student { }  //研究生
   non-sealed class Worker extends Person { }  //工人
   class RailWayWorker extends Worker{} //铁路工人
   ```

2. **Hidden Classes(隐藏类)**

   启用标准 API 来定义无法发现且具有有限生命周期的隐藏类，从而提高 JVM 上所有语言的效率。

   - 隐藏类天生为框架设计的，在运行时生成内部的class。
   - 隐藏类只能通过反射访问，不能直接被其他类的字节码访问。
   - 隐藏类可以独立于其他类加载、卸载，这可以减少框架的内存占用。

   Hidden Classes就是不能直接被其他class的二进制代码使用的class, 主要被一些框架用来生成运行时类，这些类不是用来直接使用，而是通过反射机制调用   -- 比如lambda表达式

   特性

   - 不可发现性。为某些静态的类动态生成的动态类，把这个动态生成的类看做是静态类的一部分。不会被除了该静态类之外的其他机制发现。
   - 访问控制。我们希望在访问控制静态类的同时，也能控制到动态生成的类。
   - 生命周期。动态生成类的生命周期一般都比较短，我们并不需要将其保存和静态类的生命周期一致。

   API的支持

   ```java
   java.lang.reflect.Proxy可以定义隐藏类作为实现代理接口的代理类。
   java.lang.invoke.StringConcatFactory可以生成隐藏类来保存常量连接方法；
   java.lang.invoke.LambdaMetaFactory可以生成隐藏的nestmate类，以容纳访问封闭变量的lambda主体；
   ```

   普通类是通过调用ClassLoader::defineClass创建的，而隐藏类是通过调用Lookup::defineHiddenClass创建的。这使JVM从提供的字节中派生一个隐藏类，链接该隐藏类，并返回提供对隐藏类的反射访问的查找对象。调用程序可以通过返回的查找对象来获取隐藏类的Class对象。

   

3. **Pattern Matching for instanceof (Second Preview) instanceof 自动匹配模式**

   模式匹配允许程序中的通用逻辑（主要是从对象中的条件提取组件）可以更简洁地表达

   ```java
   //新特性之前：
   @Test
   public void test1(){
       Object obj = new String("hello,before Java14");
       if(obj instanceof String){
           String str = (String) obj;//必须显式的声明强制类型转换。
           System.out.println(str.contains("Java"));
       }else{
           System.out.println("非String类型");
       }
   }
   //使用新特性
   @Test
   public void test2(){
       Object obj = new String("hello,Java15");
       obj = null;
       if(obj instanceof String str){ //新特性：省去了强制类型转换的过程
           System.out.println(str.contains("Java"));
       }else{
           System.out.println("非String类型");
       }
   }
   --------------------------------------------------------
   private String model;
   private double price;
   public boolean equals(Object o){
       return o instanceof Monitor other && model.equals(other.model) && price == other.price;
   }
   ```

   

4. **ZGC 功能转正**

   ZGC是Java 11引入的新的垃圾收集器（JDK9以后默认的垃圾回收器是G1）
   自 2018 年以来，ZGC 已增加了许多改进，从并发类卸载、取消使用未使用的内存、对类数据共享的支持到改进的 NUMA 感知。此外，最大堆大小从 4 TB 增加到 16 TB。支持的平台包括 Linux、Windows 和 MacOS。

   ZGC是一个重新设计的并发的垃圾回收器，通过减少 GC 停顿时间来提高性能。

   但是这并不是替换默认的GC，默认的GC仍然还是G1；之前需要通过-XX:+UnlockExperimentalVMOptions -XX:+UseZGC来启用ZGC，现在只需要-XX:+UseZGC就可以。相信不久的将来它必将成为默认的垃圾回收器。

   相关的参数有ZAllocationSpikeTolerance、ZCollectionInterval、ZFragmentationLimit、ZMarkStackSpaceLimit、ZProactive、ZUncommit、ZUncommitDelay ZGC-specific JFR events(ZAllocationStall、ZPageAllocation、ZPageCacheFlush、ZRelocationSet、ZRelocationSetGroup、ZUncommit)也从experimental变为product

   

5. **文本块功能转正**

   ```java
   // \:取消换行操作
   // \s:表示一个空格
   String sql2 = """
       SELECT id,NAME,email \
       FROM customers\s\
       WHERE id > 4 \
       ORDER BY email DESC\
       """;
   ```

   

6. **Records Class（预览）**

   方便创建一个常量类

   当你用Record 声明一个类时，该类将自动拥有以下功能：

   - 获取成员变量的简单方法，以上面代码为例 name() 和 partner() 。注意区别于我们平常getter的写法。
   - 一个 equals 方法的实现，执行比较时会比较该类的所有成员属性
   - 重写 equals 当然要重写 hashCode
   - 一个可以打印该类所有成员属性的 toString 方法。
   - 请注意只会有一个构造方法。

   ```java
   public record Customer(String name,Customer partner) {
       //还可以声明构造器、静态的变量、静态的方法、实例方法
   
       public Customer(String name){
           this(name,null);
       }
       public static String info;
       public static void show(){
           System.out.println("我是一个客户");
       }
       public void showName(){
           System.out.println("我的名字是：" + name());
       }
       //不可以在Record中定义实例变量
   //    public int id;
   }
   //Record不可以声明为abstract的
   //abstract record User(){}
   
   //Record不可以显式的继承于其他类
   //record User() extends Thread{}
   ```

   

## 次要新特性

1. EdDSA 数字签名算法

   基于Edwards-Curve数字签名算法(EdDSA-Edwards-Curve Digital Signature Algorithm)的加密签名，即爱德华兹曲线数字签名算法

   在许多其它加密库（如 OpenSSL 和 BoringSSL）中得到支持。

   与 JDK 中的现有签名方案相比，EdDSA 具有更高的安全性和性能，因此备受关注。它已经在OpenSSL和BoringSSL等加密库中得到支持，在区块链领域用的比较多。

   EdDSA是一种现代的椭圆曲线方案，具有JDK中现有签名方案的优点。EdDSA将只在SunEC提供商中实现。

   

2. 重新实现 DatagramSocket API

   重新实现了遗留的套接字API。
   java.net.datagram.Socket和java.net.MulticastSocket的当前实现可以追溯到JDK 1.0，那时IPv6还在开发中。因此，当前的多播套接字实现尝试调和IPv4和IPv6难以维护的方式。

   通过替换 java.net.datagram 的基础实现，重新实现旧版 DatagramSocket API。
   更改java.net.DatagramSocket 和 java.net.MulticastSocket 为更加简单、现代化的底层实现。提高了 JDK 的可维护性和稳定性。

   通过将java.net.datagram.Socket和java.net.MulticastSocket API的底层实现替换为更简单、更现代的实现来重新实现遗留的DatagramSocket API。
   新的实现：1.易于调试和维护;2.与Project Loom中正在探索的虚拟线程协同。

   

3. 禁用偏向锁定

   在默认情况下禁用偏向锁定，并弃用所有相关命令行选项。目标是确定是否需要继续支持偏置锁定的高维护成本的遗留同步优化，HotSpot虚拟机使用该优化来减少非竞争锁定的开销。尽管某些Java应用程序在禁用偏向锁后可能会出现性能下降，但偏向锁的性能提高通常不像以前那么明显。

   该特性默认禁用了biased locking(-XX:+UseBiasedLocking)，并且废弃了所有相关的命令行选型(BiasedLockingStartupDelay, BiasedLockingBulkRebiasThreshold, BiasedLockingBulkRevokeThreshold, BiasedLockingDecayTime, UseOptoBiasInlining, PrintBiasedLockingStatistics and PrintPreciseBiasedLockingStatistics)

   

4. Shenandoah 垃圾回收算法转正

   Shenandoah 的暂停时间与堆大小无关

   Shenandoah和ZGC的关系

   - 性能几乎可认为是相同的
   - 不同点：ZGC是Oracle JDK的。而Shenandoah只存在OpenJDK中，因此使用时需注意你的JDK版本

   使用-XX:+UseShenandoahGC命令行参数打开

   

5. 外部存储器访问 API

   目的是引入一个 API，以允许 Java 程序安全、有效地访问 Java 堆之外的外部存储器。如本机、持久和托管堆。

   有许多Java程序是访问外部内存的，比如Ignite和MapDB。该API将有助于避免与垃圾收集相关的成本以及与跨进程共享内存以及通过将文件映射到内存来序列化和反序列化内存内容相关的不可预测性。该Java API目前没有为访问外部内存提供令人满意的解决方案。但是在新的提议中，API不应该破坏JVM的安全性。

   

6. 移除 Solaris 和 SPARC 端口

7. 移除the Nashorn JS引擎

   在JDK11中取以代之的是GraalVM。GraalVM是一个运行时平台，它支持Java和其他基于Java字节码的语言，但也支持其他语言，如JavaScript，Ruby，Python或LLVM。性能是Nashorn的2倍以上。

   

   Graal VM在HotSpot VM基础上增强而成的跨语言全栈虚拟机，可以作为“任何语言”的运行平台使用。语言包括：Java、Scala、Groovy、Kotlin；C、C++、JavaScript、Ruby、Python、R等

   

8. 废弃RMI 激活机制

   现代应用程序，分布式系统大部分都是基于Web的，web服务器已经解决了穿越防火墙，过滤请求，身份验证和安全性的问题，并且也提供了很多延迟加载的技术。
   所以在现代应用程序中，RMI Activation已经很少被使用到了。并且在各种开源的代码库中，也基本上找不到RMI Activation的使用代码了。
   为减少RMI Activation的维护成本，在JDK8中，RMI Activation被置为可选的。在JDK15中废弃。



## order

- **升级了Unicode，支持Unicode 13.0**
- **给CharSequence新增了isEmpty方法**
- **JDK15对TreeMap提供了putIfAbsent, computeIfAbsent, computeIfPresent, compute, merge方法提供了overriding实现**
- **优化了默认G1 Heap Region Size的计算**
- jcmd的GC.heap_dump命令现在支持gz选型，以dump出gzip压缩版的heap；compression level从1(fastest)到9(slowest, but best compression)，默认为1
- jdk.net.ExtendedSocketOptions新增了SO_INCOMING_NAPI_ID选型
- 新增了jdk.tls.client.SignatureSchemes及jdk.tls.server.SignatureSchemes用于配置TLS Signature Schemes
- 支持certificate_authorities的扩展

# java16

# java17(LTS)



