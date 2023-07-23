---
title: expect
tags: man1
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Expect manpages》，</font>expect当前版本为5.45.4，手册更新日期为1994-12-29。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 名称
Expect——与交互式程序进行程序化对话，版本5。

# 总览

``` bash
expect [ -dDinN ] [ -c cmds ] [ [ -[f|b] ] cmdfile ] [ args ]
```
# 介绍校对完

**Expect**是一个根据脚本来与其他交互式程序“对话”的程序。随着脚本执行，**Expect**知道可以从程序中得到什么以及正确的响应应该是什么。解释型语言提供了分支和高级控制结构来指导对话。此外，用户可以根据需要取得控制权并直接与程序交互，然后将控制权返回给脚本。

**Expectk**是**Expect**和**Tk**的混合物。它的行为就像**Expect**和**Tk**所期望的那样。 **Expect**也可以直接在C或C\+\+中使用（即不使用Tcl）。参见libexpect(3)。

名称“**Expect**”来自在uucp，kermit和其他调制解调器控制程序中流行的<u>send/expect</u>序列的概念。但是，与uucp不同，**Expect**是通用的，因此可以作为用户级命令与任何程序和任务一起运行。**Expect**实际上可以同时与多个程序对话。

例如，以下是**Expect**可以做的一些事情：

* 使计算机回拨给您，以便您无需付费即可登录。
* 启动游戏（例如rogue），如果没有出现最佳配置，（一次又一次）重新启动它，直到出现最佳配置为止，然后将控制权移交给您。
* 运行fsck，并根据预先确定的规则回答问题，回答“yes”、“no”或将控制权还给您。
* 连接到另一个网络或BBS（例如MCI Mail，CompuServe），并自动取回您的邮件，使其看起来就像原本就发送到本地系统一样。
* 在rlogin，telnet，tip，su，chgrp等之间携带环境变量，当前目录或任意种类的信息。

Shell无法执行这些任务的原因有多种（尝试一下，您会看到的）。使用**Expect**，一切皆有可能。

通常，**Expect**主要用于运行那些必须与用户进行交互的程序，前提是交互可以字符串化和编程化。 如果需要，**Expect**也可以向用户返回控制权（而不会停止正在被控制的程序）。 同样，用户可以随时将控制权返回给脚本。

# 用法校对完

