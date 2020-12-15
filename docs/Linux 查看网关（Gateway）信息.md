# 查看网关（Gateway）

本文主要介绍如何查看 Linux 系统的网关出口信息以及默认的网关 IP。

# 使用 `route` 命令查看网关信息

route 命令主要是查看主机的路由信息，而第一条路由就是机器的出口网关。在终端输入 `route` 命令就可以列出路由信息，示例：

```bash
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    100    0        0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 ens33
_gateway        0.0.0.0         255.255.255.255 UH    100    0        0 ens33
```

而第一条信息标识为 `default _gateway` 表示是当前机器的默认官网。之后增加 `-n` 参数查看默认网关对应的 IP 即可：

```bash
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 ens33
192.168.1.1     0.0.0.0         255.255.255.255 UH    100    0        0 ens33
```

所以，当前机器的默认网关 IP 为 `192.168.1.1`。

# 使用 `traceroute` 命令查看网关信息

`traceroute` 命令主要是跟踪路由，该命令通常机器没有默认安装。如果输入该命令时提示没有该命令时就需要先进行安装软件。软件可以选择 `inetutils-traceroute` 或 `traceroute` 都可以：

```bash
## Debian/Ubuntu
$ sudo apt install -y traceroute/inetutils-traceroute

## RHEL/CentOS
$ sudo yum install -y traceroute/inetutils-traceroute
```

该命令是用于路由跟踪，也就是说输入目标域名或 IP 即可展示请求请求的具体路径，而第一个 IP 就是本机的出口 IP，即网关。比如我跟踪 `baidu.com`：

```bash
$ traceroute baidu.com

traceroute to baidu.com (39.156.69.79), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.174 ms  0.198 ms  0.316 ms
 2  114.86.224.1 (114.86.224.1)  5.272 ms * *
 3  61.152.6.73 (61.152.6.73)  5.932 ms 61.152.6.45 (61.152.6.45)  2.382 ms 61.152.6.77 (61.152.6.77)  2.710 ms
 4  61.152.24.102 (61.152.24.102)  3.846 ms 61.152.25.94 (61.152.25.94)  2.620 ms 61.152.24.226 (61.152.24.226)  2.351 ms
 5  202.97.40.45 (202.97.40.45)  28.765 ms 202.97.97.237 (202.97.97.237)  24.348 ms 202.97.97.221 (202.97.97.221)  26.101 ms
 6  202.97.17.94 (202.97.17.94)  25.762 ms
 ......
```

在每一行前面都有一个序号，表示请求所有的路径（IP 网关）。可以看到序号1展示的就是我们的默认网关。

比如我再次跟踪 `taobao.com` 请求路径：

```bash
$ traceroute taobao.com

traceroute to taobao.com (140.205.220.96), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.280 ms  0.314 ms  0.343 ms
 2  114.86.224.1 (114.86.224.1)  7.164 ms  7.427 ms  7.837 ms
 3  61.152.6.77 (61.152.6.77)  4.997 ms  5.190 ms 61.152.6.45 (61.152.6.45)  4.792 ms
 4  101.95.206.2 (101.95.206.2)  4.470 ms 124.74.166.18 (124.74.166.18)  3.968 ms 124.74.166.22 (124.74.166.22)  3.888 ms
 5  101.95.209.62 (101.95.209.62)  4.522 ms 101.95.208.90 (101.95.208.90)  5.661 ms 101.95.209.54 (101.95.209.54)  4.659 ms
 6  101.95.211.118 (101.95.211.118)  4.517 ms 180.163.38.82 (180.163.38.82)  3.706 ms 180.163.38.90 (180.163.38.90)  3.815 ms
 ......
```

可以到，除了第一条出口网关，其他的路径基本上是不同的。

这里也就能确定了我们的默认网关是 `192.168.1.1`。

# 使用 `netstat` 命令查看网关信息

`netstat` 命令同样也可以查看网关信息，因为它有一个 `-r` 查看可以查看路由信息（等价于 `route` 命令）：

```bash
$ netstat -rn

Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG        0 0          0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U         0 0          0 ens33
192.168.1.1     0.0.0.0         255.255.255.255 UH        0 0          0 ens33
```

所以，我们同样可以使用 `netstat` 命令确定网关信息。

# 使用 `ip` 命令查看网关信息

除了上面的路由跟踪命令，我们也可以直接使用 IP 命令进行查看网关。不过该命令不如上面的直观：

```bash
$ ip route show
default via 192.168.1.1 dev ens33 proto dhcp src 192.168.1.65 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
192.168.1.0/24 dev ens33 proto kernel scope link src 192.168.1.65 
192.168.1.1 dev ens33 proto dhcp scope link src 192.168.1.65 metric 100 
```

同样的，第一条信息也就能确定我们的网关信息了。