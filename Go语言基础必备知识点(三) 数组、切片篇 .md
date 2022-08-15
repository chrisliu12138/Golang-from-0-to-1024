# Go语言基础必备知识点(三) 数组、切片篇  

# 1. 数组

数组是一个由**固定长度**的特定类型元素组成的序列，一个数组可以由**零个**或多个元素组成，[注意] 数组是定长的，不可更改，在编译阶段就决定了

**因为数组的长度是固定的，所以在Go语言中很少直接使用数组**

1. 数组声明：

    ```go
    var arrName [size]Type
    ```

- arrName：数组声明及使用时的变量名(数组名)
- size：数组的元素数量，**可以是一个表达式，但最终通过编译期计算的结果必须是整型数值**，元素数量不能含有到运行时才能确认大小的数值
- Type：可以是任意基本类型，包括数组本身，**类型为数组本身时，可以实现多维数组**

2. 数组赋值：

   *(如果初始化不赋值，会默认赋值为0) (与其他语言不同！)*

* 初始化时候赋值 (常用短变量) (注意 [...] 的使用)

   ```go
   //如果第三个不赋值，就是默认值0
   var arr [3]int = [3]int{1,2}    //1 2 0
   ```

    ```go
    //可以使用简短声明(短变量)
    arr := [3]int{1,2}          //1 2 0
    ```

    ```go
    //如果不写数据数量，而使用...，表示数组的长度是根据初始化值的个数来计算
    arr := [...]int{1,2,3}
    //标准声明赋值只可以在右边使用...，并且赋值变量个数要和左边定义的相同
    var arr [3]int = [...]int{1, 2, 3}
    
    //var arr [3]int = [...]int{1, 2}  //error
    ```

* 通过索引下标赋值

    ```go
    var arr [3]int
    arr[0] = 5
    arr[1] = 6
    // arr[2]不赋值默认为0
    ```

* ***如果想要只初始化某个索引的值**

    ```go
    //给索引为2的赋值3
    arr := [3]int{2:3}  // 0,0,3
    ```

3. 数组取值：

* 通过索引下标取值 (索引从0开始)

    ```go
    fmt.Println(arr[0])
    fmt.Println(arr[1])
    ```

* **for range循环取值**

    ```go
    for index,value := range arr{
        fmt.Println("索引:%d,值：%d",index,value)
    }
    ```

4. *数组比较*

   如果两个数组的长度，数组中元素的类型都相同的情况下，我们可以直接通过较运算符（`==`和`!=`）来判断两个数组是否相等，只有当两个数组的所有元素都是相等的时候数组才是相等的，不能比较两个类型不同的数组


# 2. 多维数组

多维数组的所有维度都会在创建时自动初始化零值，多维数组尤其适合**管理具有父子关系或者与坐标系相关联的数据**

1. 多维数组声明：

```go
var arrayName [size1][size2]...[sizen]Type
```

* size1、size2...sizen为数组每一维度的长度

二维数组是最简单的多维数组，二维数组本质上是由多个一维数组组成的

2. 多维数组赋值：

* 初始化时候赋值

    ```go
    // 声明一个二维整型数组，两个维度的长度分别是 4 和 2
    var array [4][2]int
    // 初始化赋值
    array = [4][2]int{{10, 11}, {20, 21}, {30, 31}, {40, 41}}
    ```

    ```go
    // 声明并初始化数组中索引为 1 和 3 的元素
    array = [4][2]int{1: {20, 21}, 3: {40, 41}}
    // 声明并初始化数组中指定的元素
    array = [4][2]int{1: {0: 20}, 3: {1: 41}}
    ```

* **通过索引下标赋值**

    ```go
    // 声明一个 2×2 的二维整型数组
    var array [2][2]int
    // 设置每个元素的整型值
    array[0][0] = 10
    array[0][1] = 20
    array[1][0] = 30
    array[1][1] = 40
    ```
* *只要类型一致，就可以将多维数组**互相赋值**，如下所示：

    ```go
    // 声明两个二维整型数组 [2]int [2]int
    var array1 [2][2]int  
    var array2 [2][2]int
    // 为array2的每个元素赋值
    array2[0][0] = 10
    array2[0][1] = 20
    array2[1][0] = 30
    array2[1][1] = 40
    // 将 array2 的值复制给 array1
    array1 = array2
    ```

* *因为数组中每个元素都是一个值，所以可以**独立复制某个维度**，如下所示：

    ```go
    // 将 array1 的索引为 1 的维度复制到一个同类型的新数组里
    var array3 [2]int = array1[1]
    // 将数组中指定的整型值复制到新的整型变量里
    var value int = array1[1][0]
    ```

