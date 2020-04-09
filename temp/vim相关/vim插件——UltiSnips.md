---
title: vim插件——ultisnips
tags: vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《ultisnips 官方文档》](https://github.com/SirVer/ultisnips "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



# 1 简介
* **插件介绍**：代码片补全。
* **仓库地址**：<https://github.com/SirVer/ultisnips>

# 2 安装教程
*  打开`~/.vimrc`
* 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'SirVer/ultisnips'`
* 保存并退出。
* 打开vim
* 在命令行中输入`PluginInsttall`
* 显示done之后，退出vim

>**注意vim版本必须大于7.4且支持python2或python3。**

# 3 基本配置
* 在`~/.vimrc`中添加以下内容
```vim
"设置vim为不兼容模式
set nocompatible

"设置触发键
let g:ultisnipsexpandtrigger="<tab>"      

"后一个
let g:ultisnipsjumpforwardtrigger="<c-j>"  

"前一个
let g:ultisnipsjumpbackwardtrigger="<c-k>"  

"打开补全列表
let g:ultisnipslistsnippets="<c-tab>" 

"指定snippets文件的目录，可以有多个且该目录runtimepath 中的某一项之下
"let g:ultisnipssnippetsdir='~/.vim/Ultisnips' 

"指定snippets文件的搜索路径
"let g:ultisnipssnippetdirectories=['ultisnips'] 

"设置窗口打开的方式：normal当前窗口 horizontal水平vertical垂直context智能 
let g:ultisnipseditsplit="normal" 


"指定ultisnip使用python2
let g:ultisnipsusepythonversion=2 
```

# 4 ultisnips中文手册
请查看《vim插件——ultisnips中文手册》一文。




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《ultisnips 官方文档》](https://github.com/SirVer/ultisnips "点击跳转")。。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
