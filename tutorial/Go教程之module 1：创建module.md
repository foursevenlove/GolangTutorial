# Go教程之module 1：创建module

- 在这篇教程中，我们将会创建两个module，第一个是用于被其他库或应用导入的module，第二个是调用第一个module的module。

- Go的代码被分成许多个包，包又被分成许多个module。module指明了运行代码所需的依赖，包括Go的版本和一系列其他需要的module。

## 1.创建项目

```shell
cd %HOMEPATH%
mkdir greetings
cd greetings
```

## 2.初始化项目

```sh
$ go mod init example.com/greetings
go: creating new go.mod: module example.com/greetings
```

- **go mod init**命令创建go.mod文件用于追踪依赖。初始化后，其中包含且仅包含module的名称以及Go的版本。但是随着添加依赖，go.mod文件会列出依赖。这使构建保持可重复性，并且可以控制module的版本。

## 3.创建greetings.go

```go
package greetings

import "fmt"

// Hello returns a greeting for the named person.
func Hello(name string) string {
    // Return a greeting that embeds the name in a message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```

- 我们创建Hello函数会在之后进行调用。

- 声明了greetings包，用于管理相关函数。

- 实现了Hello函数并且返回打招呼信息。

  - 这个函数需要**string** 类型的 **name** 参数。
  - 返回一个**string**。
  - 在Go中，一个函数的首字母如果是大写的，那么这个函数就可以在别的包中调用。详见 [Exported names](https://go.dev/tour/basics/3) 。

  ![](https://raw.githubusercontent.com/foursevenlove/gitResource/master/Typora20220130161240.png)

- 声明了**message**变量用于保存打招呼信息，**:=** 操作符用于在一行声明并且初始化一个变量。
- 使用了**fmt**包中的**Springf**函数来打创建打招呼信息，第一个参数是一个格式化字符串，**Springf**用**name**变量的值来代替%v占位符。
- 返回打招呼信息。

