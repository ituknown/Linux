

```bash
$ ls -l /lib
lrwxrwxrwx 1 root root 7 Apr 21  2022 /lib -> usr/lib
```

/usr/lib/systemd/system 不要动
/etc/systemd/system 覆盖 /usr/lib/systemd/system

/run/systemd/system 系统运行时自动创建，重启丢失

```shell
[Section]
Directive1=value
Directive2=value

[X-Section]
Directive1=value
Directive2=value
```

```shell
[Unit]
Description=Some HTTP server
After=remote-fs.target sqldb.service
Requires=sqldb.service
```


```bash
$ ls -l /etc/systemd/system/multi-user.target.wants/

...
aria2.service -> /lib/systemd/system/aria2.service
console-setup.service -> /lib/systemd/system/console-setup.service
cron.service -> /lib/systemd/system/cron.service
dmesg.service -> /lib/systemd/system/dmesg.service
e2scrub_reap.service -> /lib/systemd/system/e2scrub_reap.service
grub-common.service -> /lib/systemd/system/grub-common.service
grub-initrd-fallback.service -> /lib/systemd/system/grub-initrd-fallback.service
```


```shell
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```