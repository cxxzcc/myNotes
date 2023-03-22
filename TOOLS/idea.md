[文档](https://www.jetbrains.com/help/idea/getting-started.html)

# 快捷键

```
Ctrl+ Alt+左方向键     退回到上一个操作的地方
Ctrl+ Alt+右方向键     前进到上一个操作的地方

ctrl + alt + enter    上方加一行
ctrl + shfit +enter   下方加一行

ctrl + alt + b       进入到实现子类中

Ctrl+Alt+Shift+T	重构

ctrl+alt+t     Surround With
ctrl+alt+m  	抽取
ctrl+f12         文件structure
```



## 查詢

```
Ctrl+E            最近访问文件
Ctrl+Shift+E      最近修改
Ctrl+F12          查看文件结构
Ctrl+Shift+A      搜索操作  
F2				 跳转到下一一个高亮错误或警告位置
```

## 提示

Alt+Enter      [检查正则表达式](https://www.jetbrains.com/help/idea/regular-expression-syntax-reference.html#check_regexp)意图

```
ctrl + alt + u 		显示类之间的关系
ctrl + h      		显示当前类继承关系

ctrl + p        	显示当前方法参数
ctrl+j        		查看快捷输入
```



## 格式化

```
Ctrl+Alt+O   			格式化import列表
Ctrl + Shift + Enter     格式化一行代碼
Ctrl+Alt+V				提取变量
```



## 多行操作

```java
按下滚轮上下拖动鼠标 
alt + 鼠标左键拖动  
Alt+J / Alt+Shift +J   多项选择同word光标
ctrl + shift + alt + j   所有同word

alt+shift + 鼠标点击
```

## 快捷输入
```java
.null   //判空
.nn     //判非空
.not    //取反
.cast   //强转
.return //返回值
.try
.twr
.synchronized
.sout
.soutv
.souf
.reqnonnull //判空
.lambda
.iter  //foreach
.else
```


# debug

Drop Frame 回退

https://www.jetbrains.com/help/idea/debugging-code.html

## 断点类型

- *行断点*：如果该行包含 lambda 表达式，您可以选择是设置常规行断点，还是仅在调用 lambda 时暂停程序。
- *方法断点*：在进入或退出指定方法或其实现之一时暂停程序，允许您检查方法的进入/退出条件。
- *字段观察点*：当指定的字段被读取或写入时暂停程序。这允许您对与特定实例变量的交互做出反应。例如，如果在一个复杂的过程结束时，您的一个字段的值明显错误，则设置一个字段观察点可能有助于确定故障的来源。
- *异常断点*：



## 远程调试

1 项目启动时，先允许远程调试

```shell
java -server -Xms512m -Xmx512m -Xdebug -Xnoagent  
-Djava.compiler=NONE  
-Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9081  
-Djava.ext.dirs=. ${main_class}    

起作用的就是
-Xdebug -Xnoagent -Djava.compiler=NONE  
-Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9081   
```

注意：远程调试从技术上讲，就是在本机与远程建立scoket通讯，所以端口不要冲突，而且本机要允许访问远程端口，另外这一段参数，放要在-jar 或 ${main_class}的前面

2 idea中设置远程调试

![image-20221216093517999](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20221216093517999.png)

## 临时执行表达式/修改变量的运行值

![image-20221216093553354](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20221216093553354.png)







# TOOLS

## rest client

https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html

# plugin

## Lombok

```java
@TableId(value="ID",type=IdType.AUTO)//自增主键
@Accessors(chain = true)//生成set方法采用链式编程方式


@Data //类上 提供getting setting equals canEqual hashCode toString方法
@Setter
@Getter
@Slf4j //类上 提供叫log的slf4j日志对象
@NoArgsConstructor //类上 无参构造方法
@AllArgsConstructor //类上  全参构造方法
@Builder //使用Builder模式构建对象

@RequiredArgsConstructor  构造函数注入
@Setter(onMethod = {@Autowired}) 
```



# 多服务

拷贝虚拟端口映射

![image-20221216092804644](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20221216092804644.png)

![image-20210419094140953](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419094140953.png)

![image-20210419094151482](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419094151482.png)

![image-20210419094201586](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210419094201586.png)

![image-20210427105949055](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210427105949055.png)



# 目录结构

安装目录结构

```
bin：容器，执行文件和启动参数等
help：快捷键文档和其他帮助文档
jre64：64 位java 运行环境
lib：idea 依赖的类库
license：各个插件许可
plugin：插件
```

设置目录结构

```
user/idea
config   C:\Users\cch\AppData\Roaming\JetBrains
system   C:\Users\cch\AppData\Local\JetBrains
		 C:\Users\cch\AppData\Local\JetBrains\IntelliJIdea2021.1\plugins
```

 IDEA 的各种配置的保存目录。删掉该目录，一切 都会还原到默认



config 设置目录。主要记录了：IDE 主要配置功能、自定义的代码模板、自定义的文件 模板、自定义的快捷键、Project 的 tasks 记录等等个性化的设置



system 目录是 IntelliJ IDEA 系统文件目录，主要有：缓存、索引、容器文件输出等等