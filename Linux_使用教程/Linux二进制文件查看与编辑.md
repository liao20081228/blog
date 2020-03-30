---
title: Linux二进制文件查看与编辑
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
<style>table{word-break:initial;}</style>
# 1 二进制文件查看
## 1.1 `od`
**命令**：od [OPTION]... [FILE]...
&emsp;&emsp;&emsp;od [-abcdfilosx]... [FILE] [[+]OFFSET[.][b]]
&emsp;&emsp;&emsp;od --traditional [OPTION]... [FILE] [[+]OFFSET[.][b] [+][LABEL][.][b]]
**描述**：
* 读取FILE并以指定格式输出到标准输出，默认为每两个字节用六个八进制数表示。
*  如果有多个FILE参数，以列出的顺序连接它们以形成输入。如果没有FILE，或者FILE是 - ，则读取标准输入。
*  如果第一和第二调用格式都适用时，第二种形式假定最后一个操作数以+或（如果有2个操作数）以数字开头。 
*  OFFSET操作数表示-j OFFSET。 
*  LABEL是打印的第一个字节的伪地址，在转储进行时递增。 
*  对于OFFSET和LABEL，前缀为0x或0X的十六进制; 后缀可以是b（表示512）。
*  长选项的参数也适用于短选项。
 
 
|短选项|长选项|秒速|
|:--|:--|:--|
|-A | ---address-radix=RADIX| 文件偏移量的输出格式; RADIX是[doxn]之一，对应十进制，八进制，十六进制或不出出偏移量
||--endian={big\|little}| 根据指定的顺序交换输入字节,big为大端，little为小端
|-j |--skip-bytes=BYTES|从开始跳过BYTES个输入字节|
|-N |--read-bytes=BYTES|只读取BYTES个输入字节|
| -S| --strings=BYTES|输出至少BYTES图形字符; 未指定BYTES时默认为3
|-t  |--format=TYPE|选择输出格式
| -v| --output-duplicatelicates|显示所有数据，如果不用-v则对于连续重复的输出行会使用星号表示
|-w|--width[=BYTES]|每行输出BYTES个字节，默认为32
|| --traditional|接受上面第三种调用形式的参数
||--help| 显示帮助信息并退出
||--version|显示版本信息并退出|
|&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|

 * TYPE由以下一个或多个规范组成：
  
|新格式|描述|
|:--|:--|
|a | 每个字节用 一个ASCII字符显示，但使用小写的控制字符名来显示控制字符。|
|c | 每个字节用 一个ASCII字符显示。除了可通过标准转义符表示的那些非打印字符外，其余非打印字符显示为三个八进制字符，用零填充。
|d[SIZE] |每SIZE个字节用一个带符号的十进制整数表示|
|f[SIZE] | 每SIZE个字节用一个浮点数表示|
|o[SIZE] |每SIZE个字节用一个八进制数表示|
|u[SIZE] |每SIZE个字节用一个无符号的十进制整数表示
|x[SIZE] | 每SIZE个字节用一个十六进制整数表示|
       
* SIZE是一个数字 
	* TYPE为 [doux]之一时， SIZE还可以为 ：
		* C ，其值等于 sizeof(char), 
		* S  ，其值等于 sizeof(short),
		* I ，其值等于sizeof(int) 
		* L ，其值等于 for sizeof(long).
	* TYPE为f时， SIZE还可以为 ：
		*  F ，其值等于 sizeof(float), 
		*  D ，其值等于 sizeof(double) or 
		*  L ，其值等于sizeof(long double)

* 将z添加到任何TYPE之后，则会在每个输出行的末尾显示可打印字符。
* 传统格式规范可以混合使用：

|**传统格式**|**新格式**|描述|
|:--|:--|:--|
|-a |-t a|选择命名字符，忽略高位
|-b|-t o1|选择八进制字节
|-c|-t c|选择可打印字符或反斜杠转义符
|-d|-t u2|选择无符号十进制2字节单位
|-f|-t fF|选择浮点数
|-i|-t dI|选择十进制整数
|-l|-t dL|选择十进制长度
|-o|-t o2|选择八进制2字节单位
|-s|-t d2|选择十进制2字节单位
|-x|-t x2|选择十六进制2字节单位


## 1.2 `hexdump`
**命令**：hexdump [-bcCdovx] [-e format_string] [-f format_file] [-n length] [-s skip] file ...
&emsp;&emsp;&emsp;hd [-bcdovx] [-e format_string] [-f format_file] [-n length] [-s skip] file ...
**描述**：hexdump实用程序以用户指定的格式显示指定的文件或标准输入（如果未指定文件）。

|选项|描述|
|:--|:--|
|-b|每字节用三个八进制数显示，每行16个字节，用空格分开，第一列为十六进制的偏移量|
|-c|每字节用ASCII字符显示。每行16个字节，用空格分开，第一列为十六进制的偏移量|
|-C|每字节用两个十六进制数显示，末尾用对应的ASCII字符显示，每行16个字节，用空格分开，第一列为十六进制的偏移量。hd命令默认此选项|
|-d|每两字节用五个十进制数表示，每行16个字节，用空格分开，第一列为十六进制的偏移量|
|-e format_string|根据指定的格式字符串显示数据|
|-f format_file|根据指定的格式文件显示数据，该文件包含一个或多个换行符分隔的格式字符串。以#开头的行和空行会被忽略|
|-n length| 只显示length个字节|
|-o|每两字节用六个八进制数表示，每行16个字节，用空格分开，第一列为十六进制的偏移量|
|-s offset|从第offset个字节开始显示，可以带有单位b表示512，k表示1024,m表示1024^2^。offset带有前导0但不是十六进制数时被视为8进制数。
|-v|显示所有数据，对于重复的输出行，如果不加-v则其余的所有重复行用一个`*`表示。
|-x|每两字节用四个十六进制数表示，每行16个字节，用空格分开，第一列为十六进制的偏移量
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||

