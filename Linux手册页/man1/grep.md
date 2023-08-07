---
title: grep
tags: 文本处理,用户命令man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《grep manpages》。</font>grep的版本为3.8，手册更新时间为2019-12-29。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称
grep, egrep, fgrep, rgrep——打印与模式匹配的行

# 概览
**grep** \[<u>OPTION</u>...] <u>PATTERNS </u>\[<u>FILE</u>...]
**grep** \[<u>OPTION</u>...] **-e** <u>PATTERNS </u>... \[<u>FILE</u>...]
**grep** \[<u>OPTION</u>...] **-f** <u>PATTERN_FILE </u>... \[<u>FILE</u>...]

# 描述

grep 在每个<u>FILE</u>中搜索<u>PATTERNS </u>。 <u>PATTERNS </u>是一个或多个由换行符分隔的模式，grep 打印与模式匹配的每一行。 通常，当在 shell 命令中使用 **grep** 时，应将 <u>PATTERN</u>S 加上引号。

<u>FILE</u>为 “-”代表标准输入。 如果未给出 <u>FILE</u>，则递归搜索将检查工作目录，非递归搜索将读取标准输入。

Debian 还包括变种程序**egrep**、**fgrep** 和**rgrep**。 这些程序分别与 **grep -E**、**grep -F** 和 **grep -r** 相同。 这些变体在上游已被弃用，但 Debian 提供了向后兼容性。 出于可移植性的原因，建议避免使用变种程序，而使用带有相关选项的 **grep**。



------


***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《grep manpages》。</font>grep的版本为3.8，手册更新时间为2019-12-29。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------