# Go语言基础必备知识点(八) 接口篇

在Go语言中接口 (interface) 是一种类型，一种抽象的类型

interface是一组method的集合，接口做的事情就像是定义一个协议 (规则) ，只要一台机器有洗衣服和甩干的功能，我就称它为洗衣机。不关心属性 (数据) ，只关心行为 (方法) 

**接口 (interface) 是一种类型**

接口类型是对其它类型行为的抽象和概括；因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式我们可以让我们的函数更加灵活和更具有适应能力

接口是双方约定的一种合作协议。接口实现者不需要关心接口会被怎样使用，调用者也不需要关心接口的实现细节。接口是一种类型，也是一种抽象结构，不会暴露所含数据的格式、类型及结构

## 1. 为什么要使用接口

```go
type Cat struct{}

func (c Cat) Say() string { return "喵喵喵" }

type Dog struct{}

func (d Dog) Say() string { return "汪汪汪" }

func main() {
    c := Cat{}
    fmt.Println("猫:", c.Say())
    d := Dog{}
    fmt.Println("狗:", d.Say())
}
```

上面的代码中定义了猫和狗，然后它们都会叫，你会发现 main 函数中明显有重复的代码，如果我们后续再加上猪、青蛙等动物的话，我们的代码还会一直重复下去。那我们能不能把它们当成 “能叫的动物” 来处理呢？

像类似的例子在我们编程过程中会经常遇到：

比如一个网上商城可能使用支付宝、微信、银联等方式去在线支付，我们能不能把它们当成 “支付方式” 来处理呢？

比如三角形，四边形，圆形都能计算周长和面积，我们能不能把它们当成 “图形” 来处理呢？

比如销售、行政、程序员都能计算月薪，我们能不能把他们当成 “员工” 来处理呢？

Go语言中为了解决类似上面的问题，就设计了接口这个概念。接口区别于我们之前所有的具体类型，接口是一种**抽象**的类型。当你看到一个接口类型的值时，你不知道它是什么，唯一知道的是**通过它的方法能做什么**

## 2. 接口定义

**Go语言提倡面向接口编程**

每个接口类型由数个方法组成，接口的形式代码如下：

```go
type 接口类型名 interface{
    方法名1( 参数列表1 ) 返回值列表1
    方法名2( 参数列表2 ) 返回值列表2
    …
}
```

对各个部分的说明：

- 接口类型名：使用 type 将接口定义为自定义的类型名。**Go语言的接口在命名时，一般会在单词后面添加 er**，如有写操作的接口叫 Writer，有字符串功能的接口叫 Stringer，有关闭功能的接口叫 Closer 等
- 方法名：当**方法名首字母是大写**时，且这个**接口类型名首字母也是大写**时，这个方法可以**被接口所在的包 (package) 之外的代码访问**
- 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以被忽略

```go
type Writer interface {
    //大写字母开头 意味着别的包 也可以访问
    Write([]byte) error
}
```

## 3. 接口实现条件

如果一个任意类型 T 的方法集为一个接口类型的方法集的超集，则我们说类型 T 实现了此接口类型

T 可以是一个非接口类型，也可以是一个接口类型

实现关系在Go语言中是隐式的，两个类型之间的实现关系不需要在代码中显式地表示出来。Go语言中没有类似于 implements 的关键字。 Go编译器将自动在需要的时候检查两个类型之间的实现关系

接口定义后，需要实现接口，调用方才能正确编译通过并使用接口

接口的实现需要遵循两条规则才能让接口可用：

