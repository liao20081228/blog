---
title: grep
tags: 文本处理,用户命令man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《grep manpages》。</font>grep的版本为3.12，手册更新时间为2025-03-21。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称
grep, egrep, fgrep, rgrep——打印与模式匹配的行

# 概览
**grep** \[<u>OPTION</u>...] <u>PATTERNS </u>\[<u>FILE</u>...]
**grep** \[<u>OPTION</u>...] **-e** <u>PATTERNS </u>... \[<u>FILE</u>...]
**grep** \[<u>OPTION</u>...] **-f** <u>PATTERN_FILE </u>... \[<u>FILE</u>...]

# 描述

grep 在每个<u>FILE</u>中搜索<u>PATTERNS </u>。在概览的第一种形式（如果未提供 `-e` 或 `-f` 选项，则使用该形式）， 第一个操作数<u>PATTERNS </u>是一个或多个由换行符分隔的模式，grep 打印与模式匹配的每一行。 通常，当在 shell 命令中使用 **grep** 时，应将 <u>PATTERN</u>S 加上引号。

<u>FILE</u>为 “-”代表标准输入。 如果未给出 <u>FILE</u>，则递归搜索将检查工作目录，非递归搜索将读取标准输入。

Debian 还包括变种程序**egrep**、**fgrep** 和**rgrep**。 这些程序分别与 **grep -E**、**grep -F** 和 **grep -r** 相同。 这些变体在上游已被弃用，但 Debian 提供了向后兼容性。 出于可移植性的原因，建议避免使用变种程序，而使用带有相关选项的 **grep**。

# 选项
**通用程序信息**
- **--help** 
输出使用信息并退出。
- **-V**，**--version** 
输出**grep**的版本号并退出。

**模式语法**
- **-E**，**--extended-regexp**	
将 <u>PATTERNS</u> 解释为扩展正则表达式（EREs，见下文）。
- **-F**，**--fixed-strings**	
将 <u>PATTERNS</u>解释为固定字符串，而不是正则表达式。
- **-G**，**--basic-regexp**	
 将 <u>PATTERNS</u>解释为基本正则表达式（BREs，见下文）。 这是默认设置。
- **-P**，**--perl-regexp**
将 <u>PATTERNS</u>解释为与 Perl 兼容的正则表达式 (PCREs)。 当与 **-z** (**--null-data**) 选项结合使用时，此选项是实验性的，**grep -P** 可能会警告未实现的功能。

**匹配控制**
- **-e** <u>PATTERNS</u>, **--regexp**=<u>PATTERNS</u>
将 <u>PATTERNS </u>用作模式。如果多次使用此选项，或将其与 `-f`（`--file`）选项组合使用，则搜索所有给出的模式。此选项可用于保护以“-”开头的模式。
- **-f** <u>FILE</u>, **--file**=<u>FILE</u>
从 <u>FILE </u>中获取模式，每行一个。如果多次使用此选项，或将其与 `-e`（`--regexp`）选项组合使用，则搜索所有给出的模式。空文件包含零个模式，因此不匹配任何内容。如果 FILE 为 `-`，则从标准输入读取模式。
- **-i**, **--ignore-case**
忽略模式和输入数据中的大小写差异，使仅大小写不同的字符彼此匹配。
- **--no-ignore-case**
不忽略模式和输入数据中的大小写差异。这是默认行为。此选项适用于传递给已经使用 `-i` 的 shell 脚本，以取消其效果，因为这两个选项会相互覆盖。
- **-v**, **--invert-match**
反转匹配的含义，以选择不匹配的行。
- **-w**, **--word-regexp**
仅选择包含构成完整单词的匹配的行。其判定条件是：匹配的子串必须位于行首，或前面是非单词构成字符；同样，它必须位于行尾，或后面是非单词构成字符。单词构成字符是字母、数字和下划线。如果同时指定了 `-x`，则此选项无效。
- **-x**, **--line-regexp**
仅选择与整行完全匹配的匹配项。对于正则表达式模式，这相当于给模式加上括号，然后在其前后分别加上 `^` 和 `$`。

**通用输出控制**

**输出行前缀控制**


**上下文行控制**

**文件和目录选择**


**其它选项**

# 正则表达式

# 退出状态

# 环境

# 注意

# 版权

# bugs

# 示例

# 另请参阅

------


***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《grep manpages》。</font>grep的版本为3.8，手册更新时间为2019-12-29。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------