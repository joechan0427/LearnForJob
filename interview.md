# java基础
## 1. 请你谈谈Java中是如何支持正则表达式操作的？
Java中的String类提供了支持正则表达式操作的方法，包括：matches()、replaceAll()、replaceFirst()、split()。此外，Java中可以用Pattern类表示正则表达式对象，它提供了丰富的API进行各种正则表达式操作，如：
```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
class RegExpTest {
    public static void main(String[] args) {
        String str = "成都市(成华区)(武侯区)(高新区)";
        Pattern p = Pattern.compile(".*?(?=\\()");
        Matcher m = p.matcher(str);
        if(m.find()) {
            System.out.println(m.group());
        }
    }
}
```

## 2. 请你简单描述一下正则表达式及其用途。
在编写处理字符串的程序时，经常会有查找符合某些复杂规则的字符串的需要。正则表达式就是用于描述这些规则的工具。换句话说，正则表达式就是记录文本规则的代码。计算机处理的信息更多的时候不是数值而是字符串，正则表达式就是在进行字符串匹配和处理的时候最为强大的工具，绝大多数语言都提供了对正则表达式的支持。

## 3. 请你比较一下Java和JavaSciprt？
JavaScript 与Java是两个公司开发的不同的两个产品。Java 是原Sun Microsystems公司推出的面向对象的程序设计语言，特别适合于互联网应用程序开发；而JavaScript是Netscape公司的产品，为了扩展Netscape浏览器的功能而开发的一种可以嵌入Web页面中运行的基于对象和事件驱动的解释性语言。JavaScript的前身是LiveScript；而Java的前身是Oak语言。
下面对两种语言间的异同作如下比较：
- 基于对象和面向对象：Java是一种真正的面向对象的语言，即使是开发简单的程序，必须设计对象；JavaScript是种脚本语言，它可以用来制作与网络无关的，与用户交互作用的复杂软件。它是一种基于对象（Object-Based）和事件驱动（Event-Driven）的编程语言，因而它本身提供了非常丰富的内部对象供设计人员使用。
- 解释和编译：Java的源代码在执行之前，必须经过编译。JavaScript是一种解释性编程语言，其源代码不需经过编译，由浏览器解释执行。（目前的浏览器几乎都使用了JIT（即时编译）技术来提升JavaScript的运行效率）
- 强类型变量和类型弱变量：Java采用强类型变量检查，即所有变量在编译之前必须作声明；JavaScript中变量是弱类型的，甚至在使用变量前可以不作声明，JavaScript的解释器在运行时检查推断其数据类型。
- 代码格式不一样。

## 4. 请你说明一下，在Java中如何跳出当前的多重嵌套循环？
在最外层循环前加一个标记如A，然后用break A;可以跳出多重循环。（Java中支持带标签的break和continue语句，作用有点类似于C和C++中的goto语句，但是就像要避免使用goto一样，应该避免使用带标签的break和continue，因为它不会让你的程序变得更优雅，很多时候甚至有相反的作用，所以这种语法其实不知道更好）

## 5. 请你讲讲&和&&的区别？
&运算符有两种用法：(1)按位与；(2)逻辑与。&&运算符是短路与运算。逻辑与跟短路与的差别是非常巨大的，虽然二者都要求运算符左右两端的布尔值都是true整个表达式的值才是true。&&之所以称为短路运算是因为，如果&&左边的表达式的值是false，右边的表达式会被直接短路掉，不会进行运算。很多时候我们可能都需要用&&而不是&，例如在验证用户登录时判定用户名不是null而且不是空字符串，应当写为：username != null &&!username.equals("")，二者的顺序不能交换，更不能用&运算符，因为第一个条件如果不成立，根本不能进行字符串的equals比较，否则会产生NullPointerException异常。

## 6. int和Integer有什么区别？
Java是一个近乎纯洁的面向对象编程语言，但是为了编程的方便还是引入了基本数据类型，但是为了能够将这些基本数据类型当成对象操作，Java为每一个基本数据类型都引入了对应的包装类型（wrapper class），int的包装类就是Integer，从Java 5开始引入了自动装箱/拆箱机制，使得二者可以相互转换。
Java 为每个原始类型提供了包装类型：
- 原始类型: boolean，char，byte，short，int，long，float，double
- 包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double

```java
class AutoUnboxingTest {
    public static void main(String[] args) {
        Integer a = new Integer(3);
        Integer b = 3;                  // 将3自动装箱成Integer类型
        int c = 3;
        System.out.println(a == b);     // false 两个引用没有引用同一对象
        System.out.println(a == c);     // true a自动拆箱成int类型再和c比较
    }
}
```

