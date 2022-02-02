# Go教程之module 6：增加测试

- 为**Hello**函数加个测试。

- Go中集成了单元测试，使用**testing**包和 **go test** 命令可以轻松进行测试。

## 1.创建greetings_test.go文件

- 在greetings文件夹中创建greetings_test.go文件，以 **_test.go** 结尾的文件表明其中包含了测试函数。

## 2.写点代码

- 在greetings_test.go文件中，新增代码如下：

```go
package greetings

import (
    "testing"
    "regexp"
)

// TestHelloName calls greetings.Hello with a name, checking
// for a valid return value.
func TestHelloName(t *testing.T) {
    name := "Gladys"
    want := regexp.MustCompile(`\b`+name+`\b`)
    msg, err := Hello("Gladys")
    if !want.MatchString(msg) || err != nil {
        t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
    }
}

// TestHelloEmpty calls greetings.Hello with an empty string,
// checking for an error.
func TestHelloEmpty(t *testing.T) {
    msg, err := Hello("")
    if msg != "" || err == nil {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```

- 在greetings包下实现了测试函数。
- 创建了两个测试函数来测试**greetings.Hello**函数。测试函数的要遵循**Test** + *Name*的命令规范，其中*Name*代码具体测试内容。并且，测试函数要传入**testing**包下的 [`testing.T` ](https://pkg.go.dev/testing/#T)的指针作为参数。我们可以使用这个参数的方法来报告和记录我们的测试。
- 两个测试函数
  - **TestHelloName**调用了**Hello**函数，传入**name**的值，理想情况下应该返回有效的信息。如果此次调用返回了error或者错误信息（即信息中不包括传入的人名），我们应该使用参数**t**的 [`Fatalf` ](https://pkg.go.dev/testing/#T.Fatalf)方法来在控制台打印信息并且终止程序。
  - **TestHelloEmpty**调用了**Hello**函数，传入空字符串。这个测试用例用于测试错误处理是否有效。如果此次调用返回了非空字符换或者没有error，我们应该使用参数**t**的 [`Fatalf` ](https://pkg.go.dev/testing/#T.Fatalf)方法来在控制台打印信息并且终止程序。

## 3.看看效果吧

- 在greetings文件夹下，运行 **go test** 命令来执行测试。
- **go test** 命令先找到测试文件（那些以 **_test.go** 结尾的文件），在测试文件中找到测试函数（那些以 **Test** 开头的函数）。可以添加 -v 标志以获得列出所有测试及其结果的详细输出。
- 正常情况下，测试通过。

```sh
$ go test
PASS
ok      example.com/greetings   0.364s

$ go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok      example.com/greetings   0.372s
```

## 4.破坏greetings.Hello函数来观察失败测试

- 为了使测试失败，在**greetings.Hello**函数中做如下修改，去掉**name**参数：

```go 
// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    // message := fmt.Sprintf(randomFormat(), name)
    message := fmt.Sprint(randomFormat())
    return message, nil
}
```

## 5.看看效果吧

- 在greetings文件夹下，运行 **go test**命令执行测试。
- 这次在 **go test**中我们不加 **-v** 标志。这样输出就只包含失败测试的结果，这在测试函数非常多的时候是很有用的。
- **TestHelloName**应该失败，**TestHelloEmpty**还是通过。

```sh
$ go test
--- FAIL: TestHelloName (0.00s)
    greetings_test.go:15: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
FAIL
exit status 1
FAIL    example.com/greetings   0.182s
```

