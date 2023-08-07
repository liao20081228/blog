---
title: cp
tags: 文件管理,用户命令,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《cp manpages》。</font>cp的版本为9.1，手册更新时间为2023-01。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
# 名称
cp——复制文件和文件夹。

# 总览

**cp** [<u>OPTION</u>]... [<u>-T</u>] <u>SOURCE</u> <u>DEST</u>
**cp** [<u>OPTION</u>]... <u>SOURCE</u>... <u>DIRECTORY</u>
**cp** [<u>OPTION</u>]... -t <u>DIRECTORY</u> <u>SOURCE...</u>


# 描述
将SOURCE复制到DEST，或将多个SOURCE复制到DIRECTORY。

长选项的强制性参数对于短选项也是强制性的。

|短选项|长选项|描述|
|:--|:--|:--|
|**-a**|**--archive**|等同于 **-dR --preserve**=<u>all</u>
||**--attributes-only**|不复制文件数据，仅复制属性|
||**--backup**\[=<u>CONTROL</u>]|备份每个现有的目标文件|
|**-b**||类似于 **--backup** 但不接受参数|
||**--copy-contents**|递归时复制特殊文件的内容|
|**-d**|| 与 **--no-dereference --preserve**=<u>links</u>相同|
|**-f**|**--force**| 如果无法打开已存在的目标文件，请将其删除并重试（当同时使用 **-n** 选项时，此选项将被忽略）|
|**-i**|**--interactive**|  覆盖前提示（覆盖之前的 **-n** 选项）|
|**-H**||跟随 <u>SOURCE</u> 中的命令行符号链接|
|**-l**| **--link**|  硬链接文件而不是复制|
|**-L**| **--dereference**|始终跟随 <u>SOURCE</u> 中的符号链接|
|**-n** |**--no-clobber**|不覆盖已存在的文件（覆盖之前的 **-i** 选项）|
|**-P** |**--no-dereference**|永不跟随 SOURCE 中的符号链接|
|**-p**||等同于 **--preserve**=mode,ownership,timestamps|
||**--preserve**\[=<u>ATTR_LIST</u>]|保留指定的属性（默认：mode,ownership,timestamps），如果可能的话，保留指定的附加属性：context, links, xattr,all|
||**--no-preserve**=<u>ATTR_LIST</u>|不保留指定的属性|
|  |**--parents**|使用 DIRECTORY 下的完整源文件名|
|**-R**, **-r**| **--recursive**|递归复制目录|
||**--reflink**\[=<u>WHEN</u>]|控制clone/CoW 副本。 见下文|
||**--remove-destination**|在尝试打开每个已存在的目标文件之前删除它（与 **--force** 对比）|
||**--sparse**=<u>WHEN</u>|控制稀疏文件的创建。 见下文|
||**--strip-trailing-slashes**|从每个 SOURCE 参数中删除所有尾随斜杠|
|**-s**|**--symbolic-link**|建立符号链接而不是复制|
|**-S**| **--suffix**=<u>SUFFIX</u>|覆盖通常的备份后缀|
|**-t** <u>DIRECTORY</u>| **--target-directory**=<u>DIRECTORY</u>|将所有 SOURCE 参数复制到 DIRECTORY 中|
|**-T**| **--no-target-director**y|将 DEST 视为普通文件|
|**-u**|**--update**|仅当源文件比目标文件新或目标文件缺失时才复制|
|**-v**|**--verbose**|解释正在做什么|
|**-x**|**--one-file-system**|保留在此文件系统上|
|**-Z**||将目标文件的 SELinux 安全上下文设置为默认类型|
||**--context**\[=<u>CTX</u>]|类似 **-Z**，或者如果指定了 CTX，则将 SELinux 或 SMACK 安全上下文设置为 CTX|
||**--help**|显示帮助信息并退出|
||**--version**|显示版本信息并退出|
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||

 默认情况下，通过粗略启发式检测到稀疏源文件，相应的目标文件也变得稀疏， 这是 **--sparse**=<u>auto</u> 选择的行为； 只要 SOURCE 文件包含足够长的零字节序列，指定 **--sparse**=<u>always</u> 即可创建稀疏 DEST 文件。 使用 **--sparse**=<u>never</u> 禁止创建稀疏文件。

当指定 **--reflink**\[=<u>always</u>] 时执行轻量级复制，其仅在修改时复制数据块，如果这不可能，则复制失败；或者如果指定了 **--reflink**=<u>auto</u>，则回退到标准复制；使用 **--reflink**=<u>never</u> 确保执行标准复制。

除非使用 **--suffix** 或 SIMPLE_BACKUP_SUFFIX 设置，备份后缀为“~”。 版本控制方法可以通过 **--backup** 选项或通过 VERSION_CONTROL 环境变量来选择。 以下是这些值：
|备份|说明|
|:--|:--|
|none, off|从不进行备份（即使给出了 **--backup**）|
|numbered, t|进行编号备份|
|existing, nil|如果编号备份存在则编号备份，否则简单备份|
|simple, never|总是进行简单的备份|

作为一种特殊情况，当指定force和backup选项并且 SOURCE 和 DEST 与现有普通文件名称相同时，cp 会备份 SOURCE。

# 作者
Torbjorn Granlund、David MacKenzie、Jim Meyering.。

# 报告bugs
GNU coreutils联机帮助：<https://www.gnu.org/software/coreutils/>
将转换bugs报告给<https://translationproject.org/team/>

# 版权
版权所有©2022自由软件基金会公司。 许可证GPLv3+: GNU GPL第3版或更高版本<https://gnu.org/licenses/gpl.html>。

这是自由软件：您可以自由更改和重新分发它。在法律允许的范围内，不提供任何保证。

# 另见

完整文档位于：<https://www.gnu.org/software/coreutils/cp>或通过以下方式在本地获得：`info '(coreutils) cp invocation'`。


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《date manpages》。</font>cp的版本为9.1，手册更新时间为2023-01。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------