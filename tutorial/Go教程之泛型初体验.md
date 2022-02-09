# Go教程之泛型初体验

- 可以使用[the Go playground in “Go dev branch” mode](https://go.dev/play/?v=gotip)来运行代码。

## 0.准备工作

- 安装Go 1.18 beta 1或者以上版本，详见[Installing and using the beta](https://go.dev/doc/tutorial/generics#installing_beta)。

- IDE
- CLI



## 1.创建项目

```sh
$ mkdir generics
$ cd generics
$ go mod init example/generics
go: creating new go.mod: module example/generics
```



## 2.添加非泛型函数

### 2.1 写点代码

- 创建main.go文件，添加以下代码：

```go
package main

// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
    var s int64
    for _, v := range m {
        s += v
    }
    return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
    var s float64
    for _, v := range m {
        s += v
    }
    return s
}
```

- 声明了两个函数用于求map中的和并且返回和。
- 在main.go最上方，包声明下方添加main函数用于初始化两个map一遍后续使用：

```go
func main() {
    // Initialize a map for the integer values
    ints := map[string]int64{
        "first": 34,
        "second": 12,
    }

    // Initialize a map for the float values
    floats := map[string]float64{
        "first": 35.98,
        "second": 26.99,
    }

    fmt.Printf("Non-Generic Sums: %v and %v\n",
        SumInts(ints),
        SumFloats(floats))
}
```

- 初始化了用于存储**float64**的map和存储**int64**的map。
- 调用声明的两个函数用于计算map中元素的和。
- 打印结果。
- 导包如下：

```go
package main

import "fmt"
```

### 2.2 看看效果

```sh
$ go run .
Non-Generic Sums: 46 and 62.97
```

---

> 有了泛型之后，就可以用一个函数来代替以上两个函数。

## 3.添加泛型函数来处理不同类型

- 在这一节中，我们将新增一个泛型函数用于接收包含整数或者浮点数的一个Map，这样就可以用一个函数来代替之前的两个函数。
- 为了支持不同类型的值，那么这个泛型函数就要声明它支持什么类型的值。换句话说，在调用这个泛型函数的时候，你要告诉这个函数接收的参数map到底是整数型的还是浮点数型的。
- 因此，泛型函数需要在普通的函数参数基础上增加*类型参数*。这些类型参数表明我们声明的是泛型函数，这样就可以处理不同类型的参数。可以通过普通函数参数和类型参数来调用函数。
- 每一个类型参数都有一个*类型约束*，代表着所有可以使用的类型。
- 尽管通常类型参数的约束代表着许多类型，但是在编译时类型参数只能代表一种在调用函数时指定的类型。如果参数类型不满足类型约束，那么代码不能编译。
- 记住，类型参数必须支持在泛型代码中所有在该种类型上的操作。比如，在我们的函数代码中要进行**string**的操作，但是在类型约束中又包含了数字型，那么代码不能编译。

### 3.1 写点代码

```go
// SumIntsOrFloats sums the values of map m. It supports both int64 and float64
// as types for map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

- 声明了**SumIntsOrFloats**函数，允许使用两种类型参数**K**和**V**，接收一个参数 `map[K]V`使用了类型参数，函数返回值是类型**V**的值。
- 指定类型参数**K**的类型约束为**comparable**，**comparable**约束是Go中预声明好的。它代表着那些可以通过 **==** 和 **!=** 比较的类型。在Go中，map的key必须是comparable。所以把**K**声明成**comparable**，这样就可以在map中用**K**作为key的类型。
- 指定类型参数**V**的类型约束为：**int64**和**float64**。使用 **|** 来分隔两种类型，代表这两种约束均可以使用。在调用代码时，两者之一的类型都可以通过编译。
- 指定参数**m**是**map[K]V**类型，其中**K**和**V**是我们声明好的类型参数。
- 在main函数中，增加代码如下：

```go
fmt.Printf("Generic Sums: %v and %v\n",
    SumIntsOrFloats[string, int64](ints),
    SumIntsOrFloats[string, float64](floats))
```

- 调用刚刚声明的泛型函数，传入创建好的两个map。
- 指定类型参数的类型，用[]括起来。
- 打印函数返回的结果。

### 3.2 看看效果

```sh
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```

---

>在调用泛型函数时，我们传入了类型参数的类型，这样可以让编译器知道需要什么类型。但是在下一节，我们可以省略这些类型参数的类型，因为编译器可以推断类型。

## 4.调用泛型函数时省略类型参数

- 在这一节中，我们在调用函数的代码部分进行微调。在这种情况下，可以省略类型参数。
- 当编译器可以推断你想使用的类型的时候就可以省略类型参数（废话吗不是）。编译器可以通过函数参数的类型来推断类型参数。
- 注意这不是通用情况。比如，当你需要调用无参的泛型函数时，在调用函数时就需要指定类型参数。

### 4.1 写点代码

- 在main函数中，增加代码如下：

```go
fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
    SumIntsOrFloats(ints),
    SumIntsOrFloats(floats))
```

### 4.2 看看效果

```sh
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
```



## 5.声明类型约束

- 在这一节中，我们将把之前定义的约束转化成接口，这样以便于复用。以这种方式声明约束可以简化代码，比如当约束很复杂的时候。

- 我们把类型约束声明为接口。只要是实现了这个接口的任意类型都可以通过约束。比如，如果我们声明了一个类型约束有三个方法，那么在使用带类型参数的泛型函数时，用于调用函数的类型参数必须有那三个方法。
- 约束接口也可以指定特定类型。

### 5.1 写点代码

- 在main函数上面，增加代码如下：

```go
type Number interface {
    int64 | float64
}
```

- 声明了**Number**接口类型用于做类型约束。
- 在接口内部声明了**int64**和**float64**。实际上，我们是把在函数声明中的类型声明转移到了一种新的类型约束。当我们想用**int64 | float64**作为类型约束的时候，我们可以用**Number**来代替。
- 添加代码如下：

```go
// SumNumbers sums the values of map m. It supports both integers
// and floats as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

- 声明了一个泛型函数，逻辑和之前的相同，但是有了新的接口类型用于取代之前的类型约束。但是效果和之前的一样。
- 在main函数中增加代码如下：

```go
fmt.Printf("Generic Sums with Constraint: %v and %v\n",
    SumNumbers(ints),
    SumNumbers(floats))
```

### 5.2 看看效果

```sh
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```



## 6.完整代码

```go
package main

import "fmt"

type Number interface {
    int64 | float64
}

func main() {
    // Initialize a map for the integer values
    ints := map[string]int64{
        "first": 34,
        "second": 12,
    }

    // Initialize a map for the float values
    floats := map[string]float64{
        "first": 35.98,
        "second": 26.99,
    }

    fmt.Printf("Non-Generic Sums: %v and %v\n",
        SumInts(ints),
        SumFloats(floats))

    fmt.Printf("Generic Sums: %v and %v\n",
        SumIntsOrFloats[string, int64](ints),
        SumIntsOrFloats[string, float64](floats))

    fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
        SumIntsOrFloats(ints),
        SumIntsOrFloats(floats))

    fmt.Printf("Generic Sums with Constraint: %v and %v\n",
        SumNumbers(ints),
        SumNumbers(floats))
}

// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
    var s int64
    for _, v := range m {
        s += v
    }
    return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
    var s float64
    for _, v := range m {
        s += v
    }
    return s
}

// SumIntsOrFloats sums the values of map m. It supports both floats and integers
// as map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}

// SumNumbers sums the values of map m. Its supports both integers
// and floats as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

