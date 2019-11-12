# 前言

在 Linux 下，如果想运行管理应用程序（`administrative application`）可以使用 `su ("switch user")` 命令切换到超级管理员用户 （`root`）。或则也可以直接使用 `sudo` 命令执行，该命令意思是使用超级管理员用户执行该命令。

至于何时使用该命令，可以通过这个提示进行判断。当执行一个命令时提示以下错误时即可使用 `su` 或 `sudo` 命令进行执行：

```
access denied
```

或

```
operation requests superuser privilege
```

但是，如果您选择使用su命令，那么您将把整个用户切换到根用户，这意味着即使在第一个命令之后，后续的每个命令也使用根凭据运行。

这虽然使命令变得非常容易，但是如果不小心，这些命令可能会造成很大的损害。


# su命令

`su` 命令意思是切换至某个用户。同 `windows` 系统下 `切换用户` 操作。因此，如果切换的用户为 `root` 用户，但是误操作了就会导致无法修复的损失。需要注意！！！！

下面就来试一下该命令，首先我们先创建创建一个组 `temp`：

先执行以下命令查看是否已存在该组，如果没有的话就创建创建一个（组是随意的，这里只是测试）：

```
# more /etc/group | grep temp
```

执行以下命令进行创建 `temp` 组：

```
# groupadd temp
```

现在再次执行 `more /etc/group | grep temp` 命令查看是否创建成功：

```
#  more /etc/group | grep temp
utempter:x:35:
temp:x:1000:
```

可以看到已经成功创建了该组。

现在在来在该组下创建一个用户 `user`。

```
# useradd -d /home/user -g temp -G temp user
```

> **useradd** 命令与 **groupadd** 命令该篇幅不做说明。


执行该命后看是否在 `/home` 下成功创建 `/user` 目录。成功创建即表示成功创建用户 `user`

```
# cd /home/user/
[root@localhost user]# 
```

当然，也可以直接通过查看 `/etc/passwd` 中是否有该用户密码信息进行验证：

```
# more /etc/passwd | grep user
user:x:1000:1000::/home/user:/bin/bash
```

现在再来修改 `user` 用户密码。使用 `root` 用户执行 `passwd user`，按提示输入两次密码进行修改即可：

```
# passwd user
更改用户 user 的密码 。
新的 密码：
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```

现在即可执行命令 `su - user` 进行切换用户了。

总而言之，`su` 命令主要用于用户切换。

# su 命令详解

`su` 命令有一下可选参数：

```
# su --help

用法：
 su [选项] [-] [USER [参数]...]

将有效用户 id 和组 id 更改为 USER 的 id。
单个 - 视为 -l。如果未指定 USER，将假定为 root。

选项：
 -m, -p, --preserve-environment  不重置环境变量
 -g, --group <组>             指定主组
 -G, --supp-group <组>        指定一个辅助组

 -, -l, --login                  使 shell 成为登录 shell
 -c, --command <命令>            使用 -c 向 shell 传递一条命令
 --session-command <命令>        使用 -c 向 shell 传递一条命令
                                 而不创建新会话
 -f, --fast                      向shell 传递 -f 选项(csh 或 tcsh)
 -s, --shell <shell>             若 /etc/shells 允许，则运行 shell

 -h, --help     显示此帮助并退出
 -V, --version  输出版本信息并退出
```

比如，当前在 `root` 用户下，现在想要切换至 `user` 用户，执行以下命令：

```
# su - user
上一次登录：二 12月  4 13:46:14 CST 2018pts/1 上
$ pwd
/home/user
$ 
```

成功切换至 `user` 用户。现在如果再想切换至 `root` 用户：

```
$ su - root
密码：
su: 鉴定故障
$ 
```

提示这个错误，只需要输入 `exit` 退出 `user` 用户后进行查找 `sudoers` 进行修改权限：

```
# find / -name sudoers

/etc/sudoers
/usr/share/doc/sudo-1.8.19p2/examples/sudoers
```

可以看到路径为 `/etc/sudoers`

```
sudo vi /etc/sudoers
```

在如下出将 `user` 用户加上：

```
## Allow root to run any commands anywhere 
root	ALL=(ALL) 	ALL

## 加上 user 用户：
user    ALL=(ALL)       ALL
```

在正常保存是会提示：

```
E45: 'readonly' option is set (add ! to override)
```

因此需要输入 `:wq!` 强制保存退出！

现在再输入切换用户命令：

```
sudo su - root
```

接着输入 `user` 用户密码即可！其中 `-` 表示切换到用户主目录，比如 `user` 用户主目录为 `/home/user`。

另外，如果没有在 `sudoers` 文件中增加 `user` 用户权限在切换用户时如果不使用 `sudo` 则会提示鉴权故障：

```
$ su - root
密码：
su: 鉴定故障
```

使用 `sudo` 则提示：

```
$ sudo su root
user 不在 sudoers 文件中。此事将被报告。
```

因此在创建用户时建议在 `sudoers` 中增加鉴定权限！
