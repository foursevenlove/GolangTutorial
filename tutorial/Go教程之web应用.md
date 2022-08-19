# Go教程之web应用

- 在这一章教程中，主要包含以下内容：
  - 使用load和save方法创建数据结构
  - 使用**net/http**来创建web应用
  - 使用**html/template**包来处理HTML模板
  - 使用**regexp**包来验证用户输入
  - 使用**closures**

## 1.开始

```sh
$ mkdir gowiki
$ cd gowiki
```

- 创建**wiki.go**文件，输入以下代码：

```go
package main

import (
	"fmt"
	"os"
)
```



## 2.数据结构

- 一个wiki由一系列互联的page组成，每一个page都有一个title和body(也就是page的内容)。下面定影**Page**结构体，其中包含两个字段用于代表title和body。

```go
type Page struct {
    Title string
    Body  []byte
}
```

- **[]byte**类型代表一个byte[数组](https://go.dev/doc/effective_go#slices)。之所以**Body**要用byte数组而不是string，是因为**io**库中需要传入byte数组类型，详见下文。
- **Page**结构体表明了page的数据是如何存在内存中的。但是持久存储呢？我们可以通过在**Page**中创建一个**save**方法来解决持久存储。

```go
func (p *Page) save() error {
    filename := p.Title + ".txt"
    return os.WriteFile(filename, p.Body, 0600)
}
```

- 这个方法将Page的body保存成txt文件，简单起见，我们将Title设置问文件名。
- **save**方法返回值为**error**，这是因为**WriteFile**（标准库中函数，用于将字节数组转为文件）的返回值是**error**。**save**方法返回值为**error**，这样可以让应用在写文件出错时来处理错误。如果不报错的话，**Page.save()**将会返回**nil**。
- 传入**WriteFile**的参数，八进制的数**0600**，表明文件只对当前用户有读写权限。
- 如果我们想读取page的话，可以：

```go
func loadPage(title string) *Page {
    filename := title + ".txt"
    body, _ := os.ReadFile(filename)
    return &Page{Title: title, Body: body}
}
```

- **loadPage**函数根据传入的参数title来构建文件名，读取文件的内容，并将内容赋给一个新的变量**body**，最终返回一个指向带有title和body的**Page**的指针。

- 函数可以返回多个值。标准库函数**os.ReadFile**返回字节数组和error。在**loadPage**函数中，并没有处理error；下划线标识符(`_`)用于抛出error。
- 但是如果**ReadFile**函数报错怎么办？比如读取一个不存在文件？我们必须要处理这种错误。修改函数如下：

```go
func loadPage(title string) (*Page, error) {
    filename := title + ".txt"
    body, err := os.ReadFile(filename)
    if err != nil {
        return nil, err
    }
    return &Page{Title: title, Body: body}, nil
}
```

- 调用这个函数时可以检查第二个返回值；如果是**nil**代表成功地读取了Page。如果不是的话，那么error将被调用者处理。
- 现在我们已经有了数据结构并且可以保存和读取文件了。让我们写一个main函数来测试一下吧：

```go
func main() {
    p1 := &Page{Title: "TestPage", Body: []byte("This is a sample Page.")}
    p1.save()
    p2, _ := loadPage("TestPage")
    fmt.Println(string(p2.Body))
}
```

- 在编译并且执行代码后，会创建一个名为TestPage.txt的文件，其中包含了p1的内容。这个文件将被会读取到p2中，并且将body的内容打印出来。

```sh
$ go build wiki.go
$ ./wiki
This is a sample Page.
```

- 如果是windows是话，直接用“wiki”，不需要加"./"。



## 3.`net/http`包

- 下面是一个完整的可以运行的web server的简单例子。

```go
//go:build ignore

package main

import (
    "fmt"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

- 调用**http.HandleFunc**函数，这告诉**http**包，用**handler**来处理所有（**“/”**）的请求。
- 调用**http.ListenAndServe**函数，指定了监听8080端口（先不用管第二个参数nil）。直到程序中断，这个函数会一直运行。
- **ListenAndServe**函数总是会返回一个error，因为它只有当一个unexpected error发生时才会返回。为了打印错误，我们调用**log.Fatal**函数。
- **handler**函数，接收**http.ResponseWriter**和**http.Request**作为参数。
-  `http.ResponseWriter` 的值构成了HTTP服务器的响应，为了写入它，我们将数据发送给HTTP客户端。
- **http.Request**是一种数据结构，代表了客户端的HTTP请求。**r.URL.Path**是请求URL的路径成分。 后面的`[1:]` 代表创建一个从第一个符号到结尾的字slice。这将会忽略路径前面的**“/”**。
- 如果运行程序并且访问这个URL：

```
http://localhost:8080/monkeys
```

- 响应如下：

```
Hi there, I love monkeys!
```



## 4.使用 `net/http` 包来处理wiki page

- 导包：

```go
import (
	"fmt"
	"os"
	"log"
	"net/http"
)
```

- 创建handler函数，**viewHandler**可以让用户访问一个wiki page。它将会处理前缀是"/view/"的URL。

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}
```

- 注意这里为了简单起见，用 `_`来接收error，这样是不好的，我们稍后再关注这个。
- 首先从**r.URL.Path**中提取出page title。**Path**被切割，忽略了开头的“/view/”，因为前面这部分是不变的，不属于page title。
- 接着就加载page data，将Page表示成一个简单的HTML，并且写入**http.ResponseWriter**类型的w。
- 为了使用这个handler，我们重写main函数如下：

```go
func main() {
    http.HandleFunc("/view/", viewHandler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

- 在项目根路径下创建文件 **test.txt**，并且编译代码。
- 在**test.txt**中写入“Hello world”（不要引号）。

```sh
$ go build wiki.go
$ ./wiki
```

- 如果是windows是话，直接用“wiki”，不需要加"./"。
- 浏览器输入 `http://localhost:8080/view/test` ，应该会展示一个page，title是“test”,内容是“Hello world”。

## 5.编辑Pages

- 如果不能编辑的页面就不叫wiki了。让我们来创建两个新的handler， `editHandler` 用来展示编辑Page的表单， `saveHandler`用来保存表单里的数据。

```go
func main() {
    http.HandleFunc("/view/", viewHandler)
    http.HandleFunc("/edit/", editHandler)
    http.HandleFunc("/save/", saveHandler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

-  `editHandler` 函数用于加载一个Page(如果不存在的话就创建一个新的page结构体)，并且展示一个HTML表单。

```go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    fmt.Fprintf(w, "<h1>Editing %s</h1>"+
        "<form action=\"/save/%s\" method=\"POST\">"+
        "<textarea name=\"body\">%s</textarea><br>"+
        "<input type=\"submit\" value=\"Save\">"+
        "</form>",
        p.Title, p.Title, p.Body)
}
```

- 这个函数虽然可以达到我们的目的，但是非常的ugly，因为其中的HTML的硬编码的，我们有更有优雅的做法。

### 6. `html/template` 包

-  `html/template` 是Go的标准库。我们可以使用 `html/template` 包来让HTML在一个单独的文件中，并且可以修改其中的HTML代码而不用修改Go代码。
- 导包：

```go
import (
	"html/template"
	"os"
	"net/http"
)
```

- 创建一个名为edit.html的文件，里面保存的HTML的表单模板。

```html
<h1>Editing {{.Title}}</h1>

