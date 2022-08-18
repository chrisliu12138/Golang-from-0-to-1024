# Go语言基础必备知识点(五) 函数(闭包)、延迟调用、异常处理篇

# 1. 函数

函数可以提高应用的模块性和代码的重复利用率

Go 语言支持**普通函数、匿名函数和闭包**，从设计上对函数进行了优化和改进，让函数使用起来更加方便

Go 语言的函数属于 **"一等公民"** (first-class) ，也就是说：

- 函数本身可以作为值 (参数) 进行传递
- 支持*匿名函数*和*闭包 (closure)*
- 函数可以满足接口

**函数定义:**

```go
func function_name( [parameter list] ) [return_types] {
   函数体
}
```

- function_name：函数名称，函数名和参数列表一起构成了函数签名
- parameter list：参数列表，参数就像一个占位符，当函数被调用时，你可以将值传递给参数，这个值被称为**实际参数**。参数列表指定的是参数类型、顺序、及参数个数。参数是**可选的**，也就是说函数**也可以不包含参数**
- return_types：**返回类型**，函数返回一列值。return_types 是该列值的数据类型。有些功能不需要返回值，这种情况下 return_types **不是必须的**

Go语言是编译型语言，函数编写的顺序是无关紧要的，但是鉴于可读性的需求，**最好把 main() 函数写在文件的前面**，其他函数按照一定**逻辑顺序**进行编写 (例如函数**被调用的顺序**) 

**返回值可以为多个：**

```go
package main

import "fmt"

func main() {
	fmt.Println(test(1, 2, "求和："))
}
func test(x, y int, s string) (int, string) {
	// 类型相同的相邻参数，参数类型可合并
	// 多返回值必须用括号
	n := x + y
	return n, fmt.Sprintf("%s%d", s, n) // Go 中格式化字符串并赋值给新串，使用 fmt.Sprintf
}

```

OUTPUT：

```go
3 求和：3
```

## *1.1 函数做为参数

> 函数做为一等公民，可以做为**参数**传递 (和其他语言不同)

```go
package main

import "fmt"

func test(fn func() int) int {
	return fn()
}
func fn() int {
	return 200
}
func main() {
	// *这是直接使用匿名函数 这里return 100有点奇怪
	s1 := test(func() int { return 100 })
	// 这是传入一个函数
	s2 := test(fn)
	fmt.Println(s1)
	fmt.Println(s2)
}
```

OUTPUT：

```go
100
200
```

***在将函数做为参数的时候，我们可以使用类型定义，将函数定义为类型，这样便于阅读**

```go
// 定义函数类型 给函数func(s string, x, y int) string起名FormatFunc
type FormatFunc func(s string, x, y int) string

// func format(func(s string, x, y int) string, s string, x, y int) string {
// 不定义函数类型阅读性不强

func format(fn FormatFunc, s string, x, y int) string {
	return fn(s, x, y)
}
func formatFun(s string,x,y int) string  {
	return fmt.Sprintf(s,x,y)
}
func main() {
    s2 := format(formatFun,"%d, %d",10,20)
	fmt.Println(s2)
}
```

**有返回值**的函数，必须有明确的**终止语句 return**，否则会引发编译错误

## 1.2 函数返回值

> 函数返回值可以有**多个**，同时 Go 支持**对返回值命名**

```go
// 多个返回值用括号扩起来
func sum(a, b int) (int, int) {
	return a, b
}
func main() {
	a, b := sum(2, 3)
	fmt.Println(a, b)
}
```

```go
package main

import "fmt"
// 支持返回值 命名，默认值为类型零值,命名返回参数可看做与形参类似的局部变量,由 return 隐式返回

/* 不命名应写为:
func f1() ([]string, map[string]int, int) {
    var names []string
    ... */

func f1() (names []string, m map[string]int, num int) {
   m = make(map[string]int)  // map必须make定义分配下空间才能使用
   m["k1"] = 2

   return   // 因为变量名已经定义好了 默认返回names, m, num
   // 写return names, m, num 也是可以的
}

func main() {
   a, b, c := f1()
   fmt.Println(a, b, c)
}
```

OUTPUT：

