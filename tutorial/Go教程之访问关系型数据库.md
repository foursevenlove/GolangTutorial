## Go教程之访问关系型数据库

- 这一章教程介绍了使用标准库中的**database/sql**包来访问关系型数据库。
- **database/sql**包中包含许多函数，如连接数据库、执行事务、取消进程的操作等等。详见[Accessing databases](https://go.dev/doc/database/index)。

- 在这一章中，我们将会创建一个数据库，并且通过代码来访问数据库。我们的示例项目是有关老式爵士唱片的仓库。



## 0.准备工作

- 安装 **[MySQL](https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/)**
-  [安装 Go](https://go.dev/doc/install)
- IDE
- CLI



## 1.创建文件夹

```sh
$ mkdir data-access
$ cd data-access
$ go mod init example/data-access
go: creating new go.mod: module example/data-access
```



## 2.设置数据库

- 可以使用 CLI 或者 Navicat ，下面演示如何使用CLI设置数据库，详见[MySQL CLI](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)。

### 2.1 登录MySQL

```sh
$ mysql -u root -p
Enter password:

mysql>
```

### 2.2 创建并使用数据库

```sh
mysql> create database recordings;

mysql> use recordings;
Database changed
```

### 2.3  create-tables.sql 

- 在data-access文件夹中，创建create-tables.sql文件来保存用于创建表的SQL脚本。
- 在create-tables.sql中修改如下：

```sql
DROP TABLE IF EXISTS album;
CREATE TABLE album (
  id         INT AUTO_INCREMENT NOT NULL,
  title      VARCHAR(128) NOT NULL,
  artist     VARCHAR(255) NOT NULL,
  price      DECIMAL(5,2) NOT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO album
  (title, artist, price)
VALUES
  ('Blue Train', 'John Coltrane', 56.99),
  ('Giant Steps', 'John Coltrane', 63.99),
  ('Jeru', 'Gerry Mulligan', 17.99),
  ('Sarah Vaughan', 'Sarah Vaughan', 34.98);
```

- 运行创建好的脚本：

```sh
mysql> source /path/to/create-tables.sql
```

### 2.4 检查是否创建成功

```sh
mysql> select * from album;
+----+---------------+----------------+-------+
| id | title         | artist         | price |
+----+---------------+----------------+-------+
|  1 | Blue Train    | John Coltrane  | 56.99 |
|  2 | Giant Steps   | John Coltrane  | 63.99 |
|  3 | Jeru          | Gerry Mulligan | 17.99 |
|  4 | Sarah Vaughan | Sarah Vaughan  | 34.98 |
+----+---------------+----------------+-------+
4 rows in set (0.00 sec)
```



## 3.导入数据库驱动

- 找到并且导入数据库驱动，这样可以把我们通过**database/sql**包中的函数创建的请求翻译成数据库可以明白的请求。

### 3.1 找到合适的驱动

- 在 [SQLDrivers](https://github.com/golang/go/wiki/SQLDrivers) 界面寻找合适的驱动，在这一章中我们使用[Go-MySQL-Driver](https://github.com/go-sql-driver/mysql/)。

### 3.2 main.go

- 在data-access文件夹中，创建main.go文件，增加代码如下：

```go
package main

import "github.com/go-sql-driver/mysql"
```

- 在main包中增加代码，这样就可以独立的执行。
- 导入了MySQL的驱动**github.com/go-sql-driver/mysql**。
- 有了驱动，这样就可以写代码来连接数据库了。



## 4.获取数据库句柄并且连接

- 我们使用了指向`sql.DB` 结构体的指针，它代表着访问特定数据库。

### 4.1 写点代码

- 在main.go文件中，**import**的下一行，粘贴以下代码来创建数据库句柄：

```go
var db *sql.DB

func main() {
    // Capture connection properties.
    cfg := mysql.Config{
        User:   os.Getenv("DBUSER"),
        Passwd: os.Getenv("DBPASS"),
        Net:    "tcp",
        Addr:   "127.0.0.1:3306",
        DBName: "recordings",
    }
    // Get a database handle.
    var err error
    db, err = sql.Open("mysql", cfg.FormatDSN())
    if err != nil {
        log.Fatal(err)
    }

    pingErr := db.Ping()
    if pingErr != nil {
        log.Fatal(pingErr)
    }
    fmt.Println("Connected!")
}
```

- 声明了[`*sql.DB`](https://pkg.go.dev/database/sql#DB)类型的**db**变量，这是我们的数据库句柄。把**db**放在全局变量的位置上来简化代码。在生产中，一般避免全局变量，只在需要这个变量的函数中传入变量或者把变量携程结构体。
- 使用MySQL驱动的**[`Config`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config)**，以及[`FormatDSN`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config.FormatDSN)用来进行属性配置并且把配置格式化成DSN的形式用于连接。
- 调用[`sql.Open`](https://pkg.go.dev/database/sql#Open)来初始化db变量，传入**FormatDSN**的返回值。
- 检查**sql.Open**是否存在error。如果数据库连接配置格式不正确那么就有可能连接失败。为了简化代码，我们调用**log.Fatal**来停止执行，并且在控制台打印错误信息。在生产中，不会这样写，因为不优雅。
- 调用 [`DB.Ping`](https://pkg.go.dev/database/sql#DB.Ping) 来验证数据库连接是否正常。在运行时，取决于驱动，**sql.Open**可能不会立即连接。这里我们使用**Ping**来确定，当需要时，可以使用**database/sql**包来连接。
- 如果连接失败，**Ping**会返回error，并且打印错误信息。

- 导入相关包：

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "os"

    "github.com/go-sql-driver/mysql"
)
```

### 4.2看看效果

- 把MySQL的驱动module作为依赖。使用[`go get`](https://go.dev/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them) 把github.com/go-sql-driver/mysql的module添加到自己的module中。使用 `.` 代表获取当前目录下的所有依赖：

```sh
$ go get .
go get: added github.com/go-sql-driver/mysql v1.6.0
```

- 因为我们在**import**中增加了依赖，所以Go会下载相关依赖，详见[Adding a dependency](https://go.dev/doc/modules/managing-dependencies#adding_dependency)。

- 设置代码中用到的环境变量：

```sh
C:\Users\you\data-access> set DBUSER=username
C:\Users\you\data-access> set DBPASS=password
```

- 在包含main.go的文件夹中，运行**go run . ** 命令：

```sh
$ go run .
Connected!
```

- 连接上之后，下面我们就来查询数据。



## 5.查询多行

- SQL语句查询多行数据，使用**database/sql**包下的**Query**方法，然后循环它返回的行。

### 5.1 写点代码

- 在main.go文件中，在**func main**上面，粘贴以下代码作为**Album**结构体的定义，我们用**Album**来保存查询返回的数据。

```go
type Album struct {
    ID     int64
    Title  string
    Artist string
    Price  float32
}
```

- 在**func main**下方，粘贴以下代码来实现**albumsByArtist**函数用于查询数据库：

```go
// albumsByArtist queries for albums that have the specified artist name.
func albumsByArtist(name string) ([]Album, error) {
    // An albums slice to hold data from returned rows.
    var albums []Album

    rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
    if err != nil {
        return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
    }
    defer rows.Close()
    // Loop through rows, using Scan to assign column data to struct fields.
    for rows.Next() {
        var alb Album
        if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
            return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
        }
        albums = append(albums, alb)
    }
    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
    }
    return albums, nil
}
```

- 声明了一个**Album** 类型的slice，变量名为**albums**。我们用这个变量来保存返回行的数据。结构体的字段名和类型和数据库中的列名和类型相对应。
- 使用 [`DB.Query`](https://pkg.go.dev/database/sql#DB.Query) 来执行**SELECT**语句来查询指定作家姓名的唱片。`Query`的第一个参数是SQL语句。在这个参数后，我们可以传入任意个数量、任意类型的参数。这些参数用于指定SQL语句中用到的特定值。通过讲SQL语句和参数值分开（而不是用**fmt.Sprintf**连接），我们可以让**database/sql**包识别哪些是SQL语句，哪些是要传入的参数值，从而避免了SQL注入的风险。
- 直到函数结束才进行**rows**的释放。
- 循环返回的多行数据，使用 [`Rows.Scan`](https://pkg.go.dev/database/sql#Rows.Scan) 来把每一行数据的列值赋值给Album结构体的属性。**Scan**函数接收一组指针作为参数，并且将列的值写入这些指针所指的地方。在这里，我们把**alb**变量的字段的地址传入**Scan**函数，然后把这一行的值按照字段名通过**alb**字段的指针赋值给**alb**的字段。
- 在循环中，检查把每一行的列值复制给结构体的字段时是否有error。
- 在循环中，把**alb**追加到**albums**中。
- 在循环后，使用**rows.Err**来检查全部查询是否有error。如果查询失败，只有通过检查是否存在error来判断结果不完整。

- 在**main**函数中调用**albumsByArtist**，在**main**函数的结构，增加以下代码：

```go
albums, err := albumsByArtist("John Coltrane")
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Albums found: %v\n", albums)
```

### 5.2 看看效果

- 在包含main.go的文件夹中，运行 go run。

```sh
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
```



## 6.查询单行

- 这一节中我们使用Go来查询单行数据。
- 使用**QueryRow**来查询单行数据，这比使用**Query**再使用循环要简单。

### 6.1 写点代码

- 在**albumsByArtist**下面，实现**albumByID**函数：

```go
// albumByID queries for the album with the specified ID.
func albumByID(id int64) (Album, error) {
    // An album to hold data from the returned row.
    var alb Album

    row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
    if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
        if err == sql.ErrNoRows {
            return alb, fmt.Errorf("albumsById %d: no such album", id)
        }
        return alb, fmt.Errorf("albumsById %d: %v", id, err)
    }
    return alb, nil
}
```

- 使用 [`DB.QueryRow`](https://pkg.go.dev/database/sql#DB.QueryRow) 来执行用于查询指定ID唱片的SQL语句。这个函数仅返回一行。为了简化代码，**QueryRow**不会返回错误。相反，从**Rows.Scan**中返回错误（例如： `sql.ErrNoRows`）。
- 使用 [`Row.Scan`](https://pkg.go.dev/database/sql#Row.Scan) 来赋值。
- 检查**Scan**是否返回error。**sql.ErrNoRows**表明查询结构为空。
- 在**main**函数中调用**albumByID**:

```go
// Hard-code ID 2 here to test the query.
alb, err := albumByID(2)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Album found: %v\n", alb)
```

### 6.2 看看效果吧

- 在包含main.go的文件夹中，运行 go run。

```sh
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
```



## 7.增加数据

- 在这一节中，使用Go来执行**INSERT**语句来向数据库中增加一行。
- 我们已经直到了如何使用**Query**和**QueryRow**来执行有返回值的SQL语句。如果执行无返回值的SQL语句，我们使用**Exec**。

### 7.1 写点代码

- 在**albumByID**下方，增加以下代码实现**addAlbum**函数：

```go
// addAlbum adds the specified album to the database,
// returning the album ID of the new entry
func addAlbum(alb Album) (int64, error) {
    result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
    if err != nil {
        return 0, fmt.Errorf("addAlbum: %v", err)
    }
    id, err := result.LastInsertId()
    if err != nil {
        return 0, fmt.Errorf("addAlbum: %v", err)
    }
    return id, nil
}
```

- 使用 [`DB.Exec`](https://pkg.go.dev/database/sql#DB.Exec) 来执行**INSERT**语句。
- 检查**INSERT**有没有error。
- 使用 [`Result.LastInsertId`](https://pkg.go.dev/database/sql#Result.LastInsertId)来获取上一次插入数据库中数据的ID。
- 检查获取ID有没有error。
- 在main函数中调用**addAlbum**函数：

```go
albID, err := addAlbum(Album{
    Title:  "The Modern Sound of Betty Carter",
    Artist: "Betty Carter",
    Price:  49.99,
})
if err != nil {
    log.Fatal(err)
}
fmt.Printf("ID of added album: %v\n", albID)
```

### 7.2 看看效果

- 在包含main.go的文件夹中，运行 go run。

```sh
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
ID of added album: 5
```



## 8. 总结

- [Effective Go](https://go.dev/doc/effective_go)、 [How to write Go code](https://go.dev/doc/code)。
-  [Go Tour](https://go.dev/tour/) 

- 完整代码：

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "os"

    "github.com/go-sql-driver/mysql"
)

var db *sql.DB

type Album struct {
    ID     int64
    Title  string
    Artist string
    Price  float32
}

func main() {
    // Capture connection properties.
    cfg := mysql.Config{
        User:   os.Getenv("DBUSER"),
        Passwd: os.Getenv("DBPASS"),
        Net:    "tcp",
        Addr:   "127.0.0.1:3306",
        DBName: "recordings",
    }
    // Get a database handle.
    var err error
    db, err = sql.Open("mysql", cfg.FormatDSN())
    if err != nil {
        log.Fatal(err)
    }

    pingErr := db.Ping()
    if pingErr != nil {
        log.Fatal(pingErr)
    }
    fmt.Println("Connected!")

    albums, err := albumsByArtist("John Coltrane")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Albums found: %v\n", albums)

    // Hard-code ID 2 here to test the query.
    alb, err := albumByID(2)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Album found: %v\n", alb)

    albID, err := addAlbum(Album{
        Title:  "The Modern Sound of Betty Carter",
        Artist: "Betty Carter",
        Price:  49.99,
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("ID of added album: %v\n", albID)
}

// albumsByArtist queries for albums that have the specified artist name.
func albumsByArtist(name string) ([]Album, error) {
    // An albums slice to hold data from returned rows.
    var albums []Album

    rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
    if err != nil {
        return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
    }
    defer rows.Close()
    // Loop through rows, using Scan to assign column data to struct fields.
    for rows.Next() {
        var alb Album
        if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
            return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
        }
        albums = append(albums, alb)
    }
    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
    }
    return albums, nil
}

// albumByID queries for the album with the specified ID.
func albumByID(id int64) (Album, error) {
    // An album to hold data from the returned row.
    var alb Album

    row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
    if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
        if err == sql.ErrNoRows {
            return alb, fmt.Errorf("albumsById %d: no such album", id)
        }
        return alb, fmt.Errorf("albumsById %d: %v", id, err)
    }
    return alb, nil
}

// addAlbum adds the specified album to the database,
// returning the album ID of the new entry
func addAlbum(alb Album) (int64, error) {
    result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
    if err != nil {
        return 0, fmt.Errorf("addAlbum: %v", err)
    }
    id, err := result.LastInsertId()
    if err != nil {
        return 0, fmt.Errorf("addAlbum: %v", err)
    }
    return id, nil
}
```





