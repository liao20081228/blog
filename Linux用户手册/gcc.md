---
title: gcc
tags: 用户命令
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《gcc manpages》</font>， gcc当前版本为11.2.0，手册更新时间为2021.7.28。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
# 1 名称
gcc——GNU 项目 C 和 C++ 编译器。
# 2 概要
```bash
gcc [-c|-S|-E] [-std=standard]
	[-g] [-pg] [-Olevel]
	[-Wwarn...] [-Wpedantic]
	[-Idir...] [-Ldir...]
	[-Dmacro[=defn]...] [-Umacro]
	[-foption...] [-mmachine-option...]
	[-o outfile] [@file] infile...
```
这里只列出最有用的一些选项； 其余部分见下文。 **g++** 接受的选项几乎与 **gcc** 相同。
# 3 描述
当您调用 GCC 时，它通常会进行预处理、编译、汇编和链接。 “整体选项”允许您在中间阶段停止此过程。例如，**-c** 选项表示不运行链接器输出由汇编器输出的目标文件组成。

其他选项被传递到一个或多个处理阶段。一些选项控制预处理器，其他选项控制编译器本身。还有其他选项控制汇编器和链接器；其中大部分都没有在这里记录，因为您很少需要使用它们中的任何一个。

大多数可以与 GCC 一起使用的命令行选项对 C 程序有用。当一个选项只对另一种语言（通常是 C++）有用时，解释会明确说明。如果特定选项的描述未提及源语言，您可以将该选项用于所有支持的语言。

运行 GCC 的常用方法是运行名为 **gcc** 的可执行文件，或者在交叉编译时运行 <u>machine</u>**-gcc**，或者在运行特定版本的 GCC 时运行 <u>machine</u>**-gcc-**<u>version</u>。当您编译 C++ 程序时，您应该将 GCC 作为 **g++** 调用。

**gcc** 程序接受选项和文件名作为操作数。许多选项具有多字母名称；因此，多个单字母选项可能无法组合：**-dv** 与 **-d** **-v** 非常地不同。

您可以混合选项和其他参数。在大多数情况下，顺序并不重要。当您使用多个同类选项时，顺序很重要；例如，如果您多次指定 **-L**，将按指定的顺序搜索目录。此外，**-l** 选项的位置也很重要。

许多选项都有以 **-f** 或 **-W** 开头的长名称。例如，**-fmove-loop-invariants**、**-Wformat** 等。其中大部分有肯定和否定两种形式； **-ffoo** 的否定形式是 **-fno-foo**。本手册仅记录了这两种形式中的一种，以非默认形式为准。

某些选项采用一个或多个参数，通常由空格或等号 (=) 与选项名称分隔。除非另有说明，否则参数可以是数字或字符串。数字参数通常必须是小的无符号十进制或十六进制整数。十六进制参数必须以 **0x** 前缀开头。

指定某种大小阈值的选项的参数可以是任意大的十进制或十六进制整数，后跟指定字节倍数的字节大小后缀，例如“kB”和“KiB”分别代表千字节和千位字节，“MB”和“MiB”代表兆字节和兆位字节，“GB”和“GiB”代表千兆字节和千兆位字节，依此类推。此类参数在以下文本中由字节大小指定。有关二进制和十进制字节大小前缀的完整列表和说明，请参阅 NIST、IEC 和其他相关的国家和国际标准。

# 4 选项
## 4.1 选项总结
这是所有选项的摘要，按类型分组。 解释在后续章节。

<u>总体选项</u>
: **-c  -S  -E  -o** <u>file</u> **-dumpbase** <u>dumpbase</u>  **-dumpbase-ext** <u>auxdropsuf</u> **-dumpdir** <u>dumppfx</u>  **-x** <u>language</u> **-v  -###  --help**\[=<u>class</u>\[,...]]  **--target-help  --version -pass-exit-codes -pipe  -specs=**<u>file</u>  **-wrapper @**<u>file</u>  **-ffile-prefix-map**=<u>old</u>=<u>new</u> **-fplugin**=<u>file</u>  **-fplugin-arg-name=arg -fdump-ada-spec**\[**-slim**]  **-fada-spec-parent**=<u>unit</u>  -**fdump-go-spec**=<u>file</u>
# 4 作者
David MacKenzie。
# 5 报告错误
GNU coreutils 在线帮助：<https://www.gnu.org/software/coreutils/>
报告 stty 翻译错误：<https://translationproject.org/team/> 

# 6 版权
版权所有 © 2018 自由软件基金会, Inc. 许可证 GPLv3+：GNU GPL 版本 3 或更高版本 <https://gnu.org/licenses/gpl.html>。

这是免费软件：您可以自由更改和重新分发它。 在法律允许的范围内，不提供任何保证。
# 7 参见
完整文档位于：<https://www.gnu.org/software/coreutils/stty> 或通过本地获取：info '(coreutils) stty invocation'。

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《gcc manpages》</font>， gcc当前版本为11.2.0，手册更新时间为2021.7.28。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