```go
[] map[k1:2] 0
```

## 1.3 参数

> 函数定义时指出，函数定义时有参数，该变量可称为函数的**形参**
>
> 形参就像定义在**函数体内的局部变量**

但当**调用函数**，传递过来的变量就是函数的**实参**，函数可以通过两种方式来传递参数：

1. **值传递**：指在调用函数时将**实际参数复制一份**传递到函数中，这样在函数中如果**对参数进行修改，将不会影响到实际参数**

   ```go
   func swap(x, y int) int {
          ... ...
     }
   ```

2. **引用传递**：是指在调用函数时将**实际参数的地址**传递到函数中，那么在函数中**对参数所进行的修改，将影响到实际参数**

   ```go
   package main
   
   import "fmt"
   
   // 定义相互交换值的函数 (无返回值) 指针接收
   func swap(x, y *int) {
   	*x, *y = *y, *x
   }
   
   func main() {
   	var a, b int = 1, 2
   	/*
   	   调用 swap() 函数
   	   &a 指向 a 指针，a 变量的地址
   	   &b 指向 b 指针，b 变量的地址
   	*/
   	swap(&a, &b)
   
   	fmt.Println(a, b)    //2 1
   }
   ```

在**默认情况下，Go 语言使用的是值传递**，即在调用过程中不会影响到实际参数

> 注意1：无论是值传递，还是引用传递，传递给函数的都是变量的副本，不过，值传递是**值**的拷贝，引用传递是**地址**的拷贝。一般来说，地址拷贝更为高效，而值拷贝取决于拷贝的对象大小，对象越大，则性能越低
>
> **注意2：map、slice、chan、指针、interface默认以引用的方式传递**

**不定参数传值**

不定参数传值 就是函数的参数不是固定的，后面的类型是固定的 (可变参数) 

Go 可变参数本质上就是 切片slice，只能有一个，且必须是最后一个

在参数赋值时可以不用用一个一个的赋值，可以直接传递一个**数组**或**切片**

格式:

```go
  func myfunc(args ...int) {    //0个或多个参数
  }

  func add(a int, args ...int) int {    //1个或多个参数
  }

  func add(a int, b int, args ...int) int {    //2个或多个参数
  }
```

**注意：其中 args 是一个 slice，我们可以通过 `arg[index]` 依次访问所有参数，通过 `len(arg)` 来判断传递参数的个数**

```go
package main

import "fmt"
// 求
func test(s string, n ...int) string {
    var x int
    for _, i := range n {  //for range遍历切片n
        x += i             //累加求和
    }

    return fmt.Sprintf(s, x)
}

func main() {
    s := []int{1, 2, 3, 4}

    // 如果以切片s := []int{1, 2, 3, 4}传参 一定要带...
    res := test("sum: %d", s...)    // slice... 展开slice

    //否则也可以 res := test("sum: %d", 1, 2, 3, 4) 逗号传参
    println(res)
}
```

*OUTPUT：

```go
sum: 10
```

# 2. 匿名函数

> 匿名函数是指**不需要定义函数名**的一种函数实现方式

在Go里面，函数可以像普通变量一样被传递或使用，Go语言支持随时在代码里定义匿名函数

匿名函数由一个不带函数名的函数声明和函数体组成。匿名函数的优越性在于**可以直接使用函数内的变量，不必声明**

匿名函数的定义格式如下：

```go
func(参数列表)(返回参数列表){
    函数体
}
```

示例：

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    //这里将一个函数当做一个变量一样的操作
    getSqrt := func(a float64) float64 {
        return math.Sqrt(a)
    }
    fmt.Println(getSqrt(4))
}
```

***在定义时调用匿名函数**

匿名函数可以在**声明后调用**，例如：

```go
func(data int) {
    fmt.Println("hello", data)
}(100) //(100)，表示对匿名函数进行调用，传递参数为 100
```

**匿名函数用作回调函数**

匿名函数作为回调函数的设计在Go语言也比较常见

```go
package main

import "fmt"