## 7. 我们在web应用开发过程中经常遇到输出某种编码的字符，如iso8859-1等，请你讲讲如何输出一个某种编码的字符串？
java底层使用16位的unicode编码存储字符串,即char数组,jdk11后换成byte数组.
- 如果一个以gbk格式存储的文件,以utf8编码读取,必然会造成乱码,此时不能再正确解码,因为底层数组已经变了
    - 以utf8读gbk文件的过程:
    对照utf8表,得到该字符,然后再查Unicode表得到该字符的编码,一旦utf8表读到不允许出现的开头,会统一转换为置换字符,然后在unicode表查到这个置换字符,写入数组对应位置,造成底层数组错误
- 把gbk编码的字符串以utf8形式输出
```java
Public String translate (String str) {
 String tempStr = “”;
 try {
 tempStr = new String(str.getBytes(“ISO-8859-1″), “GBK”);
 tempStr = tempStr.trim();
 }
 catch (Exception e) {
 System.err.println(e.getMessage());
 }
 return tempStr;
 }
```

## 8. String,StringBuffer与StringBuilder的区别?
**String 字符串常量
StringBuffer 字符串变量（线程安全）
StringBuilder 字符串变量（非线程安全）**

 简要的说， String 类型和 StringBuffer 类型的主要性能区别其实在于 String 是不可变的对象, 因此在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，那速度是一定会相当慢的。
 而如果是使用 StringBuffer 类则结果就不一样了，每次结果都会对 StringBuffer 对象本身进行操作，而不是生成新的对象，再改变对象引用。所以在一般情况下我们推荐使用 StringBuffer ，特别是字符串对象经常改变的情况下。而在某些特别情况下， String 对象的字符串拼接其实是被 JVM 解释成了 StringBuffer 对象的拼接，所以这些时候 String 对象的速度并不会比 StringBuffer 对象慢，而特别是以下的字符串对象生成中， String 效率是远要比 StringBuffer 快的：
 String S1 = “This is only a” + “ simple” + “ test”;
 StringBuffer Sb = new StringBuilder(“This is only a”).append(“ simple”).append(“ test”);
 你会很惊讶的发现，生成 String S1 对象的速度简直太快了，而这个时候 StringBuffer 居然速度上根本一点都不占优势。其实这是 JVM 的一个把戏，在 JVM 眼里，这个
 String S1 = “This is only a” + “ simple” + “test”; 其实就是：
 String S1 = “This is only a simple test”; 所以当然不需要太多的时间了。但大家这里要注意的是，如果你的字符串是来自另外的 String 对象的话，速度就没那么快了，譬如：
String S2 = “This is only a”;
String S3 = “ simple”;
String S4 = “ test”;
String S1 = S2 +S3 + S4;
这时候 JVM 会规规矩矩的按照原来的方式去做,即
new StringBuilder(s2).append(s3).append(s4).toString();

## 9. 请你讲讲数组(Array)和列表(ArrayList)的区别？什么时候应该使用Array而不是ArrayList？
Array和ArrayList的不同点：
- Array可以包含基本类型和对象类型
ArrayList只能包含对象类型,底层是一个object[]。
- Array大小是固定的，ArrayList的大小是动态变化的,当新增一个新元素,会先检查是否装得下,如果装不下,会扩容,容量为此时容量+此时容量/2。
- ArrayList提供了更多的方法和特性，比如：addAll()，removeAll()，iterator()等等。
- 对于基本类型数据，集合使用自动装箱来减少编码工作量。但是，当处理固定大小的基本数据类型的时候，这种方式相对比较慢。

## 10. 请你解释什么是值传递和引用传递？
值传递是对基本型变量而言的,传递的是该变量的一个副本,改变副本不影响原变量.
引用传递一般是对于对象型变量而言的,传递的是该对象地址的一个副本, 并不是原对象本身 。 所以对引用对象进行操作会同时改变原对象.

## 11. 请你讲讲Java支持的数据类型有哪些？什么是自动拆装箱？
Java语言支持的8种基本数据类型是：
byte
short
int
long
float
double
boolean
char
自动装箱是Java编译器在基本数据类型和对应的对象包装类型之间做的一个转化。比如：把int转化成Integer，double转化成Double，等等。反之就是自动拆箱。

## 12.请你解释为什么会出现4.0-3.6=0.40000001这种现象？
`4.0`的`float`二进制存储为:`0 10000001 0*23`
`3.6`的`float`二进制存储为:`0 10000000 11001...`(存在误差)

**follow up:**
可以使用BigInteger和BigDecimal解决,底层使用的是String

## 13.请你讲讲一个十进制的数在内存中是怎么存的？
补码形式,即:
正数与原码相同
负数为绝对值的原码取反后加一
小数为首位符号位,然后将整数和小数分别表示出来,将小数点移到第一个1的右边,左移为正,右移为负,将移位数加127后放入30到23位,剩下的位数放此时小数点所在的位置的右边23位

