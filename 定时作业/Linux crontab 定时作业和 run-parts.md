# 前言

crontab 是 Linux 内置的定时任务管理器，crontab 是 cron table 的缩写，即定是作业表。crontab 命令主要有如下两种形式：

```bash
$ crontab [-u user] file
$ crontab [-u user ] [ -i ] { -e | -l | -r }
```

第一个命令表示将指定文件设置称为指定用户的作业表文件，如果省略具体的用户就表示设置称为当前用户的作业表文件（文件必须已存在）。第二个命令表示操作具体用户的作业表文件，如果省略具体用户同样表示操作当前用户的作业表文件。

|**NOTE**|
|:--|
| `-u` 参数用于指定具体的用户，如果省略该参数就表示当前用户。|

而 option 有如下几个参数：

```
-e: 编辑作业表
-l: 列出作业表内容
-r: 删除作业表
-i: 在删除作业表之前提示确认操作(y/Y)
```

**这里要特别强调的是 -u 选项。再次说明，该选项后跟具体的 Linux 用户，表示当前作业表对这个指定的用户生效，如果不指定用户的话就默认是当前用户。**

# cron 表达式说明

crontab 是一个作业表，说直接点就是一个配置文件，用于配置定时任务，而该配置文件需要使用的 Linux 的 cron 表达式。所以在具体配置之前我们先来学习一下：

如何去学习 cron 表达式呢？直接在终端中输入如下命令：

```bash
$ crontab -e
```

咿？`-e` 不是编辑用户作业表吗？别急注意往下看：


当我们点击回车键后通常会提示如下信息，这个信息告诉你当前系统存在多个文件编辑器（如下我的有5个），选择你熟悉的编辑器即可。比我我这里选择 `vim.basic`，在下面选择中输入对应的数字序号 2：


```
no crontab for kali - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /usr/bin/code
  5. /bin/ed

Choose 1-5 [1]: 2 <== 在这里输入你想使用的编辑器
```

之后就能进入作业表文件了，一般用户初始默认的作业表是空的，不过在文件头部会有许多注释，这个注释就是告诉你该如何配置及如何编写 cron 表达式。

通常情况下，非 root 用户首次打开作业表文件，在文件首部都会有如下说明：

```
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
```

这个文件注释告诉你作业表该如何配置，而最后一行内容就是告诉你 cron 的具体格式。它代表的意思是：

```
m       h       dom     mon     dow   command
-       -       [-]     [-]     [-]   [-----]
|       |        |       |       |       |
|       |        |       |       |       +--------> 需要执行的命令
|       |        |       |       +----------------> 星期几 (0 - 7) (星期天为 0 或 7)
|       |        |       +------------------------> 月份 (1 - 12)
|       |        +--------------------------------> 每月几号 (1 - 31)
|       +-----------------------------------------> 小时 (0 - 23)
+-------------------------------------------------> 分钟 (0 - 59)
```

crontab 的命令构成是：时间 + 动作。其时间按顺序是分、时、日、月、周五种，而动作就是具体要执行的命令（command）。下面表格是每个单位的取值以及含义：


| **代表意义** | **分钟** | **小时** | **日期** | **月份** | **周** | **命令** |
| --- | --- | --- | --- | --- | --- | --- |
| 数字范围 | 0~59 | 0~23 | 1~31 | 1~12 | 0~7（0和7都表示星期日）| 需要执行的命令 |


另外，时间除了设置具体的数字之外还可以使用下面的时间操作符：

| **操作符** | **说明** |
| :--- | :--- |
| `*` | 代表单位内的所有时间 |
| `/` | 每隔指定时间单位，如 `*/5` 就表示每隔 5 个时间单位 |
| `-` | 区间取值，如 `5-10` 就表示第 5 到第 10 这几个时间单位 |
| `,` | 离散数字，指定多个具体的时间单位。如 `2,3,4` |


# cron 表达式示例


知道了表达式的基本组成我们就需要来练习一下。先来看一个简单的示例：


**1）每分钟执行一次 Command：**


```
* * * * *	Command
```

这条 cron 表达式的所有时间位都是 `*`，表示 ANY 的意思，后面的命令是一个具体要执行的命令。

