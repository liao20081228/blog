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
**bash** \[options] \[command\_string | file]
# 版权
Bash 的版权©1989-2018由自由软件基金会公司所有。

# 描述
Bash 是一个 sh 兼容的命令语言解释器，它执行从标准输入或文件中读取的命令。 Bash 还结合了来自 Korn 和 C shell（ksh 和 csh）的有用特性。

Bash 旨在成为 IEEE POSIX 规范（IEEE 标准 1003.1）的 Shell 和 Utilities 部分的一致实现。 默认情况下，Bash配置为符合 POSIX 标准。

# 选项
内置命令**set**的描述中记录的所有单字符 shell 选项，包括 **-o**，都可以在调用 shell 时用作选项。 此外，**bash** 在调用时会解释以下选项：

|短选项|描述|
|:--|:--|
|**-c**|如果存在 **-c** 选项，则从第一个非选项参数 <u>command_string</u> （命令字符串）中读取命令。 如果在 <u>command_string </u>之后有参数，则将第一个参数分配给 **\$0**，并将剩余参数分配给位置参数。 对 **\$0** 的赋值设置了 shell 的名称，该名称用于警告和错误消息。|
|**-i**|如果存在 **-i** 选项，则 shell 是交互式的。|
|**-l**|使 **bash** 的表现得像作为登录 shell 被调用一样（参见下面的 [调用](#INVOCATION)）。|
|**-r**|如果存在 **-r** 选项，shell 将成为<u>restricted</u>（受限的）（请参阅下面的 [受限的SHELL](#RESTRICTEDSHELL)）。|
|**-s**|如果存在 **-s** 选项，或者在选项处理后没有留下任何参数，则从标准输入读取命令。此选项允许在调用交互式 shell 或通过管道读取输入时设置位置参数。|
|**-v**|在读取 shell 输入行时打印它们。|
|**-x**|在执行时打印命令及其参数。|
|**-D**| 以 **\$** 开头的所有双引号字符串组成的列表将被打印在标准输出上。这些是当前本地环境不是 **C** 或 **POSIX** 时需要进行语言翻译的字符串。这隐含着 **-n** 选项；不会执行任何命令。|
|**\[-+]O \[** <u>shopt\_option</u> **\]** | <u>shopt\_option</u> 是被内置命令 **shopt** 接受的 shell 选项之一（参见下面的 [SHELL内建命令](#SHELLBUILTINCOMMANDS)）。如果存在 <u>shopt\_option</u>，则 **-O** 设置该选项的值； **+O** 取消设置。如果未提供 <u>shopt_option</u>，则 **shopt** 接受的 shell 选项的名称和值将打印在标准输出中。如果调用选项为 **+O**，则输出以可重复用作输入的格式显示。|
|**--**|一个 -- 表示选项结束并禁用后续的选项处理。-- 之后的任何参数都被视为文件名和参数。 - 的一个参数等价于 --。|
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||








------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《bash manpages》。</font>bash的版本为5.0.17，手册更新时间为2018-12-7。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
