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

GNU objcopy 工具用于将一个目标文件的内容复制到另一个文件中。objcopy 使用 GNU BFD 库读写目标文件。它可以将目标文件以不同于源文件的格式写入输出文件。objcopy 的具体行为由命令行选项控制。注意，objcopy 应该能够在任意两种格式之间复制已完成链接的文件。但是，在任意两种格式之间复制可重定位目标文件时，可能无法按预期工作。 

objcopy 会创建临时文件来完成格式转换，之后删除这些临时文件。objcopy 使用 BFD 完成所有转换工作；它可以处理 BFD 中描述的所有格式，因此通常无需显式指定即可识别大多数格式。 

通过将输出目标设置为 srec，例如使用 `-O srec`，objcopy 可以用来生成 S-records。

通过将输出目标设置为 binary，例如使用 `-O binary`，objcopy 可以用来生成原始二进制文件。当 objcopy 生成原始二进制文件时，它实质上会生成输入目标文件内容的内存转储数据。所有符号和重定位信息都会被丢弃。内存转储将从被复制到输出文件的最低节的加载地址开始。

在生成 S-record 或原始二进制文件时，使用 `-S` 移除包含调试信息的节会很有帮助。在某些情况下，使用 `-R` 移除含有二进制文件不需要信息的节也会很有用。

注意，objcopy 无法改变输入文件的字节序。如果输入格式具有字节序（部分格式并不具备字节序），那么 objcopy 只能将输入内容复制到具有相同字节序或不具备字节序的文件格式中，例如 srec。不过可以参考 `--reverse-bytes` 选项。。

# 选项
- <u>infile</u> <u>outfile</u> 
分别为输入文件和输出文件。如果未指定输出文件<u>outfile</u> ，如果不指定输出文件，objcopy 会创建一个临时文件，处理完成后用该临时文件覆盖并重命名为输入文件<u>infile</u>的名称。

- -I <u>bfdname</u>, --input-target=<u>bfdname</u> 
认为源文件的目标格式为 <u>bfdname</u>，而不自动检测推断。 

- -O <u>bfdname</u>，--output-target=<u>bfdname </u>
 以 <u>bfdname </u>指定的目标格式写入输出文件。 
 
- -F <u>bfdname</u>， --target=<u>bfdname</u>
 将 bfdname 同时作为输入和输出文件的目标格式，即仅直接复制数据，不进行格式转换。
 
- -B <u>bfdarch </u>--binary-architecture=<u>bfdarch</u>
在将无架构信息的输入文件转换为目标文件时非常有用。 这种情况下，可以将输出文件的架构设置为 <u>bfdarch</u>。 如果输入文件本身已包含明确的架构信息<u>bfdarch</u>，此选项将被忽略。 你可以在程序中通过转换过程生成的特殊符号来访问这些二进制数据。 这些符号分别是： `_binary_objfile_start`、`_binary_objfile_end` 和 `_binary_objfile_size`。 例如，你可以将一张图片文件转换成目标文件，然后在代码中通过这些符号直接访问图片数据。

- -j <u>sectionpattern </u>，--only-section=<u>sectionpattern</u>
只将输入文件中指定的节复制到输出文件。该选项可以多次使用。注意，不当使用此选项可能会导致输出文件无法使用。<u>sectionpattern </u>中支持通配符。

	如果 <u>sectionpattern </u>的第一个字符是感叹号（!），那么匹配的节将不会被复制，即使在同一命令行中此前使用的 `--only-section` 原本会复制这些节。 例如：
&emsp;&emsp;`--only-section=.text.* --only-section=!.text.foo`
将会复制所有匹配 `.text.*` 的节，但不会复制 `.text.foo` 这个节。
  
- -R <u>sectionpattern </u>，--remove-section=<u>sectionpattern</u>
从输出文件中移除所有匹配 <u>sectionpattern </u>的节。该选项可多次使用。注意，不当使用此选项可能会导致输出文件无法使用。 <u>sectionpattern </u>中支持通配符。同时使用 `-j` 和 `-R` 选项会导致未定义行为。 

	如果 <u>sectionpattern </u>的第一个字符是感叹号（!），那么匹配的节不会被移，即使在同一命令行中之前使用的`--remove-section` 原本会移除这些节。 例如： 
&emsp;&emsp;`--remove-section=.text.* --remove-section=!.text.foo`
将会移除所有匹配 `.text.*` 模式的节，但不会移除 `.text.foo` 节。
	
- --keep-section=<u>sectionpattern</u> 
在从输出文件中移除节时，保留与<u>sectionpattern</u>相匹配的节。

- --remove-relocations=<u>sectionpattern</u> 
移除输出文件中所有匹配<u>sectionpattern</u>的节的非动态重定位信息。该选项可多次使用。注意，不当使用该选项可能会导致输出文件无法使用；并且尝试通过 `--remove-relocations=.plt` 从可执行文件或共享库中移除 `.rela.plt` 这类动态重定位节是无效的。 <u>sectionpattern</u>支持通配符。例如：
&emsp;&emsp;`--remove-relocations=.text.* `
将会移除所有匹配 `.text.*` 模式的节的重定位信息。 

	如果<u>sectionpattern</u>以感叹号 `!` 开头，那么匹配的节也不会被移除重定位信息，即使在同一命令行中之前使用的`--remove-relocations` 原本本会移除这些节的重定位信息，例如： 
&emsp;&emsp;` --remove-relocations=.text.* --remove-relocations=!.text.foo `
将会移除所有匹配 `.text.*` 模式的节的重定位信息，但不会移除 `.text.foo` 节的重定位信息。
 
- --strip-section-headers 
剥离节头部。该选项专用于 ELF 文件，且隐含启用 `--strip-all` 和 `--merge-notes`。

- -S, --strip-all 
不从源文件中复制重定位和符号信息，同时删除调试节。 

- -g, --strip-debug
不从源文件中复制调试符号或调试节。

- --strip-unneeded 
除了移除由 `--strip-debug` 剥离的调试符号和调试节之外，还会移除所有重定位处理不需要的符号。 
  
- -K <u>symbolname</u>, --keep-symbol=<u>symbolname </u>
在剥离符号时，保留指定名称的符号，即使该符号在正常情况下会被剥离。该选项可以多次使用。 

- -N symbolname, --strip-symbol=symbolname 不从源文件中复制指定名称的符号。该选项可多次使用。 
- --strip-unneeded-symbol=symbolname 不从源文件中复制指定名称的符号，除非该符号被重定位操作所依赖。该选项可多次使用。 
- -G symbolname, --keep-global-symbol=symbolname 仅将指定名称的符号保留为全局符号，将其余所有符号设为文件局部符号，使其对外不可见。该选项可多次使用。注意：该选项不可与 `--globalize-symbol` 或 `--globalize-symbols` 选项同时使用。 
- --localize-hidden 在 ELF 目标文件中，将所有具备隐藏可见性或内部可见性的符号标记为局部符号。该选项会叠加在 `-L` 这类针对特定符号的本地化选项之上生效。

# 参见

nm (1)、readelf (1) 以及 binutils 的 Info 文档。

# 版权
Copyright (c) 1991-2026 Free Software Foundation, Inc.允许在 GNU 自由文档许可证 1.3 或更高版本条款下复制、分发与修改本文档，无不变章节、无前封面文本、无后封面文本。许可证副本见 “GNU Free Documentation License” 章节。




------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《objdump manpages》。</font>objdump的版本为2.46，手册更新时间为2026-03。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------