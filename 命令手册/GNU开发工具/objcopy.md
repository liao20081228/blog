---
title: objcopy
tags: GNU开发工具,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《objcopy manpages》。</font>objdump的版本为2.46，手册更新时间为2026-03。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称

objcopy —— 复制并转换目标文件

# 概要
```
      objcopy  [-F bfdname|--target=bfdname]
               [-I bfdname|--input-target=bfdname]
               [-O bfdname|--output-target=bfdname]
               [-B bfdarch|--binary-architecture=bfdarch]
               [-S|--strip-all]
               [-g|--strip-debug]
               [--strip-unneeded]
               [-K symbolname|--keep-symbol=symbolname]
               [--keep-file-symbols]
               [--keep-section-symbols]
               [-N symbolname|--strip-symbol=symbolname]
               [--strip-unneeded-symbol=symbolname]
               [-G symbolname|--keep-global-symbol=symbolname]
               [--localize-hidden]
               [-L symbolname|--localize-symbol=symbolname]
               [--globalize-symbol=symbolname]
               [--globalize-symbols=filename]
               [-W symbolname|--weaken-symbol=symbolname]
               [-w|--wildcard]
               [-x|--discard-all]
               [-X|--discard-locals]
               [-b byte|--byte=byte]
               [-i [breadth]|--interleave[=breadth]]
               [--interleave-width=width]
               [-j sectionpattern|--only-section=sectionpattern]
               [-R sectionpattern|--remove-section=sectionpattern]
               [--keep-section=sectionpattern]
               [--remove-relocations=sectionpattern]
               [--strip-section-headers]
               [-p|--preserve-dates]
               [-D|--enable-deterministic-archives]
               [-U|--disable-deterministic-archives]
               [--debugging]
               [--gap-fill=val]
               [--pad-to=address]
               [--set-start=val]
               [--adjust-start=incr]
               [--change-addresses=incr]
               [--change-section-address sectionpattern{=,+,-}val]
               [--change-section-lma sectionpattern{=,+,-}val]
               [--change-section-vma sectionpattern{=,+,-}val]
               [--change-warnings] [--no-change-warnings]
               [--set-section-flags sectionpattern=flags]
               [--set-section-alignment sectionpattern=align]
               [--add-section sectionname=filename]
               [--dump-section sectionname=filename]
               [--update-section sectionname=filename]
               [--rename-section oldname=newname[,flags]]
               [--long-section-names {enable,disable,keep}]
               [--change-leading-char] [--remove-leading-char]
               [--reverse-bytes=num]
               [--srec-len=ival] [--srec-forceS3]
               [--redefine-sym old=new]
               [--redefine-syms=filename]
               [--weaken]
               [--keep-symbols=filename]
               [--strip-symbols=filename]
               [--strip-unneeded-symbols=filename]
               [--keep-global-symbols=filename]
               [--localize-symbols=filename]
               [--weaken-symbols=filename]
               [--binary-symbol-prefixstring]
               [--add-symbol name=[section:]value[,flags]]
               [--alt-machine-code=index]
               [--prefix-symbols=string]
               [--prefix-sections=string]
               [--prefix-alloc-sections=string]
               [--add-gnu-debuglink=path-to-file]
               [--only-keep-debug]
               [--strip-dwo]
               [--extract-dwo]
               [--extract-symbol]
               [--writable-text]
               [--readonly-text]
               [--pure]
               [--impure]
               [--file-alignment=num]
               [--heap=reserve[,commit]]
               [--image-base=address]
               [--section-alignment=num]
               [--stack=reserve[,commit]]
               [--subsystem=which:major.minor]
               [--compress-debug-sections]
               [--decompress-debug-sections]
               [--elf-stt-common=val]
               [--merge-notes]
               [--no-merge-notes]
               [--verilog-data-width=val]
               [-v|--verbose]
               [-V|--version]
               [--help] [--info]
               infile [outfile]



```

## 描述

GNU objcopy 工具用于将一个目标文件的内容复制到另一个文件中。objcopy 使用 GNU BFD 库读写目标文件。它可以将目标文件以不同于源文件的格式写入输出文件。objcopy 的具体行为由命令行选项控制。注意，objcopy 应该能够在任意两种格式之间复制已完成链接的文件。但是，在任意两种格式之间复制可重定位目标文件时，可能无法按预期工作。 objcopy 会创建临时文件来完成格式转换，之后删除这些临时文件。objcopy 使用 BFD 完成所有转换工作；它可以处理 BFD 中描述的所有格式，因此通常无需显式指定即可识别大多数格式。 通过将输出目标设置为 srec，例如使用 -O srec，objcopy 可以用来生成 S 记录格式文件。 通过将输出目标设置为 binary，例如使用 -O binary，objcopy 可以用来生成原始二进制文件。当 objcopy 生成原始二进制文件时，它实质上会生成输入目标文件内容的内存转储数据。所有符号和重定位信息都会被丢弃。内存转储将从被复制到输出文件的地址最低的段的加载地址开始。 在生成 S 记录或原始二进制文件时，使用 -S 移除包含调试信息的段会很有帮助。在某些情况下，使用 -R 移除二进制文件不需要的信息所在的段也会很有用。 注意，objcopy 无法改变输入文件的字节序。如果输入格式具有字节序，部分格式并不具备字节序，那么 objcopy 只能将输入内容复制到具有相同字节序或不具备字节序的文件格式中，例如 srec。不过可以参考 --reverse-bytes 选项。

# 选项

# 参见

nm (1)、readelf (1) 以及 binutils 的 Info 文档。

# 版权
Copyright (c) 1991-2026 Free Software Foundation, Inc.允许在 GNU 自由文档许可证 1.3 或更高版本条款下复制、分发与修改本文档，无不变章节、无前封面文本、无后封面文本。许可证副本见 “GNU Free Documentation License” 章节。




------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《objdump manpages》。</font>objdump的版本为2.46，手册更新时间为2026-03。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------