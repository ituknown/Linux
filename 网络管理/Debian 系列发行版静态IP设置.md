# 前言

Ubuntu 也是基于 Debian 的发行版本，所以本篇文章不仅适用于 Debian 同样也适用于 Ubuntu。

唯一需要说明的是，Ubuntu 自18.0 开始关于网络的配置做了调整。在18.0之前使用的网络配置是 `Network，18`.0 以及之后的版本使用的是 `Netplan` 作为网络配置。

如果你不确定当前操作系统使用的是 `Network` 还是 `Netplan` 可以使用 `man` 命令测试下：


```bash
$ man networks

$ man netplan
```

比如当前 Debian 发行版（Buster）使用的是 `networks`，如果使用 `man` 查看 `netplan` 就会提示：

```
No manual entry for netplan
```

下面是 Ubuntu 和 Debian 发行版使用的网络配置，当然最简单的方式还是直接使用 `man` 命令进行确认。


| **网络配置** | **Debian**                                | **Ubuntu**             | **配置文件位置** |
| :----------- | :---------------------------------------- | :--------------------- | :--------------- |
| network      | Debian各系列发型版本（当前最新是 Buster） | Ubuntu18之前发行版本   | `/etc/network`   |
| netplan      |                                           | Ubuntu18及之后发行版本 | `/etc/netplan`   |


知道这些区别之后就开始做具体说明。首先，先介绍基于 network 的网络配置。

# 基于 Network 配置网络


| **Note**                                                     |
| :----------------------------------------------------------- |
| 虽然说本文是介绍 Debian 系列发行版如何设置静态 IP，但是只要使用 `Network` 的发行版其实都适用的（比如 CentOS 就是使用 Network）！ |


Network 的配置文件是在 `/etc/network` 目录下，该目录下有文件，而用于配置网卡的则是 `interfaces` 文件：

```bash
$ ls /etc/network
if-down.d  if-post-down.d  if-pre-up.d	if-up.d  interfaces  interfaces.d
```

`interfaces` 是接口的意思，我们通常说的网卡指的就是网络接口，有关该文件的配置可以使用 `man` 命令查看详细信息：

```bash
man interfaces
```

废话少说，来看该怎么配置！首先先进行下备份，这是修改系统配置的必须步骤，可用于配置还原，应该养成随时备份的好习惯：

```bash
sudo cp /etc/network/interfaces /etc/network/interfaces.bak
```

| **Note**                                                     |
| :----------------------------------------------------------- |
| 修改系统配置需要有超级管理员权限，也就是 `root` 用户或能够使用 `sudo` 权限的用户！ |


备份完成之后先来看下该文件中的内容：


```bash
cat /etc/network/interfaces
```

下面展示的内容是将不必要的信息删除后的内容，其中 `ens33` 就是我们的网络接口，也就是网卡：

```
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

auto ens33
allow-hotplug ens33
iface ens33 inet dhcp
```

我们可以使用 `iproute2` 命令当前系统的网络接口：

```bash
$ ip -c link show

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:1b:a9:08 brd ff:ff:ff:ff:ff:ff
```

也就是说我们的 IP 其实就是绑定在网络接口 `ens33` 上的，可以继续查看下当前系统的 IP：

```bash
$ ip -c addr show
```

输出示例：

