# 前言

Linux `curl` 是一个利用URL规则在命令行下工作的文件传输工具，可以说是一款很强大的 `http` 命令行工具。它支持文件的上传和下载，是综合传输工具，我们习惯称 `curl` 为下载工具。

当前各操作系统都有 `curl` 相关工具，我们可以利用 `curl` 干什么？`curl` 所执行的 `http` 命令，所以我们可以利用该工具实现 `GET`、`POST`、`PUT` 以及 `DELETE` 接口调用。剩下的，你懂的！

基本语法：

```bash
$ curl [option] [url]
```

具体参数可执行 `--help` 命令查看：

```bash
$ curl --help
```

# 基本用法

```bash
$ curl baidu.com
```

执行后，百度的 `html` 就会展示在控制台上：

```html
<html>
<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
</html>
```

# 保存网页

这个功能类似于浏览器快捷键 `ctrl + s` 功能，就是用于保存 HTML 页面使用，如下所示保存 `www.baidu.com` 网站并命令为 `baidu.html`

```bash
$ curl www.baidu.com >> baidu.html

% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    81  100    81    0     0     81      0  0:00:01 --:--:--  0:00:01   574

$ ls
baidu.html
```

另外，也可以使用 `-o` 选项进行保存网页：

```bash
$ curl -o baidu.html www.baidu.com

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2381  100  2381    0     0   2381      0  0:00:01 --:--:--  0:00:01 37793
```

# 保存文件

保存文件可使用 `-O` 选项：

```bash
$ curl -O https://one-road.oss-cn-hangzhou.aliyuncs.com/1.mp4

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 2285k  100 2285k    0     0  2285k      0  0:00:01  0:00:01 --:--:-- 1700k
```

# 保存网站 Cookie

由于 `HTTP` 请求是无状态的，所以有些网站会利用 `Cookie` 来进行记录 `Session` 信息。对于 `chrome` 这样的浏览器很容易去处理 `cookie` 信息。`curl` 工具增加相关参数也很容易就能处理 `cookie`：

## 保存 HttpResponse Cookie 信息

```bash
$ curl -c cookiec.txt http://www.baidu.com

$ ls
cookiec.txt

$ cat cookiec.txt
# Netscape HTTP Cookie File
# https://curl.haxx.se/docs/http-cookies.html
# This file was generated by libcurl! Edit at your own risk.

.baidu.com      TRUE    /       FALSE   1573636337      BDORZ   27315
```

## 保存 HttpResponse Header 信息

```bash
$ curl -D header.txt http://www.baidu.com

$ ls
header.txt

$ cat header.txt
HTTP/1.1 200 OK
Accept-Ranges: bytes
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Connection: Keep-Alive
Content-Length: 2381
Content-Type: text/html
Date: Tue, 12 Nov 2019 09:21:54 GMT
Etag: "588604c8-94d"
Last-Modified: Mon, 23 Jan 2017 13:27:36 GMT
Pragma: no-cache
Server: bfe/1.0.8.18
Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
```

> **注意：** `curl` 选项 `-c` 和 `-D` 保存的 Cookie 信息不一样！

## 使用 Cookie 进行网站请求

很多网站都是通过监视你的 `Cookie` 信息来判断你是否按规矩访问他们的网站的，因此我们需要使用保存的 `Cookie` 信息。

```bash
$ curl -b cookiec.txt http://www.baidu.com
```

# 模仿浏览器

有些网站需要使用特定的浏览器去访问他们，有些还需要使用某些特定的版本。`curl` 内置 `option:-A` 可以让我们指定浏览器去访问网站：

```bash
$ curl -A "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.0)" http://www.linux.com
```

这样 Linux 网站服务器端就会认为是使用 `IE8.0` 去访问的。

# 文件下载

- 使用内置选项 `-o`：

```bash
$ curl -o logo1.jpg http://www.linux.com/logo1.JPG
```

- 使用内置选项 `-O`：

