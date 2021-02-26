---
title: sed
tags: Linux用户命令
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Linux info 手册页》</font>当前版本为sed4.7。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

该文件说明了流编辑器GNU “sed”的4.7版。

版权所有（C）1998-2018自由软件基金会有限公司。

根据自由软件基金会发布的GNU自由文档许可版本1.3或任何更高版本授予复制，分发或修改本文档的权限； 没有不变的部分，没有前封面文字，也没有后封面文字。 许可证的副本包含在标题为《GNU自由文档许可证》的部分中。

# 1 介绍

“sed”是一种流编辑器。 流编辑器用于对输入流（一个文件或来自管道的输入）执行基本的文本转换。 尽管在某种程度上类似于允许脚本编辑的编辑器（例如“ed”），但“sed”仅需要对输入进行一次传递即可工作，因此效率更高。 此外，“sed”具有过滤管道中文本的能力，这使其却别于其他类型的编辑器。

# 2 运行sed
本章介绍怎样运行“sed”。下一章将讨论“sed”脚本和各个“sed”命令的详细信息。

## 2.1 概述
通常，“sed”的调用方式如下：
```shell
	sed SCRIPT INPUTFILE...
```
例如，要将文件“input.txt”中所有出现的“hello”替换为“world”：
```shell
	sed 's/hello/world/'input.txt > output.txt
```
如果您未指定INPUTFILE，或者INPUTFILE为'-'，则为'sed'过滤标准输入的内容。以下命令是等效的：
```shell
	sed 's/hello/world/' input.txt > output.txt
	sed 's/hello/world/' < input.txt > output.txt
	cat input.txt | sed 's/hello/world/' - > output.txt
```
“sed”将输出写到标准输出。使用“-i”可直接编辑文件，而不是打印到标准输出。另请参阅“W”和“s///w”命令以将输出写入其他文件。以下命令会修改“file.txt”，但不会产生任何输出：
```shell
	sed -i 's/hello/world/' file.txt
```
默认情况下，“sed”打印所有已处理的输入（已被诸如“d”之类的命令修改/删除的输入除外）。使用“-n”禁止输出，使用“p”命令打印特定行。以下命令仅打印输入文件的第45行：
```shell
	sed -n '45p' file.txt
```
“sed”将多个输入文件视为一个长流。以下示例打印第一个文件的第一行（“one.txt”）和最后一个文件的最后一行（“three.txt”）。使用“-s”可逆转此行为。
```shell
	sed -n '1p ; $p' one.txt two.txt three.txt
```
如果没有“-e”或“-f”选项，则“sed”使用第一个非选项参数作为SCRIPT，并使用随后的非选项参数作为输入文件。如果使用“-e”或“-f”选项来指定SCRIPT，则所有非选项参数都将作为输入文件。选项“-e”和“-f”可以组合使用，并且可以多次出现（在这种情况下，最终有效的SCRIPT是所有单个SCRIPT的串联）。

下面示例是等效的：

``` shell
	sed 's/hello/world/' input.txt > output.txt
	
	sed -e 's/hello/world/' input.txt > output.txt
	sed --expression='s/hello/world/' input.txt > output.txt

	echo 's/hello/world/' > myscript.sed
	sed -f myscript.sed input.txt > output.txt
	sed --file=myscript.sed input.txt > output.txt
```
## 2.2 命令行选项
调用“sed”的完整格式是：

