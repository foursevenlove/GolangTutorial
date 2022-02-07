# Go教程之Gin

- 本教程以老式爵士唱片为例。

## 0.准备工作

- 安装Go 1.16及以上版本
- IDE
- CLI
- Curl

## 1.设计API endpoints

- /albums
  - **GET**-获取所有的唱片，返回JSON格式。
  - **POST**-根据JSON形式的请求数据添加一个新的唱片。
- /albums/:id
  - **GET**-根据id获取唱片，返回JSON格式数据。



## 2.创建项目

```sh
$ mkdir web-service-gin
$ cd web-service-gin

$ go mod init example/web-service-gin
go: creating new go.mod: module example/web-service-gin
```



## 3.创建数据

- 为了简化，我们直接将数据存在内存中，实际上应该使用数据库。
- 存在内存中意味着停止服务时，数据将丢失。当重启服务时需要重新创建数据。

### 3.1 写点代码

- 在项目根路径下创建文件main.go。

```go
package main

// album represents data about a record album.
type album struct {
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
    {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}
```

- 在struct的声明中，**json:"artist"**表明：当struct的内容被序列化成JSON时，字段的名字应该是“artist”。如果没有这个标签，就是“Artist”（在JSON中，一般使用小写开头的字段）。

## 4. 创建handler-返回所有唱片

- 当客户端发起 **/albums** 的 **GET** 请求时，我们把所有的唱片以JSON形式返回。

### 4.1 写点代码

- 在上一节代码的下面添加代码如下：

```go
// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}
```

- **getAlbums** 函数创建根据**album** struct的slice创建JSON，并且返回JSON。