**2）在每小时的第 20 分钟执行一次 Command：**

```
20 * * * *	Command
```

**3）在每小时的第第30 分钟各执行一次 Command：**

```
20,30 * * * *	Command
```

**4）在每小时的 20 ~ 30 分钟这段时间，每分钟执行一次 Command：**

```
20-30 * * * *	Command
```

**5）每隔五分钟执行一次 Command：**

```
*/5 * * * *	Command
```

**6）每隔2天执行一次 Command：**

```
* * */2 * *	Command
```

**7）每隔五天，在那天上午10点的第 20 ~ 30 分钟，每分钟执行一次 Command：**

```
20-30 10 */5 * *	Command
```

**8）每隔五天，在那天的上午10点的第 20 和 30 分钟各执行一次 Command：**

```
20,30 10 */5 * *	Command
```

**9）每星期六凌晨执行一次 Command：**

```
* 0 * * 6 Command
```

**10）每天晚上11点到早上7点，每隔1小时执行一次 Command：**

```
0 23-7/1 * * * Command
```


# corntab 实战

现在呢，我们来练习下 crontab 的使用。这次我要练习的是模拟心跳检测，每分钟发送一次请求，当然并不是真的要发送，我们可以将每分钟的时间记录到心跳检测文件中。

编写一个脚本，脚本文件名就叫 hearbeat：

```bash
$ vim hearbeat
```

内容如下：

```shell
#!/bin/bash

date +"%Y-%m-%d %H:%M" >> ~/timeline.txt
```

脚本很简单，就是输出一下当前时间，并将时间输出到 timeline.txt 文件中（`timeline.txt` 文件会自动创建）。之后需要给 `hearbeat` 文件一个执行权限：

```bash
$ sudo chmod +x hearbeat
```

现在来创建定时作业表，为了更详细的介绍 `crontab` 命令我不直接使用 `-e` 参数。而是创建一个具体的文件，在该文件中编写 cron 表达式，之后将该文件设置为当前用户的作业表。如下：

创建一个文件：

```bash
$ touch cronscheduling
```

在文件中输入如下内容：

```
* * * * * cd ~ && ./hearbeat >> ~/timeline.txt
```

之后使用下面的命令将该文件设置成为当前用户的作业表：

```bash
$ crontab cronscheduling
```

可以使用下面 `-l` 参数测试是否设置成功：

```bash
$ crontab -l
* * * * * cd ~ && ./hearbeat >> ~/timeline.txt # 输出了作业表内容, 说明设置成功了
```

|**注意**|
|:--|
|上面的操作其实完全可以直接使用 `crontab -e` 来实现，之所以单独创建一个文件的原因是为了演示如何将一个自定义的文件设置为用户的作业表。|

正常的话作业表就开始执行了，但是如果你的没有执行可能需要重启 cron 服务：

```bash
sudo systemctl restart cron.service
```

之后我们使用 tail 命令看 timeline.txt 文件中是否每分钟都输出一次时间：

```bash
$ tail -f timeline.txt
2021-10-29 22:43
2021-10-29 22:44
2021-10-29 22:45
```

Prefect ~

# run-parts 的介绍与使用


crontab 定时作业表用起来真的很方便。不过呢，如果单纯的直接使用还是有些麻烦的。


从前面的示例中可以看出，想要配置一个定时任务在用户的 crontab 作业表中增加一条 cron 表达式记录就好，但是如果我有上百个定时任务呢？也要一个一个的在作业表中写 cron 表达式？别闹，要死人的~

所以就需要借助 `run-parts` 工具了。这个功能很小众，很大一部分人都没听过，所以就来简单的说下。

先来看下 `run-parts` 命令在哪里：

```bash
$ which -a run-parts
/usr/bin/run-parts
/bin/run-parts
```

就是说 run-parts 工具在 `/usr/bin` 和 `/bin` 目录下都有（且没有使用软连接）。所以如果你无法执行 `run-parts` 命令可能是你的系统没有将这两个文件设置到环境变量 `PATH` 中。

当我们使用 man 命令查看 `run-parts` 的使用方式时，有如下一段介绍：

```
run scripts or programs in a directory
```

