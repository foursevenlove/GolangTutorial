# Go教程之module 3：返回并处理错误

## 1.写点代码

- 在 greetings/greetings.go中，修改代码如下：

```go
package greetings

import (
    "errors"
    "fmt"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return "", errors.New("empty name")
    }

    // If a name was received, return a value that embeds the name
    // in a greeting message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message, nil
}
```

- 修改了Hello函数使其返回两个值，一个string类型和一个error类型。调用函数时会检查第二个返回值来判断是否发生错误。（在Go中，一个函数可以有多个返回值，详见 [Effective Go](https://go.dev/doc/effective_go.html#multiple-returns)）
- 导入了Go的标准库**errors**包，这样可以使用里面的 [`errors.New`](https://pkg.go.dev/errors/#example-New)函数。
- 增加 **if** 判断来判断**name**参数是否为空，如果是空那么就返回error。使用**errors.New**函数来返回带错误信息的一个error。
- 如果**name**参数不为空，返回**nil**（意味着没有错误）作为第二个参数值。

## 2.再写点代码

- 在hello/hello.go文件中，处理Hello函数返回的错误或者正常返回值。

```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // Request a greeting message.
    message, err := greetings.Hello("")
    // If an error was returned, print it to the console and
    // exit the program.
    if err != nil {
        log.Fatal(err)
    }

    // If no error was returned, print the returned message
    // to the console.
    fmt.Println(message)
}
```

- 使用 [`log`](https://pkg.go.dev/log/) 包来打印日志，日志以 ("greetings: ")作为开头，没有时间戳以及源文件信息。
- 用**message**、**err**变量来接收**Hello**函数的返回值。
- 把Hello函数的参数设为空字符串""，这样可以看到错误信息。

- 再尝试把**name**参数设置为别的非空值，这样就可以正常运行。

- 使用 [`log`](https://pkg.go.dev/log/) 包来输出错误信息。如果Hello函数返回错误，可以使用**log**包中的 [`Fatal` 函数](https://pkg.go.dev/log?tab=doc#Fatal) 来打印出错误并且停止程序。

## 3.试试效果吧

- 切换到hello文件夹，运行go run .
- 现在由于传入的**name**参数为空，所以会得到错误。

```sh
$ go run .
greetings: empty name
exit status 1
```

