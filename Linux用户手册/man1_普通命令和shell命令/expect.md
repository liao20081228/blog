---
title: expect
tags: 普通命令
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《EXPECT manpages》。</font>当前版本为5.45.4。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 名称
expect——使用交互式程序进行程序化对话，版本5。

# 2 概要

``` shell
expect [ -dDinN ] [ -c cmds ] [ [ -[f|b] ] cmdfile ] [ args ]
```
# 3 介绍

Expect是一个根据脚本来与其他交互式程序“对话”的程序。随着脚本执行，Expect知道程序可以期望什么，正确的响应应该是什么。解释型语言提供分支和高级控制结构来引导对话。此外，用户可以根据需要进行控制并直接进行交互，然后将控制权返回给脚本。

Expectk是Expect和Tk的混合物。它的行为就像Expect和Tk的wish。 Expect也可以直接在C或C ++中使用（即不使用Tcl）。参见libexpect(3)。

名称“Expect”来自在uucp，kermit和其他调制解调器控制程序中流行的send/expect序列的概念。但是，与uucp不同，Expect是通用的，因此可以在考虑任何程序和任务的情况下将其作为用户级命令运行。Expect实际上可以同时与多个程序对话。

例如，以下是Expect可以做的一些事情：

* 使计算机回拨给您，以便您无需付费即可登录。
* 启动游戏（例如rogue），如果没有出现最佳配置，（一次又一次）重新启动它，直到完成为止，然后将控制权移交给您。
* 运行fsck，并根据预先确定的标准回答问题，回答“yes”，“no”或将控制权还给您。
* 连接到另一个网络或BBS（例如MCI Mail，CompuServe），并自动检索您的邮件，使其看起来就像原本就发送到本地系统一样。
* 在rlogin，telnet，tip，su，chgrp等之间携带环境变量，当前目录或任何类型的信息。

 Shell无法执行这些任务的原因有多种（尝试一下，您会看到的）。使用Expect，一切皆有可能。

通常，Expect主要用于运行那些需要与用户之间进行交互的程序，前提是可以交互可以字符串化和可以编程话。 如果需要，Expect还可以向用户返回控制权（而不会暂停受控制的程序）。 同样，用户可以随时将控制权返回给脚本。

# 4 用法

|Expect&emsp;|Expectk&emsp;&emsp;&emsp;&emsp;|说明|
|:--|:--|:--|
|cmdfile||Expect读取cmdfile以获取要执行的命令列表。在支持＃!的系统上，Expect也可能被隐式调用。通过将脚本标记为可执行文件，并在脚本的第一行中添加`#!/usr/bin/expect -f`。<br /><br />当然，路径必须准确地描述Expect的安装位置。 /usr/bin只是一个示例。|
|-c cmds|-command cmds|cmds先于cmdfile中命令执行。该命令应加引号，以防止被shell破坏。此选项可以多次使用。通过使用分号将多个命令分开，可以使用单个-c执行多个命令。命令按照它们出现的顺序执行。
|-d|-diag|启用一些诊断输出，该诊断输出主要报告命令的内部活动，例如expect和interact。此标志与在Expect脚本开头的“exp_internal 1”具有相同的作用，并且还会打印Expect的版本。（strace命令对跟踪语句很有用，trace命令对跟踪变量赋值很有用。）|
|-D num|-Debug num|标志启用交互式调试器。应该跟一个整数值。如果该值不为零或按下Ctrl+C（或达到断点，或脚本中出现其他合适的调试器命令），则调试器将在下一个Tcl过程之前获得控制权。有关调试器的更多信息，请参见README文件或另请参阅。| 
|-f| -file|为从中读取命令的文件添加了前缀。该标志本身是可选的，因为它仅在使用#!表示法时才有用（请参见上文），因此可以在命令行上提供其他参数。|
|-b|-buffer|默认情况下，命令文件被读入内存并完整执行。偶尔需要一次读取一行，例如读取标准输入。为了强制以这种方式处理任意文件，请使用-b标志。请注意，仍可以进行stdio缓冲，但是从fifo或stdin读取时这不会引起问题。
|-||如果字符串“-”作为文件名提供，则改为读取标准输入。 （使用“./-”从实际名为“-”的文件中读取。）
|-i|-interactive|使Expect以交互方式提示输入命令，而不是从文件中读取命令。通过退出命令或EOF终止提示。有关更多信息，请参见解释器（见后文）。如果既不指定命令文件也不使用-c，则假定为-i。|
|--||可以用来分隔选项的结尾。如果您希望将类似选项的参数传递给脚本而不用Expect解释，则这很有用。可以将其有效地放在＃!中。行以防止Expect进行任何类似标志的解释。例如，`#!/usr/bin/expect`会将原始参数（包括脚本名称）保留在变量argv中。<br /><br />请注意，在向#!行添加参数时，必须遵守通常的getopt(3)和execve(2)约定。|
|-N,-n|-NORC,-norc|除非使用-N标志，否则文件\$exp_library/expect.rc（如果存在）会自动引入。 此后，除非使用-n标志，否则将自动引入文件~/.expect.rc。如果定义了环境变量**DOTDIR**，则将其视为目录，并从该目录中读取.expect.rc。仅在执行任何-c标志之后才进行这种引入。
|-v|-version|使Expect打印其版本号并退出。|
|args||可选的<u>args</u>被构造到一个列表中，并存储在名为<u>argv</u>的变量中。 <u>argc</u>初始化为<u>argv</u>的长度。<br /><br /><u>argv0</u>定义为脚本的名称（如果未使用脚本，则为二进制）。例如`send_user "$argv0 [lrange $argv 0 2]\n"`打印出脚本的名称和前三个参数。|




------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《EXPECT manpages》。</font>当前版本为5.45.4。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
