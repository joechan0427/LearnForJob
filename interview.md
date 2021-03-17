# java基础

## 请你解释为什么会出现4.0-3.6=0.40000001这种现象？
`4.0`的`float`二进制存储为:`0 10000001 0*23`
`3.6`的`float`二进制存储为:`0 10000000 11001...`(存在误差)

**follow up:**
可以使用BigInteger和BigDecimal解决,底层使用的是String

## 请你讲讲一个浮点数在内存中是怎么存的？
补码形式,即:
正数与原码相同
负数为绝对值的原码取反后加一
小数为首位符号位,然后将整数和小数分别表示出来,将小数点移到第一个1的右边,左移为正,右移为负,将移位数加127后放入30到23位,剩下的位数放此时小数点所在的位置的右边23位

## full fc 出现情况以及后果, 调优
**出现情况**
1. 老年代空间不足
2. 方法区空间不足
3. 调用 System.gc(), 不一定, 只是建议虚拟机

**后果:**
stw 时间 比 minor gc 长, 一般是 10 倍以上

**解决:**
1. 思想就是减少内存碎片, 或者增加 gc 的次数, 以减少晋升失败的可能
2. 使用 G1(jdk11 默认), 因为整体采用复制-压缩算法, 可以减少内存碎片, 同时可以不用硬性规定新生代, 老年代. 并且可以尽量满足设定的 stw 时间
3. 对于 cms 这种回收器(采用标志-清除算法), 可以减少触发的阈值, 让 CMS GC 尽早执行
# 网络
## 使用 linux 命令实现一个软路由

# 项目

## top 指令
```shell
top - 14:06:23 up 70 days, 16:44,  2 users,  load average: 1.25, 1.32, 1.35
Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
Cpu(s):  5.9%us,  3.4%sy,  0.0%ni, 90.4%id,  0.0%wa,  0.0%hi,  0.2%si,  0.0%st
Mem:  32949016k total, 14411180k used, 18537836k free,   169884k buffers
Swap: 32764556k total,        0k used, 32764556k free,  3612636k cached
```

