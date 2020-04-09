title: vim插件——rainbow
tags: vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《rainbow 官方文档》](https://github.com/luochen1990/rainbow "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 简介
* **插件介绍**：彩色显示括号对。
* **仓库地址**：<https://github.com/luochen1990/rainbow>

# 2 安装
* `$vim ~/.vimrc`
* 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'luochen1990/rainbow'`
* `:wq`
* `$vim`
* `:PluginInsttall`


# 3  使用
* 在`~/.vimrc`中添加`let g:rainbow_active = 1`

# 4 选项
* `let g:rainbow_active = 1`
&emsp;&emsp;自动启用插件，如果使用命令`:RainbowToggle`启用插件，则设为0
* `let g:rainbow_conf ={默认值}`
  * 配置高亮颜色，要高亮的括号对
  * 'guifgs'：GUI界面的括号颜色(将按顺序循环使用)
  * 'ctermfgs'：终端下的括号颜色(同上,插件将根据环境进行选择)
  * 'operators'：与同级括号一起高亮的运算符(`'_,\\|+\\|-_'` ,、 +、 -)
  * 'parentheses'：描述哪些模式将被当作括号处理,每一组括号由两个vim正则表达式描述
  * 'separately': 针对文件类型(由&filettype决定)作不同的配置,未被单独设置的文件类型使用\*下的配置,值为0表示仅对该类型禁用插件
  * 省略某个字段则使用默认设置
  * 示例：
```vim
let g:rainbow_conf = {
\	'guifgs': ['royalblue3', 'darkorange3', 'seagreen3', 'firebrick'],
\	'ctermfgs': ['lightblue', 'lightyellow', 'lightcyan', 'lightmagenta'],
\	'operators': '_,\|+\|-_',
\	'parentheses': ['start=/(/ end=/)/ fold', 'start=/\[/ end=/\]/ fold', 'start=/{/ end=/}/ fold'],
\	'separately': {
\		'*': {},
\		'tex': {
\			'parentheses': ['start=/(/ end=/)/', 'start=/\[/ end=/\]/'],
\		},
\		'css': 0,
\	}
\}
```


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《rainbow 官方文档》](https://github.com/luochen1990/rainbow "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
