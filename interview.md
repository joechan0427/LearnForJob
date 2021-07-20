# java基础
## a+=b a=a+b区别
涉及到自动类型转换, 低类型可以自动转换为高级别类型, 反之不行
```java
byte a = 1
byte b = 2
a += b // 没有设计自动转换, 成功执行
a = a+b // 会变成 a = (int)a+(int)b, 会报错
```

同时 `++`, `--` 有自动强制类型转换, 即使是小于 `int` 的类型

## for 和 foreach

1. for是使用下标（偏移量）定位的.
2. foreach是使用迭代器来遍历的
3. 对于ArrayList支持随机访问的数据类型,使用for的下标访问会比foreach的迭代器遍历的方式稍快一些，因为迭代器需要计算next的地址。
4. 而对于LinkedList只支持顺序查找的数据类型，由于它不支持随机访问，所以基于下标的访问每次都会从头查询，此时使用for就会效率非常低，而使用foreach进行高效的next地址运算效率会比for高很多。

但两者在遍历过程中删除 Arraylist 或 linkedlist 元素都会出现快速失败

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

# java 并发
## 如何优化多线程上下文切换
频繁上下文切换原因
1. 线程数过多
2. 频繁阻塞, 导致时间片未用完而切换上下文
3. 垃圾回收, 因为需要 stw

**优化**
1. 合理设置线程数量
2. 减少垃圾回收次数
3. 优化锁竞争
  3.1 减少锁持有的时间
  即将一些与锁无关的代码移除同步代码块
  3.2 降低锁的粒度
  比如 concurrentHashMap 1.8 将锁粒度减小使其并发度提高
  3.3 乐观锁代替
  3.4 优化 wait/notify



## 线程池的核心参数有哪些？
1. 核心线程数
2. 最大线程数
3. 存活时间 (2个, 时间加单位)
4. 阻塞队列
5. 线程工厂
6. 拒绝策略

# jvm
## 内存溢出和内存泄漏
### 内存溢出
分为堆溢出, 方法区溢出, 栈溢出

方法区溢出: 由于加载的类过多导致, 一般只能通过增加物理内存大小, 
  - 其中==注意默认-XX:MetaspaceSize 大约 20m, 这个参数表示触发 full gc 的大小, 建议把此值增大==
  - 而 -XX:MaxMetaspaceSize 默认不设置, 即可用大小为内存空间

栈溢出: 调用方法太多或递归没有正确退出, 通过 -Xss 参数设置一个线程拥有的栈大小, 默认是 1m

堆溢出: 首先要排除内存泄漏, 之后可以通过 -Xmn（新生代大小）–Xms（初始值） -Xmx（最大值）参数手动设置 Heap（堆）的大小