```bash
$ curl -O http://www.linux.com/logo1.JPG
```

这样就会以服务器上的名称保存文件到本地

## 循环下载

有时候下载图片可能前面的部分名称是一样的，就最后的尾椎名不一样：

```bash
$ curl -O http://www.linux.com/logo[1-5].JPG
```

这样就会把 `logo1`、`logo2`、`logo3`、`logo4`、`logo5` 全部保存下来!

## 循环下载并重命名

```bash
$ curl -O http://www.linux.com/{hello,bb}/logo[1-5].jpg
```

由于下载的 `hello` 与 `bb` 中的文件名都是 `logo1`、`logo2`、`logo3`、`logo4`、`logo5`。因此第二次下载的会把第一次下载的覆盖，这样就需要对文件进行重命名。

```bash
$ curl -o #1_#2.jpg http://www.linux.com/{hello,bb}/logo[1-5].jpg
```

这样在 `hello/logo1.jpg` 的文件下载下来就会变成 `hello_logo1.jpg`，其他文件依此类推，从而有效的避免了文件被覆盖。

## 通过 ftp 下载文件

`curl` 提供两种从 `ftp` 中下载的语法：

```bash
$ curl -O -u <username>:<password> ftp://www.linux.com/logo1.jpg
$ curl -O ftp://<username>:<password>@www.linux.com/logo1.jpg
```

## ftp 文件上传

`curl` 不仅仅可以通过 `ftp` 下载文件，还可以上传文件。通过内置 `option:-T` 来实现：

```bash
$ curl -T logo1.jpg -u <username>:<password> ftp://www.linux.com/img/
```

## 显示下载进度信息

```bash
$ curl -# -O http://www.linux.com/logo.jpg
```

## 隐藏下载进度信息

```bash
$ curl -s -O http://www.linux.com/logo.jpg
```

# CURL 增删改查

`curl` 指定请求类型可使用 `-X` 选项，示例：

```bash
$ curl -X GET [url]
$ curl -X PUT [url]
$ curl -X POST [url]
$ curl -X DELETE [url]
```

## 指定请求头信息

以 `POST` 为例：

```bash
$ curl -X POST [url] -H "Content-Type: application/json"
```

如果需要指定多个则指定多个即可：

```bash
$ curl -X POST [url] -H "Content-Type: application/json" -H "Token: xxxx"
```

## 提交请求数据

`curl` 请求数据可使用 `-d` 选项指定。

> 以 `POST` 为例：

- 提交 `Form` 表单数据：

```bash
$ curl -X POST [url] -H "Content-Type: application/x-www-urlencoded" -d "username=张三&age=18"
```

- 提交 `JSON` 格式数据：

```bash
$ curl -X POST [url] -H "Content-Type: application/json" -d'{"username": "张三", "age": 18}'
```

## 把文件内容作为要提交的数据

如果要提交的数据不像前面例子中只有两个 `username: 张三` 键值对，当数据比较多，都写在命令行里很不方便，也容易出错，那么可以把数据内容先写到文件里，
通过 `-d @filename` 的方式来提交数据。这是 `-d` 参数的一种使用方式，所以前面用到 `-d` 参数的地方都可以这样用。

实际上就是把 `-d` 参数值写在命令行里，变成了写在文件里。跟后面的 `multipart/form-data` 中上传文件的 `POST` 方式不是一回事。`@` 符号表明后
面跟的是文件名，要读取这个文件的内容作为 `-d` 的参数。

- 提交 `JSON` 数据

例如，有一个 `JSON` 文件 `data.json` 内容如下：

```json
{
  "username": "张三",
  "age": 18,
  "friends": ["Bob", "Linda"]
}
```

就可以通过如下方式来提交数据：

```bash
$ curl -X POST "localhost:8080/user" -H "Content-Type: application/json" -d @data.json -v
  或
$ curl -X POST "localhost:8080/user" -H "Content-Type: application/json" --data-binary @data.json -v
```