## 14.请你说明符号“==”比较的是什么？
“`==`”对比两个对象基于内存引用，如果两个对象的引用完全相同（指向同一个对象）时，“`==`”操作将返回true，否则返回false。“`==`”如果两边是基本类型，就是比较数值是否相等。

## 15.请你说说Lambda表达式的优缺点。
优点：1. 简洁。2. 非常容易并行计算。3. 可能代表未来的编程趋势。

缺点：1. 若不用并行计算，很多时候计算速度没有比传统的 for 循环快。（并行计算有时需要预热才显示出效率优势）2. 不容易调试。3. 若其他程序员没有学过 lambda 表达式，代码不容易让其他语言的程序员看懂。

## 16.请你解释Object若不重写hashCode()的话，hashCode()如何计算出来的？
Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法直接返回对象的 内存地址。

## 17.为什么重写equals还要重写hashcode？
为了HashMap能得到正确的结果,比如同样内容的String在不同位置被new出来时,hashcode应该相等才能保证hashmap的存储位置一样,然后hashmap再调用equals函数来确定key值一样
HashMap中的比较key是这样的，先求出key的hashcode(),比较其值是否相等，若相等再比较equals(),若相等则认为他们是相等的

## 18.map的分类和常见的情况
java为数据结构中的映射定义了一个接口java.util.Map;它有四个实现类,分别是HashMap Hashtable LinkedHashMap 和TreeMap.

Hashmap 是一个最常用的Map,它根据键的HashCode值存储数据,根据键可以直接获取它的值，具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。 **HashMap最多只允许一条记录的键为Null;允许多条记录的值为 Null**;HashMap不支持线程的同步，即任一时刻可以有多个线程同时写HashMap;可能会导致数据的不一致。如果需要同步，可以用 Collections的synchronizedMap方法使HashMap具有同步的能力，或者使用ConcurrentHashMap。

Hashtable与 HashMap类似,它继承自Dictionary类，不同的是:**它不允许记录的键或者值为空;它支持线程的同步**，即任一时刻只有一个线程能写Hashtable,因此也导致了 Hashtable在写入时会比较慢。

LinkedHashMap 是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的.也可以在构造时用带参数，按照应用次数排序。在遍历的时候会比HashMap慢，不过有种情况例外，**当HashMap容量很大，实际数据较少时，遍历起来可能会比 LinkedHashMap慢，因为LinkedHashMap的遍历速度只和实际数据有关，和容量无关，而HashMap的遍历速度和他的容量有关。**(原因:hashmap建了一个node数组,遍历hashmap其实就是遍历数组)

TreeMap实现SortMap接口，能够把它保存的记录根据键排序,**默认是按键值的升序排序**，也可以指定排序的比较器，当用Iterator 遍历TreeMap时，得到的记录是排过序的。

一般情况下，我们用的最多的是HashMap,在Map 中插入、删除和定位元素，HashMap 是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。如果需要输出的顺序和输入的相同,那么用LinkedHashMap 可以实现,它还可以按读取顺序来排列.

## 19.Java里面的final关键字是怎么用的？
当用final修饰一个类时，表明这个类不能被继承。也就是说，如果一个类你永远不会让他被继承，就可以用final进行修饰。final类中的成员变量可以根据需要设为final，但是要注意**final类中的所有成员方法都会被隐式地指定为final方法**。

“使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用(即直接将方法体展开而不是调用该方法,入栈,出栈等等)。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。在最近的Java版本中，不需要使用final方法进行这些优化了。“

对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。

## 20.关于Synchronized和lock 
synchronized是Java的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。JDK1.5以后引入了自旋锁、锁粗化、轻量级锁，偏向锁来有优化关键字的性能。

Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

## 21.介绍一下volatile？


## 22.介绍一下Syncronized锁，如果用这个关键字修饰一个静态方法，锁住了什么？如果修饰成员方法，锁住了什么？
synchronized修饰静态方法以及同步代码块的synchronized (类.class)用法锁的是类，线程想要执行对应同步代码，需要获得类锁。
synchronized修饰成员方法，线程获取的是当前调用该方法的对象实例的对象锁。

## 23.请解释Java中的概念，什么是构造函数？什么是构造函数重载？什么是复制构造函数？
当新对象被创建的时候，构造函数会被调用。每一个类都有构造函数。在程序员没有给类提供构造函数的情况下，Java编译器会为这个类创建一个默认的构造函数。
Java中构造函数重载和方法重载很相似。可以为一个类创建多个构造函数。每一个构造函数必须有它自己唯一的参数列表。
Java不支持像C++中那样的复制构造函数，这个不同点是因为如果你不自己写构造函数的情况下，Java不会创建默认的复制构造函数。

## 24.Java中的方法覆盖(Overriding)和方法重载(Overloading)是什么意思？
Java中的方法重载发生在同一个类里面两个或者是多个方法的方法名相同但是参数不同的情况。与此相对，方法覆盖是说子类重新定义了父类的方法。方法覆盖必须有相同的方法名，参数列表和返回类型。覆盖者可能不会限制它所覆盖的方法的访问。

