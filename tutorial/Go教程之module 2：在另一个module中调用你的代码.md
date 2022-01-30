# Go教程之module 2：在另一个module中调用你的代码

- 在前一章中，我们创建了greeting module，在这一章中我们将会写代码来调用之前写的**Hello**函数。

## 1.创建项目

- 创建hello文件夹，与greetings文件夹平级。

```
<home>/
 |-- greetings/
 |-- hello/
```

```sh
cd ..
mkdir hello
cd hello
```

## 2.开启依赖追踪

```sh
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
```

## 3.创建hello.go文件

```go
package main

import (
    "fmt"

    "example.com/greetings"
)

func main() {
    // Get a greeting message and print it.
    message := greetings.Hello("Gladys")
    fmt.Println(message)
}
```

- 声明main包。
- 导入两个包：`example.com/greetings` 和 [`fmt` 包](https://pkg.go.dev/fmt/)，这样就可以调用这两个包中的函数。
- 通过**greetings**包中的Hello函数获取打招呼信息。

## 4.编辑example.com/hello module使example.com/greetings module可用

- 在生产中，应当publish **example.com/greetings**，这样可以通过Go工具来下载。
- 但是我们并没有publish，所以我们需要调整**example.com/hello**，让它能够在本地找到**example.com/greetings**的代码。
- 为了达到上述目的，我们使用[`go mod edit`](https://go.dev/ref/mod#go-mod-edit)命令来编辑**example.com/hello** module，使Go工具能够它的module路径重定向到本地文件夹。

### 4.1 go mod edit

```sh
$ go mod edit -replace example.com/greetings=../greetings
```

- 这条命令指明了**example.com/greetings**应该用**../greetings**来替代，从而能够定位到本地依赖。
- 运行这条命令之后，go.mod文件变成：

```
module example.com/hello

go 1.16

replace example.com/greetings => ../greetings
```

## 4.2 go mod tidy

- go mod tidy这条命令可以同步**example.com/hello**的依赖，添加那些在module中使用的但是没有在go.mod文件中追踪到的依赖。

```sh
$ go mod tidy
go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
```

- 执行完上述命令，go.mod文件会变成：

```
module example.com/hello

go 1.16

replace example.com/greetings => ../greetings

require example.com/greetings v0.0.0-00010101000000-000000000000
```

- 这条命令在greetings文件夹中找到本地代码，go.mod中增加一行**require**指令用于指明**example.com/hello**需要 **example.com/greetings**。
- 跟在module path后面的是伪版本号，是替代真正版本号（在我们的module中并没有）的一串数字。

- 如果要引用已经publish的module，应该在module path后面打上tag，详见[Module version numbering](https://go.dev/doc/modules/version-numbers)。

```
require example.com/greetings v1.1.0
```

### 4.3 运行代码

- 终端切换到hello文件夹下。

```sh
$ go run .
Hi, Gladys. Welcome!
```