``` shell
	sed OPTIONS... [SCRIPT] [INPUTFILE...]
```
“sed”也可以与下列命令行选项一起被调用：
|短选项|长选项|说明|
|:--|:--|:--|
||--version|打印正在运行的“sed”的版本和版权申明，然后退出。|
||--help|打印用法消息，简要概述这些命令行选项和Bug报告地址，然后退出。|
|-n|--quiet，--silent|默认情况下，“sed”在脚本的每个循环结束时都会打印出模式空间（请注意“sed”的工作原理：执行循环）。 这些选项将禁用此自动打印，并且仅在通过“p”命令显式告知时，“sed”才产生输出。|
||--debug|以规范形式打印输入的sed程序，并解释程序执行过程。<br />&emsp;&emsp;\$ echo 1 \| sed '\%1%s21232'<br />&emsp;&emsp;3<br /><br />&emsp;&emsp;\$ echo 1 \| sed --debug '\%1%s21232'<br />&emsp;&emsp;SED PROGRAM:<br />&emsp;&emsp;&emsp;/1/ s/1/3/<br />&emsp;&emsp;INPUT:   'STDIN' line 1<br />&emsp;&emsp;PATTERN: 1<br />&emsp;&emsp;COMMAND: /1/ s/1/3/<br />&emsp;&emsp;PATTERN: 3<br />&emsp;&emsp;END-OF-CYCLE:<br />&emsp;&emsp;3
|-e SCRIPT|--expression=SCRIPT|将SCRIPT中的命令添加到处理输入时运行的命令集中。|
|-f SCRIPT-FILE|--file=SCRIPT-FILE|将文件SCRIPT-FILE中包含的命令添加到处理输入时运行的命令集中。| 
|**-i\[SUFFIX]**|--in-place\[=SUFFIX]|此选项指定直接编辑文件。 GNU “sed”通过创建一个临时文件并将输出发送到该文件而不是发送到标准输出来做到这一点。[^1]。<br /><br />此选项暗含“-s”。<br /><br />到达文件末尾时，临时文件将重命名为输入文件的原始名称。如果提供了SUFFIX，则SUFFIX用于在重命名临时文件之前修改旧文件名，从而制作一个备份副本[^2]。<br /><br /> 遵循以下规则：如果SUFFIX不包含“\*”，则将其作为后缀附加到当前文件名的末尾；如果扩展名包含一个或多个“\*” 字符，则将每个星号替换为当前文件名。这使您可以为备份文件的名字添加前缀而不仅是后缀，甚至可以将原始文件的备份副本放置在另一个目录中（前提是该目录已存在）。<br /><br />如果没有提供SUFFIX，则原始文件将被覆盖而不进行备份。<br /><br />因为“-i”带有一个可选参数，所以不应在其后接其他简短选项：<br />&emsp;&emsp;sed -Ei '...' FILE<br />与没有备份后缀的“-E -i”相同——“FILE”将在不创建备份的情况下被直接编辑。<br />&emsp;&emsp;sed -iE '...' FILE<br />等效于“--in-place=E”，创建“FILEE”作为“FILE”的备份。<br /><br />谨慎和“-n”一起使用“-i”：前者禁用行的自动打印，而后者在没有备份的情况下直接改变文件。粗心地使用“-n -i”（且没有显式的“p”命令），输出文件将为空：<br />&emsp;&emsp;＃错误的用法：“文件”将被截断。<br />&emsp;&emsp;sed -ni's / foo / bar /'文件 |
|-l N|--line-length=N|指定“l”命令的默认换行长度。长度为0（零）表示永不换行。如果未指定，则为70。|
||--posix|GNU “sed”包括POSIX sed的几个扩展。为了简化编写可移植脚本的过程，此选项将禁用本手册记录的所有扩展，包括其他命令。大多数扩展都接受POSIX规定的语法以外的“sed”程序，但是其中一些（例如bug报告中描述的“N”命令的行为）确实违反了标准。如果您只想禁用后一种扩展，则可以设置“ POSIXLY_CORRECT”变量为非空值|
|-b|--binary|该选项在每个平台上都可用，但是仅在操作系统区分文本文件和二进制文件的情况下才有效。 进行这种区分时（与MS-DOS，Windows和Cygwin一样），文本文件由回车符和换行符分隔的行组成，并且“sed”看不到结尾的CR。 指定此选项后，“sed”将以二进制模式打开输入文件，因此不要求此特殊处理，并认为行是以换行符结尾的。|
||--follow-symlinks|该选项仅在支持符号链接的平台上可用，并且仅在指定了选项“-i”后才有效。 在这种情况下，如果在命令行上指定的文件是符号链接，则“sed”将跟随该链接并编辑该链接的最终目标。 默认行为是不跟随符号链接，因此链接目标不会被修改。|
|-e，-r|-regexp-extended|使用扩展的正则表达式，而不是基本的正则表达式。 扩展的正则表达式是'egrep'接受的正则表达式。 它们反斜杠通常较少因而更清楚。 从历史上讲，这是GNU扩展，但是'-E'扩展已被添加到POSIX标准中（<http://austingroupbugs.net/view.php?id=528>），因此请使用'-E'来实现可移植性。 多年来，GNU sed都接受“-E”作为未标准化的选项，而BSD sed多年来也接受“-E”，但是使用“-E”的脚本可能无法移植到其他较旧的系统。 注意扩展的正则表达式：ERE语法。|
|-s|--separate|默认情况下，“ sed”会将命令行上指定的文件视为单个连续的长流。此GNU sed扩展允许用户将它们视为单独的文件：范围地址（例如'/abc/,/def/'）不允许跨越多个文件，行号相对于每个文件的开头， “\$”是指每个文件的最后一行，并且“R”命令调用的文件会在每个文件的开始处倒放。|
||--sandbox|在沙盒模式下，'e/w/r' 命令被拒绝——包含它们的程序将被中止而不运行。沙盒模式可确保“sed”仅在命令行上指定的输入文件上运行，不能运行外部程序。|
|-u|--unbuffered| 输入和输出缓冲都都尽可能大小。 （如果输入来自'tail -f'之类的东西，并且您希望尽快看到转换后的输出，则这特别有用。）|
|-z|--null-data，--zero-terminated|将输入视为一系列的行，每行以0（ASCII'NUL'字符）代替换行符。此选项可与“sort -z”和“find -print0”之类的命令一起使用，以处理任意文件名。|
|&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;

如果在命令行上没有给出“-e”，“-f”，“-expression”或“--file”选项，则命令行上的第一个非选项参数将被视为SCRIPT被执行。

如果在执行上述操作后仍保留任何命令行参数，则这些参数将被解释为要处理的输入文件的名称。 文件名“-”是指标准输入流。 如果未指定文件名，则将处理标准输入。


[^1]: 这适用于诸如'='，'a'，'c'，'i'，'l'，'p'之类的命令。 您仍然可以通过使用“ w”或“ W”命令以及“/dev/stdout”特殊文件来写入标准输出

[^2]: 请注意，无论输出是否真的改变，GNU “sed”都会创建备份文件。

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Linux info 手册页》</font>当前版本为sed4.7。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
