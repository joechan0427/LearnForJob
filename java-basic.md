# 一. 数据类型
## 基本类型
- byte (8 bits)
- char (16 bits)
- short (16 bits)
- int (32 bits)
- float (32 bits)
- long (64 bits)
- double (64 bits)
- bool (1 bit in theory)
    bool 理论上只需要一个位来表示 true 和 false 两种状态，JVM 在编译期间使用 int 来表示，而 bool 数组则是用 byte 数组

## 包装类型

## 缓存池

### new Integer(123) 和 Integer.valueOf(123) 的区别
两者都会得到一个值为 123 的 Integer 对象。区别就是 new 得到的对象是堆上新生成的, 而 valueOf 得到的是缓存池里的 (-128 ~ 127)
`valueOf源码`
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。

==字面量赋值会使用缓存池==

```java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```

基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values (0 ~ 256) (8 bits)
- short values between (-128 ~ 127) (8 bits)
- int values between (-128 ~ 127) (8 bits)
- char in the range (\u0000 ~ \u007F) (8 bits)

在 jdk 1.8 所有的数值类缓冲池中，`Integer` 的缓冲池 `IntegerCache` 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 `-XX:AutoBoxCacheMax=<size>` 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 `java.lang.IntegerCache.high` 系统属性，然后 `IntegerCache` 初始化的时候就会读取该系统属性来决定上界。

# 二. String

## String 不可变
String 声明为 final, 因此不能被继承 (Integer 等包装类也是)

jdk1.8 中, String 使用 char 数组存储

jdk9 之后, 使用 byte 数组, 同时使用 coder 标识编码

value 数组被声明成 final
, 意味着此数组不能指向别的引用, 同时 String 内部没有可以改变此数组内数据的方法,因此可保证 String 不可变

## String 不可变的好处

1. 可以缓存 HashCode
2. String 缓存池需要
3. 安全性
String 作为参数时, 保证参数的不可变
4. 线程安全

## String, StringBuilder, StringBuffer

1. 可变性
String 不可变, 其他两者可变

2. 线程安全
StringBuilder 非线程安全, 其他两者线程安全 (StringBuffer 通过使用 Synchronized 加锁实现)

## String Pool
字符串常量池保存着所有的 String 字面量, 在编译时期就确定了. 此外, 还可以通过 String 的 intern() 方法将该字符串存入常量池

intern() 方法首先在 常量池里寻找是否已存在同样的字符串(通过调用 equals() 方法), 如果找到,则返回该引用, 否则,先将该字符串存入常量池, 再返回引用


==常量池位置==
jdk1.7 之前在运行时常量池, 属于永久代, jdk1.7 移到堆里. 因为永久代空间有限, 大量字符串可能造成 OOM

## new String("123")
同样, new 出来的 String 一定在堆里创建新的对象, 而字面量赋值 (String str = "123") 则使用常量池

因此 new String("123") 将创建一个或两个对象

同样, new 出来的 String 一定在堆里创建新的对象, 而字面量赋值 (String str = "123") 则使用常量池

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

# 三. 运算

## 参数传递 

本质都是值传递, 基本类型复制值传入, 引用类型复制地址值传入

## float 与 double

java 不能隐式向下转型
字面量 12.13 是double类型, 不能直接赋值给 float, 12,13f 才是 float 类型

==例外==
```java
short i= 0;

// 实际做了 i = (short)(i+1)
i++;
```

## switch
从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象。

```java
String s = "a";
switch (s) {
    case "a":
        System.out.println("aaa");
        break;
    case "b":
        System.out.println("bbb");
        break;
}
```
switch 不支持 long、float、double，是因为 switch 的设计初衷是对那些只有少数几个值的类型进行等值判断，如果值过于复杂，那么还是用 if 比较合适。

# 四. 关键字

## final

1. 数据
    - 基本类型: 数值不能改变
    - 引用类型: 指针不能改变, 对象自身可以改变
2. 方法
    方法不能被重写
    private 方法隐式声明了 final, 此时子类可以重新定义一个同名的方法. 
3. 类
    此类不能被继承, 如 String

## static

1. 静态变量 
    此变量属于类, 所有实例共享, 只存在一份

2. 静态方法
    静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。
    静态方法只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因此这两个关键字与具体对象关联。
    静态方法不存在重写的概念(因为不存在调用父类的方法)

3. 静态语句块
    静态语句块在类初始化时运行一次。
    ```java
    static {

    }
    ```

4. 静态内部类
    非静态内部类依赖于外部类的实例, 即需要先创建外部类, 才能创建非静态内部类, 而静态内部类则不需要. 典型静态内部类 Arrays 工具类下的 ArrayList (设计模式, 适配器模型, 将数组转为 list 类)
    静态内部类不能访问外部类的非静态的变量和方法。

