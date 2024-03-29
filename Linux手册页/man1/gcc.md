---
title: gcc
tags: gcc,g++,man1
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《gcc manpages》</font>， gcc当前版本为11.2.0，手册更新时间为2021.7.28。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
# 名称
gcc——GNU 项目 C 和 C++ 编译器。
# 概要
```bash
gcc [-c : -S : -E] [-std=standard]
	[-g] [-pg] [-Olevel]
	[-Wwarn...] [-Wpedantic]
	[-Idir...] [-Ldir...]
	[-Dmacro[=defn]...] [-Umacro]
	[-foption...] [-mmachine-option...]
	[-o outfile] [@file] infile...
```
这里只列出最有用的一些选项； 其余部分见下文。 **g++** 接受的选项几乎与 **gcc** 相同。
# 描述
当您调用 GCC 时，它通常会进行预处理、编译、汇编和链接。 “整体选项”允许您在中间阶段停止此过程。例如，**-c** 选项表示不运行链接器输出由汇编器输出的目标文件组成。

其他选项被传递到一个或多个处理阶段。一些选项控制预处理器，其他选项控制编译器本身。还有其他选项控制汇编器和链接器；其中大部分都没有在这里记录，因为您很少需要使用它们中的任何一个。

大多数可以与 GCC 一起使用的命令行选项对 C 程序有用。当一个选项只对另一种语言（通常是 C++）有用时，解释会明确说明。如果特定选项的描述未提及源语言，您可以将该选项用于所有支持的语言。

运行 GCC 的常用方法是运行名为 **gcc** 的可执行文件，或者在交叉编译时运行 <u>machine</u>**-gcc**，或者在运行特定版本的 GCC 时运行 <u>machine</u>**-gcc-**<u>version</u>。当您编译 C++ 程序时，您应该将 GCC 作为 **g++** 调用。

**gcc** 程序接受选项和文件名作为操作数。许多选项具有多字母名称；因此，多个单字母选项可能无法组合：**-dv** 与 **-d** **-v** 非常地不同。

您可以混合选项和其他参数。在大多数情况下，顺序并不重要。当您使用多个同类选项时，顺序很重要；例如，如果您多次指定 **-L**，将按指定的顺序搜索目录。此外，**-l** 选项的位置也很重要。

许多选项都有以 **-f** 或 **-W** 开头的长名称。例如，**-fmove-loop-invariants**、**-Wformat** 等。其中大部分有肯定和否定两种形式； **-ffoo** 的否定形式是 **-fno-foo**。本手册仅记录了这两种形式中的一种，以非默认形式为准。

某些选项采用一个或多个参数，通常由空格或等号 (=) 与选项名称分隔。除非另有说明，否则参数可以是数字或字符串。数字参数通常必须是小的无符号十进制或十六进制整数。十六进制参数必须以 **0x** 前缀开头。

指定某种大小阈值的选项的参数可以是任意大的十进制或十六进制整数，后跟指定字节倍数的字节大小后缀，例如“kB”和“KiB”分别代表千字节和千位字节，“MB”和“MiB”代表兆字节和兆位字节，“GB”和“GiB”代表千兆字节和千兆位字节，依此类推。此类参数在以下文本中由字节大小指定。有关二进制和十进制字节大小前缀的完整列表和说明，请参阅 NIST、IEC 和其他相关的国家和国际标准。

# 选项
## 选项总结
这是所有选项的摘要，按类型分组。 解释在后续章节。

<u>总体选项</u>
: **-c  -S  -E  -o** <u>file</u> **-dumpbase** <u>dumpbase</u>  **-dumpbase-ext** <u>auxdropsuf</u> **-dumpdir** <u>dumppfx</u>  **-x** <u>language</u> **-v  -###  --help**\[**=**<u>class</u>\[**,...**]]  **--target-help  --version -pass-exit-codes -pipe  -specs=**<u>file</u>  **-wrapper @**<u>file</u>  **-ffile-prefix-map=**<u>old</u>**=**<u>new</u> **-fplugin=**<u>file</u>  **-fplugin-arg-name=arg -fdump-ada-spec**\[**-slim**]  **-fada-spec-parent=**<u>unit</u>  -**fdump-go-spec=**<u>file</u>

<u>C语言选项</u>
: **-ansi  -std=**<u>standard</u>  **-fgnu89-inline  -fpermitted-flt-eval-methods=**<u>standard</u> **-aux-info** <u>filename</u>  **-fallow-parameterless-variadic-functions  -fno-asm  -fno-builtin -fno-builtin-**<u>function</u>  **-fgimple -fhosted  -ffreestanding -fopenacc  -fopenacc-dim=**<u>geom</u>  **-fopenmp  -fopenmp-simd -fms-extensions  -fplan9-extensions  -fsso-struct=**<u>endianness</u>  **-fallow-single-precision  -fcond-mismatch  -flax-vector-conversions  -fsigned-bitfields  -fsigned-char  -funsigned-bitfields  -funsigned-char**

