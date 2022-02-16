`dpkg` 命令 是 Debian 系列 Linux 系统用来安装、创建和管理软件包的工具。

# 语法

```
dpkg [option] [package.deb|package]
```

其中 `[option]` 指可选参数，`package.deb` 指的是离线软件包，`package` 指的是基于 `package.deb` 安装的系统软件。

下面是参数 option 说明

```
-i <package.deb>       : 安装软件包
-r <package>           : 卸载使用 -i 参数安装的软件, 保留配置
-P <package>           : 卸载使用 -i 参数安装的软件, 同时删除其配置文件
-L <package>           : 列出与该软件关联的文件
-l <package>           : 列出已安装的软件(如果指定 package 则显示该 package 的信息)
--unpack <package.deb> : 解压软件包
-c <package.deb>       : 显示软件包文件信息
--confiugre <package>  : 配置软件包
```

# 示例