- **getAlbums** 函数接收 [`gin.Context`](https://pkg.go.dev/github.com/gin-gonic/gin#Context) 作为参数。**gin.Context**是Gin里最重要的一部分，它包括了请求处理，验证和序列化JSON等等（Go中也自带了context包，除了名字不一样，其他的区别在这里[`context`](https://go.dev/pkg/context/)）。

- 调用 [`Context.IndentedJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.IndentedJSON) 来把struct序列化成JSON，并且放在response中。这个函数的第一个参数是发给客户端的HTTP status code，在这里我们传入net/http包下的[`StatusOK`](https://pkg.go.dev/net/http#StatusOK)用来代表 **200 OK**。注意我们可以使用 [`Context.JSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.JSON) 来代替 `Context.IndentedJSON` ，这样可以返回更compact的JSON。实际上，这种形式更容易debug并且尺寸差异很小。

- 在**albums**的slice声下添加以下代码用于把endpoint path分配给handler函数。

```go
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)

    router.Run("localhost:8080")
}
```

- 使用[`Default`](https://pkg.go.dev/github.com/gin-gonic/gin#Default)初始化一个Gin router。

- 使用 [`GET`](https://pkg.go.dev/github.com/gin-gonic/gin#RouterGroup.GET) 函数把 **/albums**的endpoints path和**getAlbums** handler关联起来。注意在这里我们传入的是函数名**getAlbums**，而不是传入函数结果**getAlbums()**。
- 使用[`Run`](https://pkg.go.dev/github.com/gin-gonic/gin#Engine.Run)函数来把router和**http.Server**关联起来，并且启动server。

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)
```

- 在main.go文件的头部，导入相关包。

### 4.2 看看效果吧

- 拉取依赖。在命令行中，使用 [`go get`](https://go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) 把github.com/gin-gonic/gin module加入到我们的module依赖中。使用`.`代表拉取所有依赖。

```sh
$ go get .
go get: added github.com/gin-gonic/gin v1.7.2

$ go run .
```

- 打开新命令行窗口：

```sh
$ curl http://localhost:8080/albums
```

- 展示数据如下：

```sh
[
        {
                "id": "1",
                "title": "Blue Train",
                "artist": "John Coltrane",
                "price": 56.99
        },
        {
                "id": "2",
                "title": "Jeru",
                "artist": "Gerry Mulligan",
                "price": 17.99
        },
        {
                "id": "3",
                "title": "Sarah Vaughan and Clifford Brown",
                "artist": "Sarah Vaughan",
                "price": 39.99
        }
]
```



## 5. 创建handler-新建唱片

### 5.1 写点代码

- 在import后添加以下代码（也可以放在最后，Go中声明的顺序无关紧要）：

```go
// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}
```

- 使用 [`Context.BindJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.BindJSON) 把请求体绑定到**newAlbum**。
- 把初始化后的**album** struct添加到**albums** slice中。
- 返回代码设置为 **201**，并且把新建ablum返回。
- 修改main.go：

```go
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
```

- 把 **/album** 的 **POST** 方法和 **postAlbums** 关联起来。

### 5.2 看看效果

- 重启项目：

```sh
$ go run .
```

- 在新命令行窗口，使用curl来发请求：

```sh
$ curl http://localhost:8080/albums \
    --include \
    --header "Content-Type: application/json" \
    --request "POST" \
    --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
```

- 在windows中或许需要这样（转义双引号并且把单引号改成双引号）：

```sh
curl http://localhost:8080/albums --include  --header "Content-Type: application/json" --request "POST" --data "{\"id\": \"4\",\"title\": \"The Modern Sound of Betty Carter\",\"artist\": \"Betty Carter\",\"price\": 49.99}"
```

- 创建成功应该显示如下信息：

```sh
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Wed, 02 Jun 2021 00:34:12 GMT
Content-Length: 116

{
    "id": "4",
    "title": "The Modern Sound of Betty Carter",
    "artist": "Betty Carter",
    "price": 49.99
}
```

- 使用以下命令来查看所有albums：

```sh
$ curl http://localhost:8080/albums \
    --header "Content-Type: application/json" \
    --request "GET"
```

- 成功应该显示如下信息：

```sh
[
        {
                "id": "1",
                "title": "Blue Train",
                "artist": "John Coltrane",
                "price": 56.99
        },
        {
                "id": "2",
                "title": "Jeru",
                "artist": "Gerry Mulligan",
                "price": 17.99
        },
        {
                "id": "3",
                "title": "Sarah Vaughan and Clifford Brown",
                "artist": "Sarah Vaughan",
                "price": 39.99
        },
        {
                "id": "4",
                "title": "The Modern Sound of Betty Carter",
                "artist": "Betty Carter",
                "price": 49.99
        }
]
```



## 6. 创建handler-返回指定唱片

### 6.1 写点代码

- 在main.go中添加以下代码：

```go
// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
    id := c.Param("id")

    // Loop over the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```

- **getAlbumByID**会把请求路径中的ID提取出来，然后找到对应唱片。
- 使用 [`Context.Param`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.Param) 来获取URL中参数id。当将handler映射到路径时，将在路径中包含参数的占位符。
- 遍历slice，找到对应**ID**的唱片。如果找到，那么讲**album** struct序列化成JSON形式，连同**200 OK**状态码一起返回。实际上应该使用数据库来进行查询。
- 如果唱片找不到，那么返回 [`http.StatusNotFound`](https://pkg.go.dev/net/http#StatusNotFound)。
- 修改main.go：

```go 
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    router.GET("/albums/:id", getAlbumByID)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
```

- 把路径 **/albums/:id**和**getAlbumByID**函数关联起来。在Gin中，在路径中，冒号＋item代表item是一个路径参数。

### 6.2 看看效果

- 重启项目：

```sh
$ go run .
```

- 在新命令行窗口，使用curl来发请求：

```sh
$ curl http://localhost:8080/albums/2
```

- 成功的话应该显示如下信息：

```sh
{
        "id": "2",
        "title": "Jeru",
        "artist": "Gerry Mulligan",
        "price": 17.99
}
```



## 7. 总结

- 关于Gin，详见 [Gin Web Framework package documentation](https://pkg.go.dev/github.com/gin-gonic/gin) 以及 [Gin Web Framework docs](https://gin-gonic.com/docs/)。

- 完整代码：

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

// album represents data about a record album.
type album struct {
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
    {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)
    router.GET("/albums/:id", getAlbumByID)
    router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}

// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}

// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}

// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
    id := c.Param("id")

    // Loop through the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```