// 遍历切片的每个元素, 通过给定函数进行元素访问
func visit(list []int, f func(int)) {
    for _, v := range list {
        f(v)
    }
}
func main() {
    // 使用匿名函数打印切片内容
    visit([]int{1, 2, 3, 4}, func(v int) {
        fmt.Println(v)
    })
}
```

**返回多个匿名函数**

```go
package main

import "fmt"

func FGen(x, y int) (func() int, func(int) int) {

	//求和的匿名函数
	sum := func() int {
		return x + y
	}

	// (x+y) *z 的匿名函数
	avg := func(z int) int {
		return (x + y) * z
	}
	return sum, avg
}

func main() {

	f1, f2 := FGen(1, 2)
	fmt.Println(f1())
	fmt.Println(f2(3))
}
```

# 3. 闭包

> 所谓“闭包”，指的是一个拥有许多变量和绑定了这些变量的环境的表达式 (通常是一个函数) ，因而这些变量也是该表达式的一部分
>
> 闭包=函数+引用环境

示例：

```go
package main

import "fmt"

// 创建一个玩家生成器, 输入名称, 输出生成器
func playerGen(name string) func() (string, int) {
	// 血量一直为150
	hp := 150
	// 返回创建的闭包
	return func() (string, int) {
		// 将变量引用到闭包中
		return name, hp
	}
}

// 创建一个玩家生成器, 输入名称, 输出生成器
func playerGen1() func(string) (string, int) {
	// 血量一直为150
	hp := 150
	// 返回创建的闭包
	return func(name string) (string, int) {
		// 将变量引用到闭包中
		return name, hp
	}
}
func main() {
	// 创建一个玩家生成器
	generator := playerGen("Chris")
	// 返回玩家的名字和血量
	name, hp := generator()
	// 打印值
	fmt.Println(name, hp)

	generator1 := playerGen1()
	name1,hp1 := generator1("Chris")
	// 打印值
	fmt.Println(name1, hp1)
}
```

OUTPUT：

```go
Chris 150
Liu 150
```

# 4. 延迟调用

> Go语言的 **defer 语句**会将其后面跟随的语句进行延迟处理

 **defer特性:**

1. 关键字 defer 用于注册延迟调用
2. 这些调用直到 return 前才被执行，因此，可以用来做资源清理
3. 多个defer语句，按**先进后出**的方式执行
4. defer语句中的变量，在defer声明时就决定了

**defer的用途：**

1. 关闭文件句柄
2. 锁资源释放
3. 数据库连接释放

**go 语言的defer功能强大，对于资源管理非常方便，但是如果没用好，也会有陷阱**

```go
package main

import "fmt"

func main() {
	var whatever = [5]int{1,2,3,4,5}

	for i := range whatever {
		defer fmt.Println(i)
	}
}
```

看下面的示例：

```go
package main

import (
	"log"
	"time"
)

func main() {
	start := time.Now()
	log.Printf("开始时间为：%v", start)
  defer log.Printf("时间差：%v", time.Since(start))  // Now()此时已经copy进去了
    // 不受这3秒睡眠的影响
	time.Sleep(3 * time.Second)

	log.Printf("函数结束")
}
```

* Go 语言中所有的`函数调用都是传值的`
* 调用 defer 关键字会`立刻拷贝函数中引用的外部参数` ，包括 start 和 time.Since 中的Now
* defer的函数在`压栈的时候也会保存参数的值，并非在执行时取值`

如何解决上述问题：使用 defer fun()

```go
package main

import (
	"log"
	"time"
)

func main() {
	start := time.Now()
	log.Printf("开始时间为：%v", start)
	defer func() {
		log.Printf("开始调用defer")
		log.Printf("时间差：%v", time.Since(start))
		log.Printf("结束调用defer")
	}()
	time.Sleep(3 * time.Second)

	log.Printf("函数结束")
}
```

**因为拷贝的是`函数指针`,函数属于引用传递**

再来看一个问题：

```go
package main

import "fmt"

func main() {
	var whatever = [5]int{1,2,3,4,5}
	for i,_ := range whatever {
        // 函数正常执行,由于闭包用到的变量 i 在执行的时候已经变成4，所以输出全都是4
		defer func() { fmt.Println(i) }()
	}
}
```

怎么解决：

```go
package main

import "fmt"

