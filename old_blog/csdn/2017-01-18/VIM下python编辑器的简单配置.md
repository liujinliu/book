参考原文：http://blog.csdn.net/dyllove98/article/details/9750251
这里说下vim下python编辑器的简单配置
### 安装ctags
```
$ apt-get install ctags
```
### 安装taglists
首先安装vim-scripts，vim-scripts中带有vim-addon-manager，vim-addon-manager是Ubuntu众多插件管理器之一，用来管理vim插件。通过vim-addon-manager安装taglist。
```
$ apt-get install vim-scripts
$ vim-addons install taglist
```
### 安装高亮显示
将下载的hightlight.vim拷贝到 ~/.vim/plugin 目录下。
```
$ cp hightlight.vim ~/.vim/plugin
```

高亮搜索结果命令 :set hlsearch，使用命令 :hi Search查看高亮背景色，默认棕黄色，更改高亮背景色命令 :hi Search guibg=LightBlue。

临时关闭高亮命令 :nohlsearch，该命令可简写为 :noh。

###  taglist

简介：显示标签列表。

下载：参看前述。

安装：参看前述。

默认关闭taglist，在.vimrc写入：

let Tlist_Auto_Open=0

在正常编辑区域和tags区域切换命令 :ctrl+w+w。

TlistToggle：开关taglist。

<CR>：跳转至tag定义处。

o：在新窗口中显示光标下的tag。

u：更新taglist窗口中的tag。

s：更改排序方式，名字排序或行号排序。

X：taglist窗口放大缩小。

+：打开折叠，等同zo。

-：关闭折叠，等同zc。

*：打开所有折叠，等同zR。

=：将所有tag折叠，等同zM。

[[：跳转至前一个文件。

]]：跳转至后一个文件。

q：关闭taglist窗口。

<F1>：显示帮助。

### 折叠代码

简介：将Python代码折叠，Python的class，function，以及在{{{,}}}标记的内容将被折叠。

下载：http://vim.sourceforge.net/scripts/script.php?script_id=515

安装：

将下载的python_fold.vim拷贝到 ~/.vim/plugin 目录下。

关闭开启时默认折叠命令，在.vimrc写入：

set nofoldenable

zo: 展开单个折叠区。

zc: 聚合单个折叠区。

zn: 展开全部折叠区。

zN: 聚合全部折叠区。

### NERDTree目录树

简介：打开文件目录树，相当于文件浏览器。

下载：http://vim.sourceforge.net/scripts/script.php?script_id=515

安装：

将整个解压后的源包拷贝到 ~/.vim 目录下，需要确保 NERD_tree.vim 位于 ~/.vim/plugin 目录下， NERD_tree.txt 位于 ~/.vim/doc 目录下。

使用<F7>作为快捷键开关目录树，在.vimrc写入： 

map <F7> :NERDTreeToggle<CR>
### 配置文件内容
```
let Tlist_Auto_Highlight_Tag=1
let Tlist_Auto_Open=1
let Tlist_Auto_Update=1
let Tlist_Display_Tag_Scope=1
let Tlist_Exit_OnlyWindow=1
let Tlist_Enable_Dold_Column=1
let Tlist_File_Fold_Auto_Close=1
let Tlist_Show_One_File=1
let Tlist_Use_Right_Window=1
let Tlist_Use_SingleClick=1
nnoremap <slient> <F8> :TlistToggle<CR>

set tags=tags;

set autoindent
set tabstop=4
set shiftwidth=4
set expandtab
set nu!
```

  