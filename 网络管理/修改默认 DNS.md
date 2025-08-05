
### 使用 systemd-resolved（现代 Debian/Ubuntu 使用）

在修改之前，可以先查看下 DNS 信息：

```bash
systemd-resolve --status
```

#### 临时设置

直接使用命令：

```bash
sudo systemd-resolve --set-dns=8.8.8.8 --set-dns=8.8.4.4 --interface=eth0

# 或

sudo systemd-resolve --set-dns='8.8.8.8 8.8.4.4' --interface=eth0
```

1. `--set-dns=` 是要设置的 DNS 地址，可以指定多个。
2. `--interface=` 指定具体的网卡名（可以使用 `ip link` 确认网卡信息）。

命令行设置只能临时使用，服务器重启就会失效。

#### 永久设置

编解配置文件：

```bash
sudo vim /etc/systemd/resolved.conf
```

添加或修改：

```Plain
[Resolve]
DNS=8.8.8.8 8.8.4.4
FallbackDNS=1.1.1.1
```

1. `DNS=` 是主 DNS 服务器列表。
2. `FallbackDNS=` 是备选 DNS 服务器列表。

然后重启服务即可生效：

```bash
sudo systemctl restart systemd-resolved
```


### 使用 netplan（Ubuntu 18.04+）

Netplan 是 Ubuntu（从 17.10 开始）默认的网络配置工具，通过 YAML 文件配置，用于在系统启动时配置网络接口，最终将配置应用到后台的实际网络管理服务（如 systemd-networkd 或 NetworkManager）。

配置文件路径：

```
/etc/netplan/*.yaml
```

每台机器的 YAML 配置文件名都不相同，大多情况下都是如下两种：

1.  01-netcfg.yaml
2. 50-cloud-init.yaml

| **Note**                              |
| :------------------------------------ |
| 通过 netplan 修改 DNS 是永久生效，即使服务器重启也没有影响。 |

编辑配置文件：

```bash
sudo vim /etc/netplan/01-netcfg.yaml
```

在具体的网卡下面添加或修改如下内容（如果没有网卡配置，也可以按照下面形式添加）：

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: yes
+     nameservers:
+       addresses: [8.8.8.8, 8.8.4.4]
```

重启生效：

```bash
sudo netplan apply
```