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

objdump 显示一个或多个目标文件的信息，具体显示哪些内容由选项控制。这些信息主要对从事编译工具开发的程序员有用，而非仅希望程序正常编译运行的普通开发者。

<u>objfile</u>... 为待分析的目标文件。若指定归档文件（静态库），objdump 会显示其中每个成员目标文件的信息。

# 选项

# 参见

nm (1)、readelf (1) 以及 binutils 的 Info 文档。

# 版权
Copyright (c) 1991-2026 Free Software Foundation, Inc.允许在 GNU 自由文档许可证 1.3 或更高版本条款下复制、分发与修改本文档，无不变章节、无前封面文本、无后封面文本。许可证副本见 “GNU Free Documentation License” 章节。




------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《objdump manpages》。</font>objdump的版本为2.46，手册更新时间为2026-03。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------