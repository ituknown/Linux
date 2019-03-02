# Linux 环境下 JAVA 安装

这里以 **JDK8** 为例，首先在 Linux 中输入如下命令，检验 Liiinux 系统版本信息：

```
# 检验 Linux 系统版本信息
$ uname -sr
Linux 3.10.0-957.5.1.el7.x86_64
```

如果输出结果包含有 `x86_64` 表明系统是 `64` 位，如果出现 `i686` 则说明是 `32` 位操作系统。

进入 [oracle 官网](https://www.oracle.com/) 下载对应操作系统的 JDK 8 版本：

![JDK-8](./_images/jvm/JDK-8.png)

另外，如果想下载历史版本 JDK 则需要在下载页面找到 JDK 归档栏：

![JDK-Archive](./_images/jvm/JDK-Archive.png)

点击归档后既能看到所有 JDK 版本，点击下载即可！

![JDK-list.png](./_images/jvm/JDK-list.png)

# 登录Linux，切换到 root 用户

如果当前已是 root 用户不需要变更。否则：

```
# 获取root用户权限，当前工作目录不变（需要 root 密码）
$ su root
# 或则使用该命令如下命令，该指令不需要 root 密码直接切换成 root（需要当前用户密码）
$ sudo -i
```

# 建立、上传 JDK 安装目录

这里直接在 `/usr` 目录下建立JDK安装目录：

```
# 创建 JVM 目录
$ mkdir jvm
```

将下载的 JDK8 上传至该目录下

```
# 在 JVM 目录下输入该指令
$ rz
```

**说明：** `rz` 指令是上传文件的意思，执行该命令后，在弹出框中选择要上传的文件即可。`sz fileName` 执行该命令后，是将 fileName 文件发送到本地。
另外，如果提示 `unknown rz command` 则说明没有安装该指令，只需要输入如下命令安装即可：

```
# 安装 lrzsz
$ yum install -y lrzsz
```

# 解压 JDK 到当前目录

```
# 将文件压缩包解压到当前目录
$ tar -zxvf jdk-8u60-linux-x64.tar.gz
```

会得到 `jdk1.8.0_60` 目录。

# 配置环境变量

JDK 安装完成后进入 `/etc` 目录编辑 `profile` 文件

```
$ cd /etc
$ vim profile
```

在该文件中添加如下配置信息：

```
# JVM Environment
JAVA_HOME=/usr/jvm/jdk1.8.0_60
CLASSPATH=.:${CLASSPATH}:${JAVA_HOME}/lib
PATH=${PATH}:${JAVA_HOME}/bin
export JAVA_HOME
export CLASSPATH
export PATH

# 或者如下方式
export JAVA_HOME=/usr/jvm/jdk1.8.0_60
export CLASSPATH=.:${CLASSPATH}:${JAVA_HOME}/lib
export PATH=${PATH}:${JAVA_HOME}/bin
```

<!--sec data-title="警告" data-id="section0" data-show=true ces-->

`JAVA_HOME`、`CLASSPATH`、`PATH` 与 `=` 之间不能有空格！！！！

```
JAVA_HOME=/usr/jvm/jdk1.8.0_60
CLASSPATH=.:${CLASSPATH}:${JAVA_HOME}/lib
PATH=${PATH}:${JAVA_HOME}/bin
```
不能写成：
```
JAVA_HOME   = /usr/jvm/jdk1.8.0_60
CLASSPATH   = .:${CLASSPATH}:${JAVA_HOME}/lib
PATH        = ${PATH}:${JAVA_HOME}/bin
```

另外，在配置 `CLASSPATH` 时一定要将原 `CLASSPATH` 加上：

```
CLASSPATH=.:${CLASSPATH}:`JAVA_CLASSPATH`
```

`PATH` 也要将原 `PATH` 加上：

```
PATH=${PATH}:`JAVA_PATH`
```

否则在配置并应用后命令将不可使用。

如输入 `vi`、`yum` 等命令不能使用，原因就是这里配置错误。如果配置错误后就需要进行重置 `PATH`，执行如下命令重置：

```
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
```

然后要仔细检查 `profile` 配置文件，检查是那部分配置错误并要修正再次应用即可！

<!--endsec-->

环境变量配置完成后输入如下指令

```
# 立即重启机器
$ sudo shutdown -r now
# 或者直接输入如下指令生效配置文件
$ source /etc/profile
```

再检验 JDK，看是否配置完成

```
$ java -version
# 得到如下信息即表示完成
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) Client VM (build 25.60-b23, mixed mode)
```
