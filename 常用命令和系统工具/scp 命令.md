# 前言

`scp` 建立在 SSH 协议之上，主要用于将文件从一台机器安全地传输到另一台机器。该命令的本质是远程复制，它通过 SSH 连接在两台计算机之间传输文件。

`scp` 命令基本语法非常简单直观，主要有两种应用场景：文件上传和下载。

# 文件上传

## 上传文件到远程服务器

```bash
scp <本地文件路径> <用户名>@<远程主机IP或域名>:<远程目录路径>
```

例如，如果想把本地 `~/documents/report.pdf` 文件上传到 IP 地址为 `192.168.1.100` 的服务器上，并放到 `/home/user/reports/` 目录下，使用的用户名是 `remoteuser`，那么命令就是：

```bash
scp ~/documents/report.pdf remoteuser@192.168.1.100:/home/user/reports/
```

当执行这个命令时，系统会提示输入 `remoteuser` 的密码。输入正确的密码后，文件就会开始上传。

## 上传目录到远程服务器

```bash
scp -r <本地目录路径> <用户名>@<远程主机IP或域名>:<远程目录路径>
```

例如有个 `project_alpha` 文件夹，里面包含了很多文件，你想把整个文件夹上传到服务器的 `/var/www/` 目录下，那么只需要加上 `-r` 参数即可，表示递归复制（也就是复制目录及其内容）：

```bash
scp -r ~/projects/project_alpha remoteuser@192.168.1.100:/var/www/
```

同样地，输入正确的密码后，文件就会开始上传。

# 文件下载

下载与上传命令基本一致，只不过调换一下顺序而已：

## 远程服务器下载文件到本地

```bash
scp <用户名>@<远程主机IP或域名>:<远程文件路径> <本地目录路径>
```

## 远程服务器下载目录到本地

```bash
scp -r <用户名>@<远程主机IP或域名>:<远程目录路径> <本地目录路径>
```

# 其他实用选项

## 指定端口

SSH 服务器默认使用的是 22 端口，如果服务器使用的是其他端口可以使用 `-P` 选项指定：

```bash
scp -P 2222 ~/documents/report.pdf remoteuser@192.168.1.100:/home/user/reports/
```

## 启用压缩

对于较大的文件，使用 `-C` 选项可以启用数据压缩，在传输过程中节省带宽，尤其在网络条件不佳时能提高传输速度：

```bash
scp -C ~/large_file.txt remoteuser@192.168.1.100:/tmp/
```

## 输出详细模式

如果你想了解 `scp` 命令的执行细节，比如连接过程、认证信息等，可以使用 `-v` 参数来启用详细模式，尤其是排查问题的时候，非常有用：

```bash
scp -v ~/documents/report.pdf remoteuser@192.168.1.100:/home/user/reports/
```

## 保留文件属性

当你希望上传的文件在远程服务器上保留与本地文件相同的修改时间、访问时间以及权限等属性时，可以使用 `-p` 选项：

```
scp -p ~/source_file.txt remoteuser@192.168.1.100:/destination/
```

## 使用 SSH 免密认证

如果你的机器已经跟远程服务器做过 SSH 密钥对认证，在传输文件时就会更加方便快捷（因为可以直接免密传输）。

如果还没有认证过，你需要先在本地生成 SSH 密钥，并将公钥上传到远程服务器的 `~/.ssh/authorized_keys` 文件中。具体可以参考 [SSH Key 实现免密登录](../用户与权限系统/ssh%20远程登录及免密登录.md) 和 [设置服务器别名](../用户与权限系统/ssh%20远程登录及免密登录.md)。

比如服务器设置的别名是 example_server，文件就简单很多：

- 文件上传

```bash
scp <本地目录路径> example_server:<远程目录路径>
```

- 文件下载

```bash
scp -r example_server:<远程目录路径> <本地目录路径>
```

## 使用私钥文件进行身份认证

除了前面说的配置别名之外，你也可以直接使用 `-i` 参数指定本地私钥文件来实现免密的 SSH 密钥对认证。

`-i` 选项告诉 scp 不要尝试使用密码或者默认的私钥文件（通常是 ~/.ssh/id_rsa 或 ~/.ssh/id_dsa），而是去使用你指定路径的私钥文件进行身份验证：

```bash
scp -i <私钥文件路径> <本地文件路径> <用户名>@<远程主机IP或域名>:<远程目录路径>
```

比如私钥文件存放在 `~/.ssh/my_custom_key`，你想用它来上传 `~/documents/report.pdf` 到 `192.168.1.100` 服务器：

```bash
scp -i ~/.ssh/my_custom_key ~/documents/report.pdf remoteuser@192.168.1.100:/home/user/reports/
```
