主要语法如下：

```bash
mount [options] <source> <directory>
```

`mount` 命令所表示的含义就是将 `<source>` 挂载到 `<directory>`，可以理解为 `<directory>` 就是进入 `<source>` 的入口。

来看下 `<source>` 可以有哪些：

```bash
LABEL=<label>           文件设备的 label
UUID=<uuid>             文件设备的 UUID
PARTLABEL=<label>       文件设备的 PARTLABEL
PARTUUID=<uuid>         文件设备的 PARTUUID
<device>                设备(如 /dev/sda1)
<directory>             磁盘目录
<file>                  磁盘文件
-L,--label <label>      同 LABEL=<label>
-U,--uuid <uuid>        同 UUID=<uuid>
```

`<device>`、`<directory>` 和 `<file>` 很好理解，关键的是 LABEL、UUID 这些是什么？

其实这些 UUID、LABEL 就是磁盘文件设备的一些属性数据（可以参考下 [磁盘分区管理](磁盘分区管理.md)），我们可以使用 `blkid` 命令查看：

```bash
$ sudo blkid

/dev/sdb1: UUID="D8C7-8235" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="8591882f-1b8b-44ed-a33b-f539f356e802"
/dev/sdb2: UUID="042055c6-546b-4889-8d3f-3b7aec794412" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="016e491b-23c1-45f0-99a4-7a6d440e1aac"
/dev/sdb3: UUID="660f3d0e-0a77-4d60-8532-b867c3a9e0d8" TYPE="swap" PARTUUID="35cd0ea7-73db-463e-aad9-136e916b01c9"
...
/dev/sdc1: BLOCK_SIZE="2048" UUID="2022-04-19-10-23-19-00" LABEL="Ubuntu 22.04 LTS amd64" TYPE="iso9660" PARTLABEL="ISO9660" PARTUUID="a09db2b8-b5f6-43ae-afb2-91e0a90189a1"
/dev/sdc2: SEC_TYPE="msdos" LABEL_FATBOOT="ESP" LABEL="ESP" UUID="8D6C-A9F8" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="Appended2" PARTUUID="a09db2b8-b5f6-43ae-afb1-91e0a90189a1"
/dev/sda1: PARTUUID="c0ebb0b6-fab0-8d4d-8469-dae82cc313a1"
/dev/sda2: PARTUUID="8ca20862-a47d-b84f-bf07-12a93afe2ab5"
/dev/sda3: PARTUUID="63823989-e654-164b-a314-6b18b78769bf"
/dev/sdc3: PARTLABEL="Gap1" PARTUUID="a09db2b8-b5f6-43ae-afb0-91e0a90189a1"
```

可以看到，当我们使用 `blkid` 命令查看磁盘分区表时就会显示分取消对应的 UUID、LABEL、PARTUUID 以及 PARTLABEL。在实际中根据自己喜好选择，个人更喜欢使用分区表的 PARTUUID。


```
Options:
 -a, --all               mount all filesystems mentioned in fstab
 -c, --no-canonicalize   don't canonicalize paths
 -f, --fake              dry run; skip the mount(2) syscall
 -F, --fork              fork off for each device (use with -a)
 -T, --fstab <path>      alternative file to /etc/fstab
 -i, --internal-only     don't call the mount.<type> helpers
 -l, --show-labels       show also filesystem labels
 -n, --no-mtab           don't write to /etc/mtab
     --options-mode <mode>
                         what to do with options loaded from fstab
     --options-source <source>
                         mount options source
     --options-source-force
                         force use of options from fstab/mtab
 -o, --options <list>    comma-separated list of mount options
 -O, --test-opts <list>  limit the set of filesystems (use with -a)
 -r, --read-only         mount the filesystem read-only (same as -o ro)
 -t, --types <list>      limit the set of filesystem types
     --source <src>      explicitly specifies source (path, label, uuid)
     --target <target>   explicitly specifies mountpoint
     --target-prefix <path>
                         specifies path use for all mountpoints
 -v, --verbose           say what is being done
 -w, --rw, --read-write  mount the filesystem read-write (default)
 -N, --namespace <ns>    perform mount in another namespace

 -h, --help              display this help
 -V, --version           display version

```