### 内存泄漏
0. 通过 `jps` 查到进程的 pid
1. 通过 `jstat -gc pid` 查看 gc 情况, 如果 full gc 次数过多, 则有可能发生内存泄漏
  ![](https://upload-images.jianshu.io/upload_images/6918995-bad5d58eeb218c8c.png)
  该指令能看出 s0, s1, eden 区, old 区, 方法区的分配情况(c)和使用情况(u), Ygc, full gc 时间, 次数
2. 可以通过 `jstack` 查看线程数量, 线程过多也可能造成内存使用过大
2. 通过 `jmap -dumb:format=b,file=xxx.log pid` 保存堆现场
3. 下载堆文件, 通过工具解决, 比如 MAT, 可以选择 memory leak suspect

[参考资料](https://zhenbianshu.github.io/2018/12/troubleshooting_java_memory_leak.html)

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
## traceroute
1. 用于查看到目标 ip 所经过的路由器
利用 icmp 协议和 ip 报文头部的 ttl 实现, 第一个报文把 ttl 设置为 1, 则第一个路由器就会返回 icmp 超时差错报文, 第二个为 2 ...
2. 确定路径的MTU
## 为什么快重传要是在 3 次相同的 ack 之后
1. 一次是正常到达
2. 两次可能是乱序, 也可能是丢包
3. 三次则是丢包的概率比较大

比如发送 2,3,4,5 四个包, 如果按序到达服务端应该返回 ack(6)
如果 3 先到达, 服务端会发送 ack(2)
如果之后是 4, 服务端会第二次发送 ack(2)
而 2 在没丢包情况下比 5 还要慢到达的概率已经很小了

## socket 网络编程
[参考](https://blog.csdn.net/vipshop_fin_dev/article/details/102966081)
![](https://upload-images.jianshu.io/upload_images/206633-4b2d8622b6d9d48d.png?imageMogr2/auto-orient/strip|imageView2/2/w/696/format/webp)
![](https://img-blog.csdnimg.cn/20191111201405457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3ZpcHNob3BfZmluX2Rldg==,size_16,color_FFFFFF,t_70)
1. 服务端开启 socket
2. 服务端将 socket 与端口绑定
3. 服务端循环监听该端口
4. 客户端建立tcp连接
5. 客户端发送数据
6. 客户端 close

服务端在 java 中实现
```java
Socket socket = new Socket(8080);
while (true) {
  socket.accept();//阻塞的
  // 利用线程池去与不同的客户端通信
  BufferedReader bufferedReader =new BufferedReader(new InputStreamReader(socket.getInputStream(),"UTF-8"));
}
```

客户端在 java 的实现
```java
Socket socket = new Socket("127.0.0.1", "8080");
BufferedWriter bufferedWriter =new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
```

## 长连接短连接
1. 长连接才可以实现服务端推送, 否则服务端不知道客户端ip的情况下不能主动与客户端通信

## 301 
![](https://img-blog.csdnimg.cn/img_convert/6acb5a0292608eb656a6aca0ea30d97e.png)

存在于 location 字段, 浏览器收到后将进行缓存, 此后再访问原网站也会直接查询缓存访问新网站

## 既然IP层会分片，为什么TCP层还需要MSS呢
> MTU ：⼀个网络包的最⼤长度，以太网网中一般为1500 字节；
MSS ：除去 IP 和 TCP 头部之后，⼀个网络包所能容纳的TCP数据的最⼤长度；

**ip 分片并不是一个理想的选择**
当 ip 层出现一个大于 mtu 的数据时, ip 层便需要对其进行分片, 但是一旦某个分片在传输过程中丢失, 则整个数据包都需要重传, 即使其他分片成功到达

因此, tcp 发现数据超过 mss 时, 组成的网络包大小也就会超过 mtu, 便会先对其分片

## 浏览器输入网址过程
1. **解析 url**
比如是 http, 然后双斜杠, 然后是 web 服务器, 可能还会有具体某个资源目录以斜杠分开, get 请求可能还携带参数, 以 ? 开始, & 分割
2. **生成 http 请求包**
首行是方法, 然后是请求的 url, 然后是http 版本
然后各个字段名和字段值, 比如 cookie, 报文长度
如果是 post 请求, 把参数就放在报文体
3. **dns**
一般输入都是域名, 我们需要得到对应的 ip 地址, dns 查询是个递归加迭代的过程, 使用 udp
4. **tcp**
http 使用 tcp 协议, 因此需要建立 tcp 连接, 会经历 3 次握手
5. **ip**
加上 ip 头部, 头部会存放源 ip 地址和目标 ip 地址
6. **mac**
利用路由表得到下一跳的 ip 地址, 然后需要得到下一跳的 mac 地址, 先查看缓存, 如果没有需要利用 arp 协议
7. **交换机**(不一定需要)
根据 mac 地址表得到转发端口
8. **路由器**
根据路由表得到转发端口, 此时也可能需要根据 arp 协议得到 mac 地址


## 使用 linux 命令实现一个软路由

## tcp 握手的丢失问题
1. 第一次握手丢失
此时服务端还不知道客户端发起请求, 因此主要是客户端的重传
2. 第二次握手丢失
服务端: 一直未收到第三次握手, 会重传 5 次, 之后还未收到第三次握手则关闭
客户端: 由于第二次握手丢失, 客户端会重传第一次握手, 服务端再次收到第一次握手后会马上重传第二次握手
3. 第三次握手丢失
客户端: 由于收到第二次握手, 客户端已建立连接, 进入 established 状态, 此时可以马上发送数据
服务端: 当一直收不到第三次握手, 会重传 5 次第二次握手, ==如果收到客户端发送的正常数据, 服务端会正常建立!==
[参考](https://stackoverflow.com/questions/16259774/what-if-a-tcp-handshake-segment-is-lost)

## tcp 挥手丢失问题
1. 第一次挥手丢失
2. 第二次挥手丢失
客户端: 重新发送第一次挥手
服务端: 收到重复的第一次挥手后, 会立刻重发第二次挥手. ==或者当服务端发送第三次挥手, 客户端接收后直接由 FIN_WAIT1 状态跳转到 TIME_WAIT 状态==
3. 第三次挥手丢失
服务端: 重传
客户端: 等待, 因为此时客户端已不能发送报文, ==tcp 没有对此状态的处理, 但是 Linux 有(防止服务端突然跑路), 设定一个超时时间 tcp_fin_timeout, 超过后, 直接进入 close==
4. 第四次挥手丢失
这就是 time_wait 等待 2MSL 的意义
服务端: 重发第三次挥手
客户端: 再次收到服务端的第三次挥手, 会发送第四次挥手. ==如果此时已和新的服务器建立连接, 收到上一次连接的第三次挥手, 会直接向其发送 RST, 服务端收到后, 进入CLOSE==


## 为什么需要挥手的 time_wait 需要等待 2*MSL
1. 因为客户端最后一个报文可能丢失, 如果丢失, 则服务端不能进入关闭状态, 会一直重发一个请求确认的报文, 如果客户端在这个 TIME-WAIT 状态再次收到服务端的请求确认的报文, 则会再发送一次最后确认报文
2. 为了让这次 TCP 连接的所有报文消失在网络中, 不影响下次连接

## 为什么握手是三次, 挥手是四次
因为握手时, 第二次握手是请求建立连接和确认请求放在一个报文里
而挥手时, 服务端接收到客户端的连接释放请求后, 可以继续发送数据, 因此需要分开确认报文和连接释放报文

## 如果已经建立了连接，但是客户端突然出现故障了怎么办？
1. TCP还设有一个保活计时器，显然，客户端如果出现故障，服务器不能一直等下去，白白浪费资源。服务器每收到一次客户端的请求后都会重新复位这个计时器，时间通常是设置为2小时，若两小时还没有收到客户端的任何数据，服务器就会发送一个探测报文段，以后每隔75秒发送一次。若一连发送10个探测报文仍然没反应，服务器就认为客户端出了故障，接着就关闭连接。
2. 可以在应用层自己实现心跳, 由客户端间隔一段时间向服务端发送心跳包, 服务端不回应, 只是检测与上次心跳包间隔

## tcp 产生 rst 标志的几种情况
> 在TCP协议中，rst段标识复位，用来异常的关闭连接。在TCP的设计中它是不可或缺的，发送rst段关闭连接时，不必等缓冲区的数据都发送出去，直接丢弃缓冲区中的数据。而接收端收到rst段后，也不必发送ack来确认. 此后, ==双方进入 close 状态==

1. 目标端口未监听
2. 防火墙拦截了(比如为了防止 rst 攻击)
  > rst 攻击: 冒充服务端向客户端发送 rst 标志, 导致客户端不正常关闭连接
3. 向已关闭的端口发送数据(比如服务端处于 FIN_WAIT2, 客户端发送数据)

## 客户端 close-wait, TIME-WAIT 状态过多会产生什么后果？怎样处理？
当客户端向服务端的 80 端口发起 tcp 连接时, 会随机一个端口用于通信(比如, 12345), ==此时会在服务端的进程的文件描述符占用一个五元组(协议, 本地ip, 本地端口, 远程ip, 远程端口)==, 所以一个客户端可以向同一服务器的同一端口建立两个不同的请求, 但是不能过多, 因为客户端端口有限, 而每次请求的端口不一致

### close-wait 过多
当客户端异常退出, 而服务端没有处理这个事件, 就会导致 close-wait 状态
**危害:** 但有大量 close-wait, 服务端==相当于要打开多个文件==, 因为==要在 fd 里占一个位置==, 而打开的文件数是有限的, 这样会导致==后续无法响应其他连接==
**处理:** 使用 epoll 监听客户端退出的事件 EPOLLRDHUP, 当该事件发生, 直接 close 掉这个连接

### time-wait 过多
对客户端来说: 当同时建立大量短连接, 释放时会出现 time-wait 过多, 此时可能导致端口不够用
对服务端来说: 
1. time-wait 状态的连接会占用一些内存, 但比 established 状态少, 所以大量 time-wait 状态会使可用内存减少. 
2. time-wait 状态==已经没有占用 fd 的位置了, 所以不影响后续其他请求==
3. 一般来说, 对服务端影响不大, 而且最长占用 2MSL 的时间(一般是两分钟). 比如服务进程被 kill, 那么之前建立的连接就会变成 time-wait. 此时在 2MSL 的时间内重启服务会报端口占用
  [参考](https://stackoverflow.com/questions/1803566/what-is-the-cost-of-many-time-wait-on-the-server-side/1806033#1806033)

---
# 项目
## 百万流水及总和
[流水](https://cloud.tencent.com/developer/article/1648766?from=article.detail.1649325)

## 项目遇到的困难以及是怎么解决
  
  1. 在做 jwt 鉴权时, 需要使用到 spring security 框架
  2. 从认证, 鉴权的理论到实际的 coding 有一定距离. 因为 spring security 是一个重量的框架, 里面涉及了很多的过滤链, 而且也使用了很多的设计模式

  解决:
  1. 看源码
  2. 从官网的 demo 开始, 边看手册边 debug, 然后做笔记

### spring security 的设计模式
1. 责任链模式
过滤器链就是一种责任链模式。一个请求到达后，被过滤器链中的过滤器逐个进行处理，过滤器链中的过滤器每个都具有不同的职能并且互不相扰，我们还可以通过 HttpSecurity 来动态配置过滤器链中的过滤器（即添加/删除过滤器链中的过滤器）

2. 代理模式. 
spring security 的门面是 DelegatingFilterProxy 这个类, 但这个类并不在 spring security 这个包里, 而是在 Spring Web 包中的类. 因为 spring 考虑兼容多种 web 安全框架, 因此使用这个代理类来代理真正的 SpringSecurityFilterChain

3. 模板方法模式
  认证的时候有一个抽象类, 里面已经完成了大部分的认证流程, 只有两个方法是抽象方法
    - retrieveUser : 表示从某个数据源得到用户对象
    - additionalAuthenticationChecks : 校验客户端登录凭证的校验. 

4. 适配器模式
WebSecurityConfigurerAdapter 在配置时, 可以选择性的配置想要修改的那一部分配置，而不用覆盖其他不相关的配置
其大部分的配置已经在 init 方法完成, 我们只需要重写 configure 方法

5. 委托模式


## 实习学习到什么


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
表示过去 1, 5, 15 分钟的负载情况, 如过去15 分钟有 1.44个进程占用 cpu
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

## jwt
### CSRF (cross-site request forgery)
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9RQjZHNFpvRTE4NkRPbWhGVjUweGZMVEpWRHNpYkltUFNpYk1qOHRWY0FhSDU2YjgzbUZMOEFaeVZaUXRkemJrcWtvZVlvOWRnbFhzamlicjhFNjB5WWRSQS82NDA)

#### 防御
1. referer check
  在 http 协议, 头部有一个字段是 referer, 记录 http 请求来源, 可以通过该值来屏蔽非同源请求
  **弊端:**
    1. 使用该方法相当于将本站的安全交给浏览器来守护
    2. 该值在非主流/古老浏览器可能会被篡改
    3. 用户可以选择关闭该字段
2. 加验证码
  **弊端**
    1. 用户体验差

#### XSS
XSS全称cross-site scripting（跨站点脚本），是一种代码注入攻击，是当前 web 应用中最危险和最普遍的漏洞之一。攻击者向网页中注入恶意脚本，当用户浏览网页时，脚本就会执行，进而影响用户，比如关不完的网站、盗取用户的 cookie 信息从而伪装成用户去操作，危害数据安全。

### jwt原理
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9RQjZHNFpvRTE4NkRPbWhGVjUweGZMVEpWRHNpYkltUFNlSkdiMFVqTEtOT2RNaWFFNFBuTkE5YnFVdU1tSGVNaldzeFNyYWJXYk9QWWljeVFtS1JjMnlkZy82NDA?x-oss-process=image/format,png)
#### jwt 结构
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9RQjZHNFpvRTE4NkRPbWhGVjUweGZMVEpWRHNpYkltUFNXU3pENGdzUE5KeWliNk5RTHhSWFA3eTBQTmlhdjBaaDN0aWFBZUNkaWFFd2pCdDYxbjdUNDdsWUdBLzY0MA?x-oss-process=image/format,png)

1. **头部 (header)**
- 声明类型: jwt
- 声明签名算法: hs256
```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```
之后经过 base64url 算法编码得到实际头部
2. **载荷 (payload)**
存放有效信息, ==不要存放敏感信息==, 因为==默认不加密, 只是编码==
- 签发人
- 过期时间
- 生效时间
- 签发时间

还可以定义私有字段

3. **签名 (signature)**
对前两部分签名, 防止篡改
使用对称加密算法 HS256 进行签名, 因此服务端必须存储一个密钥 (secret), 按下面的公式签名
```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

### jwt 存放在哪里
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9RQjZHNFpvRTE4NkRPbWhGVjUweGZMVEpWRHNpYkltUFNHcE4xQmZUSGJNWm16ZXVYUHBqVXZwUEJpYXZMNFVZeWplV0lFSE1idWt6NXRWWWZieHFZT0h3LzY0MA?x-oss-process=image/format,png)
1. 保存在 localStorage
2. 保存在 sessionStorage

该域内的 js 脚本可以读取, 并将其放在请求的 http 的 header 里, 但是存在 XSS 风险

3. 保存在 cookie
只是保存在 cookie, 不使用 cookie 鉴权, 发送请求时, js读取 cookie 放在 header 里
4. 保存在 cookie, 并设置 httponly
意味着 cookie 不可读取和修改, 本域的 js 也不能, 因此只能使用 cookie 鉴权, 会有 csrf 问题
#### cookie, session storage, local storage
![](https://miro.medium.com/max/875/1*JiT0KNuYIO8l5QBtpNtOHA.png)
- local storage
在同源的所有标签页和窗口之间共享数据。
数据不会过期。它在浏览器重启甚至系统重启后仍然存在。
- session storage
具有相同页面的另一个标签页中将会有不同的存储。
但是，它在同一标签页下的 iframe 之间是共享的（假如它们来自相同的源）。
### jwt 的校验
服务端根据发送的 jwt 的前两部分加上服务端的 secret 生成新的签名, 将其与发送的签名比较
```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

### jwt 被窃取
1. 存在 cookie 设置 httponly, 不能防御跨域伪造
2. 在载荷里放入请求者的 ip, 必须对载荷进行加密

### 


# 数据库
## 分库分表
[分库分表](https://zhuanlan.zhihu.com/p/84224499)
[美团分库分表](https://tech.meituan.com/2016/11/18/dianping-order-db-sharding.html)
[mysql 分库分表](https://www.cnblogs.com/luao/p/10642180.html)

## 为什么不使用 hash 索引
1. hash 索引不支持遍历
2. hash 索引不支持排序
3. hash 索引不支持部分索引, 例如有联合索引 (a, b, c), hash 索引只能取三者做 hash, 当我们的 sql 语句只有 a 和 b 时, 不能使用到 hash 索引
4. 当数据量大时, 会增加 hash 冲突的概率

## 递归 sql
[sql](https://zhuanlan.zhihu.com/p/58563707)

## MyISAM 和 InnoDb 的区别
1. MyISAM 不支持事务, InnoDB 支持事务
2. MyISAM 锁粒度最小是表锁, 并发度较小, 而 InnoDB 最小是行锁, 还区分读锁和写锁, 为了进一步提高并发还引入了 MVCC
3. MyISAM 是非聚集索引, InnoDB 是聚集索引. 因此 InnoDB 索引即数据, 必须要有主键, 主键不宜过大, 否则其他辅助索引也会随之增大(因为辅助索引的叶子数据存储的是主键), 同时主键建议递增, 已避免主键索引需要频繁变动

## 数据库 delete 和TRUNCATE区别
- delete：删除表的内容，表的结构存在，索引定义还在，可以回滚恢复；
- drop：删除表内容和结构，释放空间，没有备份表之前要慎用；
- truncate：删除表的内容，表的结构存在，索引定义还在,相当于drop表以后建了一个同样定义的表（包括相关约束和索引），没有备份表之前要慎用；速度快(不可回滚)


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
```vim
　　[mysqld]
　　slow_query_log = 1;　　#开启
　　slow_query_log_file=/var/lib/mysql/atguigu-slow.log　　　#慢日志地址，缺省文件名host_name-slow.log
　　long_query_time=3;　　  #运行时间超过该值的SQL会被记录，默认值>10
　　log_output=FILE　　　
```
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
2. 避免在索引列进行函数运算

### 建立索引的注意事项
1. 区分度高的在前面
2. ==最左前缀匹配原则==，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整

## redis


### redis 实现分布式锁
![](https://pic2.zhimg.com/80/v2-f52d3fb4df8aaa20294bfb64d99c3901_1440w.jpg)

1. setnx key value 只有 key 不存在才设置, 此时 key 可作为锁
2. EXPIRE key timeout, 即使线程挂了, 也能释放锁

1, 2之间不是原子的可能导致死锁
此时可使用 lua 脚本(什么东西?)

#### 锁误删除
A 执行了超过设定的 key 存活时间, 此时锁已被 B 拿去, A 再释放时删除了 B 的锁
1. 判断当前锁是否是自己
2. 是的话再删除

1, 2又不是原子的, 见上面

#### 执行超时
A 执行了超过设定的 key 存活时间, 此时锁已被 B 拿去, 此时 A, B 并发执行
1. 将过期时间设置长一点, 弊端是 A 线程挂了, 其他线程要等待
2. 设置线程 A 的守护线程, 当 A 还存活, 守护线程定时增加过期时间, 当 A 挂了, 守护线程也自动退出

#### 锁等待
当一个线程获得锁, 其他锁得轮询锁是否释放
1. 可以使用 redis 的发布订阅的功能
2. 当获取锁失败, 订阅锁释放消息
3. 当释放锁时, 发布锁释放消息

### redis 做缓存
#### Redis与Mysql双写一致性方案解析/Redis缓存怎么解决数据一致性/怎么保证redis的时效性
- 更新缓存
- 删除缓存

一般采用删除缓存策略.

更新缓存有以下弊端: 
1. 不能保证所有缓存都是热点数据, 比如一个记录写频繁, 但读很少, 这样频繁的更新缓存却少读取, 造成浪费
2. 一个缓存可能设计多张表/记录, 可能涉及到复杂计算

---
- 先删除缓存, 再更新数据库
- 先更新数据库, 再删除缓存

一般采用先更新数据库, 再删除缓存

两种都可能发生脏读现象
- 先删除缓存
  1. A 删除缓存
  2. B 读取缓存失败
  3. B 读取 DB
  4. A 更新数据库
  5. B 将旧值写到缓存

- 先更新数据库
  1. B 读取缓存, 此时刚好失效, 或没缓存
  2. B 读取 DB
  3. A 更新数据库
  4. A 删除缓存(此时仍没缓存)
  5. B 将旧值写到缓存

第二种发生的概率极小
  1. 缓存刚好失效, 且有另一个线程并发写操作
  2. 要求读操作在写操作之前, 且写操作在读操作之前完成

---
解决方法:
延迟双删: 先更新数据库, 删除缓存, 再休眠一秒, 再删除缓存.
![](https://cdn.jsdelivr.net/gh/wliduo/CDN@1.1/2019/11/20191105003.png)
![](https://cdn.jsdelivr.net/gh/wliduo/CDN@1.1/2019/11/20191105004.png)


#### 如果 mysql 有 2000w 数据, 如何让 redis 只存 20w 热点数据?
使用内存淘汰策略中的 lru

#### 假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来
`keys pattern`(如 `keys aa*`)
但该命令是阻塞的且返回所有符合 pattern 的 keys, 可以使用 scan, 通过游标分步进行的，分次进行不会阻塞线程
`scan 游标 [MATHCH pattern] [COUNT cnt]`, 如`scan 0 MATCH aa* COUNT 11`

# Linux
## 远程拷贝命令
`scp <-r>(可选,递归复制) 源文件位置 目标文件位置`
例如:
`scp root@1.1.1.1:/opt/test /opt`
从远程 ip 1.1.1.1 处使用 root 登陆复制 /opt/test 文件到本地的 /opt 文件
命令输入后需要输入密码, 如果不指定用户名则需要输入用户名和密码

## 统计一个文件里, hello 出现的行数
grep "hello" file | wc -l

wc (word count)命令
- -c 统计字节数
- -l 统计行数
- -m 统计字符数
- -w 统计word数, 即以空格, 跳格(tab), 换行分隔开的字符串

## 查看进程下的线程情况
`ps -T -p <pid>`
或者
`top -H -p <pid>`

# spring 
## @Autowired和@Resource的区别是什么？
两者都是注入对象时使用
@Resource 不是 spring 的注解, 而是 javax.annotation 下的

共同点
1. 两者都可以写在属性或者 setter 方法上

不同点
1. 处理两者的 BeanPostProcessor 不同
2. @autowaired 只按照 byType 注入(如果需要 byName, 配合 @Qualifier 使用). @Resource 默认按照 byName, 也提供 ByType

