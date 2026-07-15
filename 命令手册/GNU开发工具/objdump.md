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

- --adjust-vma=<u>offset</u>
转储信息时，先将偏移量 <u>offset</u> 加到所有节的地址上。这对于节地址与符号表不对应的场景很有用，这种情况常见于使用`a.out`等无法表示节地址的文件格式，并将节放置在特定地址时发生。

- -b <u>bfdname</u>, --target=<u>bfdname</u>
指定目标文件的目标代码格式为 <u>bfdname</u>。<u>objdump</u>可以自动识别许多格式，该选项非必需。
示例：
&emsp;&emsp;`objdump -b oasys -m vax -h fu.o`
显示fu.o节头部的概要信息（`-h`），该文件通过 `-m` 明确指定为VAX架构、由 Oasys 编译器生成格式的目标文件。 你可以使用 `-i` 选项列出所有可用的文件格式。

- -C, --demangle\[=<u>style</u>]
将底层符号名解码（还原）为用户可读的名称。该操作除了会去除系统自动添加的前置下划线外，还能让 C++ 函数名变得易于阅读。 不同编译器采用不同的名称修饰规则。可选的还原风格参数<u>style</u>可用于为你所使用的编译器选择合适的名称还原方式。

- --recurse-limit, --no-recurse-limit, --recursion-limit, --no-recursion-limit
启用或禁用在符号名还原（demangle）过程中对递归层数的限制。 由于名称修饰（name mangling）格式支持无限递归层级，可能会出现某些符号字符串在解码时耗尽主机的可用栈空间，从而触发内存错误。 该限制通过将递归嵌套层数限制为2048 层来避免此类问题。 

	此限制默认是启用的，但在还原极其复杂的符号名时，可能需要将其禁用。 但请注意：一旦禁用递归限制，就有可能发生栈溢出，且因此类问题产生的漏洞报告将不予受理。

- -g, --debugging
显示调试信息。尝试解析文件中的 STABS 调试信息并以类 C 语法输出；若无 STABS，则回退到 `-W` 选项输出 DWARF 信息。

- -e, --debugging-tags
功能与 `-g` 类似，但生成的信息采用与 ctags工具兼容的格式

- -d, --disassemble, --disassemble = <u>symbol</u>
显示输入文件中的机器指令对应的汇编助记符。该选项仅对预期包含指令的节进行反汇编。如果指定了可选的<u>symbol</u>参数（可多次指定），则从这些符号所在位置开始显示汇编助记符。若该<u>symbol</u>是函数名，反汇编会在函数末尾停止；否则会在遇到下一个符号时停止。如果没有匹配到任何<u>symbol</u>，则不会显示任何内容。

  注意：如果启用了 `--dwarf=follow-links` 选项，反汇编时会读取并使用链接的调试信息文件中的符号表。

- -D, --disassemble-all
功能同 `-d`，但反汇编所有非空、非 BSS 节，而非仅指令节，可通过 `-j` 指定特定节。

	该选项还会对代码节中指令的反汇编产生一个细微影响。 当使用 `-d` 选项时，objdump 会假定代码节中的所有符号都出现在指令边界上，并且不会跨这些边界进行反汇编。 而当使用 `-D` 选项时，这一假定会被取消。 这意味着`-d` 和 `-D `的输出结果可能会不一样，例如在代码段中存放了数据时。
	如果目标架构是 ARM，该选项还会强制反汇编器把代码节中找到的数据片段当作指令来解码。

	注意：如果启用了 `--dwarf=follow-links` 选项，反汇编时会读取并使用链接的调试信息文件中的符号表。

- --no-addresses
反汇编时不打印每行地址、符号及重定位偏移，配合 `--no-show-raw-insn` 可用于对比编译器输出。

- --prefix-addresses
反汇编时每行打印完整地址，为旧版反汇编格式。

- -EB, -EL, --endian={big|little}
指定目标文件的字节序，仅影响反汇编。适用于 S-records 等不含字节序信息的文件格式。

- -f, --file-headers
显示每个目标文件<u>objfile</u>的整体文件头的摘要信息。

- -F, --file-offsets
在反汇编节时，每当显示一个符号时，也应显示即将转储的数据区域的文件偏移量。如果跳过了零值，则在反汇编恢复时，应告知用户跳过了多少个零值以及反汇编恢复位置的文件偏移量。在转储节时，应显示转储开始位置的文件偏移量。