func main() {
	var whatever = [5]int{1,2,3,4,5}
	for i,_ := range whatever {
		i := i
		defer func() { fmt.Println(i) }()
	}
}
```

# 5. 异常处理

> Go语言中使用 panic 抛出错误，recover 捕获错误

异常的使用场景简单描述：Go中可以抛出一个panic的异常，然后在defer中通过recover捕获这个异常，然后正常处理

**panic：**

1. 内置函数
2. 假如函数 F 中书写了 panic 语句，会终止其后要执行的代码，在 panic 所在函数 F 内如果存在要执行的 defer 函数列表，按照 defer 的逆序执行
3. 返回函数 F 的调用者G ，在 G 中，调用函数 F 语句之后的代码不会执行，假如函数 G 中存在要执行的 defer 函数列表，按照 defer 的逆序执行
4. 直到 goroutine 整个退出，并报告错误

**recover：**

1. 内置函数
2. 用来捕获panic，从而影响应用的行为

> Go 的错误处理流程：当一个函数在执行过程中出现了异常或遇到 panic()，正常语句就会立即终止，然后执行 defer 语句，再报告异常信息，最后退出 goroutine。如果在 defer 中使用了 recover() 函数，则会捕获错误信息，使该错误信息终止报告

**注意:**

1. 利用 recover 处理 panic 指令，defer 必须放在 panic 之前定义，另外 recover 只有在 defer 调用的函数中才有效。否则当 panic 时，recover 无法捕获到 panic，无法防止 panic 扩散
2. recover 处理异常后，逻辑并不会恢复到 panic 那个点去，函数跑到 defer 之后的那个点
3. 多个 defer 会形成 defer 栈，后定义的 defer 语句会被最先调用

```go
package main

func main() {
    test()
}

func test() {
    defer func() {
        if err := recover(); err != nil {
            println(err.(string)) // 将 interface{} 转型为具体类型。
        }
    }()

    panic("panic error!")
}
```

由于 panic、recover 参数类型为 interface{}，因此可抛出任何类型对象

```go
 func panic(v interface{})
 func recover() interface{}
```

**延迟调用中引发的错误，可被后续延迟调用捕获，但仅最后一个错误可被捕获:**

```go
package main

import "fmt"

func test() {
    defer func() {
        // defer panic 会打印
        fmt.Println(recover())
    }()

    defer func() {
        panic("defer panic")
    }()

    panic("test panic")
}

func main() {
    test()
}
```

**如果需要保护代码段，可将代码块重构成匿名函数，如此可确保后续代码被执 ：**

```go
package main

import "fmt"

func test(x, y int) {
    var z int

    func() {
        defer func() {
            if recover() != nil {
                z = 0
            }
        }()
        panic("test panic")
        z = x / y
        return
    }()

    fmt.Printf("x / y = %d\n", z)
}

func main() {
    test(2, 1)
}
```

**除用 panic 引发中断性错误外，还可返回 error 类型错误对象来表示函数调用状态:**

```go
type error interface {
    Error() string
}
```

标准库 `errors.New` 和 `fmt.Errorf `函数用于创建实现 error 接口的错误对象。通过判断错误对象实例来确定具体错误类型

```go
package main

import (
    "errors"
    "fmt"
)

var ErrDivByZero = errors.New("division by zero")

func div(x, y int) (int, error) {
    if y == 0 {
        return 0, ErrDivByZero
    }
    return x / y, nil
}

func main() {
    defer func() {
        fmt.Println(recover())
    }()
    switch z, err := div(10, 0); err {
    case nil:
        println(z)
    case ErrDivByZero:
        panic(err)
    }
}
```

**Go实现类似 try catch 的异常处理:**

```go
package main

import "fmt"

func Try(fun func(), handler func(interface{})) {
    defer func() {
        if err := recover(); err != nil {
            handler(err)
        }
    }()
    fun()
}

func main() {
    Try(func() {
        panic("test panic")
    }, func(err interface{}) {
        fmt.Println(err)
    })
}
```

**如何区别使用 panic 和 error 两种方式?**

惯例是:导致关键流程出现不可修复性错误的使用 panic，其他使用 error