3. 多维数组取值：

* 通过索引下标取值

   ```go
   fmt.Println(array[1][0])
   ```

* for range循环取值

    ```go
    for index,value := range array{
        fmt.Printf("索引:%d,值：%d \n",index,value)
    }
    ```
    
# 3. *切片 (Slice)

切片与数组一样，也是可以容纳若干类型相同的元素的容器。与数组不同的是，无法通过切片类型来确定其值的长度

每个切片值都会将数组作为其底层数据结构，我们也把这样的数组称为**切片的底层数组**

切片是对数组的一个连续片段的引用，所以切片是一个**引用类型**。这个片段可以是**整个数组**，也可以是由起始和终止索引标识的一些**项的子集**，需要注意的是，**终止索引标识的项**不包括在切片内(左闭右开的区间)

Go语言中切片的内部结构包含**地址、大小和容量**，切片一般用于快速地操作一块数据集合

**从连续内存区域生成切片是常见的操作，格式如下：**

```go
slice [开始位置 : 结束位置]
```

- slice：表示目标切片对象
- 开始位置：对应目标切片对象的索引
- 结束位置：对应目标切片的结束索引

**从数组生成切片，代码如下：**

```go
var a  = [3]int{1, 2, 3}
//a[1:3] 生成了一个新的切片(左闭右开)
fmt.Println(a, a[1:3])   //[1 2 3] [2 3]
fmt.Println(a, a[1:])    //[1 2 3] [2 3]
```

从数组或切片生成新的切片拥有如下特性：

- 取出的元素数量为：结束位置 - 开始位置
- 取出元素不包含结束位置对应的索引，切片最后一个元素使用 `slice[len(slice)]` 获取
- 当缺省开始位置时，表示从连续区域开头到结束位置 `(a[:2])`
- 当缺省结束位置时，表示从开始位置到整个连续区域末尾 `(a[0:])`
- 两者同时缺省时，与切片本身等效 `(a[:])`
- 两者同时为 0 时，等效于空切片，一般用于**切片复位** `(a[0:0])`

**注意：超界会报运行时错误，比如数组长度为3，则结束位置最大只能为3**

`invalid argument: index 4 out of bounds [0:4]`

> 切片在指针的基础上增加了大小，约束了切片对应的内存区域，**切片使用中无法对切片内部的地址和大小进行手动调整，因此切片比指针更安全、强大**


## 3.1 直接声明新的切片

除了可以从原有的数组或者切片中生成切片外，也可以**声明一个新的切片，每一种类型都可以拥有其切片类型**，表示多个相同类型元素的连续集合

**切片类型声明格式：**

```go
var name []Type
```

* name 表示切片的变量名，Type 表示切片对应的元素类型

```go
// 声明字符串切片
var strList []string
// 声明整型切片
var numList []int
// 声明一个空切片
var numListEmpty = []int{}
// 输出3个切片
fmt.Println(strList, numList, numListEmpty)                 //[] [] []
// 输出3个切片大小
fmt.Println(len(strList), len(numList), len(numListEmpty))  //0 0 0
// 切片判定空的结果
fmt.Println(strList == nil)             //true
fmt.Println(numList == nil)             //true
fmt.Println(numListEmpty == nil)        //false
```

切片是动态结构，**只能与 nil 判定相等，不能互相判定相等**

声明新的切片后，可以**使用 `append()` 函数向切片中添加元素**：

```go
var strList []string
// 追加一个元素
strList = append(strList,"码神之路")
fmt.Println(strList)
```

## 3.2 使用 make() 函数构造切片

如果需要**动态**地创建一个切片，可以使用 `make()` 内建函数，格式如下：

```go
make( []Type, size, cap )
```
 
* `Type` 是指切片的元素类型  
* `size` 指的是为这个类型分配多少个元素  
* `cap `为预分配的元素数量，**这个值设定后不影响 size，只是能提前分配空间，降低多次分配空间造成的性能问题**

```go
a := make([]string, 2)
b := make([]int, 2, 10)
fmt.Println(a, b)             //[ ] [0 0]
//容量cap不会影响当前的元素个数，因此 a 和 b 取 len 都是 2
fmt.Println(len(a), len(b))   //2 2
//给a追加一个元素，a的长度变为3 (扩容) 
//给b追加一个元素，b的长度变为3 (长度小于cap不会进行扩容)
a = append(a, "chris liu")
b = append(b, 1)
fmt.Println(len(a), len(b))   //3 3
```