`run-parts` 的主要用法就是指定可执行文件所在的目录，如果这个目录下存在多个可执行文件，`run-parts` 会将该目录下的可执行文件全部执行一遍，妙极了不是？

`run-parts` 命令使用方式如下：

```bash
run-parts [option...] DIRECTORY
```

注意看，最后跟的是一个具体的目录，它的部分可选参数如下：

```
--test            打印 DIRECTORY 目录下可运行的脚本的名字（注意并不是真的运行），这个命令在测试时很实用
--list            打印 DIRECTORY 目录下有效的文件名，这个参数不能同时与 --test 一起使用

--verbose         在运行 DIRECTORY 目录下的脚本之前，先打印脚本的名称
--report          如果 DIRECTORY 下的脚本产生的输出信息，打印脚本的名称
--reverse         默认情况下，在执行 DIRECTORY 目录下的多个脚本时会按照脚本的名称顺畅执行。我们可以使用这个参数进行按照反向顺序执行
--exit-on-error   如果 DIRECTORY 目录下的某个脚本执行后返回的状态码不是 0，直接退出
...
```

在这些可选参数中，使用最后的就是 `--test` 和 ``--report`，下面来演示一下。

我们先在用户目录下的 tmp 目录下创建两个脚本，分别为 `print_time` 和 `test_network`。

`print_time` 脚本用于输出时间，内容如下：

```shell
#!/bin/bash

date +"%Y-%m-%d %H:%M" >> ~/tmp/record.txt
```

`test_network` 脚本用户测试网络是否正常（使用 `ping`）命令，内容如下：


```shell
#!/bin/bash

ping -c 1 baidu.com | head -n2 | tail -n1 >> ~/tmp/record.txt
```

先来看下这两个文件的属性：

```bash
$ ls -l
-rw-rw-r-- 1 kali kali 52 10月 30 11:21 print_time
-rw-rw-r-- 1 kali kali 69 10月 30 11:24 test_network
```

可以看到都是读写权限，并没有执行权限。

**注意：** `run-parts` 工具只会执行指定目录下具有可执行权限的脚本。

来使用 `run-parts` 工具验证下 tmp 目录下的文件，这里直接使用 `--test` 参数：

```bash
$ run-parts --test tmp/
```

结果你会发现没有任何输出，这就表示 tmp 目录下没有符合可执行脚本条件。所以我们需要给 `print_time` 和 `test_network` 可执行权限：

```bash
$ chmod +x print_time test_network
```

再次执行 `run-parts` 的测试命令，就会输出可执行文件的名称了：

```bash
$ run-parts --test tmp
tmp/print_time
tmp/test_network
```

这就表示一切正常了。

| **再次说明** |
| :--- |
| `run-parts` 只作用于指定目录下具有可执行权限的文件，可执行权限可以使用 `chmod` 命令设置。 |


现在我们来使用 `run-parts` 真正的执行一下 tmp 目录下的两个文件试试，使用如下命令（在实际使用中建议永远加上 `--report` 参数）：

```bash
$ run-parts --report tmp
```

现在看下 record.txt 文件：

```bash
$ cat tmp/record.txt
2021-10-30 11:25
64 bytes from 220.181.38.148 (220.181.38.148): icmp_seq=1 ttl=50 time=30.9 ms
```

这就说明了一切问题了吧，所以 `run-parts` 工具还是很有用的。那么就算如此跟 `crontab` 又有啥子关系呢？

我们想一个问题，比如在我们日常服务器维护中，经常会写一些特定的脚本。很多脚本都是每隔一段时间需要执行一次，一般都是在每天的凌晨执行。这么多的脚本不可能一个一个的手动执行吗？所以我们可以将这些具有相同时间性质的脚本归类，放到不同的文件夹中，比如需要一个小时执行一次的和每隔一天执行一次的。再配合 `crontab` 作业表简直不能再完美了不是？

比如上面的两个脚本，我想每隔一分钟执行一次就可以直接在 `crontab` 中这么写：

```
* * * * * cd ~ && run-parts --report tmp
```

保存之后就可以了，如果无法正常执行的话可能需要重启 cron 服务：

```bash
$ sudo systemctl restart cron.service
```

下面是使用 asciinema 录制的演示示例：


[![](https://asciinema.org/a/t5yFkqnaVMZvGGEisCVn336rS.svg#crop=0&crop=0&crop=1&crop=1&id=x76Pu&originHeight=862&originWidth=1417&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)](https://asciinema.org/a/t5yFkqnaVMZvGGEisCVn336rS)


# 系统级别 crontab 的使用和管理

前面说的定时作业都是用户级别下的，其实还有系统级别下的。系统级别下的 crontab 作业表是直接放在 /etc 目录下的，你可以使用下面的命令查看作业表文件和目录：

```bash
$ ls /etc | grep cron
```

通常情况下你会看到如下输出：

```
anacrontab
cron.d
cron.daily
cron.hourly
cron.monthly
crontab
cron.weekly
```

其中，cron.d、cron.daily、cron.hourly、cron.monthly、cron.weekly 都是文件夹，与 crontab 文件有关，这里我们先说下 `anacrontab`。

`anacrontab` 与 `crontab` 一样都是定时作业表，但是有些区别。`anacrontab` 是由 `anacron` 服务管理，而且 `anacron` 与普通的定时作业表也不同。Linux 系统虽然在不间断的运行但是呢，像男人一样，每个月总有那么几天不是。而巧的是，如果在这几天有 `crontab` 作业表定时任务的话，那这些任务是无法运行的，因为处于关机状态。这个时候就需要 `anacrontab` 作业表了，该作业表的作用就是用于监听 `crontab` 作业表在那么几天时间段内有没有运行，如果没有运行的话会调用服务运行一次。这就是 `anacron` 的核心作用，我们不需要关心，而且在非必要情况下也不要去修改 `anacrontab` 作业表。

所以我们来具体说下 `crontab` 系统作业表。先打开该文件，看下当前有那些内容，命令如下：

```bash
$ cat /etc/crontab
```

输出内容如下：

```
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

