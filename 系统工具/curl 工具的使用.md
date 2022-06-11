cURL 是 Linux 下非常强大的 HTTP 工具，可用于各种网络协议的 HTTP 请求（GET、POST、DELETE 及 PUT 等）。另外，还可以用于文件下载和网页测试等。

想了解 cURL 的全部用法可以在终端中输入下面的命令查阅，这里只会简单地说下常用的使用方式。

```bash
$ curl --help

# 或

$ man curl
```

# 基本用法

cURL 最基本的用法就是用于网页测试，示例：

```bash
$ curl baidu.com
```

如果网络正常会将网页输出到控制台上，输出示例：

```bash
$ curl baidu.com
<html>
<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
</html>
```

另外，我们可以利用 Linux 重定向或者管道语法对网页做处理。如利用重定向（`>` 或 `>>`）将网页保存到本地文件中：

```bash
$ curl baidu.com > baidu.html
```


# 文件下载

想要使用 cURL 工具实现网络文件下载，我们可以使用 `-o` 或 `-O` 参数（一个大写一个小写）。

命令如下：

```bash
$ curl -o $file_name $remote_url

$ curl -O $remote_url
```

这两个参数的区别是：`-o` 参数我们需要指定一个下载到本地后的文件名。而 `-O` 参数不需要指定下载后的文件名，因为它会使用远程的文件名。

