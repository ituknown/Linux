# 前言

在 Linux 上只要给你分配一个系统账号，就可以直接使用 SSH 客户端进行远程登录。因为 Linux 本身就是命令终端环境，所以使用 SSH 客户端登录系统你不会有任何违和感，给你的感觉就是直接使用 Linux 系统。

以前在使用 Windows 时想要登录 Linux 服务器总是使用类似 XShell 的软件，当然这个软件不是免费的，所以经常为了找破解版而头疼。后来对 Linux 有些了解之后才知道，想要登录服务器只需要使用具有 SSH 功能的客户端软件即可，因为 XShell 就是一个 SSH 客户端。

所以后来就不再使用 XShell 这个软件了，取而代之的使用 Git Bash。因为做软件开发肯定少不了 Git，而在 Windows 上安装 Git 时会提供一个 Git bash 命令终端。这个终端就是一个 SSH 客户端，所以你 Get 到了吗 :)。当然了，现在已经很久不使用 Windows 了~

而 SSH 客户端本质就是 ssh 命令，所以接下来就来看下该命令如何使用。

# SSH 命令登录远程服务器

`ssh` 命令登录 Linux 系统的语法很简单，如下：

```bash
ssh [username@]host[:port]
```

`username` 就是你的 Linux 系统用户名，如果省略该参数就默认使用 `root` 登录。`host` 是机器地址，可以是具体的 IP 也可以是域名。`port` 是 SSH 服务的端口号，默认端口是 22。当然了，默认端口是可以改的。

是不是很简单？现在来演示下：

## SSH 登录示例

我有一个 Debian 发行版系统信息如下：

```
IP: 172.16.110.128
Port: 22
Username: kali
```

想要使用 ssh 登录直接使用下面的命令即可，回车后需要输入用户的密码：

```bash
$ ssh kali@172.16.110.128
```

登录示例：

```bash
$ ssh kali@172.16.110.128
kali@172.16.110.128's password: # 输入密码回车即可
Linux vm 4.19.0-14-amd64 #1 SMP Debian 4.19.171-2 (2021-01-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Dec 11 16:01:08 2021 from 172.16.110.1
```

是不是很简单？如果你是 Windows 用户就可以抛弃你的破解版 XShell 了，可以试着使用 Git Bash 终端来登录了~

## SSH 登录活性问题

在成功登录 Linux 之后，默认情况下如果一段时间不做任何操作会自动退出。这个就是 Session 活性问题，如果想要一直不掉线怎么办呢？只需要配置下 SSH 即可。

**在服务端：**

如果你有 Linux 的超级管理员权限的话，可以直接修改下 sshd 的配置文件 `/etc/ssh/sshd_config` 。在文件中输入如下配置，之后重启 sshd 服务即可：

```bash
ClientAliveInterval 60 #server每隔60秒发送一次请求给client,然后client响应,从而保持连接
ClientAliveCountMax 3  #server发出请求后,客户端没有响应得次数达到3,就自动断开连接.正常情况下, client不会不响应
```

但是如果你没有修改权限就只能在你的客户端配置了：

**在客户端：**

在你的 Home 目录下创建一个 config 配置文件（如果已经有的话就不需要创建了）：

```bash
$ mkdir -p ~/.ssh
$ touch ~/.ssh/config
```

|**注意**|
|:--|
|如果你使用的是 Windows 系统，那你的 Home 目录就是 `C://Users/{username}`。|

之后在 config 中添加如下配置即可：

```bash
Host *
     ServerAliveInterval 60  #client每隔60秒发送一次请求给server, 然后server响应, 从而保持连接
     ServerAliveCountMax 3   #client发出请求后, 服务器端没有响应得次数达到3, 就自动断开连接. 正常情况下, server不会不响应
```

`Host *` 表示对所有的服务器生效，这样就解决登录活性问题了~

# 文件传输

你可能会问，使用 SSH 客户端怎么实现文件传输呢？使用 `scp` 命令即可，语法如下：

```bash
# 文件上传到 Linux
scp [file] [username@]host[:port]:[path]

# Linux 文件下载到本地
scp [username@]host[:port]:[file] [path]
```

可以看到，文件上传和下载命令是一样的，只不是谁在前谁在后的问题。以文件上传为例来说下这个语法：

`file` 就是你本地的文件地址，可以是相对路径也可以是绝对路径。`[username@]host[:port]` 你肯定明白了，就是 SSH 登录服务器的命令。之后有个 `:` 不要忘记，表示 `username` 的 Home 目录，而后面的 `[path]` 就是以 Home 目录为基准的相对路径或绝对路径了~

```
scp [file] [username@]host[:port]:[path]
    [----] [--------------------]|[----]
      |              |           |   |
      |              |           |   +--------> 服务器用户的具体目录, 必须已存在
      |              |           +------------> Home 目录
      |              +------------------------> 服务器地址
      +---------------------------------------> 本地文件
```

来看下演示：

## 文件上传