<u>C++语言选项</u>
: **-fabi-version=**<u>n</u>  **-fno-access-control  -faligned-new=**<u>n</u>  **-fargs-in-order=**<u>n</u>  **-fchar8_t  -fcheck-new -fconstexpr-depth=**<u>n</u>  **-fconstexpr-cache-depth=**<u>n</u>  **-fconstexpr-loop-limit=**<u>n</u>  **-fconstexpr-ops-limit=**<u>n</u>  **-fno-elide-constructors -fno-enforce-eh-specs -fno-gnu-keywords -fno-implicit-templates -fno-implicit-inline-templates -fno-implement-inlines -fmodule-header**\[**=**<u>kind</u>] **-fmodule-only -fmodules-ts -fmodule-implicit-inline -fno-module-lazy -fmodule-mapper=**<u>specification</u> -fmodule-version-ignore -fms-extensions -fnew-inheriting-ctors -fnew-ttp-matching -fno-nonansi-builtins  -fnothrow-opt  -fno-operator-names -fno-optional-diags  -fpermissive -fno-pretty-templates -fno-rtti -fsized-deallocation -ftemplate-backtrace-limit=n -ftemplate-depth=n -fno-threadsafe-statics  -fuse-cxa-atexit -fno-weak  -nostdinc++ -fvisibility-inlines-hidden -fvisibility-ms-compat -fext-numeric-literals -flang-info-include-translate[=header] -flang-info-include-translate-not -flang-info-module-cmi[=module] -stdlib=libstdc++,libc++ -Wabi-tag  -Wcatch-value  -Wcatch-value=n -Wno-class-conversion  -Wclass-memaccess -Wcomma-subscript  -Wconditionally-supported -Wno-conversion-null  -Wctad-maybe-unsupported -Wctor-dtor-privacy  -Wno-delete-incomplete -Wdelete-non-virtual-dtor  -Wdeprecated-copy -Wdeprecated-copy-dtor -Wno-deprecated-enum-enum-conversion -Wno-deprecated-enum-float-conversion -Weffc++  -Wno-exceptions -Wextra-semi  -Wno-inaccessible-base -Wno-inherited-variadic-ctor -Wno-init-list-lifetime -Winvalid-imported-macros -Wno-invalid-offsetof  -Wno-literal-suffix -Wno-mismatched-new-delete -Wmismatched-tags -Wmultiple-inheritance -Wnamespaces  -Wnarrowing -Wnoexcept  -Wnoexcept-type  -Wnon-virtual-dtor -Wpessimizing-move  -Wno-placement-new  -Wplacement-new=n -Wrange-loop-construct -Wredundant-move -Wredundant-tags -Wreorder  -Wregister -Wstrict-null-sentinel  -Wno-subobject-linkage  -Wtemplates -Wno-non-template-friend  -Wold-style-cast -Woverloaded-virtual -Wno-pmf-conversions -Wsign-promo -Wsized-deallocation  -Wsuggest-final-methods -Wsuggest-final-types  -Wsuggest-override -Wno-terminate  -Wuseless-cast  -Wno-vexing-parse -Wvirtual-inheritance -Wno-virtual-move-assign  -Wvolatile  -Wzero-as-null-pointer-constant

## 控制输出类型的选项

编译最多可涉及四个阶段：预处理、完全编译、汇编和链接，始终按此顺序进行。 GCC 能够将多个文件预处理和编译为多个汇编器输入文件或一个汇编器输入文件； 然后每个汇编器输入文件生成一个目标文件，链接将所有目标文件（那些新编译的和指定为输入的）组合成一个可执行文件。

对于任何给定的输入文件，文件名后缀决定了进行何种编译：

<u>file</u>**.c**
必须进行预处理的C源代码。

<u>file</u>**.i**
: 不应进行预处理的C源代码。

<u>file</u>**.ii**
: 不应进行预处理的C++源代码。

<u>file</u>**.m**
: 必须进行预处理的Objective-C源代码。请注意，必须链接 <u>libobjc</u> 库才能使 Objective-C 程序工作。

<u>file</u>**.mi**
: 不应进行预处理的Objective-C源代码。

<u>file</u>**.mm**<br /><u>file</u>**.M**
: 必须进行预处理的Objective-C++源代码。请注意，必须链接 <u>libobjc</u> 库才能使 Objective-C++ 程序工作。请注意，**.M** 指的是大写字母 M。

