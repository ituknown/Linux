##### 查看交换区

Linux 查看内存空间最常用的命令是 `free`，该命令不仅能列出 `RAM` 内存使用情况还能显示出交换区（`Swap`）的使用情况。

```bash
$ free -hm
              total        used        free      shared  buff/cache   available
Mem:           1.8G        221M        612M        8.6M        989M        1.4G
Swap:          2.0G         40K        2.0G
```

`Mem` 指的是 `RAM` 内存的详细信息，`Swap` 指的是交换区的详细信息。

`Swap` 所使用的内存并不是 `RAM` 的内存，而是在物理磁盘上划出的一部分物理页。

| 关于交换区(`swap`)                                           |
| ------------------------------------------------------------ |
| Linux 把物理内存划分称作为分页（Page）的内存区块。内存交换是一个内存分页被复制到一个预配置的称为 swap 空间的硬盘空间里的过程，以此来释放内存分页。物理内存与这个 swap 空间的共同大小称为可用的虚拟内存量。虽然 swap分区可以增大物理内存大小的限制，但是如果由于内存不足而使用到 swap分区，会增加系统的响应时间，性能变差。因此在物理内存充足或者性能敏感的系统中，不建议配置 swap 分区。 |

除了使用 `free` 命令，也可以直接使用 `swapon` 命令查看已经启用的交换区及状态。

```bash
$ swapon 
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G  40K   -1
```



#### 永久关闭交换区

想要永久关闭交换区（`swap`）可以使用超级管理员命令编辑 `/etc/fstab` 文件。

```bash
sudo vim /etc/fstab
```

然后将所有带 `swap` 字符的行都注释或删掉即可。

示例，当前 `/fstab` 文件内容：

```shell
# /etc/fstab
# Created by anaconda on Fri Nov 20 13:16:42 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=a1987c5c-fe3b-423e-b377-fd5623bd3aa9 /boot                   xfs     defaults        0 0
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
```

这里带 `swap` 字符的行是：

```
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
```

直接删掉或注释掉保存即可：

```shell
# /etc/fstab
# Created by anaconda on Fri Nov 20 13:16:42 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=a1987c5c-fe3b-423e-b377-fd5623bd3aa9 /boot                   xfs     defaults        0 0
# /dev/mapper/cl-swap     swap                    swap    defaults        0 0
```

由于修改是配置文件需要关机重启（使用命令 `reboot` 或 `shutdown now -r`）才能生效。

重启完成后再看交换区状态：

```bash
$ free -hm
              total        used        free      shared  buff/cache   available
Mem:           1.8G        252M        1.4G        9.5M        162M        1.4G
Swap:            0B          0B          0B
```

现在再看交换区就会发现全是0，即表示交换区已经被关闭了。

使用 `swapon` 命令更加直观，如果没有任何输出信息即表示成功关闭交换区。

```bash
$ swapon
```



#### 临时关闭交换区

临时关闭交换区只能一次性生效，优点是无需关机重启，执行后立即生效：

```bash
sudo swapoff -a
```

命令执行完成之后可以使用 `free` 命令或 `swapon` 命令进行确认。