比如有个 mp4 的网络文件地址是：[http://vfx.mtime.cn/Video/2019/02/04/mp4/190204084208765161.mp4](http://vfx.mtime.cn/Video/2019/02/04/mp4/190204084208765161.mp4)。

如果使用 `-O` 参数的话下载到本地的文件名是 [190204084208765161.mp4](http://vfx.mtime.cn/Video/2019/02/04/mp4/190204084208765161.mp4)，不够友好。不过如果使用 `-o` 参数的话，我们重新指定一个文件名。

使用 `-O` 参数下载示例：

```bash
$ curl -O http://vfx.mtime.cn/Video/2019/02/04/mp4/190204084208765161.mp4

$ ls
190204084208765161.mp4
```

使用 `-o` 参数下载示例：

```bash
$ curl -o test.mp4 http://vfx.mtime.cn/Video/2019/02/04/mp4/190204084208765161.mp4

$ ls
190204084208765161.mp4	test.mp4
```

有时候我们可能仅仅需要测试下网网速，不需要真正的下载到本地。这个我们可以将文件标准输出重定向到 /dev/null：

```bash
$ curl -o /dev/null http://vfx.mtime.cn/Video/2019/02/04/mp4/190204084208765161.mp4
```


## 文件重定向跟踪

有时候需要下载的文件被重定向了，比如想要下载 mysql 软件包：

```
https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

该软件包就被重定向了。如果直接使用 cURL 下载的话，是没办法下载到正确的文件的：

```bash
$ curl -O https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz

$ ls -lh
total 0
-rw-r--r-- 1 root root 0 Oct 21 21:50 mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

所以我们就需要进行重定向跟踪，cURL 提供了 -L 参数。-L 参数指明若有重定向则跟踪重定向，这样即使文件被重定向了也能够正确下载了：


```bash
$ curl -O -L https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

**小提示：在实际使用中，建议每次下载文件时都加上 -L 参数。**


## 文件下载进度条

默认情况下，下载文件时不会显示进度条。而是显示一箩筐百分比的输出，如下：

```bash
$ curl -L -O https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  7 1058M    7 76.0M    0     0  1568k      0  0:11:31  0:00:49  0:10:42 1179k
```

**我们可以使用 `-#` 参数进行指定显示下载进度。**

命令示例：

```bash
$ curl -# -L -O https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

输出示例：

```bash
$ curl -# -L -O https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz

##############################                                                                   30.2%
```

这是不是就清爽多了？


## 断点续传

断点续传这个功能在实际中很有用，比如上面的 mysql 文件比较大，下载的比较慢。但是你又急着干其他事情，怎么办？没事，先终端任务。之后再使用断点续传即可。

cURL 提供了 -C 参数用于断点续传。需要说明的是，想要实现断点续传的前提是你的文件名必须相同。因为在执行断点续传时，cURL 会判断要下载的文件在当前目录是否存在，如果存在会判断下载大小，会接着下载。但是如果文件名不存在就会重新下载。

**`-C` 参数在使用时需要注意，该参数后面跟随的是偏移量，即已经下载的大小。**

我们可以使用 `ls -l` 命令查看文件已经下载的大小：

```bash
$ ls -l
total 115108
-rw-r--r-- 1 root root 117870592 Oct 21 22:12 mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

其中 117870592 就是已经下载的大小（注意单位 bit，所以上面使用的命令是 `ls -l`）。如果我们想要继续下载的话，只需要将该数值添加到 -C 参数之后即可，如下：

```bash
$ curl -L -O -C 117870592 $remote_url
```

但是如果你不想输入具体偏移量，可以直接使用 - 代替，即 `-C -`。cURL 会自己分析该从什么位置开始续传，所以在断点续传时，使用 `-C -` 更加方便。

示例：

```bash
$ curl -# -L -O https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

等下载到 10% 时手动终止任务（ctrl + c），并使用 -C 参数，后面接着一个偏移量继续下载：

```bash
$ curl -# -L -O -C https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

等下载到 20% 时再次手动终止任务（ctrl + c），并使用 -C 参数，后面使用 - 代替具体偏移量继续下载：

```bash
$ curl -# -L -O -C - https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-i686.tar.xz
```

可以看下下面的 asciinema 演示示例：

[![asciicast](https://asciinema.org/a/444124.svg)](https://asciinema.org/a/444124)


## 批量下载

这种下载方式需要保证网络文件具有一定的规则才行，最典型的就是前缀相同，后缀是递增数值。

比如在 ftp 服务器中有 jdk 软件包，文件名分别是：

```bash
jdk-7.tar.gz
jdk-8.tar.gz
jdk-9.tar.gz
```

中间的数值是递增的，这用在 curl 中就可以使用类似循环的方式进行下载，示例如下：

```bash
$ curl -O http://ip:port/software/jdk-[7-9].tar.gz
```

## 连接超时

设置连接超时时间：

```bash
--connect-timeout <seconds>
```

示例：

```bash
$ curl --connect-timeout 1 google.com
curl: (28) Connection timed out after 1000 milliseconds
```

## 失败重试

在请求或下载文件时如果由于网络问题终止下载，我们可以设置失败重试参数：

```bash
--retry-all-errors            遇到任何错误都进行失败重试
--retry <num>                 连续失败重试次数
--retry-delay <seconds>       重试延迟间隔
--retry-max-time <seconds>    最大重试时间(优先级大于 retry*retry-delay)
```

示例：

**失败重试：**

```bash
$ curl --connect-timeout 1 --retry-all-errors --retry 3 google.com
curl: (28) Connection timed out after 1001 milliseconds
Warning: Problem : timeout. Will retry in 1 seconds. 3 retries left.
curl: (28) Connection timed out after 1001 milliseconds
Warning: Problem : timeout. Will retry in 2 seconds. 2 retries left.
curl: (28) Connection timed out after 1001 milliseconds
Warning: Problem : timeout. Will retry in 4 seconds. 1 retries left.
curl: (28) Connection timed out after 1001 milliseconds
```

**失败重试间隔：**

```bash
$ curl --connect-timeout 1 --retry 3 --retry-delay 3 google.com
curl: (28) Connection timed out after 1001 milliseconds
Warning: Problem : timeout. Will retry in 3 seconds. 3 retries left.
curl: (28) Connection timed out after 1001 milliseconds
Warning: Problem : timeout. Will retry in 3 seconds. 2 retries left.
curl: (28) Connection timed out after 1001 milliseconds
Warning: Problem : timeout. Will retry in 3 seconds. 1 retries left.
curl: (28) Connection timed out after 1001 milliseconds
```

**最大重试时间：**


正常来说，最大重试时间是 `retry` * `retry-delay`，但是我们可以通过指定 `--retry-max-time` 来重置最大失败时间。比如设置的重试次数是3，重试间隔为 3s，理论上失败时间到达 9s 才会取消继续执行。但是如果将 `--retry-max-time` 设置为 6 秒，那么实际上只会重试两次就终止了。

```bash
$ curl --connect-timeout 1 --retry 3 --retry-delay 3 --retry-max-time 6 google.com
curl: (28) Connection timed out after 1001 milliseconds
Warning: Problem : timeout. Will retry in 3 seconds. 3 retries left.
curl: (28) Connection timed out after 1000 milliseconds
Warning: Problem : timeout. Will retry in 3 seconds. 2 retries left.
curl: (28) Connection timed out after 1001 milliseconds
```


# 设置代理服务器

curl 这个工具在大多时候可能都是用于文件下载，最典型的就是下载 github 上的软件包，国内的网速简直快的感人。这个时候我们可能就需要借助代理服务器去下载了~

比如我想要下载 Github 上的软件包：[https://github.com/xx/xx/releases/download/v1.7.1/file-v1.7.1.gz](https://github.com/xx/xx/releases/download/v1.7.1/file-v1.7.1.gz)

如果直接使用 cURL 下载的话可能会很慢，不过 cURL 提供了一个 `-x` 参数用于设置代理服务器：

```bash
$ curl -x $proxy_server $remote_url
```

这样，通过走代理的模式就能够光速下载了~

比如我的代理服务器地址是：192.168.1.8:7890，使用该代理下载 Clash 软件包示例命令如下：

```bash
$ curl -O -L -x 192.168.1.8:7890 https://github.com/xx/xx/releases/download/v1.7.1/file-v1.7.1.gz
```

**注意使用 -L 参数，用于跟踪重定向**


# 设置 Referer

这个 Referer 就是我们常说的盗链，设置在请求头上，cURL 工具也可以伪造盗链，使用 -e 或者 --referer 参数设置即可。

命令示例：

```bash
$ curl -e $your_websit_domain $remote_url

# 或

$ curl --referer $your_websit_domain $remote_url
```

为什么需要伪造盗链呢？

那是因为一些知名的服务器都会检查 http 访问的 referer 从而来控制访问。比如：你是先访问首页，然后再访问首页中的邮箱页面，这里访问邮箱的referer 地址就是访问首页成功后的页面地址，如果服务器发现对邮箱页面访问的referer地址不是首页的地址，就断定那是个盗连了。

示例：

```bash
$ curl --referfer "www.aliyun.com" http://bucket.aliyun.com/hangzhou_oss/xxx.png
```


#  用户认证

如果是下载/上传普通的网络文件还好，但是如果是 FTP 服务器文件可能就需要一定得权限才行，典型的就是需要用户进行登录。

cURL 提供了 `-u`（同 `--user`） 参数用于指定用户名密码，命令如下：

```bash
curl -u $username:$password $remote_url

# 或

curl --user $username:$password $remote_url
```

示例下载 ftp 服务器文件：

```bash
$ curl -O -u webuser:admin123 ftp://172.17.5.2:9000/software/os/debian-10.iso
```


# 文件上传

cURL 还可以用于文件上传，使用 -T 参数即可，后面跟具体的文件。

比如将 /opt/software 目录下的 ubunti-18.iso 文件上传到 FTP 服务器：

```bash
$ curl -T /opt/software/ubunti-18.iso -u webuser:admin123 ftp://172.17.5.2:9000/software/os/
```


# CRUD 请求

这个就比较厉害了，也是我们日常工作中用于接口测试最常用的姿势。

cURL 有个 -X 参数，用于发送具体的网络请求，即：GET、PUT、DELETE、POST 等等。下面以 POST 请求为例，其他同理：

## POST multipart/form-data 单文件上传

```BASH
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"
```

对应的后台代码（Go语言为例）：

```go
func main() {
	router := gin.Default()
	// Set a lower memory limit for multipart forms (default is 32 MiB)
	router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Single file
		file, _ := c.FormFile("file")
		log.Println(file.Filename)

		// Upload the file to specific dst.
		c.SaveUploadedFile(file, dst)

		c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
	})
	router.Run(":8080")
}
```

## POST multipart/form-data 多文件上传

```bash
curl -X POST http://localhost:8080/upload \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

对应的后台代码（Go语言为例）：

```go
func main() {
	router := gin.Default()
	// Set a lower memory limit for multipart forms (default is 32 MiB)
	router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["upload[]"]

		for _, file := range files {
			log.Println(file.Filename)

			// Upload the file to specific dst.
			c.SaveUploadedFile(file, dst)
		}
		c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
	})
	router.Run(":8080")
}
```

## POST application/json 请求

**方式一：**

```bash
curl -XPOST "localhost:8080/account" -H 'Content-Type: application' -d'{
"username": "LiLei",
"age": 18
}'
```


**方式二：**

这种方式是从文件中读取二进制流文件数据传输，适用于大 JSON 数据请求。比如一个 JSON 文件 `account.json` 在当前 `_file` 目录下，内容为：

```json
{
	"username": "LiLei",
	"age": 18
}
```

我们可以使用 `--data-binary` 参数从文件中读取二进制流来发起请求，任何数据在网络中其实都是以二进制流的形式传输的。我们只需要指定 `Content-Type` 就可以达到与方式一等效的请求（实际上方式一也是二进制流）：

```bash
curl -XPOST "localhost:8080/account" -H "Content-Type: application/json" --data-binary "@_file/accounts.json"
```

对应的后台代码（Go语言为例）：

```go
type Account struct {
	Username string `json:username`
	Age      uint8  `json:age`
}

func main() {
	router := gin.Default()
	router.POST("/account", func(c *gin.Context) {

		account := new(Account)

		if err := c.ShouldBindJSON(account); err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{
				"message": "read data fail",
			})
			return
		}

		c.JSON(http.StatusOK, gin.H{
			"message": "ok"
		})
	})

	router.Run(":8080")
}
```