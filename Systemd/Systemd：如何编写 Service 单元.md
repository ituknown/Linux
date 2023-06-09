# 前言

现代Linux发行版都抛弃了传统的 initd 转而使用 systemd 作为系统和服务管理器。这套组件可以管理服务器的方方面面，从服务到挂载设备以及系统状态。

比如，当我们使用 ssh 连接远程服务器时，实际上使用的就是 sshd 服务。如果远程服务器没有开启 sshd 服务，那么客户端就永远无法连接。新手在使用 ssh 连接远程服务器时经常会遇到无法正常登录问题。然后 Google 这个 issue 的时候通常会发现，各大论坛首先会告诉你优先使用 `systemctl` 命令查看一下服务器 sshd 服务有没有正常运行：

```bash
$ systemctl status sshd.service
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-06-06 10:03:25 CST; 3 days ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 708 (sshd)
      Tasks: 1 (limit: 948)
     Memory: 6.2M
        CPU: 346ms
     CGroup: /system.slice/ssh.service
             └─708 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```

而 systemctl 实际就是 systemd 组件中的主要系统命令：用于管理和控制服务。

不过，systemd 其实并不知道 sshd 服务。而是通过提供一套标准化的配置管理服务，这套标准化的配置被称为：单元文件。系统中的服务如何操作和管理资源都被定义在这个单元文件中，这是 systemd 知道如何管理服务的基本原理。

比如上面的 sshd 服务，他的单元文件位置在 /lib/systemd/system/ssh.service，我们可以使用 cat 命令看下这个单元文件中定义的内容。如下：

```shell
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run

[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
Alias=sshd.service
```

因此，如果你想使用 systemd 管理你自定义的服务，编写一个单元文件即可！

# Systemd 单元文件位置

定义systemd服务的单元文件不能随意放置，必须编写在指定目录下才会生效。并且，不同位置下的单元文件具有不同的含义。另外单元文件加载的优先级也与位置有关。

系统单元文件的通常保存在 `/usr/lib/systemd/system`目录。当使用包管理等工具安装软件时，它的systemd单元文件（如果有的话）默认会放到该位置，因此可以直接理解为它是操作系统意义上的单元文件目录。

譬如，当我们直接使用Linux发行版系统的包管理工具安装 mysql 服务，当安装完成之后就能看到它的单元就在该目录中：

```bash
$ sudo apt install -y mysql-server-8.0

$ systemctl status mysql.service
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2023-06-09 17:41:50 CST; 17s ago
    Process: 8826 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
   Main PID: 8834 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 948)
     Memory: 362.8M
        CPU: 915ms
     CGroup: /system.slice/mysql.service
             └─8834 /usr/sbin/mysqld

Jun 09 17:41:49 ubuntu systemd[1]: Starting MySQL Community Server...
Jun 09 17:41:50 ubuntu systemd[1]: Started MySQL Community Server.
```

因为存储在该目录的单元文件，通常是由软件开发者编写，因此在实际使用中应该严格禁止直接编辑此目录中的单元文件。如果你想修改某个单元文件的运作方式，应该选择 `/etc/systemd/system` 目录 ，该目录中的单元文件优先级高于系统其他位置。因此，你只需要将你编写的单元文件放到该位置，以此来达到覆盖默认单元文件配置的目的。

另外，systemd 还允许你覆盖单元文件中的特定指令。方法是在同级目录中创建一个以单元文件命名并以 `.d` 结尾的目录。例如，在  `/etc/systemd/system` 目录中有一个名为 `example.service` 的单元，可以创建名为 `example.service.d` 的子目录。在此目录中，创建一个或多个 `.conf` 文件。写在该文件中的指令可用于覆盖或扩展系统单元文件的属性。

除了上诉两个位置，实际上还有一个位置：`/run/systemd/system`。该位置是一个运行态的单元文件目录，完全由操作系统管理。这个位置的文件比前一个位置的重量更轻，但比后者更重。特别强调一点是：系统每次重启时，此目录中所有的单元文件都会被清除。

**Note：** 老鸟可能会发现还有一个名为 `/lib/systemd/system` 的目录。不过 `/lib` 实际上是 `/usr/lib` 的一个软链接：


```bash
$ ls -l /lib
lrwxrwxrwx 1 root root 7 Apr 21  2022 /lib -> usr/lib
```

# 单元文件类型

Systemd根据它们描述的资源类型对单位文件进行分类。确定单元文件类型的最简单方法就是看文件后缀，下面是systemd的部分单元文件类型：

| **单元类型** | **说明**                                                     |
| :----------- | :----------------------------------------------------------- |
| .service     | <span style="color: rgb(9, 105, 218);font-weight: bold;">服务单元</span>，配置服务或应用程序在服务器上如何管理。包括如何服务的启动与停止，在什么情况下应该自动启动服务，以及相关软件的依赖性和关系信息。这是我们最主要了解的一个单元类型，如 sshd 服务、mysql 服务都是该单元类型。 |
| .target      | .target 单元用于在启动或更改状态时为其他单元提供同步点，也可以用来将系统带入一个新的状态。.target 单元除了提供的通用功能不提供任何附加功能，最常用的方式是作为依赖项用作引导目标。如 multi-user.target、graphical.target。 |
| .socket      | 套接字单元，用于配置网络或IPC套接字。一个程序如果有 .socket 单元，必定会有一个与之关联的 .service 文件。如配置 .service 服务的 socket 套接字监听端口。 |

上面列举的单元类型是我们在实际工作中接触比较最多的三个。实际上，systemd 提供了很多的单元类型，不再赘述。具体可以使用 `man` 命令查看：