1. 接口的方法与实现接口的类型方法格式一致

   > 在类型中添加与接口签名一致的方法就可以实现该方法
   >
   > 签名包括方法中的名称、参数列表、返回参数列表
   >
   > 也就是说，只要实现接口类型中的方法的名称、参数列表、返回参数列表中的任意一项与接口要实现的方法不一致，那么接口的这个方法就不会被实现

   示例：

   ```go
   package main

   import "fmt"

   // 定义一个数据写入器 (接口)
   type DataWriter interface {
       WriteData(data interface{}) error  // interface{}接口类型，为空代表任意类型
   }

   // 定义文件结构，用于实现DataWriter
   type file struct {
   }

   // 实现DataWriter接口的WriteData方法
   func (d *file) WriteData(data interface{}) error {
       // 模拟写入数据
       fmt.Println("WriteData:", data)
       return nil
   }

   func main() {
       // 实例化file
       f := new(file)
       // 声明一个DataWriter的接口
       var writer DataWriter
       // 将接口赋值f，也就是*file类型
       writer = f
       // 使用DataWriter接口进行数据写入
       writer.WriteData("data")
   }
   ```

    OUTPUT：

    ```go
    WriteData: data
    ```

   **当类型无法实现接口时，编译器会报错：**

   * 函数名不一致导致的报错
   * 实现接口的方法签名不一致导致的报错

   

2. 接口中所有方法均被实现

   当一个接口中有多个方法时，只有这些方法都被实现了，接口才能被正确编译并使用。

   ```go
   // 定义一个数据写入器
   type DataWriter interface {
       WriteData(data interface{}) error
       // 新增一个方法 能否写入
       CanWrite() bool
   }
   ```

   在此运行上述的程序，就会报错：

   ```go
   cannot use f (type *file) as type DataWriter in assignment:
   	*file does not implement DataWriter (missing CanWrite method)
   ```

   需要在 file 中实现 CanWrite() 方法才能正常使用 DataWriter()

   Go语言的接口实现是隐式的，无须让实现接口的类型写出实现了哪些接口

   **这个设计被称为非侵入式设计**

## 4. 类型与接口的关系

> 在Go语言中类型和接口之间有一对多和多对一的关系

**一个类型可以实现多个接口**

一个类型可以同时实现多个接口，而接口间彼此独立，不知道对方的实现

例如，狗可以叫，也可以动

我们就分别定义Sayer接口和Mover接口，如下： 

```go
// Sayer 接口
type Sayer interface {
    say()
}

// Mover 接口
type Mover interface {
    move()
}
```

dog既可以实现Sayer接口，也可以实现Mover接口

```go
type dog struct {
    name string
}

// 实现Sayer接口
func (d dog) say() {
    fmt.Printf("%s会叫汪汪汪\n", d.name)
}

// 实现Mover接口
func (d dog) move() {
    fmt.Printf("%s会动\n", d.name)
}

func main() {
    var x Sayer
    var y Mover

    var a = dog{name: "旺财"}
    x = a
    y = a   // a.say()只是普通的对象调用方法，而不是利用接口来调用犯法
    x.say()
    y.move()
}
```

OUTPUT：

```go
旺财会叫汪汪汪
旺财会动
```

**多个类型实现同一接口**

Go语言中不同的类型还可以实现同一接口，首先我们定义一个Mover接口，它要求必须有一个move方法

```go
// Mover 接口
type Mover interface {
    move()
}
```

例如：狗可以动，汽车也可以动，可以使用如下代码实现这个关系：

```go
type dog struct {
    name string
}

type car struct {
    brand string
}

// dog类型实现Mover接口
func (d dog) move() {
    fmt.Printf("%s会跑\n", d.name)
}

// car类型实现Mover接口
func (c car) move() {
    fmt.Printf("%s速度70迈\n", c.brand)
}
```

这个时候我们在代码中就可以把狗和汽车当成一个会动的物体来处理了，不再需要关注它们具体是什么，只需要调用它们的move方法就可以了

```go
func main() {
    var x Mover
    var a = dog{name: "旺财"}
    var b = car{brand: "保时捷"}
    x = a
    x.move()
    x = b
    x.move()
}
```

并且一个接口的方法，不一定需要由一个类型完全实现，接口的方法可以通过在类型中嵌入其他类型或者结构体来实现

