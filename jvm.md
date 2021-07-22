# jvm

[toc]

---

## 类加载子系统

### 1.类加载子系统作用

- 类加载子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识；
- ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定
- 加载的类信息存放于一块称为方法区的内存空间。除了类信息之外，方法区还会存放运行时常量池信息，可能还包括字符串字面量和数字常量[^方法区存放内容]（这部分常量信息是Class文件中常量池部分的内存映射）
[^方法区存放内容]: jdk1.7之前

#### 1.1类加载器ClassLoader角色

![classloader](https://user-gold-cdn.xitu.io/2020/3/18/170ec7d7217f0c1c?imageslim)

#### 1.2加载

1. 通过一个类的全限定名获取定义此类的二进制字节流；
2. 将这个字节流所代表的的静态存储结构转化为方法区的运行时数据；
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

#### 1.3 链接

![链接过程](https://user-gold-cdn.xitu.io/2020/3/18/170ec7daeca85a52?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 1.3.1 验证

- 目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全。
- 主要包括四种验证，文件格式验证，源数据验证，字节码验证，符号引用验证。

##### 1.3.2 准备

- 为类变量(静态变量)分配内存并且设置该类变量的默认初始值，即零值；
- 这里不包含用final修饰的常量，因为final在编译的时候就会分配了，准备阶段会显式初始化；
- 这里不会为实例变量分配初始化，类变量(静态变量)会分配在方法区中，而实例变量是会随着对象一起分配到java堆中。

##### 1.3.3 解析

- 将常量池内的符号引用转换为直接引用的过程。
- 事实上，解析操作会伴随着jvm在执行完初始化之后再执行
- 符号引用就是一组符号来描述所引用的目标。符号应用的字面量形式明确定义在《java虚拟机规范》的class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄
- 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的`CONSTANT_Class_info`/`CONSTANT_Fieldref_info`、`CONSTANT_Methodref_info`等。

#### 1.4初始化

- 初始化阶段就是执行类构造器方法`clinit()`的过程。
- 此方法不需要定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。**我们注意到如果没有静态变量c，那么字节码文件中就不会有clinit方法**

    ![初始化1](https://user-gold-cdn.xitu.io/2020/3/18/170ec7ee5e625699?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
    ![初始化2](https://user-gold-cdn.xitu.io/2020/3/18/170ec7f508f951e6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 构造器方法中指令按语句在源文件中出现的顺序执行
    ![初始化3](https://user-gold-cdn.xitu.io/2020/3/18/170ec7fc6f50f714?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- clinit()不同于类的构造器。（关联：构造器是虚拟机视角下的init()）
- 若该类具有父类，jvm会保证子类的clinit()执行前，父类的clinit()已经执行完毕
    ![初始化4](https://user-gold-cdn.xitu.io/2020/3/18/170ec833466b3af3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 虚拟机必须保证一个类的clinit()方法在多线程下被同步加锁。
    ![初始化5](https://user-gold-cdn.xitu.io/2020/3/18/170ec8415e75466b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 2.类加载器分类

- JVM支持两种类型的加载器，分别为引导类加载器（BootStrap ClassLoader）和自定义类加载器（User-Defined ClassLoader）
- 从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是java虚拟机规范却没有这么定义，而是将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器。
- 无论类加载器的类型如何划分，在程序中我们最常见的类加载器始终只有三个，如下所示：
    ![类加载器](https://user-gold-cdn.xitu.io/2020/3/18/170ec88cffd157f0?imageslim)

#### 2.1 自定义类与核心类库的加载器

- 对于用户自定义类来说：使用系统类加载器AppClassLoader进行加载
- java核心类库都是使用引导类加载器BootStrapClassLoader加载的

```java
/**
 * ClassLoader加载
 */
public class ClassLoaderTest {
    public static void main(String[] args) {
        //获取系统类加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

        //获取其上层  扩展类加载器
        ClassLoader extClassLoader = systemClassLoader.getParent();
        System.out.println(extClassLoader);//sun.misc.Launcher$ExtClassLoader@610455d6

        //获取其上层 获取不到引导类加载器
        ClassLoader bootStrapClassLoader = extClassLoader.getParent();
        System.out.println(bootStrapClassLoader);//null

        //对于用户自定义类来说：使用系统类加载器进行加载
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

        //String 类使用引导类加载器进行加载的  -->java核心类库都是使用引导类加载器加载的
        ClassLoader classLoader1 = String.class.getClassLoader();
        System.out.println(classLoader1);//null

    }
}
```

#### 2.2 虚拟机自带的加载器

1. 启动类加载器（引导类加载器，BootStrap ClassLoader）

    - 这个类加载使用C/C++语言实现的，嵌套在JVM内部
    - 它用来加载java的核心库（JAVA_HOME/jre/lib/rt.jar/resources.jar或sun.boot.class.path路径下的内容），用于提供JVM自身需要的类
    - 并不继承自java.lang.ClassLoader,没有父加载器
    - 加载拓展类和应用程序类加载器，并指定为他们的父加载器
    - 处于安全考虑，BootStrap启动类加载器只加载包名为java、javax、sun等开头的类

2. 拓展类加载器（Extension ClassLoader）
    - java语言编写 ，由sun.misc.Launcher$ExtClassLoader实现。
    - 派生于ClassLoader类
    - 父类加载器为启动类加载器
    - 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录（扩展目录）下加载类库。**如果用户创建的JAR放在此目录下，也会由拓展类加载器自动加载**

3. 应用程序类加载器（系统类加载器，AppClassLoader）
    - java语言编写， 由sun.misc.Launcher$AppClassLoader实现。
    - 派生于ClassLoader类
    - 父类加载器为拓展类加载器
    - 它负责加载环境变量classpath或系统属性 java.class.path指定路径下的类库
    - **该类加载器是程序中默认的类加载器**，一般来说，java应用的类都是由它来完成加载
    - 通过ClassLoader#getSystemClassLoader()方法可以获取到该类加载器

```java
/**
 * 虚拟机自带加载器
 */
public class ClassLoaderTest1 {
    public static void main(String[] args) {
        System.out.println("********启动类加载器*********");
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        //获取BootStrapClassLoader能够加载的api路径
        for (URL e:urls){
            System.out.println(e.toExternalForm());
        }

        //从上面的路径中随意选择一个类 看看他的类加载器是什么
        //Provider位于 /jdk1.8.0_171.jdk/Contents/Home/jre/lib/jsse.jar 下，引导类加载器加载它
        ClassLoader classLoader = Provider.class.getClassLoader();
        System.out.println(classLoader);//null

        System.out.println("********拓展类加载器********");
        String extDirs = System.getProperty("java.ext.dirs");
        for (String path : extDirs.split(";")){
            System.out.println(path);
        }
        //从上面的路径中随意选择一个类 看看他的类加载器是什么:拓展类加载器
        ClassLoader classLoader1 = CurveDB.class.getClassLoader();
        System.out.println(classLoader1);//sun.misc.Launcher$ExtClassLoader@4dc63996
    }
}
```

#### 2.3 用户自定义类加载器

**为什么**

- 隔离加载类
- 修改类加载的方式
- 拓展加载源
- 防止源码泄漏

![自定义类加载器](https://user-gold-cdn.xitu.io/2020/3/18/170ec89d3903bfb7?imageslim)

### 3 ClassLoader的常用方法及获取方法

#### 3.1 ClassLoader类，它是一个抽象类，其后所有的类加载器都继承自ClassLoader（不包括启动类加载器）

|方法名称            |               描述         |
|-                     |               - |
|getParent()| 返回该类加载器的超类加载器|
|loadClass（String name）	|加载名称为name的类，返回结果为java.lang.Class类的实例
|findClass（String name）	|查找名称为name的类，返回结果为java.lang.Class类的实例|
findLoadedClass（String name）|	查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例
defineClass（String name，byte[] b,int off,int len）	|把字节数组b中的内容转换为一个Java类 ，返回结果为java.lang.Class类的实例
defineClass（String name，byte[] b,int off,int len）	|把字节数组b中的内容转换为一个Java类 ，返回结果为java.lang.Class类的实例

#### 3.2 ClassLoader继承关系

拓展类加载器和系统类加载器间接继承于ClassLoader抽象类

![classloader继承关系](https://user-gold-cdn.xitu.io/2020/3/18/170ec8a4eaae0b43?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 3.3 获取ClassLoader的途径
![获取classloader](https://user-gold-cdn.xitu.io/2020/3/18/170ec8b36f5d4187?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![获取classloader代码](https://user-gold-cdn.xitu.io/2020/3/18/170ec8b6c081f466?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 4. 双亲委派机制
**Java虚拟机对class文件采用的是按需加载的方式，也就是说当需要使用该类时才会将它的class文件加载到内存生成的class对象。而且加载某个类的class文件时，java虚拟机采用的是双亲委派模式，即把请求交由父类处理，它是一种任务委派模式**

#### 4.1 双亲委派机制工作原理
![双亲委派机制](https://user-gold-cdn.xitu.io/2020/3/18/170ec8cbbe16af0c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**例**
如图，虽然我们自定义了一个java.lang包下的String尝试覆盖核心类库中的String，但是由于双亲委派机制，启动加载器会加载java核心类库的String类（BootStrap启动类加载器只加载包名为java、javax、sun等开头的类），而核心类库中的String并没有main方法
![双亲委派机制例子](https://user-gold-cdn.xitu.io/2020/3/18/170ec8d5358f0991?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 4.2 双亲委派机制的优势
- 避免类的重复加载
- 保护程序安全，防止核心API被随意篡改
    - 自定义类：java.lang.String
    - 自定义类：java.lang.MeDsh（java.lang包需要访问权限，阻止我们用包名自定义类）

![保护jar包](https://user-gold-cdn.xitu.io/2020/3/18/170ec8ddb41c5559?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**破坏双亲委派:**
![双亲委派流程](https://user-gold-cdn.xitu.io/2020/3/18/170ec8e47d7e861b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
或许你会想，我在自定义的类加载器里面强制加载自定义的java.lang.String类，不去通过调用父加载器不就好了吗?确实，这样是可行。但是，在JVM中，==判断一个对象是否是某个类型时，如果该对象的实际类型与待比较的类型的类加载器不同==，那么会返回false。
**jdbc 破坏双亲委派:**
**tomcat 破坏双亲委派**
### 5. 沙箱安全机制
自定义String类，但是在加载自定义String类的时候回率先使用引导类加载器加载，而引导类加载器在加载过程中会先加载jdk自带的文件（rt.jar包中的java\lang\String.class）,报错信息说没有main方法就是因为加载的是rt.jar包中的String类。这样可以保证对java核心源代码的保护，这就是沙箱安全机制.

### 6.其他

- 在jvm中表示两个class对象是否为同一个类存在的两个必要条件

    - 类的完整类名必须一致，包括包名
    - 加载这个类的ClassLoader（指ClassLoader实例对象）必须相同

- 换句话说，在jvm中，即使这两个类对象（class对象）来源同一个Class文件，被同一个虚拟机所加载，但只要加载它们的ClassLoader实例对象不同，那么这两个类对象也是不相等的.

#### 对类加载器的引用
JVM必须知道一个类型是由启动类加载器加载的还是由用户类加载器加载的。如果一个类型由用户类加载器加载的，那么**jvm会将这个类加载器的一个引用作为类型信息的会议部分保存在方法区中**。当解析一个类型到另一个类型的引用的时候，JVM需要保证两个类型的加载器是相同的。

#### 类的主动使用和被动使用
**java程序对类的使用方式分为：主动使用和被动使用**

- 主动使用，分为七种情况

    - 创建类的实例
    - 访问某各类或接口的静态变量，或者对静态变量赋值
    - 调用类的静态方法
    - 反射 比如Class.forName(com.dsh.jvm.xxx)
    - 初始化一个类的子类
    - java虚拟机启动时被标明为启动类的类
    - JDK 7 开始提供的动态语言支持：
    java.lang.invoke.MethodHandle实例的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic句柄对应的类没有初始化，则初始化

- 除了以上七种情况，其他使用java类的方式都被看作是对类的被动使用，都不会导致类的初始化。


## 运行时数据区1-[程序计数器+虚拟机栈+本地方法栈]

### 内存与线程

#### 1 内存
内存是非常重要的系统资源，是硬盘和cpu的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。JVM内存布局规定了JAVA在运行过程中内存申请、分配、管理的策略，保证了JVM的高效稳定运行。**不同的jvm对于内存的划分方式和管理机制存在着部分差异**（对于Hotspot主要指方法区）

![内存划分](https://user-gold-cdn.xitu.io/2020/3/18/170ecae266df65ba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
JDK8的元数据区+JIT编译产物 就是JDK8以前的方法区

#### 2 分区介绍
java虚拟机定了了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程对应的数据区域会随着线程开始和结束而创建和销毁。
如图，灰色的区域为单独线程私有的，红色的为多个线程共享的，即

- 每个线程：独立包括程序计数器、栈、本地栈
- 线程间共享：堆、堆外内存（方法区、永久代或元空间、代码缓存）

![内存分区](https://user-gold-cdn.xitu.io/2020/3/18/170ecae9790e6eac?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
一般来说，jvm优化95%是优化堆区，5%优化的是方法区

#### 3 线程

- 线程是一个程序里的运行单元，JVM允许一个程序有多个线程并行的执行；
- ==在HotSpot JVM，每个线程都与操作系统的本地线程直接映射==。
    - 当一个java线程准备好执行以后，此时一个操作系统的本地线程也同时创建。java线程执行终止后。本地线程也会回收。
- 操作系统负责所有线程的安排调度到任何一个可用的CPU上。一旦本地线程初始化成功，它就会调用java线程中的run（）方法.

##### 3.1 JVM系统线程

- 如果你使用jconsole或者任何一个调试工具，都能看到在后台有许多线程在运行。这些后台线程不包括调用main方法的main线程以及所有这个main线程自己创建的线程；
- 这些主要的后台系统线程在HotSpot JVM里主要是以下几个：

    - 虚拟机线程这种线程的操作时需要JVM达到安全点才会出现。这些操作必须在不同的线程中发生的原因是他们都需要JVM达到安全点，这样堆才不会变化。这种线程的执行包括“stop-the-world”的垃圾收集，线程栈收集，线程挂起以及偏向锁撤销
    - 周期任务线程：这种线程是时间周期事件的提现（比如中断），他们一般用于周期性操作的调度执行。
    - GC线程：这种线程对于JVM里不同种类的垃圾收集行为提供了支持
    - 编译线程：这种线程在运行时会降字节码编译成本地代码
    - 信号调度线程：这种线程接收信号并发送给JVM,在它内部通过调用适当的方法进行处理。

### 1.程序计数器（PC寄存器）
JVM中的程序计数寄存器（Program Counter Register）中，Register的命名源于CPU的寄存器，寄存器存储指令相关的现场信息。CPU只有把数据装载到寄存器才能够运行。JVM中的PC寄存器是对屋里PC寄存器的一种抽象模拟

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecaecbef6c19d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 1.1 作用
PC寄存器是用来存储指向下一条指令的地址，也即将将要执行的指令代码。由执行引擎读取下一条指令。

- 它是一块很小的内存空间，几乎可以忽略不计。也是运行速度最快的存储区域
- 在jvm规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致
- 任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前线程正在执行的java方法的JVM指令地址；或者，如果是在执行native方法，则是未指定值（undefined）。
- 程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成
- 字节码解释器工作时就是通过改变这个计数器的值来选取下一跳需要执行的字节码指令
- 它是唯一一个在java虚拟机规范中没有规定任何OOM情况的区域

#### 1.2 代码示例
利用javap -v xxx.class反编译字节码文件，查看指令等信息
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecaef254ef627?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 1.3 面试常问
1. **使用PC寄存器存储字节码指令地址有什么用呢？/
    为什么使用PC寄存器记录当前线程的执行地址呢？**
    因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行
    JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令
2. **PC寄存器为什么会设定为线程私有**
    我们都知道所谓的多线程在一个特定的时间段内指回执行其中某一个线程的方法，CPU会不停滴做任务切换，这样必然会导致经常中断或恢复，如何保证分毫无差呢？为了能够准确地记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器,这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。
    由于CPU时间片轮限制，众多线程在并发执行过程中，任何一个确定的时刻，一个处理器或者多核处理器中的一个内核，只会执行某个线程中的一条指令。
    这样必然导致经常中断或恢复，如何保证分毫无差呢？每个线程在创建后，都会产生自己的程序计数器和栈帧，程序计数器在各个线程之间互不影响。

### 2.虚拟机栈
#### 2.1概述
##### 2.1.1 背景
由于跨平台性的设计，java的指令都是根据栈来设计的。不同平台CPU架构不同，所以不能设计为基于寄存器的。
**优点是跨平台，指令集小，编译器容易实现，缺点是性能下降，实现同样的功能需要更多的指令。**
##### 2.1.2 内存中的堆与栈

- **栈是运行时的单位，而堆是存储的单位**
    即：栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。堆解决的是数据存储的问题，即数据怎么放、放在哪儿。
- 一般来讲，对象主要都是放在堆空间的，是运行时数据区比较大的一块
- 栈空间存放 基本数据类型的局部变量，以及引用数据类型的对象的引用

##### 2.1.3 虚拟机栈是什么

- java虚拟机栈（Java Virtual Machine Stack），早期也叫Java栈。
每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧（Stack Frame），对应这个一次次的java方法调用。它是线程私有的
- 生命周期和线程是一致的
- 作用：主管java程序的运行，它保存方法的局部变量（8种基本数据类型、对象的引用地址）、部分结果，并参与方法的调用和返回。

    - 局部变量：相对于成员变量（或属性）
    - 基本数据变量： 相对于引用类型变量（类，数组，接口）


##### 2.1.4 栈的特点
- 栈是一种快速有效的分配存储方式，访问速度仅次于PC寄存器（程序计数器）
- JVM直接对java栈的操作只有两个
    - 每个方法执行，伴随着进栈（入栈，压栈）
    - 执行结束后的出栈工作
- 对于栈来说不存在垃圾回收问题

##### 2.1.5 栈中可能出现的异常
java虚拟机规范允许**Java栈的大小是动态的或者是固定不变的**

- 如果采用固定大小的Java虚拟机栈，那每一个线程的java虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过java虚拟机栈允许的最大容量，java虚拟机将会抛出一个 **StackOverFlowError异常**

- 如果java虚拟机栈可以动态拓展，并且在尝试拓展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那java虚拟机将会抛出一个 **OutOfMemoryError异常**

##### 2.1.6设置栈的内存大小
我们可以使用参数-Xss选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度。 （IDEA设置方法：Run-EditConfigurations-VM options 填入指定栈的大小-Xss256k）

#### 2.2 栈的存储结构和运行原理
##### 2.2.1

- 每个线程都有自己的栈，栈中的数据都是以栈帧(Stack Frame)的格式存在
- 在这个线程上正在执行的每个方法都对应各自的一个栈帧
- 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息
- JVM直接对java栈的操作只有两个，就是对栈帧的压栈和出栈，遵循先进后出/后进先出的和原则。
- 在一条活动线程中，一个时间点上，只会有一个活动的栈帧。即只有当前正在执行的方法的栈帧（栈顶栈帧）是有效的，这个栈帧被称为当前栈帧(Current Frame),与当前栈帧对应的方法就是当前方法（Current Frame）
- 执行引擎运行的所有字节码指令只针对当前栈帧进行操作
- 如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，成为新的当前栈帧。
- 不同线程中所包含的栈帧是不允许相互引用的，即不可能在另一个栈帧中引用另外一个线程的栈帧
- 如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧
- Java方法有两种返回函数的方式，一种是正常的函数返回，使用return指令；另外一种是抛出异常。不管使用哪种方式，都会导致栈帧被弹出。
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecafb0c5ada68?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 2.2.2 栈帧的内部结构
每个栈帧中存储着：

- 局部变量表（Local Variables）
- 操作数栈（Operand Stack）(或表达式栈)
- 动态链接（Dynamic Linking）(或执行运行时常量池的方法引用)
- 方法返回地址（Return Adress）（或方法正常退出或者异常退出的定义）
- 一些附加信息
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecafe0fab0cb2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 2.3 局部变量表（Local Variables）
##### 2.3.1 概述
- 局部变量表也被称之为局部变量数组或本地变量表
- **定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量**这些数据类型包括各类基本数据类型、对象引用（reference），以及returnAddress类型
- 由于局部变量表是建立在线程的栈上，是线程私有的数据，因此不存在数据安全问题
- **局部变量表所需的容量大小是在编译期确定下来的**,并保存在方法的Code属性的maximum local variables数据项中。在方法运行期间是不会改变局部变量表的大小的
- **方法嵌套调用的次数由栈的大小决定。一般来说，栈越大，方法嵌套调用次数越多。** 对一个函数而言，他的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需传递的信息增大的需求。进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少。
- **局部变量表中的变量只在当前方法调用中有效**。在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。**当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁。**
利用javap命令对字节码文件进行解析查看局部变量表，如图：
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb01f86381ee?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
也可以在IDEA 上安装jclasslib byte viewcoder插件查看字节码信息,以main()方法为例

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb062339c98b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb0825056429?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb0cf610a812?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb109244b776?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 2.3.2 变量槽slot的理解与演示
- 参数值的存放总是在局部变量数组的index0开始，到数组长度-1的索引结束
局部变量表，**最基本的存储单元是Slot(变量槽)**
- 局部变量表中存放编译期可知的各种基本数据类型（8种），引用类型（reference），returnAddress类型的变量。
- 在局部变量表里，32位以内的类型只占用一个slot（包括returnAddress类型），64位的类型（long和double）占用两个slot。
    - byte、short、char、float在存储前被转换为int，boolean也被转换为int，0表示false，非0表示true；
    - long和double则占据两个slot。

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb1565b0252f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- JVM会为局部变量表中的每一个slot都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值
- 当一个实例方法被调用的时候，它的方法参数和方法体内部定义的局部变量将会**按照顺序被复制到局部变量表中的每一个slot上**
- **如果需要访问局部变量表中一个64bit的局部变量值时，只需要使用第一个索引即可。**（比如：访问long或者double类型变量）
- 如果当前帧是由构造方法或者实例方法创建的，那么**该对象引用this将会存放在index为0的slot处**,其余的参数按照参数表顺序排列。

##### 2.3.3 slot的重复利用
栈帧中的局部变量表中的槽位是可以重复利用的，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变量就很有可能会复用过期局部变量的槽位，从而达到节省资源的目的。

```java
private void test2() {
        int a = 0;
        {
            int b = 0;
            b = a+1;
        }
        //变量c使用之前以及经销毁的变量b占据的slot位置
        int c = a+1;
    }
```

##### 2.3.4 静态变量与局部变量的对比及小结
变量的分类：

- 按照数据类型分：
    1. 基本数据类型;
    2. 引用数据类型；
- 按照在类中声明的位置分：
    1. 成员变量：在使用前，都经历过默认初始化赋值
        - static修饰：类变量：类加载linking的准备阶段给类变量默认赋值——>初始化阶段给类变量显式赋值即静态代码块赋值；
        - 不被static修饰：实例变量：随着对象的创建，会在堆空间分配实例变量空间，并进行默认赋值
    2. 局部变量：在使用前，必须要进行显式赋值的！否则，编译不通过 补充：
- 在栈帧中，与性能调优关系最为密切的部分就是局部变量表。在方法执行时，虚拟机使用局部变量表完成方法的传递
- **局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收**

#### 2.4 操作数栈（Operand Stack）
栈 ：可以使用数组或者链表来实现

- 每一个独立的栈帧中除了包含局部变量表以外，还包含一个后进先出的操作数栈，也可以成为表达式栈
- **操作数栈，在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据，即入栈（push）或出栈（pop）**
    - 某些字节码指令将值压入操作数栈，其余的字节码指令将操作数取出栈，使用他们后再把结果压入栈。（如字节码指令bipush操作）
    - 比如：执行复制、交换、求和等操作

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb180342dcf0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 2.4.1 概述
- 操作数栈，**主要用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间。**
- 操作数栈就是jvm执行引擎的一个工作区，当一个方法开始执行的时候，一个新的栈帧也会随之被创建出来，这个方法的操作数栈是空的
- 每一个操作数栈都会拥有一个明确的栈深度用于存储数值，其所需的最大深度在编译器就定义好了，保存在方法的code属性中，为max_stack的值。
- 栈中的任何一个元素都是可以任意的java数据类型
    - 32bit的类型占用一个栈单位深度
    - 64bit的类型占用两个栈深度单位
- 操作数栈**并非采用访问索引的方式来进行数据访问**的，而是只能通过标砖的入栈push和出栈pop操作来完成一次数据访问
- **如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新PC寄存器中下一条需要执行的字节码指令。
- 操作数栈中的元素的数据类型必须与字节码指令的序列严格匹配，这由编译器在编译期间进行验证，同时在类加载过程中的类验证阶段的数据流分析阶段要再次验证。
- 另外，我们说Java虚拟机的**解释引擎是基于栈的执行引擎**,其中的栈指的就是操作数栈。
**结合上图结合下面的图来看一下一个方法（栈帧）的执行过程**
①15入栈；②存储15，15进入局部变量表
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb1c4797b788?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
③压入8；④存储8，8进入局部变量表；
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb53336d048b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
⑤从局部变量表中把索引为1和2的是数据取出来，放到操作数栈；⑥iadd相加操作，8和15出栈
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb55fd42e99f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
⑦iadd操作结果23入栈；⑧将23存储在局部变量表索引为3的位置上
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb5802369d83?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 2.4.2 i++ 和 ++i的区别
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb6e2832562f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 2.4.3 栈顶缓存技术ToS（Top-of-Stack Cashing）
- 基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要更多的指令分派（instruction dispatch）次数和内存读/写次数
- 由于操作数是存储在内存中的，因此频繁地执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM的设计者们提出了栈顶缓存技术，**将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率**

#### 2.5 动态链接（Dynamic Linking）
- ==每一个栈帧内部都包含一个指向运行时常量池或该栈帧所属方法的引用==。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接。比如invokedynamic指令
- 在Java源文件被编译成字节码文件中时，所有的变量和方法引用都作为符号引用（symbolic Refenrence）保存在class文件的常量池里。比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么**动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用**。

**为什么需要常量池呢?**
常量池的作用，就是为了提供一些符号和常量，便于指令的识别。
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb79de12c6c0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb7b7f2457c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 2.5.1方法的调用
**在JVM中，将符号引用转换为调用方法的直接引用与方法的绑定机制相关**

- **静态链接**
    当一个 字节码文件被装载进JVM内部时，如果被调用的目标方法在编译期可知，且运行期保持不变时。这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接。
- **动态链接**
    如果被调用的方法在编译期无法被确定下来，也就是说，只能够在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接。

对应的方法的绑定机制为：早期绑定（Early Binding）和晚期绑定（Late Bingding）。绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生一次。

- **早期绑定**
    早期绑定就是指被调用的目标方法如果在编译期可知，且运行期保持不变时，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用。
- **晚期绑定**
    如果被调用的方法在编译期无法被确定下来，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式也就被称之为晚期绑定。

随着高级语言的横空出世，类似于java一样的基于面向对象的编程语言如今越来越多，尽管这类编程语言在语法风格上存在一定的差别，但是它们彼此之间始终保持着一个共性，那就是都支持封装，集成和多态等面向对象特性，既然这一类的编程语言具备多态特性，那么自然也就具备早期绑定和晚期绑定两种绑定方式。
Java中任何一个普通的方法其实都具备虚函数的特征，它们相当于C++语言中的虚函数（C++中则需要使用关键字virtual来显式定义）。如果在Java程序中不希望某个方法拥有虚函数的特征时，则可以使用关键字final来标记这个方法。

##### 2.5.2虚方法和非虚方法
**子类对象的多态性使用前提：①类的继承关系②方法的重写**

非虚方法

- 如果方法在编译器就确定了具体的调用版本，这个版本在运行时是不可变的。这样的方法称为非虚方法
- 静态方法、私有方法、final方法、实例构造器、父类方法都是非虚方法
- 其他方法称为虚方法

**虚拟机中提供了以下几条方法调用指令：**
普通调用指令：

1. invokestatic：调用静态方法，解析阶段确定唯一方法版本；
2. invokespecial:调用方法、私有及 final 方法，解析阶段确定唯一方法版本；
3. invokevirtual调用所有虚方法；
4. invokeinterface：调用接口方法；

动态调用指令：

5. invokedynamic：动态解析出需要调用的方法，然后执行 .

前四条指令固化在虚拟机内部，方法的调用执行不可人为干预，而invokedynamic指令则支持由用户确定方法版本。**其中invokestatic指令和invokespecial指令调用的方法称为非虚方法，其余的（final修饰的除外）称为虚方法。**

```java
/**
 * 解析调用中非虚方法、虚方法的测试
 */
class Father {
    public Father(){
        System.out.println("Father默认构造器");
    }

    public static void showStatic(String s){
        System.out.println("Father show static"+s);
    }

    public final void showFinal(){
        System.out.println("Father show final");
    }

    public void showCommon(){
        System.out.println("Father show common");
    }

}

public class Son extends Father{
    public Son(){
        super();
    }

    public Son(int age){
        this();
    }

    public static void main(String[] args) {
        Son son = new Son();
        son.show();
    }

    //不是重写的父类方法，因为静态方法不能被重写
    public static void showStatic(String s){
        System.out.println("Son show static"+s);
    }

    private void showPrivate(String s){
        System.out.println("Son show private"+s);
    }

    public void show(){
        //非虚方法如下
        //invokestatic
        showStatic(" 大头儿子");
        //invokestatic
        super.showStatic(" 大头儿子");
        //invokespecial
        showPrivate(" hello!");
        //invokespecial
        super.showCommon();
        //invokevirtual 因为此方法声明有final 不能被子类重写，所以也认为该方法是非虚方法
        showFinal();

        //虚方法如下
        //invokevirtual
        showCommon();//没有显式加super，被认为是虚方法，因为子类可能重写showCommon
        info();

        MethodInterface in = null;
        //invokeinterface  不确定接口实现类是哪一个 需要重写
        in.methodA();

    }

    public void info(){

    }

}

interface MethodInterface {
    void methodA();
}

```

**关于invokedynamic指令**
- JVM字节码指令集一直比较稳定，一直到jdk7才增加了一个invokedynamic指令，这是Java为了实现【动态类型语言】支持而做的一种改进
- 但是jdk7中并没有提供直接生成invokedynamic指令的方法，需要借助ASM这种底层字节码工具来产生invokedynamic指令.直到Jdk8的Lambda表达式的出现，invokedynamic指令的生成，在java中才有了直接生成方式
- Jdk7中增加的动态语言类型支持的本质是对java虚拟机规范的修改，而不是对java语言规则的修改，这一块相对来讲比较复杂，增加了虚拟机中的方法调用，最直接的受益者就是运行在java平台的动态语言的编译器

**动态类型语言和静态类型语言**
- 动态类型语言和静态类型语言两者的却别就在于对类型的检查是在编译期还是在运行期，满足前者就是静态类型语言，反之则是动态类型语言。
- 直白来说 **静态语言是判断变量自身的类型信息；动态类型语言是判断变量值的类型信息，变量没有类型信息，变量值才有类型信息**,这是动态语言的一个重要特征
Java是静态类型语言（尽管lambda表达式为其增加了动态特性），js，python是动态类型语言.

##### 2.5.3方法重写的本质
1. 找到操作数栈的第一个元素所执行的对象的实际类型，记作C。
2. 如果在类型C中找到与常量中的描述符合简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；如果不通过，则返回`java.lang.IllegalAccessError`异常。
3. 否则，按照继承关系从下往上依次对c的各个父类进行第二步的搜索和验证过程。
4. 如果始终没有找到合适的方法，则抛出`java.lang.AbstractMethodError`异常。 `IllegalAccessError`介绍 程序视图访问或修改一个属性或调用一个方法，这个属性或方法，你没有权限访问。一般的，这个会引起编译器异常。这个错误如果发生在运行时，就说明一个类发生了不兼容的改变。

##### 2.5.4 虚方法表
- 在面向对象编程中，会很频繁期使用到动态分派，如果在每次动态分派的过程中都要重新在类的方法元数据中搜索合适的目标的话就可能影响到执行效率。因此，为了提高性能，jvm采用在类的方法区建立一个虚方法表（virtual method table）（非虚方法不会出现在表中）来实现。使用索引表来代替查找。==虚方法表中存放着各个方法的实际入口地址，如果子类没有覆盖父类的方法，那么子类的虚方法表里面的地址入口与父类是一致的；如果重写父类的方法，那么子类的方法表的地址将会替换为子类实现版本的地址。方法表是在类加载的连接阶段进行初始化，准备了子类的初始化值后，虚拟机会把该类的虚方法表也进行初始化。==
- ==每个类中==都有一个虚方法表，表中存放着各个方法的实际入口。
- 那么虚方法表什么时候被创建？ 虚方法表会在==类加载的链接阶段==被创建 并开始初始化，类的变量初始值准备完成之后，jvm会把该类的虚方法表也初始化完毕。

![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb7f8233cc27?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 2.6 方法返回地址（Return Address）
- 存放调用该方法的PC寄存器的值。
- 一个方法的结束，有两种方式：
    - 正常执行完成
    - 出现未处理的异常，非正常退出
- 无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。**方法正常退出时，调用者的pc计数器的值作为返回地址**，即调用该方法的指令的下一条指令的地址。而通过异常退出时，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息。
- 本质上，方法的退出就是当前栈帧出栈的过程。此时，需要恢复上层方法的局部变量表、操作数栈、将返回值也如调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去。
- **正常完成出口和异常完成出口的区别在于：通过异常完成出口退出的不会给他的上层调用者产生任何的返回值**。

当一个方法开始执行后，只要两种方式可以退出这个方法： 
1. 执行引擎遇到任意一个方法返回的字节码指令（return），会有返回值传递给上层的方法调用者，简称正常完成出口；
    - 一个方法在正常调用完成之后究竟需要使用哪一个返回指令还需要根据方法返回值的实际数据类型而定
    - 在字节码指令中，返回指令包含ireturn（当返回值是boolena、byte、char、short和int类型时使用）、lreturn、freturn、dreturn以及areturn，另外还有一个return指令供声明为void的方法、实例初始化方法、类和接口的初始化方法使用
2. 在方法执行的过程中遇到了异常（Exception），并且这个异常没有在方法内进行处理，也就是只要在本方法的异常表中没有搜素到匹配的异常处理器，就会导致方法退出，简称**异常完成出口**
方法执行过程中抛出异常时的异常处理，存储在一个异常处理表，方便在发生异常的时候找到处理异常的代码。
![](https://user-gold-cdn.xitu.io/2020/3/18/170ecb82ce5a6a35?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

2.7 虚拟机栈的5道面试题
1. 举例栈溢出的情况？（StackOverflowError）
    - 递归调用等，通过-Xss设置栈的大小；
2. 调整栈的大小，就能保证不出现溢出么？
    - 不能 如递归无限次数肯定会溢出，调整栈大小只能保证溢出的时间晚一些
3. 分配的栈内存越大越好么？
    - 不是 会挤占其他线程的空间
4. 垃圾回收是否会涉及到虚拟机栈？

    | 内存块 | error | GC |
    | -         | :-: | :-: |
    | 程序计数器 | ❌ | ❌ |
    |本地方法栈	|✅	|❌
    jvm虚拟机栈|	✅|	❌
    堆	|✅	|✅
    方法区|	✅	|✅

5. 方法中定义的局部变量是否线程安全？

```java
/**
 * 面试题：
 * 方法中定义的局部变量是否线程安全？具体情况具体分析
 *
 * 何为线程安全？
 *     如果只有一个线程可以操作此数据，则毙是线程安全的。
 *     如果有多个线程操作此数据，则此数据是共享数据。如果不考虑同步机制的话，会存在线程安全问题
 *
 * StringBuffer是线程安全的，StringBuilder不是
 */
public class StringBuilderTest {

    //s1的声明方式是线程安全的
    public static void method1(){
        StringBuilder s1 = new StringBuilder();
        s1.append("a");
        s1.append("b");
    }

    //stringBuilder的操作过程：是不安全的，因为method2可以被多个线程调用
    public static void method2(StringBuilder stringBuilder){
        stringBuilder.append("a");
        stringBuilder.append("b");
    }

    //s1的操作：是线程不安全的 有返回值，可能被其他线程共享
    public static StringBuilder method3(){
        StringBuilder s1 = new StringBuilder();
        s1.append("a");
        s1.append("b");
        return s1;
    }

    //s1的操作：是线程安全的 ，StringBuilder的toString方法是创建了一个新的String，s1在内部消亡了
    public static String method4(){
        StringBuilder s1 = new StringBuilder();
        s1.append("a");
        s1.append("b");
        return s1.toString();
    }

    public static void main(String[] args) {
        StringBuilder s = new StringBuilder();
        new Thread(()->{
            s.append("a");
            s.append("b");
        }).start();

        method2(s);

    }

}
```

### 3.本地方法栈
- Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用
- 本地方法栈，也是线程私有的。
- 允许被实现成固定或者是可动态拓展的内存大小。（在内存溢出方面是相同的）
    - 如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java虚拟机将会抛出一个StackOverFlowError异常。
    - 如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么java虚拟机将会抛出一个OutOfMemoryError异常。
- 本地方法是使用C语言实现的
- 它的具体做法是Native Method Stack中登记native方法，在Execution Engine执行时加载本地方法库。
- **当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有同样的权限**
    - 本地方法可以通过本地方法接口来 访问虚拟机内部的运行时数据区
    - 它甚至可以直接使用本地处理器中的寄存器
    - 直接从本地内存的堆中分配任意数量的内存
- 并不是所有的JVM都支持本地方法。因为Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。如果JVM产品不打算支持native方法，也可以无需实现本地方法栈。
- 在hotSpot JVM中，直接将本地方法栈和虚拟机栈合二为一。

**为什么要使用Native Method?**
java使用起来非常方便，然而有些层次的任务用java实现起来不容易，或者我们对程序的效率很在意时，问题就来了。

- 与java环境外交互：
有时java应用需要与java外面的环境交互，这是本地方法存在的主要原因。
你可以想想java需要与一些底层系统，如操作系统或某些硬件交换信息时的情况。本地方法正式这样的一种交流机制：它为我们提供了一个非常简洁的接口，而且我们无需去了解java应用之外的繁琐细节。
- 与操作系统交互
JVM支持着java语言本身和运行库，它是java程序赖以生存的平台，它由一个解释器（解释字节码）和一些连接到本地代码的库组成。然而不管怎样，它毕竟不是一个完整的系统，它经常依赖于一些底层系统的支持。这些底层系统常常是强大的操作系统。通过使用本地方法，我们得以用java实现了jre的与底层系统的交互，甚至jvm的一些部分就是用C写的。还有，如果我们要使用一些java语言本身没有提供封装的操作系统特性时，我们也需要使用本地方法。
- Sun’s Java
Sun的解释器是用C实现的，这使得它能像一些普通的C一样与外部交互。jre大部分是用java实现的，它也通过一些本地方法与外界交互。例如：类`java.lang.Thread`的`setPriority()`方法是用Java实现的，但是它实现调用的事该类里的本地方法`setPriority0()`。这个本地方法是用C实现的，并被植入JVM内部，在`Windows 95`的平台上，这个本地方法最终将调用`Win32` `SetProority()`API。这是一个本地方法的具体实现由JVM直接提供，更多的情况是本地方法由外部的动态链接库（`external dynamic link library`）提供，然后被JVM调用。

## 运行时数据区2-堆
### 1.核心概述
一个进程对应一个jvm实例，一个运行时数据区，又包含多个线程，这些线程共享了方法区和堆，每个线程包含了程序计数器、本地方法栈和虚拟机栈。

1. 一个jvm实例只存在一个堆内存，堆也是java内存管理的核心区域
2. Java堆区在JVM启动的时候即被创建，其空间大小也就确定了。是JVM管理的最大一块内存空间（堆内存的大小是可以调节的）
3. 《Java虚拟机规范》规定，堆可以处于==物理上不连续==的内存空间中，但在==逻辑上它应该被视为连续的==
4. 所有的线程共享java堆，在这里还可以划分线程私有的缓冲区（`TLAB:Thread Local Allocation Buffer`）.（面试问题：堆空间一定是所有线程共享的么？不是，TLAB线程在堆中独有的）
5. 《Java虚拟机规范》中对java堆的描述是：所有的对象实例以及数组都应当在运行时分配在堆上。
    - 从实际使用的角度看，“几乎”所有的对象的实例都在这里分配内存 （‘几乎’是因为可能存储在栈上）
6. 数组或对象永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置
7. 在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除
8. 堆，是GC(Garbage Collection，垃圾收集器)执行垃圾回收的重点区域

#### 1.1 配置jvm及查看jvm进程
- 编写HeapDemo/HeapDemo1代码
```java
public class HeapDemo {
    public static void main(String[] args) {
        System.out.println("start...");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end...");
    }
}
```

- 首先对虚拟机进行配置，如图 `Run-Edit configurations`

    ![](https://user-gold-cdn.xitu.io/2020/5/28/1725a5fe12fef584?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 在jdk目录，我的是`/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Contents/Home/bin`下找到`jvisualvm` 运行（或者直接终端运行   ），查看进程，可以看到我们设置的配置信息
    ![](https://user-gold-cdn.xitu.io/2020/5/28/1725a6053aec9841?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
- 可以看到HeapDemo配置-Xms10m， 分配的10m被分配给了新生代3m和老年代7m
    ![](https://user-gold-cdn.xitu.io/2020/5/28/1725a616c775325d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 1.2 分析SimpleHeap的jvm情况

```java
public class SimpleHeap {
    private int id;//属性、成员变量

    public SimpleHeap(int id) {
        this.id = id;
    }

    public void show() {
        System.out.println("My ID is " + id);
    }
    public static void main(String[] args) {
        SimpleHeap sl = new SimpleHeap(1);
        SimpleHeap s2 = new SimpleHeap(2);

        int[] arr = new int[10];

        Object[] arr1 = new Object[10];
    }
}
```

![](https://user-gold-cdn.xitu.io/2020/5/28/1725a63f3bd98cd1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 1.3 堆的细分内存结构
- JDK 7以前： 新生区+养老区+永久区
    - `Young Generation Space`：又被分为`Eden`区和`Survior`区 ==Young/New==
    - `Tenure generation Space`： ==Old/Tenure==
    - `Permanent Space`： ==Perm==

![](https://user-gold-cdn.xitu.io/2020/5/28/1725a665af15dfb5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- JDK 8以后： 新生区+养老区+元空间
    - `Young Generation Space`：又被分为`Eden`区和`Survior`区 ==Young/New==
    - `Tenure generation Space`： ==Old/Tenure==
    - `Meta Space`： ==Meta==

![](https://user-gold-cdn.xitu.io/2020/5/28/1725a66b57177838?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 2.设置堆内存大小与OOM
- Java堆区用于存储java对象实例，堆的大小在jvm启动时就已经设定好了，可以通过 `-Xmx`和 `-Xms`来进行设置
    - `-Xms` 用于表示堆的起始内存，等价于 `-XX:InitialHeapSize`
        - `-Xms` 用来设置堆空间（年轻代+老年代）的初始内存大小
            - `-X` 是jvm的运行参数
            - `ms` 是memory start
    - `-Xmx` 用于设置堆的最大内存，等价于 `-XX:MaxHeapSize`
- 一旦堆区中的内存大小超过 -Xmx所指定的最大内存时，将会抛出OOM异常
- ==通常会将-Xms和-Xmx两个参数配置相同的值，其目的就是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能==(频繁的申请, 释放内存消耗资源)
- 默认情况下，初始内存大小：物理内存大小/64;最大内存大小：物理内存大小/4
    - 手动设置：`-Xms600m` `-Xmx600m`
- 查看设置的参数：
    - 方式一： ==终端输入jps== ， 然后 ==jstat -gc 进程id==
    - 方式二：（控制台打印）Edit Configurations->VM Options 添加 ==-XX:+PrintGCDetails==

#### 2.1 查看堆内存大小
```java
public class HeapSpaceInitial {
    public static void main(String[] args) {

        //返回Java虚拟机中的堆内存总量
        long initialMemory = Runtime.getRuntime().totalMemory() / 1024 / 1024;
        //返回Java虚拟机试图使用的最大堆内存量
        long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;

        System.out.println("-Xms : " + initialMemory + "M");//-Xms : 245M
        System.out.println("-Xmx : " + maxMemory + "M");//-Xmx : 3641M

        System.out.println("系统内存大小为：" + initialMemory * 64.0 / 1024 + "G");//系统内存大小为：15.3125G
        System.out.println("系统内存大小为：" + maxMemory * 4.0 / 1024 + "G");//系统内存大小为：14.22265625G

        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### 2.2 堆大小分析
设置堆大小为600m，打印出的结果为575m，这是因为幸存者区S0和S1各占据了25m，但是他们始终有一个是空的，存放对象的是伊甸园区和一个幸存者区
![](https://user-gold-cdn.xitu.io/2020/5/28/1725a678630ccbec?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 3.年轻代与老年代
- 存储在JVM中的java对象可以被划分为两类：

    - 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速
    - 另外一类对象时生命周期非常长，在某些情况下还能与JVM的生命周期保持一致
- Java堆区进一步细分可以分为年轻代（`YoungGen`）和老年代（`OldGen`）

- 其中年轻代可以分为`Eden`空间、`Survivor0`空间和`Survivor1`空间（有时也叫`from`区，`to`区）
![](https://user-gold-cdn.xitu.io/2020/5/28/1725a6811c74d76b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 配置新生代与老年代在堆结构的占比

    - 默认`-XX：NewRatio=2`，表示新生代占1，老年代占2，新生代占整个堆的1/3
    - 可以修改`-XX:NewRatio=4`，表示新生代占1，老年代占4，新生代占整个堆的1/5

- 在hotSpot中，`Eden`空间和另外两个`Survivor`空间缺省所占的比例是==8：1：1==（测试的时候是6：1：1），开发人员可以通过选项 `-XX:SurvivorRatio` 调整空间比例，如`-XX:SurvivorRatio=8`

- 几乎所有的Java对象都是在Eden区被new出来的

- 绝大部分的Java对象都销毁在新生代了（IBM公司的专门研究表明，新生代80%的对象都是“朝生夕死”的）

- 可以使用选项`-Xmn`设置新生代最大内存大小（这个参数一般使用默认值就好了）

**测试代码**
```java
/**
 * -Xms600m -Xmx600m
 *
 * -XX:NewRatio ： 设置新生代与老年代的比例。默认值是2.
 * -XX:SurvivorRatio ：设置新生代中Eden区与Survivor区的比例。默认值是8
 * -XX:-UseAdaptiveSizePolicy ：关闭自适应的内存分配策略 '-'关闭,'+'打开  （暂时用不到）
 * -Xmn:设置新生代的空间的大小。 （一般不设置）
 *
 */
public class EdenSurvivorTest {
    public static void main(String[] args) {
        System.out.println("我只是来打个酱油~");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 4.图解对象分配过程

#### 4.1
为新对象分配内存是件非常严谨和复杂的任务，JVM的设计者们不仅需要考虑内存如何分配、在哪里分配的问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行完内存回收后是否会在内存空间中产生内存碎片。

1. new的对象先放伊甸园区。此区有大小限制。
2. 当伊甸园的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC),将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区
3. 然后将伊甸园中的剩余对象移动到幸存者0区。
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的，如果没有回收，就会放到幸存者1区。
5. 如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区。
6. 啥时候能去养老区呢？可以设置次数。默认是15次。·可以设置参数：`-XX:MaxTenuringThreshold`进行设置。
7. 在养老区，相对悠闲。当老年区内存不足时，再次触发GC：Major GC，进行养老区的内存清理。
8. 若养老区执行了Major GC之后发现依然无法进行对象的保存，就会产生OOM异常。

**总结**
==针对幸存者s0,s1区：复制之后有交换，谁空谁是to==
==关于垃圾回收：频繁在新生区收集，很少在养老区收集，几乎不再永久区/元空间收集。==

![](https://user-gold-cdn.xitu.io/2020/5/28/1725a6b5a5099b93?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 4.2 对象分配的特殊情况
![](https://user-gold-cdn.xitu.io/2020/5/28/1725a6bccc4ef0bb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 5.Minor GC、Major GC、Full GC
JVM在进行GC时，并非每次都针对上面三个内存区域（新生代、老年代、方法区）一起回收的，大部分时候回收都是指新生代。

针对hotSpot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（`Partial GC`），一种是整堆收集（`Full GC`）

- 部分收集：不是完整收集整个Java堆的垃圾收集。其中又分为：
    - 新生代收集（`Minor GC`/`Young GC`）：只是新生代的垃圾收集
    - 老年代收集（`Major GC`/`Old GC`）：只是老年代的垃圾收集
        - 目前，只有`CMS GC`会有单独收集老年代的行为
        - 注意，==很多时候`Major GC` 会和 `Full GC`混淆使用，需要具体分辨是老年代回收还是整堆回收==
    - 混合收集（`Mixed GC`）：收集整个新生代以及部分老年代的垃圾收集
        - 目前，只有`G1 GC`会有这种行为

- 整堆收集（`Full GC`）：收集整个java堆和方法区的垃圾收集
- 年轻代GC（`Minor GC`）触发机制：
当年轻代空间不足时，就会触发`Minor GC`，==这里的年轻代满指的是`Eden`区满，`Survivor`满不会引发GC==.(每次`Minor GC`会清理年轻代的内存，`Survivor`是被动GC，不会主动GC)
因为Java对象大多都具备朝生夕灭的特性，所以`Minor GC` 非常频繁，一般回收速度也比较快，这一定义既清晰又利于理解。
`Minor GC` 会引发`STW`（`Stop the World`），暂停其他用户的线程，等垃圾回收结束，用户线程才恢复运行。

- 老年代GC(`Major GC`/`Full GC`)触发机制
    - 指发生在老年代的GC, 就会触发`Major GC` 或者 `Full GC` 
    - 出现了`Major GC`，经常会伴随至少一次的`Minor GC`（不是绝对的，在`Parallel Scavenge` 收集器的收集策略里就有直接进行Major GC的策略选择过程）
        - 也就是老年代空间不足时，会先尝试触发`Minor GC`。如果之后空间还不足，则触发`Major GC`
    - `Major GC`速度一般会比`Minor GC`慢10倍以上，`STW`时间更长
    - 如果Major GC后，内存还不足，就报OOM了

- **`Full GC`触发机制**
    - 触发`Full GC`执行的情况有以下五种
        1. 调用`System.gc()`时，系统建议执行`Full GC`，但是不必然执行
        2. 老年代空间不足
        3. 方法区空间不足
        4. 通过`Minor GC`后进入老年代的平均大小小于老年代的可用内存
        5. 由Eden区，`Survivor S0`（`from`）区向`S1`（`to`）区复制时，对象大小由于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小
    - 说明：Full GC 是开发或调优中尽量要避免的，这样暂停时间会短一些

### 6.内存分配策略
- 如果对象在Eden出生并经过第一次Minor GC后依然存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，把那个将对象年龄设为1.对象在Survivor区中每熬过一次MinorGC，年龄就增加一岁，当它的年龄增加到一定程度（默认15岁，其实每个JVM、每个GC都有所不同）时，就会被晋升到老年代中
    - 对象晋升老年代的年龄阈值，可以通过选项 -XX：MaxTenuringThreshold来设置
- 针对不同年龄段的对象分配原则如下：
    - 优先分配到Eden
    - 大对象直接分配到老年代
        - 尽量避免程序中出现过多的大对象
    -   长期存活的对象分配到老年代
    - 动态对象年龄判断
        - **如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入到老年代。无需等到MaxTenuringThreshold中要求的年龄**
    - 空间分配担保
        - -XX: HandlePromotionFailure
#### 晋升到老年代的情况
1. 分配到新生代时, 如果 minor gc 后, 新生代还不能放下该对象, 则直接分配到老年代(如果此时老年代也放不下就 major gc)
2. minor gc 时, 如果 eden 区的对象不能分配到 survivor 区, 则晋升到老年代
3. 在 survivor 区达到晋升阈值(默认15)时, 晋升到老年区
4. 如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入到老年代

### 7.为对象分配内存：TLAB（线程私有缓存区域）
![](https://user-gold-cdn.xitu.io/2020/5/28/1725a6e5328e2947?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**为什么有TLAB（Thread Local Allocation Buffer）**

- 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
- 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度

**什么是TLAB**
- 从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，JVM为每个线程分配了一个私有缓存区域，它==包含在Eden空间内==
- 多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称之为快速分配策略
- 所有OpenJDK衍生出来的JVM都提供了TLAB的设计

**说明**
- 尽管不是所有的对象实例都能够在TLAB中成功分配内存，但JVM明确是将TLAB作为内存分配的首选
- 在程序中，开发人员可以通过选项“-XX:UseTLAB“ 设置是够开启TLAB空间
- 默认情况下，TLAB空间的内存非常小，仅占有整个EDen空间的1%，当然我们可以通过选项 ”-XX:TLABWasteTargetPercent“ 设置TLAB空间所占用Eden空间的百分比大小
- 一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在Eden空间中分配了内存

**TLAB对象分配过程**
![](https://user-gold-cdn.xitu.io/2020/5/28/1725a6f315abfcae?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 8.小结堆空间的参数设置
- -XX:PrintFlagsInitial: 查看所有参数的默认初始值
- -XX:PrintFlagsFinal：查看所有的参数的最终值（可能会存在修改，不再是初始值）
    - 具体查看某个参数的指令：
        - jps：查看当前运行中的进程
        - jinfo -flag SurvivorRatio 进程id： 查看新生代中Eden和S0/S1空间的比例
- -Xms: 初始堆空间内存（默认为物理内存的1/64）
- -Xmx: 最大堆空间内存（默认为物理内存的1/4）
- -Xmn: 设置新生代大小（初始值及最大值）
- -XX:NewRatio: 配置新生代与老年代在堆结构的占比
- -XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例
- -XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄(默认15)
- -XX:+PrintGCDetails：输出详细的GC处理日志
    - 打印gc简要信息：① -XX:+PrintGC ② -verbose:gc
- -XX:HandlePromotionFailure：是否设置空间分配担保

**说明**
在发生Minor Gc之前，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象的总空间。

- 如果大于，则此次Minor GC是安全的
- 如果小于，则虚拟机会查看-XX:HandlePromotionFailure设置值是否允许担保失败。（==JDK 7以后的规则HandlePromotionFailure可以认为就是true==）
    - 如果HandlePromotionFailure=true,那么会继续检查老年代最大可用连续空间是否大于**历次晋升到老年代的对象的平均大小**。
        - √如果大于，则尝试进行一次Minor GC,但这次Minor GC依然是有风险的；
        - √如果小于，则改为进行一次Full GC。
    - 如果HandlePromotionFailure=false,则改为进行一次Fu11 GC。

在JDK6 Update24之后（JDK7），HandlePromotionFailure参数不会再影响到虚拟机的空间分配担保策略，观察openJDK中的源码变化，虽然源码中还定义了HandlePromotionFailure参数，但是在代码中已经不会再使用它。JDK6 Update24之后的规则变为==只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC,否则将进行Full GC。==

### 9.堆是分配对象的唯一选择么（不是）

  在《深入理解Java虚拟机》中关于Java堆内存有这样一段描述：随着JIT编译期的发展与逃逸分析技术逐渐成熟，==栈上分配、标量替换优化技术==将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。
  在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果==经过逃逸分析（Escape Analysis)后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配==。这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术。
  此外，前面提到的基于OpenJDK深度定制的TaoBaoVM,其中创新的GCIH(GCinvisible heap)技术实现off-heap,将生命周期较长的Java对象从heap中移至heap外，并且GC不能管理GCIH内部的Java对象，以此达到降低GC的回收频率和提升GC的回收效率的目的。

- 如何将堆上的对象分配到栈，需要使用逃逸分析手段。
- 这是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。
- 通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。
- 逃逸分析的基本行为就是分析对象动态作用域：
    - 当一个对象在方法中被定义后，==对象只在方法内部使用，则认为没有发生逃逸==。
    - 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。
- ==如何快速的判断是否发生了逃逸分析，就看new的对象实体是否有可能在方法外被调用==

**逃逸分析**
```java
/**
 * 逃逸分析
 *
 *  如何快速的判断是否发生了逃逸分析，就看new的对象实体是否有可能在方法外被调用。
 */
public class EscapeAnalysis {

    public EscapeAnalysis obj;

    /*
    方法返回EscapeAnalysis对象，发生逃逸
     */
    public EscapeAnalysis getInstance(){
        return obj == null? new EscapeAnalysis() : obj;
    }
    /*
    为成员属性赋值，发生逃逸
     */
    public void setObj(){
        this.obj = new EscapeAnalysis();
    }
    //思考：如果当前的obj引用声明为static的？仍然会发生逃逸。

    /*
    对象的作用域仅在当前方法中有效，没有发生逃逸
     */
    public void useEscapeAnalysis(){
        EscapeAnalysis e = new EscapeAnalysis();
    }
    /*
    引用成员变量的值，发生逃逸
     */
    //比如此时去修改e的值可能会影响成员变量的值
    public void useEscapeAnalysis1(){
        EscapeAnalysis e = getInstance();
        //getInstance().xxx()同样会发生逃逸
    }
}

```

**代码优化**
使用逃逸分析，编译器可以对代码做如下优化：

1. 栈上分配：将堆分配转化为栈分配。如果一个对象在子线程中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配
2. 同步省略：如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步
3. 分离对象或标量替换：有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

**栈上分配**
- JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成之后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无须机型垃圾回收了
- 常见的栈上分配场景：给成员变量赋值、方法返回值、实例引用传递

**代码分析**
以下代码，关闭逃逸分析（-XX:-DoEscapeAnalysi），维护10000000个对象，如果开启逃逸分析，只维护少量对象（JDK7 逃逸分析默认开启）
```java
/**
 * 栈上分配测试
 * -Xmx1G -Xms1G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
 */
public class StackAllocation {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        // 查看执行时间
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User();//未发生逃逸
    }

    static class User {

    }
}

```

**同步省略**

- 线程同步的代价是相当高的，同步的后果是降低并发性和性能
- 在动态编译同步块的时候，JIT编译器可以借助逃逸分析来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫同步省略，也叫==锁消除==

```java
/**
 * 同步省略说明
 */
public class SynchronizedTest {
    public void f() {
        Object hollis = new Object();
        synchronized(hollis) {
            System.out.println(hollis);
        }
    }
    //代码中对hollis这个对象进行加锁，但是hollis对象的生命周期只在f（）方法中
    //并不会被其他线程所访问控制，所以在JIT编译阶段就会被优化掉。
    //优化为 ↓
    public void f2() {
        Object hollis = new Object();
        System.out.println(hollis);
    }
}
```

**分离对象或标量替换**
- ==标量Scalar==是指一个无法在分解成更小的数据的数据。Java中的原始数据类型就是标量
- 相对的，那些还可以分解的数据叫做==聚合量(Aggregate)==，Java中对象就是聚合量，因为它可以分解成其他聚合量和标量
- 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来替代。这个过程就是标量替换

```java
public class ScalarTest {
    public static void main(String[] args) {
        alloc();   
    }
    public static void alloc(){
        Point point = new Point(1,2);
    }
}
class Point{
    private int x;
    private int y;
    public Point(int x,int y){
        this.x = x;
        this.y = y;
    }
}
```
以上代码，经过标量替换后，就会变成
```java
public static void alloc(){
    int x = 1;
    int y = 2;
}
```
   可以看到，Point这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个标量了。那么标量替换有什么好处呢？就是可以大大减少堆内存的占用。因为一旦不需要创建对象了，那么就不再需要分配堆内存了。
   标量替换为栈上分配提供了很好的基础。

**逃逸分析小结**

- 关于逃逸分析的论文在1999年就已经发表了，但直到JDK1.6才有实现，而且这项技术到如今也并不是十分成熟的。
- 其根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程。
- 一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。
- 虽然这项技术并不十分成熟，但是它也是即时编译器优化技术中一个十分重要的手段。
- 注意到有一些观点，认为通过逃逸分析，JVM会在栈上分配那些不会逃逸的对象，这在理论上是可行的，但是取决于JVM设计者的选择。据我所知，Oracle HotspotJVM中并未这么做，这一点在逃逸分析相关的文档里已经说明，所以可以明确所有的对象实例都是创建在堆上。
- 目前很多书籍还是基于JDK7以前的版本，JDK已经发生了很大变化，intern字符串的缓存和静态变量曾经都被分配在永久代上，而永久代已经被元数据区取代。但是，intern字符串缓存和静态变量并不是被转移到元数据区，而是直接在堆上分配，所以这一点同样符合前面一点的结论：==对象实例都是分配在堆上==。
- 年轻代是对象的诞生、省长、消亡的区域，一个对象在这里产生、应用、最后被垃圾回收器收集、结束生命
- 老年代防止长生命周期对象，通常都是从Survivor区域筛选拷贝过来的Java对象。当然，也有特殊情况，我们知道普通的对象会被分配在TLAB上，如果对象较大，JVM会试图直接分配在Eden其他位置上；如果对象他打，完全无法在新生代找到足够长的连续空闲空间，JVM就会直接分配到老年代
- 当GC只发生在年轻代中，回收年轻对象的行为被称为MinorGC。当GC发生在老年代时则被称为MajorGC或者FullGC。一般的，MinorGC的发生频率要比MajorGC高很多，即老年代中垃圾回收发生的频率大大低于年轻代


## 运行时数据区3-方法区
### 1. 堆、栈、方法区的交互关系
运行时数据区结构图
![](https://user-gold-cdn.xitu.io/2020/6/7/1728de07f0a8ea3c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

堆、栈、方法区的交互关系
![](https://user-gold-cdn.xitu.io/2020/6/7/1728de0bcde73307?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 2. 方法区的理解
《Java虚拟机规范》中明确说明：‘尽管所有的方法区在逻辑上属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。’但对于HotSpotJVM而言，方法区还有一个别名叫做Non-heap（非堆），目的就是要和堆分开。
  所以，==方法区可以看作是一块独立于Java堆的内存空间。==

- 方法区（Method Area）与Java堆一样，是**各个线程共享的内存区域**
- 方法区在JVM启动时就会被创建，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的
- 方法区的大小，跟堆空间一样，可以选择固定大小或者可拓展
- **方法区的大小决定了系统可以保存多少个类**，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：java.lang.OutOfMemoryError:PermGen space 或者 java.lang,OutOfMemoryError:Metaspace，比如：

    - 加载大量的第三方jar包；
    - Tomcat部署的工程过多；
    - 大量动态生成反射类；

- 关闭JVM就会释放这个区域的内存
例，使用jvisualvm查看加载类的个数
- 在jdk7及以前，习惯上把方法区称为永久代。jdk8开始，使用元空间取代了永久代
- 本质上，方法区和永久代并不等价。仅是对hotSpot而言的。《java虚拟机规范》对如何实现方法区，不做统一要求。例如：BEA JRockit/IBM J9中不存在永久代的概念
    - 现在看来，当年使用永久代，不是好的idea。导致Java程序更容易OOM(超过-XX:MaxPermSize上限)

    **方法区在jdk7及jdk8的落地实现**
    ![](https://user-gold-cdn.xitu.io/2020/6/7/1728de1aa1e29c4c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 在jdk8中，终于完全废弃了永久代的概念，改用与JRockit、J9一样在本地内存中实现的元空间（Metaspace）来代替


- 元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代最大的区别在于：==元空间不再虚拟机设置的内存中，而是使用本地内存==

- 永久代、元空间并不只是名字变了。内部结构也调整了

- 根据《Java虚拟机规范》得规定，如果方法区无法满足新的内存分配需求时，将抛出OOM异常.

### 3.设置方法区大小与OOM
方法区的大小不必是固定的，jvm可以根据应用的需要动态调整。

**jdk7及以前**：

- 通过 -XX：PermSize来设置永久代初始分配空间。默认值是20.75M
- -XX ： MaxPermSize来设定永久代最大可分配空间。32位机器默认是64M，64位机器模式是82M
- 当JVM加载的类信息容量超过了这个值，会报异常OutOfMemoryError ： PermGen space

**jdk8及以后：**

- 元数据区大小可以使用参数一XX：MetaspaceSize和一XX ：MaxMetaspaceSize指定，替代上述原有的两个参数。
- 默认值依赖于平台。windows下，一XX：MetaspaceSize是21M，一 XX：MaxMetaspaceSize的值是一1， 即没有限制
- 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。 如果元数据区发生溢出，虚拟机一样会拋出异常OutOfMemoryError： Metaspace
- -XX：MetaspaceSize： 设置初始的元空间大小。对于一个64位的服务器端JVM来说， 其默认的一XX ：MetaspaceSize值为21MB.这就是初始的高水位线，一旦触及这个水位线，Full GC将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个高水位线将会重置。新的高水位线的值取决于GC后释放了多少元空间。如果释放的空间不足，那么在不超过MaxMetaspaceSize时，适当提高该值。如果释放空间过多，则适当降低该值。
- 如果初始化的高水位线设置过低，.上 述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到Full GC多次调用。为了避免频繁地GC，建议将- XX ：MetaspaceSize设置为一个相对较高的值。

**方法区OOM**
1. 要解决00M异常或heap space的异常，一般的手段是首先通过内存映像分析工具（如Eclipse Memory Analyzer） 对dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory 0verflow） 。
2. 如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots 的引用链。于是就能找到泄漏对象是通过怎样的路径与GCRoots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及GC Roots引用链的信息，就可以比较准确地定位出泄漏代码的位置。
3. 如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（一Xmx与一Xms） ，与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。

### 4.方法区的内部结构
![](https://user-gold-cdn.xitu.io/2020/6/7/1728de2ebc0a6dce?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

《深入理解Java虚拟机》书中对方法区存储内容描述如下：它用于存储已被虚拟机加载的==类型信息、常量、静态变量、即时编译器编译后的代码缓存==等。

![](https://user-gold-cdn.xitu.io/2020/6/7/1728de306a7ca881?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 类型信息
对每个加载的类型（ 类class、接口interface、枚举enum、注解annotation），JVM必须在方法区中存储以下类型信息：

1. 这个类型的完整有效名称（全名=包名.类名）
2. 这个类型直接父类的完整有效名（对于interface或是java. lang.Object，都没有父类）
3. 这个类型的修饰符（public， abstract， final的某个子集）
4. 这个类型直接接口的一个有序列表

#### 域信息（成员变量）
- JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。
- 域的相关信息包括：域名称、 域类型、域修饰符（public， private， protected， static， final， volatile， transient的某个子集）

#### 方法信息
JVM必须保存所有方法的以下信息，同域信息一样包括声明顺序：

- 方法名称
- 方法的返回类型（或void）
- 方法参数的数量和类型（按顺序）
- 方法的修饰符（public， private， protected， static， final， synchronized， native ， abstract的一个子集）
- 方法的字节码（bytecodes）、操作数栈、局部变量表及大小（ abstract和native 方法除外）
- 异常表（ abstract和native方法除外）
    - 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

**non-final的类变量**
- 静态变量和类关联在一起，随着类的加载而加载，他们成为类数据在逻辑上的一部分
- 类变量被类的所有实例所共享，即使没有类实例你也可以访问它。

以下代码不会报空指针异常
```java
public class MethodAreaTest {
    public static void main(String[] args) {
        Order order = null;
        order.hello();
        System.out.println(order.count);
    }
}

class Order {
    public static int count = 1;
    public static final int number = 2;


    public static void hello() {
        System.out.println("hello!");
    }
}
```

**全局常量 static final**
被声明为final的类变量的处理方法则不同，每个全局常量在编译的时候就被分配了。
**代码解析**
Order.class字节码文件，右键Open in Teminal打开控制台，使用javap -v -p Order.class > tst.txt 将字节码文件反编译并输出为txt文件,可以看到==被声明为static final的常量number在编译的时候就被赋值了，这不同于没有被final修饰的static变量count是在类加载的准备阶段被赋值==

#### 运行时常量池
##### 常量池
![](https://user-gold-cdn.xitu.io/2020/6/7/1728de46fa20ca21?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述信息外，还包含一项信息那就是常量池表（Constant Poo1 Table），包括各种字面量和对类型域和方法的符号引用。
- 一个 java 源文件中的类、接口，编译后产生一个字节码文件。而 Java 中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池这个字节码包含了指向常量池的引用。在动态链接的时候会用到运行时常量池.
- 比如如下代码，虽然只有 194 字节，但是里面却使用了 string、System、Printstream 及 Object 等结构。这里代码量其实已经很小了。如果代码多，引用到的结构会更多！

几种在常量池内存储的数据类型包括：

- 数量值
- 字符串值
- 类引用
- 字段引用
- 方法引用

**小结**
常量池，可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名，方法名，参数类型、字面量等信息。

##### 运行时常量池
- 运行时常量池（ Runtime Constant Pool）是方法区的一部分。
- 常量池表（Constant Pool Table）是Class文件的一部分，==用于存放编译期生成的各种字面量与符号引用==，这部分内容将在类加载后存放到方法区的运行时常量池中。
- 运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池。
- JVM为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过索引访问的。
- 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或者字段引用。此时不再是常量池中的符号地址了，这里换为真实地址。
    - 运行时常量池，相对于Class文件常量池的另一重要特征是：==具备动态性==。
    String.intern()
- 运行时常量池类似于传统编程语言中的符号表（symbol table） ，但是它所包含的数据却比符号表要更加丰富一些。
- 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则JVM会抛OutOfMemoryError异常。

### 5.方法区的使用举例
```java
public class MethodAreaDemo {
    public static void main(String[] args) {
        int x = 500;
        int y = 100;
        int a = x / y;
        int b = 50;
        System.out.println(a + b);
    }
}
```
main方法的字节码指令

```makefile
 0 sipush 500
 3 istore_1
 4 bipush 100
 6 istore_2
 7 iload_1
 8 iload_2
 9 idiv
10 istore_3
11 bipush 50
13 istore 4
15 getstatic #2 <java/lang/System.out>
18 iload_3
19 iload 4
21 iadd
22 invokevirtual #3 <java/io/PrintStream.println>
25 return
```

![](https://user-gold-cdn.xitu.io/2020/6/7/1728de4f75844d83?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 6.方法区的演进细节
1. 首先明确：只有HotSpot才有永久代。 BEA JRockit、IBM J9等来说，是不存在永久代的概念的。原则上如何实现方法区属于虛拟机实现细节，不受《Java虚拟机规范》管束，并不要求统一。
2. Hotspot中 方法区的变化：
- jdk1.6及之前：有永久代（permanent generation） ，静态变量存放在 永久代上
- jdk1.7：有永久代，但已经逐步“去永久代”，字符串常量池、静态变量移除，保存在堆中
- jdk1.8及之后： 无永久代，**类型信息、字段、方法、常量**保存在**本地内存的元空间**，但**字符串常量池、静态变量仍在堆**

![](https://user-gold-cdn.xitu.io/2020/6/7/1728de5521ca1aa0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2020/6/7/1728de56a783aba0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2020/6/7/1728de5864bfe4c6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**永久代为什么要被元空间替换**
- 随着Java8的到来，HotSpot VM中再也见不到永久代了。但是这并不意味着类的元数据信息也消失了。这些数据被移到了一个与堆不相连的本地内存区域，这个区域叫做元空间（ Metaspace ）。
- 由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间。
- 这项改动是很有必要的，原因有：
    1) 为永久代设置空间大小是很难确定的。 在某些场景下，如果动态加载类过多，容易产生Perm区的OOM。比如某个实际Web工程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现致命错误。 "`Exception in thread' dubbo client x.x connector’java.lang.OutOfMemoryError： PermGenspace`" 而元空间和永久代之间最大的区别在于：==元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制==。
    2) 对永久代进行调优是很困难的。

**StringTable 为什么要调整**
jdk7中将StringTable放到了堆空间中。因为永久代的回收效率很低，在full gc的时候才会触发。而full GC 是老年代的空间不足、永久代不足时才会触发。这就导致了StringTable回收效率不高。而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存.

**如何证明静态变量存在哪**
```java
/**
 * 《深入理解Java虚拟机》中的案例：
 * staticObj、instanceObj、localObj存放在哪里？
 */
public class StaticObjTest {
    static class Test {
        static ObjectHolder staticObj = new ObjectHolder();
        ObjectHolder instanceObj = new ObjectHolder();

        void foo() {
            ObjectHolder localObj = new ObjectHolder();
            System.out.println("done");
        }
    }

    private static class ObjectHolder {
    }

    public static void main(String[] args) {
        Test test = new StaticObjTest.Test();
        test.foo();
    }
}
```

- 测试发现：三个对象的数据在内存中的地址都落在Eden区范围内，所以结论：只要是对象实例必然会在Java堆中分配。
- 接着，找到了一个引用该staticObj对象的地方，是在一个java. lang . Class的实例里，并且给出了这个实例的地址，通过Inspector查看该对象实例，可以清楚看到这确实是一个 java.lang.Class类型的对象实例，里面有一个名为staticObj的实例字段：

### 7.方法区的垃圾回收
一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻。但是这部分区域的回收有时又确实是必要的。以前 Sun 公司的 Bug 列表中，曾出现过的若干个严重的 Bug 就是由于低版本的 Hotspot 虚拟机对此区域未完全回收而导致内存泄漏。
  方法区的垃圾收集主要回收两部分内容：常量池中废奔的常量和不再使用的类型

- 先来说说方法区内常量池之中主要存放的两大类常量：字面量和符号引用。 字面量比较接近Java语言层次的常量概念，如文本字符串、被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：
    1. 类和接口的全限定名
    2. 字段的名称和描述符
    3. 方法的名称和描述符
- HotSpot虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。
- 回收废弃常量与回收Java堆中的对象非常类似。
- 判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：
    1. 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
    2. 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、JSP的重加载等，否则通常是很难达成的。|】
    3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。
- Java虛拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会回收。关于是否要对类型进行回收，HotSpot虚拟机提供了一Xnoclassgc 参数进行控制，还可以使用一verbose：class以及一XX： +TraceClass一Loading、一XX：+TraceClassUnLoading查 看类加载和卸载信息
- 在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及oSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。

### 8. 总结
![](https://user-gold-cdn.xitu.io/2020/6/7/1728de69bc18c8a5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 运行时数据区4-对象的实例化内存布局与访问定位+直接内存

### 1.对象的实例化
![](https://user-gold-cdn.xitu.io/2020/6/7/1728e4cbb14c0bd3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 1.1 创建对象的方式
- new(最常见的方式)
    - 变形1 ： Xxx的静态方法
    - 变形2 ： XxBuilder/XxoxFactory的静态方法
- Class的newInstance（）：反射的方式，只能调用空参的构造器，权限必须是public
- Constructor的newInstance（Xxx）：反射的方式，可以调用空参、带参的构造器，权限没有要求
- 使用clone（） ：不调用任何构造器，当前类需要实现Cloneable接口，可选: 实现clone(), 需要 jvm 去 clone, 属于浅拷贝, 如果不实现 cloneable 接口而去调用 clone() 方法会抛异常
[clone - 失败的设计](https://www.zhihu.com/question/52490586)
- 使用反序列化：从文件中、从网络中获取一个对象的二进制流
    需要实现 Serialable 接口
- 第三方库Objenesis

#### 1.2 创建对象的步骤
1. 判断对象对应的类是否加载、链接、初始化
2. 为对象分配内存
    1. 如果内存规整一指针碰撞
    2. 如果内存不规整：
        1. 虚拟机需要维护一个列表
        2. 空闲列表分配
3. 处理并发安全问题
    1. 采用CAS配上失败重试保证更新的原子性
    2. 每个线程预先分配一块TLAB
4. 初始化分配到的空间一所有属性设置默认值，保证对象实例字段在不赋值时可以直接使用
5. 设置对象的对象头
6. 执行init方法进行初始化

##### 1) 判断对象对应的类是否加载、链接、初始化
虚拟机遇到一条new指令，首先去检查这个指令的参数能否在Metaspace的常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化。（ 即判断类元信息是否存在）。
1. 如果没有，那么在双亲委派模式下，使用当前类加载器以ClassLoader+包名+类名为Key进行查找对应的.class文件。如果没有找到文件，则抛出ClassNotFoundException异常，
2. 如果找到，则进行类加载，并生成对应的Class类对象

##### 2) 为对象分配内存
首先计算对象占用空间大小，接着在堆中划分一块内存给新对象。 如果实例成员变量是引用变量，仅分配引用变量空间即可，即4个字节大小。

1. 如果内存规整，使用指针碰撞
如果内存是规整的，那么虚拟机将采用的是指针碰撞法（BumpThePointer）来为对象分配内存。意思是所有用过的内存在一边，空闲的内存在另外一边，中间放着一个指针作为分界点的指示器，分配内存就仅仅是把指针向空闲那边挪动一段与对象大小相等的距离罢了。如果垃圾收集器选择的是Serial、ParNew这种基于压缩算法的，虚拟机采用这种分配方式。一般使用带有compact （整理）过程的收集器时，使用指针碰撞。
如果内存不规整，虚拟机需要维护一个列表，使用空闲列表分配
2. 如果内存不是规整的，已使用的内存和未使用的内存相互交错，那么虛拟机将采用的是空闲列表法来为对象分配内存。意思是虚拟机维护了一个列表，记录上哪些内存块是可用的，再分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的内容。这种分配方式成为“空闲列表（Free List） ”。

说明：选择哪种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。

给对象的属性赋值的操作：
① 属性的默认初始化
② 显式初始化
③ 代码块中初始化
④ 构造器中初始化

##### 3) 处理并发安全问题
在分配内存空间时，另外一个问题是及时保证new对象时候的线程安全性：创建对象是非常频繁的操作，虚拟机需要解决并发问题。虚拟机采用 了两种方式解决并发问题：

- CAS （ Compare And Swap ）失败重试、区域加锁：保证指针更新操作的原子性；
- TLAB把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲区，（TLAB ，Thread Local Allocation Buffer） 虚拟机是否使用TLAB，可以通过一XX：+/一UseTLAB参数来 设定。

##### 4) 初始化分配到的空间
内存分配结束，虚拟机将分配到的内存空间都初始化为零值（不包括对象头）。这一步保证了对象的实例字段在Java代码中可以不用赋初始值就可以直接使用，程序能访问到这些字段的数据类型所对应的零值。

##### 5) 设置对象的对象头
将对象的所属类（即类的元数据信息）、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现。

##### 6) 执行init方法进行初始化
在Java程序的视角看来，初始化才正式开始。初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量。 因此一般来说（由字节码中是否跟随有invokespecial指令所决定），new指令之 后会接着就是执行方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全创建出来。

##### 代码示例
```java
/**
 * 测试对象实例化的过程
 *  ① 加载类元信息 - ② 为对象分配内存 - ③ 处理并发问题  - ④ 属性的默认初始化（零值初始化）
 *  - ⑤ 设置对象头的信息 - ⑥ 属性的显式初始化、代码块中初始化、构造器中初始化
 *
 *  给对象的属性赋值的操作：
 *  ① 属性的默认初始化 - ② 显式初始化 / ③ 代码块中初始化 - ④ 构造器中初始化
 * 
 */
public class Customer{
    int id = 1001;
    String name;
    Account acct;

    {
        name = "匿名客户";
    }
    public Customer(){
        acct = new Account();
    }

}

class Account{

}
```

### 2. 对象的内存布局
#### 对象头（Header）
包含两部分

- 运行时元数据
    - 哈希值（ HashCode ）
    - GC分代年龄
    - 锁状态标志
    - 线程持有的锁
    - 偏向线程ID
    - 偏向时间戳
- 类型指针：指向类元数据的InstanceKlass，确定该对象所属的类型

说明：如果是数组，还需记录数组的长度

#### 实例数据（Instance Data）
说明：它是对象真正存储的有效信息，包括程序代码中定义的各种类型的字段（包括从父类继承下来的和本身拥有的字段） 规则：

- 相同宽度的字段总被分配在一起
- 父类中定义的变量会出现在子类之前
- 如果CompactFields参数为true（默认为true），子类的窄变量可能插入到父类变量的空隙

#### 对齐填充（Padding）
不是必须的，也没特别含义，仅仅起到占位符作用

#### 小结
```java
public class CustomerTest {
    public static void main(String[] args) {
        Customer cust = new Customer();
    }
}
```

![](https://user-gold-cdn.xitu.io/2020/6/7/1728e4d06baca2bf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


### 3.对象的访问定位
JVM是如何通过栈帧中的对象引|用访问到其内部的对象实例的呢？-> 定位,通过栈上reference访问

![](https://user-gold-cdn.xitu.io/2020/6/7/1728e4d419936de2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

对象访问的主要方式有两种
- 句柄访问
    ![](https://user-gold-cdn.xitu.io/2020/6/7/1728e4d9d0cb81f4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 直接指针(HotSpot采用)
    ![](https://user-gold-cdn.xitu.io/2020/6/7/1728e4de9c0c7bc9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


## 执行引擎（Execution Engine）
### 执行引擎概述
- 执行引擎是Java虚拟机的核心组成部分之一
- 虚拟机是一个相对于“物理机”的概念，这两种机器都有代码执行能力，其区别是物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层面上的，而虚拟机的执行引擎则是由软件自行实现的，因此可以不受物理条件制约地定制指令集与执行引擎的结构体系，能够执行那些不被硬件直接支持的指令集格式。
- JVM的主要任务是==负责装载字节码到其内部==，但字节码并不能够直接运行在操作系统之上，因为字节码指令并非等价于本地机器指令，它内部包含的仅仅只是一些能够被JVM锁识别的字节码指令、符号表和其他辅助信息
- 那么，如果想让一个Java程序运行起来、执行引擎的任务就是==将字节码指令解释/编译为对应平台上的本地机器指令才可以==。简单来说，JVM中的执行引擎充当了将高级语言翻译为机器语言的译者.
- 执行引擎的工作过程
    - 从外观上来看，所有的Java虚拟机的执行引擎输入、输出都是一致的：输入的是字节码二进制流，处理过程是字节码解析执行的等效过程，输出的是执行结果。
    1）执行引擎在执行的过程中究竟需要执行什么样的字节码指令完全依赖于PC寄存器。
    2）每当执行完一项指令操作后，PC寄存器就会更新下一条需要被执行的指令地址。
    3）当然方法在执行的过程中，执行引擎有可能会通过存储在局部变量表中的对象引用准确定位到存储在Java堆区中的对象实例信息，以及通过对象头中的元数据指针定位到目标对象的类型信息。
![](https://user-gold-cdn.xitu.io/2020/6/8/1729333fda0586ed?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Java代码编译和执行过程
大部分的程序代码转换成物理机的目标代码或虚拟机能执行的指令集之前，都需要经过下面图中的各个步骤：
![](https://user-gold-cdn.xitu.io/2020/6/8/17293345b33057a0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Java代码编译是由Java源码编译器来完成，流程图如下所示：
![](https://user-gold-cdn.xitu.io/2020/6/8/172933480ac44fb8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Java字节码的执行是由JVM执行引擎来完成，流程图如下所示：
![](https://user-gold-cdn.xitu.io/2020/6/8/1729334a1a335e4c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**什么是解释器（ Interpreter），什么是JIT编译器？**

**解释器**：当Java虚拟机启动时会根据预定义的规范对字节码采用逐行解释的方式执行，将每条字节码文件中的内容“翻译”为对应平台的本地机器指令执行。
**JIT （Just In Time Compiler）编译器（即时编译器）**：就是虚拟机将源代码直接编译成和本地机器平台相关的机器语言。

**为什么说Java是半编译半解释型语言？**
JDK1.0时代，将Java语言定位为“解释执行”还是比较准确的。再后来，Java也发展出可以直接生成本地代码的编译器。
现在JVM在执行Java代码的时候，通常都会将解释执行与编译执行二者结合起来进行。
![](https://user-gold-cdn.xitu.io/2020/6/8/172933531eb0cea7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 解释器
JVM设计者们的初衷仅仅只是单纯地为了==满足Java程序实现跨平台特性==，因此避免采用静态编译的方式直接生成本地机器指令，从而诞生了实现解释器在运行时采用逐行解释字节码执行程序的想法。
![](https://user-gold-cdn.xitu.io/2020/6/8/172933694cc12661?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
- 解释器真正意义上所承担的角色就是一个运行时“翻译者”，将字节码文件中的内容“翻译”为对应平台的本地机器指令执行。
- 当一条字节码指令被解释执行完成后，接着再根据PC寄存器中记录的下一条需要被执行的字节码指令执行解释操作。

在Java的发展历史里，一共有两套解释执行器，即古老的==字节码解释器==、现在普遍使用的==模板解释器==。
- 字节码解释器在执行时通过纯软件代码模拟字节码的执行，效率非常低下。
- 而模板解释器将每一条字节码和一个模板函数相关联，模板函数中直接产生这条字节码执行时的机器码，从而很大程度上提高了解释器的性能。
    - 在HotSpot VM中，解释器主要由Interpreter模块和Code模块构成。
        - Interpreter模块：实现了解释器的核心功能
        - Code模块：用于管理HotSpot VM在运行时生成的本地机器指令

**现状**
- 由于解释器在设计和实现上非常简单，因此除了Java语言之外，还有许多高级语言同样也是基于解释器执行的，比如Python、 Perl、Ruby等。但是在今天，基于解释器执行已经沦落为低效的代名词，并且时常被一些C/C+ +程序员所调侃。
- 为了解决这个问题，JVM平台支持一种叫作即时编译的技术。即时编译的目的是避免函数被解释执行，而是将整个函数体编译成为机器码，每次函数执行时，只执行编译后的机器码即可，这种方式可以使执行效率大幅度提升。
- 不过无论如何，基于解释器的执行模式仍然为中间语言的发展做出了不可磨灭的贡献。

### JIT编译器
#### HotSpot VM 为何解释器与JIT编译器共存
java代码的执行分类：

- 第一种是将源代码编译成字节码文件，然后再运行时通过解释器将字节码文件转为机器码执行
- 第二种是编译执行（直接编译成机器码）。现代虚拟机为了提高执行效率，会使用即时编译技术（JIT,Just In Time）将方法编译成机器码后再执行

  HotSpot VM是目前市面上高性能虛拟机的代表作之一。它采用==解释器与即时编译器并存的架构==。在Java虛拟机运行时，解释器和即时编译器能够相互协作，各自取长补短，尽力去选择最合适的方式来权衡编译本地代码的时间和直接解释执行代码的时间。
  在今天，Java程序的运行性能早已脱胎换骨，已经达到了可以和C/C++程序一较高下的地步。

**解释器依然存在的必要性**
有些开发人员会感觉到诧异，既然HotSpotVM中已经内置JIT编译器了，那么为什么还需要再使用解释器来“拖累”程序的执行性能呢？比如JRockit VM内部就不包含解释器，字节码全部都依靠即时编译器编译后执行。

- 当程序启动后，解释器可以马上发挥作用，省去编译的时间，立即执行。 编译器要想发挥作用，把代码编译成本地代码，需要一定的执行时间。但编译为本地代码后，执行效率高。
- 尽管JRockitVM中程序的执行性能会非常高效，但程序在启动时必然需要花费更长的时间来进行编译。对于服务端应用来说，启动时间并非是关注重点，但对于那些看中启动时间的应用场景而言，或许就需要采用解释器与即时编译器并存的架构来换取一一个平衡点。在此模式下，==当Java虚拟器启动时，解释器可以首先发挥作用，而不必等待即时编译器全部编译完成后再执行，这样可以省去许多不必要的编译时间。随着时间的推移，编译器发挥作用，把越来越多的代码编译成本地代码，获得更高的执行效率。==

**HotSpot JVM的执行方式**
当虛拟机启动的时候，解释器可以首先发挥作用，而不必等待即时编译器全部编译完成再执行，这样可以省去许多不必要的编译时间。并且随着程序运行时间的推移，即时编译器逐渐发挥作用，==根据热点探测功能，将有价值的字节码编译为本地机器指令，以换取更高的程序执行效率。==

**案例**
注意解释执行与编译执行在线上环境微妙的辩证关系。机器在热机状态可以承受的负载要大于冷机状态。如果以热机状态时的流量进行切流，可能使处于冷机状态的服务器因无法承载流量而假死。

在生产环境发布过程中，以分批的方式进行发布，根据机器数量划分成多个批次，每个批次的机器数至多占到整个集群的1/8。曾经有这样的故障案例：某程序员在发布平台进行分批发布，在输入发布总批数时，误填写成分为两批发布。如果是热机状态，在正常情况下一半的机器可以勉强承载流量，但由于刚启动的JVM均是解释执行，还没有进行热点代码统计和JIT动态编译，导致机器启动之后，当前1/2发布成功的服务器马上全部宕机，此故障说明了JIT的存在。一阿里团队

#### JIT编译器
**概念解释**
- Java 语言的“编译器” 其实是一段“不确定”的操作过程，因为它可能是指一个==前端编译器==（其实叫“编译器的前端” 更准确一些）把.java文件转变成.class文件的过程；
- 也可能是指虚拟机的==后端运行期编译器==（JIT 编译器，Just In Time Compiler）把字节码转变成机器码的过程。
- 还可能是指使用==静态提前编译器==（AOT 编译器，Ahead Of Time Compiler）直接把. java文件编译成本地机器代码的过程。

前端编译器： Sun的Javac、 Eclipse JDT中的增量式编译器（ECJ）
JIT编译器： HotSpot VM的C1、C2编译器。
AOT编译器： GNU Compiler for the Java （GCJ） 、Excelsior JET。

**热点代码及探测方式**
当然是否需要启动JIT编译器将字节码直接编译为对应平台的本地机器指令，则需要根据代码被调用执行的频率而定。关于那些需要被编译为本地代码的字节码，也被称之为“热点代码” ，JIT编译器在运行时会针对那些频繁被调用的“热点代码”做出深度优化，将其直接编译为对应平台的本地机器指令，以此提升Java程序的执行性能。

- 一个被多次调用的方法，或者是一个方法体内部循环次数较多的循环体都可以被称之为“热点代码”，因此都可以通过JIT编译器编译为本地机器指令。由于这种编译方式发生在方法的执行过程中，因此也被称之为栈上替换，或简称为OSR （On StackReplacement）编译。
- 一个方法究竟要被调用多少次，或者一个循环体究竟需要执行多少次循环才可以达到这个标准？必然需要一个明确的阈值，JIT编译器才会将这些“热点代码”编译为本地机器指令执行。这里主要依靠==热点探测功能==。
- ==目前HotSpot VM所采用的热点探测方式是基于计数器的热点探测==。
- 采用基于计数器的热点探测，HotSpot VM将会为每一个 方法都建立2个不同类型的计数器，分别为方法调用计数器（Invocation Counter） 和回边计数器（BackEdge Counter） 。
    - 方法调用计数器用于统计方法的调用次数
    - 回边计数器则用于统计循环体执行的循环次数

**方法调用计数器**
- 这个计数器就用于统计方法被调用的次数，它的默认阈值在Client 模式 下是1500 次，在Server 模式下是10000 次。超过这个阈值，就会触发JIT编译。
- 这个阈值可以通过虚拟机参数一XX ：CompileThreshold来人为设定。
- 当一个方法被调用时， 会先检查该方法是否存在被JIT编译过的版本，如 果存在，则优先使用编译后的本地代码来执行。如果不存在已被编译过的版本，则将此方法的调用计数器值加1，然后判断方法调用计数器与回边计数器值之和是否超过方法调用计数器的阈值。如果已超过阈值，那么将会向即时编译器提交一个该方法的代码编译请求。

![](https://user-gold-cdn.xitu.io/2020/6/8/172933785afec215?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**热度衰减**
- 如果不做任何设置，方法调用计数器统计的并不是方法被调用的绝对次数，而是一一个相对的执行频率，即一段时间之内方法被调用的次数。当超过一定的时间限度， 如果方法的调用次数仍然不足以让它提交给即时编译器编译，那这个方法的调用计数器就会被减少一半，这个过程称为方法调用计数器热度的衰减（Counter Decay） ，而这段时间就称为此方法统计的半衰周期（Counter Half Life Time）。
- 进行热度衰减的动作是在虚拟机进行垃圾收集时顺便进行的，可以使用虚拟机参数 -XX：-UseCounterDecay来关闭热度衰减，让方法计数器统计方法调用的绝对次数，这样，只要系统运行时间足够长，绝大部分方法都会被编译成本地代码。
- 另外， 可以使用-XX： CounterHalfLifeTime参数设置半衰周期的时间，单位是秒。

**回边计数器**
它的作用是统计一个方法中循环体代码执行的次数，在字节码中遇到控制流向后跳转的指令称为“回边” （Back Edge）。显然，建立回边计数器统计的目的就是为了触发OSR编译。

## 字符串常量池StringTable

### 1.String的基本特性
- String：字符串，使用一对""引起来表示。
    - String sl = "hello"；//字面量的定义方式
    - String s2 = new String（"hello"） ；
- String声明为final的， 不可被继承
- String实现了Serializable接口：表示字符串是支持序列化的。 实现了Comparable接口：表示String可以比较大小
- ==String在jdk8及以前内部定义了final char[],value用于存储字符串数据。jdk9时改为byte[]==
byte[] 加上编码标记，节约了一些空间。StringBuffer和StringBuilder也做了一些修改
- String：代表不可变的字符序列。简称：不可变性。
    - 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
    - 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
    - 当调用String的replace（）方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
- 通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中。
- ==字符串常量池中是不会存储相同内容的字符串的。==
    - String的String Pool 是一个固定大小的Hashtable，默认值大小长度是1009。如果放进StringPool的String非常多， 就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用String. intern时性能会大幅下降。
    - 使用一XX： StringTableSize可设置StringTable的长度
    - 在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。StringTableSize设 置没有要求
    - 在jdk7中，StringTable的长度默认值是60013
    - jdk8开始,1009是StringTable长度可设置的最小值

### 2.String的内存分配
- 在Java语言中有8种基本数据类型和一种比较特殊的类型String。这些 类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种常量池的概念。
- 常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量 池都是系统协调的，String类 型的常量池比较特殊。它的主要使用方法有两种。
    - 直接使用双引号声明出来的String对象会直接存储在常量池中。
        - 比如： String info = "abc" ；
    - 如果不是用双引号声明的String对象，可以使用String提供的intern（）方法。这个后面重点谈
- Java 6及以前，字符串常量池存放在永久代。
- Java 7中Oracle的工程师对字符串池的逻辑做了很大的改变，即将字符串常量池的位置调整到Java堆内。
    - 所有的字符串都保存在堆（Heap）中，==也就是说字符串里面存储的是字符串的引用==, 和其他普通对象一样，这样可以让你在进行调优应用时仅需要调整堆大小就可以了。
    - 字符串常量池概念原本使用得比较多，但是这个改动使得我们有足够的理由让我们重新考虑在Java 7中使用String. intern（）。
- Java8元空间，字符串常量在堆

**StringTable为什么要调整**
①永久代permSize默认比较小;
②永久代的垃圾回收频率低;

### 3.String的基本操作
![](https://user-gold-cdn.xitu.io/2020/6/16/172bac675fab2b1f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 4.字符串拼接操作
1. ==常量与常量的拼接结果在常量池，原理是编译期优化==
2. 常量池中不会存在相同内容的常量。
3. ==只要其中有一个是变量，结果就在堆中==。变量拼接的原理是StringBuilder
4. 如果拼接的结果调用intern（）方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。

```java
 @Test
    public void test1(){
        String s1 = "a" + "b" + "c";//编译期优化：等同于"abc"
        String s2 = "abc"; //"abc"一定是放在字符串常量池中，将此地址赋给s2
        /*
         * 最终.java编译成.class,再执行.class
         * String s1 = "abc";
         * String s2 = "abc"
         */
        System.out.println(s1 == s2); //true
        System.out.println(s1.equals(s2)); //true
    }

    @Test
    public void test2(){
        String s1 = "javaEE";
        String s2 = "hadoop";

        String s3 = "javaEEhadoop";
        String s4 = "javaEE" + "hadoop";//编译期优化
        //如果拼接符号的前后出现了变量，则相当于在堆空间中new String()，具体的内容为拼接的结果：javaEEhadoop
        String s5 = s1 + "hadoop";
        String s6 = "javaEE" + s2;
        String s7 = s1 + s2;

        System.out.println(s3 == s4);//true
        System.out.println(s3 == s5);//false
        System.out.println(s3 == s6);//false
        System.out.println(s3 == s7);//false
        System.out.println(s5 == s6);//false
        System.out.println(s5 == s7);//false
        System.out.println(s6 == s7);//false
        //intern():判断字符串常量池中是否存在javaEEhadoop值，如果存在，则返回常量池中javaEEhadoop的地址；
        //如果字符串常量池中不存在javaEEhadoop，则在常量池中加载一份javaEEhadoop，并返回次对象的地址。
        String s8 = s6.intern();
        System.out.println(s3 == s8);//true
    }
```

**字符串拼接**
```java
@Test
    public void test3(){
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        /*
        如下的s1 + s2 的执行细节：(变量s是我临时定义的）
        ① StringBuilder s = new StringBuilder();
        ② s.append("a")
        ③ s.append("b")
        ④ s.toString()  --> 约等于 new String("ab")

        补充：在jdk5.0之后使用的是StringBuilder,
        在jdk5.0之前使用的是StringBuffer
         */
        String s4 = s1 + s2;//
        System.out.println(s3 == s4);//false
    }

    /*
    1. 字符串拼接操作不一定使用的是StringBuilder!
       如果拼接符号左右两边都是字符串常量或常量引用，则仍然使用编译期优化，即非StringBuilder的方式。
    2. 针对于final修饰类、方法、基本数据类型、引用数据类型的量的结构时，能使用上final的时候建议使用上。
     */
    @Test
    public void test4(){
        final String s1 = "a";
        final String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2;
        System.out.println(s3 == s4);//true
    }
    
    //练习：
    @Test
    public void test5(){
        String s1 = "javaEEhadoop";
        String s2 = "javaEE";
        String s3 = s2 + "hadoop";
        System.out.println(s1 == s3);//false

        final String s4 = "javaEE";//s4:常量
        String s5 = s4 + "hadoop";
        System.out.println(s1 == s5);//true

    }
```

**拼接操作与append的效率对比**
append效率要比字符串拼接高很多

```java
/*
    体会执行效率：通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式！
    详情：① StringBuilder的append()的方式：自始至终中只创建过一个StringBuilder的对象
          使用String的字符串拼接方式：创建过多个StringBuilder和String的对象
         ② 使用String的字符串拼接方式：内存中由于创建了较多的StringBuilder和String的对象，内存占用更大；如果进行GC，需要花费额外的时间。

     改进的空间：在实际开发中，如果基本确定要前前后后添加的字符串长度不高于某个限定值highLevel的情况下,建议使用构造器实例化：
               StringBuilder s = new StringBuilder(highLevel);//new char[highLevel]
     */
    @Test
    public void test6(){

        long start = System.currentTimeMillis();

//        method1(100000);//4014
        method2(100000);//7

        long end = System.currentTimeMillis();

        System.out.println("花费的时间为：" + (end - start));
    }

    public void method1(int highLevel){
        String src = "";
        for(int i = 0;i < highLevel;i++){
            src = src + "a";//每次循环都会创建一个StringBuilder、String
        }
//        System.out.println(src);

    }

    public void method2(int highLevel){
        //只需要创建一个StringBuilder
        StringBuilder src = new StringBuilder();
        for (int i = 0; i < highLevel; i++) {
            src.append("a");
        }
//        System.out.println(src);
    }
```

### 5.intern()的使用
如果不是用双引号声明的String对象，可以使用String提供的intern方法： ==intern方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。若存在则不做操作==

- 比如： String myInfo = new String("I love u").intern()；
也就是说，如果在任意字符串上调用String. intern方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下 列表达式的值必定是true： （"a" + "b" + "c"）.intern（）== "abc";
通俗点讲，Interned String就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池（String Intern Pool）。

**new String("ab")会创建几个对象,new String("a")+new String("b")呢**

- new String("ab")会创建几个对象？看字节码，就知道是两个。
    - 一个对象是：new关键字在堆空间创建的
    - 另一个对象是：字符串常量池中的对象"ab"。 字节码指令：ldc
![](https://user-gold-cdn.xitu.io/2020/6/16/172bac7ac131c390?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- new String("a") + new String("b")呢？
    - 对象1：new StringBuilder()
    - 对象2： new String("a")
    - 对象3： 常量池中的"a"
    - 对象4： new String("b")
    - 对象5： 常量池中的"b"
    - 对象6 ：new String("ab")

==强调一下，toString()的调用，在字符串常量池中，没有生成"ab"==

**关于String.intern()的面试题**
```java
/**
 * 如何保证变量s指向的是字符串常量池中的数据呢？
 * 有两种方式：
 * 方式一： String s = "shkstart";//字面量定义的方式
 * 方式二： 调用intern()
 *         String s = new String("shkstart").intern();
 *         String s = new StringBuilder("shkstart").toString().intern();
 *
 */
public class StringIntern {
    public static void main(String[] args) {
        String s = new String("1");
        String s1 = s.intern();//调用此方法之前，字符串常量池中已经存在了"1"
        String s2 = "1";
        //s  指向堆空间"1"的内存地址
        //s1 指向字符串常量池中"1"的内存地址
        //s2 指向字符串常量池已存在的"1"的内存地址  所以 s1==s2
        System.out.println(s == s2);//jdk6：false   jdk7/8：false
        System.out.println(s1 == s2);//jdk6: true   jdk7/8：true
        System.out.println(System.identityHashCode(s));//491044090
        System.out.println(System.identityHashCode(s1));//644117698
        System.out.println(System.identityHashCode(s2));//644117698

        //s3变量记录的地址为：new String("11")
        String s3 = new String("1") + new String("1");
        //执行完上一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！

        //在字符串常量池中生成"11"。如何理解：jdk6:创建了一个新的对象"11",也就有新的地址。
        //         jdk7:此时常量中并没有创建"11",而是创建一个指向堆空间中new String("11")的地址
        //此时并没有������������s3
        s3.intern();
        //s4变量记录的地址：使用的是上一行代码代码执行时，在常量池中生成的"11"的地址
        String s4 = "11";
        System.out.println(s3 == s4);//jdk6：false  jdk7/8：true
    }

}
```

![](https://user-gold-cdn.xitu.io/2020/6/16/172bac960ae887a7?imageslim)
![](https://user-gold-cdn.xitu.io/2020/6/16/172bac97d066c47d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1 )

**总结String的intern（）的使用**
- jdk1.6中，将这个字符串对象尝试放入串池。
    - 如果字符串常量池中有，则并不会放入。返回已有的串池中的对象的地址
    - 如果没有，会把此对象复制一份，放入串池，并返回串池中的对象地址
- Jdk1.7起，将这个字符串对象尝试放入串池。
    - 如果字符串常量池中有，则并不会放入。返回已有的串池中的对象的地址
    - 如果没有，则会把对象的引用地址复制一份，放入串池，并返回串池中的引用地址

**练习**
```java
public class StringExer1 {
    public static void main(String[] args) {
        //String x = "ab";
        String s = new String("a") + new String("b");//new String("ab")
        //在上一行代码执行完以后，字符串常量池中并没有"ab"

        String s2 = s.intern();//jdk6中：在串池中创建一个字符串"ab"
                               //jdk8中：串池中没有创建字符串"ab",而是创建一个引用，指向new String("ab")，将此引用返回

        System.out.println(s2 == "ab");//jdk6:true  jdk8:true
        System.out.println(s == "ab");//jdk6:false  jdk8:true
    }
}
```
**jdk6**
![](https://user-gold-cdn.xitu.io/2020/6/16/172bac9a78f97a95?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```java
public class StringExer2 {
    public static void main(String[] args) {
        String s1 = new String("ab");//执行完以后，会在字符串常量池中会生成"ab"
//        String s1 = new String("a") + new String("b");////执行完以后，不会在字符串常量池中会生成"ab"
        s1.intern();
        String s2 = "ab";
        System.out.println(s1 == s2); //false
    }
}
```

## 垃圾回收1-概述+相关算法
### 为什么需要GC
- 对于高级语言来说，一个基本认知是如果不进行垃圾回收，内存迟早都会被消耗完，因为不断地分配内存空间而不进行回收，就好像不停地生产生活垃圾而从来不打扫一样。
- 除了释放没用的对象，垃圾回收也可以清除内存里的记录碎片。碎片整理将所占用的堆内存移到堆的一端，以便JVM 将整理出的内存分配给新的对象。
- 随着应用程序所应付的业务越来越庞大、复杂，用户越来越多，没有GC就不能保证应用程序的正常进行。而经常造成STW的GC又跟不上实际的需求，所以才会不断地尝试对GC进行优化。

### 垃圾回收相关算法
#### 垃圾标记阶段:对象存活判断
- 在堆里存放着几乎所有的Java对象实例，在GC执行垃圾回收之前，首先需要区分出内存中哪些是存活对象，哪些是已经死亡的对象。只有被标记为己经死亡的对象，GC才会在执行垃圾回收时，释放掉其所占用的内存空间，因此这个过程我们可以称为垃圾标记阶段。
- 那么在JVM中究竟是如何标记一个死亡对象呢？简单来说，当一个对象已经不再被任何的存活对象继续引用时，就可以宣判为已经死亡。
- 判断对象存活一般有两种方式：==引用计数算法==和==可达性分析算法==。

#### 1 标记阶段:法1_引用计数法 (java没有采用)
- 引用计数算法（Reference Counting）比较简单，对每个对象保存一个整型 的引用计数器属性。用于记录对象被引用的情况。
- 对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1；当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，即表示对象A不可能再被使用，可进行回收。
- 优点：实现简单，垃圾对象便于辨识；判定效率高，回收没有延迟性。
- 缺点：
    - 它需要单独的字段存储计数器，这样的做法增加了存储空间的开销。
    - 每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了时间开销。
    - 引用计数器有一个严重的问题，即无法处理循环引用的情况。这是一 条致命缺陷，导致==在Java的垃圾回收器中没有使用这类算法==。

![](https://user-gold-cdn.xitu.io/2020/6/18/172c6e82e8f9fd95?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 2 标记阶段:法2_可达性分析算法
- 相对于引用计数算法而言，可达性分析算法不仅同样具备实现简单和执行高 效等特点，更重要的是该算法可以有效地解决在引用计数算法中循环引用的问题，防止内存泄漏的发生。
- 相较于引用计数算法，这里的可达性分析就是Java、C#选择的。这种类型的垃圾收集通常也叫作追踪性垃圾收集（Tracing GarbageCollection）。
- 所谓"GC Roots"根集合就是一组必须活跃的引用。
基本思路：
    - 可达性分析算法是以根对象集合(GCRoots）为起始点，按照从上至下的方式搜索被根对象集合所连接的目标对象是否可达。
    - 使用可达性分析算法后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为引用链（Reference Chain）
    - 如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象。
    - 在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象。

##### GC Roots
- 虚拟机栈中引用的对象
    - 比如：各个线程被调用的方法中使用到的参数、局部变量等。
- 本地方法栈内JNI（通常说的本地方法）引用的对象
- 方法区中类静态属性引用的对象
    - 比如：Java类的引用类型静态变量
- 方法区中常量引用的对象
    - 比如：字符串常量池（string Table） 里的引用
- 所有被同步锁synchronized持有的对象
- Java虚拟机内部的引用。
    - 基本数据类型对应的Class对象，一些常驻的异常对象（如： NullPointerException、OutOfMemoryError） ，系统类加载器。
- 反映java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等
- 除了这些固定的GCRoots集合以外，根据用户所选用的垃圾收集器以及当 前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完整GC Roots集合。比如：分代收集和局部回收（Partial GC）。
    - 如果只针对Java堆中的某一块区域进行垃圾回收（比如：典型的只针 对新生代），必须考虑到内存区域是虚拟机自己的实现细节，更不是孤立封闭的，这个区域的对象完全有可能被其他区域的对象所引用，这时候就需要一.并将关联的区域对象也加入GC Roots集合中去考虑，才能保证可达性分析的准确性。==Remembered Set(详情查看 G1)==

小技巧：由于Root采用栈方式存放变量和指针，所以如果一个指针，它保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个Root

**注意**
- 如果要使用可达性分析算法来判断内存是否可回收，那么分析工作必须在 一个能保障一致性的快照中进行。这点不满足的话分析结果的准确性就无法保证。
- 这点也是导致GC进行时必须“StopTheWorld"的一个重要原因。
    - 即使是号称（几乎）不会发生停顿的CMS收集器中，枚举根节点时也是必须要停顿的

#### 3 对象的finalization机制
- Java语言提供了对象终止（finalization）机制来允许开发人员提供对象被销毁之前的自定义处理逻辑。
- 当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的finalize（）方法。
- finalize（）方法允许在子类中被重写，用于在对象被回收时进行资源释放。通常在这个方法中进行一些资源释放和清理的工作，比如关闭文件、套接字和数据库连接等。
- 应该交给垃圾回收机制调用。理由包括下面三点：永远不要主动调用某个对象的finalize （）方法
    - 在finalize（） 时可能会导致对象复活。
    - finalize（）方法的执行时间是没有保障的，它完全由Gc线程决定，极端情况下，若不发生GC，则finalize（） 方法将没有执行机会。
    - 一个糟糕的finalize （）会严重影响GC的性能。
- 从功能上来说，finalize（）方法与C++ 中的析构函数比较相似，但是Java采用的是基于垃圾回收器的自动内存管理机制，所以finalize（）方法在本质，上不同于C++ 中的析构函数。

**对象是否"死亡"**
- 由于finalize （）方法的存在，==虚拟机中的对象一般处于三种可能的状态==
- 如果从所有的根节点都无法访问到某个对象，说明对象己经不再使用了。一般来说，此对象需要被回收。但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。==一个无法触及的对象有可能在某一个条件下“复活”自己==，如果这样，那么对它的回收就是不合理的，为此，定义虚拟机中的对象可能的三种状态。如下：
    - ==可触及的==：从根节点开始，可以到达这个对象。
    - ==可复活的==：对象的所有引用都被释放，但是对象有可能在finalize（）中复活。
    - ==不可触及的==：对象的finalize（）被调用，并且没有复活，那么就会进入不可触及状态。不可触及的对象不可能被复活，因为finalize（） 只会被调用一一次。
- 以上3种状态中，是由于finalize（）方法的存在，进行的区分。只有在对象不可触及时才可以被回收。 判定是否可以回收具体过程 判定一个对象objA是否可回收，至少要经历两次标记过程：
    1. 如果对象objA到GC Roots没有引用链，则进行第一 次标记。
    2. 进行筛选，判断此对象是否有必要执行finalize（）方法
        1. 如果对 象objA没有重写finalize（）方法，或者finalize （）方法已经被虚拟机调用过，则虚拟机视为“没有必要执行”，objA被判定为不可触及的。
        2. 如果对象objA重写了finalize（）方法，且还未执行过，那么objA会被插入到F一Queue队列中，由一个虚拟机自动创建的、低优先级的Finalizer线程触发其finalize（）方法执行。
        3. finalize（）方法是对象逃脱死亡的最后机会，稍后Gc会对F一Queue队列中的对象进行第二次标记。如果objA在finalize（）方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，objA会被移出“即将回收”集合。之后，对象会再次出现没有引用存在的情况。在这个情况下，finalize方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，一个对象的finalize方法只会被调用一次。

```java
/**
 * 测试Object类中finalize()方法，即对象的finalization机制。
 *
 */
public class CanReliveObj {
    public static CanReliveObj obj;//类变量，属于 GC Root


    //此方法只能被调用一次
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("调用当前类重写的finalize()方法");
        obj = this;//当前待回收的对象在finalize()方法中与引用链上的一个对象obj建立了联系
    }


    public static void main(String[] args) {
        try {
            obj = new CanReliveObj();
            // 对象第一次成功拯救自己
            obj = null;
            System.gc();//调用垃圾回收器
            System.out.println("第1次 gc");
            // 因为Finalizer线程优先级很低，暂停2秒，以等待它
            Thread.sleep(2000);
            if (obj == null) {
                System.out.println("obj is dead");
            } else {
                System.out.println("obj is still alive");
            }
            System.out.println("第2次 gc");
            // 下面这段代码与上面的完全相同，但是这次自救却失败了
            obj = null;
            System.gc();
            // 因为Finalizer线程优先级很低，暂停2秒，以等待它
            Thread.sleep(2000);
            if (obj == null) {
                System.out.println("obj is dead");
            } else {
                System.out.println("obj is still alive");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### 5 清除阶段:法1_标记-清除算法
当成功区分出内存中存活对象和死亡对象后，GC接下来的任务就是执行垃圾回收，释放掉无用对象所占用的内存空间，以便有足够的可用内存空间为新对象分配内存.
  目前在JVM中比较常见的三种垃圾收集算法是标记一清除算法（ Mark一Sweep）、复制算法（Copying）、标记一压缩算法（Mark一Compact）

**执行过程：**
当堆中的有效内存空间（available memory） 被耗尽的时候，就会停止整个程序（也被称为stop the world），然后进行两项工作，第一项则是标记，第二项则是清除。

- 标记： Collector从引用根节点开始遍历，标记所有被引用的对象。==一般是在对象的Header中记录为可达对象==。
- 清除： Collector对堆 内存从头到尾进行线性的遍历，如果发现某个对象在其Header中没有标记为可达对象，则将其回收。

![](https://user-gold-cdn.xitu.io/2020/6/18/172c65a088d91e05?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**缺点**
- 效率不算高
- 在进行Gc的时候，需要停止整个应用程序，导致用户体验差
- ==这种方式清理出来的空闲内存是不连续的，产生内存碎片==。需要维护一个空闲列表

#### 6 清除阶段:法2_复制算法
为了解决标记一清除算法在垃圾收集效率方面的缺陷，M.L.Minsky于1963年发表了著名的论文，“ 使用双存储区的Li sp语言垃圾收集器CALISP Garbage Collector Algorithm Using SerialSecondary Storage ）”。M.L. Minsky在该论文中描述的算法被人们称为复制（Copying）算法，它也被M. L.Minsky本人成功地引入到了Lisp语言的一个实现版本中。

**核心思想：**
将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在.使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收。
堆中S0和S1使用的就是复制算法
![](https://user-gold-cdn.xitu.io/2020/6/18/172c65aa92730f66?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
**优点：**
- 没有标记和清除过程，实现简单，运行高效
- 复制过去以后保证空间的连续性，不会出现“碎片”问题。

**缺点：**
- 此算法的缺点也是很明显的，就是需要两倍的内存空间。
- 对于G1这种分拆成为大量region的GC，复制而不是移动，意味着GC需要维护region之间对象引用关系，不管是内存占用或者时间开销也不小。
- 特别的 如果系统中的垃圾对象很多，复制算法不会很理想,复制算法需要复制的存活对象数量并不会太大，或者说非常低才行。

**应用场景：**
在新生代，对常规应用的垃圾回收，一次通常可以回收70% - 99%的内存空间。回收性价比很高。所以现在的商业虚拟机都是用这种收集算法回收新生代。
![](https://user-gold-cdn.xitu.io/2020/6/18/172c65b23eed0ab9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 7 清除阶段:法3_标记-压缩(整理,Mark-Compact)算法
复制算法的高效性是建立在存活对象少、垃圾对象多的前提下的。这种情况在新生代经常发生，但是在老年代，更常见的情况是大部分对象都是存活对象。如果依然使用复制算法，由于存活对象较多，复制的成本也将很高。因此，基于老年代垃圾回收的特性，需要使用其他的算法。
  标记一清除算法的确可以应用在老年代中，但是该算法不仅执行效率低下，而且在执行完内存回收后还会产生内存碎片，所以JVM的设计者需要在此基础之上进行改进。==标记一压缩（Mark一Compact） 算法由此诞生==。
  1970年前后，G. L. Steele 、C. J. Chene和D.S. Wise 等研究者发布标记一压缩算法。在许多现代的垃圾收集器中，人们都使用了标记一压缩算法或其改进版本。

**执行过程：**
- 第一阶段和标记一清除算法一样，从根节点开始标记所有被引用对象.
- 第二阶段将所有的存活对象压缩到内存的一端，按顺序排放。
- 之后，清理边界外所有的空间。

![](https://user-gold-cdn.xitu.io/2020/6/18/172c65b7f3a1e5cb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 标记一压缩算法的最终效果等同于标记一清除算法执行完成后，再进行一次内存碎片整理，因此，也可以把它称为标记一清除一压缩（Mark一 Sweep一Compact）算法。
- 二者的本质差异在于标记一清除算法是一种非移动式的回收算法，标记压缩是移动式的。是否移动回收后的存活对象是一项优缺点并存的风险决策。
- 可以看到，标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可，这比维护一个空闲列表显然少了许多开销。

**优点**
- 消除了标记一清除算法当中，内存区域分散的缺点，我们需要给新对象分配内存时，JVM只 需要持有一个内存的起始地址即可。
- 消除了复制算法当中，内存减半的高额代价。

**缺点**
- 从效率上来说，标记一整理算法要低于复制算法。(因为要标记后再移动, 即 O(2n). 而复制算法边标记边移动, O(n) )
- 移动对象的同时，如果对象被其他对象引用，则还需要调整引用的地址。移动过程中，需要全程暂停用户应用程序。即:STW

#### 8 小结
效率上来说，复制算法是当之无愧的老大，但是却浪费了太多内存。
而为了尽量兼顾上面提到的三个指标，标记一整理算法相对来说更平滑一些，但是效率.上不尽如人意，它比复制算法多了一个标记的阶段，比标记一清除多了一个整理内存的阶段。

|       | Mark-Sweep |	Mark-Compact	| Copying | 
| -      | -                 |   -     | - |
|速度    |	中等	       | 最慢	    | 最快
空间开销 |	少(但会堆积碎片)	| 少(不堆积碎片)	| 通常需要活对象的2倍大小(不堆积碎片)
移动对象	| 否               |	是	| 是

#### 9 分代收集算法
难道就没有一种最优的算法么?
==没有最好的算法,只有更合适的算法==
  前面所有这些算法中，并没有一种算法可以完全替代其他算法，它们都具有自己独特的优势和特点。分代收集算法应运而生。
  分代收集算法，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，==不同生命周期的对象可以采取不同的收集方式，以便提高回收效率==。一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点使用不同的回收算法，以提高垃圾回收的效率。
  在Java程序运行的过程中，会产生大量的对象，其中有些对象是与业务信息相关，比如Http请求中的Session对象、线程、Socket连接， 这类对象跟业务直接挂钩，因此生命周期比较长。但是还有一些对象，主要是程序运行过程中生成的临时变量，这些对象生命周期会比较短，比如： String对象， 由于其不变类的特性，系统会产生大量的这些对象，有些对象甚至只用一次即可回收。

  目前几乎所有的GC都是采用分代收集（Generational Collecting） 算法执行垃圾回收的。   在HotSpot中，基于分代的概念，GC所使用的内存回收算法必须结合年轻代和老年代各自的特点。

- 年轻代（Young Gen）

    - 年轻代特点：区域相对老年代较小，对象生命周期短、存活率低，回收频繁。
    - 这种情况==复制算法==的回收整理，速度是最快的。复制算法的效率只和当前存活对象大小有关，因此很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过hotspot中的两个survivor的设计得到缓解。

- 老年代（Tenured Gen）

    - 老年代特点：区域较大，对象生命周期长、存活率高，回收不及年轻代频繁。
    - 这种情况存在大量存活率高的对象，复制算法明显变得不合适。一般是由标记一清除或者是标记一清除与标记一整理的混合实现。
        - Mark阶段的开销与存活对象的数量成正比。
        - Sweep阶段的开销与所管理区域的大小成正相关。
        - Compact阶段的开销与存活对象的数据成正比。

以HotSpot中的CMS回收器为例，CMS是基于Mark一 Sweep实现的，对于对象的回收效率很高。而对于碎片问题，CMS采用基于Mark一Compact算法的Serial 0ld回收器作为补偿措施：当内存回收不佳（碎片导致的Concurrent Mode Failure时），将采用Serial 0ld执行Full GC以达到对老年代内存的整理。
  分代的思想被现有的虚拟机广泛使用。几乎所有的垃圾回收器都区分新生代和老年代。

#### 10 增量收集算法、分区算法
**增量收集算法**
上述现有的算法，在垃圾回收过程中，应用软件将处于一种stop the World的状态。在Stop the World状态下，应用程序所有的线程都会挂起，暂停一切正常的工作，等待垃圾回收的完成。如果垃圾回收时间过长，应用程序会被挂起很久，将严重影响用户体验或者系统的稳定性。为了解决这个问题，即对实时垃圾收集算法的研究直接导致了增量收集（Incremental Collecting） 算法的诞生。

**基本思想**
如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成。
  总的来说，增量收集算法的基础仍是传统的标记一清除和复制算法。增量收集算法通过对线程间冲突的妥善处理，允许垃圾收集线程以分阶段的方式完成标记、清理或复制工作。
**缺点：**
使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，造成系统吞吐量的下降。

**分区算法**
  一般来说，在相同条件下，堆空间越大，一次GC时所需要的时间就越长，有关GC产生的停顿也越长。为了更好地控制GC产生的停顿时间，将一块 大的内存区域分割成多个小块，根据目标的停顿时间，每次合理地回收若干个小区间，而不是整个堆空间，从而减少一次GC所产生的停顿。
  分代算法将按照对象的生命周期长短划分成两个部分，分区算法将整个堆空间划分成连续的不同小区间。
  每一个小区间都独立使用，独立回收。这种算法的好处是可以控制一次回收多少个小区间。

## 垃圾回收2-垃圾回收相关概念
### System.gc()的理解
- 在默认情况下，通过System.gc （）或者Runtime . getRuntime（） .gc（）的调用，会显式触发Full GC，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。
- 然而System.gc（）调用附带一个免责声明，==无法保证对垃圾收集器的调用(无法保证马上触发GC)==。
- JVM实现者可以通过system.gc（）调用来决定JVM的GC行为。而一般情况下，垃圾回收应该是自动进行的，无须手动触发，否则就太过于麻烦了。在一些特殊情况下，如我们正在编写一个性能基准，我们可以在运行之，间调用System.gc（）。
- 以下代码,如果注掉System.runFinalization(); 那么控制台不保证一定打印,证明了System.gc（）无法保证GC一定执行

```java
public class SystemGCTest {
    public static void main(String[] args) {
        new SystemGCTest();
        System.gc();//提醒jvm的垃圾回收器执行gc,但是不确定是否马上执行gc
        //与Runtime.getRuntime().gc();的作用一样。
        System.runFinalization();//强制调用使用引用的对象的finalize()方法
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("SystemGCTest 重写了finalize()");
    }
}
```

#### 手动gc理解不可达对象的回收行为
```java
public class LocalVarGC {
    public void localvarGC1() {
        byte[] buffer = new byte[10 * 1024 * 1024];//10MB
        System.gc();
        //输出: 不会被回收, FullGC时被放入老年代
        //[GC (System.gc()) [PSYoungGen: 14174K->10736K(76288K)] 14174K->10788K(251392K), 0.0089741 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]
        //[Full GC (System.gc()) [PSYoungGen: 10736K->0K(76288K)] [ParOldGen: 52K->10649K(175104K)] 10788K->10649K(251392K), [Metaspace: 3253K->3253K(1056768K)], 0.0074098 secs] [Times: user=0.01 sys=0.02, real=0.01 secs]
    }

    public void localvarGC2() {
        byte[] buffer = new byte[10 * 1024 * 1024];
        buffer = null;
        System.gc();
        //输出: 正常被回收
        //[GC (System.gc()) [PSYoungGen: 14174K->544K(76288K)] 14174K->552K(251392K), 0.0011742 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
        //[Full GC (System.gc()) [PSYoungGen: 544K->0K(76288K)] [ParOldGen: 8K->410K(175104K)] 552K->410K(251392K), [Metaspace: 3277K->3277K(1056768K)], 0.0054702 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]

    }

    public void localvarGC3() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        System.gc();
        //输出: 不会被回收, FullGC时被放入老年代
        //[GC (System.gc()) [PSYoungGen: 14174K->10736K(76288K)] 14174K->10784K(251392K), 0.0076032 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]
        //[Full GC (System.gc()) [PSYoungGen: 10736K->0K(76288K)] [ParOldGen: 48K->10649K(175104K)] 10784K->10649K(251392K), [Metaspace: 3252K->3252K(1056768K)], 0.0096328 secs] [Times: user=0.01 sys=0.01, real=0.01 secs]
    }

    public void localvarGC4() {
        {
            byte[] buffer = new byte[10 * 1024 * 1024];
        }
        int value = 10;
        System.gc();
        //输出: 正常被回收
        //[GC (System.gc()) [PSYoungGen: 14174K->496K(76288K)] 14174K->504K(251392K), 0.0016517 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
        //[Full GC (System.gc()) [PSYoungGen: 496K->0K(76288K)] [ParOldGen: 8K->410K(175104K)] 504K->410K(251392K), [Metaspace: 3279K->3279K(1056768K)], 0.0055183 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]
    }

    public void localvarGC5() {
        localvarGC1();
        System.gc();
        //输出: 正常被回收
        //[GC (System.gc()) [PSYoungGen: 14174K->10720K(76288K)] 14174K->10744K(251392K), 0.0121568 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]
        //[Full GC (System.gc()) [PSYoungGen: 10720K->0K(76288K)] [ParOldGen: 24K->10650K(175104K)] 10744K->10650K(251392K), [Metaspace: 3279K->3279K(1056768K)], 0.0101068 secs] [Times: user=0.01 sys=0.02, real=0.01 secs]
        //[GC (System.gc()) [PSYoungGen: 0K->0K(76288K)] 10650K->10650K(251392K), 0.0005717 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
        //[Full GC (System.gc()) [PSYoungGen: 0K->0K(76288K)] [ParOldGen: 10650K->410K(175104K)] 10650K->410K(251392K), [Metaspace: 3279K->3279K(1056768K)], 0.0045963 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
    }

    public static void main(String[] args) {
        LocalVarGC local = new LocalVarGC();
        local.localvarGC5();
    }
}
```

### 内存溢出与内存泄漏
- 内存溢出相对于内存泄漏来说，尽管更容易被理解，但是同样的，内存溢出也是引发程序崩溃的罪魁祸首之一。
- 由于GC一直在发展，所有一般情况下，除非应用程序占用的内存增长速度非常快，造成垃圾回收已经跟不上内存消耗的速度，否则不太容易出现OOM的情况。
- 大多数情况下，GC会进行各种年龄段的垃圾回收，实在不行了就放大招，来一次独占式的Full GC操作，这时候会回收大量的内存，供应用程序继续使用。
- javadoc中对OutOfMemoryError的解释是，==没有空闲内存，并且垃圾收集器也无法提供更多内存==。

#### 内存溢出
- 首先说没有空闲内存的情况：说明Java虚拟机的堆内存不够。原因有二：
    1. Java虚拟机的堆内存设置不够。
    比如：可能存在内存泄漏问题；也很有可能就是堆的大小不合理，比如我们要处理比较可观的数据量，但是没有显式指定JVM堆大小或者指定数值偏小。我们可以通过参数一Xms、一Xmx来调整。
    2. 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）对于老版本的Oracle JDK，因为永久代的大小是有限的，并且JVM对永久代垃圾回收（如，常量池回收、卸载不再需要的类型）非常不积极，所以当我们不断添加新类型的时候，永久代出现OutOfMemoryError也非常多见，尤其是在运行时存在大量动态类型生成的场合；类似intern字符串缓存占用太多空间，也会导致OOM问题。对应的异常信息，会标记出来和永久代相关： "java. lang. OutOfMemoryError： PermGen space"。
    随着元数据区的引入，方法区内存已经不再那么窘迫，所以相应的00M有所改观，出现00M，异常信息则变成了：“java. lang. OutOfMemoryError： Metaspace"。 直接内存不足，也会导致OOM。
- 这里面隐含着一层意思是，在抛出0utOfMemoryError之 前，通常垃圾收集器会被触发，尽其所能去清理出空间。
    - 例如：在引用机制分析中，涉及到JVM会去尝试回收软引用指向的对象等。
    - 在java.nio.BIts.reserveMemory（）方法中，我们能清楚的看到，System.gc（）会被调用，以清理空间。
- 当然，也不是在任何情况下垃圾收集器都会被触发的
    - 比如，我们去分配一一个超大对象，类似一个超大数组超过堆的最大值，JVM可以判断出垃圾收集并不能解决这个问题，所以直接拋出OutOfMemoryError

#### 内存泄漏(Memory Leak)
- 也称作“存储渗漏”。严格来说，==只有对象不会再被程序用到了，但是GC又不能回收他们的情况，才叫内存泄漏==。
- 但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致OOM，也可以叫做宽泛意义上的“内存泄漏
- 尽管内存泄漏并不会立刻引起程序崩溃，但是一旦发生内存泄漏，程序中的可用内存就会被逐步蚕食，直至耗尽所有内存，最终出现0utOfMemory异常，导致程序崩溃。
- 注意，这里的存储空间并不是指物理内存，而是指虚拟内存大小，这个虚拟内存大小取决于磁盘交换区设定的大小。

![](https://user-gold-cdn.xitu.io/2020/6/22/172daa92e77f06cb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**举例**
1. 单例模式
单例的生命周期和应用程序是一样长的，所以单例程序中，如果持有对外部对象的引用的话，那么这个外部对象是不能被回收的，则会导致内存泄漏的产生。
2. 一些提供close的资源未关闭导致内存泄漏 数据库连接（ dataSourse. getConnection（）），网络连接（socket）和io连接必须手动close，否则是不能被回收的。

### Stop The World
- Stop一the一World，简称STW，指的是Gc事件发生过程中，会产生应用程序的停顿。停顿产生时整个应用程序线程都会被暂停，没有任何响应，有点像卡死的感觉，这个停顿称为STW。.
    - 可达性分析算法中枚举根节点（GC Roots）会导致所有Java执行线程停顿。.
        - 分析工作必须在一个能确保一致性的快照 中进行
        - 一致性指整个分析期间整个执行系统看起来像被冻结在某个时间点上V- - 如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证
- 被STW中断的应用程序线程会在完成GC之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样， 所以我们需要减少STW的发生。
- STW事件和采用哪款GC无关，所有的GC都有这个事件。
- 哪怕是G1也不能完全避免Stop一the一world情况发生，只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。
- STW是JVM在后台自动发起和自动完成的。在用户不可见的情况下，把用户正常的工作线程全部停掉。
- 开发中不要用System.gc（）；会导致Stop一the一world的发生。

### 安全点与安全区域
#### 安全点(Safepoint)
- 程序执行时并非在所有地方都能停顿下来开始GC，只有在特定的位置才能停顿下来开始GC，这些位置称为“安全点（Safepoint） ”
- Safe Point的选择很重要，如果太少可能导致GC等待的时间太长，如果太频繁可能导致运行时的性能问题。大部分指令的执行时间都非常短暂，通常会根据“是否具有让程序长时间执行的特征”为标准。比如：选择些执行时间较长的指令作为Safe Point， 如方法调用、循环跳转和异常跳转等。

**如何在GC发生时，检查所有线程都跑到最近的安全点停顿下来呢？**
- 抢先式中断： （目前没有虚拟机采用了） 首先中断所有线程。如果还有线程不在安全点，就恢复线程，让线程跑到安全点。
- 主动式中断： 设置一个中断标志，各个线程运行到Safe Point的时候主动轮询这个标志，如果中断标志为真，则将自己进行中断挂起。

#### 安全区域(Safe Region)
  Safepoint机制保证了程序执行时，在不太长的时间内就会遇到可进入GC的Safepoint 。但是，程序“不执行”的时候呢？例如线程处于Sleep 状态或Blocked状态，这时候线程无法响应JVM的中断请求，“走” 到安全点去中断挂起，JVM也不太可能等待线程被唤醒。对于这种情况，就需要安全区域（Safe Region）来解决。
  安全区域是指在一段代码片段中，对象的引用关系不会发生变化，在这个区域中的任何位置开始GC都是安全的。我们也可以把Safe Region 看做是被扩展了的Safepoint。

**实际执行时:**
1. 当线程运行到Safe Region的代码时，首先标识已经进入了Safe Region，如果这段时间内发生GC，JVM会 忽略标识为Safe Region状态 的线程；
2. 当线程即将离开Safe Region时， 会检查JVM是否已经完成GC，如果完成了，则继续运行，否则线程必须等待直到收到可以安全离开SafeRegion的信号为止；

### 引用
- 我们希望能描述这样一类对象： 当内存空间还足够时，则能保留在内存中；如果内存空间在进行垃圾收集后还是很紧张，则可以抛弃这些对象。 -【既偏门又非常高频的面试题】强引用、软引用、弱引用、虚引用有什么区别？具体使用.场景是什么？
- 在JDK 1.2版之后，Java对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference） 、弱引用（Weak Reference） 和虚引用（Phantom Reference） 4种，这4种引用强度依次逐渐减弱。
- 除强引用外，其他3种引用均可以在java.lang.ref包中找到它们的身影。如下图，显示了这3种引用类型对应的类，开发人员可以在应用程序中直接使用它们。

Reference子类中只有终结器引用是包内可见的，其他3种引用类型均为public，可以在应用程序中直接使用
- 强引用（StrongReference）I ：最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，即类似“0bject obj=new object（ ）”这种引用关系。==无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象==。
- 软引用（SoftReference） ：在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存溢出异常。
- 弱引用（WeakReference） ：被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。
- 虚引用（PhantomReference） ：一个对象是否有虛引用的存在，完全不会对其生存时 间构成影响，也无法通过虚引用来获得一个对象的实例。==为一个对象设置虛引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知(回收跟踪)==。

#### 强引用: 不回收
- 在Java程序中，最常见的引用类型是强引用（普通系统99%以上都是强引用），也就是我们最常见的普通对象引用，也是默认的引用类型。
- 当在Java语言中使用new操作符创建一个新的对象， 并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。
- 强引用的对象是可触及的，垃圾收集器就永远不会回收掉被引用的对象。
- 对于一一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，就是可以当做垃圾被收集了，当然具体回收时机还是要看垃圾收集策略。
- 相对的，软引用、 弱引用和虚引用的对象是软可触及、弱可触及和虛可触及的，在一定条件下，都是可以被回收的。所以，强引用是造成Java内存泄漏的主要原因之一。

StringBuffer str = new StringBuffer ("Hello,尚硅谷");
局部变量str指向StringBuffer实例所在堆空间，通过str可以操作该实例，那么str就是StringBuffer实例的强引用
对应内存结构：
![](https://user-gold-cdn.xitu.io/2020/6/22/172daabab71a33cf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
此时,如果再运行一个赋值语句:
StringBuffer str1 = str;
对应内存结构:
![](https://user-gold-cdn.xitu.io/2020/6/22/172daac11e6cc19b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
本例中的两个引用，都是强引用，强引用具备以下特点：

- 强引用可以直接访问目标对象。
- 强引用所指向的对象在任何时候都不会被系统回收，虚拟机宁愿抛出OOM异常，也不会回收强引用所指向对象。
- 强引用可能导致内存泄漏。

#### 软引用: 内存不足即回收
- 软引用是用来描述一 些还有用，但非必需的对象。只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。
- ==软引用通常用来实现内存敏感的缓存==。比如：高速缓存就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。
- 垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列（ Reference Queue）。
- 类似弱引用，只不过Java虚拟机会尽量让软引用的存活时间长一些，迫不得.已才清理。
- 软引用：
    - 当内存足够: 不会回收软引|用的可达对象
    - 当内存不够时: 会回收软引用的可达对象
- 在JDK 1. 2版之后提供了java.lang.ref.SoftReference类来实现软引用。

```java
Object obj = new object（）； //声明强引用
SoftReference<0bject> sf = new SoftReference<0bject>（obj）；
obj = null； //销毁强引用
```

```java
/**
 * 软引用的测试：内存不足即回收
 * -Xms10m -Xmx10m -XX:+PrintGCDetails
 */
public class SoftReferenceTest {
    public static class User {
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int id;
        public String name;

        @Override
        public String toString() {
            return "[id=" + id + ", name=" + name + "] ";
        }
    }

    public static void main(String[] args) {
        //创建对象，建立软引用
//        SoftReference<User> userSoftRef = new SoftReference<User>(new User(1, "songhk"));
        //上面的一行代码，等价于如下的三行代码
        User u1 = new User(1,"songhk");
        SoftReference<User> userSoftRef = new SoftReference<User>(u1);
        u1 = null;//取消强引用


        //从软引用中重新获得强引用对象
        System.out.println(userSoftRef.get());

        System.gc();
        System.out.println("After GC:");
//        //垃圾回收之后获得软引用中的对象
        System.out.println(userSoftRef.get());//由于堆空间内存足够，所有不会回收软引用的可达对象。
//
        try {
            //让系统认为内存资源紧张、不够
//            byte[] b = new byte[1024 * 1024 * 7];
            byte[] b = new byte[1024 * 7168 - 399 * 1024];//恰好能放下数组又放不下u1的内存分配大小 不会报OOM
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            //再次从软引用中获取数据
            System.out.println(userSoftRef.get());//在报OOM之前，垃圾回收器会回收软引用的可达对象。
        }
    }
}
```

#### 弱引用: 发现即回收
- 弱引用也是用来描述那些非必需对象，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。在系统GC时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象。
- 但是，由于垃圾回收器的线程通常优先级很低，因此，并不一 定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间。
- 弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。
- 软引用、弱引用都非常适合来保存那些可有可无的缓存数据。如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。
- 在JDK1.2版之后提后了java.lang.ref.WeakReference类来实现弱引用
- 弱引用对象与软引用对象的最大不同就在于，当GC在进行回收时，需要通过算法检查是否回收软引用对象，而对于弱引用对象，GC总是进行回收。弱引用对象更容易、更快被GC回收。
- 面试题：你开发中使用过WeakHashMap吗？
通过查看WeakHashMap源码,可以看到其内部类Entry使用的就是弱引用
line 702 -> private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {...}

```java
public class WeakReferenceTest {
    public static class User {
        public User(int id, String name) {
            this.id = id;
            this.name = name;
        }

        public int id;
        public String name;

        @Override
        public String toString() {
            return "[id=" + id + ", name=" + name + "] ";
        }
    }

    public static void main(String[] args) {
        //构造了弱引用
        WeakReference<User> userWeakRef = new WeakReference<User>(new User(1, "songhk"));
        //从弱引用中重新获取对象
        System.out.println(userWeakRef.get());

        System.gc();
        // 不管当前内存空间足够与否，都会回收它的内存
        System.out.println("After GC:");
        //重新尝试从弱引用中获取对象
        System.out.println(userWeakRef.get());
    }
}
```

#### 虚引用: 对象回收跟踪
- 虚引用(Phantom Reference),也称为“幽灵引用”或者“幻影引用”，是所有引用类型中最弱的一个。
- 一个对象是否有虚引用的存在，完全不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它和没有引用几乎是一样的，随时都可能被垃圾回收器回收。
- 它不能单独使用，也无法通过虚引用来获取被引用的对象。当试图通过虚引用的get（）方法取得对象时，总是null。
- ==为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程==。比如：能在这个对象被收集器回收时收到一个系统通知。
- 虚引用必须和引用队列一起使用。虚引用在创建时必须提供一个引用队列作为参数。当垃圾回收器准备回收一个对象时，如果发现它还有虛引用，就会在回收对象后，将这个虚引用加入引用队列，以通知应用程序对象的回收情况。
- 由于虚引用可以跟踪对象的回收时间，因此，也可以将一些资源释放操作放置在虛引用中执行和记录。
- 在JDK 1. 2版之后提供了PhantomReference类来实现虚引用。

```java
object obj = new object();
ReferenceQueuephantomQueue = new ReferenceQueue( ) ;
PhantomReference<object> pf = new PhantomReference<object>(obj, phantomQueue); 
obj = null;
```

```java
public class PhantomReferenceTest {
    public static PhantomReferenceTest obj;//当前类对象的声明
    static ReferenceQueue<PhantomReferenceTest> phantomQueue = null;//引用队列

    public static class CheckRefQueue extends Thread {
        @Override
        public void run() {
            while (true) {
                if (phantomQueue != null) {
                    PhantomReference<PhantomReferenceTest> objt = null;
                    try {
                        objt = (PhantomReference<PhantomReferenceTest>) phantomQueue.remove();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    if (objt != null) {
                        System.out.println("追踪垃圾回收过程：PhantomReferenceTest实例被GC了");
                    }
                }
            }
        }
    }

    @Override
    protected void finalize() throws Throwable { //finalize()方法只能被调用一次！
        super.finalize();
        System.out.println("调用当前类的finalize()方法");
        obj = this;
    }

    public static void main(String[] args) {
        Thread t = new CheckRefQueue();
        t.setDaemon(true);//设置为守护线程：当程序中没有非守护线程时，守护线程也就执行结束。
        t.start();

        phantomQueue = new ReferenceQueue<PhantomReferenceTest>();
        obj = new PhantomReferenceTest();
        //构造了 PhantomReferenceTest 对象的虚引用，并指定了引用队列
        PhantomReference<PhantomReferenceTest> phantomRef = new PhantomReference<PhantomReferenceTest>(obj, phantomQueue);

        try {
            //不可获取虚引用中的对象
            System.out.println(phantomRef.get());

            //将强引用去除
            obj = null;
            //第一次进行GC,由于对象可复活，GC无法回收该对象
            System.gc();
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 可用");
            }
            System.out.println("第 2 次 gc");
            obj = null;
            System.gc(); //一旦将obj对象回收，就会将此虚引用存放到引用队列中。
            Thread.sleep(1000);
            if (obj == null) {
                System.out.println("obj 是 null");
            } else {
                System.out.println("obj 可用");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```console
null
调用当前类的finalize()方法
obj 可用
第 2 次 gc
追踪垃圾回收过程：PhantomReferenceTest实例被GC了
obj 是 null
```

## 垃圾回收3-垃圾回收器
### GC的分类与性能指标
- 垃圾收集器没有在规范中进行过多的规定，可以由不同的厂商、不同版本的JVM来实现。
- 由于JDK的版本处于高速迭代过程中，因此Java发展至今已经衍生了众多的GC版本。
- 从不同角度分析垃圾收集器，可以将GC分为不同的类型。

1. **按线程数分，可以分为串行垃圾回收器和并行垃圾回收器**

![](https://user-gold-cdn.xitu.io/2020/6/28/172f886ddbb6f49a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 串行回收指的是在同一时间段内只允许有一个CPU用于执行垃圾回收操作，此时工作线程被暂停，直至垃圾收集工作结束。
    - 在诸如单CPU处理器或者较小的应用内存等硬件平台不是特别优越的场合，串行回收器的性能表现可以超过并行回收器和并发回收器。所以，串行回收默认被应用在客户端的Client模式下的JVM中
    - 在并发能力比较强的CPU上，并行回收器产生的停顿时间要短于串行回收器。
- 和串行回收相反，并行收集可以运用多个CPU同时执行垃圾回收，因此提升了应用的吞吐量，不过并行回收仍然与串行回收一样，采用独占式，使用了“ Stop一the一world”机制。

2. **按照工作模式分，可以分为并发式垃圾回收器和独占式垃圾回收器**
- 并发式垃圾回收器与应用程序线程交替工作，以尽可能减少应用程序的停顿时间。
- 独占式垃圾回收器（Stop the world）一旦运行，就停止应用程序中的所有用户线程，直到垃圾回收过程完全结束。
![](https://user-gold-cdn.xitu.io/2020/6/28/172f8873dadc3689?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

3. **按碎片处理方式分，可分为压缩式垃圾回收器和非压缩式垃圾回收器**

- 压缩式垃圾回收器会在回收完成后，对存活对象进行压缩整理，消除回收后的碎片。
  - 再分配对象空间使用: 指针碰撞
- 非压缩式的垃圾回收器不进行这步操作。
  - 再分配对象空间使用: 空闲列表

4. **按工作的内存区间分，又可分为年轻代垃圾回收器和老年代垃圾回收器**

#### 评估GC的性能指标

- ==吞吐量：运行用户代码的时间占总运行时间的比例==
  - （总运行时间：程序的运行时间十内存回收的时间）
- 垃圾收集开销：吞吐量的补数，垃圾收集所用时间与总运行时间的比例。

- ==暂停时间：执行垃圾收集时，程序的工作线程被暂停的时间==

- 收集频率：相对于应用程序的执行，收集操作发生的频率。

- ==内存占用： Java堆区所占的内存大小==

- 快速：一个对象从诞生到被回收所经历的时间。

- 这三者共同构成一个“不可能三角”。三者总体的表现会随着技术进步而越来越好。一款优秀的收集器通常最多同时满足其中的两项。

- 这三项里，暂停时间的重要性日益凸显。因为随着硬件发展，内存占用 多些越来越能容忍，硬件性能的提升也有助于降低收集器运行时对应用程序的影响，即提高了吞吐量。而内存的扩大，对延迟反而带来负面效果。

简单来说，主要抓住两点：

1. 吞吐量
2. 暂停时间

##### 吞吐量

- 吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量=运行用户代码时间/ （运行用户代码时间+垃圾收集时间）
    - 比如：虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%
- 这种情况下，应用程序能容忍较高的暂停时间，因此，高吞吐量的应用程序有更长的时间基准，快速响应是不必考虑的。
- 吞吐量优先，意味着在单位时间内，STW的时间最短： 0.2 + 0.2 = 0.4

##### 暂停时间

- 暂停时间”是指一个时间段内应用程序线程暂停，让GC线程执行的状态
    - 例如，GC期间100毫秒的暂停时间意味着在这100毫秒期间内没有应用程序线程是活动的。.
- 暂停时间优先，意味着尽可能让单次STW的时间最短： 0.1 + 0.1 + 0.1 + 0.1+0.1=0.5
- 高吞吐量较好因为这会让应用程序的最终用户感觉只有应用程序线程在做“生产性”工作。直觉上，吞吐量越高程序运行越快。
- 低暂停时间（低延迟）较好因为从最终用户的角度来看不管是GC还是其他原因导致一个应用被挂起始终是不好的。这取决于应用程序的类型，有时候甚至短暂的200毫秒暂停都可能打断终端用户体验。因此，具有低的较大暂停时间是非常重要的，特别是对于一一个交互式应用程序。
- 不幸的是”高吞吐量”和”低暂停时间”是一对相互竞争的目标（矛盾）。
    - 因为如果选择以吞吐量优先，那么必然需要降低内存回收的执行频率，但是这样会导致GC需要更长的暂停时间来执行内存回收。
    - 相反的，如果选择以低延迟优先为原则，那么为了降低每次执行内存回收时的暂停时间，也只能频繁地执行内存回收，但这又引起了年轻代内存的缩诚和导致程序吞吐量的下降。
- 在设计（或使用） GC算法时，我们必须确定我们的目标： 一个GC算法只可能针对两个目标之一（即只专注于较大吞吐量或最小暂停时间），或尝试找到一个二者的折衷。
- 现在标准：在最大吞吐量优先的情况下，降低停顿时间。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88838b4a0d54?imageslim)

### 不同的垃圾回收器概述

#### 7款经典的垃圾收集器

- 串行回收器：Serial. Serial Old
- 并行回收器：ParNew. Parallel Scavenge. Parallel Old
- 并发回收器：CMS. G1

![](https://user-gold-cdn.xitu.io/2020/6/28/172f888c00412b4c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 7款经典的垃圾收集器与垃圾分代之间的关系

- 新生代收集器： Serial、 ParNeW、Parallel Scavenge；
- 老年代收集器： Serial 0ld、 Parallel 0ld、 CMS；
- 整堆收集器： G1；

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88913bd8914c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 垃圾收集器的组合关系

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88972faa533c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

1. 两个收集器间有连线，表明它们可以搭配使用： Serial/Serial 01d、Serial/CMS、 ParNew/Serial 01d、ParNew/CMS、 Parallel Scavenge/Serial 01d、Parallel Scavenge/Parallel 0ld、G1；
2. 其中Serial 0ld作为CMS 出现"Concurrent Mode Failure"失败的后 备预案。
3. (红色虚线)由于维护和兼容性测试的成本，在JDK 8时将Serial+CMS、 ParNew+Serial 01d这两个组合声明为废弃（JEP 173） ，并在JDK 9中完全取消了这些组合的支持（JEP214），即：移除。
（绿色虚线）JDK 14中：弃用Parallel Scavenge和Serial0ld GC组合（JEP366 ）
4. （青色虚线）JDK 14中：删除CMS垃圾回收器 （JEP 363）

- 为什么要有很多收集器个不够吗？ 因为Java的使用场景很多， 移动端，服务器等。所以就需要针对不同的场景，提供不同的垃圾收集器，提高垃圾收集的性能。
- 虽然我们会对各个收集器进行比较，但并非为了挑选一个最好的收集器出来。没有一种放之四海皆准、任何场景下都适用的完美收集器存在，更加没有万能的收集器。所以我们选择的只是对具体应用最合适的收集器。

**查看默认的垃圾收集器**

- -xx：+PrintCommandLineFlags： 查看命令行相关参数（包含使用的垃圾收集器）
- 使用命令行指令： jinfo 一flag相关垃圾回收器参数进程ID

```java
/**
 *  -XX:+PrintCommandLineFlags
 *
 *  -XX:+UseSerialGC:表明新生代使用Serial GC ，同时老年代使用Serial Old GC
 *
 *  -XX:+UseParNewGC：标明新生代使用ParNew GC
 *
 *  -XX:+UseParallelGC:表明新生代使用Parallel GC
 *  -XX:+UseParallelOldGC : 表明老年代使用 Parallel Old GC
 *  说明：二者可以相互激活
 *
 *  -XX:+UseConcMarkSweepGC：表明老年代使用CMS GC。同时，年轻代会触发对ParNew 的使用
 */
public class GCUseTest {
    public static void main(String[] args) {
        ArrayList<byte[]> list = new ArrayList<>();

        while(true){
            byte[] arr = new byte[100];
            list.add(arr);
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```console
-XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC 
```

jdk8 使用的是parallel
![](https://user-gold-cdn.xitu.io/2020/6/28/172f88a0a1ecb6fe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

jdk9 使用的是G1
![](https://user-gold-cdn.xitu.io/2020/6/28/172f88aa975aae3a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Serial回收器:串行回收

- Serial收集器是最基本、历史最悠久的垃圾收集器了。JDK1.3之前回收新生代唯一的选择。
- Serial收集器作为HotSpot中Client模式下的默认新生代垃圾收集器。
- Serial收集器采用复制算法、串行回收和"Stop一 the一World"机制的方式执行内存回收。 ）
- 除了年轻代之外，Serial收集器还提供用于执行老年代垃圾收集的Serial 0ld收集器。 Serial 0ld收集器同样也采用了串行回收 和"Stop the World"机制，只不过内存回收算法使用的是标记一压缩算 法。
    - Serial 0ld是运行在Client模式下默认的老年代的垃圾回收器
    - Serial 0ld在Server模式下主要有两个用途：①与新生代的ParallelScavenge配合使用; ②作为老年代CMS收集器的后备垃圾收集方案
- 这个收集器是一个单线程的收集器，但它的“单线程”的意义并不仅仅说明它只会使用一个CPU或一条收集线程去完成垃圾收集工作，更重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束（Stop The World ）。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88af4c12170b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**优势**

- 简单而高效（与其他收集器的单线程比），对于限定单个CPU的环境来说，Seria1收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。
    - 运行在Client模式下的虛拟机是个不错的选择。
- 在用户的桌面应用场景中，可用内存一般不大（几十MB至一两百MB）， 可以在较短时间内完成垃圾收集（几十ms至一百多ms） ，只要不频繁发生，使用串行回收器是可以接受的。
- 在HotSpot虛拟机中，使用一XX： +UseSerialGC 参数可以指定年轻代和老年代都使用串行收集器。
    - 等价于新生代用Serial GC，且老年代用Serial 0ld GC
    - 控制台输出 -XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseSerialGC

**总结**

- 这种垃圾收集器大家了解，现在已经不用串行的了。而且在限定单核cpu才可以用。现在都不是单核的了。
- 对于交互较强的应用而言，这种垃圾收集器是不能接受的。一般在Javaweb应用程序中是不会采用串行垃圾收集器的。

### ParNew回收器:并行回收

- 如果说Serial GC是年轻代中的单线程垃圾收集器，那么ParNew收集器则是Serial收集器的多线程版本。
    - Par是Paralle1的缩写，New： 只能处理的是新生代
- ParNew收集器除了采用并行回收的方式执行内存回收外，两款垃圾收集器之间几乎没有任何区别。ParNew收集器在年轻代中同样也是采用复制算法、"Stop一 the一World"机制。
- ParNew是很多JVM运行在Server模式下新生代的默认垃圾收集器。
- 对于新生代，回收次数频繁，使用并行方式高效。
- 对于老年代，回收次数少，使用串行方式节省资源。（CPU并行 需要切换线程，串行可以省去切换线程的资源）
- 由于ParNew收集器是基于并行回收，那么是否可以断定ParNew收集器的回收效率在任何场景下都会比Serial收集器更高效？
    - ParNew 收集器运行在多CPU的环境下，由于可以充分利用多CPU、 多核心等物理硬件资源优势，可以更快速地完成垃圾收集，提升程序的吞吐量。
    - 但是在单个CPU的环境下，ParNew收 集器不比Serial收集器更高 效。虽然Serial收集器是基于串行回收，但是由于CPU不需要频繁地做任务切换，因此可以有效避免多线程交互过程中产生的一些额外开销。
- 因为除Serial外，目前只有ParNew GC能与CMS收集器配合工作
- 在程序中，开发人员可以通过选项"一XX： +UseParNewGC"手动指定使用.ParNew收集器执行内存回收任务。它表示年轻代使用并行收集器，不影响老年代。
- -XX：ParallelGCThreads 限制线程数量，默认开启和CPU数据相同的线程数。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88b5da16f393?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Parallel回收器:吞吐量优先

- HotSpot的年轻代中除了拥有ParNew收集器是基于并行回收的以外， Parallel Scavenge收集器同样也采用了复制算法、并行回收和"Stop the World"机制。
- 那么Parallel收集器的出现是否多此一举？
    - 和ParNew收集器不同，Parallel Scavenge收集 器的目标则是==达到一个可控制的吞吐量==（Throughput），它也被称为吞吐量优先的垃圾收集器。
    - 自适应调节策略也是Parallel Scavenge 与ParNew一个重要区别。
- 高吞吐量则可以高效率地利用CPU 时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。因此，常见在服务器环境中使用。例如，那些执行批量处理、订单处理、工资支付、科学计算的应用程序。
- Parallel收集器在JDK1.6时提供了用于执行老年代垃圾收集的 Parallel 0ld收集器，用来代替老年代的Serial 0ld收集器。
- Parallel 0ld收集器采用了标记一压缩算法，但同样也是基于并行回收和”Stop一the一World"机制。
- 在程序吞吐量优先的应用场景中，Parallel 收集器和Parallel 0ld收集器的组合，在Server模式下的内存回收性能很不错。
- **在Java8中，默认是此垃圾收集器**

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88bd8272c80d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**参数配置**

- 一XX： +UseParallelGC手动指定 年轻代使用Parallel并行收集器执行内存回收任务。
- 一XX： +UseParallel0ldGc手 动指定老年代都是使用并行回收收集器。
    - 分别适用于新生代和老年代。默认jdk8是开启的。
    - 上面两个参数，默认开启一个，另一个也会被开启。 （互相激活）
- 一XX： ParallelGCThreads设置年轻代并行收集器的线程数。一般地，最好与CPU数量相等，以避免过多的线程数影响垃圾收集性能。
    - 在默认情况下，当CPU数量小于8个， Paralle lGCThreads 的值等于CPU数量。
    - 当CPU数量大于8个， ParallelGCThreads的值等于3+[5*CPU_ Count]/8]
- 一XX ：MaxGCPau3eMillis设置垃圾收集器最大停顿时间（即STw的时间）。单位是毫秒。
    - 为了尽可能地把停顿时间控制在MaxGCPauseMills以内，收集器在.工作时会调整Java堆大小或者其他一些参数。
    - 对于用户来讲，停顿时间越短体验越好。但是在服务器端，我们注重 高并发，整体的吞吐量。所以服务器端适合Parallel，进行控制。该参数使用需谨慎。
- 一XX：GCTimeRatio垃圾收集时间占总时间的比例（= 1 / （N + 1））用于衡量吞吐量的大小。
    - 取值范围（0， 100）。默认值99，也就是垃圾回收时间不超过1号。
    - 与前一个一XX：MaxGCPauseMillis参数有一定矛盾性。暂停时间越长，Radio参数就容易超过设定的比例。
- 一XX： +UseAdaptiveSizePolicy设 置Parallel Scavenge收 集器具有自适应调节策略
    - 在这种模式下，年轻代的大小、Eden和Survivor的比例、晋升老年 代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。
    - 在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指 定虚拟机的最大堆、目标的吞吐量（GCTimeRatio）和停顿时间（MaxGCPauseMills），让虚拟机自己完成调优工作。

### CMS回收器:低延迟

- 在JDK1.5时期， HotSpot推出了一款在强交互应用中几乎可认为有划 时代意义的垃圾收集器： CMS （Concurrent 一Mark 一 Sweep）收集器，这款收集器是HotSpot虚拟机中第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程同时工作。
- CMS收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间。停顿时 间越短（低延迟）就越适合与用户交互的程序，良好的响应速度能提升用户体验。
    - 目前很大一部分的Java应用集中在互联网站或者B/s系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求。
- CMS的垃圾 收集算法采用标记一清除算法，并且也 会" stop一the一world"
- 不幸的是，CMS 作为老年代的收集器，却无法与JDK 1.4.0 中已经存在的新生代收集器Parallel Scavenge配合工作，所以在JDK 1. 5中使用CMS来收集老年代的时候，新生代只能选择ParNew或者Serial收集器中的一个。
- 在G1出现之前，CMS使用还是非常广泛的。一直到今天，仍然有很多系统使用CMS GC。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88c4cd91d748?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

CMS整个过程比之前的收集器要复杂，整个过程分为4个主要阶段，即初始标记阶段、并发标记阶段、重新标记阶段和并发清除阶段。

- 初始标记（Initial一Mark） 阶段：在这个阶段中，程序中所有的工作线程都将会因为. “Stop一the一World"机制而出现短暂的暂停，这个阶段的**主要任务仅仅只是标记出GCRoots能直接关联到的对象**。一旦标记完成之后就会恢复之前被暂停的所有应用.线程。由于直接关联对象比较小，所以这里的速度非常快。
- 并发标记（Concurrent一Mark）阶段：从GC Roots的 直接关联对象开始遍历整个对 象图的过程，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行, ==但此过程可能又产生一些垃圾, 不能被本次 GC 收集, 这些被称为浮动垃圾==。
- 重新标记（Remark） 阶段：由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短。
- 并发清除（ Concurrent一Sweep）阶段：此阶段清理删除掉标记阶段判断的已经死亡的对象，释放内存空间。**由于不需要移动存活对象**，所以这个阶段也是可以与用户线程同时并发的

尽管CMS收集器采用的是并发回收（非独占式），但是在其初始化标记和再次标记这两个阶段中仍然需要执行“Stop一the一World”机制暂停程序中的工作线程，不过暂停时间并不会太长，因此可以说明目前所有的垃圾收集器都做不到完全不需要“Stop一the一World”，只是尽可能地缩短暂停时间。
 由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的。
  另外，由于在垃圾收集阶段用户线程没有中断，所以在CMS回收过程中，还应该确保应用程序用户线程有足够的内存可用。因此，CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是当堆内存使用率达到某一阈值时，便开始进行回收，以确保应用程序在CMS工作过程中依然有足够的空间支持应用程序运行。要是CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时虚拟机将启动后备预案：临时启用Serial 0ld收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。
CMS收集器的垃圾收集算法采用的是标记一清除算法，这意味着每次执行完内存回收后，由于被执行内存回收的无用对象所占用的内存空间极有可能是不连续的一些内存块，不可避免地将会产生一些内存碎片。 那么CMS在为新对象分配内存空间时，将无法使用指针碰撞（Bump the Pointer） 技术，而只能够选择空闲列表（Free List） 执行内存分配。

**有人会觉得既然Mark Sweep会造成内存碎片，那么为什么不把算法换成Mark Compact呢？**

答案其实很简答，因为当并发清除的时候，用Compact整理内存的话，原来的用户线程使用的内存还怎么用呢？要保证用户线程能继续执行，前提的它运行的资源不受影响嘛。Mark Compact更适合“Stop the World”这种场景”下使用

### G1回收器:区域化分代式

**为什么名字叫做Garbage First （G1）呢？**

- 因为G1是一个并行回收器，它把堆内存分割为很多不相关的区域（Region） （物理上 不连续的）。使用不同的Region来表示Eden、幸存者0区，幸存者1区，老年代等。
- G1 GC有计划地避免在整个Java 堆中进行全区域的垃圾收集。G1跟踪各个Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。
- 由于这种方式的侧重点在于回收垃圾最大量的区间（Region），所以我们给G1一个名字：垃圾优先（Garbage First） 。
- G1 （Garbage一First） 是一款面向服务端应用的垃圾收集器，主要针对配备多核CPU及大容量内存的机器，以极高概率满足GC停顿时间的同时，还兼具高吞吐量的性能特征。
- 在JDK1. 7版本正式启用，移除了Experimental的标识，是JDK 9以后的默认垃圾回收器，取代了CMS回收器以及Parallel + Parallel 0ld组合。被Oracle官方称为“全功能的垃圾收集器” 。
- 与此同时，CMS已经在JDK 9中被标记为废弃（deprecated） 。在jdk8中还不是默认的垃圾回收器，需要使用一XX： +UseG1GC来启用。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88cff30208a5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 优势

与其他GC收集器相比，G1使用了全新的分区算法，其特点如下所示：

- 并行与并发
    - 并行性： G1在回收期间，可以有多个Gc线程同时工作，有效利用多核计算能力。此时用户线程STW
    - 并发性： G1拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况
- 分代收集
    - 从分代上看，G1依然属于分代型垃圾回收器，它会区分年轻代和老年代，年轻代依然有Eden区和Survivor区。但从堆的结构，上看，它不要求整个Eden区、年轻代或者老年代都是连续的，也不再坚持固定大小和固定数量。
    - 将堆空间分为若干个区域（Region） ，这些区域中包含了逻辑上的年轻代和老年代。
    - 和之前的各类回收器不同，它同时兼顾年轻代和老年代。对比其他回收器，或者工作在年轻代，或者工作在老年代；

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88d5eaac9036?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 空间整合
    - CMS： “标记一清除”算法、内存碎片、若干次Gc后进行一次碎片整理
    - G1将内存划分为一个个的region。 内存的回收是以region作为基本单位的.Region之间是复制算法，但整体上实际可看作是标记一压缩（Mark一Compact）算法，两种算法都可以避免内存碎片。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。尤其是当Java堆非常大的时候，G1的优势更加明显。
- 可预测的停顿时间模型（即：软实时soft real一time） 这是G1相对于CMS的另一大优势，G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。
    - 由于分区的原因，G1可以只选取部分区域进行内存回收，这样缩小了回收的范围，因此对于全局停顿情况的发生也能得到较好的控制。
    - G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以 及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。保证了G1 收集器在有限的时间内可以获取尽可能高的收集效率。
    - 相比于CMS，G1未必能做到CMS在最好情况下的延时停顿，但是最差情况要.好很多。

#### 缺点

- 相较于CMS，G1还不具备全方位、压倒性优势。比如在用户程序运行过程中，G1无论是为了垃圾收集产生的内存占用（Footprint） 还是程序运行时的额外执行负载（overload） 都要比CMS要高。
- 从经验上来说，在小内存应用上CMS的表现大概率会优于G1，而G1在大内存应用，上则发挥其优势。平衡点在6一8GB之间。

#### 参数设置

- 一XX：+UseG1GC 手动指定使用G1收集器执行内存回收任务。
- 一XX：G1HeapRegionSize 设置每个Region的大小。值是2的幂，范围是1MB 到32MB之间，目标是根据最小的Java堆大小划分出约2048个区域。默认是堆内存的1/2000。
- 一XX：MaxGCPauseMillis 设置期望达到的最大Gc停顿时间指标（JVM会尽力实现，但不保证达到）。默认值是200ms
- 一xX：ParallelGCThread 设置sTw.工作线程数的值。最多设置为8
- 一XX：ConcGCThreads 设置并发标记的线程数。将n设置为并行垃圾回收线程数（ParallelGCThreads）的1/4左右。
- 一XX：Ini tiatingHeapOccupancyPercent 设置触发并发GC周期的Java堆占用率阈值。超过此值，就触发GC。默认值是45。

#### G1回收器的常见操作步骤
G1的设计原则就是简化JVM性能调优，开发人员只需要简单的三步即可完成调优：

- 第一步：开启G1垃圾收集器
- 第二步：设置堆的最大内存
- 第三步：设置最大的停顿时间

G1中提供了三种垃圾回收模式： YoungGC、 Mixed GC和Full GC， 在不同的条件下被触发。

#### 适用场景

- 面向服务端应用，针对具有大内存、多处理器的机器。（在普通大小的堆里表现并不.惊喜）
- 最主要的应用是需要低GC延迟，并具有大堆的应用程序提供解决方案；
- 如：在堆大小约6GB或更大时，可预测的暂停时间可以低于0.5秒； （ G1通过每次只清理一部分而不是全部的Region的增量式清理来保证每次GC停顿时间不会过长）。
- 用来替换掉JDK1.5中的CMS收集器； 在下面的情况时，使用G1可能比CMS好：
    1. 超过50%的Java堆被活动数据占用；
    2. 对象分配频率或年代提升频率变化很大；
    3. GC停顿时间过长（长于0. 5至1秒）。
- HotSpot垃圾收集器里，除了G1以外，其他的垃圾收集器使用内置的JVM线程执行 GC的多线程操作，而G1 GC可以采用应用线程承担后台运行的GC工作，即当JVM的GC线程处理速度慢时，系统会调用应用程序线程帮助加速垃圾回收过程。

#### 分区region,化整为零
使用G1收集器时，它将整个   Java堆划分成约2048个大小相同的独立Region块，每个Region块大小根据堆空间的实际大小而定，整体被控制在1MB到32MB之间，且为2的N次幂，即1MB， 2MB， 4MB， 8MB， 1 6MB， 32MB。可以通过一 XX：G1HeapRegionSize设定。所有的Region大小相同，且在JVM生命周期内不会被改变。
  虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分Region （不需要连续）的集合。通过Region的动态分配方式实现逻辑_上的连续。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88e292405e6f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 一个region 有可能属于Eden， Survivor 或者0ld/Tenured 内存区域。但是一个region只可能属于一个角色。图中的E表示该region属于Eden内存区域，s表示属于Survivor内存区域，0表示属于0ld内存区域。图中空白的表示未使用的内存空间。
- G1垃圾收集器还增加了一种新的内存区域，叫做Humongous内存区域，如图中的H块。主要用于存储大对象，如果超过1. 5个region，就放到H。
- 设置H的原因：
    - 对于堆中的大对象，默认直接会被分配到老年代，但是如果它是一个短期存在的大对象，就会对垃圾收集器造成负面影响。为了解决这个问题，G1划分了一个Humongous区，它用来专门存放大对象。如果一个H区装不下一个大对象，那么G1会寻找连续的H区来存储。为了能找到连续的H区，有时候不得不启动Full GC。G1的大多数行为都把H区作为老年代的一部分来看待。

#### G1回收器垃圾回收过程
G1 GC的垃圾回收过程主要包括如下三个环节：

- 年轻代GC （Young GC ）
- 老年代并发标记过程（ Concurrent Marking）
- 混合回收（Mixed GC ）
（如果需要，单线程、独占式、高强度的Full GC还是继续存在的。它针对GC的评估失败提供了一种失败保护机制，即强力回收。）

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88e82e57b525?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

顺时针， young gc 一> young gc + concurrent mark 一> Mixed GC顺序，进行垃圾回收。

- 应用程序分配内存，当年轻代的Eden区用尽时开始年轻代回收过程； G1的年轻代收集阶段是一个并行的独占式收集器。在年轻代回收期，G1 GC暂停所有应用程序线程，启动多线程执行年轻代回收。然后从年轻代区间移动存活对象到Survivor区间或者老年区间，也有可能是两个区间都会涉及。
- 当堆内存使用达到一定值（默认45%）时，开始老年代并发标记过程。
- 标记完成马.上开始混合回收过程。对于一个混合回收期，G1 GC从老年区间移动存活对象到空闲区间，这些空闲区间也就成为了老年代的一部分。和年轻代不同，老年代的G1回收器和其他GC不同，G1的老年代回收器不需要整个老年代被回收，一次只需要扫描/回收一小部分老年代的Region就可以了。同时，这个老年代Region是和年轻代一起 被回收的。
- ==G1 GC是一个响应时间优先的GC算法，它与CMS最大的不同是，用户可以设定整个GC过程的期望停顿时间==
    - 通过控制年轻代的region个数，即年轻代内存大小，来控制young GC的时间开销
    -在用户指定的开销目标范围内尽可能选择收益高的老年代Region

#### 记忆集与写屏障

- 一个对象被不同区域引用的问题(分代引用问题)
- 一个Region不可能是孤立的，一个Region中的对象可能被其他任意Region中对象引用，判断对象存活时，是否需要扫描整个Java堆才能保证准确？
- 在其他的分代收集器，也存在这样的问题（ 而G1更突出）
回收新生代也不得不同时扫描老年代？这样的话会降低MinorGC的效率；
- 解决方法：
    - 无论G1还是其他分代收集器，JVM都是使用RememberedSet来避免全局扫描：
    - 每个Region都有 一个对
    - 每次Reference类 型数据写操作时，都会产生一个Write Barrier暂 时中断操作； .
    - 然后检查将要写入的引用指向的对象是否和该Reference类型数据在不同的Region （其他收集器：检查老年代对象是否引用了新生代对象） ；
    - 如果不同，通过CardTable把相关引用信息记录到引用指向对象的所在Region对应的Remembered Set中；
    - 当进行垃圾收集时，在GC根节点的枚举范围加入Remembered Set；就可以保证不进行全局扫描，也不会有遗漏。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88f075afe7cf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### G1回收过程详解

1. **年轻代GC(完全Stop The World)**

- JVM启动时，G1 先准备好Eden区，程序在运行过程中不断创建对象到Eden区，当Eden空间耗尽时，G1会启动一次年轻代垃圾回收过程。
- 年轻代垃圾回收只会回收Eden区和Survivor区。
- YGC时，首先G1停止应用程序的执行（Stop一The一World），G1创建回收集（Collection Set），回收集是指需要被回收的内存分段的集合，年轻代回收过程的回收集包含年轻代Eden区和Survivor区所有的内存分段。
- 第一阶段，扫描根。
根是指static变量指向的对象，正在执行的方法调用链条上的局部变量等。根引用连同RSet记录的外部引用作为扫描存活对象的入口。
- 第二阶段，更新RSet.
处理dirty card queue（ 见备注）中的card，更新RSet。 此阶段完成后，RSet可 以准确的反映老年代对所在的内存分段中对象的引用。
    - dirty card queue: 对于应用程序的引用赋值语句object.field=object，JVM会在之前和之后执行特殊的操作以在dirty card queue中入队一个保存了对象引用信息的card。在年轻代回收的时候， G1会对Dirty Card Queue中所有的card进行处理，以更新RSet，保证RSet实时准确的反映引用关系。 那为什么不在引用赋值语句处直接更新RSet呢？这是为了性能的需要，RSet的处理需要线程同步，开销会很大，使用队列性能会好很多。
- 第三阶段，处理RSet。
识别被老年代对象指向的Eden中的对象，这些被指向的Eden中的对象被认为是存活的对象。
- 第四阶段，复制对象。
此阶段，对象树被遍历，Eden区 内存段中存活的对象会被复制到Survivor区中空的内存分段，Survivor区内存段中存活的对象如果年龄未达阈值，年龄会加1，达到阀值会被会被复制到01d区中空的内存分段。如果Survivor空间不够，Eden空间的 部分数据会直接晋升到老年代空间。
- 第五阶段，处理引用。
处理Soft，Weak， Phantom， Final， JNI Weak等引用。最终Eden空间的数据为空，GC停止工作，而目标内存中的对象都是连续存储的，没有碎片，所以复制过程可以达到内存整理的效果，减少碎片。

![](https://user-gold-cdn.xitu.io/2020/6/28/172f88f5d8bec04e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

2. **并发标记过程 (堆内存使用达到一定值（默认45%）时)**
- 初始标记阶段：标记从根节点直接可达的对象。这个阶段是STW的。initial mark是共用了Young GC的暂停，这是因为他们可以复用root scan操作，所以可以说global concurrent marking是伴随Young GC而发生的
- 根区域扫描（Root Region Scanning） ： G1 GC扫描Survivor区 直接可达的老年代区域对象，并标记被引用的对象。这一过程必 须在young GC之前完成。
- 并发标记（Concurrent Marking）： 在整个堆中进行并发标记（和应用程序并发执行），此过程可能被young GC中断。在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那这个区域会被立即回收。同时，并发标记过程中，会计算每个区域的对象活性（区域中存活对象的比例）。
- 再次标记（Remark）： 由 于应用程序持续进行，需要修正上一次的标记结果。是STW的。G1中采用了比CMS更快的初始快照算法：snapshot一at一the一beginning （SATB）。
- 独占清理（cleanup，STW）：计算各个区域的存活对象和GC回收比例，并进行排序，识别可以混合回收的区域。为下阶段做铺垫。是STW的。
    - 这个阶段并不会实际上去做垃圾的收集
- 并发清理阶段：识别并清理完全空闲的区域。

3. **混合回收(完全Stop The World)**

![](https://user-gold-cdn.xitu.io/2020/6/28/172f8907bb964563?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当越来越多的对象晋升到老年代oldregion时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即Mixed GC， 该算法并不是一个0ldGC，除了回收整个Young Region，还会回收一部分的0ldRegion。这里需要注意：是一部分老年代， 而不是全部老年代。可以选择哪些0ldRegion进行收集，从而可以对垃圾回收的耗时时间进行控制。也要注意的是Mixed GC并不是Full GC。

- 并发标记结束以后，老年代中百分百为垃圾的内存分段被回收了，部分为垃圾的内存分段被计算了出来。默认情况下，这些老年代的内存分段会分8次（可以通过一XX： G1MixedGCCountTarget设置）被回收。
- 混合回收的回收集（Collection Set） 包括八分之一的老年代内存分段，Eden区内存分段，Survivor区内存分段。混合回收的算法和年轻代回收的算法完全一样，只是回收集多了老年代的内存分段。具体过程请参考上面的年轻代回收过程。
- 由于老年代中的内存分段默认分8次回收，G1会优先回收垃圾多的内存分段。垃圾占内存分段比例越高的，越会被先回收。并且有一个阈值会决定内存分段是否被回收，一xX： G1MixedGCLiveThresholdPercent，默认为65%，意思是垃圾占内存分段比例要达到65%才会被回收。如果垃圾占比太低，意味着存活的对象占比高，在复制的时候会花费更多的时间。
- 混合回收并不一定要进行8次。有一个阈值一Xx： G1HeapWastePercent，默认值为10%，意思是允许整个堆内存中有10%的空间被浪费，意味着如果发现可以回收的垃圾占堆内存的比例低于10%，则不再进行混合回收。因为GC会花费很多的时间但是回收到的内存却很少。

4. Full GC
  G1的初衷就是要避免Full GC的出现。但是如果上述方式不能正常工作，G1会停止应用程序的执行（Stop一 The一World），使用单线程的内存回收算法进行垃圾回收，性能会非常差，应用程序停顿时间会很长。
  要避免Full GC的发生，一旦发生需要进行调整。什么时候会发生Full GC呢？比如堆内存太小，当G1在复制存活对象的时候没有空的内存分段可用，则会回退到full gc， 这种情况可以通过增大内存解决。
  导致G1Full GC的原因可能有两个：
    1. Evacuation的时候没有足够的to一 space来存放晋升的对象；
    2. 并发处理过程完成之前空间耗尽。

### 垃圾回收器总结

![](https://user-gold-cdn.xitu.io/2020/6/28/172f890f1baad386?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

