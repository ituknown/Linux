网络驱动不属于 Debian 的官方软件，因此虽然 Debian 的系统镜像 ISO 文件中有许多 deb 软件包，但因为网络驱动不属于官方软件，所以在安装系统时因为缺少了相应的驱动就导致系统安装后无法上网的问题。

我这里以 Debian 11（代号为 bullseye）为例。安装后就会有因为没有网络驱动而无法上网。尤其是在笔记本上，因为通常都是使用 WI-FI 连接网络遇到这种问题就直接歇逼了（截图如下）。直接提示你没有 Adapter，你说气不气？

![show-setting-wifi-164225356031Ch2u](http://linux-media.knowledge.ituknown.cn/NetworkManager/Debian-NoNetworkDriver/show-setting-wifi-164225356031Ch2u.png)

解决方式也很简单，安装驱动即可！

但是怎么找网络驱动呢？

其实在 Debian 的软件库中有一个叫非官方软件的软件仓库，对应的地址是：https://cdimage.debian.org/cdimage/unofficial/。而我们的网络驱动就不属于官方软件包，所以如果你想要什么驱动的话就到这个仓库下找即可，如果真的找不到再去驱动官网去找。

同样的，这里还牵扯到 free 和 non-free 的问题，Debian 号称自由软件，所以对应的就是 free，因此只要不是 Debian 官方软件都属于 non-free。这一点可以在 Debian 的系统 ISO 文件中可以体现出来，在它的 ISO 文件中固件（驱动）对应的目录叫 firmware-free：

![debian11-iso-deb-1642254184zV3DpY](http://linux-media.knowledge.ituknown.cn/NetworkManager/Debian-NoNetworkDriver/debian11-iso-deb-1642254184zV3DpY.png)

网络驱动属于 firmware，因此我们需要到 Debian 的固件目录下下载，地址是：https://cdimage.debian.org/cdimage/unofficial/non-free/firmware/。

在该目录下会有对应的操作系统版本，文件夹的名称是对应版本的代号。比如 Debian11 的代号是 bullseye，所以我到该目录下下载即可。需要特别强调的是，虽然 Debian11 的代号是 bullseye，但是这个大版本下会有许多小版本，比如 11.0、11.1。所以在 firmware 目录下通常还会分具体的小版本。我们找某个软件包应该优先到具体的小版本下找，如果没有找到需要的软件包再考虑去其他小版本下找，防止存在兼容性问题。

Debian 的驱动程序都被打包成一个 firmware.zip 文件（这个包里面有许多驱动，但并不是所有的驱动都在里面，如果没有你需要的驱动可以去驱动官网下载）。比如我的 bullseye 小版本是 11.0，对应的 firmware.zip 地址就是 https://cdimage.debian.org/cdimage/unofficial/non-free/firmware/bullseye/11.0.0/firmware.zip 了，下载即可。

下载完成之后借助 U 盘拷贝到我们的 Debian 系统中，然后解压。解压后会看到许多 deb 软件包，不需要全部安装。我们只需要找我们需要的驱动程序即可，执行下面的命令来确定我们的网络驱动信息：

```bash
$ lspci | grep -i 'Ethernet|Network'
```

输出信息如下：

```
02:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 0c)
03:00.0 Network controller: Intel Corporation Wireless 3165 (rev 81)
```

因此我需要的网卡是驱动是 Inet 的 Realtek，另外还需要一个 WI-FI 驱动模块，解压 firmware.zip 后找到我自己需要的软件包即可，比如我这里需要的是  `firmware-realtek_20210315-3_all.deb` 和 `firmware-iwlwifi_20210315-3_all.deb`：

![firmware-driver-list-16422556713Fzxyk](http://linux-media.knowledge.ituknown.cn/NetworkManager/Debian-NoNetworkDriver/firmware-driver-list-16422556713Fzxyk.png)

之后拷贝到一个临时目录双击即可安装就好了（需要超级管理员权限），或者直接使用 `dpkg` 命令安装：

```bash
$ sudo dpkg -i firmware-realtek_20210315-3_all.deb firmware-iwlwifi_20210315-3_all.deb
```

安装完成后重启就可以看到 WI-FI 正常了：

![show-desktop-after-install-driver-16422558589Wybrr](http://linux-media.knowledge.ituknown.cn/NetworkManager/Debian-NoNetworkDriver/show-desktop-after-install-driver-16422558589Wybrr.png)

或者到 Setting 里面看到 WI-FI 显示信息：

![show-setting-wifi-after-install-driver-1642255884SXZ9pe](http://linux-media.knowledge.ituknown.cn/NetworkManager/Debian-NoNetworkDriver/show-setting-wifi-after-install-driver-1642255884SXZ9pe.png)

这就大功告成了~