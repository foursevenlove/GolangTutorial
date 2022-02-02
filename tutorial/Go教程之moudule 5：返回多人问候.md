# Go教程之moudule 5：返回多人问候

- 传入一组人物姓名，返回一组问候信息。
- 我们可以通过修改原来的Hello函数来达到效果，但是问题在于如果我们已经publish了 `example.com/greetings`这个module，那么可能有人已经调用了原来的Hello函数，一旦我们修改了Hello函数，他们的程序就会报错。所以我们不应该直接修改Hello函数，而是应该写一个新的函数。

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

// Hellos returns a map that associates each of the named people
// with a greeting message.
func Hellos(names []string) (map[string]string, error) {
    // A map to associate names with messages.
    messages := make(map[string]string)
    // Loop through the received slice of names, calling
    // the Hello function to get a message for each name.
    for _, name := range names {
        message, err := Hello(name)
        if err != nil {
            return nil, err
        }
        // In the map, associate the retrieved message with
        // the name.
        messages[name] = message
    }
    return messages, nil
}

// Init sets initial values for variables used in the function.
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

    // Return one of the message formats selected at random.
    return formats[rand.Intn(len(formats))]
}
```

- 增加了**Hellos**函数，参数是name的**slice**，返回值是一个**map**。
- **Hellos**函数内部调用了**Hello**函数，这减少了重复代码。
- 创建一个存储问候信息的map，把人物的名字作为Key，问候信息作为Value。在Go语言中，用这样的语法来初始化一个map ：make(map[*key-type*]*value-type*)。**Hellos**函数返回这个map。关于map，详见 [Go maps in action](https://blog.golang.org/maps) 。
- 检查传入的slice中的每一个name是否为空值，如果不是空，那么给这个name关联一个问候信息。在**for**循环中，**range**返回两个值：一个是当前内容的index下标，另一个是当前内容的value值。因为我们用不到index下标值，所以用 `_`下划线来忽略，详见 [The blank identifier](https://go.dev/doc/effective_go.html#blank)。

## 2.再写点代码

- 在hello/hello.go中调用刚刚写的函数Hellos，传入name的slice，打印返回得到的姓名和问候信息。

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

    // A slice of names.
    names := []string{"Gladys", "Samantha", "Darrin"}

    // Request greeting messages for the names.
    messages, err := greetings.Hellos(names)
    if err != nil {
        log.Fatal(err)
    }
    // If no error was returned, print the returned map of
    // messages to the console.
    fmt.Println(messages)
}
```

- 用**names**变量来保存三个姓名。
- 把**names**传入**Hellos**函数。

## 3.看看效果吧

- 在hello文件夹下，运行go run

```sh
$ go run .
map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hail, Samantha! Well met!]
```

