---
title: objdump
tags: GNU开发工具,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《objdump manpages》。</font>objdump的版本为2.46，手册更新时间为2026-03。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称

objdump —— 显示目标文件中的信息

# 概要
```
       objdump [-a|--archive-headers]
               [-b bfdname|--target=bfdname]
               [-C|--demangle[=style] ]
               [-d|--disassemble[=symbol]]
               [-D|--disassemble-all]
               [-z|--disassemble-zeroes]
               [-EB|-EL|--endian={big | little }]
               [-f|--file-headers]
               [-F|--file-offsets]
               [--file-start-context]
               [-g|--debugging]
               [-e|--debugging-tags]
               [-h|--section-headers|--headers]
               [-i|--info]
               [-j section|--section=section]
               [-l|--line-numbers]
               [-S|--source]
               [--source-comment[=text]]
               [-m machine|--architecture=machine]
               [-M options|--disassembler-options=options]
               [-p|--private-headers]
               [-P options|--private=options]
               [-r|--reloc]
               [-R|--dynamic-reloc]
               [-s|--full-contents]
               [-Z|--decompress]
               [-W[lLiaprmfFsoORtUuTgAck]|
                --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,=frames-interp,=str,=str-offsets,=loc,=Ranges,=pubtypes,=trace_info,=trace_abbrev,=trace_aranges,=gdb_index,=addr,=cu_index,=links]]
               [-WK|--dwarf=follow-links]
               [-WN|--dwarf=no-follow-links]
               [-wD|--dwarf=use-debuginfod]
               [-wE|--dwarf=do-not-use-debuginfod]
               [-L|--process-links]
               [--ctf=section]
               [--sframe=section]
               [-G|--stabs]
               [-t|--syms]
               [-T|--dynamic-syms]
               [-x|--all-headers]
               [-w|--wide]
               [--start-address=address]
               [--stop-address=address]
               [--no-addresses]
               [--prefix-addresses]
               [--[no-]show-raw-insn]
               [--adjust-vma=offset]
               [--show-all-symbols]
               [--dwarf-depth=n]
               [--dwarf-start=n]
               [--ctf-parent=section]
               [--no-recurse-limit|--recurse-limit]
               [--special-syms]
               [--prefix=prefix]
               [--prefix-strip=level]
               [--insn-width=width]
               [--visualize-jumps[=color|=extended-color|=off]
               [--disassembler-color=[off|terminal|on|extended]
               [-U method] [--unicode=method]
               [-V|--version]
               [-H|--help]
               objfile...


```

## 描述

objdump 显示一个或多个目标文件的信息，具体显示哪些内容由选项控制。这些信息主要对从事编译工具开发的程序员有用，而非仅希望程序正常编译运行的普通开发者。

<u>objfile</u>... 为待分析的目标文件。若指定归档文件（静态库），objdump 会显示其中每个成员目标文件的信息。

# 选项

选项的长格式与短格式等效，必须至少指定以下选项之一： `-a,-d,-D,-e,-f,-g,-G,-h,-H,-p,-P,-r,-R,-s,-S,-t,-T,-V,-x`

- -a, --archive-header
若目标文件<u>objfile</u>为归档文件，显示归档头部信息（格式类似 `ls -l`）。除 `ar tv` 可显示的内容外，还会展示每个归档成员的目标文件格式。

- --adjust-vma = <u>offset</u>
转储信息时，先将偏移量 <u>offset</u> 加到所有节的地址上。这对于节地址与符号表不对应的场景很有用，这种情况常见于使用**a.out**等无法表示节地址的文件格式，并将节放置在特定地址时发生。

- -b <u>bfdname</u>, --target=<u>bfdname</u>
指定目标文件的目标代码格式为 <u>bfdname</u>。<u>objdump</u>可以自动识别许多格式，该选项非必需。
示例：
&emsp;&emsp;`objdump -b oasys -m vax -h fu.o`
显示fu.o节头部的概要信息（`-h`），该文件通过 `-m` 明确指定为VAX架构、由 Oasys 编译器生成格式的目标文件。 你可以使用 `-i` 选项列出所有可用的文件格式。

- -C, --demangle\[=<u>style</u>]
将底层符号名解码（还原）为用户可读的名称。该操作除了会去除系统自动添加的前置下划线外，还能让 C++ 函数名变得易于阅读。 不同编译器采用不同的名称修饰规则。可选的还原风格参数<u>style</u>可用于为你所使用的编译器选择合适的名称还原方式。

- --recurse-limit, --no-recurse-limit, --recursion-limit, --no-recursion-limit
启用或禁用在符号名还原（demangle）过程中对递归层数的限制。 由于名称修饰（name mangling）格式支持无限递归层级，可能会出现某些符号字符串在解码时耗尽主机的可用栈空间，从而触发内存错误。 该限制通过将递归嵌套层数限制为2048 层来避免此类问题。 

   此限制默认是启用的，但在还原极其复杂的符号名时，可能需要将++其++禁用。 但请注意：一旦禁用递归限制，就有可能发生栈溢出，且因此类问题产生的漏洞报告将不予受理。

