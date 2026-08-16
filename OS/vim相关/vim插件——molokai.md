---
title: vim插件——molokai
tags: vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《molokai 官方文档》](https://github.com/tomasr/molokai "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>




# 1 简介

* **插件介绍**：vim主题。
* **仓库地址**：<https://github.com/tomasr/molokai>

# 2 安装
* `$vim ~/.vimrc`
* 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'tomasr/molokai'`
* `:wq`
* `$vim`
* `:PluginInsttall`


# 3 使用

* 如果要使用256色，且终端支持256色
```bash
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
* `let g:molokai_original = 0`
  使用原始molokai配色，默认为0
  
* `let g:rehash256 = 0`
 使用接近图形界面用的配色，默认为0






------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《molokai 官方文档》](https://github.com/tomasr/molokai "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