### 初始化顺序

静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在**代码中的顺序。**
```java
// 1
static int i = 0;
static {
    // 2
    i ++;
    // 访问不到 j
}
// 3
static int j = 1;
```
最后才是构造函数的初始化。

存在继承的情况下
父类的静态语句块, 静态变量 (一次)
子类的静态语句块, 静态变量 (一次)

父类的普通语句块 (多次)
父类的构造函数 (多次)
子类的普通语句块 (多次)
子类的构造函数 (多次)

# 五. Object

```java
hashCode
equals
wait (3个)
notify
notifyAll
clone
toString
```

## equals
对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false

与 == 相比, == 判断引用, 未重写的 equals 方法与 == 返回相同

## hashCode
重写 equals 方法一定要重写 hashCode 方法, 来保证相等的对象有一样的 hashCode

## clone()
clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

应该注意的是，clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

### clone() 的替代方案
使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。


```java
// 拷贝构造函数
public class CloneConstructorExample {

    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
```

# 六. 继承
## 访问权限

java 有三个访问权限修饰符, private, protected, public. 未加修饰符表示包可见

可以给 成员变量 和 成员方法 加上访问权限修饰符
- 类可见(用 public 修饰)表示其它类可以用这个类创建实例对象。
- 成员(包括变量和方法)可见表示其它类可以用这个类的实例对象访问到该成员；

protected 用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义(不加public就是包可见)。

如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例去代替，也就是确保满足里氏替换原则。

## 抽象类与接口

### 1. 抽象类

抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。

抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承。(但可以有默认实现)

### 2. 接口

接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。

从 Java 8 开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，让它们都实现新增的方法。

接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected。从 Java 9 开始，允许将方法定义为 private，这样就能定义某些复用的代码又不会把方法暴露出去。

==接口的字段默认都是 static 和 final 的。==

## super

- 访问父类的构造函数：可以使用 super() 函数访问父类的构造函数，从而委托父类完成一些初始化的工作。应该注意到，==子类一定会调用父类的构造函数来完成初始化工作==，一般是调用父类的默认构造函数，如果子类需要调用父类其它构造函数，那么就可以使用 super() 函数。
- 访问父类的成员方法：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

## 重写与重载
### 1. 重写 (Override)

存在于继承体系中，指子类实现了一个与父类(接口)在方法声明上完全相同的一个方法。

为了满足里式替换原则，重写有以下三个限制：
1. 访问权限大于等于父类
2. 返回类型是父类返回类型的子类
3. 抛出的异常是父类抛出异常的子类

在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有再到父类中查看，看是否从父类继承来。否则就要对参数进行转型，转成父类之后看是否有对应的方法。总的来说，方法调用的优先级为：
1. this.func(this)
2. super.func(this)
3. this.func(super)
4. super.func(super)

```java
/*
    A
    |
    B
    |
    C
    |
    D
 */


class A {

    public void show(A obj) {
        System.out.println("A.show(A)");
    }

    public void show(C obj) {
        System.out.println("A.show(C)");
    }
}

class B extends A {

    @Override
    public void show(A obj) {
        System.out.println("B.show(A)");
    }
}

class C extends B {
}

class D extends C {
}
```

```java
public static void main(String[] args) {

    A a = new A();
    B b = new B();
    C c = new C();
    D d = new D();

    // 在 A 中存在 show(A obj)，直接调用
    a.show(a); // A.show(A)
    // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
    a.show(b); // A.show(A)
    // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
    b.show(c); // A.show(C)
    // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
    b.show(d); // A.show(C)

    // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
    A ba = new B();
    ba.show(c); // A.show(C)
    ba.show(d); // A.show(C)
}
```

### 2. 重载 (Overload)

存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。

应该注意的是，返回值不同，其它都相同不算是重载。

# 七. 反射
每个类都有一个 Class 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。

类加载相当于 Class 对象的加载，类在第一次使用时才动态加载到 JVM 中。也可以使用 Class.forName("com.mysql.jdbc.Driver") 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- Field ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- Method ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- Constructor ：可以用 Constructor 的 newInstance() 创建新的对象。

# 八、异常

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： Error 和 Exception。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

- 受检异常 ：需要用 try...catch... 语句捕获并进行处理(或抛出)，并且可以从异常中恢复；
- 非受检异常 ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。

![](https://camo.githubusercontent.com/8b2dfb0bfaeaf7ea0f0ae387ce2cbb0da1ec8d88a23929623cafc6bcbff044af/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f50506a77502e706e67)

