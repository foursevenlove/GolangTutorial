# Go教程之module 4：返回随机打招呼信息

- 这一章中会使用Go中的slice。一个slice相当于一个数组，但是和数组有一点不同，slice的尺寸可以动态的改变。
- 我们使用一个slice来保存三个打招呼信息，每次随机返回其中一个。
- 关于slice，详见 [Go slices](https://blog.golang.org/slices-intro) 。

## 1.写点代码

- 在greetings/greetings.go中，修改代码如下：

```go
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
    // A slice of message formats.
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // Return a randomly selected message format by specifying
    // a random index for the slice of formats.
    return formats[rand.Intn(len(formats))]
}
```

- 添加了**randomFormat**函数，用于返回随机选择的打招呼信息格式。注意，该函数首字母为小写，这意味着该函数只能在同一个包中调用。
- 在**randomFormat**函数中，声明了一个**formats** slice保存了三条打招呼信息。当声明一个slice时，在方括号中省略slice的大小，如：**[]string**。这表明slice的大小是动态变化的。
- 使用 [`math/rand` ](https://pkg.go.dev/math/rand/)包来生成随机数用于从slice中选择内容。
- 添加了 **init**函数，在其中调用 `rand`包下的**Seed**函数，传入当前时间作为参数。这样每次运行程序时传入的时间都是不同的，**rand.Intn**得到的值也都是不同的。如果**rand.Seed**每次传入相同值，那么**rand.Intn**得到的值是相同的。关于**init**函数，详见 [Effective Go](https://go.dev/doc/effective_go.html#init)。

- 在**Hello**函数中，调用**randomFormat**函数来得到随机信息，并且传入**name**参数，生成打招呼信息。
- 返回信息（或者error）。

## 2.再写点代码

- 在hello/hello.go文件中，修改代码如下：

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
    message, err := greetings.Hello("Gladys")
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

- 修改传入的**name**参数，“Gladys”。

## 3.看看效果吧

```sh
$ go run .
Great to see you, Gladys!

$ go run .
Hi, Gladys. Welcome!

$ go run .
Hail, Gladys! Well met!
```

