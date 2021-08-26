## go module ???
`set GO111MODULE=on `
### go111Module
[go 模块](https://learnku.com/go/t/39086)

# 指针
## 注意事项
1. go 不允许指针的运行, 如&p++;
## 指针的好处
1. 传递参数的过程中, 传递指针是廉价的(只需要4/8个字节)

# 函数
## 传入变长参数
形参的最后一个参数是 `...type` 的形式, 此时函数可以传入变长参数( >= 0)
```go
func f (a, b, arg ...int) {}
```

如果参数储存在一个 slice 类型的变量中, 可以通过`f(a, b, sli...)` 的形式传入

```go
sli := []int{1,2,3}
f(4,5, sli...)
```



# 结构体
## 匿名属性
```go
type PersonC struct {
	id      int
	country string
}

//匿名属性
type Worker struct {
	//如果Worker有属性id,则worker.id表示Worker对象的id
	//如果Worker没有属性id,则worker.id表示Worker对象中的PersonC的id
	id   int
	name string
	int
	*PersonC
}
```

## 结构体的方法
```go
package main

import (
   "fmt"  
)

/* 定义结构体 */
type Circle struct {
  radius float64
}

func main() {
  var c1 Circle
  c1.radius = 10.00
  fmt.Println("圆的面积 = ", c1.getArea())
}

//该 method 属于 Circle 类型对象中的方法
func (c Circle) getArea() float64 {
  //c.radius 即为 Circle 类型对象中的属性
  return 3.14 * c.radius * c.radius
}
```

### 推荐在结构体方法里使用指针
例如
```go
  func (c *type) method() {
    c.field = "同步修改 c 的属性"
  }
  // 而不是
  func (c type) method() type{
    c.field = "调用者的属性不会改变, 需要把 c 传回去"
    return c
  }
```

ps:
1. 当结构体方法是上面 1 时, 下面的用法是正确的, 因为 go 会自动优化为 `&c.method()`
```go
  var c type
  c.method()
```

# defer
## 遵循后进先出
```go
defer fmt.Println(1);
defer fmt.Println(2);
defer fmt.Println(3);

// 结果打印 3,2,1
```

## 保存当下的参数值
```go
func a() {
    i := 0;
    defer fmt.Println(i);
    i = 1;
    return;
}
// 打印 0
```

# 数组
go 中数组是一种**值类型**(不像 c 语言是指向数组首元素的指针), 因此`var a1 = new([5]int)`, 与`var a2 = [5]int` 不一样, a1 的类型是 `*[5]int`, 而a2 的类型是`[5]int`. 当 a2 赋值给另一个数组时/或作为参数传入函数时, 会经历数组拷贝

## 数组的初始化
1. `var arr = [5]int{1,2,3,4} //其他为0`
2. `var arr = [...]int{1,2,3} //其他为0` 
3. `var arr = [5]int{1:1, 4:4}`
注意以下初始化的结果是 slice
4. `var slice = []int{5, 6, 7, 8, 22}`	
5. `var slice = []string{3: "Chris", 4: "Ron"}`

## var, make() 与 new()
### var
```go
var a int
var b sturtT
等于
a := int
b := sturtT
```
适用于值类型

### new
new(T) 返回一个**指针**, 指向一个新分配的, 类型为 T 的零值. 它适用于**值类型**如**数组**和**结构体**
与 var 的区别为new()返回的是指针

### make
make(T) **只**适用于**切片**, **map** 和 **channel**, 返回一个已经初始化(而非置零)的类型 T (而非*T), 原因在于这三种类型本身就是**引用类型(指针)**, 因此他们在使用前就必须初始化. 比如切片本身是一种包含三个内容(指向数组的指针, 长度len, 容量capacity)的数据结构, 因此在初始化之前指针为 nil, 对切片进行操作将造成空指针异常
![](https://github.com/unknwon/the-way-to-go_ZH_CN/raw/master/eBook/images/7.2_fig7.3.png?raw=true)

[参考](https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/07.2.md)


# 切片 (slice)

**注意:**
1. **当指定了切片长度时, append 将从末尾插入**
  ```go
  sli := make(int[], 3)
  sli2 := append(sli, 1, 2)
  // 此时 sli 的值为 [0,0,0], sli2 为[0, 0, 0, 1, 2]
  ```

# map
和数组不同，map 可以根据新增的 key-value 对动态的伸缩，因此它不存在固定长度或者最大限制。但是你也可以选择标明 map 的初始容量 capacity，就像这样：`make(map[keytype]valuetype, cap)`

**注意:**
1. **关于 map 不能进行结构体 value 赋值的问题**
  ```go
  var m map[string]Student
  // 此行编译不过
  m["s1"].name = "aa"
  ```
  map 的 value 本身是不可寻址(也就是你不能取&map["key"]), 因为 map 本身是会扩容的, 而且 go 里面的结构体是值拷贝(例子1), 因此
  1. 当你想进行 x = y 的赋值操作时, 你至少得知道 x 的地址, 而 go 的 map 却不允许寻址 value
  2. 当你进行 map 的赋值操作时, 当不存在 key 时会自动进行赋值操作

  ```go
  例子1
  stu1 := Student{"name":"zhangsan"}
  var m map[string]Student
  m["s1"] = stu1
  stu2 := m["s1"]
  // 此时
  &stu1 != &stu2
  ```
2. **关于不能遍历赋值的问题**
  ```go
  type student struct {
	Name string
	Age  int
  }

  func pase_student() {
    m := make(map[string]*student)
    stus := []student{
      {Name: "zhou", Age: 24},
      {Name: "li", Age: 23},
      {Name: "wang", Age: 22},
    }
    for _, stu := range stus {
      m[stu.Name] = &stu
    }
  }
  ```
  此时, map 里的所有 key(zhou, li, wang) 对应的 value 都是{Name: "wang", Age: 22}
  出现这种现象的原因在于赋值时都是取的 stu 的地址, 而在 for 循环中, stu 是会复用的, 即:
  ```go
  for i, _ := range stus {
    // 此时是值赋值
    stu := stus[i]
    // 而 stu 一直是同一个地址值(假设为0x01)
    m[stu.Name] = &stu
  }
  // 执行完, map 里的所有 value 都是 0x01, 指向的就是最后赋值的一个结构体
  ```

# 结构体
## 结构体的比较
1. 结构体能不能使用 `==` 比较取决于结构体的成员能不能比较
  - 可比较：Integer，Floating-point，String，Boolean，Complex(复数型)，Pointer，Channel，Interface，Array
  - 不可比较：Slice，Map，Function
2. 但是即使成员有 slice 等类型, 也可以使用 reflect.DeepEqual 来进行比较

### reflect.DeepEqual
DeepEqual函数用来判断两个值是否深度一致。具体比较规则如下：

1. 不同类型的值永远不会DeepEqual
2. 当两个数组的元素对应DeepEqual时，两个数组DeepEqual
3. 当两个相同结构体的所有字段对应DeepEqual的时候，两个结构体DeepEqual
4. 当两个函数都为nil时，两个函数DeepEqual，==其他情况不相等（相同函数也不相等）==
5. 当两个interface的真实值DeepEqual时，两个interface DeepEqual
6. map的比较需要同时满足以下几个
两个map都为nil或者都不为nil，并且长度要相等相同的map对象或者所有key要对应相同map对应的value也要DeepEqual 
7. 指针，满足以下其一即是深度相等
  - 两个指针满足go的==操作符两个指针指向的值是深度相等的
  - 切片，需要同时满足以下几点才是深度相等
8. 两个切片都为nil或者都不为nil，并且长度要相等两个切片底层数据指向的第一个位置要相同或者底层的元素要深度相等注意：空的切片跟nil切片是不深度相等的
其他类型的值（numbers, bools, strings, channels）如果满足go的==操作符，则是深度相等的。要注意不是所有的值都深度相等于自己，例如函数，以及嵌套包含这些值的结构体，数组等

# 垃圾回收算法
[参考](https://zhuanlan.zhihu.com/p/297177002)

## 1.3之前
标记清除算法, 整个 GC 期间 STW

## 1.5 三色标记法
- 白色: 回收周期开始前, 所有对象都是白色. 回收周期结束后, 将所有标记为白色的对象删除
- 黑色: 已经完成搜索的对象, 其引用已经被扫描
- 灰色: 正在等待搜索的对象

1. 初始时所有对象都是白色对象
2. 从GC Root对象出发，扫描所有可达对象并标记为灰色，放入待处理队列
3. 从队列取出一个灰色对象并标记为黑色，将其引用对象标记为灰色放入队列
4. 重复上一步骤，直到灰色对象队列为空
5. 此时所有剩下的白色对象就是垃圾对象

![](https://segmentfault.com/img/remote/1460000018161591)

### 三色标记法并发时的问题
当黑色对象引用白色对象时可能出现误回收
![](https://pic2.zhimg.com/80/v2-556a9afa216dbf30981f2cfe3ba29999_1440w.jpg)

## 写屏障与删除屏障
解决并发问题有两个思路
1. 防止黑色对象引用白色对象
2. 防止非垃圾的白色对象丢失存在可达关系的灰色对象

由此衍生出两种屏障

- 写屏障: 当黑色对象引用新的对象时, 强制使其下游对象变为灰色(注意: ==只针对堆, 栈还是采用 re-scan==)
- 删除屏障: 当灰色对象删除下游对象时, 强制将其下游对象变为灰色
![删除屏障](https://pic4.zhimg.com/80/v2-58bac6eb968dacdeb15151a6c224286b_1440w.jpg)
(如上图, 删除屏障保护了 C, D 对象)

### 写屏障与删除屏障的缺点

- 插入写屏障：一轮标记结束后需要STW重新扫描栈上对象
- 删除屏障：回收精度低，在垃圾回收开始前使用STW扫描所有GC Root对象形成初始快照，用户程序Mutator从灰色/白色对象中删除白色指针时会将下游对象标记为灰色，相当于保护了所有初始快照中的白色对象不被删除

## 1.8 混合写屏障
- GC开始时将栈上所有对象标记为黑色，无须STW
- GC期间在栈上创建的新对象均标记为黑色
- 将被删除的下游对象标记为灰色
- 将被添加的下游对象标记为灰色

### 逃逸分析
所谓逃逸分析（Escape analysis）是指由编译器决定内存分配的位置，不需要程序员指定。 函数中申请一个新的对象如果分配在栈中，则函数执行结束可自动将内存回收；如果分配在堆中，则函数执行结束可交给GC（垃圾回收）处理.

每当函数中申请新的对象，编译器会跟据该对象是否被函数外部引用来决定是否逃逸：
1. 如果函数外部没有引用，则优先放到栈中；
2. 如果函数外部存在引用，则必定放到堆中.

注意，对于函数外部没有引用的对象，也有可能放到堆中，比如内存过大超过栈的存储能力。

逃逸分析通常有四种情况:
1. 指针逃逸.
2. 栈空间不足逃逸.
3. 动态类型逃逸.
4. 闭包引用对象逃逸.

# GPM
M:N 关系
![](https://pic1.zhimg.com/80/v2-52576a60c97ef993924aa35ce54f5c18_1440w.jpg)

- 协程跟线程是有区别的，线程由CPU调度是抢占式的，**协程由用户态调度是协作式的**，一个协程让出CPU后，才执行下一个协程。

Goroutine 特点
1. 占用内存小(几 Kb)
2. 调度灵活

## 为什么需要 P
![](https://pic1.zhimg.com/80/v2-1b5bcc1d528eab7c89ddc4b0191b4878_1440w.jpg)

M想要执行、放回G都必须访问全局G队列，并且M有多个，即多线程访问同一资源需要加锁进行保证互斥/同步，所以全局G队列是有互斥锁进行保护的。

缺点:
1. 锁竞争激烈
2. 局部性差, G 上创建 G' 与 G 有较强的相关性, 最好在同一个线程 M 执行

## GPM 
[参考](https://zhuanlan.zhihu.com/p/323271088)

![](https://pic1.zhimg.com/80/v2-a66cb6d05fb61e74681410feb3bd1df4_1440w.jpg)

1. 全局队列
2. 本地队列. 数量不超过 256 个. 如果已满, 则将本地队列的一半 G 放到全局队列
3. P 列表. 
4. M. 优先从绑定的 P 的本地队列中获取 G, 本地队列为空, 则从全局队列中拿一批 G 放到 P 的本地队列. 拿不到再从别的 P 的本地队列中偷一半

> P 与 M 的个数
- P 的数量由环境变量决定. 程序的任意时刻只有固定的 P 的数量
- M 的数量
  1. go 语言限制, 最大 10000. 但是内核一般达不到
  2. 可以通过设置 runtime.SetMaxThreads 函数
  3. 当 M 阻塞时, P 会创建/唤醒空闲

> P 与 M 何时创建
- P. 程序一开始就创建
- M. 阻塞时

## 调度器的设计
- 复用线程: 避免频繁的创建, 销毁
  1. 偷取
  2. 阻塞释放
- 利用并行: P 表示同时执行的任务数
- 抢占: 每个 goroutine 最多执行 10 ms, 防止其他 goroutine 饿死
- 全局 G 队列

## go func()
![](https://pic1.zhimg.com/80/v2-08a62309cb2c22fe765c20d2f640e15c_1440w.jpg)

## Go 调度器调度场景
1. P 拥有 G1, M1 获取 P 后开始运行 G1, G1 使用 go func() 创建 G2, 根据局部性原理把 G2 放到 P1 的本地队列
![](https://pic2.zhimg.com/80/v2-a02d5dca1ff3e015848a15aa821ba211_1440w.jpg)

2. G1 运行结束后, M 上运行的 goroutine 切换为 G0(每个 M 特有的, 负责调度). 从 P 的本地队列获取 G2, 从 G0 切换到 G2
![](https://pic4.zhimg.com/80/v2-6c08c8e5842c200c4e582fa9f9a2ed53_1440w.jpg)

3. 假设每个 P 的本地队列只能存储 4 个 G. G2 要创建 6 个 G, 前 3 个 G (G3, G4, G5) 已经加入 P1 的本地队列
![](https://pic4.zhimg.com/80/v2-ed714c33975af4efd210c324522d9aaf_1440w.jpg)

4. G2 在创建 G7 时, 发现 P1 的本地队列已满, 需要执行 **负载均衡**, (把 P1 的本地队列前一半 G, 还有新创建的 G **打乱顺序后**转移到全局队列)
  > 如果由 G2 新创建的 G7 是在 G2 之后就执行, 那么 G7 在具体实现时可以加入本地队列, 而由某个老的 G 加入全局队列

![](https://pic4.zhimg.com/80/v2-f81d058c78f0c93f03c8c3b3714faec7_1440w.jpg)

5. 在创建 G 时, 运行的 G 会尝试唤醒其他空闲的 P 和 M 组合去执行
![](https://pic1.zhimg.com/80/v2-1970994c97f32af214213aa9a5556db8_1440w.jpg)

6. M2 尝试从全局队列(GQ)中取一批 G 放到 P2 的本地队列. M2 从全局队列取的 G 数量
```c
n = min(len(GQ/2), len(GQ)/numsP + 1)
n = max(n, 1)
```
![](https://pic2.zhimg.com/80/v2-1cb901227eabb3b6152d71f919024b01_1440w.jpg)

7. 若全局队列已空, 则 M2 则从其他 P 的本地队列的队尾取一半 G 放到 P2 的本地队列
![](https://pic3.zhimg.com/80/v2-e3b4b24c420df0806ce279fbb89d7ff2_1440w.jpg)

8. 系统最多运行 GOMAXPROCS 个自旋的线程, 多余的将休眠
![](https://pic3.zhimg.com/80/v2-68bb29077e4ced8dfbca60e03f029006_1440w.jpg)

9. G9 创建了 G9, G8 进行了系统调用, M2 和 P2 解绑, P2 会执行以下判断: 如果 P2 本地队列有 G, 全局队列有 G 或有空闲的 M, P2 会唤醒一个 M 与他绑定. 否则 P2 会加入到空闲 P 列表, 等待 M 来获取可用的 P
![](https://pic3.zhimg.com/80/v2-1c5c20356a77ebddb9632232254dd722_1440w.jpg)

10. G8 创建了 G9, 加入 G8 进行非阻塞的系统调用. M2 和 P2 会解绑, 但 M2 会记住 P2. 当 G8 与 M2 退出系统调用, 会尝试获取 P2, 如果无法获取, 则获取空闲的 P, 如果没有, G8 会被标记为可运行状态, 并加入全局队列, M2 因为没有 P 的绑定而变成休眠状态 (长时间休眠等待 GC 回收)
![](https://pic1.zhimg.com/80/v2-b4454f0efda1d528e5fbbf584e0f5f90_1440w.jpg)

# channel
[参考](https://zhuanlan.zhihu.com/p/27917262)

channel 在底层是一个 hchan 结构体. 
```go
type hchan struct {
    qcount   uint           // total data in the queue
    dataqsiz uint           // size of the circular queue
    buf      unsafe.Pointer // points to an array of dataqsiz elements
    elemsize uint16
    closed   uint32
    elemtype *_type // element type
    sendx    uint   // send index
    recvx    uint   // receive index
    recvq    waitq  // list of recv waiters
    sendq    waitq  // list of send waiters

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex
}
```

chan 本身就是指针, 且必须初始化, 因此需要 make

## 发送与接收
主要涉及 hchan 的四个成员变量
![](https://pic1.zhimg.com/v2-c2549285cd3bbfd1fcb9a131d8a6c40c_b.webp)

G1 为发送者, G2 为接收者, chan 为缓冲区为3, 初始时, sendx 与 recvx 都为 0, 当 G1 往 ch 发送数据时, 首先对 buf 加锁, 然后将数据 copy 到 buf 里, 增加 sendx 的值, 然后释放锁

G2 消费时便反操作.

整个过程, G1 和 G2 并没有共享内存, 而是通过通信 copy 的方式来实现共享内存

## 阻塞与恢复
### 发送阻塞
当 G1 向 buf 已经满了的 ch 发送数据时, 当 runtime 检测到对应的 hchan 的 buf 已满, 会通知对应的 P, P 会将 G1 的状态设置为 waiting, 移除与线程 M 的联系, 然后从 P 的本地队列中选择一个 G 在线程 M 中执行, 此时 G1 是阻塞状态, 但并不是操作系统的线程阻塞, 所以这个时候只用少量资源

### 恢复
当 G1 变为 waiting 状态后, 会创建一个 sudog 结构体, 放到对应的 chan 的 sendq 这个 list 中. sudog 保存了对应的 goroutine 以及待传输的变量的地址值

### 接收阻塞
接收者 G2 创建 sudog 后放入对应的 chan 的 recvq list 中. 
当 G1 发送数据时, 并不需要加锁复制到 buf 中, 而是直接复制到 G2 sudog 结构对应的 elem 的地址中
![](https://pic3.zhimg.com/80/v2-4466a9880e997d27357b778583a7e166_1440w.png)

# context
[参考](https://zhuanlan.zhihu.com/p/68792989)

## 为什么要有 context (作用)
在 go 的 server 里，通常每来一个请求都会启动若干个 goroutine 同时工作：有些去数据库拿数据，有些调用下游接口获取相关数据

![](https://pic2.zhimg.com/80/v2-f028da2b74c2ed08718d206c74cc6a9d_1440w.jpg)

这些 goroutine 需要共享这个请求的基本数据，例如登陆的 token，处理请求的最大超时时间（如果超过此值再返回数据，请求方因为超时接收不到）等等。当请求被取消或是处理时间太长，这有可能是使用者关闭了浏览器或是已经超过了请求方规定的超时时间，请求方直接放弃了这次请求结果。这时，所有正在为这个请求工作的 goroutine 需要快速退出，因为它们的“工作成果”不再被需要了。在相关联的 goroutine 都退出后，系统就可以回收相关的资源。

在Go 里，我们不能直接杀死协程，协程的关闭一般会用 channel+select 方式来控制。但是在某些场景下，例如处理一个请求衍生了很多协程，这些协程之间是相互关联的：需要共享一些全局变量、有共同的 deadline 等，而且可以同时被关闭。再用 channel+select 就会比较麻烦，这时就可以通过 context 来实现。

==一句话：context 用来解决 goroutine 之间退出通知、元数据传递的功能。==

## 底层原理
![](https://pic3.zhimg.com/80/v2-6a27526f536505cea08a5813ccce05b2_1440w.jpg)

### context
```go
type Context interface {
    // 当 context 被取消或者到了 deadline，返回一个被关闭的 channel
    Done() <-chan struct{}

    // 在 channel Done 关闭后，返回 context 取消原因
    Err() error

    // 返回 context 是否会被取消以及自动取消时间（即 deadline）
    Deadline() (deadline time.Time, ok bool)

    // 获取 key 对应的 value
    Value(key interface{}) interface{}
}
```

Context 是一个接口，定义了 4 个方法，它们都是幂等的。也就是说连续多次调用同一个方法，得到的结果都是相同的。

Done() 返回一个 channel，可以表示 context 被取消的信号：当这个 channel 被关闭时，说明 context 被取消了。注意，这是一个只读的channel。 我们又知道，读一个关闭的 channel 会读出相应类型的零值。并且源码里没有地方会向这个 channel 里面塞入值。换句话说，这是一个 receive-only 的 channel。因此在子协程里读这个 channel，除非被关闭，否则读不出来任何东西。也正是利用了这一点，子协程从 channel 里读出了值（零值）后，就可以做一些收尾工作，尽快退出。
```go
select {
  // 一旦ctx.done()有值, 便说明chan已关闭
	case <-ctx.Done():
		fmt.Println("main", ctx.Err())
  }
}
```

Deadline() 返回 context 的截止时间，通过此时间，函数就可以决定是否进行接下来的操作，如果时间太短，就可以不往下做了，否则浪费系统资源。当然，也可以用这个 deadline 来设置一个 I/O 操作的超时时间。

Value() 获取之前设置的 key 对应的 value。

### canceler
```go
type canceler interface {
    cancel(removeFromParent bool, err error)
    Done() <-chan struct{}
}
```

上面定义的两个方法的 Context，就表明该 Context 是可取消的

### emptyCtx

```go
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
    return
}

func (*emptyCtx) Done() <-chan struct{} {
    return nil
}

func (*emptyCtx) Err() error {
    return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
    return nil
}
```

emptyCtx 为 context 的空实现, 永远不会被 cancel，没有存储值，也没有 deadline

```go
var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)
```

background 通常用在 main 函数中，作为所有 context 的根节点。

todo 通常用在并不知道传递什么 context的情形。例如，调用一个需要传递 context 参数的函数，你手头并没有其他 context 可以传递，这时就可以传递 todo。这常常发生在重构进行中，给一些函数添加了一个 Context 参数，但不知道要传什么，就用 todo “占个位子”，最终要换成其他 context。

### cancelCtx
```go
type cancelCtx struct {
    Context

    // 保护之后的字段
    mu       sync.Mutex
    done     chan struct{}
    children map[canceler]struct{}
    err      error
}
```

这是一个可以取消的 Context，实现了 canceler 接口。它直接将接口 Context 作为它的一个匿名字段，这样，它就可以被看成一个 Context。

### timerCtx
```go
type timerCtx struct {
    cancelCtx
    timer *time.Timer // Under cancelCtx.mu.

    deadline time.Time
}
```

timerCtx 基于 cancelCtx，只是多了一个 time.Timer 和一个 deadline。Timer 会在 deadline 到来时，自动取消 context。

### valueCtx
```go
type valueCtx struct {
    Context
    key, val interface{}
}
```

创建 valueCtx 的函数
```go
func WithValue(parent Context, key, val interface{}) Context {
    if key == nil {
        panic("nil key")
    }
    if !reflect.TypeOf(key).Comparable() {
        panic("key is not comparable")
    }
    return &valueCtx{parent, key, val}
}
```

对 key 的要求是可比较，因为之后需要通过 key 取出 context 中的值，可比较是必须的。

通过层层传递 context，最终形成这样一棵树：
![](https://pic1.zhimg.com/80/v2-b0dbe9549219fca2c96f1699fb61fbc0_1440w.jpg)

和链表有点像，只是它的方向相反：Context 指向它的父节点，链表则指向下一个节点。通过 WithValue 函数，可以创建层层的 valueCtx，存储 goroutine 间可以共享的变量。

取值的过程，实际上是一个递归查找的过程：
```go
func (c *valueCtx) Value(key interface{}) interface{} {
    if c.key == key {
        return c.val
    }
    return c.Context.Value(key)
}
```

它会顺着链路一直往上找，比较当前节点的 key 是否是要找的 key，如果是，则直接返回 value。否则，一直顺着 context 往前，最终找到根节点（一般是 emptyCtx），直接返回一个 nil。所以用 Value 方法的时候要判断结果是否为 nil。

因为查找方向是往上走的，所以，父节点没法获取子节点存储的值，子节点却可以获取父节点的值。