| Expect&emsp; | Expectk&emsp;&emsp;&emsp;&emsp; | 说明                                                                                                                                                                                                                                                                                                     |
| :----------- | :------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| cmdfile | | **Expect**读取<u>cmdfile</u>以获取要执行的命令列表。通过将脚本标记为可执行文件，并在脚本的第一行中添加<br/>&emsp;&emsp;`#!/usr/bin/Expect -f`。<br/>在支持 `＃!` 的系统上，**Expect**也可以被隐式调用，当然，路径必须准确地描述**Expect**的安装位置。 `/usr/bin`只是一个示例。 |
| -c cmds  | -command cmds| 在执行脚本中的命令前先执行<u>cmds</u>。该命令应加引号，以防止被shell破坏。此选项可以多次使用。可以使用单个 **-c** 执行多个命令，多个命令之间用分号";"分开。命令按照它们出现的顺序被执行。|                                                                                                                |
| -d | -diag  | 启用一些诊断输出，该诊断输出主要报告命令的内部活动，例如**expect**和**interact**。此标志与在**Expect**脚本开头的“exp_internal 1”具有相同的作用，并且还会打印**Expect**的版本。（**[strace](#strace)** 命令用于跟踪语句，**trace**命令用于跟踪变量赋值。）|                                              |
| -D num  | -Debug num | <span id="Debug">启用交互式调试器</span>。应该跟一个整数值。如果该值不为零或按下Ctrl+C（或达到断点，或脚本中出现其他合适的调试器命令），则调试器将在下一个Tcl过程之前获得控制权。有关调试器的更多信息，请参见README文件或[另请参阅](#SeeAlso)。 |
| -f   | -file    | 在执行脚本先执行<u>file</u>。该标志本身是可选的，因为它仅在使用`#!`注释时才有用（请参见上文），以便可以在命令行上提供其他参数。 |
| -b | -buffer   | 默认情况下，命令文件被读入内存并完整执行。有时希望一次读取一行文件。 例如，stdin 就是这样读取的。 为了强制以这种方式处理任意文件，请使用 **-b** 标志。 请注意，标准IO缓存可能仍会发生，但从 fifo 或 stdin 读取时不会引起问题。|                                                                           |
| - |  | 如果字符串“-”作为文件名提供，则改为读取标准输入。 （“./-”从实际名为“-”的文件中读取。）  |
| -i   | -interactive | **Expect**以交互方式提示用户输入命令，而不是从文件中读取命令。通过**exit**命令或EOF终止提示。有关更多信息，请参见[解释器](#interpreter)（见后文）。如果既不指定命令文件也不使用 **-c**，则假定为 **-i**。 |
| --       | | 可以用来分隔选项的结尾。主要用于将类似选项的参数传递给脚本而不要**Expect**进行解释。可以将其放在`＃!`行中，以防止**Expect**进行任何类似标志的解析。例如，`#!/usr/bin/expect`会将原始参数（包括脚本名称）保留在变量argv中。<br/><br/>请注意，在向`#!`行添加参数时，必须遵守getopt(3)和execve(2)通常约定。 |
| -N,-n | -NORC,-norc | 除非使用-N标志，否则文件`$exp_library/expect.rc`（如果存在）会自动引入。 此后，除非使用-n标志，否则将自动引入文件`~/.expect.rc`。如果定义了环境变量**DOTDIR**，则将其视为目录，并从该目录中读取`.expect.rc`。仅在执行-**c**标志之后才进行这种引入。 |
| -v           | -version                        | 使**Expect**打印其版本号并退出。 |
| args         | | 可选的<u>args</u>被构造到一个列表中，并存储在名为<u>argv</u>的变量中。 <u>argc</u>初始化为<u>argv</u>的长度。<br/><br/><u>argv0</u>定义为脚本的名称（如果未使用脚本，则为二进制程序名）。例如`send_user "$argv0 [lrange $argv 0 2]\n"`打印出脚本的名称和前三个参数。           |

# 命令 校对完
**Expect**使用了Tcl（工具命令语言）。 Tcl提供控制流（例如，if、for、break）、表达式求值和其他一些特性，如递归、过程定义等。本文中使用了但没有定义的命令（例如，set，if，exec）是Tcl命令（请参阅tcl(3)）。 **Expect**支持的其他命令如下所述。除非另有说明，否则命令将返回空字符串。

命令按字母顺序列出，以便可以快速找到它们。但是，新用户可能会发现按 **[spawn](#spawn)**、**[send](#send)**、**[expect](#expect)** 和 **[interact](#interact)** 的顺序会更容易开始。

请注意，在《探索Expect》一书中提供了对语言（**Expect**和**Tcl**）的最佳介绍（请参阅下面的[另请参阅](#SeeAlso)）。此手册页包含了一些示例，但由于本手册页主要是作为参考资料，因此它们是非常有限。

请注意，在此手册页的文本中，带有大写字母“E”的“**Expect**”指的是**Expect**程序，而带有小写字母“e”的“**expect**”指的是**Expect**程序中的**expect**命令。


|命令|命令|命令|命令|命令|
|:--|:--|:--|:--|:--|
|[close](#close)|[debug](#debug)|[disconnect](#disconnect)|[exit](#exit)|[exp_continue](#exp_continue)|
|[exp_internal](#exp_internal)|[exp_open](#exp_open)|[exp_pid](#exp_pid)|[exp_send](#exp_send)|[exp_send_error](#exp_send_error)|
|[exp_send_log](#exp_send_log)|[exp_send_tty](#exp_send_tty)|[exp_send_user](#exp_send_user)|[exp_version](#exp_version)|[expect](#expect)|
|[expect_after](#expect_after)|[expect_background](#expect_background)|[expect_before](#expect_before)|[expect_tty](#expect_tty)|[expect_user](#expect_user)|
|[fork](#fork)|[interact](#interact)|[interpreter](#interpreter)|[log_file](#log_file)|[log_user](#log_user)|
|[match_max](#match_max)|[overlay](#overlay)|[parity](#parity)|[remove_nulls](#remove_nulls)|[send](#send)|
|[send_error](#send_error)|[send_log](#send_log)|[send_tty](#send_tty)|[send_user](#send_user)|[sleep](#sleep)|
|[spawn](#spawn)|[strace](#strace)|[stty](#stty)|[system](#system)|[timestamp](#timestamp) |
|[trap](#trap)|[wait](#wait)|

## close校对完
```tcl
close [-slave] [-onexec 0|1] [-i spawn_id]
```
<span id="close">关闭</span>与当前进程的连接。大多数交互式程序会在其标准输入stdin中检测 EOF 并退出； 因此 **close** 通常也足以终止进程。 **-i** 标志声明了要关闭的进程，该进程与指定的spawn_id对应。

当前进程退出时，**expect** 和 **interact** 都会检测到，并隐式执行**close**。 但是，如果您通过`exec kill $pid`来终止进程，则需要显式调用 **close**。

**-onexec**标志确定在任何新派生进程中或该进程被覆盖时是否关闭<u>spawn id</u>。要使<u>spawn id</u>保持打开状态，请使用值0。非0整数值将在所有新派生进程中强制关闭<u>spawn id</u>（默认值）。

**-slave** 标志关闭与<u>spawn id</u> 关联的从属。 （参见“[spawn -pty](#spawn_pty)”。）当连接关闭时，如果slave仍然打开，它也会自动关闭。

无论是隐式还是显式关闭连接，都应调用**wait**清除相应的内核进程槽。 **close**不会调用**wait**，因为不能保证关闭进程连接将导致这个进程退出。请参阅下面的 **[wait](#wait)** 以获取更多信息。

## debug校对完
``` tcl
debug [[-now] 0|1]
```
<span id="debug">控制</span>Tcl调试器，使您可以单步执行语句，设置断点等。

不带参数的情况下，如果调试器未运行，则返回1，否则返回0。

参数为1时，将启动调试器。如果参数为<u>0</u>，则调试器将停止。如果1参数前面带有 **-now** 标志，则调试器将立即启动（即，在**debug**命令本身中）。否则，调试器将随下一个Tcl语句启动。

**debug**命令不会更改任何[trap](#trap)。将其与用 **-D** 启动的**Expect**（请[参见上文](#Debug)）进行比较。

有关调试器的更多信息，请参见README文件或[另请参阅](#SeeAlso)。

## disconnect校对完
<span id="disconnect">断开</span>fork进程与终端的连接。fork进程继续在后台运行。该fork进程将被还给他自己的进程组（如果可能）。标准I/O重定向到`/dev/null`。

以下片段使用**disconnect**在后台继续运行脚本。

``` tcl
if {[fork]!=0} exit
disconnect
. . .
```

下面的脚本读取一个密码，然后每小时运行一个程序，每次运行该程序都需要输入密码。该脚本提供了密码，因此您只需键入一次即可。 （请参阅[stty](#stty)命令，该命令演示了如何关闭密码回显。）

``` tcl
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

## exit校对完

``` tcl
exit [-opts] [status]
```
导致**Expect**退出或准备<span id="exit">退出</span>。

**-onexit**标志使下一个参数用作退出处理程序。不带参数的情况下，将返回当前的退出处理程序。

**-noexit**标志使**Expect**准备退出，但没有将控制权真正返回给操作系统。用户定义的退出处理程序以及**Expect**自己的内部处理程序都将运行。不再执行其他**Expect**命令。如果您正在运行 **Expect** 和其他 Tcl 扩展，这非常有用。当前解释器（和主窗口，如果在Tk环境中）将保留以便其他Tcl扩展可以清理。如果再次调用**Expect**的**exit**（可能会发生这种情况），则不会重新运行处理程序。

退出后，与派生进程的所有连接都将关闭。派生进程会将关闭当作EOF。除了正常的_exit(2)过程，**exit**不会执行其他任何操作。因此，不检查EOF的派生进程可能会继续运行。 （例如，各种条件对于确定发送什么信号到一个派生进程是很重要的，但这些条件取决于系统，通常记录在exit(3)下。）继续运行的派生进程将被init继承。

返回<u>status</u>（如果未指定，则为0）作为**Expect**的退出状态。如果到达脚本末尾，则隐式执行**exit**。

## exp_continue校对完

```tcl
exp_continue [-continue_timer]
```
命令 **<span id="exp_continue">exp_continue</span>** 允许 **expect**自身继续执行，而不是像往常那样返回。默认情况下，**exp_continue**重置超时计时器。 <u>-continue_timer</u>标志可防止重新启动计时器。 （请参阅 **[expect](#expect)** 以获取更多信息。）
## exp_internal校对完

``` tcl
exp_internal [-f file] value
```
如果<u>value</u>非零，则<span id="exp_internal">导致</span>其他命令将**Expect**内部的诊断信息发送到stderr。如果<u>value</u>为0，则禁用输出。诊断信息包括接收到的每个字符，以及将当前输出与模式进行匹配的所有尝试。

如果提供了可选<u>file</u>，则所有普通输出和调试输出都将写入该文件（与value的值无关）。任何先前的诊断输出文件都将被关闭。

**-info**标志使**exp_internal**返回给定的最新non-info参数的描述。

## exp_open校对完

``` tcl
exp_open [args] [-i spawn_id]
```
返回与原始spawn id对应的Tcl文件标识符。 然后可以使用文件标识符，就好像它是由Tcl的 **<span id="exp_open">open</span>** 命令打开的一样。 （spawn id不应再使用）。不应执行**wait**。

**-leaveopen**标志使spawn id保持打开状态，以便通过Expect命令进行访问。 必须在spawn id上执行**wait**。

## exp_pid校对完

``` tcl
exp_pid [-i spawn_id]
```
返回与当前派生进程相对应的进程<span id="exp_id">id</span>。 如果使用 **-i** 标志，则返回给定<u>spawn id</u>的pid。

## exp_send校对完
**<span id="exp_send">send</span>** 的别名。
## exp_send_error校对完
**<span id="exp_send_error">send_error</span>** 的别名。
## exp_send_log校对完
**<span id="exp_send_log">send_log</span>** 的别名。
## exp_send_tty校对完
**<span id="exp_send_tty">send_tty</span>** 的别名。
## exp_send_user校对完
**<span id="exp_send_user">send_user</span>** 的别名。

## exp_version校对完
``` tcl
exp_version [[-exit] version]
```

主要用于确保脚本与**Expect**的当前<span id="exp_version">版本</span>兼容。

不带任何参数的情况下，返回**Expect**的当前版本。然后可以将此版本编码在您的脚本中。如果您知道自己没有使用最新版本的特性，则可以指定一个较早的版本。

版本由点号分隔的三个数字组成。首先是主版本号。为主版本号与当前版本不同的**Expect**编写的脚本几乎肯定无法工作。如果主数版本号不匹配，则**exp_version**返回错误。

第二个是次版本号。为次版本号比当前版本更大的**Expect**编写的脚本可能依赖某些新特性，且可能无法运行。如果脚本的主版本号匹配，但次版本号比当前运行的**Expect**更大，则**exp_version**返回一个错误。

第三个是在版本比较中不起作用的数字。但是，以任何方式（例如通过附加文档或优化）更改**Expect**软件发行版时，该值都会增加。每个新的次版本将其重置为0。

如果版本过时，**-exit** 使得**Expect**打印错误并退出。

## expect校对完

``` tcl
expect [[-opts] pat1 body1] ... [-opts] patn [bodyn]
```

<span id="expect">一直</span>等待直到其中一个模式与派生进程的输出匹配、经过了指定的时间段或看到文件结束为止。如果最后的<u>body</u>为空，则可以省略。

来自最近**expect_before**命令的模式将在任何其他模式之前隐式使用。来自最近**expect_after**命令的模式将在任何其他模式之后隐式使用。

如果整个**expect**语句的参数需要多行，则所有参数都可以用花括号“括起来”为一行，从而避免以反斜杠续行。在这种情况下，除了括号，仍会发生正常的Tcl替换。

如果一个模式是关键字**eof**，则在文件结束时执行相应的<u>body</u>。如果一个模式是关键字**timeout**，则在超时时执行相应的<u>body</u>。如果不使用**timeout**关键字，则在超时时执行隐式的null动作。默认超时时间为10秒，但可以通过命令“`set timeout 30`”将其设置为30秒，-1表示永不超时。如果一个模式是关键字**default**，则在超时或文件结束时执行相应的<u>body</u>。

如果一个模式匹配了，则相应的<u>body</u>将被执行。 **expect**返回<u>body</u>的执行结果（如果没有匹配的模式，则返回空字符串）。如果有多个模式匹配，则执行首先出现的模式的<u>body</u>。

每次新输出到达时，都会按照模式列出的顺序将其与每个模式进行比较。因此，您可以通过确保最后一个模式的出现来测试是否缺少匹配项，例如提示。在没有提示的情况下，您必须使用**timeout**（就像您手动进行交互一样）。

模式以三种方式指定。默认情况下，使用Tcl的**字符串匹配**命令指定模式。 （这种模式也类似于通常称为“glob”模式的C shell正则表达式）。 **-gl**标志可用于保护可能与**expect**标志匹配的模式。任何以“-”开头的模式都应采用这种方式进行保护。 （所有以“-”开头的字符串都保留用于将来的选项。）

例如，以下片段查找一个成功的登录。 （请注意，推测**abort**是脚本中其他地方定义的过程。）

``` tcl
expect {
	busy               {puts busy\n ; exp_continue}
	failed             abort
	"invalid password" abort
	timeout            abort
	connected
}

```
第三个模式必须用引号引起来，因为它包含一个空格，否则该空格会将模式与动作分开。 具有相同动作的模式（例如第三和第四）需要再次列出动作。 可以通过使用正则表达式样式的模式来避免这种情况（请[参阅下文](#regexp)）。 在Tcl手册中可以找到有关形成glob-style模式的更多信息。

<span id="regexp">Regexp</span>风格的模式遵循Tcl的**regexp**（“正则表达式”的缩写）命令定义的语法。 **-re**表示要使用正则表达式模式。 可以使用regexp将前面的示例重写为：

``` tcl
expect {
	busy                         {puts busy\n ; exp_continue}
	-re "failed|invalid password" abort
	timeout                       abort
	connected
}
```
两种类型的模式都是“未锚定的”。这意味着模式不必匹配整个字符串，可以在字符串中的任何位置开始和结束匹配（只要其他所有内容都匹配）。使用\^来匹配字符串的开头，使用\$来匹配字符串的结尾。请注意，如果您不等待字符串的结尾，则当字符串是从派生进程中回显时，你的响应很容易在字符串的中间结束。虽然仍产生正确结果，输出看起来可能不自然。因此，如果可以准确地描述字符串末尾的字符，则鼓励使用\$。

请注意，在许多编辑器中，\^和\$分别匹配行的开头和结尾。但是，由于**Expect**不是面向行的，因此这些字符是与当前expect匹配缓冲区中的数据的开头和结尾（与行相对）匹配。 （另请参见下面有关“[system indigestion](#SystemIndigestion)”的说明。）

**-ex**标志使模式匹配为“精确”字符串。不对\*，\^等进行解释（尽管仍然必须遵守常规的Tcl约定）。精确模式总是不锚定的。

**-nocase**标志使模式匹配忽略大小写。模式不受影响。

在读取输出时，超过2000个字节后会强制“忘记”较早的字节。这可以通过函数**match_max**进行更改。 （请注意，太大的值可能会减慢模式匹配的速度。）如果<u>patlist</u>为 **full_buffer**，则如果已接收到<u>match_max</u>字节且没有匹配其他模式时，将执行相应的<u>body</u>。不管是否使用**full_buffer**关键字，都会将“忘掉”的字符写到expect_out(buffer)中。

如果<u>patlist</u>是关键字**null**，并且允许使用nulls（通过**remove_nulls**命令），则匹配到单个ASCII 0时则将执行相应的<u>body</u>。无法通过glob或regexp模式匹配0字节。

一旦匹配模式（或eof或full_buffer）后，所有匹配的和先前不匹配的输出都将保存在变量<u>expect_out(buffer)</u>中。最多9个正则表达式子字串匹配项保存在<u>expect_out(1，string)</u>到<u>expect_out(9，string)</u>中。如果在一个模式前使用 **-indices**标志，则这10个字符串的开始和结束索引（以适合**lrange**的形式）存储在变量<u>expect_out(X，start)</u>和<u>expect_out(X，end)</u>中，其中X为数字，对应缓冲区中子串的位置。 0表示与整个模式匹配的字符串，由glob模式和regexp模式产生。例如，如果某个进程的输出为“abcdefgh\\n”，则：

``` tcl
expect "cd"
```
的结果就像执行了以下语句：

``` tcl
set expect_out(0,string) cd
set expect_out(buffer) abcd
```
而"efgh\\n"则留在输出缓冲区中。 如果某个进程产生输出“abbbcabkkkka\\n”，则：

``` tcl
expect -indices -re "b(b*).*(k+)"
```
的结果就想执行了下列语句：

``` tcl
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

通常，匹配的输出会从Expect的内部缓冲区中丢弃。 可以通过在模式前面加上 **-notransfer**标志来阻止这种情况。 该标志在实验中特别有用（为方便起见，在实验中可以缩写为“-not”）。

与匹配的输出（或eof或full_buffer）关联的spawn id被存储在<u>expect_out(spawn_id)</u>中。

**-timeout**标志使当前的**expect**命令使用跟随的值作为超时，而不是使用**timeout**变量的值。

默认情况下，模式与当前进程的输出匹配，但是 **-i**标志指明来自指定spawn_id列表的输出可以与后面跟随的任何模式匹配（直到下一个 **-i**）。 spawn_id列表应该是空白符分隔的spawn_ids列表，或者是引用此类spawn_id列表的变量。

例如，以下示例等待当前进程的“connected”，或者等待\$proc2指定的spawn_id的“busy”，“fail”或“invalid password”。

``` tcl
expect {
	-i $proc2 busy {puts busy\n ; exp_continue}
	-re "failed|invalid password" abort
	timeout abort
	connected
}
```
全局变量<u>any_spawn_id</u>的值可用于将模式与当前**expect**命令中所有其他 **-i**标志列出的任何spawn_id进行匹配。 来自 **-i** 标志的没有关联模式的spawn_id（即另一个“**-i**”紧随其后）可用于 与<u>any_spawn_id</u>关联的同一<u>expect</u>命令中的任何其他模式。

**-i**标志还可以命名全局变量，在这种情况下，这个全局变量用于读取spawn id列表。 变量每次变化时都会重新读取。 这提供了一种在命令执行期间更改I/O源的方法。 通过这种方式提供的spawn id被称为“间接”spawn id。 

**break**和**continue**之类的动作会导致控制结构（即，**for**，**proc**）以常规的方式运行。 命令**exp_continue**允许**expect**自己继续执行，而不是像往常那样返回。 

这主要用于避免显式循环或重复**expect**语句。以下示例是自动化rlogin片段的一部分。如果rlogin提示输入密码，则**exp_continue**避免编写重复的**expect**语句（来再次查找提示）。

``` tcl
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
例如，以下片段可能帮助用户指导已经完全自动化的交互。 在这种情况下，终端将进入原始模式。 如果用户按下“+”，则变量增加。 如果按下“p”，则会向该过程发送多个返回消息，也许以某种方式对其进行poke，而“i”使用户可以与该进程进行交互，从而有效地从脚本中窃取控制权。 在每种情况下，**exp_continue**允许当前**expect**在执行当前动作后继续模式匹配。

``` tcl
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

## expect_after
<span id="expect_after">与**expect_before**</span>的工作方式相同，只是如果**expect**和**expect_after** 的模式都可以匹配，则使用**expect** 模式。 有关详细信息，请参阅[**expect_before**](#expect_before)命令。


## expect_background
<span id="expect_background">采用</span>与**expect**相同的参数，但它立即返回。 无论什么时候只要有新输入到达时都会测试模式。模式**timeout**和**default**对于**expect_background**来说毫无意义，会被默默丢弃。否则，**expect_background** 命令将像**expect** 一样使用**expect_before** 和**expect_after** 模式。

当 **expect_background** 动作执行时，同一spawn ID 的后台处理将被阻塞。 动作完成后，后台处理将被解除阻塞。 当后台处理被阻塞时，可以对同一个spawn ID 执行（前台）**expect**。

当**expect_background** 未被阻塞时，不可能执行**expect**。 通过声明具有相同spawn id 的新**expect_background** 来删除特定spawn id 的**expect_background**。 声明不带模式的**expect_background**会删除给定的spawn id在后台匹配模式的能力。


## expect_before
<span id="expect_before">采用</span>与expect相同的参数，但它立即返回。

## expect_tty校对完
``` tcl
expect_tty [expect_args]
```
<span id="expect_tty">就像</span>**expect**一样，但是它从/dev/tty中读取字符（即，用户击键）。 默认情况下，读取是在cooked模式下进行的。 因此，行必须以回车结尾以便expect看到它们。 这可以通过**stty**更改（请参见下面的 **[stty](#stty)** 命令）。
## expect_user校对完

``` tcl
expect_user [expect_args]
```
<span id="expect_user">就像</span>**expect**一样，但是它从stdin读取字符（即，用户的击键）。 默认情况下，读取是在cooked模式下进行的。 因此，行必须以回车结尾以便expect看到它们。 这可以通过**stty**更改（请参见下面的 **[stty](#stty)** 命令）。

## fork

## interact

## interpreter

## log_file

## log_user校对完

``` tcl
log_user -info|0|1
```
默认情况下，send/expect对话框记录到stdout（如果一个日志文件被打开，则记录到该日志文件）。 通过命令“log_user 0”禁用到stdout的日志记录，并通过“log_user 1”重新启用。 到日志文件的记录不变。

**-info**标志使log_user返回给定的最新no-info参数的描述。

## match_max

## overlay

## parity
## remove_nulls

## send

``` tcl
send [-flags] string
```
<span id="send">发送</span>字符串到当前进程。例如，命令

``` tcl
send "hello world\r"
```
发送字符`hello空格world回车`到当前进程。 （Tcl引入了类似printf的命令（称为**format**），可以构建任意复杂的字符串。）

尽管具有行缓冲输入的程序在发送回车字符之前不会读取字符，但**Expect**仍会立即发送字符。回车字符表示为“\r”。

**--** 标志强制将下一个参数解释为字符串而不是标志。任何字符串都可以以“ **--** ”开头，无论它实际上是否看起来像一个标志。这提供了一种可靠的机制来指定可变字符串，而不会被那些偶然看起来像标志的字符串妨碍。 （所有以“**-**”开头的字符串都保留用于将来的选项。）

**-i** 标志指明将字符串发送到指定的spawn_id。如果spawn_id为<u>user_spawn_id</u>，并且终端处于原始模式，则将字符串中的换行符转换为回车换行符，以便它们看起来像终端处于cooked模式。 **-raw** 标志禁用此转换。

**-null** 标志发送空字符（0字节）。默认情况下，发送一个空值。可以在 **-null** 后面跟随一个整数，以指示要发送多少个null。

**-break** 标志生成一个中断条件。仅当spawn id引用了通过“spawn -open”打开的tty设备时，才有意义。如果生成了诸如tip之类的进程，则应使用tip的约定来生成中断。

**-s** 标志强制输出“缓慢地”发送，因此避免了一种场景，即计算机输出超过了输入缓冲区，该输入缓冲区为人类设设计的，而人类输入时永远不会超过。该输出由变量“send_slow”的值控制，该变量是一个有两个元素的列表。第一个元素是整数，它描述要原子发送的字节数。第二个元素是一个实数，它描述原子发送必须间隔的秒数。例如，“`set send_slow {10 .001}`”将强制“`send -s`”以每发送10个字符之间间隔1毫秒的方式发送字符串。

 **-h** 标志强制输出（以某种方式）像人类实际键入一样地发送。类似人一样的延迟会出现字符之间。（该算法基于Weibull分布，并进行了修改以适合该特定应用。）此输出由变量“send_human”的值控制，该变量是一个五元素列表。前两个元素是字符的平均间隔到达时间，以秒为单位。默认情况下使用第一个。第二个用于单词结尾，以模拟这种过渡时偶尔发生的微妙的停顿。第三个参数是可变性的度量，其中.1十分可变，1是合理可变，而10十分不可变。极限是0到无穷大。最后两个参数分别是最小和最大到达间隔时间。最小值和最大值被用于维持(last)和剪断(clip)最终时间。如果最小值和最大值差距足够大，则最终平均值可能与给定平均值有很大不同。

例如，以下命令模拟快速一致的打字员：

``` tcl
set send_human {.1 .3 1 .05 2}
send -h "I'm hungry.  Let's do lunch."

```

而宿醉后，以下各项可能更适合：

``` tcl
set send_human {.4 .4 .2 .5 100}
send -h "Goodd party lash night!"
```
请注意，虽然您可以通过将错误和更正嵌入到send参数中来自行设置错误更正情况，但不会模拟错误。

用于发送空字符、发送中断、强制缓慢输出、人类风格输出的标志是互斥的。仅使用指定的最后一个。用于发送空字符或中断的标志不能指定<u>string</u>参数。

在第一次用**send**发送到进程之前，最好有一个 **expect**。 **expect** 将等待进程启动，而 **send** 不会。 特别是，如果第一次发送在进程开始运行之前完成，则可能会导致数据被忽略。 在交互式程序不提供启动提示的情况下，您可以在发送之前先进行延迟，如下所示：

``` tcl
# To avoid giving hackers hints on how to break in,
# this system does not prompt for an external password.
# Wait for 5 seconds for exec to complete
spawn telnet very.secure.gov
sleep 5
send password\r
```
**exp_send** 是**send**的别名。 如果在Tk环境中使用 **Expectk** 或 **Expect** 的某些其他变体，则**send**由Tk定义，目的完全不同。 提供 **exp_send** 是为了实现环境之间的兼容性。 为其他Expect的其他发送命令提供了类似的别名。

## send_error
```tcl
send_error[-flags] string
```
<span id="send_error">与 **send** 类似</span>，只不过输出被发送到 stderr 而不是当前进程。

## send_log
```tcl
send_log [--] string
```
<span id="send_log">与 **send** 类似</span>，只不过string被发送到日志文件（请参阅[**log_file**](#log_file)）而不是当前进程。如果没有日志文件打开，则参数被忽略。

## send_tty
```tcl
send_tty [-flags] string
```
<span id="send_tty">与 **send** 类似</span>，只不过输出被发送到/dev/tty 而不是当前进程。

## send_user
```tcl
send_user [-flags] string
```
<span id="send_user">与 **send** 类似</span>，只不过输出被发送到 stdout 而不是当前进程。

## sleep
<span id="sleep">使脚本</span>休眠给定的秒数。 秒可以是小数。 中断（如果您使用 Expectk，则为 Tk 事件）在 Expect 休眠时进行处理。

## spawn 校对完毕
```tcl
spawn [args] program [args]
```
<span id="spawn">创建</span>一个运行<u>program</u> <u>args</u>的新进程。它的 stdin、stdout 和 stderr 都连接到 **Expect**，以便它们可以被其他 **Expect** 命令读取和写入。连接因**close**或进程本身关闭任意文件标识符而中断。

当进程由 **spawn** 启动时，变量 <u>spawn_id</u> 被设置为引用该进程的描述符。 <u>spawn_id</u> 描述的进程被认为是<u>当前进程</u>。 <u>spawn_id</u> 可以被读取或写入，实际上提供了作业控制。

<u>user_spawn_id</u> 是一个全局变量，包含一个引用用户的描述符。例如，当 <u>spawn_id</u> 设置为该值时，**expect** 的行为类似于 **expect_user**。

<u>error_spawn_id</u> 是一个全局变量，包含一个引用标准错误的描述符。例如，当 <u>spawn_id</u> 设置为此值时，**send** 的行为类似于 **send_error**。

<u>tty_spawn_id</u> 是一个全局变量，包含一个指向 /dev/tty 的描述符。如果 /dev/tty 不存在（例如在 cron、at 或批处理脚本中），则 <u>tty_spawn_id</u>是未定义的。

这可以被测试：
```tcl
if {[info vars tty_spawn_id]} {
	# /dev/tty exists
} else {
	# /dev/tty doesn't exist
	# probably in cron, batch, or at script
}
```
**spawn** 返回 UNIX 进程 id。 如果没有派生进程，则返回 0。 变量 <u>spawn_out(slave,name</u>) 设置为 pty 从设备的名称。

默认情况下， **spawn** 会回显命令名称和参数。 **-noecho** 标志会阻止 **spawn** 这样做。

**-console** 标志导致控制台输出被重定向到派生进程。 并非所有系统都支持此功能。

在内部， **spawn** 使用 pty，以与用户的 tty 相同的方式初始化。 这将进一步初始化，以便所有设置都是“sane”（根据 stty(1)）。 如果定义了变量 <u>stty_init</u>，它将以 stty 参数的样式被解释为进一步的配置。 例如，“set stty_init raw”将导致之后派生进程的终端以原始模式（raw mode）启动。 **-nottycopy** 根据用户的 tty 跳过初始化。 **-nottyinit** 跳过“sane”初始化。

通常， **spawn** 执行所需的时间很少。 如果您注意到 **spawn** 花费了大量时间，则可能是遇到了挤入的 pty（wedged pty）。 在 ptys 上运行了许多测试，以避免与出错进程纠缠。 （每个挤入的pty 需要 10 秒。）使用 **-d** 选项运行 **Expect** 将显示 **Expect** 是否遇到许多处于奇怪状态的 pty。 如果您无法终止这些 pty 所连接的进程，您唯一的办法可能是重新启动。

如果由于 exec(2) 失败（例如，当<u>program</u>不存在时）无法成功派生<u>program</u>，则下一个**interact**或**expect**命令将返回错误消息，就像<u>program</u>已运行并产生错误消息作为输出一样。 这种行为是执行 **spawn** 的自然结果。 在内部，spawn forks，之后产生的进程无法与原始的 **Expect** 进程进行通信，除非通过 spawn_id 进行通信。

**-open** 标志导致下一个参数被解释为 Tcl 文件标识符（即，由 **open** 返回）。然后可以使用 spawn id，就好像它是一个 派生进程一样。 （不应再使用文件标识符。）这使您可以将原始（raw）设备、文件和管道视为派生进程，而无需使用 pty。 返回 0 表示没有关联的进程。 当与派生进程的连接关闭时，Tcl 文件标识符也关闭。 **-leaveopen** 标志与 **-open** 类似，除了 **-leaveopen** 导致文件标识符即使在 spawn id 关闭后仍保持打开状态。

**<span id="spawn_pty">-pty</span>** 标志导致打开一个 pty 但不派生进程。 返回 0 表示没有关联的进程。 spawn_id 照常设置。

变量 <u>spawn_out(slave,fd)</u> 设置为对应于 pty slave 的文件标识符。 它可以使用“close -slave”关闭。

**-ignore** 标志指明了要在派生进程中忽略的信号。 否则，信号将获得默认行为。 除了每个信号都需要一个单独的标志外，信号就像在 **trap** 命令中指明一样。

## strace
## stty
## system
## timestamp
## trap
## wait校对完毕
```tcl
wait [args]
```
<span id="wait">延迟</span>直到派生的进程（或当前进程，如果没有指明）终止。

 **wait** 通常返回一个包含四个整数的列表。第一个整数是等待的进程的 pid。第二个整数是对应的 spawn id。如果发生操作系统错误，则第三个整数为 -1，否则为 0。如果第三个整数是 0，则第四个整数是派生进程返回的状态。如果第三个整数是-1，那么第四个整数就是操作系统设置的errno的值。此外，还设置了全局变量 errorCode。

其他元素可以会出现在**wait**返回值的末尾。可选的第五个元素标识一类信息。目前，此元素的唯一可能值是 CHILDKILLED，在这种情况下，接下来的两个值是 C 样式信号名称和简短的文本描述。

**-i** 标志声明了要等待的进程对应于指明的 spawn_id（不是进程 id）。在 SIGCHLD 处理程序中，可以使用 spawn id -1 等待任何派生进程。

**-nowait** 标志使wait立即返回并指示wait成功。当进程退出（稍后）时，它会自动消失，无需显式等待。

 **wait** 命令也可以用于等待使用参数“-i -1”的fork进程。与用于派生进程不同的是，该命令可以随时执行。无法控制等到的进程，但可以检查进程 id 的返回值。




------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Expect manpages》</font>，expect当前版本为5.45.4，手册更新日期为1994-12-29。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