- 提交 `form` 表单数据

通常，我们提交表单数据的请求头都是： `application/x-www-form-urlencoded` 方式提交。以上面的 `JSON` 提交数据为例，将表单数据写入 `txt` 文件
进行提交。

文件名命名为：`data.txt`，上面的 `JSON` 数据写成如下样式（注意数组参数的写法）:

```
username=张三&age=18&friends=Bob&friends=Linda
```

现在就可以使用如下方式进行提交数据：

```bash
curl -XPOST "localhost:8080/user" -H "Content-Type: application/x-www-form-urlencoded" -d @data.txt -v
```

> **注意：** 在示例中使用了 `-v` 选项，该选项作是输出请求的详细信息。以 `form` 请求为例，使用 `-v` 后输出信息如下：
```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> POST /txt HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.64.1
> Accept: */*
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 42
> 
* upload completely sent off: 42 out of 42 bytes
< HTTP/1.1 200 
< Content-Length: 0
< Date: Wed, 13 Nov 2019 07:56:06 GMT
< 
* Connection #0 to host localhost left intact
* Closing connection 0
```
>
> 另外，在使用 `data.txt` 提交 `x-www-urlencoded` 数据时，默认的请求头就是 `Content-Type: application/x-www-form-urlencoded`，
>所以可以不进行指定请求头。


## 文件上传

文件上传请求头信息一般为 `multipart/form-data`，假设现在有一个头像上传需求，`curl` 命令示例如下所示：

```bash
$ curl -X POST "localhost:3000/api/multipart" -F "raw=@raw.data" -F "hello=world"
```

> **注意：** `-F` 选项默认使用请求头为 `multipart/form-data`，所以不需要明确指定。另外，`-F` 指定的键值对的形式，下面要具体说下。

- 用户头像上传

以上传用户头像为例，在上传用户头像是设置请求头 `Token`：

```bash
$ curl -X POST "localhost:8080/user/upload/1006" -H "token: D6919E6F581CE9B5CCEAB5C9B148FB8D" -F "headImg=@wechat.jpg" -v
  或
$ curl -X POST "localhost:8080/user/upload/1006" -H "token: D6919E6F581CE9B5CCEAB5C9B148FB8D" -H"Content-Type: multipart/form-data" -F "headImg=@wechat.jpg" -v
```

加 `-v` 选项后，请求信息如下所示：

```
Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> POST /upload HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.64.1
> Accept: */*
> Content-Length: 124078
> Content-Type: multipart/form-data; boundary=------------------------055839905a1fc177
> Expect: 100-continue
> 
< HTTP/1.1 100 
* We are completely uploaded and fine
< HTTP/1.1 200 
< Content-Length: 0
< Date: Wed, 13 Nov 2019 08:04:14 GMT
< 
* Connection #0 to host localhost left intact
* Closing connection 0
```

**另外，对于 `-F` 选项的键值对需要特别说明！用下面一段 `java` 示例程序进行演示说明：**

```java
@PostMapping("/upload")
public void upload(MultipartHttpServletRequest multipartHttpServletRequest) {
    Iterator<String> iterator = multipartHttpServletRequest.getFileNames();
    while (iterator.hasNext()) {

        String key = iterator.next();
        System.out.println("key: " + key);
        MultipartFile multipartFile = multipartHttpServletRequest.getFile(key);
        assert multipartFile != null;
        String original = multipartFile.getOriginalFilename();

        System.out.println(original);
    }
}
```

这段程序中，输出信息如下：

```
key: headImg
wechat.jpg
```

现在使用 `MultipartFile` 类替换 `MultipartHttpServletRequest`：

```java
@PostMapping("/upload")
public void upload(@RequestParam("/headImg") MultipartFile file) {
    String original = file.getOriginalFilename();

    System.out.println(original);    // ==> wechat.jpg
}
```

这里需要注意！！！！