&emsp;&emsp;**格式字符串**由任意数量的以空格隔开的格式单元组成，包含在一对单引号中。每个格式单元包括字节计数，迭代计数，格式 三部分。形如：`'迭代计数/字节计数 "format1" "format2" 迭代计数/字节计数 "format1" "format2"  ...'`。

* 字节计数是一个可选的正整数，表示每几个字节使用format1来显示（比如每字节用三个八进制数显示）
	*  `%_c, %_p, %_u, %c ` 只支持字节计数为1。
    *   `%d, %i, %o, %u, %X, %x`  支持字节计数为1，2，4（默认）。
    * `%E, %e, %f, %G, %g `    支持字节计数为4，8（默认），12。 
* 迭代计数是一个可选的正整数，缺省为1，表示每几个字节使用format2来显示。通常迭代计数要大于字节计数。（比如每行16个字节）
* format1 format2 ....必须用双引号包围，且format1不可省略。
	* **与C函数printf类似**，但有几点不同：
		* 星号（\*）不能用作域宽或精度。在printf中如果域宽和精度用星号则表示其由紧随的参数决定。
		* 对每个转换字符`s`都需要指定字节计数或精度（与fprintf不同，printf如果未指定精度则打印整个字符串）。
		* 不支持转换字符“％”，“h”，“l”，“n”，“p”和“q”。
		* 支持C标准中描述的单字符转义序列：
			* NUL  \0
			* 警告  \a
			* 退格  \b
			* 换页  \f
			* 换行   \n
			* 回车   \r
			* 水平制表   \t
			* 垂直制表   \v
	* 支持以下附加转换字符串：
		* `_a[d|o|x]`   以十进制，八进制,十六进制显示偏移量。
		* `_A[dox]`   与`_a`转换字符串类似，只是在处理完所有输入数据后才执行一次。
        * `_c` 输出默认字符集中的字符。除了可通过标准转义符号表示的那些非打印字符外，其余非打印字符显示为三个八进制字符，用零填充。
        * ` _p`输出默认字符集中的字符。非打印字符显示为单个“.”。
        * `_u`  输出US ASCII字符，但使用小写的控制字符名来显示控制字符。大于0xff的十六进制的字符显示为十六进制字符串。
	* 如果未指定格式字符串，则默认显示等效于指定-x选项。

## 1.3 `xxd`

**命令**：xxd -h[elp]
&emsp;&emsp;&emsp;xxd [options] [infile [outfile]]
 &emsp;&emsp;&emsp;xxd -r[evert] [options] [infile [outfile]]
**描述**：
* xxd创建给定文件或标准输入的十六进制转储。
*  它还可以将十六进制转储转换回其原始二进制形式。 
*  与uuencode和uudecode一样，它允许以“邮件安全”的ASCII字符表示方式来传输二进制数据，且具有解码到标准输出的优点。 
*  而且它可以用于执行二进制文件补丁。
*  如果没有给出infile，或将infile指定为“ - ”字符，则从标准输入中获取输入。 如果没有给出outfile，或将outfile指定为“ - ”字符，结果将被发送到标准输出。
*  请注意：xxd使用的是“lazy”解析器，它不会检查超过选项字母的其它字母，除非该选项后跟参数。 因此所有选项应放在infile的前面，并且选项不能连写。
*  单个选项字母与其参数之间的空格是可选的。 可以使用十进制，十六进制或八进制表示法指定选项的参数。 因此-c8，-c 8，-c 010和-cols 8都是等价的。

|短选项|长选项|描述|
|:--|:--|:--|
|-a|-autoskip|使用星号代替空行|
|-b|-bits|每个字节用8个二进制数表示，每个输出行前面是十六进制的偏移量，后面是对应的ASCII字符|
|-c cols | -cols cols|每个输出行 cols个字节，默认 16 (-i: 12, -ps: 30, -b: 6).最大256.
-E |-EBCDIC|将右侧列中的字符编码从ASCII更改为EBCDIC。 这不会改变十六进制表示。 与-r，-p或-i组合使用该选项毫无意义。
|-e||切换到little-endian hexdump。 此选项将字节分组视为以little-endian字节顺序排列的。 可以使用-g更改4字节的默认分组。 此选项仅适用于hexdump，而ASCII（或EBCDIC）表示不变。 命令行开关-r，-p，-i不适用于此模式。
|-g bytes | -groupsize bytes|用空格分隔每个bytes字节的输出。指定-g 0以禁止分组。 Bytes在普通模式下默认为2，在little-endian模式下默认为4，在位模式下默认为1。分组不适用于postscript或include style。
|-h | -help|显示帮助信息并退出|
|-i | |以C无符号字符数组显示. 数组名为文件名，除非从标准输入读入.
|-l len | -len len|只输出len个字节|
|-o offset||将offset添加到被显示的文件位置
|-p | -ps  -postscript  -plain|连续十六进制输出。也称为普通hexdump风格。
|-r | -revert|反向操作：将hexdump转换（或补丁）为二进制。 如果没有输出到stdout，xxd会写入其输出文件而不截断它。 使用组合-r -p可以读取没有偏移信息且没有特定列布局的普通十六进制转储。 任何地方都允许使用额外的空格和换行符。
||-seek offset|在-r后使用：将offset恢复为在hexdump中找到的文件位置。
|-s [+][-]seek|| 从seek字节开始输出。  +表示相对于开头，-表示相对于结尾
| -u||使用大写十六进制字母。 默认为小写。
|-v | -version| 显示版本。
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||


# 2 二进制文件编辑 
------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