<form action="/save/{{.Title}}" method="POST">
<div><textarea name="body" rows="20" cols="80">{{printf "%s" .Body}}</textarea></div>
<div><input type="submit" value="Save"></div>
</form>
```

- 修改 `editHandler` 函数来使用模板，而不是硬编码的HTML：

```go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    t, _ := template.ParseFiles("edit.html")
    t.Execute(w, p)
}
```

-  `template.ParseFiles`函数将会读取**edit.html**的内容并且返回 `*template.Template`。
-  `t.Execute` 方法将会执行上述模板，将生成的HTML写入 `http.ResponseWriter`。**.Title**和**.Body**代表**p.Title**和**p.Body**。
- 模板指令用双花括号括起来。 **printf "%s" .Body** 指令是一个函数调用，它将 **.Body** 作为字符串而不是字节流输出，与对 **fmt.Printf** 的调用相同。

-  `html/template` 包只会通过模板操作生成安全并且正确的HTML。例如，它会自动将所有大于号（>）转为**&gt**，确保用户的输入不会破坏HTML表单。
- 因为我们现在在用模板，所以我们为 `viewHandler` 创建一个模板，名叫**view.html**。

```html
<h1>{{.Title}}</h1>

<p>[<a href="/edit/{{.Title}}">edit</a>]</p>