1. **第一行, 系统平均负债率**
```shell
top - 14:06:23 up 70 days, 16:44,  2 users,  load average: 1.25, 1.32, 1.35
```
load average: 1.15, 1.42, 1.44
表示过去 1, 5, 15 分钟的负债情况, 如过去15 分钟有 1.44个进程占用 cpu
2. **第二行, 进程相关**
```shell
Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
```
![](https://pic2.zhimg.com/80/v2-6796586b616e9ea296e7114615416b2d_1440w.jpg)
总共进程数, 运行, 休眠, 停止, 僵死
3. **第三行, cpu 状态**
```shell
Cpu(s):  5.9%us,  3.4%sy,  0.0%ni, 90.4%id,  0.0%wa,  0.0%hi,  0.2%si,  0.0%st
```
![](https://pic2.zhimg.com/80/v2-face652aa489f870177a94bf8d936981_1440w.jpg)
cpu 负载量: 可简单理解为进程数
cpu 利用率: 可简单理解为 cpu 实际工作的占比
4. **第四行, 内存状态**
```shell
Mem:  32949016k total, 14411180k used, 18537836k free,   169884k buffers
```
也可使用 `free -m(以m为单位)` 查看
5. **第五行, swap**
```shell
Swap: 32764556k total,        0k used, 32764556k free,  3612636k cached
```
硬盘划分出来的用于虚拟内存的区域, 如果这个值持续变化, 说明硬盘频繁的 I/O 操作, 物理内存可能不足

**进程:**
![](https://pic4.zhimg.com/80/v2-ca4da63f21f18d05c327aa370f2aa34f_1440w.jpg)


## 一个接口很慢的原因定位
1. 内存使用过高，频繁gc导致cpu占满
2. 内存使用不高，出现了类似死循环场景(同样 cpu 占满)
3. 死锁

## 假设有锁, 排查的行动/ cpu 利用率过高/ 一个线程占 cpu 过高
1. 先使用 `top` 然后键盘 `P` 进行交互, 按 cpu 利用率排序(ps: `M` 是按内存使用排序)
2. 找到占用高的 java 进程
3. `top -p 进程号` 可以查看占用高的线程
4. 但是我们需要将其转换成 16 进制
5. 使用 `jstack 进程id | grep 线程id` 查看到当前线程执行情况
```shell
"Abandoned connection cleanup thread" #13 daemon prio=5 os_prio=0 tid=0x00007f3d1544c000 nid=0x7fa5 in Object.wait() [0x00007f3cfce26000]
   java.lang.Thread.State: TIMED_WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x00000000c518e910> (a java.lang.ref.ReferenceQueue$Lock)
	at com.mysql.cj.jdbc.AbandonedConnectionCleanupThread.run(AbandonedConnectionCleanupThread.java:70)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```
### 假如是死锁
第五步 jstack 能显示
```shell
Found one Java-level deadlock:
=============================
"Thread-0":
  waiting to lock monitor 0x0000000026e64a00 (object 0x0000000718992ce8, a java.lang.Object),
  which is held by "Thread-1"
"Thread-1":
  waiting to lock monitor 0x0000000026e64b00 (object 0x0000000718992cd8, a java.lang.Object),
  which is held by "Thread-0"

Java stack information for the threads listed above:
===================================================
"Thread-0":
        at pkg.DeadLock$Thread1.run(DeadLock.java:20)
        - waiting to lock <0x0000000718992ce8> (a java.lang.Object)
        - locked <0x0000000718992cd8> (a java.lang.Object)
        at java.lang.Thread.run(java.base@11.0.8/Thread.java:834)
"Thread-1":
        at pkg.DeadLock$Thread2.run(DeadLock.java:34)
        - waiting to lock <0x0000000718992cd8> (a java.lang.Object)
        - locked <0x0000000718992ce8> (a java.lang.Object)
        at java.lang.Thread.run(java.base@11.0.8/Thread.java:834)

Found 1 deadlock.
```
## 接口测试可以看出什么
![](https://pic2.zhimg.com/80/v2-deae7ff96efe1362b90fdb2af91629dd_1440w.jpg)
后端重点关注, 接口响应时间, 并发数, qps, 服务端资源使用情况
## qps 和 并发数
1. QPS
QPS Queries Per Second 是每秒查询率 ,是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准, 即每秒的响应请求数，也即是最大吞吐能力。
2. TPS
TPS Transactions Per Second 也就是事务数/秒。一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数

区别: Qps基本类似于Tps，但是不同的是，对于一个页面的一次访问，形成一个Tps；但一次页面请求，可能产生多次对服务器的请求，服务器对这些请求，就可计入“Qps”之中。

并发数（并发度）：指系统同时能处理的请求数量，同样反应了系统的负载能力。这个数值可以分析机器1s内的访问日志数量来得到

```math
QPS(TPS)=并发数/平均响应时间
```
一个系统吞吐量通常有QPS(TPS),并发数两个因素决定，每套系统这个两个值都有一个相对极限值，在应用场景访问压力下，只要某一项达到系统最高值，系统吞吐量就上不去了，如果压力继续增大，系统的吞吐量反而会下降，原因是系统超负荷工作，上下文切换，内存等等其他消耗导致系统性能下降

- pv : page view a day
- uv : unique view
- dau : day active user, 日活量. DAU通常统计一日（统计日）之内，登录或使用了某个产品的用户数（去除重复登录的用户），与UV概念相似
# 数据库
## mysql 如何去分析sql的性能

## 如何根据 explain 的结果调优


## 慢SQL
阿里云 mysql 监控与报警 引擎监控 qps 
### 慢 SQL 查找
（1）设置开启：SET GLOBAL slow_query_log = 1;　　　#默认未开启，开启会影响性能，mysql重启会失效
（2）查看是否开启：SHOW VARIABLES LIKE '%slow_query_log%';
（3）设置阈值：SET GLOBAL long_query_time=3;
（4）查看阈值：SHOW 【GLOBAL】 VARIABLES LIKE 'long_query_time%';　　#重连或新开一个会话才能看到修改值
（5）通过修改配置文件my.cnf永久生效，在[mysqld]下配置：
　　[mysqld]
　　slow_query_log = 1;　　#开启
　　slow_query_log_file=/var/lib/mysql/atguigu-slow.log　　　#慢日志地址，缺省文件名host_name-slow.log
　　long_query_time=3;　　  #运行时间超过该值的SQL会被记录，默认值>10
　　log_output=FILE　　　　　　　　　　　
 (6) 慢 sql 语句将存放在 mysql 目录下

### 慢查询优化基本步骤

0. 先运行看看是否真的很慢，**注意设置SQL_NO_CACHE**

1. where条件单表查，锁定最小返回记录表。这句话的意思是把查询语句的where都应用到表中返回的记录数最小的表开始查起，单表每个字段分别查询，看哪个字段的区分度最高

2. explain查看执行计划，是否与1预期一致（从锁定记录较少的表开始查询）

3. order by limit 形式的sql语句让排序的表优先查

4. 了解业务方使用场景

5. 加索引时参照建索引的几大原则

6. 观察结果，不符合预期继续从0分析

### 具体例子
有一个财务统计(统计各个学校学生的消费, 充值情况)的 sql 语句, 查询了 order 表(每天 2w 的订单), 并且用到了多表 join, 而且根据时间排序. 此时 explain 出来的结果是全表扫描, 而且用到了 filesort (没有利用索引排序). 
解决方法: 加上联合索引 (区分度高的放前面, 如学校), 后续把财务查询单独新建表, 记录以天为单位, 配合定时任务, 每天把前一天的消费情况从 order 表 transfer 到新表

> filesort 有两种排序方式
    1. 对需要排序的记录生成 <sort_key,rowid> 的元数据进行排序，该元数据仅包含排序字段和rowid。排序完成后只有按字段排序的rowid，因此还需要通过rowid进行回表操作获取所需要的列的值，可能会导致大量的随机IO读消耗；
    2. 对需要排序的记录生成 <sort_key,additional_fields> 的元数据，该元数据包含排序字段和需要返回的所有列。排序完后不需要回表，但是元数据要比第一种方法长得多，需要更多的空间用于排序。

### sql 编写优化步骤:
1. 避免所有字段都返回
