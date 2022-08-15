# Go语言基础必备知识点(二) 字符串篇  
## 1. 字符串的应用  
一般的比较运算符 (==、!=、<、<=、>=、>) 是通过在内存中按**字节比较**来实现字符串比较的，因此比较的结果是字符串**自然编码的顺序**

1. 计算字符串的长度：  
  ASCII字符(字节长度)使用`len()`函数  
  Unicode字符串(中文字符个数(*最常用 )*)使用`utf8.RuneCountInString()`函数  
    ```go
    func main(){
        str1 := "chris"
        str2 := "克里斯"
        fmt.Println(len(str1))  // 5 (1个字母占1个字节)
        fmt.Println(len(str2))  // 9 (1个中文占3个字节，go从底层支持utf8)
        fmt.Println(utf8.RuneCountInString(str2)) // 3 (中文长度)
    }
    ```  
2. 字符串索引  
  字符串的内容 (纯字节) 可以通过标准索引法来获取，在方括号`[]`内写入索引，索引从 0 开始计数  
    ```go
    func main(){
        str := "chris"
        //注意索引是99、115而不是'c'、's'
        fmt.Println(str[0])  // 99 ('c') (str 的第一个字节)
        fmt.Println(str[len(str)-1])  // 115('s') (str 的最后一个字节)
    }
    ```  
3. 字符串拼接  
* 字符串拼接符 `"+"` 和 `"+="`  
  两个字符串 s1 和 s2 可以通过 `s := s1 + s2` 或 `s1 += s2` 拼接在一起
    ```go
    func main(){
        //若换行，拼接字符串用的加号“+”必须放在第一行末尾(行尾自动补全分号)
        str := "chris " +
            "liu"
        fmt.Println(str)    //chris liu
    }
    ```  
* **\*使用空的字节缓冲区 `WriteString()`**  
  对于简单而少量的拼接，使用运算符 `+` 和 `+=` 的效果虽然很好，但随着拼接操作次数的增加，这种做法的效率并不高  
  
  如果需要在**循环**中拼接字符串，则使用**空的字节缓冲区** `WriteString()`来拼接的**效率更高** 

  **bytes.Buffer代码：** **(常用buffer.WriteString())**  
    ```go
    package main

    import (
        "bytes"
        "fmt"
    )

    func main() {
        str1 := "chris "
        str2 := "liu"
        var stringBuilder bytes.Buffer
        //*还有一种常用写法(命名为buffer) 
        //var buffer bytes.Buffer
        
        stringBuilder.WriteString(str1)
        stringBuilder.WriteString(str2)
        fmt.Println(stringBuilder.String())  //chris liu
        //节省内存分配，提高处理效率
    }
    ```

* 从测试结果来看，性能从高到低排序如下：  
**bytes.Buffer** > strings.Builder > string + string > fmt.Sprintf   

  *strings.Builder 是非线程安全的*，它和 bytes.Buffer 的性能很接近，在一个水平上。另外有人针对**大量字符串拼接**的情况也做了测试，发现 strings.Builder 以微弱的优势胜出了，不过日常我们很少用到大量的字符串拼接的情况

4. **字符串的格式化**
   * `Printf / Println`：结果写到标准输出
   * `Sprintf`：结果会以字符串形式返回，用传入的格式化规则符将传入的变量格式化 (终端中不会有显示)
    ```go
    package main

    import "fmt"

    func main() {
        str1 := []byte("chris liu")
        str2 := "chris liu"
        fmt.Println(str1)                //[116 101 115 116 32 115 116 114]
        fmt.Println(str2)                //chris liu
        fmt.Println(str1, "southampton") //[116 101 115 116 32 115 116 114] southampton
        fmt.Printf("%s\n", str1)         //chris liu
        fmt.Sprintf("%s", str1)          //空，无IO输出
        // Sprint 以字符串形式返回
        printStr := fmt.Sprintf("%s", str1)
        fmt.Println(printStr) //chris liu
    }
    ```  
    **注意：str1 和 str2 的区别！**  
    常用格式化：
    ```go
    %c  单一字符
    %T  动态类型
    %v  本来值的输出
    %+v 字段名+值打印
    %d  十进制打印数字
    %p  指针，十六进制
    %f  浮点数
    %b  二进制
    %s  string
    ```
