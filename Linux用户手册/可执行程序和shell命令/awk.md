---
title: awk
tags: Linux用户命令
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Linux man手册页》</font>当前版本为sed4.7。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 名称

gawk——模式扫描和处理语言。

# 2 概要

``` shell
	gawk [ POSIX or GNU style options ] -f program-file [ -- ] file ...
	gawk [ POSIX or GNU style options ] [ -- ] program-text file ...
```
# 3 说明

Gawk是GNU工程对AWK编程语言的实现。它符合POSIX 1003.1标准。该版本基于Aho，Kernighan和Weinberger撰写的《AWK编程语言》中的描述。 Gawk提供了Brian Kernighan awk中的额外功能以及许多特定于GNU的扩展。

命令行由gawk本身选项、AWK程序文本（如果未通过-f或--include选项提供）、在ARGC和ARGV预定义的AWK变量中的可用值组成。

使用--profile选项调用gawk时，它将从程序的执行中收集性能分析统计信息。 Gawk在这种模式下运行速度较慢，并且在完成后会自动在文件awkprof.out中生成一个执行配置文件。请参阅下面的--profile选项。

Gawk还具有集成的调试器。可以通过在命令行中指定--debug选项来启动交互式调试会话。在这种执行方式下，gawk将加载AWK源代码，然后提示您调试命令。 Gawk只能调试-f和--include选项提供的AWK程序源码。调试器文档在[高效AWK编程](#effective_awk_programing)中。

# 4 选项格式

Gawk选项可以是POSIX风格的单字符选项，或者GNU风格的长选项。POSIX选项是以“-”开始，而GNU长选项是以“--”开始。GNU特定功能和POSIX必须功能都提供了长选项。

特定于Gawk的选项通常以长选项形式使用。 长选项的参数可以紧跟在选项后的“=”符号之后（中间没有空格），也可以在下一个命令行参数中提供。 只要缩写保持唯一，就可以长选项就可以缩写。

此外，每一个长选项与偶一个相应的短选项，因此可以在 #!可执行脚本中使用该选项的功能。

# 5 选项
Gawk接受以下选项。 首先列出标准选项，然后列出gawk扩展的选项。按短选项按字母顺序列出。
|短选项|长选项|说明|
|:--|:--|:--|
|-b|--characters-as-bytes|将所有输入数据视为单字节字符。 换句话说，在尝试将字符串作为多字节字符处理时，不必特别注意语言环境信息。 --posix选项将覆盖此选项。|
|-c|--traditional|以兼容模式运行。 在兼容模式下，gawk的行为与Brian Kernighan awk相同； 没有特定于GNU的扩展。 有关更多信息，请参见下面的[GNU扩展](#GNU_EXTENSIONS)。|
|-C|--copyright|在标准输出上打印GNU版权信息的简短版本，然后成功退出。|
|-d\[file]|--dump-variables\[=file]|将全局变量以及它们的类型和最终值的排序列表打印到文件file中。如果未提供文件file，则gawk在当前目录中使用名为awkvars.out的文件。列出所有全局变量是在程序中查找打字错误的好方法。如果您的大型程序具有很多函数，并且还希望确保您的函数不会无意中使用本应是局部变量的全局变量，那么您也可以使用此选项。（使用i，j等简单变量名称时，这是一个特别容易犯的错误。）|
|-D\[file]|--debug\[=file]|启用AWK程序调试。默认情况下，调试器从键盘（标准输入）以交互方式读取命令。可选的file参数指定一个文件，其中包含命令列表，以供调试器以非交互方式执行。|
|-e program-text|--source program-text|使用program-text作为AWK程序源代码。通过此选项，可以轻松地将库函数（通过-f和--include选项使用）与在命令行中输入的源代码混合在一起。它主要用于Shell脚本中的中大型AWK程序。通过-e提供的每个参数都被视为以隐式@namespace "awk"语句开头。|
|-E file|--exec file|与-f相似，但是，此选项是最后处理的选项。这个选项与＃!脚本一起使用，以避免在命令行中从URL传递选项或源代码（！），尤其是CGI应用程序脚本。此选项禁用命令行变量赋值。|
|-f program-file|--file program-file|从文件program-file中读取AWK程序，而不是从第一个命令行参数中读取。可以使用多个-f（或--file）选项。使用-f读取的文件被视为以隐式的@namespace "awk"语句开头。|
|-F fs|--field-separator fs|将fs用作输入字段分隔符（预定义变量FS的值）。|
|-g|--gen-pot|扫描并解析AWK程序，并在标准输出上生成GNU .pot（可移植对象模板）格式文件，其中包含该程序中所有可本地化字符串条目。程序本身未执行。有关.pot文件的更多信息，请参见[GNU gettext](#gnu_gettext)分发。|
|-h|--help|在标准输出上打印可用选项的简短摘要。 （根据GNU编码标准，这选项会导致立即成功退出。）
|-i include-file|--include include-file|加载awk源码库。这将使用环境变量AWKPATH来搜索该库。如果首次搜索失败，则在加上.awk后缀之后再次尝试。该文件将仅加载一次（即消除重复），并且这些代码不构成主程序源码。用--include读取的文件被视为以隐式@namespace “awk”语句开头。|
|-l lib|--load lib|从共享库库中加载gawk扩展。 这将使用环境变量AWKLIBPATH来搜索该库。 如果首次搜索失败，则在为加上平台默认共享库后缀后再次尝试。 库的初始化例程应命名为dl_load（）。|
|-L \[value]|--lint\[=value]|提供有关可疑或不可移植到其他AWK实现的构造的警告。 如果使用fatal的可选参数，则温柔警告会变成致命错误。 这可能很激烈，但是使用它肯定会鼓励开发更清洁的AWK程序。 使用invalid可选参数，仅发出有关确实无效的警告。 （这尚未完全实现）使用no-ext可选参数，将禁用有关gawk扩展的警告。|
|-M|--bignum|对数字强制执行任意精度计算。如果编译gawk未使用GNU MPFR和GMP库，则此选项无效。（在这种情况下，gawk会发出警告。）|
| -n|--non-decimal-data|识别输入数据中的八进制和十六进制值。请谨慎使用此选项！|
|-N|--use-lc-numeric|强制gawk在解析输入数据时使用语言环境的小数点字符。尽管POSIX标准要求此行为，并且当--posix生效时gawk会要求这样做，但默认情况是遵循传统行为，并使用句点作为小数点，即使在句点不是小数点字符的语言环境中也是如此。此选项将覆盖默认行为，而没有--posix选项的严格要求。|
|-o\[file]|--pretty-print\[=file]|将程序的完美打印版本输出到文件file中。如果未提供文件file，则gawk在当前目录中使用名为awkprof.out的文件。此选项暗示-no-optimize。|
|-O| --optimize|在程序的内部表示形式上启用gawk的默认优化。当前，这仅包括简单的常量折叠（一种编译优化技术）。默认情况下，此选项处于启用状态。|
|-p\[prof-file]|-profile\[=prof-file]|启动性能分析会话，然后将性能数据发送到prof-file。默认值为awkprof.out。该配置文件在左边包含程序中每个语句的执行计数，以及每个用户定义函数的函数调用计数。此选项暗示--no-optimize。|
|-P|--posix|这将打开兼容模式，并具有以下限制：<br />&emsp;&emsp;无法识别\x转义序列<br />&emsp;&emsp;在?和:后无法继续<br />&emsp;&emsp;无法识别关键字function的同义词func<br />&emsp;&emsp;不能使用\*\*和\*\*=来代替\^和\^=|
|-r|--re-interval|在正则表达式匹配中启用间隔表达式的使用（请参见下面的[正则表达式](#regexp)）。传统上，间隔表达式在AWK语言中不可用。 POSIX标准添加了它们，以使awk和egrep彼此一致。它们是默认启用的，但此选项仍可与--traditional一起使用。|
|-s|--no-optimize|在程序的内部表示形式上禁用gawk的默认优化。|
|-S|--sandbox|在沙盒模式下运行gawk，禁用system()函数、getline输入重定向，print和printf输出重定向，以及加载动态扩展。命令执行（通过管道）也被禁用。这有效地阻止了脚本访问本地资源，命令行上指定的文件除外。|
|-t|--lint-old|提供有关不可移植到UNIX awk原始版本的构造的警告。|
|-v var=val|--assign var=val|在程序开始执行之前，将值val分配给变量var。这样的变量值可用于AWK程序的BEGIN规则。|
|-V|--version|在标准输出上打印该gawk特定副本的版本信息。 这主要用于了解当前系统上的gawk是否是自由软件基金会正在分发的最新版本。 报告bug时，这也很有用。 （根据GNU编码标准，此选项会导致立即成功退出。）|
|--||选项结束信号。 这允许AWK程序的其他参数以“-”开头。 这与大多数其他POSIX程序使用的参数解析约定保持一致。|

在兼容模式下，任何其他选项均标记为无效，否则将被忽略。 在正常操作中，只要提供了程序文本，在ARGV数组中的未知选项会传递给AWK程序进行处理。这对于通过＃!可执行解释器机制运行AWK程序特别有用。|

为了实现POSIX兼容性，可以使用-W选项，后跟长选项的名称。

# 6 AWK程序执行
AWK程序由一系列可选指令、模式操作语句和可选的函数定义组成。

``` awk
	@include "filename"
	@load "filename"
	@namespace "name"
	pattern   { action statements }
	function name(parameter list) { statements }
```
Gawk首先从program-files（如果指定了）、--source的参数、或命令行上的第一个非选项参数中读取程序源码。 -f和--source选项可以在命令行上多次使用。 Gawk读取程序文本就好像所有程序文件和命令行源码都连接在一起一样。这对于构建AWK函数库很有用，而不必在使用它们的每个新AWK程序中都包含它们。它还提供了将库函数与命令行程序混合的功能。

此外，以@include开头的行可用于将其他源文件包括到您的程序中，从而使库的使用更加容易。这等效于使用--include选项。

以@load开头的行可用于将扩展功能加载到您的程序中。这等效于使用--load选项。|

环境变量AWKPATH指定查找-f和--include选项指定的源文件时使用的搜索路径。如果该变量不存在，则默认路径为“.:/usr/local/share/awk”。 （实际目录可能会有所不同，具体取决于gawk的构建和安装方式。）如果给-f选项指定的文件名包含“/”字符，则不会执行路径搜索。

环境变量AWKLIBPATH指定查找--load选项指定的源文件时要使用的搜索路径。如果该变量不存在，则默认路径为“/usr/local/lib/gawk”。 （实际目录可能会有所不同，具体取决于gawk的构建和安装方式。）

Gawk按以下顺序执行AWK程序。首先，执行通过-v选项指定的所有变量赋值。接下来，gawk将程序编译为内部形式。然后，gawk执行BEGIN规则（如果有）中的代码，然后继续读取ARGV数组中的每个文件（直到ARGV\[ARGC-1]）。如果在命令行上没有指定文件，则gawk将读取标准输入。

如果命令行上的文件名格式为var=val，则将其视为变量赋值。变量var将被赋值为val。（这会在任何BEGIN规则被运行之后发生。）命令行变量赋值对于将值动态地赋值给AWK用于控制如何将输入分成字段和记录的变量最有用。如果在单个数据文件上需要多次传递，则命令行变量赋值对于控制状态也很有用。

如果ARGV的特定元素的值为空（""），则gawk将跳过它。

对于每个输入文件，如果存在BEGINFILE规则，则gawk将在处理文件内容之前执行关联的代码。同样，gawk在处理文件后执行与ENDFILE关联的代码。

对于输入中的每个记录，gawk进行测试以查看其是否与AWK程序中的任何模式匹配。对于记录匹配的每个模式，gawk将执行关联的操作。模式按照它们在程序中出现的顺序进行测试。

最后，在所有输入完毕后，gawk将执行END规则（如果有）中的代码。



## 6.1 命令行目录

根据POSIX，在awk命令行上指定的文件必须是文本文件。 如果不是文本文件，则行为是未定义的。 大多数版本的awk将命令行上的目录视为致命错误。

从gawk的4.0版本开始，命令行的目录会生成警告，但会被跳过。 如果给出了--posix或--traditional选项，则gawk将恢复为将命令行上的目录视为致命错误。

# 7 变量、记录、字段
AWK变量是动态的；它们在首次使用时就存在。它们的值可以是浮点数或字符串，或两者兼有，这取决于它们的使用方式。此外，gawk允许变量具有正则表达式类型。 AWK还有一维数组；可以模拟具有多维数组。 Gawk提供了真正的数组；请参阅后面的[数组](#arrays)。在程序运行时会设置几个预定义的变量；这些内容因为需要进行了描述，并在总结在后面。

## 7.1 记录

通常，记录由换行符分隔。您可以通过给内建变量RS赋值来控制记录的分隔方式。如果RS是任何单个字符，则该字符将分隔记录。否则，RS是一个正则表达式。输入中与此正则表达式匹配的文本将记录分隔开。但是，在兼容模式下，仅将其字符串的第一个字符用于分隔记录。如果将RS设置为空字符串，那么记录将由空行分隔。当RS设置为空字符串时，除FS可能具有的值外，换行符还始终充当字段分隔符。注意：当RS为正则表达式时，IGNORECASE的值（请[参见下文](#IGNORECASE)）会影响如何分隔记录。
<span id='filed'></span>
## 7.2 字段
读取每个输入的记录时，gawk会将FS变量的值用作字段分隔符以将记录分成多个字段。如果FS是单个字符，则字段由该字符分割。如果FS是空字符串，则各个字符都将成为一个单独的字段。否则，FS应该是完整的正则表达式。在特殊情况下，FS是单个空格，字段之间通过空格、制表符、换行符分割。注意：当FS为正则表达式时，IGNORECASE的值（请[参见下文](#IGNORECASE)）会影响如何拆分字段。

如果将FIELDWIDTHS变量设置为以空格分隔的数字列表，则每个字段应具有固定的宽度，且gawk使用指定的宽度拆分记录。每个字段宽度可以可选地以冒号分隔的值开头，该值指定在字段开始之前要跳过的字符数。 FS的值将被忽略。为FS或FPAT赋新值将覆盖FIELDFIDTHS的使用。

类似的，如果将FPAT变量设置为表示正则表达式的字符串，则每个字段均由与该正则表达式匹配的文本组成。在这种情况下，正则表达式描述字段本身，而不是分割字段的文本。为FS或FIELDWIDTHS分配新值将覆盖FPAT的使用。

输入记录中的每个字段都可以通过其位置引用：\$1，\$2，依此类推。 \$0是整个记录，包括前导和尾随空白。字段不需通过常量引用：

``` awk
	n = 5
	print $n
```

打印输入记录中的第五个字段。

变量NF为输入记录中字段的总数。

对不存在的字段（即\$NF之后的字段）的引用将产生空字符串。但是，赋值给不存在的字段（例如，\$(NF + 2)= 5）会增加NF的值，创建以空字符串作为值的中间字段，并导致重新计算\$0的值，字段之间用OFS的值分割。引用负数字段会导致致命错误。减小NF会导致超过新值的字段的值丢失，并重新计算\$0的值，其字段之间用OFS值分割。

为已有字段赋值值会导致在引用\$0时重建整个记录。 同样，为\$0赋值会导致记录被重拆分，从而为字段创建新值。

## 7.3 内建变量
Gawk的内置变量如下：

|变量|说明|
|:--|:--|
|ARGC|命令行参数的数量（不包括gawk的选项或程序源码）|
|ARGIND|正在处理的当前文件在ARGV中的索引|
|ARGV |命令行参数数组。该数组的索引从0到ARGC-1。动态更改ARGV的内容可以控制用于数据的文件。|
|BINMODE|在非POSIX系统上，为所有文件I/O指定使用“二进制”模式。数值1、2、3分别指定输入文件、输出文件、所有文件应使用二进制I/O。字符串值"r"或"w"指定输入文件或输出文件使用二进制I/O。字符串值"rw"或"wr"指定所有文件应使用二进制I/O。任何其他字符串值都被视为"rw"，但会生成警告消息。|
|CONVFMT|默认情况下，数字的转换格式为"％.6g"。|
|ENVIRON|包含当前环境值的数组。该数组通过环境变量进行索引，每个元素都是该变量的值（例如ENVIRON["HOME"]可能是"/home/ arnold"）。<br />在POSIX模式下，更改此数组不会影响gawk通过重定向或system()函数生成的程序所看到的环境。否则，gawk将更新其实际环境，以便其生成的程序可以看到更改。|
| ERRNO|如果在重定向getline、读取getline、close()期间发生错误，则将ERRNO设置为描述该错误的字符串。该值可以在非英语语言环境中进行翻译。如果ERRNO中的字符串与errno(3)变量中的系统错误相匹配，则可以在PROCINFO["errno"]中找到该数值。对于非系统错误，PROCINFO["errno"]将为零。|
|FIELDWIDTHS|以空格分隔的字段宽度列表。设置后，gawk会将输入解析为固定宽度的字段，而不是使用FS变量的值作为字段分隔符。可选地，每个字段宽度前面都可以有一个冒号分隔的值，该值指定在字段开始之前要跳过的字符数。参见前面的[字段](#filed)。|
|FILENAME| 当前输入文件的名称。如果在命令行上未指定文件，则FILENAME的值为“-”。但是，FILENAME在BEGIN规则内是未定义的（除非由getline设置）。|
|FNR|当前输入文件中的输入记录号。|
|FPAT|描述记录中字段内容的正则表达式。设置后，gawk会将输入解析为与正则表达式匹配的字段，而不是使用FS的值作为字段分割符。参见前面的[字段](#filed)。|
|FS|输入字段分割符，默认为空格。参见前面的[字段](#filed)。|
|FUNCTAB |一个数组，其索引和对IGNORECASE|应值是程序中所有用户定义或扩展函数的名称。注意：您不能与FUNCTAB数组一起使用delete语句。|
|<span id='IGNORECASE'>IGNORECASE</span>| 控制所有正则表达式和字符串操作的是否区分大小写。如果IGNORECASE具有非零值，则在规则中进行字符串比较和模式匹配，使用FS和FPAT进行字段拆分，使用RS进行记录分隔，使用\~和!\~匹配正则表达式，以及内置函数gensub()、gsub()、index()、match()、patsplit()、split()和sub()在进行正则表达式操作时都会忽略大小写。注意：数组下标不受影响，但是asort()和asorti()函数会受到影响。<br /><br />因此，如果IGNORECASE不等于零，则/aB/匹配所有字符串“ab”，“aB”，“Ab”和“AB”。与所有AWK变量一样，IGNORECASE的初始值为零，因此所有正则表达式和字符串操作通常区分大小写。|
|LINT|在AWK程序中提供--lint选项的动态控制。 如果为true，gawk将打印温和警告。 如果为fasle，则不会。 当为LINT赋值"fatal"时，温和警告将成为致命错误，与--lint=fatal一样。 其他任何真值只会显示警告。|
|NF|当前输入记录中的字段数。|
|NR|到目前为止看到的输入记录总数。|
|OFMT|默认情况下，数字的输出格式为"％.6g"。|
|OFS|输出字段分隔符，默认为空格。|
|ORS|输出记录分隔符，默认情况下为换行符。|
|PREC|任意精度浮点数的有效精度，默认为53。|
|PROCINFO|该数组的元素提供了访问正在运行的AWK程序的信息的接口。参见下面的[PROCINFO](#PROCINFO)。|
|ROUNDMODE|用于对数字进行任意精度算术的舍入模式，默认情况下为"N"（IEEE-754 roundTiesToEven模式）。可接受的值为：<br /><br /> "A" 或 "a"&emsp;&emsp;roundAwayFromZero。仅当您的GNU MPFR库版本支持roundAwayFromZero才可用。<br /><br />"D" 或 "d"&emsp;&emsp; roundTowardNegative。<br /><br />"N" 或 "n"&emsp;&emsp;roundTiesToEven。<br /><br />"U" or "u" &emsp;&emsp;roundTowardPositive。<br /><br />"Z" or "z" &emsp;&emsp;roundTowardZero。|
|RS|输入记录分隔符，默认情况下为换行符。|
|RT|记录终止符。 Gawk将RT设置为与RS指定的字符或正则表达式匹配的输入文本。|
|RSTART|match()匹配的第一个字符的索引。如果没有匹配，则为0。（这意味着字符索引从1开始。）|
|RLENGTH |match()匹配的字符串的长度；如果没有匹配，则为-1。|
|SUBSPEP|默认情况下，用于分隔数组元素中多个下标的字符串“\034”。|
|SYMTAB|一个数组，其索引是程序中所有当前定义的全局变量和数组的名称。该数组可用于间接访问以读取或写入变量的值：<br /><br />&emsp;&emsp; foo = 5<br /><br />&emsp;&emsp;SYMTAB["foo"] = 4<br /><br />&emsp;&emsp;print foo    # prints 4<br /><br />typeof()函数可用于测试SYMTAB中的元素是否为数组。您不能将delete语句与SYMTAB数组一起使用，也不能赋值给索引不是变量名的元素。|
|TEXTDOMAIN |AWK程序的文本域；用于查找程序字符的本地化翻译。|

在某些系统上，PROCINFO数组中可能存在一些元素，从"group1"到"groupn"，n是进程具有的补充组的数量。使用in运算符测试这些元素。以下元素保证可以使用：

|<span id='PROCINFO'>PROCINFO数组元素</span>|说明|
|:--|:--|
|PROCINFO["argv"]|gawk在C语言级别上接收到的命令行参数。下标从零开始。|
|PROCINFO["egid"]|系统调用getegid(2)的值。|
|PROCINFO["errno"] |当ERRNO设置为关联的错误消息时，errno(3)的值。|
|PROCINFO["euid"]|系统调用geteuid(2)的值。|
|PROCINFO ["FS"]| 如果使用FS进行字段拆分则为"FS"；如果使用FPAT进行字段拆分，则为"FPAT"；如果使用FIELDWIDTHS进行字段拆分，则为"FIELDWIDTHS"；如果使用API输入解析器拆分字段，则为"API"。|
|PROCINFO["gid"]|系统调用getgid(2)的值。|
|PROCINFO["identifiers"] |子数组，由AWK程序文本中使用的所有标识符的名称索引。这些值表示gawk在完成对程序的解析之后对标识符的了解；它们不会在程序运行时更新。对于每个标识符，元素的值是以下值之一：<br /><br />"array"&emsp;&emsp;标识符是一个数组。<br /><br />"builtin"&emsp;&emsp;标识符是内置函数。<br /><br />"extension"&emsp;&emsp; 标识符是通过@load或--load加载的扩展函数。<br /><br />"scalar"&emsp;&emsp;标识符是标量。<br /><br />"untyped"&emsp;&emsp;标识符是无类型的（可以用作标量或数组，gawk尚不知道）。<br /><br />"user"&emsp;&emsp;标识符是用户定义的函数。|
|PROCINFO["pgrpid"]|系统调用getpgrp(2)的值。|
|PROCINFO["pid"]|系统调用getpid(2)的值。
|PROCINFO["platform"]|一个字符串，指示编译gawk的平台。它是以下之一：<br /><br />"djgpp"，"mingw"&emsp;&emsp;Microsoft Windows，分别使用DJGPP或MinGW的。<br /><br />"os2"&emsp;&emsp;OS/2。<br /><br />"posix"&emsp;&emsp;GNU/Linux，Cygwin，Mac OS X和传统的Unix系统。<br /><br />"vms"&emsp;&emsp;OpenVMS或Vax/VMS。<br /><br />|
|PROCINFO["ppid"]|系统调用getppid(2)的值。
|PROCINFO["strftime"]|strftime()的默认时间格式字符串。更改其值会影响无参数调用strftime()时格式化时间值的方式。|
|PROCINFO["uid"]系统调用getuid(2)的值。|
|PROCINFO["version"]|gawk的版本。|

如果可以加载动态扩展，则存在以下元素：
|PROCINFO数组元素|说明|
|:--|:--|
| PROCINFO["api_major"]|扩展API的主要版本。|
|PROCINFO["api_minor"]|扩展API的次要版本。|

如果将MPFR支持编译到gawk中，则可以使用以下元素：
|PROCINFO数组元素|说明|
|:--|:--|
|PROCINFO["gmp_version"]| Gaw GMP中用于任意精度数字支持的GNU GMP库版本。|
| PROCINFO["mpfr_version"]| Gaw MP中用于任意精度数字支持的GNU MPFR库版本。|
|PROCINFO["prec_max"]|GNU MPFR库支持的任意精度浮点数的最大精度。|
|PROCINFO["prec_min"]|GNU MPFR库允许的任意精度浮点数的最低精度。|

程序可以设置以下元素来更改gawk的行为：

|PROCINFO数组元素|说明|
|:--|:--|
|PROCINFO["NONFATAL"]|如果存在，则所有重定向的I/O错误都不会致命。|
|PROCINFO["name", "NONFATAL"]|使指明的I/O错误非致命。|
|PROCINFO["command", "pty"]|使用伪tty与命令进行双向通信，而不是设置两个单向管道。|
|PROCINFO["input", "READ_TIMEOUT"]|从输入读取数据的超时时间（以毫秒为单位），其中输入是重定向字符串或文件名。零或小于零的值表示没有超时设置。|
|PROCINFO["input", "RETRY"]|如果从输入读取数据时可能发生重试I/O错误，并且此数组项存在，则getline返回-2，而不是遵循默认行为返回-1，并将输入配置为不返回其他数据。可能重试的I/O错误是errno(3)的值为EAGAIN，EWOULDBLOCK，EINTR或ETIMEDOUT的错误。这在与PROCINFO["input", "READ_TIMEOUT"]联用，或在文件描述符已配置为以非阻塞方式运行的情况下很有用。|
|PROCINFO["sorted_in"]|如果PROCINFO中存在此元素，则其值控制在for循环中遍历数组元素的顺序。 支持的值是"@ind_str_asc"、"@ind_num_asc"、"@val_type_asc"、"@val_str_asc"、"@val_num_asc"、"@ind_str_desc"、"@ind_num_desc"、"@val_type_desc"、"@val_str_desc"、"@val_num_desc"和 "@unsorted"。 该值也可以是定义如下的比较函数的名称（以字符串形式）：<br /><br />&emsp;&emsp;function cmp_func(i1,v1,i2,v2)<br /><br />其中i1和i2是索引，而v1和v2是要比较的两个元素的对应值。 它应返回小于、等于或大于0的数字，具体取决于数组元素的排序方式。|
<span id='arrays'> </span>
## 7.4 数组

数组用用方括号（[]）之间的表达式作为下标。如果表达式是表达式列表（expr，expr ...），则数组下标是由每个表达式的（字符串）值的串联组成一个字符串，并由SUBSEP变量的值分隔。此功能用于模拟多维数组。例如：

``` awk
	i = "A"; j = "B"; k = "C"
	x[i, j, k] = "hello, world\n"
```
将字符串“hello，world\n”赋值给由字符串“A\034B\034C”索引的数组x的元素。AWK中的所有数组都是关联的，即由字符串值索引。

特殊运算符in可用于测试数组是否具有由特定值组成的索引：
```awk
	if (val in array)
		print array[val]
```
如果数组有多个下标，请在数组中使用(i，j)。

in运算符也可以在for循环中使用，以迭代数组的所有元素。 但是，数组结构中的(i,j)仅适用于测试，不适用于for循环。
可以使用delete语句从数组中删除元素。delete语句可以通过指定不带下标的数组名来删除数组的全部内容。

gawk支持真正的多维数组。 它不需要像C或C++中那样的数组是“矩形”的。 例如：

``` c++
	a[1] = 5
	a[2][1] = 6
	a[2][2] = 7
```
  注意：您可能需要告诉gawk，数组元素实际是一个子数组，以便在gawk需要数组的地方使用它（例如，在split()的第二个参数中）。 您可以通过在子数组中创建一个元素，然后使用delete语句将其删除来做到这一点。 

## 7.5 名称空间

Gawk提供了一种简单的名称空间功能，可帮助解决AWK中所有变量都是全局变量这一事实。

限定名称由两个简单标识符中间加一个冒号(::)组成。左侧标识符代表名称空间，右侧标识符是其中的变量。所有简单（非限定）名称都被视为在当前默认名称空间awk中。但是，即使当前名称空间不同，仅由大写字母组成的简单标识符也会被放入awk名称空间。

您可以使用@namespace “name”指令更改当前名称空间。

标准的预定义内置函数名不能用作名称空间的名称。由gawk提供的附加函数名可以用作名称空间名称或其他名称空间中的简单标识符。有关更多详细信息，请参见GAWK：[高效AWK编程](#effective_awk_programing)。

## 7.6 变量键入和转换
 变量和字段可以是（浮点数）数字、字符串或两者都是，也可以是正则表达式。变量的值如何解释取决于其上下文。如果在数字表达式中使用，它将被视为数字。如果用作字符串，它将被视为字符串。

要强制将变量视为数字，请向其添加零。要强制将其视为字符串，请将其与空字符串连接。

未初始化的变量的数字值为零，字符串值为""（空字符串）。

当必须将字符串转换为数字时，可以使用strtod(3)来完成。通过将CONVFMT的值作为sprintf(3)的格式字符串并将变量的数值作为参数来将数字转换为字符串。然而，即使AWK中的所有数字都是浮点数，整数值始终会转换为整数。因此，给定
```awk
              CONVFMT =“％2.2f”
              a = 12
              b = a""
```
变量b的字符串值为“12”，而不是“ 12.00”。

注意：在POSIX模式下运行时（例如使用--posix选项），请注意语言环境设置可能会干扰十进制数字的处理方式：输入给gawk的数字的十进制分隔符必须符合您的语言环境，可能是逗(,)或句点(.)。

Gawk执行以下比较：如果两个变量为数字，则将对其进行数字比较。如果一个值是数字，而另一个值是“数字字符串”，则比较也会以数字方式进行。否则，将数值转换为字符串并执行字符串比较。当然，将两个字符串变量是作为字符串进行比较。

请注意，字符串常量（例如“57”）不是数字字符串，而是字符串常量。 “数字字符串”的概念仅适用于字段、getline输入、FILENAME、ARGV元素、ENVIRON元素以及由split()或patsplit()创建的数字字符串数组的元素。基本思想是看起来像数字的用户输入且只有看起来像数字的用户输入应该以这种方式处理。

## 7.7 八进制和十六进制常数

您可以在AWK程序源代码中使用C风格的八进制和十六进制常量。例如，八进制值011等于十进制9，十六进制值0x11等于十进制17。

## 7.8 字符串常量
AWK中的字符串常量是用双引号引起来的字符序列（例如“value”）。在字符串中，可以识别某些转义序列，就像C语言中一样。它们是：
|转义序列|说明|
|:--|:--|
|\\\|反斜杠。|
|\a|“警报”字符；通常是ASCII BEL字符。|
|\b|退格键。|
|\f|换页符。|
|\n|换行符。|
|\r|回车。|
|\t|水平制表符。|
|\v|垂直制表符。|
|\xhex|数字。\x后面跟十六进制数字。\x后面跟两个十六进制数字被认为是转义序列的一部分。例如，“\x1B”是ASCII ESC（转义）字符。|
|\ddd|数字。由的1位，2位或3位八进制数序列表示的字符。例如，“\033”是ASCII ESC（转义）字符。|
|\c|文字字符c。|

在兼容模式下，当在正则表达式常量中使用时，由八进制和十六进制转义序列表示的字符将按字面处理。因此，/a\52b/等效于/a\*b/。

## 7.9 正则表达式常量

正则表达式常量是用正斜杠（例如/value/）括起来的一系列字符。 正则表达式匹配将在下面regexp更全面地描述。 请参阅[正则表达式](#regexp)。

前面描述的转义序列也可以在正则表达式常量中使用（例如/[\t\f\n\r\v]/匹配空白字符）。

Gawk提供强类型的正则表达式常量。 这些均以@符号开头（例如：@/value/）。 可以将此类常量分配给标量（变量，数组元素），并传递给用户定义的函数。 这样赋值的变量具有正则表达式类型。

  
------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Linux man手册页》</font>当前版本为sed4.7。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
