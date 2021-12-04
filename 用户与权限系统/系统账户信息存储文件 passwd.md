# 前言

Linux 系统文件 `/etc/passwd` 存储的是系统用户的数据信息，因为这个文件名与系统用于管理用户密码的命令 `passwd` 同名，所以开始我理所当然的以为这个文件存储的就是用户的密码相关信息。其实不是的，它真正存储的是用户的账户信息。

事实上在以前的 Linux 中，`/etc/passwd` 中确实存储用户的密码信息。只不过现代 Linux 系统为了安全性，将密码信息全部移动到了 `/etc/shadow` 文件，在 `/etc/passwd` 文件中虽然还保留着密码位，但使用的是 x 字符标识。这就是为什么这个文件名叫 `passwd` 的原因。


|**Note**|
|:-------|
|`/etc/passwd` 文件与系统用户有关，所以千万不要使用 `root` 用户或具有超级管理员权限的用户直接编辑该文件数据。否者可能某些用户不可用的问题，得不偿失。|


# 数据格式

虽然不能直接编辑该文件，但是可以直接使用 cat 等命令查看该文件中的内容，因为该文件的权限是 644（`rw-r--r--`）：

```bash
$ ls -l /etc/passwd
-rw-r--r-- 1 root root 1914 Nov 29 14:31 /etc/passwd
```

直接在屏幕上使用 cat 命令打印 `/etc/passwd` 文件内容的话可能会感觉是一坨杂乱无章的数据。但其实该文件中每一行就是一个用户的账户信息，并且每行的数据格式也是固定的。

数据格式如下（`:` 符号是分隔符）：


```
kali:x:1000:1000:kali,,,:/home/kali:/bin/bash
[--] - [--] [--] [-----] [--------] [-------]
|    |  |    |      |         |         |
|    |  |    |      |         |         +-------> 7. Shell
|    |  |    |      |         +-----------------> 6. Home Directory
|    |  |    |      +---------------------------> 5. Username comment
|    |  |    +----------------------------------> 4. GID
|    |  +---------------------------------------> 3. UID
|    +------------------------------------------> 2. Password
+-----------------------------------------------> 1. Username
```


每一行一共有六个 `:` 分隔符，七个数据。从左到右分别是：用户名（`Username`）、密码（`Passwd`）、用户ID（`UID`）、用户组ID（`GID`）、用户的注释信息（`Username comment`）、用户Home目录（`Home Directory`）以及操作 Shell。

|**Note**|
|:-----|
|下面会做相应的演示说明，特别注意的是。下面的命令需要使用超级管理员用户 `root` 或具有超级管理员角色的用户才能执行！|


# 用户名

这个就是我们创建的系统用户的用户名，用于系统登录的用户。这个用户名在系统中是唯一的，用户名的最大长度通常不能超过32个字符。

下面是使用 `useradd` 命令新增一个用户的示例，我们创建的新用户的用户名是：`exampleuser`，示例：

**1. 先使用 id 命令查看用户 `exampleuser` 在系统中是否存在，如果存在的话先删除：**

```bash
# id exampleuser
id: ‘exampleuser’: no such user <== 用户不存在, 如果存在的话需要使用 userdel 命令删除
```

**2. 创建 `exampleuser` 用户：**

```bash
# useradd exampleuser
```

**3. 查看 `/etc/passwd` 文件信息：**

```bash
# grep exampleuser /etc/passwd
exampleuser:x:1003:1003::/home/exampleuser:/bin/sh
[---------]
     |
     +------------------------------> 我们新建的用户
```

|**Note**|
|:-------|
|之后的示例中会省略 `grep [username] /etc/passwd` 命令|


# 密码

这个原本存储的是用户的密码加密后的字符串，只不过现在已经将密码移到 `/etc/shadow` 文件中了。所以这里你通常会看到一个 `x` 字符，表示密码在 `/etc/shadow` 文件中。