```go
// WashingMachine 洗衣机
type WashingMachine interface {
    wash()
    dry()
}

// 甩干器
type dryer struct{}

// 实现WashingMachine接口的dry()方法
func (d dryer) dry() {
    fmt.Println("甩一甩")
}

// 海尔洗衣机
type haier struct {
    dryer //嵌入甩干器
}

// 实现WashingMachine接口的wash()方法
func (h haier) wash() {
    fmt.Println("洗刷刷")
}
```

**接口嵌套**

接口与接口间可以通过嵌套创造出新的接口

```go
// Sayer 接口
type Sayer interface {
    say()
}

// Mover 接口
type Mover interface {
    move()
}

// 接口嵌套
type animal interface {
    Sayer
    Mover
}
```

嵌套得到的接口的使用与普通接口一样，这里我们让cat实现animal接口：

```go
type cat struct {
    name string
}

func (c cat) say() {
    fmt.Println("喵喵喵")
}

func (c cat) move() {
    fmt.Println("猫会动")
}

func main() {
    var x animal
    x = cat{name: "花花"}
    x.move()
    x.say()
}
```

## 5. 空接口

空接口是指没有定义任何方法的接口

因此**任何类型都实现了空接口**

空接口类型的变量可以存储**任意类型**的变量

```go
func main() {
    // 定义一个空接口x
    var x interface{}
    s := "码神之路"
    x = s
    fmt.Printf("type:%T value:%v\n", x, x)
    i := 100
    x = i
    fmt.Printf("type:%T value:%v\n", x, x)
    b := true
    x = b
    fmt.Printf("type:%T value:%v\n", x, x)
}
```

OUTPUT：

```go
type:string value:Chris Liu
type:int value:100
type:bool value:true
```

### 5.1 空接口的应用

**空接口作为函数的参数**

使用空接口实现可以接收任意类型的函数参数

```go
// 空接口作为函数参数
func show(a interface{}) {
    fmt.Printf("type:%T value:%v\n", a, a)
}
```

**空接口作为map的值**

使用空接口实现可以保存任意值的字典

```go
// 空接口作为map值
    var studentInfo = make(map[string]interface{})
    studentInfo["name"] = "李白"
    studentInfo["age"] = 18
    studentInfo["married"] = false
    fmt.Println(studentInfo)
```

### 5.2 类型断言

空接口可以存储任意类型的值，那我们如何获取其存储的具体数据呢？

**接口值**

一个接口的值 (简称接口值) 是由一个具体类型和具体类型的值两部分组成的

这两部分分别称为**接口的动态类型**和**动态值**

想要判断空接口中的值这个时候就可以使用类型断言，其语法格式：

```go
x.(T)
```

其中：

* x：表示类型为 interface{} 的变量

* T：表示断言 x 可能是的类型

**该语法返回两个参数，第一个参数是 x 转化为 T 类型后的变量，第二个值是一个布尔值，若为 true 则表示断言成功，为 false 则表示断言失败**

   ```go
   func main() {
       var x interface{}
       x = "Chris Liu"
       v, ok := x.(string)
       if ok {
           fmt.Println(v)
       } else {
           fmt.Println("类型断言失败")
       }
   }
   ```

上面的示例中如果要断言多次就需要写多个 if 判断，这个时候我们可以使用switch语句来实现：

```go
func justifyType(x interface{}) {
    switch v := x.(type) {
    case string:
        fmt.Printf("x is a string，value is %v\n", v)
    case int:
        fmt.Printf("x is a int is %v\n", v)
    case bool:
        fmt.Printf("x is a bool is %v\n", v)
    default:
        fmt.Println("unsupport type！")
    }
}
```

因为空接口可以存储任意类型值的特点，所以空接口在Go语言中的使用十分广泛

> 关于接口需要注意的是，只有当有两个或两个以上的具体类型必须以相同的方式进行处理时才需要定义接口
> 不要为了接口而写接口，那样只会增加不必要的抽象，导致不必要的运行时损耗