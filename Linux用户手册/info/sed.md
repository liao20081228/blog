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
	sed's/hello/world/'input.txt > output.txt
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
||--debug|以规范形式打印输入的sed程序，并解释程序执行过程。&emsp;&emsp;\$ echo 1 \| sed '\%1%s21232'
          3

          $ echo 1 | sed --debug '\%1%s21232'
          SED PROGRAM:
            /1/ s/1/3/
          INPUT:   'STDIN' line 1
          PATTERN: 1
          COMMAND: /1/ s/1/3/
          PATTERN: 3
          END-OF-CYCLE:
          3
|
|&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Linux info 手册页》</font>当前版本为sed4.7。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
