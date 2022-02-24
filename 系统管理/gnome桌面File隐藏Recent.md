先看下 gnome 默认 File 展示的侧边栏信息：

![gnome-desktop-default](http://linux-media.knowledge.ituknown.cn/SystemManager/GnomeDesktop/gnome-desktop-default.png)

这个侧边栏上的 Recent 我怎么看怎么烦，暴露我隐私不是？

干掉他！

```bash
gsettings set org.gnome.desktop.privacy remember-recent-files false
```

然后就好了：

![gnome-desktop-disable-recent](http://linux-media.knowledge.ituknown.cn/SystemManager/GnomeDesktop/gnome-desktop-disable-recent.png)

--

可以看下 asciimena 演示实例：

[![asciicast](https://asciinema.org/a/v5vsO6os2Rjno02BOYVkQpsN3.svg)](https://asciinema.org/a/v5vsO6os2Rjno02BOYVkQpsN3)
