# 简介

## Go 是编译型语言

Go 使用编译器来编译代码。编译器将源代码编译成二进制（或字节码）格式；在编译代码时，编译器检查错误、优化性能并输出可在不同平台上运行的二进制文件。要创建并运行 Go 程序，程序员必须执行如下步骤。

1. 使用文本编辑器创建 Go 程序。
2. 保存文件。
3. 编译程序。
4. 运行编译得到的可执行文件。

这不同于 Python、Ruby 和 JavaScript 等语言，它们不包含编译步骤。Go 自带了编译器，因此无须单独安装编译器。

### 编译和解释区别

- **编译：**是把源代码的每一条语句都编译成机器语言，并最终生成二进制文件，这样运行时计算机可以直接以机器语言来运行此程序，在运行时会有很好的性能。
- **解释：**是只有在执行到对应的语句时才会将源代码一行一行的解释成机器语言，给计算机来执行，所以使用解释器来执行的语言也被称为动态语言。

举个现实中的例子，比如你现在想读一本英文书，但你自己又不懂英文，然后你去找了个英文翻译小姐姐来帮忙，翻译小姐姐给你提供了两种选择：

1. 全本翻译：由翻译小姐姐帮你把整本书翻译完，完成校稿后给你一本翻译完成的中文书，在这个过程中翻译就会花费较长的时间，你阅读时就会很快、很轻松。

2. 随身翻译：就是翻译小姐姐随时守在你身边，你想阅读那一句，他就给你翻译那一句，这这种方式翻译时很快，但对你来说，阅读就会花费较长的时间。

# 基础语法

中文官网：https://go-zh.org

在线标准库文档：https://studygolang.com/pkgdoc

参考教程：https://www.qfgolang.com、http://c.biancheng.net/golang

## 变量声明

**1）变量指定后不赋值**

```go
// 默认为0
var name int
// 继续赋值
name = 1
```

**2）根据值自行判定变量类型**

```go
// 直接赋值
var name = "Hello"
```

**3）省略var，:=左边变量不能是已经声明过的变量**

```go
// 必须是首次声明
name := "Hello"
// 可以再次赋值
name = "World"

// 存在同名变量，再次用:=单独声明会报错（no new variables on left side of :=）
name := "Hello two"
// :=声明同名变量时，最少要有一个新的变量被定义，且在同一作用域
name, age := "Hello three", 1
```

> :=这种方式只能用在函数体，不能声明全局变量

全局声明示例：

```go
package main
var a = "Hello"
var b string = "World"
var c bool

func main(){
    println(a, b, c)
}
```

输出结果：

```console
Hello World false
```

**4）多变量声明**

```go
// 指定统一类型
var name1, name2 string
name1, name2 = "zs1", "zs2"

// 指定多种类型
var name1 string, name2 int = "zs", 12
```

> 注意：定义变量一定要使用，否则会报错

```go
// 报错（a declared but not used）
func main() {
   var a string = "abc"
   fmt.Println("hello, world")
}
```

## Dos命令

目录操作

```shell
# 创建目录（md：make directory）
md test01

md test01 test02
# 移除空目录（rd：remove directory）
rd test01
# 带询问移除非空目录（s：层级删除）
rd /s test01
# 不带询问移除非空目录（q：不再询问、s：层级删除）
rd /q/s test01

dir

cd /d e:\test01

cd ..

cd \
```

文件操作

```shell

echo Hello > hello.txt

copy hello.txt d:\test

copy hello.txt d:\test\world.txt

move hello.txt d:\test

move hello.txt d:\test\world.txt

del hello.txt

del *.txt

cls

exit
```

