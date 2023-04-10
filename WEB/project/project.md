
# 定时任务

## 单机

### 定时调度

Spring @Scheduled
* cron 表达式
* fixedRated类配置参数。常用的案例：

```java
// cron表达式
@Scheduled(cron="0 0/30 9-17 * * ?") //按cron规则执行,朝九晚五工作时间内每半小时
@Scheduled(cron="0 0 12 ? * WED") //按cron规则执行,表示每个星期三中午12点

// fixedRated类配置
@Scheduled(fixedRate=5000) //上一次开始执行时间点后5秒再次执行；
@Scheduled(fixedDelay=3000) //上一次执行完毕时间点后3秒再次执行；
@Scheduled(initialDelay=1000, fixedDelay=2000)//第一次延时1s,在上次执行完2s后执行

```
用于业务处理流程比较简单的场景，比如定时生成简单报表，发送通知。对于复杂耗时的场景，处理效率不高，业务高峰期会积压大量待处理数据，影响业务。

### 定时+批处理

为了解决复杂耗时场景下定时调度效率不高的问题，可以引入批处理框架

Spring Batch

Spring Batch批处理框架将任务拆分成多个Step，同时每个Step里面又分为itemReader,itemProcessor, itemWriter。通过将任务分层细化，能够让多个阶段并行处理，提高任务处理效率。批处理框架结合定时调度框架，可以在单机情况下，对大量复杂的业务进行高效的批处理。


## 集群
xxljob
quartz
