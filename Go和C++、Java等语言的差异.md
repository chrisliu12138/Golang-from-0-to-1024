# Go和C++、Java等语言的差异
## 1. 编码差异
   1. 开大括号`{`不能放在单独的一行
      ```go
      func main()  
      { //error, can't have the opening brace on a separate line
         fmt.Println("")
      }
      ```
      报错：`syntax error: unexpected semicolon or newline before {`  
      建议开发规则：变量命名遵循小驼峰，一般不用下划线`_`，如myName。全局变量首字母大写，常量全大写加`_`  
      **终于不用纠结代码应该是 Google 风格还是 Microsoft 风格！**
   2. 自动分号`;`的注入 (没有预读)   
   go中也是有分号的，采用语法解析器自动在每行末增加分号，因此写代码的时候可以把分号缩略
   3. 未使用的变量  
    如果有未使用的变量，代码将编译失败，在函数内一定要使用声明的变量，但未使用的全局变量是没问题的  
   **如果给未使用的变量分配了一个新的值，代码还是会编译失败，需要在某个地方使用这个变量。(赋值不等于使用，但是将该变量赋值给其他变量是可以的)**
      ```go
      func main() {  
         var one int
         _ = one

         var three int 
         three = 3
         one = three

         var four int
         four = four
      }
      ```
   4. 简式变量声明(短变量) `:=`   
   仅可以在函数内部使用
   5. 未使用的import  
   如果引入一个包，而没有使用其中的任何函数、接口、结构体或者变量的话，代码将会编译失败(类似未使用的变量)  
   **如果有未来可能需要使用的包，可以添加一个下划线标记符_，来作为这个包的名字，从而避免编译失败。下滑线标记符用于引入，但不使用**
      ```go
      package main

      import (  
         _ "fmt"
         "log"
         "time"
      )

      var _ = log.Println

      func main() {  
         _ = time.Now
      }
      ```
   6. 使用简式声明重复声明变量  
      不能在一个单独的声明中重复声明一个变量
      ```go
      func main() {  
         one := 0
         one := 1  //error，重复声明变量
      }
      ```
      但在多变量声明中这是允许的，其中至少要有一个新的声明变量。重复变量需要在相同的代码块内，否则你将得到一个隐藏变量
      ```go
      func main() {  
         one := 0
         one, two := 1, 2     //此时重复声明one是允许的
         one, two = two, one  //防止变量未使用
      }
      ```
   7. *偶然的变量隐藏Accidental Variable Shadowing  
      短式变量声明的语法如此的方便（尤其对于那些使用过动态语言的开发者而言），很容易让人把它当成一个正常的分配操作  
      如果你在一个新的代码块中犯了这个错误，将不会出现编译错误，但你的应用将不会做你所期望的事情
      ```go
      func main() {  
         x := 1
         fmt.Println(x)     //prints 1
         {
            fmt.Println(x) //prints 1
            x := 2
            fmt.Println(x) //prints 2
         }
         fmt.Println(x)     //prints 1 (wrong if you need 2)
      }
      ```  
      *即使对于经验丰富的go开发者而言，这也是一个非常常见的陷阱。这个坑很容易挖，但又很难发现    
      可以使用 vet 命令来发现一些这样的问题，默认情况下，vet不会执行这样的检查，需要设置-shadow参数：`go tool vet -shadow your_file.go`*
   8. 自增和自减不支持前置操作 ~~`++i`~~
   9. if else 语句中，else必须在}后：`}else`，不可以单独起一行，否则会编译失败；若只有一行执行语句也必须用{}括起来
   10. if else、for、for range后边都不需要加括号
   11. switch 声明中的失效行为  
      在 switch 声明语句中的 `case` 语句块在默认情况下会 `break` 。这和其他语言中的进入下一个 “next” 代码块的默认行为不同   
      可以通过在每个 `case` 块的结尾使用 **`fallthrough`**，来强制 `case` 代码块进入。也可以重写switch语句，来使用 `case` 块中的表达式列表
   12. map的容量  
      可以在 map 创建时指定它的容量，但无法在 map 上使用 `cap()` 函数  
         ```go
         func main() {  
            m := make(map[string]int,99)
            cap(m)    //error
         }
         ```
         报错：`invalid argument m (type map[string]int) for cap`
## 2. 概念差异
   1. Go没有支持构造和析构的 `class` 类型，也没有继承和虚函数的概念。但是 go 提供接口 `interfaces` 支持，我们可以把接口看作是 C++ 中模板类似的技术
   2. **Go提供垃圾内存回收支持**  
   我们没有必要显式释放内存，go的运行时系统会帮我们收集垃圾内存
   (*map中没有清空所有元素的函数，清空map的唯一方法就是重新make一个新的map，不用担心垃圾回收的效率，go语言中的并行垃圾回收效率比写一个清空函数要高效的多*)
   3. Go中有指针，但是没有指针算术  
   因此，不可能通过指针以字节方式来遍历一个字符串。当用数组作为参数调用函数时，将会复制整个数组。当然，Go语言中一般用切片 `slices` 代替数组作为参数，切片是建立在底层数组地址之上的，因此传递的是数组的地址
   4. **Go不使用头文件，每个源文件都被定义在特定的包 (package) 中**  
   在包中以大写字母定义的对象 (例如类型，常量，变量，函数等) 对外是可见的，可以被别的代码导入使用
   5. Go不会作隐式类型转换。如果在不同类型之间赋值，必须强制转换类型  
   ~~`b = (int64)a`~~  `b = int64(a)`
   6. Go不支持函数重载，也不支持运算符定义
   7. Go不支持 `const` 和 `volatile` 修饰符
   8. Go使用 `nil` 表示无效的指针，C++中使用 `null` 或 `0` 表示空指针。字符串不会为 `nil`，字符串的零值为空字符串 `""`。
   9. `nil` 和其他语言的 `null` 是不同的
      * **nil标识符是不能比较的**
      * **nil不是关键字或保留字**
      * nil没有默认类型(%T)  //error: use of untyped nil
      * 不同类型nil的指针是一样的
      * **nil是map、slice、pointer、channel、func、interface的零值**
      * 不同类型的nil值占用的内存大小可能是不一样的，具体大小取决于编译器和架构