- --file-start-context
指定当对尚未显示过的文件交错显示源码与反汇编内容（隐含启用 `-S`）时，将上下文范围扩展至文件开头。

- -h, --section-headers, --headers
显示目标文件的节头部的摘要信息。 

	文件段可能会被重定位到非标准地址，例如通过向链接器 `ld` 指定 `-Ttext`、`-Tdata` 或 `-Tbss` 等选项实现。 然而，某些目标文件格式（如 a.out）并不存储文件段的起始地址。 在这类情况下，尽管 `ld` 已正确完成节的重定位，但使用 `objdump -h` 列出文件节头部时，却无法显示正确的地址，而是会显示目标平台默认的常规地址。

	注意：在某些场景下，一个节可能同时被设置 READONLY（只读）和 NOREAD（不可读） 属性。 出现这种冲突时，NOREAD 属性优先级更高，但 objdump 仍会同时列出这两个属性，因为标志位的具体设置情况可能具有重要意义。

- -H, --help
打印 objdump 选项摘要并退出。

- -i, --info
显示可通过 `-b` 或 `-m` 指定的所有架构与目标文件格式列表。

- -j <u>name</u>，--section=<u>name</u>
仅显示指定节<u>name</u>的信息，可多次指定。

- -L, --process-links
显示与主文件相链接的、独立调试信息文件中所含的非调试节内容。 该选项会自动隐含启用 `-WK` 选项，并且仅显示由其他命令行选项指定需要展示的节。

- -l, --line-numbers
借助调试信息，为显示内容标注出目标代码或重定位项所对应的文件名与源代码行号。 该选项仅在配合 `-d`、`-D` 或 `-r` 使用时有效。

- -m <u>machine</u>，--architecture=<u>machine</u>
指定反汇编目标文件所使用的架构，适用于 S-records 等没有描述架构信息的文件，可用 `-i` 查看支持的所有架构。 

	多数架构支持以`架构名:机器名`的格式指定，如 `foo:bar`，指定foo架构下的bar机器类型。若 objdump 配置为支持多种架构，此用法会很有帮助。

	若目标为 ARM 架构，该选项还会产生额外效果： 它会将反汇编范围限制为指定机器<u>machine</u>的架构所支持的指令集。 如果因输入文件不含架构信息而必须使用该选项，但又希望反汇编所有指令，可使用 `-marm`。

