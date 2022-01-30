# Go教程之用GO开始

## 1.安装Go

- https://go.dev/doc/install

## 2.写点代码

### 2.1 创建项目根目录

```shell
cd %HOMEPATH%
mkdir hello
cd hello
```

### 2.2 开启依赖追踪

- go.mod用于定义代码的module。
- 当代码导入了别的module的包时，通过module又可以来管理依赖。
- 通过 [`go mod init`](https://go.dev/ref/mod#go-mod-init) 命令来创建go.mod文件，从而开启依赖追踪。go mod init定义代码所在的module的名字，这个名字是module的module path。
- 在实际开发中，module path通常是代码的仓库地址，如github.com/mymodule。如果想让他人使用你公开的module，那么module path*必须*是一个可以通过Go工具来下载你的moudle的地址。详见[Managing dependencies](https://go.dev/doc/modules/managing-dependencies#naming_module)。
- 出于教学目的，就采用**example/hello**。

```shell
$ go mod init example/hello
go: creating new go.mod: module example/hello
```

### 2.3 创建hello.go文件

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

- 声明了一个main包（包是用于分类函数的一种方式，并且一个包由同一目录下的所有文件组成，也就是说在一个文件下的.go文件属于同一个包）。
- 导入了常用包[`fmt`](https://pkg.go.dev/fmt/)，其中包含了处理文本的函数，如打印到控制台。这个包是在安装go是自带的[standard library](https://pkg.go.dev/std)。
- 实现了main函数用于向控制台打印信息。当运行main包时，默认运行**main**函数。

### 2.4 运行代码

```shell
$ go run .
Hello, World!
```

-  [`go run`](https://go.dev/cmd/go/#hdr-Compile_and_run_Go_program)命令用于运行代码。

## 3.调用外部包的代码

- 可以通过别人写好的包里的函数来实现你想要的功能。

### 3.1 使用外部module的函数

- 访问 pkg.go.dev，搜索["quote"](https://pkg.go.dev/search?q=quote)包；在搜索结果中找到rsc.io/quote；在**Documentation** 部分，找到**Index**，这里列出了可以在自己的代码中调用的函数，我们将使用**Go**函数**quote**包是包含于**rsc.io/quote**这个module下的。
- 可以在pkg.go.dev中找到你想要的module，可以在你的代码中调用这些module的包里的函数。

### 3.2 引入 rsc.io/quote 包并且调用Go函数

```go
package main

import "fmt"

import "rsc.io/quote"

func main() {
    fmt.Println(quote.Go())
}
```

### 3.3 添加新的module requirements和sums

```shell
$ go mod tidy
go: finding module for package rsc.io/quote
go: found rsc.io/quote in rsc.io/quote v1.5.2
```

- 运行 `go mod tidy`时，将定位并且下载rsc.io/quote这个module，它包含了你导入的包，默认下载最新版本。
- go将**quote** module添加为requirement。
- go.sum文件用于验证module，参考 [Authenticating modules](https://go.dev/ref/mod#authenticating) 。

### 3.4 运行代码

```shell
$ go run .
Don't communicate by sharing memory, share memory by communicating.
```