## 25.请你谈一下面向对象的"六原则一法则"
- 单一职责原则：一个类只做它该做的事情。（单一职责原则想表达的就是"高内聚"，写代码最终极的原则只有六个字"高内聚、低耦合"，所谓的高内聚就是一个代码模块只完成一项功能，在面向对象中，如果只让一个类完成它该做的事，而不涉及与它无关的领域就是践行了高内聚的原则，这个类就只有单一职责。另一个是模块化，好的自行车是组装车，从减震叉、刹车到变速器，所有的部件都是可以拆卸和重新组装的，好的乒乓球拍也不是成品拍，一定是底板和胶皮可以拆分和自行组装的，一个好的软件系统，它里面的每个功能模块也应该是可以轻易的拿到其他系统中使用的，这样才能实现软件复用的目标。）

- 开闭原则：软件实体应当对扩展开放，对修改关闭。（在理想的状态下，当我们需要为一个软件系统增加新功能时，只需要从原来的系统派生出一些新类就可以，不需要修改原来的任何一行代码。要做到开闭有两个要点：①抽象是关键，一个系统中如果没有抽象类或接口系统就没有扩展点；②封装可变性，将系统中的各种可变因素封装到一个继承结构中，如果多个可变因素混杂在一起，系统将变得复杂而换乱，如果不清楚如何封装可变性，可以参考《设计模式精解》一书中对桥梁模式的讲解的章节。）

- 依赖倒转原则：面向接口编程。（该原则说得直白和具体一些就是声明方法的参数类型、方法的返回类型、变量的引用类型时，尽可能使用抽象类型而不用具体类型，因为抽象类型可以被它的任何一个子类型所替代，请参考下面的里氏替换原则。）里氏替换原则：任何时候都可以用子类型替换掉父类型。（关于里氏替换原则的描述，Barbara Liskov女士的描述比这个要复杂得多，但简单的说就是能用父类型的地方就一定能使用子类型。里氏替换原则可以检查继承关系是否合理，如果一个继承关系违背了里氏替换原则，那么这个继承关系一定是错误的，需要对代码进行重构。例如让猫继承狗，或者狗继承猫，又或者让正方形继承长方形都是错误的继承关系，因为你很容易找到违反里氏替换原则的场景。需要注意的是：子类一定是增加父类的能力而不是减少父类的能力，因为子类比父类的能力更多，把能力多的对象当成能力少的对象来用当然没有任何问题。）

- 接口隔离原则：接口要小而专，绝不能大而全。（臃肿的接口是对接口的污染，既然接口表示能力，那么一个接口只应该描述一种能力，接口也应该是高度内聚的。例如，琴棋书画就应该分别设计为四个接口，而不应设计成一个接口中的四个方法，因为如果设计成一个接口中的四个方法，那么这个接口很难用，毕竟琴棋书画四样都精通的人还是少数，而如果设计成四个接口，会几项就实现几个接口，这样的话每个接口被复用的可能性是很高的。Java中的接口代表能力、代表约定、代表角色，能否正确的使用接口一定是编程水平高低的重要标识。）

- 合成聚合复用原则：优先使用聚合或合成关系复用代码。（通过继承来复用代码是面向对象程序设计中被滥用得最多的东西，因为所有的教科书都无一例外的对继承进行了鼓吹从而误导了初学者，类与类之间简单的说有三种关系，Is-A关系、Has-A关系、Use-A关系，分别代表继承、关联和依赖。其中，关联关系根据其关联的强度又可以进一步划分为关联、聚合和合成，但说白了都是Has-A关系，合成聚合复用原则想表达的是优先考虑Has-A关系而不是Is-A关系复用代码，原因嘛可以自己从百度上找到一万个理由，需要说明的是，即使在Java的API中也有不少滥用继承的例子，例如Properties类继承了Hashtable类，Stack类继承了Vector类，这些继承明显就是错误的，更好的做法是在Properties类中放置一个Hashtable类型的成员并且将其键和值都设置为字符串来存储数据，而Stack类的设计也应该是在Stack类中放一个Vector对象来存储数据。记住：任何时候都不要继承工具类，工具是可以拥有并可以使用的，而不是拿来继承的。）

- 迪米特法则：迪米特法则又叫最少知识原则，一个对象应当对其他对象有尽可能少的了解。再复杂的系统都可以为用户提供一个简单的门面，Java Web开发中作为前端控制器的Servlet或Filter不就是一个门面吗，浏览器对服务器的运作方式一无所知，但是通过前端控制器就能够根据你的请求得到相应的服务。调停者模式也可以举一个简单的例子来说明，例如一台计算机，CPU、内存、硬盘、显卡、声卡各种设备需要相互配合才能很好的工作，但是如果这些东西都直接连接到一起，计算机的布线将异常复杂，在这种情况下，主板作为一个调停者的身份出现，它将各个设备连接在一起而不需要每个设备之间直接交换数据，这样就减小了系统的耦合度和复杂度。