- -g, --debugging
显示调试信息。尝试解析文件中的 STABS 调试信息并以类 C 语法输出；若无 STABS，则回退到 `-W` 选项输出 DWARF 信息。

- -e, --debugging-tags
功能与 `-g` 类似，但生成的信息采用与 **ctags** 工具兼容的格式

- -d, --disassemble, --disassemble = <u>symbol</u>
显示输入文件中的机器指令对应的汇编助记符。该选项仅对预期包含指令的节进行反汇编。如果指定了可选的<u>symbol</u>参数（可多次指定），则从这些符号所在位置开始显示汇编助记符。若该<u>symbol</u>是函数名，反汇编会在函数末尾停止；否则会在遇到下一个符号时停止。如果没有匹配到任何<u>symbol</u>，则不会显示任何内容。

  注意：如果启用了 `--dwarf=follow-links` 选项，反汇编时会读取并使用链接的调试信息文件中的符号表。

- -D, --disassemble-all
功能同 `-d`，但反汇编所有非空、非 BSS 节，而非仅指令节，可通过 `-j` 指定特定节。

	该选项还会对代码节中指令的反汇编产生一个细微影响。 当使用 **`-d`** 选项时，objdump 会假定代码段中的所有符号都出现在**指令边界**上，并且**不会跨这些边界进行反汇编**。 而当使用 **-D** 选项时，这一假定会被取消。 这意味着，例如在代码段中存放了数据时，**-d 和 -D 的输出结果可能会不一样**。 如果目标架构是 ARM，该选项还会强制反汇编器把代码段中找到的数据片段**当作指令来解码**。

	注意：如果启用了 `--dwarf=follow-links` 选项，反汇编时会读取并使用链接的调试信息文件中的符号表。


### --no-addresses

反汇编时不打印每行地址、符号及重定位偏移，配合 `--no-show-raw-insn` 可用于对比编译器输出。

### --prefix-addresses

反汇编时每行打印完整地址，为旧版反汇编格式。

### -EB, -EL, --endian={big|little}

指定目标文件的字节序，仅影响反汇编。适用于 S-records 等不含字节序信息的文件格式。

### -f, --file-headers

显示每个目标文件整体文件头的摘要信息。

### -F, --file-offsets

反汇编节时，显示符号对应数据区域的文件偏移；跳过零块时提示跳过数量及恢复反汇编的偏移；转储节时显示起始偏移。

### --file-start-context

配合 `-S` 显示源码与反汇编穿插内容时，将未显示过的文件上下文延伸至文件开头。

### -h, --section-headers, --headers

显示目标文件节头部的摘要信息。 部分文件格式（如 a.out）不存储段起始地址，即便链接器正确重定位，`objdump -h` 仍显示目标默认地址。 某些节可能同时设置 READONLY 与 NOREAD 属性，此时 NOREAD 优先级更高，但两者均会被标注。

### -H, --help

打印 objdump 选项摘要并退出。

### -i, --info

显示可通过 `-b` 或 `-m` 指定的所有架构与目标文件格式列表。

### -j 名称，--section = 名称

仅显示指定节的信息，可多次指定。

### -L, --process-links

显示主文件链接的独立调试信息文件中的非调试节内容，自动隐含 `-WK`，仅展示其他选项请求的节。

### -l, --line-numbers

配合 `-d/-D/-r` 使用，通过调试信息为目标代码或重定位项标注文件名与源码行号。

### -m 架构，--architecture = 架构

指定反汇编使用的架构，适用于 S-records 等不含架构信息的文件，可用 `-i` 查看支持架构。 多数架构支持以`架构:机器`格式指定，如 `foo:bar`。 ARM 架构下可限制仅反汇编指定架构支持的指令，无架构信息时需反汇编全部指令可使用 `-marm`。

### -M 选项，--disassembler-options = 选项

向反汇编器传递目标平台特定参数，部分平台支持，多选项可用逗号分隔或多次指定 `-M`。

*   **ARC**：控制 DSP/FPX 指令打印、立即数进制、指定 ISA 等。
*   **ARM/AArch64**：选择寄存器命名集、强制 Thumb 指令解码、禁用指令别名等。
*   **x86**：选择架构、语法（Intel/AT&T）、地址 / 操作数大小等。
*   **PowerPC**：选择原生指令而非别名、指定 CPU 型号、启用扩展指令集等。
*   **RISC-V**：无限制反汇编、数字寄存器名、禁用指令别名、指定特权架构版本等。
*   **MIPS**：禁用伪指令、启用 MSA / 虚拟化指令、选择寄存器命名 ABI / 架构等。
*   **VAX**：指定函数入口地址，适配无符号表的二进制文件。