> 使用 `make()` 函数生成的切片**一定发生了内存分配操作**  

> 但给定开始与结束位置（包括切片复位）的切片只是将新的切片结构指向已经分配好的内存区域，设定开始与结束位置，**不会发生内存分配操作**

## 3.3 思考题

```go
var numbers4 = [...]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
myslice := numbers4[4:6]
//打印出来 myslice 长度为2
fmt.Printf("myslice为 %d, 其长度为: %d\n", myslice, len(myslice))
//为什么容量为6？
fmt.Println("myslice的容量为: %d\n", cap(myslice))
myslice = myslice[:cap(myslice)]
//为什么 myslice 的长度为2，却能访问到第四个元素
fmt.Printf("myslice的第四个元素为: %d", myslice[3])
fmt.Printf("myslice为 %d, 其长度为: %d\n", myslice, len(myslice))
```

OUTPUT：

```go
myslice为 [5 6], 其长度为: 2
myslice的容量为: 6
myslice的第四个元素为: 8
myslice为 [5 6 7 8 9 10], 其长度为: 6
```

原理：

实际容量在分配的时候把数组后边的容量都分给了切片，在执行完`myslice = myslice[:cap(myslice)]`后， `len(myslice)` 也变成了6

## 3.4 切片复制

Go语言的内置函数 `copy()` 可以将一个数组切片复制到另一个数组切片中，如果加入的两个数组切片不一样大，就会按照其中**较小**的那个数组切片的元素个数进行复制

`copy()` 函数的使用格式如下：

```go
copy( destSlice, srcSlice []T) int
```

* `srcSlice `为数据来源切片
* `destSlice `为复制的目标（也就是将 `srcSlice` 复制到 `destSlice`），**目标切片必须分配过空间且足够承载复制的元素个数**，并且来源和目标的**类型必须一致**，`copy()` 函数的**返回值**表示实际**发生复制的元素个数**

下面的代码展示了使用 `copy()` 函数将一个切片复制到另一个切片的过程：

```go
slice1 := []int{1, 2, 3, 4, 5}
slice2 := []int{5, 4, 3}
copy(slice2, slice1) // 只会复制slice1的前3个元素到slice2中
copy(slice1, slice2) // 只会复制slice2的3个元素到slice1的前3个位置
```

*切片的**引用**和**复制**操作对切片元素的影响:

```go
package main

import "fmt"

func main() {
    // 设置元素数量为1000
    const elementCount = 1000
    // 预分配足够多的元素切片
    srcData := make([]int, elementCount)
    // 将切片赋值
    for i := 0; i < elementCount; i++ {
        srcData[i] = i
    }
    // 引用切片数据 切片不会因为等号操作进行元素的复制
    refData := srcData
    // 预分配足够多的元素切片
    copyData := make([]int, elementCount)
    // 将数据复制到新的切片空间中
    copy(copyData, srcData)
    // 修改原始数据的第一个元素
    srcData[0] = 999
    // 打印引用切片的第一个元素 引用数据的第一个元素将会发生变化
    fmt.Println(refData[0])
    // 打印复制切片的第一个和最后一个元素 由于数据是复制的，因此不会发生变化。
    fmt.Println(copyData[0], copyData[elementCount-1])
    // 复制原始数据从4到6(不包含)
    copy(copyData, srcData[4:6])
    for i := 0; i < 5; i++ {
        fmt.Printf("%d ", copyData[i])
    }
}
```

OUTPUT：

```go
999          //refData[]的值为999 1 2 3 4...(和srcData相同)
0 999        //copyData[]的值仍为0 1 2 3 4...
4 5 2 3 4    //srcData[4:6]的值为4 5，复制到copyData[0] copyData[0]
```
**结论：**

**引用**操作 `refData := srcData`，修改源切片 `srcData` 的数据，引用切片 `refData` 的元素也**会**发生变化

**复制**操作 `copy(copyData, srcData)`，修改源切片 `srcData` 的数据，引用切片 `refData` 的元素**不会**发生变化

# 4. new和make

make 关键字的主要作用是创建 slice、map 和 Channel 等内置的数据结构

而 new 的主要作用是为类型申请一片内存空间，并返回指向这片内存的指针

* make 分配空间后，会进行初始化；new分配的空间被清零
* new 分配返回的是指针，即类型 *Type；make 返回的是引用，即 Type
* new 可以分配任意类型的数据