在我的 tmp 目录下有个 baidu.png 文件，以 [ssh 登录示例](#ssh-登录示例) 中的机器为例，将该文件上传到 kali 用户下的 Downloads 目录中（Downloads 必须已存在）：

```bash
$ scp ./tmp/baidu.png kali@172.16.110.128:Downloads
```

需要特别注意命令中的 `kali@172.16.110.128:Downloads`。`kali@172.16.110.128` 表示使用 kali 用户进行登录，之后的 `:` 表示 kali 的 Home 目录，在之后的 Downloads 表示的是 Home 目录下的 Downloads 目录。当然了这里指定的是相对路径，你也可以指定绝对路径，比如想文件上传到 /opt 目录：

```bash
$ scp ./tmp/baidu.png kali@172.16.110.128:/opt
```

虽然文件上传的目录可以随意指定，但前提是你使用的登录用户要有权限才行，如果没有权限就会下面的信息：

```
Permission denied
```

好了，来看下操作示例：

```bash
$ scp ./tmp/baidu.png kali@172.16.110.128:Downloads
kali@172.16.110.128's password: # 需要输入 kali 的密码
baidu.png                                                                                       100%   13KB   4.5MB/s   00:00
```

现在文件就上传上去了，你可以测试下，之后再使用 SSH 登录服务器看文件是否存在。

## 文件下载

文件下载与文件上传一样，只不过是位置调换而已。比如再将上传的 baidu.png 文件下载到本地的当前目录：

```bash
$ scp kali@172.16.110.128:Downloads/baidu.png ./
```

看下示例：

```bash
$ scp kali@172.16.110.128:Downloads/baidu.png ./
kali@172.16.110.128's password:
baidu.png                                                                                       100%   13KB   7.4MB/s   00:00

$ ls
baidu.png
```

# 禁止 root 远程登录

`root` 用户是系统管理员用户，不受系统任何权限限制，因此直接对外网暴露 `root` 用户是很危险的。为了系统安全我们通常会禁止直接使用 `root` 用户远程登录，仅对外开发权限比较低的用户，当需要执行某些超级管理员操作时再使用 `su` 命令切换到 `root` 用户即可。

禁止 `root` 用户远程登录只需要修改 sshd 服务的配置文件 `/etc/ssh/sshd_config`，将 `PermitRootLogin` 的值设置为 `no` 即可：

```bash
PermitRootLogin no
```

`PermitRootLogin` 意为允许 root 用户登录，可选值有四个：

|**参数类别**|**是否允许ssh登陆**|**登录方式**|
|:---|:---|:--|:---|:---|
|`without-password`|允许|除密码以外的登录方式，如 rsa 认证|Debian 默认值|
|`forced-commands-only`|允许|仅允许使用密钥||
|`yes`|允许|没有限制|RHEL/CentOS默认值|
|`no`|不允许|N/A||

修改之后重启下 `sshd` 服务即可：

```bash
$ sudo systemctl restart sshd
```

# rsa 实现免密登录

有没有发现每次使用 `ssh username@host:port` 命令很累，而且每次都要输入密码。有没有什么解决办法呢？当然有了，使用 rsa 非对称加密即可~

使用 `ssh-keygen` 命令创建一个 rsa 非对称加密文件：

```bash
$ ssh-keygen -t rsa [-f ~/.ssh/custom_file]
```

之后就会在你的 `~/.ssh` 目录下多了一对 rsa 文件，如果不使用 `-f` 参数指定具体的文件名的话默认文件名是 `id_rsa` 和 `id_rsa.pub`：

```bash
$ ls ~/.ssh/
id_rsa  id_rsa.pub
```

接下来需要使用 `ssh-copy-id` 命令将公钥文件 `id_rsa.pub` 发送到 Linux 服务器，该命令的基本语法如下：

```bash
ssh-copy-id [-i identity_file] [-p port] [user@]hostname
```

可选参数说明：

```
-i: 指定 rsa 公钥文件
-p: 登录目标服务器指定端口，默认是 22
```

以 [ssh 登录示例](#ssh-登录示例) 中的机器为例，将公钥文件 `id_rsa.pub` 发送到机器 `172.16.110.128`：

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub kali@172.16.110.128
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
kali@172.16.110.128's password: # 输入密码回车即可

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'kali@172.16.110.128'"
and check to make sure that only the key(s) you wanted were added.
```

现在如果再登录服务器只需要使用下面的命令直接回车即可，不需要输入任何密码了：

```bash
ssh kali@172.16.110.128
```

# 设置服务器别名

虽然现在登录服务器不需要输入密码了，但是每次输入 IP 还是很麻烦。所以我们可以更进一步的使用别名实现 ssh 登录。

起别名好处特别多，比如你有三台服务器：用户服务器、订单服务器、交易服务器。如果直接使用 IP 是不是有点难记？相比较别名而言更容易理解。

怎么配置呢？在你的 `~/.ssh/config` 增加如下配置：

```
Host alias
     Port 22
     User <username>
     HostName 172.16.110.128
     IdentityFile ~/.ssh/id_rsa
     PreferredAuthentications publickey
```

- `alias` 是你为远程主机 `172.16.110.128` 设置的别名。
- `Port` 是你要登录远程主机的端口号，默认就是 22。
- `User` 是你要登录远程主机的用户，默认为 root。
- `HostName` 是你的远程主机 IP。
- `IdentityFile` 是你使用 `ssh-keygen` 命令生成的 rsa 私钥文件，必须与你发送的公钥是一对的。
- `PreferredAuthentications` 指定你登录远程主机认证方法，我们使用的是公钥，指定为 `publickey` ，当然也可以使用密码。

以 [ssh 登录示例](#ssh-登录示例) 中的机器为例，配置如下：

```
Host order_srv
     Port 22
     User kali
     HostName 172.16.110.128
     IdentityFile ~/.ssh/id_rsa
     PreferredAuthentications publickey
```

这里我将服务器起了个别名订单服务：`order_srv`，现在我再登录只需要使用下面的命令即可，是不是很 Nice？

```bash
$ ssh order_srv
```

你以为起别名的好处就这一点？当然不止了。你前面是不是学习了使用 `scp` 实现文件传输？现在有了别名就可以直接将 `[username@]host[:port]` 替换成你设置的别名了。

比如前面的文件上传下载你就可以直接使用下面的命令来代替了（因为前面使用了 `ssh-copy-id` 将公钥上传到服务器，现在文件传输也不需要输入密码了）：

```bash
# 文件上传
$ scp ./tmp/baidu.png order_srv:Downloads

# 文件下载
$ scp order_srv:Downloads/baidu.png ./
```

# rsa 文件设置密码问题

|**注意**|
|:---|
|这里介绍的解决方案只适用于 Mac 和 Linux，如果你使用的是 Windows 系统的话就只能去谷歌度娘了。|

前面 [rsa 实现免密登录](#rsa-实现免密登录) 使用的是没有密码的 rsa 密匙对。但是如果你生成 rsa 文件时指定了密码的话可能会遇到一个问题，虽然使用 `ssh` 登录不要账号的密码的，但是却要 rsa 文件的密码。

比如我使用下面的命令生成 rsa 密匙对，指定的密码是 `1234`，rsa 文件是 `id_pwd`：

```bash
$ ssh-keygen -t rsa -C "PwdTest" -N "1234" -f ~/.ssh/id_pwd
```

按照正常流程操作后给目标服务器起了个别名：`vm.trading`，然后使用 ssh 登录时要求输入 rsa 密匙对的口令密码（密码是前面设置的 `1234`）：

```bash
$ ssh vm.trading
Enter passphrase for key '/home/kali/.ssh/id_pwd':
```

这就有点麻烦了，使用 rsa 密匙对的本质就是为了 “免密”，现在又来了一个 rsa 的密码。怎么解决呢？使用 `ssh-add` 命令来解决。

命令如下（注意 `~/.ssh/id_pwd` 是你的私钥文件名）：

```bash
$ ssh-add ~/.ssh/id_pwd
```

之后会要求你输入 rsa 私钥密码：

```
Enter passphrase for /home/user/.ssh/id_pwd:
```

输入完成后输出如下信息即表示完成了：

```
Identity added: /home/user/.ssh/id_pwd (/home/user/.ssh/id_pwd)
```

之后，直接使用 `ssh vm.trading` 即可完成登录！

但是，如果你在使用 `ssh-add` 命令是遇到了如下错误。原因是 `ssh-add` 与 `ssh` 需要一个环境变量来完成如何与 `ssh` 代理通信。

```
Could not open a connection to your authentication agent.
```

只需要执行下如下命令即可：

```bash
eval "$(ssh-agent)"
```

不过，该命令无法持久，退出登录后下载再次登陆执行 `ssh-add` 命令依然会提示该问题。如果想要一劳永逸，在你的用户目录下配置文件（如 `~/.bash_profile`）中添加如下配置，当然也可以直接在系统配置 `/etc/profile` 中增加如下配置：

```bash
# Add ssh-add to your Bash config file
SSHAGENT=/usr/bin/ssh-agent
SSHAGENTARGS="-s"
if [ -z "$SSH_AUTH_SOCK" -a -x "$SSHAGENT" ]; then
    eval `$SSHAGENT $SSHAGENTARGS`
    trap "kill $SSH_AGENT_PID" 0
fi
```

接着重试一下上面 `ssh-add ~/.ssh/id_pwd` 命令就好了~


# 常见问题

有时候，我们使用 `ssh-copy-id` 命令将 rsa 公钥已经发送到了远程服务器，但是依然遇到无法直接 `ssh username@host` 登录问题。

这个可能是服务器配置的问题，你需要登录到目标服务器，使用具有超级管理员权限的用户或 `root` 查看 `/etc/ssh/sshd_config` 文件的配置了，看下下面两个配置的值是否如下：

```bash
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```