写在后面：由于之前考研408专业基础知识的学习，决定去留学后雅思等英语成绩的学习时间基本取代了编码时间，所以对于Java语法、Java实习和项目、SSM框架、SVM等基础知识已经稍有淡忘。并且本人一直非常看好go语言的天然高并发以及越来越多大厂开始用go重构代码以及用go编写新项目，而且go框架也在发展阶段，好多轮子需要自己造，身边同学有很多Java面试后入职转go。因此，在种种理由下，我打算把go作为主语言备战2024秋招，从零开始熟悉go语言语法、gin框架等并用go刷LeetCode，完成mit 6.824分布式系统的项目等，舍弃之前Java的实习和项目，重新挑战一个新的语言。

# Go语言基础必备知识点(四) map篇

# 1. map定义

map 是一种**无序**的**键值对** (key-value)的集合

map 最重要的一点是通过 key 来快速检索数据，key 类似于索引，指向数据的值 value

map 是一种集合，所以我们可以像迭代数组和切片那样迭代它。不过，map 是无序的，我们无法决定它的返回顺序，这是因为 map 是使用 hash 表来实现的

## 1.1 标准声明方式

```go
//[keytype] 和 valuetype 之间允许有空格
var mapname map[keytype]valuetype
```

其中：

- mapname 为 map 的变量名
- keytype 为键类型
- valuetype 是键对应的值类型

> 在声明的时候不需要知道 map 的长度，因为 map 是可以**动态增长**的，未初始化的 map 的值是 nil，使用函数 len() 可以获取 map 中 键值对的数目

```go
package main

import "fmt"

func main() {
	var mapList map[string]int
	var mapAssigned map[string]int
	//初始化用{key: value}，多个用,隔开
	mapList = map[string]int{"one": 1, "two": 2}
	mapAssigned = mapList
	//mapAssigned 是 mapList 的引用(不是复制)，对 mapAssigned 的修改也会影响到 mapList 的值
	mapAssigned["two"] = 3
	fmt.Printf("mapList[one]: %d\n", mapList["one"])
	fmt.Printf("mapList[two]: %d\n", mapList["two"])
	fmt.Printf("mapList[ten]: %d\n", mapList["ten"])
}
```

OUTPUT：

```go
mapList[one]: 1
mapList[two]: 3
mapList[ten]: 0
```

**注意：**

> **map是引用类型**
> `mapAssigned = mapList`
> mapAssigned 是 mapList 的引用，**对 mapAssigned 的修改也会影响到 mapList 的值**

## 1.2 make()函数声明方式

```go
make(map[keytype]valuetype)
```

**切记不要使用new创建map，否则会得到一个空引用的指针**

map 可以根据新增的 key-value 动态的伸缩，因此它不存在固定长度或者最大限制，但是也可以选择标明 map 的**初始容量 cap (capacity)**，格式如下：

```go
make(map[keytype]valuetype, cap)
```

例如：

```go
mapList := make(map[string]int, 100)
```

> 当 map 增长到容量上限的时候，如果再增加新的 key-value，map 的大小会自动加 1，所以**出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明**

***既然一个 key 只能对应一个 value，而 value 又是一个原始类型，那么如果一个 key 要对应多个值怎么办？**

答案是：使用**切片**

例如，当我们要处理 unix 机器上的所有进程，以父进程 (pid 为整形) 作为 key，所有的子进程 (以所有子进程的 pid 组成的切片) 作为 value

通过将 value 定义为 []int 类型或者其他类型的切片，就可以优雅的解决这个问题，示例代码如下所示：

```go
mp1 := make(map[int][]int)
mp2 := make(map[int]*[]int)
```

# 2. 遍历map

map 的遍历过程使用 `for range` 循环完成，代码如下：

```go
scene := make(map[string]int)
scene["cat"] = 66
scene["dog"] = 4
scene["pig"] = 960
for k, v := range scene {
    fmt.Println(k, v)
}
```

```go
pig 960
cat 66
dog 4     //每次运行输出顺序都可能不固定
```

**注意：map是无序的，不要期望 map 在遍历时返回某种期望顺序的结果**

# 3. 删除键值对

使用 `delete()` 内建函数从 map 中删除一组键值对，`delete()` 函数的格式如下：

```go
delete(map, 键key)
```

map 为要删除的 map 实例，键为要删除的 map 中键值对 (key-value) 的键 (key)

```go
scene := make(map[string]int)
scene["cat"] = 66
scene["dog"] = 4
scene["pig"] = 960
// 删除map键值对dog 4
delete(scene, "dog")
for k, v := range scene {
    fmt.Println(k, v)
}
```

OUTPUT：

```go
pig 960
cat 66   //每次运行输出顺序都可能不固定
```

**Go语言中并没有为 map 提供任何清空所有元素的函数、方法，清空 map 的唯一办法就是重新 make 一个新的 map，不用担心垃圾回收的效率，Go语言中的并行垃圾回收效率比写一个清空函数要高效的多**

**注意 map 在并发情况下，只读是线程安全的，同时读写是线程不安全的**

# *4. 线程安全的map

并发情况下读写 map 时会出现问题，代码如下：

```go
// 创建一个int到int的映射
m := make(map[int]int)
// 开启一段并发代码
go func() {
    // 不停地对map进行写入
    for {
        m[1] = 1
    }
}()
// 开启一段并发代码
go func() {
    // 不停地对map进行读取
    for {
        _ = m[1]
    }
}()
// 无限循环, 让并发程序在后台执行
for {
}
```

运行代码会报错：

`fatal error: concurrent map read and map write`

错误信息显示，并发的 map 读和 map 写，也就是说使用了两个并发函数不断地对 map 进行读和写而发生了竞态问题，map 内部会对这种并发操作进行检查并提前发现

需要并发读写时，一般的做法是**加锁**，但这样**性能并不高**，Go语言在 1.9 版本中提供了一种效率较高的并发安全的 `sync.Map`，sync.Map 和 map 不同，不是以语言原生形态提供，而是在 sync 包下的特殊结构

sync.Map 有以下特性：

- **无须初始化**，直接声明即可，sync.Map 不能使用 make 创建
- sync.Map 不能使用 map 的方式进行取值和设置等操作，而是使用 **sync.Map 的方法进行调用，Store 表示存储，Load 表示获取，Delete 表示删除**
- 使用 Range 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，Range 参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回 false

```go
package main
import (
      "fmt"
      "sync"
)
func main() {
    //sync.Map 不能使用 make 创建
    var scene sync.Map
    //将键值对保存到sync.Map
    //sync.Map 将键和值以 interface{} 类型进行保存。
    scene.Store("greece", 97)
    scene.Store("london", 100)
    scene.Store("egypt", 200)
    //从sync.Map中根据键取值
    fmt.Println(scene.Load("london"))
    //根据键删除对应的键值对
    scene.Delete("london")
    //遍历所有sync.Map中的键值对
    //遍历需要提供一个匿名函数，参数为 k、v，类型为 interface{}，每次 Range() 在遍历一个元素时，都会调用这个匿名函数把结果返回。
    scene.Range(func(k, v interface{}) bool {
        fmt.Println("iterate:", k, v)
        return true
    })
}
```

**sync.Map 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 map 相比使用 sync.Map 会有更好的性能**