5. 字符串查找 (返回所在位置，未查找到返回-1)  
* strings.Index()： 正向搜索子字符串  
* strings.LastIndex()：反向搜索子字符串  
    ```go
    package main

    import (
        "fmt"
        "strings"
    )

    func main() {
        // 查找
        str := "chris,liu 12138哈哈哈"

        // 正向搜索字符串
        index := strings.Index(str, ",")
        fmt.Println(",所在的位置:", index) //5 (从0开始)
        fmt.Println(str[index+1:])    // liu 12138哈哈哈(输出所在位置后所有字符)

        add := strings.Index(str, "+")
        fmt.Println("+所在的位置:", add) //+所在的位置:-1 (查找失败返回-1)
    }
    ```  
## 2. 类型转换  
由于Go语言**不存在隐式**类型转换，因此所有的类型转换都必须**显式**的声明：  
```go
//类型 B 的值 = 类型 B(类型 A 的值)
valueOfTypeB = type B(valueOfTypeA)
```

示例：  
```go
a := 1.0
b := int(a)
```

类型转换只能在定义正确的情况下转换成功，例如从一个取值范围较小的类型转换到一个取值范围较大的类型 (将 int16 转换为 int32)  

当从一个取值范围较大的类型转换到取值范围较小的类型时 (将 int32 转换为 int16 或将 float32 转换为 int) ，会发生**精度丢失**  

只有相同底层类型的变量之间可以进行相互转换 (如将 int16 类型转换成 int32 类型) ，不同底层类型的变量相互转换时会引发编译错误 (如将 bool 类型转换为 int 类型)   

浮点数在转换为整型时，会将小数部分去掉，只保留整数部分  

## 3. 修改字符串

Go语言的字符串是**不可变的**，修改字符串时，可以将字符串**转换为 []byte** 进行修改  

**[]byte 和 string 可以通过强制类型转换**  

```go
package main

import "fmt"

func main() {

    //将8080改为8081
    s1 := "localhost:8080"
    fmt.Println(s1)
    // 强制类型转换 string to byte
    strByte := []byte(s1)

    // 下标修改
    strByte[len(s1)-1] = '1'
    fmt.Println(strByte)

    // 强制类型转换 []byte to string
    s2 := string(strByte)
    fmt.Println(s2)
}
```
## 4. 字符串与其他数据类型的转换 (strconv包)
简单的强制类型转换方式不能对 int / float 和 string 进行互转，要跨大类型转换，可以使用 strconv 包提供的函数

* **strconv**
strconv 包提供了简单数据类型之间的类型转换功能，可以将简单类型转换为字符串，也可以将字符串转换为其它简单类型  


* strconv 包提供了以下几类函数：

    >string 转 int：`Atoi()`
    >int 转 string: `Itoa()`

    >ParseTP 类函数将 string 转换为 TP 类型：`ParseBool()、ParseFloat()、ParseInt()、ParseUint()` (因为string转其它类型可能会失败，所以这些函数都有第二个返回值表示是否转换成功)
    >其它类型转 string 类型：`FormatBool()、FormatFloat()、FormatInt()、FormatUint()`
    >AppendTP 类函数用于将 TP 转换成字符串后 append 到一个 slice 中：`AppendBool()、AppendFloat()、AppendInt()、AppendUint()`
    
    **最常见**的是 string 和 int 之间的转换：

    * int 转换为 string：`Itoa()`

    * string转换为int：`Atoi()`
        ```go
        func Atoi(s string) (int, error)
        ```
        由于string可能无法转换为 int，所以这个函数有两个返回值：第一个返回值是转换成 int 的值，第二个返回值**判断是否转换成功**  

  示例：  
    ```go
    package main

    import (
        "fmt"
        "strconv"
    )

    func main() {

        // int 转 str
        intValue2 := 1
        strValue2 := strconv.Itoa(intValue2)
        fmt.Printf("%T, %s\n", strValue2, strValue2)  // string, 1
        
        // string 转 int
        strValue1 := "1"
        intValue1, _ := strconv.Atoi(strValue1)      //只接收第一个返回值转化成int的值
        //intValue1, a := strconv.Atoi(strValue1)
        //fmt.Printf("a: %v\n", a)       //a: <nil>
        fmt.Printf("%T,%d\n", intValue1, intValue1)  // int,1
    }
    ```