这个作业表的内容相信你肯定很熟悉，唯一的区别是定时任务的配置有具体的用户（示例 root）。那这是为什么呢？

原因是这个配置文件是系统级别的定时作业表，普通用户无权限执行。如果想要将上面的某个命令使用普通用户执行，那就需要将这个普通用户加入到超级管理员用户组才行。

我们看下上面系统作业表 crontab 的示例内容：

```
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

这个定时作业很有趣，注意看后面的具体命令，你会发现使用的是 `run-parts` 工具，那么后面的跟的肯定就是具体的文件夹了。现在知道 /etc 目录下的几个文件夹 cron.daily、cron.hourly、cron.monthly、cron.weekly 的作用了吗？每个文件夹对应的就是相应日期的定时作业脚本。

比如 /etc/cron.daily 文件夹，从明明也可以看到是每天执行一次的作业表。再看它的 cron 表达式：`25 6	* * *`，这不就是每天早上6点25分执行一次吗。

你会发现有了前面的知识这里就简单易懂了，而 Linux 也很贴心。给我们设定了几个默认的 cron 周期，比如以后你如果想要写一个每周运行一次的定时作业任务只需要直接在 /etc/cron.weekly 目录下编写一个对应的可执行脚本即可。是不是很简单？

# 关于 /etc/cron.allow 和 /etc/cron.deny

`/etc/cron.allow` 文件是用于定义可执行定时任务的用户白名单，对应的还有一个黑名单文件 `/etc/cron.deny`。如果不想让某个用户使用 cron 定时任务只需要将该用户加入到 `/etc/cron.deny` 文件中即可（一个用户占用一行）。

比如我将系统中的某个用户加入到 `/etc/cron.deny` 文件中，那么该用户就无法继续使用 cron 了。示例：

```bash
$ id
uid=1000(kali) gid=1000(kali) groups=1000(kali)

$ sudo echo kali >> /etc/cron.deny

$ sudo systemctl restart cron.service

$ crontab -l
You (kali) are not allowed to use this program (crontab)  # 好了，我无权限使用了
See crontab(1) for more information
```

好了，关于定时作业表就说这么多。相信这些知识足够满足我们日常需要了~

--

完结，撒花🎉🎉🎉~