## 26.如何通过反射获取和设置对象私有字段的值？
可以通过类对象的getDeclaredField()方法字段（Field）对象，然后再通过字段对象的setAccessible(true)将其设置为可以访问，接下来就可以通过get/set方法来获取/设置字段的值了。下面的代码实现了一个反射的工具类，其中的两个静态方法分别用于获取和设置私有字段的值，字段可以是基本类型也可以是对象类型且支持多级对象操作，例如ReflectionUtil.get(dog, "owner.car.engine.id");可以获得dog对象的主人的汽车的引擎的ID号。

## 27.重载（Overload）和重写（Override）的区别。重载的方法能否根据返回类型进行区分？
方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载；重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。重载对返回类型没有特殊的要求。

## 28.对象值相同(x.equals(y) == true)，但却可有不同的hash code，该说法是否正确，为什么？
不对，如果两个对象x和y满足x.equals(y) == true，它们的哈希码（hash code）应当相同。Java对于eqauls方法和hashCode方法是这样规定的：(1)如果两个对象相同（equals方法返回true），那么它们的hashCode值一定要相同；(2)如果两个对象的hashCode相同，它们并不一定相同。当然，你未必要按照要求去做，但是如果你违背了上述原则就会发现在使用容器时，相同的对象可以出现在Set集合中，同时增加新元素的效率会大大下降（对于使用哈希存储的系统，如果哈希码频繁的冲突将会造成存取性能急剧下降）。

## 29.内部类可以引用他包含类的成员吗，如果可以，有没有什么限制吗？
一个内部类对象可以访问创建它的外部类对象的内容
- 内部类如果不是static的，那么它可以访问创建它的外部类对象的所有属性内部类.
- 如果是sattic的，即为nested class，那么它只可以访问创建它的外部类对象的所有static属性
- 一般普通类只有public或package的访问修饰，而内部类可以实现static，protected，private等访问修饰。
- 当从外部类继承的时候，内部类是不会被覆盖的，它们是完全独立的实体，每个都在自己的命名空间内，如果从内部类中明确地继承，就可以覆盖原来内部类的方法。

## 30.JAVA语言如何进行异常处理，关键字：throws,throw,try,catch,finally分别代表什么意义？在try块中可以抛出异常吗？
- Java 通过面向对象的方法进行异常处理，把各种不同的异常进行分类，并提供了良好的接口。在Java中，每个异常都是一个对象，它是Throwable类或其它子类的实例。
- 当一个方法出现异常后便抛出一个异常对象，该对象中包含有异常信息，调用这个对象的方法可以捕获到这个异常并进行处理。Java的异常处理是通过5个关键词来实现的：try、catch、throw、throws和finally。
- 一般情况下是用try来执行一段程序，如果出现异常，系统会抛出（throws）一个异常，这时候你可以通过它的类型来捕捉（catch）它，或最后（finally）由缺省处理器来处理。
- 用try来指定一块预防所有”异常”的程序。紧跟在try程序后面，应包含一个catch子句来指定你想要捕捉的”异常”的类型。throw语句用来明确地抛出一个”异常”。throws用来标明一个成员函数可能抛出的各种”异常”。Finally为确保一段代码不管发生什么”异常”都被执行一段代码。
- 可以在一个成员函数调用的外面写一个try语句，在这个成员函数内部写另一个try语句保护其他代码。每当遇到一个try语句，”异常“的框架就放到堆栈上面，直到所有的try语句都完成。如果下一级的try语句没有对某种”异常”进行处理，堆栈就会展开，直到遇到有处理这种”异常”的try语句。

## 31.说说Static Nested Class 和 Inner Class的不同
Static Nested Class是被声明为静态（static）的内部类，它可以不依赖于外部类实例被实例化。而通常的内部类需要在外部类实例化后才能实例化。
Static-Nested Class 的成员, 既可以定义为静态的(static), 也可以定义为动态的(instance)
Nested Class的静态方法(Method)只能对Outer Class的静态成员(static memebr)进行操作(ACCESS), 而不能Access Outer Class的实例成员(instance member).
而 Nested Class的实例方法(instance method) 却可以访问Outer Class的所有成员

## 32.请你解释一下类加载机制，双亲委派模型，好处是什么？
某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

使用双亲委派模型的好处在于Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类java.lang.Object，它存在在rt.jar中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型最顶端的Bootstrap ClassLoader进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。相反，如果没有双亲委派模型而是由各个类加载器自行加载的话，如果用户编写了一个java.lang.Object的同名类并放在ClassPath中，那系统中将会出现多个不同的Object类，程序将混乱。因此，如果开发者尝试编写一个与rt.jar类库中重名的Java类，可以正常编译，但是永远无法被加载运行。