### -p, --private-headers

打印目标文件格式专属的头部信息，具体内容依格式而定，部分格式无额外信息。

### -P 选项，--private = 选项

打印格式专属信息，选项为逗号分隔列表，支持情况依格式而定（帮助信息中列出）。

*   XCOFF：支持 header、aout、sections 等。
*   PE：支持 header、sections。 ELF 不支持该选项。

### -r, --reloc

打印文件的重定位条目，配合 `-d/-D` 可与反汇编穿插显示。

### -R, --dynamic-reloc

打印动态重定位条目，仅对动态目标（如共享库）有效，可与反汇编穿插显示。 注：不支持 RELR 类型重定位，可使用 readelf 查看。

### -s, --full-contents

显示节的完整内容，常配合 `-j` 使用，默认展示所有非空非 BSS 节。压缩节默认显示压缩形式，添加 `-Z` 可解压显示。

### -S, --source

尽可能穿插显示源码与反汇编，自动隐含 `-d`。

### --show-all-symbols

反汇编时显示匹配某地址的所有符号，而非仅第一个。

### --source-comment [= 文本]

同 `-S`，但为所有源码行添加指定前缀，默认前缀为 `#` 。

### --prefix = 前缀

配合 `-S` 为绝对路径添加前缀。

### --prefix-strip = 层级

配合 `--prefix` 剥离指定层级的初始目录名，无 `--prefix` 则无效。

### --show-raw-insn

反汇编时同时打印十六进制指令码与符号形式，`--prefix-addresses` 除外均为默认行为。

### --no-show-raw-insn

反汇编时不打印指令字节，`--prefix-addresses` 下为默认行为。

### --insn-width = 宽度

反汇编时单行显示指定字节数的指令。

### --visualize-jumps[=color|=extended-color|=off]

以 ASCII 图形可视化函数内跳转，可选普通彩色 / 8 位彩色，`=off` 关闭。

### --disassembler-color=off|terminal|on|extended

控制反汇编语法高亮着色，`terminal` 仅在终端输出时生效，`extended` 使用 8 位色。

### -W [参数], --dwarf [= 参数]

显示文件中的 DWARF 调试节内容，压缩节自动临时解压，可指定仅显示特定类型信息：

*   a/abbrev：.debug_abbrev
*   i/info：.debug_info
*   l/rawline：.debug_line（原始）
*   L/decodedline：.debug_line（解析后）
*   k/links：调试链接节
*   K/follow-links：跟随链接的调试文件
*   其余参数对应各类 DWARF 子节

### --dwarf-depth=n

限制 `.debug_info` 转储的子节点深度为 n，0 表示无限制。

### --dwarf-start=n

仅从编号为 n 的 DIE 开始打印，可配合深度限制使用。

### --ctf [= 节名]

显示指定 CTF 节内容，默认显示 `.ctf`。

### --ctf-parent = 成员

指定 CTF 归档中父字典成员名，适配链接时重命名父成员的场景。

### --sframe [= 节名]

显示指定 SFrame 节内容，默认显示 `.sframe`。

### -G, --stabs

显示 ELF 文件中 .stab/.stab.index 等节内容，适用于使用 STABS 调试格式的系统。

### --start-address = 地址

从指定地址开始显示数据，影响 `-d/-r/-s`。

### --stop-address = 地址

在指定地址停止显示数据，影响 `-d/-r/-s`。

### -t, --syms

打印文件的符号表条目，功能类似 nm 但格式不同，输出包含符号值、标志、节、大小、名称等信息。

### -T, --dynamic-syms

打印动态符号表条目，仅对动态目标有效，格式同 `-t` 并额外显示符号版本信息。

### --special-syms

显示目标平台定义的特殊符号，此类符号通常不对用户开放。

### -U [参数], --unicode = 方式

控制 UTF-8 多字节字符显示，支持默认、当前区域、十六进制、转义序列、红色高亮等模式。

### -V, --version

打印 objdump 版本号并退出。

### -x, --all-headers

显示所有可用头部信息，等价于 `-a -f -h -p -r -t`。

### -w, --wide

适配宽屏输出，不截断符号名。

### -z, --disassemble-zeroes

正常反汇编会跳过零块，该选项强制反汇编零块。

### -Z, --decompress

配合 `-s` 使用，解压压缩节后再显示内容。

### @文件

从文件读取命令行选项，文件中选项以空白分隔，支持引号与转义，可嵌套 `@文件`。

## 参见

nm (1)、readelf (1) 以及 binutils 的 Info 文档。

## 版权

Copyright (c) 1991-2026 Free Software Foundation, Inc.允许在 GNU 自由文档许可证 1.3 或更高版本条款下复制、分发与修改本文档，无不变章节、无前封面文本、无后封面文本。许可证副本见 “GNU Free Documentation License” 章节。




------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《objdump manpages》。</font>objdump的版本为2.46，手册更新时间为2026-03。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------