# 前言

在 Linux 下, Vim 的全局配置文件一般是 `/etc/vimrc` 文件或者 `/etc/vim/vimrc`。

在实际中不推荐去修改全局的 Vim 配置, 如果想要做定制话配置推荐修改用户个人的 Vim 配置：`~/.vimrc`。

下面是 Vim 配置文件的优先级别（从高到底）：


|**级别**|**文件**|
|:--|:----|
|用户|`~/.vimrc`|
|系统|`/etc/vimrc`|
|系统|`/etc/vim/vimrc`|


需要说明的一点时，在 vimrc 中注释使用的是英文 `"` 符号。在大多数情况下如果想要关闭某个配置只需要在前面加上 `no` 前缀即可，如关闭行好：`set nonumber`。

下面是配置说明：


# 配置示例

```vim
" 显示行号
set number "同 set nu

" 默认不显示行号, 如果显示设置不显示行号只需要在前面加上 no 前缀即可. 示例:
"set nonumber

" 突出当前行
set cursorline "同 set cul

" 突出当前列
"set cursorcolumn "同 set cuc

" 状态栏显示行列
set ruler

" 开启自动保存
set autowrite

" 高亮显示搜索结果
set hlsearch

" 搜过到最后匹配的位置时, 再次搜索不回到第一个匹配出
set nowrapscan

" 在处理未保存或只读文件时, 弹出确认
set confirm

" 高亮显示括号匹配
set showmatch

" 设置 TAB 长度未指定空格
set tabstop=4

" 设置自动缩进长度未指定空格
set shiftwidth=4

" 使用空格替代制表符
set expandtab

" 语法高亮
syntax on

" 在VIM底部显示模式, 如插入模式
set showmode

" 显示当前插入的明来, 如 2yy
set showcmd

" 设置编码格式
set encoding=utf-8

" 启用256色
set t_Co=256

" 开启文件类型检查, 并且载入与该类型对应的缩进规则. 比如编辑的
" 是.py文件, Vim 就是会找 Python 的缩进规则 ~/.vim/indent/python.vim
"filetype indent on

" 设置字体
"set guifont="JetBrains Mono"

" 记录命令条数(与之后的撤销有关)
set history=1000

" 允许撤销次数
set undolevels=1000

" 发生错误时, 视觉提示(通常是屏幕闪烁)
"set visualbell

" 自动删除行尾的空格和制表符
autocmd BufWritePre *.c :%s/\s\+$//e
```