<u>file</u>**.mii**
: 不应进行预处理的Objective-C源代码。

<u>file</u>**.h**
: 将被转换为预编译头文件（默认）的C、C++、Objective-C 或 Objective-C++ 头文件，或将被转换为 Ada specs的 C、C++ 头文件（通过 -fdump-ada-spec 开关）。

<u>file</u>**.cc**<br /><u>file</u>**.cp**<br /><u>file</u>**.cxx**<br /><u>file</u>**.cpp**<br /><u>file</u>**.CPP**<br /><u>file</u>**.c++**<br /><u>file</u>**.C**
: 必须进行预处理的C++源代码。请注意，在 **.cxx** 中，最后两个字母必须都是小写字母x。 同样，**.C** 指的是大写字母的 C。

<u>file</u>**.hh**<br /><u>file</u>**.H**<br /><u>file</u>**.hp**<br /><u>file</u>**.hxx**<br /><u>file</u>**.hpp**<br /><u>file</u>**.HPP**<br /><u>file</u>**.h++**<br /><u>file</u>**.tcc**
: 将被转换为预编译头文件（默认）或Ada specs的C++头文件。

<u>file</u>**.f**<br /><u>file</u>**.for**<br /><u>file</u>**.ftn**
: 不应进行预处理的固定形式的 Fortran 源代码。

<u>file</u>**.F**<br /><u>file</u>**.FOR**<br /><u>file</u>**.fpp**<br /><u>file</u>**.FPP**<br /><u>file</u>**.FTN**
: 必须进行预处理（使用传统的预处理器）的固定形式的 Fortran 源代码。

<u>file</u>**.f90**<br /><u>file</u>**.f95**<br /><u>file</u>**.f03**<br /><u>file</u>**.f08**
: 不应进行预处理的 Fortran 源代码。

<u>file</u>**.f90**<br /><u>file</u>**.f95**<br /><u>file</u>**.f03**<br /><u>file</u>**.f08**
: 必须进行预处理（使用传统的预处理器）的 Fortran 源代码。

<u>file</u>**.go**
: GO源代码。

<u>file</u>**.brig**
: BRIG 文件（HSAIL 的二进制表示）。

<u>file</u>**.d**
: D源代码。

<u>file</u>**.di**
: D接口文件。

<u>file</u>**.dd**
: D文档代码(Ddoc)。

<u>file</u>**.ads**
: 包含库单元声明（包、子程序或泛型或泛型实例化的声明）或库单元重命名声明（包、泛型或子程序重命名声明）的 Ada 源代码文件。 此类文件也称为<u>specs</u>。

<u>file</u>**.adb**
: 包含库单元体（子程序或包体）的 Ada 源代码文件。 此类文件也称为<u>bodies</u>。

<u>file</u>**.s**
: 不应进行预处理的汇编代码。

<u>file</u>**.S**<br /><u>file</u>**.sx**
: 必须进行预处理的汇编代码。

<u>other</u>
: 将被直接输入到链接器的目标文件。 任何没有可识别后缀的文件名都以这种方式处理。


您可以使用 **-x** 选项显式指定输入语言：

-x <u>language</u>
: 为以下输入文件明确指定语言<u>language</u>（而不是让编译器根据文件名后缀选择默认语言）。 此选项适用于所有后续输入文件，直到下一个 **-x** 选项。 语言<u>language</u>的可能值是：
&emsp;&emsp;c c-header cpp-output
&emsp;&emsp;c++  c++-header  c++-system-header c++-user-header c++-cpp-output
&emsp;&emsp;objective-c  objective-c-header  objective-c-cpp-output
&emsp;&emsp;objective-c++ objective-c++-header objective-c++-cpp-output
&emsp;&emsp;assembler  assembler-with-cpp
&emsp;&emsp;ada
&emsp;&emsp; d
&emsp;&emsp;f77  f77-cpp-input f95  f95-cpp-input
&emsp;&emsp;go
&emsp;&emsp;brig




# 作者
David MacKenzie。
# 报告错误
GNU coreutils 在线帮助：<https://www.gnu.org/software/coreutils/>
报告 stty 翻译错误：<https://translationproject.org/team/> 

# 版权
版权所有 © 2018 自由软件基金会, Inc. 许可证 GPLv3+：GNU GPL 版本 3 或更高版本 <https://gnu.org/licenses/gpl.html>。

这是免费软件：您可以自由更改和重新分发它。 在法律允许的范围内，不提供任何保证。
# 参见
完整文档位于：<https://www.gnu.org/software/coreutils/stty> 或通过本地获取：info '(coreutils) stty invocation'。

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《gcc manpages》</font>， gcc当前版本为11.2.0，手册更新时间为2021.7.28。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
