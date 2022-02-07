---
title: bash
tags: shell,man1
---


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《bash manpages》。</font>bash的版本为5.0.17，手册更新时间为2018-12-7。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称
bash--GNU Bourne-Again Shell

# 总览
**bash** \[options] \[command_string | file]
# 版权
Bash 的版权©1989-2018由自由软件基金会公司所有。

# 描述
Bash 是一个 sh 兼容的命令语言解释器，它执行从标准输入或文件中读取的命令。 Bash 还结合了来自 Korn 和 C shell（ksh 和 csh）的有用特性。

Bash 旨在成为 IEEE POSIX 规范（IEEE 标准 1003.1）的 Shell 和 Utilities 部分的一致实现。 默认情况下，Bash配置为符合 POSIX 标准。

# 选项
内置命令**set**的描述中记录的所有单字符 shell 选项，包括 **-o**，都可以在调用 shell 时用作选项。 此外，**bash** 在调用时会解释以下选项：

|短选项|描述|
|:--|:--|
|**-c**|如果存在 **-c** 选项，则从第一个非选项参数 <u>command_string</u> 中读取命令。 如果在 <u>command_string </u>之后有参数，则将第一个参数分配给 **\$0**，并将剩余的任意参数分配给位置参数。 对 **\$0** 的赋值设置了 shell 的名称，该名称用于警告和错误消息。|
|**-i**|如果存在 **-i** 选项，则 shell 是交互式的。|
|**-l**|使 bash 的表现得像作为登录 shell 被调用一样（参见下面的 调用）。|
|**-r**|
|**-s**|
|**-v**|
|**-x**|
|**-D**|
|**\[-+]O \[** <u>shopt_option</u> **]**|
|**--**|












------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《bash manpages》。</font>bash的版本为5.0.17，手册更新时间为2018-12-7。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
