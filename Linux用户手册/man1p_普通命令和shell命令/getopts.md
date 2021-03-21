---
title: getopts
tags: shell命令,命令行参数处理
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《POSIX程序员手册2017》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 0 序言
该手册页是《POSIX程序员手册》的一部分。该接口的Linux实现可能有所不同（有关Linux行为的详细信息，请查看相应的Linux手册页），或者该接口在Linux上没有实现。

# 1 名称
getopts——解析程序命令行选项。

# 2 概要

```shell
getopts optstring name [arg ...]
```

# 3 说明

实用程序<u>getopts</u>从参数列表中检索选项和选项参数。它支持POSIX.1-2017的基本定义卷第12.2节实用程序语法准则中3~10所述的准则。

每次调用<u>getopts</u>，<u>getopts</u>都会将下一个选项字符放入由操作数<u>name</u>指定的shell变量中，并将要处理的下一个参数的索引放入shell变量<u>OPTIND</u>中。每次调用shell时，<u>OPTIND</u>都初始化为1。

当选项要求选项参数时，<u>getopts</u>将选项参数放置在在shell变量<u>OPTARG</u>中。如果未找到任何选项，或者找到的选项没有选项参数，则<u>OPTARG</u>将被置空。

如果找到了未包含在操作数<u>optstring</u>中的选项字符:<u>name</u>指定的shell变量被设置为问号('?')字符。在这种情况下，如果<u>optstring</u>中的第一个字符是冒号（':'），则将shell变量<u>OPTARG</u>设置为找到的选项字符，但不输出任何错误；否则，shell变量<u>OPTARG</u>将置空，并将诊断消息写入标准错误。应将这种情况视为将提供给调用程序的参数有错误，而不应视为<u>getopts</u>处理中的错误。

如果选项缺少选项参数：

* 如果<u>optstring</u>的第一个字符是冒号，则将由<u>name</u>指定的shell变量设置冒号字符，并将shell变量<u>OPTARG</u>设置为找到的选项字符。

* 否则，应将由<u>name</u>指定的shell变量设置为问号字符并将<u>OPTARG</u>置为空，并向标准错误写入诊断消息。应将这种情况视为将提供给调用程序的参数有错误，而不应视为<u>getopts</u>处理中的错误；诊断消息应按规定编写，但退出状态应为零。


当遇到选项的结尾时，<u>getopts</u>将以大于零的返回值退出。shell变量<u>OPTIND</u>将设置为第一个操作数的索引，如果没有操作数，则为"\$#"+1；<u>name</u>变量设置为问号字符。以下任何一项都应标识选项的结尾：第一个不是选项参数的“--”参数，找到不是选项参数且不以“-”开头的参数，或者遇到错误 。。

Shell变量<u>OPTIND</u>和<u>OPTARG</u>应该是getopts调用者的本地变量，并且默认情况下不会导出。

操作数<u>name</u>指定的Shell变量、<u>OPTIND</u>和<u>OPTARG</u>将影响当前的shell执行环境。请参见第2.12节“Shell执行环境”。

如果应用程序将OPTIND设置为值1，则可以使用一组新的参数：当前位置参数或新的<u>arg</u>值。任何企图在单个Shell执行环境中调用getopts多次，但使用的参数（位置参数或<u>arg</u>操作数）却不完全相同，或将<u>OPTIND</u>值修改为非1，均会产生未指定的结果。

# 4 选项
没有。
# 5 操作数

支持以下操作数：

* <u>optstring</u>——一个字符串，其中包含应用程序调用getopts识别的选项字符。如果一个字符后跟一个冒号，则该选项应具有一个参数，且该参数应作为单独的参数提供。应用程序应将选项字符及其选项参数指定为单独的参数，但是无论是否执行此操作，<u>getopts</u>都会将跟随在要求选项参数的选项字符后的字符解释为选项参数。如果在调用<u>getopts</u>时未将其作为单独的参数提供，则无需识别显式的null选项参数。 （另请参阅POSIX.1-2017的系统接口卷中定义的getopt()函数）。问号和冒号不应被应用程序用作选项字符。使用非字母数字的其他选项字符会产生未指定的结果。如果没有将选项参数作为与选项字符分开的参数提供，则<u>OPTARG</u>中的值应去除选项字符和“-”。<u>optstring</u>中的第一个字符决定了在遇到无法识别的选项字符或缺少选项参数时的getopts行为。

* <u>name</u>——shell变量名，getopts实将找到选项字符保存在该变量中。


默认情况下，<u>getopts</u>解析传递给正在调用的Shell程序的位置参数。如果给出了args，则应解析args而不是位置参数。

# 6 标准输入
不曾用过。

# 7 输入文件
没有。

# 8 环境变量

以下环境变量将影响<u>getopts</u>的执行：

<u>LANG</u>——为未设置或为null的国际化变量提供默认值。 （有关用于确定语言环境类别的国际化变量的优先级，请参阅POSIX.1-2017的基本定义卷第8.2节“国际化变量”。）

<u>LC_ALL</u>——如果设置为非空字符串值，则会覆盖所有其他国际化变量的值。