```bash
$ man systemd.
systemd.automount              systemd.journal-fields         systemd.offline-updates        systemd.socket
systemd.device                 systemd.kill                   systemd.path                   systemd.special
systemd.directives             systemd.link                   systemd.positive               systemd.swap
systemd.dnssd                  systemd.mount                  systemd.preset                 systemd.syntax
systemd.environment-generator  systemd.negative               systemd.resource-control       systemd.target
systemd.exec                   systemd.netdev                 systemd.scope                  systemd.time
systemd.generator              systemd.net-naming-scheme      systemd.service                systemd.timer
systemd.index                  systemd.network                systemd.slice                  systemd.unit
```

# 单元文件规范

单元文件的内部结构由各个部分（Section）组成。Section 的定义使用方括号“[SectionName]”表示，每个 Section 都延伸到后续 Section 的开始位置。

特别注意的是，Section 的名字严格区分大小写，如 `[Unit]` 不能写成 `[UNIT]`。如果你需要添加自定义的Section（非 Systemd 定义的标准 Section），需要使用 `X-` 作为 Section 的前缀，如 `[X-UNIX]`。

在每个 Section 中，通过使用键值对格式的简单指令来定义单元行为和元数据。如下：

```shell
[Section]
Directive1=value
Directive2=value

# 自定义Section
[X-Section]
Directive3=value
Directive4=value
```

如果需要在 `.d` 目录下的 `.conf` 配置文件中覆盖或扩展单元文件的属性，可以通过将指令分配给空字符串来重置指令。例如，需要删除 Directive1 指令，同时更新 Directive2 指令可以这样设置：

```shell
# 删除Directive1
Directive1=
# 修改Directive2
Directive2=new_value
```

一般来说，systemd允许简单灵活的配置。例如，接受布尔表达式“真”可以使用 1、yes、on、true 来表示。“假”可以使用0、no、off、false 来表示。

Systemd 单元文件通常由三个 Section 组成。常见的配置项通常配置在 `[Unit]` 和 `[Install]` 中。不过，如果是 .service 类型单元需要特别的配置在 `[Service]` 中。

接下来就分别对这三个 Section 以及内部指令做个说明：

## [Unit] Section指令

大多数单元文件中的第一部分是`[Unit]`，通常用于定义单元的元数据以及单元与其他单元的关系。虽然在解析单元文件时，Section 定义的顺序对systemd无关紧要，但 `[Unit]` 通常放在单元文件的顶部，因为它提供了单元的概述。`[Unit]` 常见的指令有如下几个：

| **指令**         | **说明**                                                     |
| :--------------- | :----------------------------------------------------------- |
| Description      | 对单元的描述，可用于描述该单元的名称和基本功能。因此描述信息最好简短、直接、具体及翔实。 |
| Documentation    | 对单元（或服务）的文档说明。可以是 `man` 手册，也可以是一个HTTP链接。 |
| Requires         | 依赖的单元（多个使用空格分隔）。如果当前单元被启动，此处列出的单元也必须全部被成功启动，否则该单元将启动失败。默认情况下，这些单元与当前单元并行启动。 |
| Wants            | 与Requires一样，不过并不强制要求所依赖的单元被成功启动。这是配置大多数依赖关系的推荐方法。 |
| Conflicts        | 用于列出无法与当前单元同时运行的单元，启动该单元将导致其他单元停止。 |
| Before，After    | 单元启动的顺序。                                             |
| Condition        | 有许多指令以条件开头，允许管理员在启动单元之前测试某些条件。这可用于提供通用单元文件，该单元仅在适当的系统上运行。如果不符合条件，则会优雅地跳过该单元。 |
| StopWhenUnneeded | 布尔值，为true则表示如果当前单元长时间不被使用则会自动停止。 |

需要特别强调的一点是，这里仅列出常用的配置。实际上配置指令特别多，具体可以使用 `man` 指令查看。接下来看几个简单的示例，这几个示例都来源于系统服务。

bluetooth.target 蓝牙单元 [Unit] 配置：

```shell
[Unit]
Description=Bluetooth Support
Documentation=man:systemd.special(7)
StopWhenUnneeded=yes
```

不同的机器上的蓝牙配置也不近相同，不过大多数都会有这三个配置。通过蓝牙单元文件可以了解到 Documentation 指令的用法，可以是 `man` 手册也可以是 URL 链接。比如如果想要具体知道该单元信息就可以通过 `man systemd.special` 命令查看。

nginx.service 单元 [Unit] 配置：

```shell
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target
```

nginx.service 就比较有意思了，因为 nginx 涉及到网络，因此在启动时要求先启动网络环境。




















```bash
$ ls -l /etc/systemd/system/multi-user.target.wants/

...
aria2.service -> /lib/systemd/system/aria2.service
console-setup.service -> /lib/systemd/system/console-setup.service
cron.service -> /lib/systemd/system/cron.service
dmesg.service -> /lib/systemd/system/dmesg.service
e2scrub_reap.service -> /lib/systemd/system/e2scrub_reap.service
grub-common.service -> /lib/systemd/system/grub-common.service
grub-initrd-fallback.service -> /lib/systemd/system/grub-initrd-fallback.service
```




```shell


[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```



--



[https://en.wikipedia.org/wiki/Systemd](https://en.wikipedia.org/wiki/Systemd)

[https://www.freedesktop.org/software/systemd/man/systemd.unit.html](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)

[https://www.shellhacks.com/systemd-service-file-example/#comment-24301](https://www.shellhacks.com/systemd-service-file-example/#comment-24301)

[https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files#unit-specific-section-directives](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files#unit-specific-section-directives)







