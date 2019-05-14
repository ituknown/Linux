```
$ wget https://github.com/antirez/redis/archive/5.0.4.tar.gz
```

```
$ make

......
make[2]: 离开目录“/opt/redis-4.0.0/deps”
make[1]: [persist-settings] 错误 2 (忽略)
    CC adlist.o
/bin/sh: cc: 未找到命令
make[1]: *** [adlist.o] 错误 127
make[1]: 离开目录“/opt/redis-4.0.0/src”
make: *** [all] 错误 2
......
```

```
$ yum -y install gcc automake autoconf libtool make 
```

```
$ make

......
cd src && make all
make[1]: 进入目录“/opt/redis-4.0.2/src”
    CC Makefile.dep
make[1]: 离开目录“/opt/redis-4.0.2/src”
make[1]: 进入目录“/opt/redis-4.0.2/src”
    CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:31: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录
 #include <jemalloc/jemalloc.h>
                               ^
编译中断。
make[1]: *** [adlist.o] 错误 1
make[1]: 离开目录“/opt/redis-4.0.2/src”
make: *** [all] 错误 2
......
```

分配器allocator， 如果有MALLOC  这个 环境变量， 会有用这个环境变量的 去建立Redis。

而且libc 并不是默认的 分配器， 默认的是 jemalloc, 因为 jemalloc 被证明 有更少的 fragmentation problems 比libc。

但是如果你又没有jemalloc 而只有 libc 当然 make 出错。 所以加这么一个参数,运行如下命令：

```
$ make MALLOC=libc
```