## 33.请你谈谈StringBuffer和StringBuilder有什么区别，底层实现上呢？
StringBuffer线程安全，StringBuilder线程不安全，底层实现上的话，StringBuffer其实就是比StringBuilder多了Synchronized修饰符。

## 34.请说明”static”关键字是什么意思？Java中是否可以覆盖(override)一个private或者是static的方法？
“static”关键字表明一个成员变量或者是成员方法可以在没有所属的类的实例变量的情况下被访问。
Java中static方法不能被覆盖，因为方法覆盖是基于运行时动态绑定的，而static方法是编译时静态绑定的。static方法跟类的任何实例都不相关，所以概念上不适用。

## 35.列举你所知道的Object类的方法并简要说明。
- Object()默认构造方法。
- clone() 创建并返回此对象的一个副本。
- equals(Object obj) 指示某个其他对象是否与此对象“相等”。
- finalize()当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。
- getClass()返回一个对象的运行时类。
- hashCode()返回该对象的哈希码值。 
- notify()唤醒在此对象监视器上等待的单个线程。 
- notifyAll()唤醒在此对象监视器上等待的所有线程。
- toString()返回该对象的字符串表示。
- wait()导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。
- wait(long timeout)导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。wait(long timeout, int nanos) 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。

## 36.Java有哪些特性，并举一个和多态有关的例子。
封装、继承、多态

## 37.wait方法的底层原理
1. 将当前线程封装成objectwaiter对象node
2. 通过objectmonitor::addwaiter方法将node添加到_WaitSet列表中
3. 通过ObjectMonitor:exit方法释放当前的ObjectMonitor对象，这样其他竞争线程就可以获取该ObjectMonitor对象
4. 最终底层的park方法会挂起线程
notify方法就是随机唤醒等待池中的一个线程

## 为什么Java 中只有值传递？
基本数据类型: 直接拷贝
引用类型: 引用拷贝,所以能改变里面的值`a[0]=1`,却不能改变引用`a = new int[4]` 对原函数的引用没有变化

