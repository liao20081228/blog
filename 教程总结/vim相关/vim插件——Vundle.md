---
title: vim插件——Vundle 
tags:  vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Vundle 官方文档》](https://github.com/VundleVim/Vundle.vim.git "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



# 1 简介
* **插件介绍**：管理vim插件。
* **仓库地址**：<https://github.com/VundleVim/Vundle.vim.git>

# 2 安装

* 下载代码仓库
```
$git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
$vim ~/.vimrc
```
* 在`~/.vimrc`添加以下内容：
```
set nocompatible 
filetype off 
set rtp+=~/.vim/bundle/Vundle.vim 
call vundle#begin() 
Plugin 'VundleVim/Vundle.vim'
call vundle#end() 
filetype plugin indent on 
```
* 保存但不退出vim
* 安装插件管理器Vundle
```
:PluginInstall
```
* 显示done则成功

# 3 使用
## 3.1 安装插件
 * 在~/.vimrc文件中加上下面的语句，保存后执行执行PluginInstall即可自动下载并安装插件。
```vim-scripts
set nocompatible              "不与vi兼容
filetype off                  "关闭打开文件类型检测
set runtimep+=~/.vim/bundle/Vundle.vim 
call vundle#begin() 
Plugin 'VundleVim/Vundle.vim'
plugin '用户名/插件仓库名'          "GitHub上的插件
Plugin 'vim-scripts/插件仓库名'     "来自vim-scripts.org的插件
Plugin 'git clone 后面的地址'        "由Git支持但不再github上的插件仓库
Plugin 'file:///+本地插件仓库绝对路径'  "本地的Git仓库(例如自己的插件)
Plugin 'rstacruz/sparkup', {'rtp':'vim/'} "插件在仓库的子目录vim/中
... ...
call vundle#end() 
filetype plugin indent on           "打开文件类型检测，并加载相关插件，根据文件类型设置缩进距离
```
* 可以先用git clone克隆仓库源码到 ~/.vim/bundle 文件夹， 或者直接将别人下载好的仓库拷贝到~/.vim/bundle，直接安装。

## 3.2 卸载插件
* 先在`~/.vimrc`中删除相应插件
* 然后输入vim,再输入`:BundleClean`

## 3.3 Vundle常用命令
```
BundleList         -列举列表(也就是.vimrc)中配置的所有插件  
BundleInstall      -安装列表中的全部插件  
BundleInstall!     -更新列表中的全部插件  
BundleSearch foo   -查找foo插件  
BundleSearch! foo  -刷新foo插件缓存  
BundleClean        -清除列表中没有的插件  
BundleClean!       -清除列表中没有的插件
```


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Vundle 官方文档》](https://github.com/VundleVim/Vundle.vim.git "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
