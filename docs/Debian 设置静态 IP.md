# Debian 设置静态IP

Ubuntu 也是基于 Debian 的发行版本，所以本篇文章不仅适用于 Debian 同样也适用于 Ubuntu。

唯一需要说明的是，当前 Debian 的最新版本是 10.7。而 Debian 自 10 之后的网络配置与 10 之前的版本有些区别。这个区别就导致 Debian 10 之前的版本配置适用于 Ubuntu 18.0 之前的版本，自从 Debian 10 开始的版本的网络配置适用于 Ubuntu18.0 及之后的版本。

Debian 的网络配置分两种：老版本使用的是 network，新版本使用的是 netplan 进行网络配置。看下下面的 Tabel 表格：

| 网络配置 | Debian           | Ubuntu           | 文件位置       |
| :------- | :--------------- | :--------------- | :------------- |
| network  | Debian10之前版本 | Ubuntu18之前版本 | `/etc/network` |
| netplan  | Debian10开始     | Ubuntu18开始     | `/etc/netplan` |

知道这些区别之后就开始做具体说明。首先，先介绍基于 netplan 的网络配置

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

可以看到，我这是在局域网内。所以 DNS 服务地址是 `127.0.0.53` 。大多数文章都写着修改配置为：`8.8.8.8`，原因咱也不懂，也不做过多说明，得到这个 DNS 服务器地址即可。

找到上面的两个信息之后就开始做具体配置了，关于 YAML 配置文件有哪些配置信息可以使用 man 命令查看 netplan 的详细信息，介绍的很详细。

```bash
$ man netplan
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
                    - 192.168.0.53
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

在以往的配置中都是 `dncp` 替换为 `static`，但是 `netplan` 中不同，使用 yes 或 no 表示使用静态还是动态。

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

在之后就是进行配置网关了，与 `dhcp` 一样，也是支持配置 IPv4 和 IPv6。比如上面我配置的是 IPv4，IP 就是文章开始查找得到的 IP。

`nameservers` 配置的是 DNS 解析服务器，下面的属性 `addresses` 是配置具体的 DNS 服务器地址。同样支持多个和支持数组输入：

```yaml
network:
    version: 2
    ethernets:
        ens33:
            nameservers:
                    addresses:
                    - 192.168.0.53
                    - 8.8.8.8
```

最后保存即可，输入如下命令使配置生效：

```bash
$ sudo netplan apply
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

| 注意                                                         |
| :----------------------------------------------------------- |
| 在进行设置静态 IP 时，如果指定的 IP 不是当前机器正在使用的 IP 的话一定要保证选择的 IP 并没有被占用。在配置该 IP 之前先使用 `ping` 命令看能否 PING 的通，如果通了就表示该 IP 已被占用，就不能进行设置成该 IP 了。 |

# 基于 Network 配置网络

基于 network 的网络配置文件是在 `/etc/network` 文件夹下，配置文件叫做 `interfaces`。

其他的不做过多解释，直接进行配置： 

首先最好先进行下备份：

```bash
$ sudo cp /etc/network/interfaces /etc/network/interfaces.bak
```

备份完成之后先来看下该文件中的内容：

```bash
$ cat /etc/network/interfaces
```

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens33
iface ens33 inet dhcp
```

看完信息就可以进行修改了，不过在之前最好先进行下备份：

我们要修改的内容有如下：

```bash
auto lo
iface ens33 inet dhcp
```

其中，`auto` 表示的是设置开机自动连接网络，如果你注意过 `ifconfig` 输出的内容，你就会看到有一项输出信息是 `lo`。

在前面我们就查到我们的网络接口是 `ens33`，现在将 `lo` 修改为 `ens33` 即可（你的可能是 `ens0`，不同发行版本可能不一致，不要做拿来主义~）

`iface` 表示的时网络接口配置，`dhcp` 表示的是使用动态 IP 配置，而要实现的就是静态 IP，所以将该值修改为 `static` 即可。

这个就明显与 Netplan 配置不一样，在 Netplan 的配置中是指定 IPv4 还是 IPv6。这里就完全不需要~

最后，我们再增加如下信息：

```bash
address 192.168.0.112        # 设置固定IP
netmask 255.255.255.0        # 设置子网掩码, 这个在 ifconfig 输出的信息中能看到
gateway 192.168.0.1          # 设置网关 IP
```

最后完整信息如下：

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto ens33
iface lo inet loopback

# The primary network interface
allow-hotplug ens33
iface ens33 inet static
address 192.168.1.81  
netmask 255.255.255.0    
gateway 192.168.1.1 
```

| 注意                                                         |
| :----------------------------------------------------------- |
| 在 `etc/network/interfaces` 配置文件中不要在配置信息行后面使用注释，否则之后可能会导致网络重启失败！！！！ |

最后重启网络：

使用命令：

```bash
sudo systemctl restart networking.service
```

或者使用命令：

```bash
$ sudo /etc/init.d/networking restart

[ ok ] Restarting networking (via systemctl): networking.service.
```

如果没有输出错误信息即表示网络配置成功！

这样，即使虚拟机重启也不怕 IP 变化了~
