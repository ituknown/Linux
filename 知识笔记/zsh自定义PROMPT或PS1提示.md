# 基本配置

zsh 提示配置的工作方式类似于bash，只是它使用不同的语法。zsh 与其他 shell 一样可以直接使用变量PS1来存储默认提示，除此之外，zsh 也可以用变量名 PROMPT 或 prompt 代替PS1。

当我们什么都不配置的时候默认是`%n@%m %1~ %#`：

```bash
export PROMPT='%n@%m %1~ %#'
```

其中：

- `%n` 是当前用户名
- `%m` 是当前主机名的第一元素
- `%1~` 是当前目录，不过会自动将用户目录替换为`~`

另外，`%#` 是zsh的提示符，默认提示符号是%，当具有超级用户权限时会显示`#`。

<img src="http://linux-media.knowledge.ituknown.cn/KnowledgeNotes/zsh_prompt/default_prompt.png" alt="default_prompt.png" width="500">

因此，如果想要完整的展示用户所在目录，只保留 `%~ %#` 就可以了（即`%1~` 替换为`%~` 就可以展示完整路径）：

```bash
ituknown@192 JetBrains %
ituknown@192 JetBrains %export PROMPT='%n@%m %~ %#'
ituknown@192 ~/JetBrains %   # 展示了完成路径
```

# 进阶配置

在提示中添加一点颜色、字体加粗更具有可读性：

```bash
export PROMPT='%F{13}%~ %F{50}%B%# %f%b'
```

其中：

- `%F{color}` 是配置颜色，{}中的color是256色的颜色值，也可以使用black、red、green、yellow、blue、magenta、cyan和white等常用色
- `%B` 表示使用粗体字
- `%f` 表示后面恢复默认颜色
- `%b` 表示后面恢复常规字体

<img src="http://linux-media.knowledge.ituknown.cn/KnowledgeNotes/zsh_prompt/advice_color.png" alt="advice_color.png" width="500">

另外，zsh 还定义了 git_prompt_info 变量，用于 git 信息提示，具体可以自行摸索。我常用的 PROMPT 是：

```bash
export PROMPT='%F{green}%B$ %f$(git_prompt_info)'
```