<div>{{printf "%s" .Body}}</div>
```

- 相应地修改 `viewHandler` ：

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    t, _ := template.ParseFiles("view.html")
    t.Execute(w, p)
}
```

- 注意我们在两个函数中几乎使用了相同的代码，让我们去掉重复代码：

```go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, _ := template.ParseFiles(tmpl + ".html")
    t.Execute(w, p)
}
```

- 相应地修改函数：

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    renderTemplate(w, "view", p)
}
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}
```



## 7.处理不存在的Page

- 如果访问[`/view/APageThatDoesntExist`](http://localhost:8080/view/APageThatDoesntExist)会怎么样？你将会看见一个包含HTML的页面。这是因为我们忽略了**loadPage**的error返回值，并且继续试图用没有数据的模板来填充。相反，如果访问page不存在的话，应该重定向到编辑Page的页面，这样就可以创建新内容。

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}
```

-  `http.Redirect` 函数添加了HTTP code， `http.StatusFound` (302)和HTTP响应的定位。

##  8.保存pages

-  `saveHandler` 函数将会处理表单的提交。

```go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    p.save()
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

- Page的title（在URL里）和表单唯一的字段Body，储存在新的**Page**中。**save()**方法用于将data写成文件，然后客户端将被重定向到**/view/**页面。
- **FormValue**的返回值是**string**类型。在将其传入**Page**结构体之前我们需要转换成字节数组类型。我们使用**[]byte(body)**来完成转换。

## 9.错误处理

- 我们的代码有很多地方都忽略了error。这是不好的，尤其是因为当确实发生错误时，程序会出现意外行为。更好的做法是处理错误并且返回用户错误信息。这样，如果出现问题，服务器将完全按照我们想要的方式运行，并且可以通知用户。

- 首先我们在 `renderTemplate`中处理error：

```go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, err := template.ParseFiles(tmpl + ".html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    err = t.Execute(w, p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
```

-  `http.Error`函数发送了指定的HTTP状态码和错误信息。

- 现在修改 `saveHandler`函数：

```go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err := p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

- 这样在**p.save()**里发生的error就会报告给用户。

## 10.模板缓存

- 这段代码效率低下：每次渲染页面时，**renderTemplate** 都会调用 **ParseFiles**。更好的方法是在程序初始化时调用 **ParseFiles**，将所有模板解析为单个 ***Template**。然后我们可以使用 **ExecuteTemplate** 方法来渲染特定的模板。

- 首先，我们创建一个名为 templates 的全局变量，并使用 ParseFiles 对其进行初始化。

```go
var templates = template.Must(template.ParseFiles("edit.html", "view.html"))
```

- 函数 **template.Must** 是一个方便的包装器，当传递一个非 nil 错误值时会发生panic，否则返回 *Template 不变。恐慌在这里是合适的；如果无法加载模板，唯一明智的做法是退出程序。

- **ParseFiles** 函数采用任意数量的字符串参数来标识我们的模板文件，并将这些文件解析为以基本文件名命名的模板。如果我们要向我们的程序添加更多模板，我们会将它们的名称添加到 **ParseFiles** 调用的参数中。

- 然后我们修改 **renderTemplate** 函数以使用适当模板的名称调用 **templates.ExecuteTemplate** 方法：

```go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    err := templates.ExecuteTemplate(w, tmpl+".html", p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
```

- 请注意，模板名称是模板文件名，因此我们必须将“**.html**”附加到 **tmpl** 参数。

## 11.验证

- 显然，该程序有一个严重的安全漏洞：用户可以提供任意路径以在服务器上读取/写入。为了缓解这种情况，我们可以编写一个函数来使用正则表达式验证标题。

- 首先导包，然后创建一个全局变量来存储我们的验证表达式：

```go
var validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)$")
```

- 函数 **regexp.MustCompile** 将解析和编译正则表达式，并返回一个 **regexp.Regexp**。 **MustCompile** 与 **Compile** 的不同之处在于，如果表达式编译失败，它将**panic**，而 Compile 作为第二个参数返回错误。

- 现在，让我们编写一个函数，使用 **validPath** 表达式来验证路径并提取页面标题：

```go
func getTitle(w http.ResponseWriter, r *http.Request) (string, error) {
    m := validPath.FindStringSubmatch(r.URL.Path)
    if m == nil {
        http.NotFound(w, r)
        return "", errors.New("invalid Page Title")
    }
    return m[2], nil // The title is the second subexpression.
}
```

- 如果标题有效，它将与 nil 错误值一起返回。如果标题无效，该函数将向 HTTP 连接写入“404 Not Found”错误，并向**handler**返回错误。要创建新error，我们必须导入error包。

- 让我们在每个**handler**中调用 **getTitle**：

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err = p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

## 12. 函数Literals和Closures

- 在每一个handler中判断是否有error引进了大量的重复代码。如果我们可以用一个函数来包装每一个handler，用于验证检查错误会如何？Go的 [function literals](https://go.dev/ref/spec#Function_literals)提供了有力的工具来抽象函数。
- 首先，我们重写下列函数：

```go
func viewHandler(w http.ResponseWriter, r *http.Request, title string)
func editHandler(w http.ResponseWriter, r *http.Request, title string)
func saveHandler(w http.ResponseWriter, r *http.Request, title string)
```

- 现在我们定义一个包裹函数，接收上面的函数作为参数，返回 `http.HandlerFunc` ：

```go
func makeHandler(fn func (http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		// Here we will extract the page title from the Request,
		// and call the provided handler 'fn'
	}
}
```

- 返回的函数称为**closure**，因为它包含在其外部定义的值。在这种情况下，变量 fn（**makeHandler** 的单个参数）被**closure**包围。变量 **fn** 将是我们的保存、编辑或查看handler之一。
- 现在我们可以从 **getTitle** 获取代码并在这里使用它（稍作修改）：

```go
func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        m := validPath.FindStringSubmatch(r.URL.Path)
        if m == nil {
            http.NotFound(w, r)
            return
        }
        fn(w, r, m[2])
    }
}
```

- **makeHandler** 返回的**closure**是一个函数，它接受一个 http.ResponseWriter 和 http.Request（换句话说，一个 http.HandlerFunc）。**closure**从请求路径中提取标题，并使用 **validPath** 正则表达式对其进行验证。如果标题无效，则会使用 http.NotFound 函数向 ResponseWriter 写入错误。如果标题有效，则将调用包含的handler函数 **fn**，并将 ResponseWriter、Request 和标题作为参数。

- 现在我们可以在 main 中使用 **makeHandler** 包装处理函数，然后再将它们注册到 **http** 包中：

```go
func main() {
    http.HandleFunc("/view/", makeHandler(viewHandler))
    http.HandleFunc("/edit/", makeHandler(editHandler))
    http.HandleFunc("/save/", makeHandler(saveHandler))

    log.Fatal(http.ListenAndServe(":8080", nil))
}
func viewHandler(w http.ResponseWriter, r *http.Request, title string) {
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}
func editHandler(w http.ResponseWriter, r *http.Request, title string) {
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}
func saveHandler(w http.ResponseWriter, r *http.Request, title string) {
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err := p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

## 13.看看效果

```sh
$ go build wiki.go
$ ./wiki
```

- 访问 http://localhost:8080/view/ANewPage 应该会显示页面编辑表单。然后，您应该能够输入一些文本，单击“保存”，然后被重定向到新创建的页面。