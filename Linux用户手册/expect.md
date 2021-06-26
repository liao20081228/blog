---
title: **Expect**
tags: 普通命令
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Expect manpages》。</font>当前版本为5.45.4。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 名称
Expect——使用交互式程序进行程序化对话，版本5。

# 2 概要

``` bash
Expect [ -dDinN ] [ -c cmds ] [ [ -[f|b] ] cmdfile ] [ args ]
```
# 3 介绍

**Expect**是一个根据脚本来与其他交互式程序“对话”的程序。随着脚本执行，**Expect**知道可以从程序中得到什么以及正确的响应应该是什么。解释型语言提供分支和高级控制结构来引导对话。此外，用户可以根据需要进行控制并直接进行交互，然后将控制权返回给脚本。

**Expectk**是****Expect****和**Tk**的混合物。它的行为就像****Expect****和**Tk**的**wish**。 ****Expect****也可以直接在C或C ++中使用（即不使用Tcl）。参见lib**Expect**(3)。

名称“**Expect**”来自在uucp，kermit和其他调制解调器控制程序中流行的send/**Expect**序列的概念。但是，与uucp不同，****Expect****是通用的，因此可以在考虑任何程序和任务的情况下将其作为用户级命令运行。****Expect****实际上可以同时与多个程序对话。

例如，以下是****Expect****可以做的一些事情：

* 使计算机回拨给您，以便您无需付费即可登录。
* 启动游戏（例如rogue），如果没有出现最佳配置，（一次又一次）重新启动它，直到完成为止，然后将控制权移交给您。
* 运行fsck，并根据预先确定的标准回答问题，回答“yes”，“no”或将控制权还给您。
* 连接到另一个网络或BBS（例如MCI Mail，CompuServe），并自动检索您的邮件，使其看起来就像原本就发送到本地系统一样。
* 在rlogin，telnet，tip，su，chgrp等之间携带环境变量，当前目录或任何类型的信息。

 Shell无法执行这些任务的原因有多种（尝试一下，您会看到的）。使用**Expect**，一切皆有可能。

通常，**Expect**主要用于运行那些需要与用户之间进行交互的程序，前提是可以交互可以字符串化和可以编程话。 如果需要，**Expect**还可以向用户返回控制权（而不会暂停受控制的程序）。 同样，用户可以随时将控制权返回给脚本。

# 4 用法

|Expect&emsp;|Expectk&emsp;&emsp;&emsp;&emsp;|说明|
|:--|:--|:--|
|cmdfile||**Expect**读取cmdfile以获取要执行的命令列表。在支持＃!的系统上，**Expect**也可能被隐式调用。通过将脚本标记为可执行文件，并在脚本的第一行中添加`#!/usr/bin/Expect -f`。<br /><br />当然，路径必须准确地描述**Expect**的安装位置。 /usr/bin只是一个示例。|
|-c cmds|-command cmds|cmds先于cmdfile中命令执行。该命令应加引号，以防止被shell破坏。此选项可以多次使用。通过使用分号";"将多个命令分开，可以使用单个-c执行多个命令。命令按照它们出现的顺序执行。
|-d|-diag|启用一些诊断输出，该诊断输出主要报告命令的内部活动，例如**Expect**和**interact**。此标志与在**Expect**脚本开头的“exp_internal 1”具有相同的作用，并且还会打印**Expect**的版本。（strace命令对跟踪语句很有用，trace命令对跟踪变量赋值很有用。）|
|-D num|-Debug num|标志启用交互式调试器。应该跟一个整数值。如果该值不为零或按下Ctrl+C（或达到断点，或脚本中出现其他合适的调试器命令），则调试器将在下一个Tcl过程之前获得控制权。有关调试器的更多信息，请参见README文件或另请参阅。| 
|-f| -file|为从中读取命令的文件添加序言。该标志本身是可选的，因为它仅在使用#!表示法时才有用（请参见上文），以便可以在命令行上提供其他参数。|
|-b|-buffer|默认情况下，命令文件被读入内存并完整执行。偶尔需要一次读取一行，例如读取标准输入。为了强制以这种方式处理任意文件，请使用-b标志。请注意，仍可以进行stdio缓冲，但是从fifo或stdin读取时这不会引起问题。
|-||如果字符串“-”作为文件名提供，则改为读取标准输入。 （使用“./-”从实际名为“-”的文件中读取。）
|-i|-interactive|使**Expect**以交互方式提示用户输入命令，而不是从文件中读取命令。通过exit命令或EOF终止提示。有关更多信息，请参见解析器（见后文）。如果既不指定命令文件也不使用-c，则假定为-i。|
|--||可以用来分隔选项的结尾。如果您希望将类似选项的参数传递给脚本而不用**Expect**解释，则这很有用。可以将其放在＃!行中，以防止**Expect**进行任何类似标志的解析。例如，`#!/usr/bin/Expect`会将原始参数（包括脚本名称）保留在变量argv中。<br /><br />请注意，在向#!行添加参数时，必须遵守通常的getopt(3)和execve(2)约定。|
|-N,-n|-NORC,-norc|除非使用-N标志，否则文件\$exp_library/Expect.rc（如果存在）会自动引入。 此后，除非使用-n标志，否则将自动引入文件~/.Expect.rc。如果定义了环境变量**DOTDIR**，则将其视为目录，并从该目录中读取.Expect.rc。仅在执行任何-c标志之后才进行这种引入。
|-v|-version|使**Expect**打印其版本号并退出。|
|args||可选的<u>args</u>被构造到一个列表中，并存储在名为<u>argv</u>的变量中。 <u>argc</u>初始化为<u>argv</u>的长度。<br /><br /><u>argv0</u>定义为脚本的名称（如果未使用脚本，则为二进制）。例如`send_user "$argv0 [lrange $argv 0 2]\n"`打印出脚本的名称和前三个参数。|

# 5 命令
**Expect**使用了Tcl（工具命令语言）。 Tcl提供控制流（例如，if,for,break）、表达式求值和其他一些特性，如递归、过程定义等。本文中使用了但额没有定义的命令（例如，set，if，exec）是Tcl命令（请参阅tcl(3)）。 **Expect**支持其他命令，如下所述。除非另有说明，否则命令将返回空字符串。

命令按字母顺序列出，以便可以快速找到它们。但是，新用户可能会发现通过按**spawn**、**send**、**Expect**和**interact**会更容易开始。

请注意，在“Exploring Expect”一书中提供了对语言（**Expect**和**Tcl**）的最佳介绍（请参阅下面的“另请参阅”）。此手册页包含了一些示例，但由于本手册页主要是作为参考资料，因此它们的使用范围非常有限。

请注意，在此手册页的文本中，带有大写字母“E”的“**Expect**”指的是**Expect**程序，而带有小写字母“e”的“**expect**”指的是**Expect**程序中的**expect**命令。

## 5.1 close
``` bash
	close [-slave] [-onexec 0|1] [-i spawn_id]
```
关闭与当前进程的连接。大多数交互式程序在其stdin上检测EOF，然后退出；因此，close通常也足以终止该进程。 -i标志声明了对应spawn_id的进程，这个进程也将关闭。

当当前进程退出并隐式关闭时，**expect**和**interact**会检测到。但是，如果通过“exec kill \$pid”杀死进程，则需要显式调用close。

**-onexec**标志确定是在新生成的进程中关闭spawn ID还是覆盖该进程。要使spawn ID保持打开状态，请使用值0。非0整数值将在所有新进程中强制关闭spawn（默认值）。

**-slave**标志关闭与生成ID关联的从站。 （请参阅“spawn -pty”。）当连接关闭时，从站也会自动关闭（如果仍然打开）。

无论是隐式还是显式关闭连接，都应调用**wait**清除相应的内核进程插槽。 **close**不调用**wait**，因为不能保证关闭进程连接将导致close退出。请参阅下面的**wait**以获取更多信息。

## 5.2 debug
``` bash
	debug [[-now] 0|1]
```
控制Tcl调试器，使您可以单步执行语句，设置断点等。

不带参数的情况下，如果调试器未运行，则返回1，否则返回0。

参数为1时，将启动调试器。如果参数为0，则调试器将停止。如果1参数前面带有-now标志，则调试器将立即启动（即，在debug命令本身中）。否则，调试器将随下一个Tcl语句启动。

debug命令不会更改任何trap。将其与以-D标志开头的**expect**（请参见上文）进行比较。

有关调试器的更多信息，请参见README文件或另请参阅（以下）。

## 5.3 disconnect
断开分支（forked）进程与终端的连接。它继续在后台运行。该进程将被还给他自己的进程组（如果可能）。标准I/O重定向到/dev/null。

以下片段使用**disconnect**接在后台继续运行脚本。

``` bash
	if {[fork]!=0} exit
		disconnect
		. . .
```

下面的脚本读取一个密码，然后每小时运行一个程序，每次运行该程序都需要输入密码。该脚本提供了密码，因此您只需键入一次即可。 （请参阅stty命令，该命令演示了如何关闭密码回显。）

``` c
	send_user "password?\ "
	expect_user -re "(.*)\n"
	for {} 1 {} {
		if {[fork]!=0} {sleep 3600;continue}
		disconnect
		spawn priv_prog
		expect Password:
		send "$expect_out(1,string)\r"
		. . .
		exit
	}

```
与shell异步进程特性(＆)相比，使用**disconnect**的优点是**Expect**可以在断开连接之前保存终端参数，然后将其应用于新的ptys。使用＆，**Expect**没有机会读取终端的参数，因为在**Expect**收到控制权之前，终端已经断开连接。
## 5.4 exit

``` bash
	exit [-opts] [status]
```
导致**Expect**退出或准备退出。

**-onexit**标志使下一个参数用作退出处理程序。不带参数的情况下，将返回当前的退出处理程序。

**-noexit**标志使Expect准备退出，停止将控制权真正返回给操作系统。用户定义的退出处理程序以及Expect自己的内部处理程序都将运行。不再执行其他Expect命令。如果您在运行带有其他Tcl扩展的Expect，这将很有用。保留当前的解释器（如果在Tk环境中，则保留主窗口），以便其他Tcl扩展可以clean up。如果再次调用Expect的exit（但是可能会发生这种情况），则不会重新运行处理程序。

退出后，与派生进程的所有连接都将关闭。生成的进程会将关闭视为EOF。除了正常的_exit(2)过程，exit不会执行其他任何操作。因此，不检查EOF的派生进程可能会继续运行。 （例如，确定发送信号到到一个派生进程的条件是很重要的，但这些条件取决于系统，通常记录在exit(3)下。）继续运行的spawn进程将被init继承。

返回<u>status</u>（如果未指定，则为0）作为Expect的退出状态。如果到达脚本末尾，则隐式执行**exit**。

## 5.5 exp_continue

``` bash
	exp_continue [-continue_timer]
```
命令**exp_continue**允许**expect**自身继续执行，而不是像往常那样返回。默认情况下，**exp_continue**重置超时计时器。 **-continue_timer**标志可防止重新启动计时器。 （请参阅**expect**以获取更多信息。）
## 5.6 exp_internal

``` bash
	exp_internal [-f file] value
```
如果<u>value</u>非零，则导致其他命令将**Expect**内部的诊断信息发送到stderr。如果<u>value</u>为0，则禁用输出。诊断信息包括接收到的每个字符，以及将当前输出与模式进行匹配的所有尝试。

如果提供了可选file，则所有普通输出和调试输出都将写入该文件（与value的值无关）。任何先前的诊断输出文件都已关闭。

**-info**标志使exp_internal返回给定的最新non-info参数的描述。

## 5.7 exp_open

``` bash
	exp_open [args] [-i spawn_id]
```
返回与原始spawn ID对应的Tcl文件标识符。 然后可以使用文件标识符，就好像它是由Tcl的open命令打开的一样。 （不应再使用spawn ID。不应执行**wait**）。

**-leaveopen**标志使spawn ID保持打开状态，以便通过Expect命令进行访问。 必须在spawn ID上执行**wait**。

## 5.8 exp_pid

``` bash
	exp_pid [-i spawn_id]
```
返回与当前派生进程相对应的进程ID。 如果使用-i标志，则返回给定spawn ID的pid。

## 5.9 exp_send
**send**的别名。
## 5.10 exp_send_error
**send_error**的别名。
## 5.11 exp_send_log
**send_log**的别名。
## 5.12 exp_send_tty
**send_tty**的别名。
## 5.13 exp_send_user
**send_user**的别名。

## 5.14 exp_version
``` bash
exp_version [[-exit] version]
```

对于确保脚本与Expect的当前版本兼容很有用。

不带任何参数的情况下，返回Expect的当前版本。然后可以将此版本编码在您的脚本中。如果您知道自己没有使用最新版本的特性，则可以指定一个较早的版本。

版本由点号分隔的三个数字组成。首先是主版本号。为主版本号与当前版本不同的**Expect**编写的脚本几乎肯定无法工作。如果主数版本号不匹配，则exp_version返回错误。

第二个是次版本号。为次版本号比当前版本更大的**Expect**编写的脚本可能依赖某些新特性，且可能无法运行。如果脚本的主版本号匹配，但次版本号更大，则exp_version返回一个错误。

第三个是在版本比较中不起作用的数字。但是，以任何方式（例如通过附加文档或优化）更改Expect软件发行版时，该值都会增加。每个新的次版本将其重置为0。

**-exit**标志使得版本过时，Expect打印错误并退出。

## 5.15 expect

``` bash
	expect [[-opts] pat1 body1] ... [-opts] patn [bodyn]
```

一直等待直到其中一个模式与派生进程的输出匹配、经过了指定的时间段或看到文件结束为止。如果最后的body为空，则可以省略。

来自最近**expect_before**命令的模式将在任何其他模式之前隐式使用。来自最近**expect_after**命令的模式将在任何其他模式之后隐式使用。

如果整个**expect**语句的参数需要多行，则所有参数都可以用花括号“括起来”为一行，从而避免以反斜杠续行。在这种情况下，除了括号，仍会发生普通的Tcl替换。

如果模式是关键字**eof**，则在文件结束时执行相应的body。如果模式是关键字**timeout**，则在超时时执行相应的body。如果不使用timeout关键字，则在超时时执行隐式的null动作。默认超时时间为10秒，但可以通过命令“set timeout 30”将其设置为30秒，-1表示永不超时。如果模式是关键字**default**，则在超时或文件结束时执行相应的body。

如果一个模式匹配了，则执行相应的body。 **expect**返回body的执行结果（如果没有匹配的模式，则返回空字符串）。如果有多个模式匹配，则执行首先出现的模式的body。

每次新输出到达时，都会按照列出的顺序将其与每个模式进行比较。因此，您可以通过确保最后一个模式（如提示）的出现来测试是否缺少匹配项，例如提示。在没有提示的情况下，您必须使用**timeout**（就像您手动进行交互一样）。

模式以三种方式指定。默认情况下，使用Tcl的**字符串匹配**命令指定模式。 （这种模式也类似于通常称为“glob”模式的C shell正则表达式）。 **-gl**标志可用于保护可能与**Expect**标志匹配的模式。任何以“-”开头的模式都应采用这种方式进行保护。 （所有以“-”开头的字符串都保留用于将来的选项。）

例如，以下片段查找一个成功的登录。 （请注意，推测**abort**是脚本中其他地方定义的过程。）

``` bash
	expect {
		busy               {puts busy\n ; exp_continue}
		failed             abort
		"invalid password" abort
		timeout            abort
		connected
	}

```
第三个模式必须用引号引起来，因为它包含一个空格，否则该空格会将模式与动作分开。 具有相同动作的模式（例如第三和第四）需要再次列出动作。 可以通过使用正则表达式样式的模式来避免这种情况（请参阅下文）。 在Tcl手册中可以找到有关glob-style模式的更多信息。

Regexp风格的模式遵循Tcl的**regexp**（“正则表达式”的缩写）命令定义的语法。 **-re**表示要使用正则表达式模式。 可以使用regexp将前面的示例重写为：

``` bash
	expect {
		busy       {puts busy\n ; exp_continue}
		-re "failed|invalid password" abort
		timeout    abort
		connected
	}
```
两种类型的模式都是“未锚定的”。这意味着模式不必匹配整个字符串，可以在字符串中的任何位置开始和结束匹配（只要其他所有内容都匹配）。使用\^来匹配字符串的开头，使用\$来匹配字符串的结尾。请注意，如果您不等待字符串的结尾，则当字符串是从派生进程中回显时，响应很容易在字符串的中间结束。虽然仍产生正确结果，输出看起来可能不自然。因此，如果可以准确地描述字符串末尾的字符，则鼓励使用\$。

请注意，在许多编辑器中，\^和\$分别匹配行的开头和结尾。但是，由于**Expect**不是面向行的，因此这些字符与当前在**Expect**匹配缓冲区中的数据的开头和结尾（与行相对）匹配。 （另请参见下面有关“系system indigestion”的注释。）

**-ex**标志使模式匹配为“精确”字符串。不对\*，\^等进行解释（尽管仍然必须遵守常规Tcl约定）。精确模式总是不锚定的。

**-nocase**标志使模式匹配忽略大小写。模式不受影响。

在读取输出时，超过2000个字节会强制“忘记”较早的字节。这可以通过函数**match_max**进行更改。 （请注意，太大的值可能会减慢模式匹配的速度。）如果为**full_buffer**，则如果已接收到<u>match_max</u>字节且没有匹配其他模式时，将执行相应的body。不管是否使用**full_buffer**关键字，都会将“忘掉”的字符写到expect_out(buffer)中。

如果<u>patlist</u>是关键字**null**，并且允许使用nulls（通过**remove_nulls**命令），则匹配单个ASCII 0时则将执行相应的body。无法通过glob或regexp模式匹配0字节。

匹配模式（或eof或full_buffer）后，所有匹配的和以前不匹配的输出都将保存在变量<u>expect_out(buffer)</u>中。最多9个正则表达式子字串匹配项保存在<u>expect_out(1，string)</u>到<u>expect_out(9，string)</u>中。如果在一个模式前使用 **-indices**标志，则这10个字符串的开始和结束索引（以适合**lrange**的形式）存储在变量<u>expect_out(X，start)</u>和<u>expect_out(X，end)</u>中，其中X为数字，对应缓冲区中子串的位置。 0表示与整个模式匹配的字符串，由glob模式和regexp模式产生。例如，如果某个进程的输出为“abcdefgh\\n”，则：

``` bash
	expect "cd"
```
的结果就像执行了以下语句：

``` bash
	set expect_out(0,string) cd
	set expect_out(buffer) abcd
```
而"efgh\\n"则留在输出缓冲区中。 如果某个进程产生输出“abbbcabkkkka\\n”，则：

``` bash
	expect -indices -re "b(b*).*(k+)"
```
的结果就想执行了下列语句：

``` bash
	set expect_out(0,start) 1
	set expect_out(0,end) 10
	set expect_out(0,string) bbbcabkkkk
	set expect_out(1,start) 2
	set expect_out(1,end) 3
	set expect_out(1,string) bb
	set expect_out(2,start) 10
	set expect_out(2,end) 10
	set expect_out(2,string) k
	set expect_out(buffer) abbbcabkkkk
```
而“a\\n”则留在输出缓冲区中。 模式“\*”（和-re “.\*”）将冲刷输出缓冲区，而不从进程中读取更多输出。

通常，匹配的输出会从Expect的内部缓冲区中丢弃。 可以通过在模式前面加上 **-notransfer**标志来防止这种情况。 该标志在实验中特别有用（为方便起见，在实验中可以缩写为“-not”）。

与匹配的输出（或eof或full_buffer）关联的spawn ID被存储在<u>expect_out(spawn_id)</u>中。

**-timeout**标志使当前的**expect**命令使用跟随的值作为超时，而不是使用**timeout**变量的值。

默认情况下，模式与当前进程的输出匹配，但是 **-i**标志声明后面跟随的任何模式与给出的spawn_id列表中进程的输出匹配（直到下一个 **-i**）。 spawn_id列表应该是空格分隔的spawn_ids列表，或者是引用此类spawn_ids列表的变量。

例如，以下示例等待当前进程的“conneced”，或者等待\$proc2指定的spawn_id的“busy”，“fail”或“invalid password”。

``` bash
	expect {
		-i $proc2 busy {puts busy\n ; exp_continue}
		-re "failed|invalid password" abort
		timeout abort
		connected
	}
```
全局变量<u>any_spawn_id</u>的值可用于将模式与当前**expect**命令中所有其他  **-i**标志列出的任何spawn_id进行匹配。 **-i**标志中没有关联模式的spawn_id（即另一个“-i”紧随其后）可用于 与<u>any_spawn_id</u>关联的同一<u>expect</u>命令中的任何其他模式。

**-i**标志还可以命名全局变量，在这种情况下，这个全局变量用于读取spawn ID列表。 变量每次变化时都会重新读取。 这提供了一种在命令执行期间更改I/O源的方法。 通过这种方式提供的spawn ID被称为“间接”spawn ID。 

**break**和**continue**之类的动作会导致控制结构（即，for，proc）以通常的方式运行。 命令exp_continue允许expect自己继续执行，而不是像往常那样返回。 

这对于避免显式循环或重复的expect语句很有用。以下示例是自动化rlogin片段的一部分。如果rlogin提示输入密码，则**exp_continue**避免编写第二个Expect语句（再次查找提示）。

``` bash
	expect {
		Password: {
			stty -echo
			send_user "password (for $user) on $host: "
			expect_user -re "(.*)\n"
			send_user "\n"
			send "$expect_out(1,string)\r"
			stty echo
			exp_continue
		} incorrect {
			send_user "invalid password or account\n"
			exit
		} timeout {
			send_user "connection to $host timed out\n"
			exit
		} eof {
			send_user \
				"connection to host failed: $expect_out(buffer)"
			exit
		} -re $prompt
	}

```
例如，以下片段可能帮助用户指导已经完全自动化的交互。 在这种情况下，终端将进入原始模式。 如果用户按下“+”，则变量增加。 如果按下“p”，则会向该过程发送多个返回消息，也许以某种方式对其进行poke，而“i”使用户可以与该进程进行交互，从而有效地从脚本中窃取控制权。 在每种情况下，**exp_continue**允许当前**expect**在执行当前操作后继续模式匹配。

``` bash
	stty raw -echo
	expect_after {
		-i $user_spawn_id
		"p" {send "\r\r\r"; exp_continue}
		"+" {incr foo; exp_continue}
		"i" {interact; exp_continue}
		"quit" exit
	}
```
默认情况下，**exp_continue**重置超时计时器。 如果使用 **-continue_timer**标志调用**exp_continue**，则计时器不会重新启动。

## 5.3 expect_tty

``` bash
	expect_tty [expect_args]
```
就像**expect**一样，但是它从/dev/tty中读取字符（即，用户击键）。 默认情况下，读取是在cooked模式下进行的。 因此，行必须以回车结尾以便expect看到它们。 这可以通过**stt**y更改（请参见下面的**stty**命令）。
## 5.3

``` bash
expect_user [expect_args]
```
就像**expect**一样，但是它从stdin读取字符（即，用户的击键）。 默认情况下，读取是在cooked模式下进行的。 因此，行必须以回车结尾才能期望看到它们。 这可以通过**stty**更改（请参见下面的**stty**命令）。

## 5.3 log_user

``` bash
	log_user -info|0|1
```
默认情况下，send/expect对话框记录到stdout（如果一个日志文件被打开，则记录到该日志文件）。 通过命令“log_user 0”禁用到stdout的日志记录，并通过“log_user 1”重新启用。 到日志文件的记录不变。

**-info**标志使log_user返回给定的最新no-info参数的描述。
## 5.3 send

``` bash
	send [-flags] string
```
发送字符串到当前进程。例如，命令

``` bash
send "hello world\r"
```
发送字符`hello空格world回车`到当前进程。 （Tcl引入了类似printf的命令（称为**format**），可以构建任意复杂的字符串。）

尽管具有行缓冲输入的程序在发送回车字符之前不会读取字符，但仍会立即发送字符。回车字符表示为“\r”。
--标志强制将下一个参数解释为字符串而不是标志。任何字符串都可以以“--”开头，无论它实际上是否看起来像一个标志。这提供了一种可靠的机制来指定可变字符串，而不会被那些偶然看起来像标志的字符串妨碍。 （所有以“-”开头的字符串都保留用于将来的选项。）

**-i**标志声明将字符串发送到指定的spawn_id。如果spawn_id为<u>user_spawn_id</u>，并且终端处于原始模式，则将字符串中的换行符转换为回车换行符，以便它们看起来像终端处于cooked模式。 **-raw**标志禁用此转换。

**-null**标志发送空字符（0字节）。默认情况下，发送一个空值。可以在**-null**后面跟随一个整数，以指示要发送多少个null。

**-break**标志生成一个中断条件。仅当spawn id引用了通过“spawn -open”打开的tty设备时，才有意义。如果生成了诸如tip之类的进程，则应使用tip的约定来生成中断。



**-s**标志强制输出“缓慢地”发送，因此避免了通常情况，即计算机键出（outtype）一个为永远不会键出（outtype）相同buffer的人类设计的输入buffer。该输出由变量“send_slow”的值控制，该变量带有两个元素列表。第一个元素是整数，它描述要原子发送的字节数。第二个元素是一个实数，它描述原子发送必须间隔的秒数。例如，“set send_slow {10 .001}”将强制“send -s”每发送10个字符之间间隔1毫秒的方式发送字符串。

 **-h**标志强制输出（以某种方式）像人类实际键入一样发送。类似人一样的延迟会出现字符之间。（该算法基于Weibull分布，并进行了修改以适合该特定应用。）此输出由变量“send_human”的值控制，该变量具有五个元素。前两个元素是字符的平均到达时间，以秒为单位。默认情况下使用第一个。第二个用于单词结尾，以模拟这种过渡时偶尔发生的微妙的停顿。第三个参数是可变性的度量，其中.1十分可变，1是合理可变，而10十分不可变。极限是0到无穷大。最后两个参数分别是最小和最大到达间隔时间。最小值和最大值被用于维持(last)和剪断(clip)最终时间。如果最小值和最大值足够大，则最终平均值可能与给定平均值有很大不同。

例如，以下命令模拟快速一致的打字员：

``` bash
	set send_human {.1 .3 1 .05 2}
	send -h "I'm hungry.  Let's do lunch."

```

而宿醉后，以下各项可能更适合：

``` bash
	set send_human {.4 .4 .2 .5 100}
	send -h "Goodd party lash night!"
```
请注意，虽然您可以通过将错误和更正嵌入到send参数中来自行设置错误更正情况，但不会模拟错误。

用于发送空字符，用于发送中断，用于强制缓慢输出和用于人类风格输出的标志是互斥的。仅使用指定的最后一个。用于发送空字符或中断的标志不能指定字符串参数。

在第一次发送到进程之前，最好有一个**expect**。 **expect**将等待进程启动，而**send**不会。 特别是，如果第一次发送在进程开始运行之前完成，则可能会导致数据被忽略。 在交互式程序不提供初始提示的情况下，您可以在发送之前先进行延迟，如下所示：

``` bash
	# To avoid giving hackers hints on how to break in,
	# this system does not prompt for an external password.
	# Wait for 5 seconds for exec to complete
	spawn telnet very.secure.gov
	sleep 5
	send password\r
```
**exp_send**是发送的别名。 如果在Tk环境中使用**Expectk** 或**Expect**的某些其他变体，则send由Tk定义，目的完全不同。 提供**exp_send**是为了实现环境之间的兼容性。 为其他Expect的其他发送命令提供了类似的别名。

## 5.3

## 5.3
## 5.3
## 5.3
## 5.3
## 5.3
## 5.3
## 5.3
## 5.3
## 5.3


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《**Expect** manpages》。</font>当前版本为5.45.4。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
