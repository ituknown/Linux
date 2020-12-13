# 使用 `uname` 命令查看系统版本

`uname` 命令可以显示系统的多个信息，包括 Linux 内核体系架构、版本名称和发行版。

查看系统全部信息：

```bash
$ uname -a

Linux Debian-VM1 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64 GNU/Linux
```

只查看内核版本：

```bash
$ uname -srm

Linux 4.19.0-13-amd64 x86_64
```

# 使用 `hostnamectl` 命令查看系统版本

`hostnamectl` 工具是 systemd 的一部分，用于查询和更改信息主机名。它还显示 Linux 发行版和内核版本：

```bash
$ hostnamectl 

   Static hostname: Debian-VM1
         Icon name: computer-vm
           Chassis: vm
        Machine ID: e7ad641889044f5cab8b0070353ea602
           Boot ID: b512a45e2d3e4c51b3b5d4f079d84917
    Virtualization: vmware
  Operating System: Debian GNU/Linux 10 (buster)
            Kernel: Linux 4.19.0-13-amd64
      Architecture: x86-64
```

很直观的就能看出主机名为 `Debian-VM1`，操作系统的 Debian 10，系统架构是 x86-64。该命令比 `uname` 命令更加直观。

比如只看内核信息：

```bash
$ hostnamectl | grep -i Kernel

Kernel: Linux 4.19.0-13-amd64
```

# 使用 `lsb_release` 查看发行信息

`hostnamectl` 命令展示的信息很多，包括发行版本和内核信息。不过如果只是想查看发行版本的话也可以直接使用 `lsb_release`：

```bash
$ lsb_release -a

No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 10 (buster)
Release:	10
Codename:	buster
```

这个就只是展示发行版本信息，更加简单直观。

# `/proc` 目录

Linux系统上的 `/proc` 目录是一种文件系统，即 `proc` 文件系统。与其它常见的文件系统不同的是，`/proc` 是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。

我们可以查看一下该文件夹中的文件信息：

```bash
$ ls /proc/

1    141  1623  20   238  29    33   372  409  442   5154  55   6   66  73  8   86      buddyinfo  devices      fs          keys         locks    net           softirqs       timer_list
10   142  1677  200  25   3     34   391  417  443   5177  551  60  67  74  80  87      bus        diskstats    interrupts  key-users    meminfo  pagetypeinfo  stat           tty
11   143  17    21   26   30    35   393  427  4935  5192  552  61  68  75  81  88      cgroups    dma          iomem       kmsg         misc     partitions    swaps          uptime
12   146  19    22   265  31    351  396  428  5110  533   56   62  69  76  82  9       cmdline    driver       ioports     kpagecgroup  modules  sched_debug   sys            version
139  148  195   228  27   3170  36   397  429  5117  534   57   63  70  77  83  98      consoles   execdomains  irq         kpagecount   mounts   schedstat     sysrq-trigger  vmallocinfo
14   15   199   230  28   32    37   4    431  5118  539   58   64  71  78  84  acpi    cpuinfo    fb           kallsyms    kpageflags   mpt      self          sysvipc        vmstat
140  16   2     231  284  3258  371  404  440  5141  545   59   65  72  79  85  asound  crypto     filesystems  kcore       loadavg      mtrr     slabinfo      thread-self    zoneinfo
```

该文件夹下有许多数值文件，比如 141，这里的 141 文件夹其实是进行号为 141 的文件信息。

除了一些数值文件外还有其他的文件，如 version、cpuinfo 等文件。这些文件都是字面上的意思，如 version 指的就是系统的基本信息，cpuinfo 指的则是 cpu 信息。

## 查看 `/proc/version` 文件确认内核版本

该文件展示的信息与 `uname` 命令基本相同：

```bash
$ cat /proc/version 

Linux version 4.19.0-13-amd64 (debian-kernel@lists.debian.org) (gcc version 8.3.0 (Debian 8.3.0-6)) #1 SMP Debian 4.19.160-2 (2020-11-28)
```

虽然很详细，但是都不如 `hostnamectl` 命令直观。

## 查看 `/proc/cpuinfo` 文件查看 CPU 信息

```bash
$ cat /proc/cpuinfo 

processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 158
model name	: Intel(R) Core(TM) i5-7400 CPU @ 3.00GHz
stepping	: 9
microcode	: 0xb4
cpu MHz		: 2999.999
cache size	: 6144 KB
physical id	: 0
siblings	: 2
core id		: 0
cpu cores	: 2
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 22
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc cpuid pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp fsgsbase tsc_adjust bmi1 avx2 smep bmi2 invpcid mpx rdseed adx smap clflushopt xsaveopt xsavec xsaves arat md_clear flush_l1d arch_capabilities
bugs		: cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs itlb_multihit srbds
bogomips	: 5999.99
clflush size	: 64
cache_alignment	: 64
address sizes	: 43 bits physical, 48 bits virtual
power management:

processor	: 1
vendor_id	: GenuineIntel
cpu family	: 6
model		: 158
model name	: Intel(R) Core(TM) i5-7400 CPU @ 3.00GHz
stepping	: 9
microcode	: 0xb4
cpu MHz		: 2999.999
cache size	: 6144 KB
physical id	: 0
siblings	: 2
core id		: 1
cpu cores	: 2
apicid		: 1
initial apicid	: 1
fpu		: yes
fpu_exception	: yes
cpuid level	: 22
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc cpuid pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp fsgsbase tsc_adjust bmi1 avx2 smep bmi2 invpcid mpx rdseed adx smap clflushopt xsaveopt xsavec xsaves arat md_clear flush_l1d arch_capabilities
bugs		: cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs itlb_multihit srbds
bogomips	: 5999.99
clflush size	: 64
cache_alignment	: 64
address sizes	: 43 bits physical, 48 bits virtual
power management:
```