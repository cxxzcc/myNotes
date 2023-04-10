








# 算法

## 时间轮算法

HashedWheelTimer使用时间轮数据结构的工具类

时间轮是一种高效来利用线程资源来进行批量化调度的一种调度模型

把大批量的调度任务全部都绑定到同一个的调度器上面，使用这一个调度器来进行所有任务的管理（manager），触发（trigger）以及运行（runnable）。能够高效的管理各种延时任务，周期任务，通知任务等等

在Linux内核中使用广泛，是Linux内核定时器的实现方法和基础之一


优点
* **可以在一个线程中动态的添加定时(延时)任务**
	* Timer,ScheduledExecutorService,Spring的Scheduled这些都是无法做到这一点,线程开启定时任务,不能再添加
* 适合很多小定时任务
	* 订单超时
	* 分布式锁中为线程续期的看门狗
	* 心跳检测


### 时间轮原理

一种环形的数据结构，可以想象成时钟，分成很多格子，一个格子代表一段时间（这个时间越短，Timer的精度越高）。并用一个链表表示在该格子上的到期任务，同时一个指针随着时间一格一格转动，并执行相应格子中的到期任务。任务通过取摸决定放入那个格子

![image.png](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/20230410101110.png)

构造方法的三个参数
* tickDuration  每一tick的时间
- timeUnit  时间单位
- ticksPerWheel  就是轮子一共有多个格子

将间隔时间的长度先用hash进行处理,放入wheel的tick 即为HashedWheelTimer

### 注意点
* 一个HashedWheelTimer对象只有一个worker线程
* 每次添加的任务只会执行一次
* 时间轮的参数非常重要
	假设tick 6s 添加两个定时任务 一个延时2s一个延时5s,最终同时执行,因为分配到同一个tick
* 所有的任务都是顺序串行执行的
	上一个任务的异常延时会影响到下一个任务
	所以时间轮执行的任务都是比较快的
* HashedWheelTimer被实例化启动后，会创建一个新的线程，因此，你的项目里应该只创建它的唯一一个实例 