<u>LC_CTYPE</u>——将文本数据的字节序列解释为字符的语言环境（例如，单字节字符或多字节字符）。

<u>LC_MESSAGES</u>——影响写入到标准错误的诊断消息的格式和内容的语言环境。

<u>NLSPATH</u>——用于处理LC_MESSAGES的消息目录的位置。

<u>OPTIND</u>——getopts设置为要处理的下一个参数的索引。

# 9 异步事件
默认。

# 10 标准输出
不曾用过。

# 11 标准错误

每当检测到错误并且操作数<u>optstring</u>中的第一个字符不是冒号（':'）时，应以未指定格式将诊断消息写入标准错误，并提供以下信息：

* 调用程序名称应在消息中标识。调用程序的名称应为在调用getopts实用程序时的shell特殊参数0（请参见第2.5.2节，特殊参数）的值。等同于`basename "$0"`。
* 如果发现未在<u>optstring</u>中指定的选项，则将识别此错误，并在消息中标识该无效选项字符。

* 如果找到了需要选项参数的选项，但没有找到选项参数，则应识别此错误，并在消息中标识无效选项字符。

# 12 输出文件
没有。

# 13 扩展说明
没有。

# 14 退出状态
应返回以下退出值：

* 0——找到了一个由<u>optstring</u>指定或未指定的选项。

* \>0——遇到选项结尾或发生错误。

## 15 错误的后果
默认。

以下各节提供信息。

## 16 应用程序用法

由于<u>getopts</u>影响当前的shell执行环境，因此通常将其作为shell内置命令提供。如果在子shell或单独的程序执行环境中调用它，例如：
```shell
	(getopts abc value "$@")
	nohup getopts ...
	find . -exec getopts ... \;\;
```
它不会影响调用者环境中的shell变量。

请注意，即使位置参数已更改，shell函数也与调用shell共享<u>OPTIND</u>。如果调用shell及其任何函数使用getopts解析参数，则结果未指定。

# 17 示例
以下示例脚本分析并显示其参数：
```shell
aflag=
bflag=
while getopts ab: name
do
	case $name in
		a)	 aflag=1;;
		b)	 bflag=1
			bval="$OPTARG";;
		?)	printf "Usage: %s: [-a] [-b value] args\n" $0
				exit 2;;
	esac
done
if [ ! -z "$aflag" ]; then
	printf "Option -a specified\n"
fi
if [ ! -z "$bflag" ]; then
	printf 'Option -b "%s" specified\n' "$bval"
fi
shift $(($OPTIND - 1))
printf "Remaining arguments are: %s\n$*"
```
# 18 根本原因

优先选择POSIX <u>getopts</u>，而不是System V <u>getopt</u>。因为 <u>getopts</u>可以处理包含空白字符的选项参数。

“环境变量”部分未提及<u>OPTARG</u>变量，因为它不会影响<u>getopts</u>的执行。它是标准程序使用的少数**仅输出**的变量之一。

不允许将冒号作为选项字符，因为这不是历史行为，并且违反了《实用程序语法准则》。现在指定冒号具有与KornShell版getopts相同的行为。当用作<u>optstring</u>操作数中的第一个字符时，它将禁用有关缺少选项参数和不识别选项字符的诊断。这代替了早期建议中指定的<u>OPTERR</u>变量的使用。

没有完全指定由getopts和getopt()函数生成的诊断消息的格式，因为具有高级（“更友好”）格式的实现会反对某些历史实现所使用的格式。标准开发人员认为在getopts和getopt()之间使用消息中的信息统一是很重要的。可能无法精确复制消息，特别是如果程序是在具有不同getopt()函数的另一个系统上构建的，但是消息中必须包含可以由用户区分的特定信息，如程序名称，无效的选项字符和错误类型。

只有极少数的应用程序会截获<u>getopts</u>标准错误消息，并希望对其进行解析。因此，可以自由实现选择他们想要的最有用的消息。许多历史实现都使用以下格式：

"%s: illegal option -- %c\n", program name, option character

"%s: option requires an argument -- %c\n", program name, option character

具有内置getopt()或getopts版本的历史Shell使用了不同的格式，通常甚至不指示错误中发现的选项字符。

# 19 未来发展方向
       没有。

## 20 另请参阅
2.5.2节，特殊参数。

POSIX.1-2017的基本定义卷，第8章，环境变量，第12.2节，实用程序语法准则。

POSIX.1-2017的系统接口卷，getopt()。

## 21 版权
本文的某些部分以电子形式从IEEE Std 1003.1-2017，信息技术标准-便携式操作系统接口（POSIX），开放组基础规范第7版，2018年版，版权所有（C）2018以电子形式进行复制和复制。电气电子工程师协会和The Open Group。如果此版本与原始IEEE和The Open Group Standard之间存在任何差异，则原始IEEE和The Open Group Standard是裁判文档。原始标准可以在http://www.open group.org/unix/online.html上在线获得。

       在将源文件转换为手册页格式的过程中，最有可能在此页中出现任何印刷或格式错误。要报告此类错误，请参阅https://www.kernel.org/doc/man-pages/reporting_bugs.html。


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《POSIX程序员手册2017》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
