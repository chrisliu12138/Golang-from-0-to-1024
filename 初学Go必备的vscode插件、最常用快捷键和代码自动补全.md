# 初学Go必备的vscode插件、最常用快捷键和代码自动补全
# vscode写go的必会操作和常见编译错误
   *写在前面：go 1.18和1.19已经不需要手动配置环境变量了，安装后即自动配置。vscode安装教程和go语言环境配置网上相关教程文档非常多，非常简单！*
## 1. vscode必备插件
   1. Go、Code Runner
   ![](Go.png)  
   ![](Code%20Runner.png)  
   不解释，运行必备
   2. Markdown All in One、Markdown Preview Enhanced、Paste Image
   ![](Markdown%20All%20in%20One.png)  
   为进行Markdown文档编写提供很多快捷键和自动补全功能，使vscode可以完全代替Typora(学生党拒绝付费！且不用切换app，写代码和文档都可以在vscode)
   ![](2022-08-08-15-53-07.png)  
   边写边看到Markdown渲染之后的样子，在 Preview 界面按住鼠标右键可以打开功能栏，选择Open in Browser可以将文件在浏览器打开，还可以选择生成HTML或者PDF等
   ![](2022-08-08-15-57-56.png)  
   在Markdown中快捷插入图片，复制图片后在文档中 `ctrl + alt + v` 粘贴后图片自动添加到文件夹目录下;  
   也可以将图片手动添加到文件夹，将图片拖到需要插入图片的位置同时按 `shift`;  
   也可以直接输入 `![]()` 后括号内会自动出现所含图片名称，上下选择即可
   3. vscode-icons
   ![](2022-08-10-19-18-22.png)  
   不同的文件展示不同的图标，方便快速识别文件类型，非常好用！效果如下图：  
   ![](2022-08-10-19-24-04.png)
## 2. vscode最常用快捷键
   1. 行注释 `ctrl + /`
   2. **块注释** `shift + alt + a` (~~按习惯可修改为 `ctrl + shift + /`~~)
   3. 删除行 直接 `ctrl + x` 和剪切一样且不用选中整行(~~正常为 `ctrl + shift + k`~~)
   4. **向下向上复制行** `shift + alt + up/down`
   5. **多行批量缩进**  
    `ctrl + ] 或 tab`         //向右缩进
    `ctrl + [ 或 shift + tab` //向左缩进  
   6. 向上/向下移动行(也可理解为交换该行与上/下一行) `alt+ up/down`
   7. 查找、替换 `ctrl + f / ctrl + h`
   8. 移动到行首/尾 `home / end`
## 3. vscode快速生成golang代码片段
   1. `pkgm`：生成main包+main主函数
        ```go
        package main
        func main() {
        } 
        ```
   2. `ff`：格式化输出
       ```go
       fmt.Printf("", var)
       ```
   3. `fp`：Println换行输出
       ```go
       fmt.Println("")
       ```
   4. `a.Print!`(输入`a.p`第一个就是,直接回车即可)：格式化输出变量`a:`
      ```go
      a := 1
      fmt.Printf("a: %v\n", a)
      ```
   5. `for`：for循环
        ```go
        for i := 0; i < count; i++ {
        }
        ```
   6. `forr`：for range
        ```go
        for _, v := range v {
        }
        ```
   7. `tys`：快捷构建结构体
        ```go
        type name struct {
        }
        ```
## 4. 常见编译错误
#### 1. `expected 'package', found 'EOF'`
   1. 运行文件未保存, `ctrl + s` 即可
   2. 项目文件存在空文件，将空文件移除，保存即可(常见由于多个main报错后将整个文档注释)
   3. ~~忘记在文件的首行写package包会报`expected 'package', found 'import'`~~
#### 2. `main redeclared in this block`(不影响运行)
   同一个目录下面不能有多个 main，调整或者创建多个文件夹分别放入对应的文件下执行即可
#### 3. *`expected ';', found 'EOF'`(不影响运行)
   1. 这种错误是 gopls 自身的 bug，好久了，一直没有解决，所以直接重新加载 vscode，然后就正常了
   2. 打开go项目时，重新 `Install/Update gopls` 这样在整个项目过程中，就不会出现只要一新建go文件就报上面的错误了。但是重新打开还是会出现，这个问题一直是官方gopls的问题