以 [用户名](#用户名) 中创建的 `exampleuser` 用户为例：

```
exampleuser:x:1003:1003::/home/exampleuser:/bin/sh
            -
            |
            +-------------------------> 密码数据, 被 x 标识, 说明密码存储在 /etc/shadow 文件中
```

# 用户ID

这个就是用户的唯一ID，在 POSIX 标准中，要求 UID 为整数类型，而大多数类 UNIX 操作系统都采用无符号整形数值，这也是为什么我们会看到 UID 是数值的原因。

另外，一些 UNIX 系统 UID 采用 15位整形数值，也就是说最大允许创建的 UID 为 32767。而在 Linux v2.4 内核之前的发行版采用的都是 16 位整形数值，也就是说最大允许分配的 UID 为 65536。

但是呢，科技在进步，现在的操作系统架构也在不断的演进。以前系统都是 32 位，现在基本上都是 64 位架构操作系统（x86_64 架构）。而在 64位架构的现在操作系统，整形数据结构在内存中基本上都占用 32 位，所以现在UID最大允许分配的数值是：4,294,967,296 (2^32)。

当然这是都是理论上的，实际上不同的 Linux 发行版都会对最大允许分配的 UID 做一定的限制。这些限制被定义在 `/etc/login.defs` 配置文件中，比如查看 UID 的限制：

```bash
$ grep UID /etc/login.defs
UID_MIN			 1000    <== 允许创建的UID的最小值
UID_MAX			60000    <== 允许创建的UID的最大值
#SYS_UID_MIN		  100    <== 系统用户UID允许最小值
#SYS_UID_MAX		  999    <== 系统用户UID允许最大值
```

--

关于 UID 的取值大家需要了解下：

**超级管理员用户 root 的 UID 是 0**，这也表示 root 用户是系统的最高权限：

```
root:x:0:0:root:/root:/bin/bash
       -
       |
       +--------------------------> root 的 UID
```

在 Linux 标准中，1 ~ 99 范围的 UID 是由系统静态分配，100 ~ 1000 则是保留给超级管理员分配以及用于安装脚本自动分配，自 1000 开始的 UID 才是可用于我们创建的普通用户使用的 UID。

简单点，就记住 1000 以下的 UID 是用于系统使用，1000 以上才是用于创建普通用户使用的 UID，关于这点会在 `useradd` 命令中进行说明。

比如我们刚刚创建的 `exampleuser` 用户，它的 UID 为 1003：

```
exampleuser:x:1003:1003::/home/exampleuser:/bin/sh
              [--]
               |
               +-----------------------------> UID
```

再比如我们重新使用 `useradd` 命令创建一个系统用户 `examplesysuser`：

```bash
# useradd -r examplesysuser

# grep examplesysuser /etc/passwd
examplesysuser:x:998:997::/home/examplesysuser:/bin/sh
                 [-]
                  |
                  +--------------------> 此时的UID小于 1000
```


# 用户组ID

与用户一样，用户组也有自己的ID，被称为 GID。而这个 GID 的分配策略与 UID 也是一样的。**0 是 root 用户组**，1000 以下用于系统超级管理员分配以及安装脚本自动分配，1000 以上才是供我们创建组时分配的 GID。

与 UID 一样，不同的 Linux 发行版对 GID 也有自己的限制：

```bash
$ grep GID /etc/login.defs
GID_MIN			 1000
GID_MAX			60000
#SYS_GID_MIN		  100
#SYS_GID_MAX		  999
```

另外，很重要的一点一定要记住。**`/etc/passwd` 文件中的用户 GID 是该用户的主要组ID（Primary GID）**，会在之后进行说明！！！！

使用 `useradd` 命令创建用户时，默认会创建对应的用户组，用户组与用户名同名。

比如超级管理员 `root` 用户的 GID：

```
root:x:0:0:root:/root:/bin/bash
         -
         |
         +--------------------> root 的 GID 为 0
```

比如之前我们创建的普通用户 `exampleuser`：

```
exampleuser:x:1003:1003::/home/exampleuser:/bin/sh
                   [--]
                    |
                    +----------------------> GID 为 1003
```

再比如刚刚创建的系统用户 `examplesysuser`：

```
examplesysuser:x:998:997::/home/examplesysuser:/bin/sh
                     [-]
                      |
                      +-------------------> GID小于 1000
```



# 用户的注释信息

这个就是创建用户时设置的简单注释信息，可省略。我们刚刚创建的 `exampleuser` 就没有注释信息：

```
exampleuser:x:1003:1003::/home/exampleuser:/bin/sh
                       |
                       |
                       +-----------------> 无注释信息
```

现在来使用 `usermod` 命令添加下注释：

```bash
# usermod -c "Linux Example User" exampleuser
```

再来看下：

```
exampleuser:x:1003:1003:Linux Example User:/home/exampleuser:/bin/sh
                        [----------------]
                                |
                                +--------------------> 刚添加的注释信息
```


# 用户Home目录

这个是用户的跟目录，除了超级管理员用户 `root` 外。其他用户的默认根目录都是在 /Home 目录下，比如刚刚前面创建的 `exampleuser` 用户，它对应的 Home 目录就为 `/home/exampleuser`：

```
exampleuser:x:1003:1003::/home/exampleuser:/bin/sh
                         [----------------]
                                   |
                                   +----------------> Home 目录
```

如果在创建用户时不进行明确指定，它的根目录名称与用户名一致，这也是为什么我们会看到 `exampleuser` 用户对应的根目录是 `/home/exampleuser` 而不是 `/home/example` 的原因，这个也可以在创建用户时明确进行指定。

另外创建的新用户也可以指定不创建对应的 Home 目录。示例：

**创建不带 Home 目录的 exampleNoHome 用户**

```bash
# useradd -M exampleNoHome
```

**查看 /etc/passwd 文件中该用户的信息：**

```
exampleNoHome:x:1003:1003::/home/exampleNoHome:/bin/sh
```

可以看到虽然指定了不创建 Home 目录，这里依然显示有目录，这是什么原因呢？切换到该用户进入 Home 目录试试：

**切换到 exampleNoHome 用户：**

```bash
# su exampleNoHome

$ cd ~
sh: 3: cd: can't cd to /home/exampleNoHome   <=== 警告信息
```

当我们切换到 exampleNoHome 用户后尝试进入 Home 目录时提示 Home 目录不存在！


# 操作 Shell

这个就是我们可以使用的 Shell 了，比如上面的 `exampleuser` 用户对应的 Shell 是 `/bin/sh`：

```
exampleuser:x:1003:1003::/home/exampleuser:/bin/sh
                                           [-----]
                                              |
                                              +-------------> 操作 Shell
```

这就是该用户的默认 Shell，这也是为什么切换到该用户能使用 Shell 命令的原因。另外，当我们查看用户的默认 Shell 时你就会看到与 `/etc/passwd` 文件中的 Shell 是一致的：

```bash
$ echo $SHELL
/bin/sh
```

有时你会看到某个用户在 `/etc/passwd` 文件中的 Shell 是 nologin。那么说明该用户是无法登录系统的，**甚至也连 root 用户也无法使用 `su` 命令切换到该用户**，这点很重要！

下面来演示下，创建一个名为 `examplenologin` 的用户，同时指定该用户的 Shell 为 `nologin`：

```bash
# useradd -s /usr/sbin/nologin examplenologin
```

看下 `/etc/passwd` 文件中该用户的 Shell 信息：

```
examplenologin:x:1001:1001::/home/examplenologin:/usr/sbin/nologin
                                                 [---------------]
                                                         |
                                                         +-------> nologin
```

之后使用 `chpasswd` 命令设置一个简单的密码 123456，用于测试登录：

```bash
# echo "examplenologin:123456" | chpasswd
```

现在使用 ssh 登录服务器：

```bash
$ ssh examplenologin@172.17.21.163
examplenologin@172.17.21.163's password:
Linux vm 4.19.0-18-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Nov 30 14:18:46 2021 from 172.17.13.43
Could not chdir to home directory /home/examplenologin: No such file or directory
This account is currently not available.         <=== 注意这里
Connection to 172.17.21.163 closed.              <=== 注意这里
```

当你尝试登录时你会发现提示你：`This account is currently not available`！之后就关闭连接了。

现在再测试 `root` 用户使用 `su` 命令切换到该用户：

```bash
# su -l examplenologin
su: warning: cannot change directory to /home/examplenologin: No such file or directory
This account is currently not available.  <=== 同样的提示
```

nologin Shell 用户无法登录也无法使用 `su` 切换，那么该用户存在的意义是什么呢？因为 Linux 系统中有大量的这类用户，你可以使用下面的命令看下这类用户的信息：

```bash
$ grep nologin /etc/passwd
```

事实上，这类用户通常是系统分配仅用于执行少量命令的用户，权限特别低，比如用于邮件服务器。所以划重点，**nologin 用户无法使用用户名登录系统，且无法使用 `su` 命令进行切换**！

另外，除了 nologin 之外，有可能还会看到操作 Shell 是 /bin/false。有关这个的话就不做解释了。在 stackexchange 也有一个关于这两个 Shell 区别问题：[What's the difference between /sbin/nologin and /bin/false](https://unix.stackexchange.com/questions/10852/whats-the-difference-between-sbin-nologin-and-bin-false)，可以自行参考下。


--

参考：https://en.wikipedia.org/wiki/User_identifier

完结，撒花🎉🎉🎉~