- -M <u>options</u>，--disassembler-options=<u>options</u>
向反汇编器传递与目标平台相关的特定信息。该功能仅在部分目标平台上受支持。若需要指定多个反汇编选项，可多次使用 `-M` 参数，或将多个选项以逗号分隔的形式合并书写。

	针对 ARC 架构： `dsp`：控制显示DSP 指令。 `spfp`：选择显示FPX 单精度FP指令。`dpfp`：选择显示FPX 双精度FP指令。 `quarkse_em`：选择显示专用的 QuarkSE-EM 指令。`fpuda`：选择显示双精度辅助指令。 `fpus`：选择显示 FPU 单精度FP指令。`fpud`：选择显示 FPU 双精度FP指令。 此外，可通过 `hex` 让所有立即数以十六进制形式打印。 默认情况下，短立即数以十进制显示，长立即数以十六进制显示。 

	`cpu=...` 可在反汇编指令时强制指定某一指令集架构（ISA），覆盖 `-m` 参数或 ELF 文件中记录的架构信息。该参数常用于指定 ARC EM 或 HS 指令集架构，因为二者架构相同，反汇编器需依靠 ELF 私有头部数据区分代码目标平台。该参数可多次指定，仅最后一次生效。合法取值与汇编器 `-mcpu=...` 选项一致。 
	
	针对 ARM 架构 该选项可用于选择反汇编时使用的寄存器命名集：`-M reg-names-std`（默认）：采用 ARM 指令集文档中的寄存器命名，其中寄存器 13 为 `sp`、14 为 `lr`、15 为 `pc`。`-M reg-names-apcs`：采用 ARM 过程调用标准（APCS）的寄存器命名规则 。`-M reg-names-raw`：直接使用 `r` 加数字的原始寄存器名。

	另有两种 APCS 命名变体： `-M reg-names-atpcs` 和 `-M reg-names-special-atpcs`， 二者均遵循 ARM/Thumb 过程调用标准命名约定（分别使用普通寄存器名与专用寄存器名）。 
	
	同时，ARM 平台可通过 `--disassembler-options=force-thumb` 强制反汇编器将所有指令按Thumb 指令解析，适用于反汇编其他编译器生成的纯 Thumb 代码。 
	
	针对 AArch64 架构：  `-M no-aliases`：以最通用的指令形式反汇编，不使用指令别名。 `-M notes`：在反汇编结果中以注释形式生成指令说明。

	针对 x86 架构：部分选项与 `-m` 功能重复，但支持更精细的控制： 
	`x86-64`/`i386`/`i8086`：为指定架构执行反汇编。
	`intel`/`att`：在 Intel 语法与 AT&T 语法间切换。
	`amd64`/`intel64`：在 AMD64 与 Intel64 指令集架构间选择。
	`intel-mnemonic` / `att-mnemonic`：在intel指令助记符模式和AT&T指令助记符模式选择（`intel-mnemonic` 隐含 `intel`，`att-mnemonic` 隐含 `att`） 
	`addr64`/`addr32`/`addr16`/`data32`/`data16`：指定默认地址长度与操作数长度；若选项串后出现 `x86-64`/`i386`/`i8086`，则以上设置会被覆盖
	`suffix`：在 AT&T 模式下，以及 Intel 模式下的部分指令中，强制打印助记符后缀，即便该后缀可通过操作数或执行模式默认值推导得出。
	
	针对 PowerPC 架构， `-M raw` 表示直接反汇编硬件原生指令，而非别名指令。例如显示 `rlwinm` 而非 `clrlwi`，显示 `addi` 而非 `li`。 支持gas汇编器中所有用于选择 CPU 的 `-m` 参数，包括： 403、405、440、464、476、601、603、604、620、7400、7410、7450、7455、750cl、821、850、860、a2、booke、booke32、cell、com、e200z2、e200z4、e300、e500、e500mc、e500mc64、e500x2、e5500、e6500、efs、power4、power5、power6、power7、power8、power9、power10、power11、ppc、ppc32、ppc64、ppc64bridge、ppcp、pwr、pwr2、pwr4、pwr5、pwr5x、pwr6、pwr7、pwr8、pwr9、pwr10、pwr11、pwrx、titan、vle、future。 `32` 和 `64` 可修改默认或此前指定的 CPU 架构，分别禁用/启用 64 位指令。 此外，`altivec`、`any`、`lsp`、`htm`、`vsx`、`spe`、`spe2` 可为前后指定的 CPU 追加扩展能力。 `any` 会反汇编 binutils 支持的所有操作码，但在操作码存在多义性或不同参数时，反汇编结果可能不符合预期。 若未指定 CPU，反汇编器会通过 BFD 从目标文件头读取信息选用默认架构，结果同样可能与预期不符。

	针对 RISC-V 架构 支持以下选项： 
	`max`：直接反汇编而不检查架构字符串。这是一种尽力而为的模式，对存在重叠的指令集扩展，将采用首个匹配项（在特定上下文下可能并不正确）来对指令进行解码。如果 ELF 文件未暴露 ISA 字符串，导致无法自动检测IAS子集，且默认回退的 ISA 字符串（`rv64gc`）无法覆盖二进制文件中的所有指令时，该模式会非常有用。 
	`numeric`：以数字形式打印寄存器名，而非 ABI 名称（例如打印 `x2` 而非 `sp`） 
	`no-aliases`：仅以标准原生指令形式反汇编。例如压缩指令会原样显示（`addi sp,sp,-128` 会显示为 `c.addi16sp sp,-128`） 
	`priv-spec=SPEC`：按指定的特权架构规范版本打印 CSR 寄存器，如 1.10、1.11、1.12、1.13 

	针对 MIPS 架构， 该选项控制反汇编结果中指令助记符与寄存器名的显示方式。可通过逗号分隔指定多个选项，无效选项会被忽略：

	`no-aliases`：打印原生指令助记符而非伪指令助记符。例如打印 `daddu`/`or` 而非 `move`，打印 `sll` 而非 `nop`。
	`msa`：反汇编 MSA 指令。
	`virt`：反汇编虚拟化 ASE 指令。
	`xpa`：反汇编扩展物理地址（XPA）ASE 指令。
	`gpr-names=ABI`： 显示适用于指定ABI的通用寄存器（GPR）名称，默认按待反汇编二进制文件的 ABI 选择。
	`fpr-names=ABI`：显示适用于指定 ABI 的浮点寄存器（FPR）名称，默认直接打印FPR编号而不是名字。
	`cp0-names=ARCH`：按<u>ARCH</u>指定的 CPU/架构显示系统协处理器 CP0 寄存器名，默认按待反汇编文件的架构与 CPU 选择CP0寄存名。
	`hwr-names=ARCH`：按<u>ARCH</u>指定 CPU/架构显示硬件寄存器（HWR，`rdhwr` 指令使用）名称，默认按待反汇编文件的架构与 CPU 选择HWR名。
	`reg-names=ABI`：按所选 ABI 显示 GPR 与 FPR 名称。
	`reg-names=ARCH`：按所选 CPU/架构显示 CPU 专用寄存器名（CP0 与 HWR）

	以上所有选项中，<u>ABI</u> 或 <u>ARCH</u> 均可设为数字化的，使对应寄存器以数字而非名称显示。可通过 `--help` 查看支持的 <u>ABI</u>与<u>ARCH</u>取值.
	
	针对 VAX 架构，可通过 `-M entry:0xf00ba` 指定函数入口地址。你可以多次使用此选项来正确反汇编不包含符号表的VAX二进制文件（例如ROM转储文件）。在这些情况下，如果不做此设置，函数入口掩码会被解码为VAX指令，这可能导致函数的其余部分被错误地反汇编。

