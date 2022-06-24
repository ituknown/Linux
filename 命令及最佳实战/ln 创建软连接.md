# 前言

`ln` 命令主要用于给文件或目录创建有个符号链接，符号链接主要分为软链接和硬链接。

所谓的软链接就是快捷键，比如在 Windows 上，我们经常会为了能够快速启动应用程序或打开文件夹通常都会在桌面上创建一个快捷键。

而硬链接通俗一点的讲就是 “数据拷贝”，拷贝后的数据与源文件是独立的两份，两个操作之间互不影响。

当然了，`ln` 命令在实际中用的最多的还是软链接，所以这里就不对硬链接做任何说明了。

先来看下软链接的具体应用，在我的 `sdk` 目录下安装了多个版本的 Go：

```bash
$ ls ~/sdk
go1.15   go1.16   go1.17   go1.18.3
```

比如现在我有如下一段 Go 程序代码：

```go
package main

func main() {

    var ia int32 = 20
    var ib int32 = 10

    var fa float64 = 20
    var fb float64 = 10

    _ = subtract[int32](ia, ib)

    _ = subtract[float64](fa, fb)
}

func subtract[T int32 | float64](a, b T) T {

    return a - b
}
```

现在有个疑问，如果我现在运行下面的命令，我该使用哪个版本的 Go 呢？

```bash
$ go run mian.go
```

再比如说，如果我当前默认使用的是 `go1.15`，但是我的这段代码使用了泛型（泛型是 `go1.18` 新特性）又该怎么办呢？

这个时候就该考虑使用 `ln` 软链接了，我只需要创建一个软链接符号，在需要的时候分别链接到不同的 Go 版本即可。

比如我的软链接符号是 `/usr/local/go`，当我需要使用 `go1.15` 时我只需要修改下软链接源文件即可：

```bash
$ sudo ln -sf ~/sdk/go1.15 /usr/local/go
$ go version
go version go1.15 darwin/amd64
```

当我想使用泛型特性时，再将软链接的源文件修改为 `go1.18.3` 即可：

```bash
$ sudo ln -sf ~/sdk/go1.18.3 /usr/local/go
$ go version
go version go1.18.3 darwin/amd64
```

现在是不是体会到了 `ln` 软链接的妙用了？现在来看下具体的语法：

# 语法与实例

`ln` 常用的参数语法如下：

```bash
$ ln [-sfiv] SOURCE_FILE LINK_NAME

-s 给源文件(SOURCE_FILE)创建一个软符号链接(LINK_NAME), 即软链接
-f 强制执行, 如果已存在链接则覆盖
-i 交互模式, 文件存在则提示用户是否覆盖
-v 显示详细的处理过程
```

有两个点需要注意：

`SOURCE_FILE` 指的是系统上的具体文件或文件夹。

`LINK_NAME` 则指的是要创建的符号链接。注意，在结尾处一定不要加 `/`。

现在来看下具体示例：

在我的 sdk 目录下有两个版本的 Go：

```bash
$ ls ~/sdk/
go1.15  go1.18.3
```

为了不需要配置环境变量能够直接使用 Go，我只需要将指定版本的 Go 放到 `/usr` 目录下即可。但是直接移动似乎不太好，该怎么办呢？

那就使用软链接！比如我给 `go1.15` 建立一个软链接：`/usr/lib/golang/go`，命令如下：

```bash
$ sudo ln -s ~/sdk/go1.15 /usr/lib/golang/go
```

现在我就可以直接使用 go 环境了，看下 go 版本：

```bash
$ go version
go version go1.15 linux/amd64
```

怎么样？这样效率是不是比配置环境变量方便了好多？

现在我又想使用 `go1.18.3` 该咋办？修改下软链接源文件不就好了？现在将软链接链接到 `go1.18.3`：

```bash
$ sudo ln -s ~/sdk/go1.18.3 /usr/lib/golang/go
ln: failed to create symbolic link '/usr/lib/golang/go': File exists
```

结果提示 `File exists`，意思就是当前软链接已经关联了某个源文件了无法再进行关联。

该怎么办呢？使用 `-i` 参数以交互模式执行即可。这样，当判断软链接已经关联了某个文件时会提示你使用执行替换（replace），我们只需要输入 Y 确认替换即可：

```bash
$ sudo ln -si ~/sdk/go1.18.3 /usr/lib/golang/go
ln: replace '/usr/lib/golang/go'? y
```

或者更简单直接的使用 `-f` 强制执行：

```bash
$ sudo ln -sf ~/sdk/go1.18.3 /usr/lib/golang/go
```

之后再看下 go 版本：

```bash
$ go version
go version go1.18.3 linux/amd64
```

其实，如果想要看链接执行信息的话可以加上 `-v` 参数：

```bash
$ sudo ln -sv /home/ituknown/sdk/go1.18.3 /usr/lib/golang/go
'/usr/lib/golang/go' -> '/home/ituknown/sdk/go1.18.3'
```

怎么样，`ln` 命令使用起来是不是特别简单？

--

https://superuser.com/questions/1702125/operation-not-permitted-when-creating-hard-link-but-soft-link-works

https://unix.stackexchange.com/questions/377676/why-can-i-not-hardlink-to-a-file-i-dont-own-even-though-i-can-move-it/377719#377719

https://unix.stackexchange.com/questions/233275/hard-link-creation-permissions