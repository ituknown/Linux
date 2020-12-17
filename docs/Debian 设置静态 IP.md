# Debian 设置静态IP

Debian 9 以及之前的版本网络配置使用的是 `/etc/network/interfaces` 文件，从 Debian 10 开始使用的时 `netpla` 文件进行配置。

# Debian 9 及之前版本配置静态 IP

首先看下当前系统网络信息：

```bash
$ ifconfig

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.81  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::20c:29ff:fe40:bce  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:40:0b:ce  txqueuelen 1000  (Ethernet)
        RX packets 1797  bytes 179737 (175.5 KiB)
        RX errors 0  dropped 12  overruns 0  frame 0
        TX packets 188  bytes 20395 (19.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

这里要注意的是 `ens33` 网络接口（网卡）输出的信息，需要注意这个值。大多数的网卡名都是 `ens0`，这个可能根据发行版本以及操作软硬件环境不同可能不同，知道这个即可。

另外，该信息中包含当前机器的 IPv4 地址 `192.168.1.81` 以及 IPv6 地址 `fe80::20c:29ff:fe40:bce` 。我们一般都设置静态 IPv4，所以无需理会 IPv6，注意自己的 IPv4 即可。

之后就是网络子掩码信息 IP 了：`255.255.255.0`。

得到上面的信息之后现在还需要查看自己的网关 IP，执行如下命令查看：

```bash
$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG        0 0          0 ens33
192.168.1.0     0.0.0.0         255.255.255.0   U         0 0          0 ens33
```

其中第一行输出的信息 `192.168.1.1` 就是我们的网关。同样的，每个人的可能不同。

之后再来看下 DNS 信息：

```bash
$ cat /etc/resolv.conf | grep nameserver
nameserver 192.168.1.1
```

这里的 DNS 地址是 `192.168.1.1`，事实上，大多数需要做什么修改，这里仅展示说明一下。

现在就开始修改 `/etc/network/interfaces` 文件，先看下该文件中的内容：

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

```bash
$ sudo cp /etc/network/interfaces /etc/network/interfaces.bak
```

我们要修改的内容有如下：

```bash
auto lo
iface ens33 inet dhcp
```

其中，`auto` 表示的是设置开机自动连接网络，在文章开始时我们就查到我们的网卡是 `ens33`，之后将 `lo` 修改为 `ens33` 即可（你的可能是 `ens0`，不同发行版本可能不一致，不要做拿来主义~）

`iface` 表示的时网络接口配置，`dhcp` 表示的是使用动态 IP 配置，而要实现的就是静态 IP，所以将该值修改为 `static` 即可。

最后，我们再增加如下信息：

```bash
address 192.168.1.81         # 设置固定IP
netmask 255.255.255.0        # 设置子网掩码, 这个就是之前查询得到的掩码
gateway 192.168.1.1          # 设置网关 IP
```

`address` 表示的是要设置的 IP，根据之前 `ifconfig` 输出信息配置即可。

`netmask` 表示表示的是子网掩码，这个就是之前查询得到的掩码。

`gateway` 表示的是网关，值为之间查看的即可。

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