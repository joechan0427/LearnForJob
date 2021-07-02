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

## make() 与 new()
new(T) 返回一个指针, 指向一个新分配的, 类型为 T 的零值. 它适用于值类型如**数组**和**结构体**

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