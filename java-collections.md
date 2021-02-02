# 一. 概览
容器主要包括 Collection 和 Map 两种 (都是接口)

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/source-code/dubbo/java-collection-hierarchy.png)

## Collection

![](https://camo.githubusercontent.com/c0506ba8f5134d89ed6a398e1c165865d50c68f6c7af4e01b75248a95e0da37d/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383232303934383038342e706e67)

### 1. Set
- TreeSet: 基于红黑树, 有序, 例如 ceil(), floor()
- HashSet
- LinkedHashSet: HashSet 加 双向链表

### 2. List
- ArrayList
- Vector: 线程安全, 对整个实例对象加锁, 增删改查全部对实例(synchronized 在方法上)加锁, 不推荐使用
- LinkedList: 双向链表

### 3. Queue
- LinkedList
- PriorityQueue

## Map
![](https://camo.githubusercontent.com/ce6470fc8cfd0f0c74ba53bd16ee9467b21c5ca7fc0566413df4701342a96a15/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303230313130313233343333353833372e706e67)

- TreeMap
- HashMap
- HashTable: 线程安全, 锁住整个实例对象(get 和 put 都是)
- LinkedHashMap

# 二. 源码分析
基于 jdk1.8

## ArrayList
1. 实现 RandomAccess 接口, 标志其可以随机访问
默认无参构造的数组长度为10, 但==采用懒汉式==, 即开始初始化一个空数组, 等到第一个元素添加时再分配一个容量为 10 的数组. 带参数的两种构造方式在参数为 0 或传入集合的长度为 0 时也是如此操作

2. 扩容
添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 oldCapacity + (oldCapacity >> 1)，即 oldCapacity+oldCapacity/2。
扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价(O(n)的时间复杂度)很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

3. 删除元素
需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的平均时间复杂度为 O(N)，删除最后一个为O(1), 可以看到 ArrayList 删除元素的代价是非常高的。

4. 序列化
ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。
保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。
ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。
序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为字节流并输出。而 writeObject() 方法在==传入的对象(ArrayList)存在 writeObject() 的时候会去反射调用该对象的 writeObject() 来实现序列化==。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。
5. Fail-Fast
modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。
在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。代码参考上节序列化中的 writeObject() 方法。
```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

## LinkedList
1. 概览
基于双向链表实现
每个链表存储了 first 和 last 指针：
    ```java
    transient Node<E> first;
    transient Node<E> last;
    ```
    为什么 transient ?
因为 Node 本身还带有前序和后续节点的指针信息, 而这并不是我们需要的, 甚至可能在反序列化后导致出错. HashMap也是如此(因为不同jvm对同一对象的hashcode的实现不同), 因此他们都重写 readObject 和 writeObject.

## HashMap

### 1. 存储结构
HashMap 使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的 Entry。
![](https://camo.githubusercontent.com/363d11bc36ca5b717ea62374b76ea2a47c287c15b072fd076ab75388abb202a5/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383233343934383230352e706e67)

### 2. 拉链法
1.7之前使用头插法, 即后来的相同hash值的不同对象插在链表头部, 这样在多并发时会产生问题(多个线程同时rehash的时候会导致循环链表), 1.8改成尾插法, 不过并发环境仍然不推荐HashMap, 同时put可能覆盖

### 3. put
HashMap 允许插入键为 null 的键值对。但是因为无法调用 null 的 hashCode() 方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。HashMap 使用第 0 个桶存放键为 null 的键值对。

1. 确定下标. key的hashCode 与 高16位做异或操作(扰动)
2. 取余. 使用hash值与数组长度-1 做 & 操作. 前提是数组长度是2的n次方

### 4. 扩容
假设总的键值有N个, Node数组长度为M, 则平均的链表长度为 N/M.
因此为了减小查找成本 O(N/M), 应使 M 尽可能的长
与扩容有关的长度为

|参数 | 含义|
| -| -|
|capacity|数组长度大小,必须保证为 2 的 N 次方
|size| key的数量|
|threshold| size 的临界值, 当 size 大于 threshold 则要扩容
|loadFactor| 装载因子 threshold = capacity*loadFactor|

### 5. 扩容-重新计算桶下标
由于每次扩容都是当前数组长度乘2, 可以很简单地使用位运算 (length << 1) 左移一位实现
此时, 要再次确定已有的key值的桶下标. 
可以很简单地利用已计算出来的hash & old capacity 得到, 此时得到的值有两种:
- 0 : 原位置
- 1 : 原位置+old capacity

### 6. 链表转红黑树
1.8开始, 当数组长度 capacity 大于 64 且链表长度大于 8 时, 会将链表转为红黑树

## ConcurrentHashMap
### 1. 存储结构
**jdk1.7**
![](https://camo.githubusercontent.com/7a6d1a946245fd7215a329f41ce9b182c468a3a974e0b5b39eba937d76e334c8/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230393030313033383032342e706e67)
采用分段锁, 多个线程可以同时访问不同分段锁上的桶, ==并发度就是桶的个数==, 默认为16, 最大为65536, 且一旦确定不能修改

**jdk1.8**
![](https://snailclimb.gitee.io/javaguide/docs/java/collection/images/java8_concurrenthashmap.png)
Node 数组 + 链表 / 红黑树。当冲突链表达到一定长度时，链表会转换成红黑树。当冲突下降又会退化成链表

### 2. put
**jdk1.7**
1. 根据 hash 获取segment 的位置. (hash的高 n 位与 mask 做 & 运算. 如 segment 个数为16, 则取 hash 的高4位与 1111 做 & 运算)
2. 如该 segment 为空, 则初始化(其实就是单例模式):
    1. 检查 segment 是否为 null
    2. 如果为空, 以 segment[0] 为模板, 得到loadFactor, threshold, capacity
    3. 自旋判断是否为空, CAS 将新建的segment赋值给对应的地方
3. segment.put

**jdk1.8**
1. 根据 key 计算出 hashcode 。
2. 判断是否需要进行初始化。
3. 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
4. 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容。
5. 如果都不满足(当前位置有数据, 不管是链表(尾插法)还是红黑树)，则利用 synchronized 锁写入数据。
6. 如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树。

# 三. 容器中的设计模式
## 适配器模式
java.util.Arrays.asList() 函数可以将数组转为 List
其实质是在 Arrays 类内部有一个静态内部类 ArrayList, 其并不真正保存数据, 而是保存指向数组的指针, 所以该内部类不能实现 add, remove 等方法. 并且注意不能传入基本类型数组
```java
int ints = {1,2};
Arrays.asList(ints).size(); // 1 
```
## 迭代器模式
![](https://camo.githubusercontent.com/e44932dbf2dc015828f878428f58df99b9e808a6fb5cc76ab65b8baab7c0d2bb/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f696d6167652d32303139313230383232353330313937332e706e67)

Collection 接口继承了 Iterable 接口, 其中的iterator() 方法可以产生一个 Iterator 对象, 通过这个对象可以遍历元素

