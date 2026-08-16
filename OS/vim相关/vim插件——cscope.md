---
title: vim插件——cscope
tags: vim插件
---

------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>《cscope man文档 v15.9》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------



# 1 简介
&emsp;&emsp;cscope 是一个 C语言的浏览工具，通过这个工具可以很方便地找到某个函数或变量的定义位置、被调用的位置等信息。目前支持 C 和 C++。cscope 自身带一个基于文本的用户界面，不过 gvim 提供了cscope接口，因此可以在 gvim 中调用 cscope，方便快捷地浏览源代码。

&emsp;&emsp;cscope是一个类似ctags的工具。 你可以把它想作是超过频的ctags，因为它功能比ctags强大很多。 在 Vim里，通过cscope查询结果来跳转就象跳转到其他的标签完全一样；它被保存在标签栈里。这样你就可以象使用tags一样在函数等等之间便捷的跳转。在VIM中使用cscope非常简单，首先调用`cscope add cscope.out`命令添加一个cscope数据库，然后就可以调用`cscope find`命令进行查找了。VIM支持8种cscope的查询功能，然后列出目标出现的所有位置。我们还可以进行字符串查找，它会在双引号或单引号括起来的内容中查找。还可以输入一个正则表达式，这类似于egrep程序的功能。

# 2 安装
* 手动编译安装
	* 从<http://cscope.sourceforge.net/>下载源码
	* 解压后进入源码根目录
	* 配置：`./configure --with-flex ` (注：如果平台是Linux,最好带上 --with-flex选项)
	* 编译：`make ` (注：我没有遇到错误)
	* 安装：`make install `
* 下载已编译的二进制包
	* `sudo apt install cscope` 

>**注意**：cscope即可以单独使用，也可以在作为vim的一个插件来使用，但要求vim编译时配置了`--enable-cscepe`特性。可以使用`$ vim --version |grep cscepe`来查看是否支持。


# 3 配置
* 修改vim配置文件vimrc.你可以修改/etc/vimrc使用所有用户都使用本配置文件，当然你还可以修改~/.vimrc
* 下载配置文件：[cscope_map.vim ](http://cscope.sourceforge.net/cscope_maps.vim) 
* 把cscope_map.vim里从 if has("cscope")  到 endif里边的内容复制到`/etc/vimrc`里边去，也可以自行修改。

# 4 使用
* 建立数据库：`cscope -Rbq`
* 进入vim
* 添加数据库: `:cscope|cs add file|dir [pre-path] [flags])`
* 搜索: `:cscope|cs find  {querytype} {name}`，除了 4 和 6 之外的类型忽略 name之前的空格。4 和 6 在 querytype和name 之间只能有一个空格分隔。多余的空格是 name的一部分。
	* 0 或 s: 查找 C 语言标识符
	* 1 或 g: 查找函数、宏、枚举等定义的位置，
	* 2 或 d: 查找函数调用的函数
	* 3 或 c: 查找调用本函数的函数
	* 4 或 t: 查找字符串
	* 6 或 e: 查找符合egrep 模式的字符串
	* 7 或 f: 查找文件
	* 8 或 i: 查找#include本文件的文件
	* 9 或 a: 查找变量被赋值的位置
* 显示帮助: `:cscope|cs help`
* 断开连接: `:cscope|cs kill #`
* 重置连接:` :cscope|cs reset`
* 显示连接:` :cscope|cs show`

# 5 cscope命令详解
 
 请查看《源码搜索工具cscope》。
 
------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>《cscope man文档 v15.9》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

