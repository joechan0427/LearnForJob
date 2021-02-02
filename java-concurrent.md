# 使用线程
- 继承 Thread 类
- 实现 Runnable 接口
- 实现 Callable 接口

实现两个接口的类智能当作一个可在线程中执行的任务, 不是真正意义上的线程, 最后还是得通过 Thread 类来调用
## 实现 Callable 接口
与 Runnable 相比, Callable 可以有返回值, 返回值通过 FutureTask 进行封装

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

## 继承 Thread 类
同样也是需要实现 run 方法, 因为 Thread 类也实现了 Runnable 接口

## 实现接口 Vs 继承 Thread
实现接口更好一点
- Java 不允许多重继承, 因此继承 Thread 类无法继承其他类, 而可以实现多个接口
- 继承 Thread 类可能开销过大

# 中断
一个线程执行完毕会自动退出, 如果执行过程发生异常也会提前结束

## InterruptedException
调用一个类的 interrupt() 方法来中断线程执行, 如果线程处于 blocked, timed_waiting, waiting 的状态, 就会抛出该异常, 中断执行. 但不能中断 i/o 阻塞或 synchronized 锁阻塞

# 互斥同步 (加锁)
- synchronized (jvm 实现)
- ReentrantLock (jdk 实现)

## synchronized
在 Java 早期版本中，synchronized 属于 重量级锁，效率低下。
因为监控器锁 (monitor) 依赖底层操作系统 mutex Lock 实现, Java 的线程映射到操作系统的原生线程上, 因此挂起和唤醒都需要操作系统实现, 而操作系统的线程切换需要从用户态转为内核态, 这个转换的时间成本较高
==注意: 对象的wait(), notify() 等方法需要在同步代码块里才能调用, 否则会抛异常==

### 使用
1. 修饰实例方法 (锁住该对象)
2. 修饰静态方法 (锁住类)
==注意: 允许两个线程, 在同一时间, 一个访问类的同步方法, 一个访问实例的同步方法, 因为这两个方法锁住的对象不同==
3. 修饰代码块 (锁住括号里面的对象)
==注意: 尽量不要锁住 String, Integer 这类有缓存池的对象==

### 单例模式
一个类只有一个实例对象, 如 Spring 中的 Service, Controller
```java
class Singleton {
    private static volatile Singleton single;

    public static Singleton getSingle() {
        if (single == null) {
            synchronized (Singleton.class) {
                if (single == null) {
                    single = new Singleton();
                }
            }
        }
        return single;
    }
}
```

volatile 修饰的目的:
1. 保证可见性, 保证每个线程得到的都是当前内存中的状态
2. 保证有序性, 确保 jvm 不会指令重排, 以保证得到的 single 对象一定已经创建好了

### Java 对象头
对象头包含三部分
1. markword
存储 hashcode, gc分代年龄, 锁标志, 偏向线程
2. 类型指针
指向对象的类元数据, jvm 通过此指针确定是哪个类的实例
3. 数组特有的储存长度

对象头
 长度 | 内容 | 说明
 - | - | - 
 一个字(32 bits/ 64bits) | markword | 存储hashcode等信息
 同上 | Class metadata Addr | 存储对象类型指针
 同上 | array length | 数组长度

### synchronized 锁分类
级别低到高依次是
1. 无锁
2. 偏向锁
3. 轻量锁 (自旋)
4. 重量锁

锁只能升级, 不能降级

各状态 markword

**无锁**
25 bits | 4 bits | 1 bit (是否偏向) | 2 bits
 - | - | - | -
hashcode | gc 分代年龄 | 0 | 01

**偏向锁**
23 bits | 2 bits | 4 bits | 1 bit (是否偏向) | 2 bits
 - | - | - | - | -
线程id | epoch | gc 分代年龄 | 1 | 01

**轻量锁**
30 bits  | 2 bits
- | -
指向==线程栈中的锁记录== | 00

**重量锁**
30 bits  | 2 bits
- | -
指向==monitor== | 10

### 加锁过程

![](https://upload-images.jianshu.io/upload_images/4491294-e3bcefb2bacea224.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)
![](./java-imgs/synchronized.jpg)

1. 判断锁对象的 markword 最后两位, 来判断锁对象此时有没有上锁
2. 接着判断偏向锁标志位是否为 1 , 如果不是, 则进行 CAS 竞争轻量锁, 即到 5
3. (偏向锁) 此时判断 markword 的 threadId 是否为本线程, 如果是, 则添加一条 lock record, 表示重入次数. 如果否, 则判断锁对象的 epoch 和该类的 cEpoch 是否相同
    1. 如果 epoch < cEpoch, 说明发生过批量重偏向, 直接CAS加偏向锁
    2. 如果 epoch = cEpoch, 竞争锁
4. 竞争锁得等到全局安全点(safe point，代表了一个状态，在该状态下所有线程都是暂停的), 此时有两个判断
    1. 持有偏向锁的对象的线程已退出同步代码块, 或者持有锁对象的线程已不存活, 则撤销偏向锁, 恢复到无锁状态(且不能偏向, 最后三位 001)
    2. 持有锁对象的线程仍存活且在同步代码块内, 则锁升级为轻量锁, 且该线程仍持有锁
5. (轻量锁) 先在线程的栈帧里赋值当前锁对象的markword, 然后 CAS 替换, 成功则获取到锁, 失败则首先自旋, 当自旋次数到达一定次数进入锁膨胀
6. (重量锁) 此时锁对象的markword将指向monitor对象, 获取不到锁的线程将进入 EntrySet 集合中, 并处于 blocked (阻塞)状态
7. (轻量锁的释放) 获得锁对象的线程进行 CAS 将自己拥有的 Markword 替换回此时锁对象的 markword, 如果成功, 则直接退出. 如果失败, 说明此时已升级为重量锁, 唤醒其他线程进行新一轮竞争

特别说明:
1. CAS记录owner时，expected == null，newValue == ownerThreadId，因此，只有第一个申请偏向锁的线程能够返回成功，后续线程都必然失败
2. 当一个对象已经计算过identity hash code，它就无法进入偏向锁状态
3. 当一个对象当前正处于偏向锁状态，并且需要计算其identity hash code的话，则它的偏向锁会被撤销，并且锁会膨胀为重量锁
4. 重量锁的实现中，ObjectMonitor类里有字段可以记录非加锁状态下的mark word，其中可以存储identity hash code的值。或者简单说就是重量锁可以存下identity hash code。
    这里讨论的hash code都只针对identity hash code。用户自定义的hashCode()方法所返回的值跟这里讨论的不是一回事。Identity hash code是未被覆写的 java.lang.Object.hashCode() 或者 java.lang.System.identityHashCode(Object) 所返回的值。
5. 重量锁还有一个 waitSet 集合, 用于线程调用锁对象的 wait() 方法时, 将线程加入此集合中(此时会释放锁), 等待其他线程调用 notify() 方法将其唤醒