# Go教程之module 7：编译、安装应用

- **go build**命令编译包以及用到的依赖，但是并没有安装。
- **go install**命令编译并且安装包。

## 1.go build

- 在hello文件夹下，运行 **go build**命令把代码编译成可执行文件。

```sh
$ go build
```

## 2.运行编译好的可执行文件

- 在hello文件夹下运行以下命令来运行刚刚编译好的可执行文件：

```sh
$ hello.exe
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

- 在第一步中我们把应用编译成了可执行文件，这样就可以直接运行。但是想要运行可执行文件，我们必须位于该可执行文件的文件夹中，或者我们需要指定该可执行文件的路径。
- 在下一步中我们讲安装可执行文件，这样运行时就不需要指定路径。

## 3.找到Go安装路径

- 使用 go list 命令来找到将会把包安装在什么路径，在hello文件夹下运行一下命令：

```sh
$ go list -f '{{.Target}}'
```

- 命令可能的输出是：**/home/gopher/bin/hello**，这意味着可执行文件将会安装在/home/gopher/bin文件夹下。
- 关于go list，详见[`go list` ](https://go.dev/cmd/go/#hdr-List_packages_or_modules)。

## 4.把Go的安装路径添加到系统脚本路径中

- 也就是把第三步得到的路径加到环境变量中。
- 在Linux或Mac：

```sh
$ export PATH=$PATH:/path/to/your/install/directory
```

- 在windows：

```sh
$ set PATH=%PATH%;C:\path\to\your\install\directory
```

- 或者，如果你已经在脚本路径中设置了**$HOME/bin**这样的文件夹并且你想把程序安装在那里，那么可以通过设置**GOBIN**来改变目标安装路径，使用 [`go env` ](https://go.dev/cmd/go/#hdr-Print_Go_environment_information)命令。

```sh
$ go env -w GOBIN=/path/to/your/bin
```

或者：

```sh
$ go env -w GOBIN=C:\path\to\your\bin
```



## 5.go install

- 在hello文件夹下，运行**go install**命令。

## 6.看看效果吧

- 打开一个新的命令行窗口，不用cd到代码所在目录，直接运行hello命令。

```sh
$ hello
map[Darrin:Hail, Darrin! Well met! Gladys:Great to see you, Gladys! Samantha:Hail, Samantha! Well met!]
```