![show-ip-1637739047LlljmB](http://linux-media.knowledge.ituknown.cn/NetworkManager/Debian-StaticIP/show-ip-1637739047LlljmB.png)

看完信息就可以进行修改了，不过在之前最好先进行下备份：


其中，`auto` 表示的是设置开机自动连接网络，如果你注意过 `ifconfig` 输出的内容，你就会看到有一项输出信息是 `lo`，`lo` 表示的是回环网络。


在前面我们就查到我们的网络接口是 `ens33`，现在新增一个 `auto ens33` 用于开机自动启动 `ens33` 网卡即可（你的可能是 `ens0`，不同发行版本可能不一致，不要做拿来主义~）。


`iface` 表示的时网络接口配置，`dhcp` 表示的是使用动态 IP 配置，而要实现的就是静态 IP，所以将该值修改为 `static` 即可。


这个就明显与 Netplan 配置不一样，在 Netplan 的配置中是指定 IPv4 还是 IPv6。这里就完全不需要~


最后，我们再增加如下信息：


```bash
address 192.168.0.112                     # 设置固定IP
netmask 255.255.255.0                     # 设置子网掩码, 这个在 ifconfig 输出的信息中能看到
gateway 192.168.0.1                       # 设置网关 IP
dns-nameservers 8.8.8.8 114.114.114.114   # 设置 DNS
```


最后完整信息如下：


```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens33

# 配置自动启动 ens33 网桥
auto ens33
# 具体配置
iface ens33 inet static
     address 192.168.1.81
     netmask 255.255.255.0
     gateway 192.168.1.1
     dns-nameservers 114.114.114.114 8.8.8.8
```



| 注意 |
| --- |
| 在 `/etc/network/interfaces` 配置文件中不要在配置信息行后面使用注释，否则之后可能会导致网络重启失败。另外，看上面的配置信息，注意缩进！！！！ |



最后重启网络使用命令：


```bash
sudo systemctl restart networking.service
```


或者使用命令：


```bash
$ sudo /etc/init.d/networking restart

[ ok ] Restarting networking (via systemctl): networking.service.
```


如果没有输出错误信息即表示网络配置成功！


注意：使用 Network 配置安装上述方式配置静态 IP 时并没有设置 DNS 解析地址，如果应用网络之后 `ping baidu.com` 时遇到如下错误可能的原因原因是 DNS 解析的问题，具体解决方式见 QAQ。
```
Temporary failure in name resolution DNS
```
这样，即使虚拟机重启也不怕 IP 变化了~
​

# 资源链接


[https://wiki.debian.org/NetworkConfiguration](https://wiki.debian.org/NetworkConfiguration)
​

[https://askubuntu.com/questions/143819/how-do-i-configure-my-static-dns-in-interfaces](https://askubuntu.com/questions/143819/how-do-i-configure-my-static-dns-in-interfaces)
​

# FAQ
## Temporary failure in name resolution DNS？


该问题的可能原因是网关配置错误，比如按照上述本文说明配置的 IP 地址为： `192.168.0.112/24` 。那么网段就是 `192.168.0` ，所以网关的地址就是 `192.168.0.1~225` 之间，具体是其中的哪一个可以使用 `netstat -rn` 命令进行确定。


如果将网关地址设置为 `192.168.1.1` 那么肯定会出现该错误的。


除了网关的原因之外，还一个原因可能是 DNS 配置错误。如前面说的，在查看 `/etc/resolv.conf` 文件得到的 DNS 地址 `127.0.0.53` 。如果在配置静态 IP 时将 DNS 解析地址设置为该值那么也可能会得到该错误，解决方式就是在配置静态 IP 时将 DNS 首选解析地址设置为国内的 `114.114.114.114` ，另外还可以加上谷歌的 DNS 解析地址 `8.8.8.8` 。


比如基于 Netplan 配置 DNS 地址：


```yaml
network:
  version: 2
  ethernets:
    ens33:
      nameservers:
        addresses:
        - 114.114.114.114 # 国内首选 DNS 解析地址
        - 8.8.8.8         # 谷歌 DNS 解析地址
```


如果是基于 Network 配置的话可以通过修改 `/etc/systemd/resolved.conf` 文件进行设置 DNS 解析地址。将该文件中的 DNS 值首选设置为 `114` 之后再跟一个谷歌的 `8.8` ：
```
[Resolve]
DNS=114.114.114.114 8.8.8.8
```
目前笔者知道的解决方式就这两种，如果还是无法解决该问题就向度娘、谷歌求助了~


[![asciicast](https://asciinema.org/a/451110.svg)](https://asciinema.org/a/451110)


# 基于 Netplan 配置网络


netplan 与之前的版本的网络配置有些区别，它的网络配置文件在 `/etc/netplan` 目录下面。该目录下可能存在一个或多个配置文件，比如我这里就只有一个网络配置文件：


```bash
$ ls /etc/netplan/

50-cloud-init.yaml
```


从文件命名也能看出一些区别，它是一个基于 YAML 的配置文件。这个文件中默认有些配置信息：


```bash
$ cat /etc/netplan/50-cloud-init.yaml

network:
  ethernets:
    ens33:
      dhcp4: true
  version: 2
```


当前还有其他的一些配置信息，我们暂时先不管。我们由于要进行配置静态网络所以我们首先需要查找一些基本的配置信息：


**查找网关**


```bash
$ netstat -rn

Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.0.1     0.0.0.0         UG        0 0          0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
192.168.0.0     0.0.0.0         255.255.255.0   U         0 0          0 ens33
```


其中第一条输出信息中的 IP `192.168.0.1` 就是我们的网关，先记下。


**查找 DNS**


这个 DNS 直接在 `/etc/resolv.conf` 中进行查找即可：


```bash
$ cat /etc/resolv.conf | grep nameserver
nameserver 127.0.0.53
```


可以看到，我这是在局域网内，所以 DNS 服务地址是 `127.0.0.53` 。


但是如果在配置静态 IP 时将 DNS 解析地址设置成该值就无法连接网络。比如 `ping baidu.com` 时就会提示如下错误：


```
Temporary failure in name resolution DNS
```


所以，在配置静态 IP 时我们需要将 DNS 解析地址设置为 `114.114.114.114` 或者 `8.8.8.8` 。


其中 `114.114.114.114` 是国内的 DNS 解析地址， `8.8.8.8` 是谷歌的 DNS 解析地址。在设置 DNS 解析时，如果仅仅设置为 `8.8.8.8` 的话在国内..... ，所以在实际使用中最好将两者全部配置上去。


找到上面的两个信息之后就开始做具体配置了，关于 YAML 配置文件有哪些配置信息可以使用 man 命令查看 netplan 的详细信息，介绍的很详细。


```bash
man netplan
```


先来看下我的配置文件中内容：


```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      optional: no
      addresses:
      - 192.168.0.112/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
        - 114.114.114.114 # 国内 DNS 解析地址
        - 8.8.8.8 # 谷歌 DNS 解析地址
```


`network.version` 这个信息是死的，不用管。我们主要关心的是 `network.ethernets` 网卡下的配置信息。


`ens33` 指的是网络接口，大多数用的的网络接口名都是 `ens0`。怎么知道自己的网络接口名呢？直接使用 `ifconfig` 命令查看输出的信息，其中包含自己 IP 的那个就是你的网络接口名，比如我的：


```bash
$ ifconfig

...
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.117  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::20c:29ff:fecd:48cf  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:cd:48:cf  txqueuelen 1000  (Ethernet)
        RX packets 140745  bytes 93399083 (93.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 28012  bytes 2392981 (2.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

...
```


指定了网络接口后就可以进行设置该接口下指定的 IP 是静态还是动态了。


在以往基于 Netwoek 的配置中都是将 `dncp` 替换为 `static` 表示使用静态 IP，但是 `netplan` 则不同，使用 yes 或 no 表示使用静态还是动态。


可以看到我这里配置的是 `dhcp4: no` 表示静态 IPv4。如果你要设置静态 IPv6 就设置 `dhcp6: no` 就好。


`optional` 指的是是否为可选，不懂啥意思，不配置也没关系。


在之后就是 `addresses` 的配置了，这里就是进行设置你要设置的静态 IP，支持多个，也支持数组输入，示例：


```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      optional: no
      addresses:
      - 192.168.0.112/24
      - 192.168.0.117/24

# 或者

network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      optional: no
      addresses: [192.168.0.112/24, 192.168.0.117/24]
```


再之后就是进行配置网关了，与 `dhcp` 一样，也是支持配置 IPv4 和 IPv6。比如上面我配置的是 IPv4，IP 就是文章开始查找得到的 IP：


```yaml
network:
  version: 2
  ethernets:
    ens33:
      gateway4: 192.168.0.1 # 配置 IPv4 网关
      # gateway6: # 配置 IPv6 网关
```


`nameservers` 配置的是 DNS 解析服务器，下面的属性 `addresses` 是配置具体的 DNS 服务器地址。同样支持多个和支持数组输入：


```yaml
network:
  version: 2
  ethernets:
    ens33:
      nameservers:
        addresses:
        - 114.114.114.114   # 国内首选 DNS 解析地址
        - 8.8.8.8           # 国外谷歌 DNS 解析地址
```


最后保存即可，输入如下命令使配置生效：


```bash
sudo netplan apply
```


之后在查看自己的 IP 就发现 IP 变成了自己配置的 IP 了，即使关机重启 IP 也不再发生变化了：


```bash
$ ifconfig

...
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.112  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::20c:29ff:fecd:48cf  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:cd:48:cf  txqueuelen 1000  (Ethernet)
        RX packets 140745  bytes 93399083 (93.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 28012  bytes 2392981 (2.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

...
```



| 注意 |
| --- |
| 在进行设置静态 IP 时，如果指定的 IP 不是当前机器正在使用的 IP 的话一定要保证选择的 IP 并没有被占用。在配置该 IP 之前先使用 `ping` 命令看能否 PING 的通，如果通了就表示该 IP 已被占用，就不能进行设置成该 IP 了。 |



