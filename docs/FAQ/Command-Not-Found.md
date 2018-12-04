# ifconfig：command not found

---

## 不推荐

执行命令查找是哪个包提供的 `ifconfig` 命令：

```
yum provides ifconfig
```

Console：

```
Loaded plugins: fastestmirror
base                                                             | 3.6 kB  00:00:00     
extras                                                           | 3.4 kB  00:00:00     
updates                                                          | 3.4 kB  00:00:00     
(1/4): base/7/x86_64/group_gz                                    | 156 kB  00:00:00     
(2/4): extras/7/x86_64/primary_db                                | 130 kB  00:00:00     
(3/4): base/7/x86_64/primary_db                                  | 5.7 MB  00:00:04     
(4/4): updates/7/x86_64/primary_db                               | 3.6 MB  00:00:44     
Determining fastest mirrors
 * base: mirrors.aliyun.com
 * extras: mirrors.cn99.com
 * updates: centosr4.centos.org
base/7/x86_64/filelists_db                                       | 6.7 MB  00:00:04     
extras/7/x86_64/filelists_db                                     | 494 kB  00:00:00     
updates/7/x86_64/filelists_db                                    | 2.1 MB  00:00:03     
net-tools-2.0-0.22.20131004git.el7.x86_64 : Basic networking tools
Repo        : base
Matched from:
Filename    : /sbin/ifconfig
```

可以看到是 `net-tools` 包来使用 `ifconfig` 命令，所以只需要安装该工具包即可：

```
yum install net-tools
```

现在再次执行命令输出结果如下所示：

```
[root@localhost ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.*.*  netmask 255.255.254.0  broadcast 10.81.*.*
        ether 00:16:3e:0e:e2:bb  txqueuelen 1000  (Ethernet)
        RX packets 6182278  bytes 5521748866 (5.1 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3366920  bytes 615374611 (586.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.*.*  netmask 255.255.252.0  broadcast 47.97.*.*
        ether 00:16:3e:0d:48:95  txqueuelen 1000  (Ethernet)
        RX packets 6141499  bytes 779657050 (743.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10066674  bytes 928282780 (885.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 1900960  bytes 384324308 (366.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1900960  bytes 384324308 (366.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

>**注意：** 以上 IP 使用 * 进行代替

---

## 推荐

现在 `ifconfig` 命令被 `ip a` 所取代。

不过一般情况下还是能找到 `/sbin/ifconfig` 命令，只不过可能没有安装，它是软件包 `net-tools` 的一部分，默认情况下不会安装，因为它已被弃用并被包 `iproute2` 中的命令 `ip` 取代。

`ifconfig` 命令相当于 `ip addr show`，因为对象命令可以被省略，所以也可以直接使用命令 `ip a`。

不过输出格式有些不同：

<u>ifconfig</u>

```
[root@localhost /]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.31.129  netmask 255.255.255.0  broadcast 192.168.31.255
        inet6 fe80::e91f:f60b:60f3:b9a2  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:24:c1:38  txqueuelen 1000  (Ethernet)
        RX packets 22101  bytes 25801866 (24.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6134  bytes 590352 (576.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

<u>ip addr</u>

```
[root@localhost /]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:24:c1:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.129/24 brd 192.168.31.255 scope global noprefixroute dynamic ens33
       valid_lft 1249sec preferred_lft 1249sec
    inet6 fe80::e91f:f60b:60f3:b9a2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

注意，输出更简洁：它不显示以正常或其他方式处理的数据包计数。

所以，可以增加参数选项 `-s` (`-stats`,`-statistics`):

```
[root@localhost /]# ip -s addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
    RX: bytes  packets  errors  dropped overrun mcast   
    0          0        0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    0          0        0       0       0       0       
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:24:c1:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.129/24 brd 192.168.31.255 scope global noprefixroute dynamic ens33
       valid_lft 1097sec preferred_lft 1097sec
    inet6 fe80::e91f:f60b:60f3:b9a2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    RX: bytes  packets  errors  dropped overrun mcast   
    25811560   22228    0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    597344     6183     0       0       0       0  
```

不过，或许更想看到的是这个：

```
[root@localhost /]# ip -stats -color -human addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
    RX: bytes  packets  errors  dropped overrun mcast   
    0          0        0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    0          0        0       0       0       0       
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:24:c1:38 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.129/24 brd 192.168.31.255 scope global noprefixroute dynamic ens33
       valid_lft 1033sec preferred_lft 1033sec
    inet6 fe80::e91f:f60b:60f3:b9a2/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    RX: bytes  packets  errors  dropped overrun mcast   
    25.8M      22.3k    0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns 
    604k       6.23k    0       0       0       0 
```

可以看到通过这种方式显示的数据更加直观，并且增加了 `-color` 选项后对 `ip` 地址做了不同颜色的高亮显示。

如果觉得命令 `ip -stats -color -human addr` 太长可以直接使用如下缩写命令：

```
ip -s -c -h a
```


总而言之，现在 `ifconfig` 命令已经不推荐使用，并且快要被废弃。


- **Link：** [Unix & Linux: ifconfig command not found?](https://unix.stackexchange.com/questions/145447/ifconfig-command-not-found)

---
