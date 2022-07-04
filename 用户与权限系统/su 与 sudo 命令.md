# su 用户身份切换命令

`su` 命令主要用于切换当前登录用户的身份，比如当前 Linux 系统上有两个用户：`ituknown` 和 `kaka`。在登录系统时你使用的是 `ituknown` 用户，当你登录成功之后如果想要切换到 `kaka` 这个用户该怎么办呢？

通常的做法是先登出（`logout`），然后再使用 `kaka` 这个用户登录即可！但是呢，Linux 系统了一个更方便的命令 `su`，可以实现在不登出的情况下随意切换当前用户身份！

现在看下 `su` 命令的常用语法参数：

```bash
su [- [USER] | -l USER] [-c COMMAND]


说明:

- : 如果单纯的使用 - 不加任何用户表示切换到 root 用户身份. 如果后面加了 USER 表示切换到指定用户.
-l : 该参数与 - 含义相同, 唯一不同的是 -l 后面必须跟要切换的用户
-c : 以交互的方式执行单条命令
```

`su` 命令其实与 `/etc/passwd` 文件有关。我们都知道，当一个用户登录成功后通常都会有一个默认 SHELL（如果在创建用户时指定的话），比如我当前的 `ituknown` 用户，该用户的默认 SHELL 就是 `/bin/bash`：

```bash
$ id
uid=1000(ituknown) gid=1002(ituknown) groups=1002(ituknown),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev),113(bluetooth),119(scanner),998(admin)

$ echo $SHELL
/bin/bash
```

其实该用户的默认 SHELL 就配置在 `/etc/passwd` 文件中：

```bash
$ sudo cat /etc/passwd | grep ituknown
ituknown:x:1000:1002:ituknown:/home/ituknown:/bin/bash
```

当然了，默认 SHELL 是可以修改的，可以参考下 [系统用户管理#登录hell](系统用户管理.md#登录-shell)。

之说以这里会说下默认 SHELL 是因为当我们使用 `su` 命令切换用户身份时其实会重新加载要切换用户的 SHELL 环境。这个其实很好理解，正常来说，我登录系统的用户是 `ituknown`，那么我的 HOME 环境变量就是 `/home/ituknown`。结果当我切换到 root 用户后环境变量就变了：

```bash
$ echo $HOME # 当前用户的 HONE 环境变量
/home/ituknown

$ su - # 切换到 root
Password:

# echo $HOME # 在看下 HOME 环境变量
/root
```

所以，`su` 命令会重新加载指定用户的 SHELL 环境！

`su` 命令使用起来很简单，到这里基本上该说得都说了，来看下简单的示例：

**切换到 root 用户：**

```bash
su -
# 或
su - root
# 或
su -l root
```

**切换到指定用户（如 kaka）：**

```bash
su - kaka
# 或
su -l kaka
```

**使用交互的方式执行单条命令：**

```bash
su -c "ls"
```

`-c` 参数其实特别有意思，可以以指定的用户来执行单条命令。比如在 Debian 发行版的 Linux 系统上，想要执行 docker 命令其实需要超级管理员权限：

```bash
$ docker image ls
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/json": dial unix /var/run/docker.sock: connect: permission denied
```

但是呢，我可以借助 root 用户身份来执行该命令：

```bash
$ su - -c "docker image ls"
Password:
REPOSITORY           TAG              IMAGE ID       CREATED         SIZE
whyour/qinglong      latest           72ca611211dc   3 weeks ago     455MB
ffmpeg               latest           c87d5eb54dcf   3 weeks ago     380MB
yt-dlp               latest           5e79839139af   3 weeks ago     150MB
ubuntu               latest           825d55fb6340   2 months ago    72.8MB
gitlab/gitlab-ce     14.7.5-ce.0      a64261d15ccb   3 months ago    2.39GB
mysql                5.7              11d8667108c2   3 months ago    450MB
openjdk              8-jdk-buster     e512716a5569   4 months ago    514MB
```

是不是很 nice？