- -p, --private-headers
打印与目标文件格式相关的特定信息。 具体输出的信息取决于目标文件格式本身。 对于部分目标文件格式，不会输出任何额外信息。

- -P <u>options</u>，--private=<u>options</u>
打印目标文件格式专属的信息。参数 <u>options</u>是一个以逗号分隔的选项列表，具体内容取决于文件格式（可用选项列表可通过帮助信息查看）。

	对于 XCOFF 格式，可用选项为："header"、"aout"、"sections"、"syms"、"relocs"、"lineno"、"loader"、"except"、"typchk"、 "traceback"、"toc"、"ldinfo"。
	
	对于 PE 格式，可用选项为："header"、"sections"。

	并非所有目标文件格式都支持该选项，ELF 格式尤其不使用该选项。

- -r, --reloc
打印文件的重定位条目，配合 `-d/-D` 可与反汇编穿插显示。

- -R, --dynamic-reloc
打印文件中的动态重定位条目。 该选项仅对动态目标文件有意义，例如某些类型的共享库。 与 `-r` 类似，如果同时使用 `-d` 或 `-D`，重定位信息会穿插在反汇编内容中一起显示。

	注意：objdump 不支持显示 RELR 类型的重定位项。这类重定位信息可以通过 `readelf` 程序查看。

- -s, --full-contents
显示节的完整内容，此选项常与 `-j` 配合使用，以指定需要查看的特定节。 默认情况下，会显示所有非空且非 bss 类型的节。 默认任何被压缩的节都会以压缩形式展示。若要查看解压后的内容，请在命令行中添加 `-Z` 选项。

- -S, --source
尽可能穿插显示源码与反汇编，自动隐含 `-d`。

- --show-all-symbols
反汇编时显示匹配某地址的所有符号，而非仅第一个。

- --source-comment \[= <u>txt</u>]
效果与 `-S` 选项类似，但所有源代码行都会以<u>txt</u>作为前缀显示。 通常<u>txt</u>是一段注释字符串，用于将汇编代码与源代码区分开来。 如果未提供<u>txt</u>参数，则会使用默认字符串 `# `（井号加一个空格）。
- --prefix=<u>prefix</u>
指定与 -S 配合使用时，添加到绝对路径前的前缀<u>prefix</u>。

- --prefix-strip=<u>level</u>
指定需要从固化的绝对路径中剥离开头的多少级目录名。 若未同时指定 `--prefix=prefix`，则该选项无效。

- --show-raw-insn
反汇编时同时打印十六进制指令码与符号形式，除非使用了 `--prefix-addresses` 选项，否则这是默认行为。

- --no-show-raw-insn
反汇编时不打印指令字节，当使用`--prefix-addresses`选项时，这是默认行为。

- --insn-width=<u>width</u>
反汇编指令时单行显示<u>width</u>字节数的指令。

- --visualize-jumps[=color|=extended-color|=off]
以 ASCII 字符绘制线条，直观展示**函数内部跳转**的起始地址与目标地址之间的流向。 可选参数 `=color` 可使用基础终端颜色为输出内容上色。 此外，`=extended-color` 可使用 8 位色增强色彩效果，但并非所有终端都支持。若此前已启用跳转可视化，需要关闭该功能时，可使用 `visualize-jumps=off`。
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