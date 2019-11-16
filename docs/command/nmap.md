# 前言

[Nmap](https://nmap.org) 是世界上最受欢迎的网络扫描器之一。网络安全专业人员和新手都利用它来审计和发现本地和远程开放端口以及主机和网络信息。

[Nmap](https://nmap.org) 是开源的，免费的，多平台的，并且每年都会不断更新。它还具有很大的优势：它是可用的最完善的主机和网络扫描器之一。它包含大量选项来增强您的扫描任务，
[Nmap](https://nmap.org) 可用于：

- 创建完整的计算机网络图
- 查找任何主机的远程IP地址
- 获取操作系统系统和软件详细信息
- 检测本地和远程系统上的开放端口
- 审核服务器安全性标准
- 查找远程和本地主机上的漏洞

# Nmap 安装

[Nmap 官网下载页面](https://nmap.org/download.html) 分别提供了 Windows、Linux 以及 MacOS 三个平台下载版本。另外，在 Mac 中建议直接使
用 `brew` 管理工具下载 `nmap`，下面以 Mac 为例安装：

- 使用 `brew` 搜索 `nmap` 软件包

```bash
$ brew search nmap

==> Formulae
nmap ✔

==> Casks
zenmap

If you meant "nmap" specifically:
true
```

[Nmap](https://nmap.org) 分别提供了两种版本，分别是命令版的 `nmap` 以及基于 GUI 图形化工具的 `zenmap`。根据喜好自行选择，这里以 `nmap`
为例：

- 使用 `brew` 查看 `nmap` 软件包信息

```bash
$ brew info nmap

nmap: stable 7.80 (bottled), HEAD
Port scanning utility for large networks
https://nmap.org/
Conflicts with:
  ndiff (because both install `ndiff` binaries)
/usr/local/Cellar/nmap/7.80_1 (821 files, 27.0MB) *
  Poured from bottle on 2019-11-15 at 10:10:10
From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/nmap.rb
==> Dependencies
Required: openssl@1.1 ✔
==> Options
--HEAD
	Install HEAD version
==> Analytics
install: 30,981 (30 days), 125,515 (90 days), 343,793 (365 days)
install_on_request: 29,100 (30 days), 113,950 (90 days), 323,261 (365 days)
build_error: 0 (30 days)
```

在信息列表中可以看到 [Nmap](https://nmap.org) 官网以及版本发布信息。开始安装：

```bash
$ brew install nmap
```

安装完整后在 DOS 中输入 `nmap -V` 查看版本信息，输出如下所示即安装完成：

```bash
$ nmap -V

Nmap version 7.80 ( https://nmap.org )
Platform: x86_64-apple-darwin19.0.0
Compiled with: nmap-liblua-5.3.5 openssl-1.1.1d nmap-libssh2-1.8.2 libz-1.2.11 nmap-libpcre-7.6 nmap-libpcap-1.9.0 nmap-libdnet-1.12 ipv6
Compiled without:
Available nsock engines: kqueue poll select
```

下面让我们来看看最常用的 [Nmap](https://nmap.org) 扫描命令。

# 针对 IP 或主机的基本 Nmap 扫描

```bash
$ nmap IP/Host/域名
```

以 IP `172.16.6.96` 为例（下面都将基于本 IP 做测试）：

```bash
$ nmap 172.16.6.96

Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-15 10:16 CST
Nmap scan report for 172.16.6.96
Host is up (0.019s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2179/tcp open  vmrdp
3306/tcp open  mysql
5357/tcp open  wsdapi
```

可以看到，扫描 IP `172.16.6.96` 发现该机器对外开放了 `135`、`3306` 等端口号信息。

# 扫描本地或远程服务器上的特定端口或扫描整个端口范围

```bash
$ nmap -p 1-65535 IP/Host/域名
```

在此示例中，我们扫描了本地计算机的所有（从 1 到 65535，没有包括 0 端口）端口。

`Nmap` 能够扫描所有可能的端口，但是您也可以扫描特定的端口。

```bash
$ nmap -p 80,443,3306 IP/Host/域名
```

这样，只会扫描指定 IP 的 `80`、`443` 和 `3306` 端口了。

```bash
$ nmap -p 80,443,3306 172.16.6.96

Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-15 10:38 CST
Nmap scan report for 172.16.6.96
Host is up (0.0078s latency).

PORT     STATE  SERVICE
80/tcp   closed http
443/tcp  closed https
3306/tcp open   mysql
```

# 扫描多个IP地址

让我们尝试扫描多个IP地址。为此，您需要使用以下语法：

```bash
$ nmap 1.1.1.1 8.8.8.8 
```

您还可以扫描连续的IP地址：

```bash
$ nmap -p 1.1.1.1,2,3,4 
```

这将扫描 `1.1.1.1`、`1.1.1.2`、`1.1.1.3` 和 `1.1.1.4`。

# 扫描IP范围

您还可以使用 `Nmap` 扫描整个 `CIDR` IP 范围，例如：

```bash
$ nmap -p 8.8.8.0/28 
```

这将扫描从 `8.8.8.1` 开始14个连续IP范围到 `8.8.8.14`。

另一种选择是简单地使用这种范围：

```bash
$ nmap 8.8.8.1-14 
```

您甚至可以使用通配符来扫描整个C类IP范围，例如：

```bash
$ nmap 8.8.8.* 
```

这将扫描 256 个 IP 地址 `8.8.8.1` 到 `8.8.8.256`。

如果您需要从 IP 范围扫描中排除某些 IP ，则可以使用 `–exclude` `选项，如下所示：

```bash
$ nmap -p 8.8.8.* --exclude 8.8.8.1 
```

# 扫描最受欢迎的端口

结合使用 `–top-ports` `参数和特定数字，您可以扫描该主机的 X 个最常见端口，如我们所见：

```bash
$ nmap --top-ports 20 172.16.6.96
```

```
$ nmap --top-ports 20 172.16.6.96

Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-15 10:44 CST
Nmap scan report for 172.16.6.96
Host is up (0.0092s latency).

PORT     STATE  SERVICE
21/tcp   closed ftp
22/tcp   closed ssh
23/tcp   closed telnet
25/tcp   closed smtp
53/tcp   closed domain
80/tcp   closed http
110/tcp  closed pop3
111/tcp  closed rpcbind
135/tcp  open   msrpc
139/tcp  open   netbios-ssn
143/tcp  closed imap
443/tcp  closed https
445/tcp  open   microsoft-ds
993/tcp  closed imaps
995/tcp  closed pop3s
1723/tcp closed pptp
3306/tcp open   mysql
3389/tcp closed ms-wbt-server
5900/tcp closed vnc
8080/tcp closed http-proxy
```

# 扫描从文本文件读取的主机和IP地址

在这种情况下，`Nmap` 还可用于读取内部包含主机和IP的文件，假设您创建一个 `list.txt` 文件，其中包含以下几行：

```text
172.16.6.96
192.168.1.106
127.0.0.1
microsoft.com 
```

使用 `-iL` 参数使您可以读取该文件，并为您扫描所有这些主机：

```bash
$ nmap -iL list.txt
```

# 将Nmap扫描结果保存到文件中

另一方面，在下面的示例中，我们将不会从文件中读取内容，而是将结果导出/保存到文本文件中：

```bash
$ nmap -oN output.txt microsoft.com 
```

`Nmap` 也可以将文件导出为 XML 格式，如下示例：

```bash
$ nmap -oX output.xml microsoft.com 
```

# 禁用DNS名称解析

如果您需要稍微加快扫描速度，则可以选择对所有扫描禁用反向DNS解析。只需添加 `-n` 参数：

```bash
$ nmap -p 80 -n 8.8.8.8
```