## 接口和抽象类的区别是什么？
1. 接口的方法默认是public，所有方法在接口中不能有实现(Java 8
开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。
2. 接口中除了static、final变量，不能有其他变量，而抽象类中则不一定。
3. 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过extends关键字扩展多个接口。
4. 接口方法默认修饰符是public，抽象方法可以有public、protected
和default这些修饰符（抽象方法就是为了被重写所以不能使用private
关键字修饰！）。
5. 从设计层面来说，抽象是对类的抽象，是⼀种模板设计，而接口是对行为的抽象，是⼀种行为的规范

# 集合
![](https://uploadfiles.nowcoder.com/images/20200215/1034884_1581738199162_50125D538C24748FCB1032CB38EEE0A9)
## 38.List、Map、Set三个接口存取元素时，各有什么特点？
List以特定索引来存取元素，可以有重复元素。
Set不能存放重复元素（用对象的equals()方法来区分元素是否重复）。
Map保存键值对（key-value pair）映射，映射关系可以是一对一或多对一。
Set和Map容器都有基于哈希存储和排序树的两种实现版本，基于哈希存储的版本理论存取时间复杂度为O(1)，而基于排序树版本的实现在插入或删除元素时会按照元素或元素的键（key）构成排序树从而达到排序和去重的效果。

## 39.ArrayList、Vector、LinkedList的存储性能和特性
ArrayList和Vector都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，
Vector中的方法由于添加了synchronized修饰，因此Vector是线程安全的容器，但性能上较ArrayList差，因此已经是Java中的遗留容器。
LinkedList使用**双向链表**实现存储（将内存中零散的内存单元通过附加的引用关联起来，形成一个可以按序号索引的线性结构，这种链式存储方式与数组的连续存储方式相比，内存的利用率更高），按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。
Vector属于遗留容器（Java早期的版本中提供的容器，除此之外，Hashtable、Dictionary、BitSet、Stack、Properties都是遗留容器），已经不推荐使用，但是由于ArrayList和LinkedListed都是非线程安全的，如果遇到多个线程操作同一个容器的场景，则可以通过工具类Collections中的synchronizedList方法将其转换成线程安全的容器后再使用(这是对装潢模式的应用，将已有对象传入另一个类的构造器中创建新的对象来增强实现)。

## 40.判断List、Set、Map是否继承自Collection接口？
List、Set 是，Map 不是。Map是键值对映射容器，与List和Set有明显的区别，而Set存储的零散的元素且不允许有重复元素（数学中的集合也是如此），List是线性结构的容器，适用于按数值索引访问元素的情形。

## 41.你所知道的常用集合类以及主要方法？
最常用的集合类是List 和 Map。
- LinkedList类
　　LinkedList实现了List接口，允许null元素。此外LinkedList提供额外的get，remove，insert方法在LinkedList的首部或尾部。这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。
　　注意LinkedList没有同步方法。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：
　　　　List list = Collections.synchronizedList(new LinkedList(...));

- ArrayList类
　　ArrayList实现了可变大小的数组。它允许所有元素，包括null。ArrayList没有同步。
size，isEmpty，get，set方法运行时间为常数。但是add方法开销为分摊的常数，添加n个元素需要O(n)的时间。其他的方法运行时间为线性。
　　每个ArrayList实例都有一个容量（Capacity），即用于存储元素的数组的大小。这个容量可随着不断添加新元素而自动增加，但是增长算法并没有定义。当需要插入大量元素时，在插入前可以调用ensureCapacity方法来增加ArrayList的容量以提高插入效率。
　　和LinkedList一样，ArrayList也是非同步的（unsynchronized）。

## 42.说明Collection和Collections的区别。
Collection是集合类的上级接口，继承与他的接口主要有Set 和List.
Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。

## 43.ArrayList和LinkedList的区别？
ArrayList和LinkedList都实现了List接口，他们有以下的不同点：
ArrayList是基于索引的数据接口，它的底层是数组。它可以以O(1)时间复杂度对元素进行随机访问。与此对应，LinkedList是以元素列表的形式存储它的数据，每一个元素都和它的前一个和后一个元素链接在一起，在这种情况下，查找某个元素的时间复杂度是O(n)。
相对于ArrayList，LinkedList的插入，添加，删除操作速度更快，因为当元素被添加到集合任意位置的时候，不需要像数组那样重新计算大小或者是更新索引。
LinkedList比ArrayList更占内存，因为LinkedList为每一个节点存储了两个引用，一个指向前一个元素，一个指向下一个元素。

## 44.说明HashMap和Hashtable的区别？
HashMap和Hashtable都实现了Map接口，因此很多特性非常相似。但是，他们有以下不同点：
HashMap允许键和值是null，而Hashtable不允许键或者值是null。
Hashtable是同步的，而HashMap不是。因此，HashMap更适合于单线程环境，而Hashtable适合于多线程环境。
HashMap提供了可供应用迭代的键的集合，因此，HashMap是快速失败的。另一方面，Hashtable提供了对键的列举(Enumeration)。
一般认为Hashtable是一个遗留的类。

## 45.快速失败(fail-fast)和安全失败(fail-safe)的区别？
Iterator的快速失败是指在遍历集合时修改集合,会导致遍历时维护的`modcount`的值非预期,因此会抛出ConcurrentModificationException异常
Iterator的安全失败是基于对底层集合做拷贝，因此，它不受源集合上修改的影响。
java.util包下面的所有的集合类都是快速失败的，而java.util.concurrent包下面的所有的类都是安全失败的。快速失败的迭代器会抛出ConcurrentModificationException异常，而安全失败的迭代器永远不会抛出这样的异常。

## 46.Iterator和ListIterator的区别？
Iterator和ListIterator的区别是：
Iterator可用来遍历Set和List集合，但是ListIterator只能用来遍历List。
Iterator对集合只能是前向遍历，ListIterator既可以前向也可以后向。
ListIterator实现了Iterator接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。

## 47.什么是迭代器？
Iterator提供了统一遍历操作集合元素的统一接口, Collection接口实现Iterable接口,
每个集合都通过实现Iterable接口中iterator()方法返回Iterator接口的实例, 然后对集合的元素进行迭代操作.
有一点需要注意的是：在迭代元素的时候不能通过集合的方法删除元素, 否则会抛出ConcurrentModificationException 异常. 但是可以通过Iterator接口中的remove()方法进行删除.
remove()方法必须跟在next()方法后面,且每次next()调用完尽可调用一次remove(),用于删除上个元素

## 48.为什么集合类没有实现Cloneable和Serializable接口？
克隆(cloning)或者是序列化(serialization)的语义和含义是跟具体的实现相关的。因此，应该由集合类的具体实现来决定如何被克隆或者是序列化。
实现Serializable序列化的作用：将对象的状态保存在存储媒体中以便可以在以后重写创建出完全相同的副本；按值将对象从一个从一个应用程序域发向另一个应用程序域。
实现 Serializable接口的作用就是可以把对象存到字节流，然后可以恢复。所以你想如果你的对象没有序列化，怎么才能进行网络传输呢？要网络传输就得转为字节流，所以在分布式应用中，你就得实现序列化。如果你不需要分布式应用，那就没必要实现实现序列化。

## 49.Java集合类框架的基本接口有哪些？
集合类接口指定了一组叫做元素的对象。集合类接口的每一种具体的实现类都可以选择以它自己的方式对元素进行保存和排序。有的集合类允许重复的键，有些不允许。
Java集合类提供了一套设计良好的支持对一组对象进行操作的接口和类。Java集合类里面最基本的接口有：
Collection：代表一组对象，每一个对象都是它的子元素。
Set：不包含重复元素的Collection。
List：有顺序的collection，并且可以包含重复元素。
Map：可以把键(key)映射到值(value)的对象，键不能重复。

## 50.说明一下ConcurrentHashMap的原理？
(jdk1.7)ConcurrentHashMap 类中包含两个静态内部类 HashEntry 和 Segment。HashEntry 用来封装映射表的键 / 值对；Segment 用来充当锁的角色，每个 Segment 对象守护整个散列映射表的若干个桶。每个桶是由若干个 HashEntry 对象链接起来的链表。一个 ConcurrentHashMap 实例中包含由若干个 Segment 对象组成的数组。HashEntry 用来封装散列映射表中的键值对。在 HashEntry 类中，key，hash 和 next 域都被声明为 final 型，value 域被声明为 volatile 型。
在ConcurrentHashMap 中，在散列时如果产生“碰撞”，将采用“分离链接法”来处理“碰撞”：把“碰撞”的 HashEntry 对象链接成一个链表。由于 **HashEntry 的 next 域为 final 型**，所以新节点只能在链表的表头处插入。 下图是在一个空桶中依次插入 A，B，C 三个 HashEntry 对象后的结构图：
![](https://uploadfiles.nowcoder.com/images/20180925/308572_1537878284540_B6C31D01D41C9E1714958F9C56D01D8F)
注意：由于只能在表头插入，所以链表中节点的顺序和插入的顺序相反。
Segment 类继承于 ReentrantLock 类，从而使得 Segment 对象能充当锁的角色。每个 Segment 对象用来守护其（成员对象 table 中）包含的若干个桶。

## 51.解释一下TreeMap
TreeMap是一个有序的key-value集合，基于红黑树（Red-Black tree）的 NavigableMap实现。该映射根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator进行排序，具体取决于使用的构造方法。
TreeMap的特性：
根节点是黑色
每个节点都只能是红色或者黑色
每个叶节点（NIL节点，空节点）是黑色的。
如果一个节点是红色的，则它两个子节点都是黑色的，也就是说在一条路径上不能出现两个红色的节点。
从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

## 52.concurrenthashmap有什么优势以及1.7和1.8区别
Concurrenthashmap线程安全的，1.7是在jdk1.7中采用Segment + HashEntry的方式进行实现的，lock加在Segment上面。1.7size计算是先采用不加锁的方式，连续计算元素的个数，最多计算3次：1、如果前后两次计算结果相同，则说明计算出来的元素个数是准确的；2、如果前后两次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数；

1.8中放弃了Segment臃肿的设计，取而代之的是采用Node + CAS + Synchronized来保证并发安全进行实现，1.8中使用一个volatile类型的变量baseCount记录元素的个数，当插入新数据或则删除数据时，会通过addCount()方法更新baseCount，通过累加baseCount和CounterCell数组中的数量，即可得到元素的总个数；

## 53.HashMap的容量为什么是2的n次幂？
负载因子默认是0.75， 2^n是为了让散列更加均匀，例如出现极端情况都散列在数组中的一个下标，那么hashmap会由O（1）复杂退化为O（n）的。

## 54.如果hashMap的key是一个自定义的类，怎么办？
如果key是自定义的类，就必须重写hashcode()和equals()。

## 55.hashMap具体如何实现的？
1. Hashmap基于数组实现的，通过对key的hashcode的高16位和低16位异或(扰动函数) & 数组的长度-1(相当于对数组长度取余, 数组长度为2n的原因)得到在数组中位置
2. 如果该位置已经有元素(哈希冲突),该位置就变成链表. jdk1.7在并发下会形成循环链表, jdk1.8修复了. (出现循环链表的原因是jdk1.7形成新的链表是原链表的反向, jdk1.8使用头尾两个指针来使得新链表与原链表同向),同时,jdk1.8在节点数超过64个时会转成红黑树来提升链表索引效率

## 56.说明一下Map和ConcurrentHashMap的区别
hashmap是线程不安全的，put时在多线程情况下，会形成环从而导致死循环。CoucurrentHashMap是线程安全的，采用分段锁机制，减少锁的粒度。

# 项目

## 57. 你负责了项目的哪块内容？
## 58. 项目的难点痛点是什么？你们怎么解决的？
## 59. 你使用XX技术栈的时候有没有什么坑，你们怎么解决的？
## 60. 项目中遇到过什么印象比较深的Bug？
## 70. 遇到XX情况么？怎么解决的，怎么优化的？能多说几种方案么？
## 71. 你是根据哪些指标进行针对性优化的？

# 数据库

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
