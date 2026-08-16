---
title: vim插件——vim-colors-solarized 
tags: vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《vim-colors-solarized 官方文档》](https://github.com/altercation/vim-colors-solarized "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>




# 1 简介

* **插件介绍**：vim主题。
* **仓库地址**：<https://github.com/altercation/vim-colors-solarized>

# 2 安装
* `$vim ~/.vimrc`
* 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'altercation/vim-colors-solarized'`
* `:wq`
* `$vim`
* `:PluginInsttall`


# 3 使用

* 如果要使用256色，且终端支持256色
```shell
#在`~/.bashrc`中添加代码：

if [ -e /lib/terminfo/x/xterm-256color ]
then
	export TERM='xterm-256color'
else
	export TERM='xterm-color'
fi


# 在`~/.vimrc`中添加代码：

if has("gui_runing")              "如果在图形用户界面
    set background=light          "设置背景色为亮色
else                              "设置背景色为深色
    set background=dark           "就开启256色支持，默认为8色  
    set t_Co=256
endif
```
* 在`~/.vimrc`中添加
```shell
syntax enable
colorscheme solarized
```
# 4 选项
* `let g:solarized_termcolors=256`
设置颜色为256色，默认为16

* `let g:solarized_termtrans=0`
设置背景色为透明，使用终端的背景色

* `let g:solarized_degrade=0`
强制使用256级色彩，只用于测试，默认为0

* `let g:solarized_bold=1`
使用粗体，默认为1

* `let g:solarized_underline=1`
使用下划线，默认为1

* `let g:solarized_italic=1`
使用斜体，默认为1

* `let g:solarized_contrast="normal"`
设置对比度，默认为normal，还可以设置为low或者high

* `let g:solarized_visibility="normal"`
设置空白符的可见性，默认为normal，还可以设置为low或者high

* `letg:solarized_hitrail=0`
光标高亮时，空白符仍可见，默认为0

* `let g:solarized_menu=1`
启用solarized_menu菜单，默认为1




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《vim-colors-solarized 官方文档》](https://github.com/altercation/vim-colors-solarized "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
