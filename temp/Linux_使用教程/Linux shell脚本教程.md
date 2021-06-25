---
title: Linux shell脚本教程 
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 1 shell脚本简介
&emsp;&emsp;shell script 是利用 shell 的功能所写的一个程序 ，这个程序是使用纯文本文件，将一些shell的语法与指令(含外部指令)写在里面，搭配正则表达式、管道命令与数据流重导向等功能，以达到我们所想要的处理目的。
## 1.1 执行shell脚本
*  bash script-name或者sh script-name
这是当脚本文件本身没有可执行权限（即文件权限属性x位为-号）时常使用的方法，或者脚本文件开头没有指定解释器时需要使用的方法。推荐使用这种方法。

* fullpath/script-name或者./script-name
指在当前路径下执行脚本（脚本需要有执行权限），需要将脚本文件的权限改为可执行（即文件权限属性为x位）。具体方法为:chmod a+x script-name。然后通过执行脚本绝对路径或者相对路径就可以执行脚本了。



* source script-name或者. script-name
source或者“.”命令的功能是：读入脚本并执行脚本，即在当前Shell中执行source或“.”加载并执行的相关脚本文件的命令及语句，而不是产生一个子Shell来执行文件中的命令。这是和其他几种执行shell方式的最大不同。

>注意：在生产环境中，运维人员由于忘记为该脚本设置可执行权限，然后直接使用，导致出错。因此，推荐第一种 bash script-name。



>**注意**：
>
>1. 指令的执行是从上而下、从左而右的分析与执行；
>2. 指令、选项与参数间的多个空白都会被忽略掉；
>3. 空白行也将被忽略掉，并且 [tab] 按键所推开的空白同样视为空格键；
>4. 如果读取到一个 Enter 符号 (CR) ，就尝试开始执行该行 (或该串) 命令。



## 1.2 shell解释器


不同的shell解释器具备不同的功能。


* sh
  sh 的全称是 Bourne shell，由 AT&T 公司的 Steve Bourne开发，为了纪念他，就用他的名字命名了。sh 是 UNIX 上的标准 shell，很多 UNIX 版本都配有 sh。sh 是第一个流行的 Shell。在Linux中已经成为其它shell解释器的符号链接。

* bash
 由 GNU 组织开发，大多数Linux系统默认使用的shell，保持了对 sh 的兼容性，支持c shell 和 korn shell 的特色功能，是各种 Linux 发行版默认配置的 shell。bash 兼容 sh 意味着，针对 sh 编写的 Shell 代码可以不加修改地在 bash 中运行。尽管如此，bash 和 sh 还是有一些不同之处：
  * 一方面，bash 扩展了一些命令和参数；
  * 另一方面，bash 并不完全和 sh 兼容，它们有些行为并不一致，但在大多数企业运维的情况下区别不大，特殊场景可以使用 bash 代替 sh。

* dash
GNU/Linux操作系统中的 /bin/sh 本是 bash (Bourne-Again Shell) 的符号链接，但鉴于 bash 过于复杂，有人把 bash 从 NetBSD 移植到 Linux 并更名为 dash (Debian Almquist Shell)，并建议将 /bin/sh 指向它，以获得更快的脚本执行速度。Dash Shell 比 Bash Shell 小的多，符合POSIX标准。Debian和Ubuntu中，/bin/sh默认已经指向dash，这是一个不同于bash的shell，它主要是为了执行脚本而出现，而不是交互，它速度更快，但功能相比bash要少很多，语法严格遵守POSIX标准。可以使用`sudo dpkg-reconfigure dash`来禁用dash。

* csh
C shell 使用的是“类C”语法，csh是具有C语言风格的一种shell，由bill joy开发，其内部命令有52个，较为庞大。目前使用的并不多，已经被/bin/tcsh所取代。

* ksh
Korn shell 的语法与Bourne shell相同，同时具备了C shell的易用特点。许多安装脚本都使用ksh,ksh 有42条内部命令，与bash相比有一定的限制性。

* tcsh
tcsh是csh的增强版，与C shell完全兼容。

* zsh
目前Linux里最庞大的一种shell：zsh。它有84个内部命令，使用起来也比较复杂。一般情况下，不会使用
该shell。


## 1.3 shell脚本组成
* 第一行用`#!/bin/shell解释器`开头 ，声明这个脚本使用的 shell 名称，一般有bash、dash、sh、csh、ksh，如果省略，则使用默认的shell解释器。
* 程序内容的说明，包括：
  * script 的功能；
  * script 的版本信息；
  * script 的作者与联络方式；
  * script 的版权宣告方式；
  * script 的 History (历史纪录)；
  * script 内较特殊的指令，使用绝对路径的方式来下达；
* 主要环境变量设置
* 程序，主要包括：
  * 变量定义
  * 函数定义
  * shell命令
  * 流程控制
* 返回执行结果：`exit n`。exit使脚本中断执行，并将n返回给系统。

# 2 shell 基本语法
## 2.1 注释
* 单行注释
```bash
#注释
```
* 多行注释
```bash
：<<标识符
注释
注释
标识符

```
## 2.2 一行多句与多行一句
* 如果一行的内容太多，则可以使用`\`来延伸至下一行
* 如果一行中写多个语句，语句之间用分号隔开，最后的分号可以省略
  * `cmd1;cmd2`或`cmd1;cmd2;`，最后的分号可省略，在当前shell中执行。
  * `（cmd1;cmd2）`或`（cmd1；cmd2;）`，在子shell中执行。
  * ` { cmd1;cmd2; }`，空格与最后的分号不可省略，在当前shell中执行。
* 对于复合语句（条件、循环）如果写作一句话,关键字then、eles/fi、do、done前方要加上分号。



## 2.3 命令替换与管道
* 命令替换是将一个命令的输出作为另一个命令的输入。有两种形式：
  * \`命令\`
     * 容易与单引号混淆
     * 不支持嵌套
  * \$(命令) 
     * 支持命令嵌套
```bash
echo `pwd` 

#等效于

echo $(pwd) 
```
* 管道是将一个命令的输出作为另一个命令的输入。
```bash
cmd1 | cmd2
```



## 2.4  重定向
&emsp;&emsp;大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。
### 2.4.1 标准重定向
|命令|	说明|
|:--|:--|
|command > file|	将输出重定向到 file。
|command < file|	将输入重定向到 file。
|command >> file|	将输出以追加的方式重定向到 file。
|n > file|	将文件描述符为 n 的文件重定向到 file。
|n >> file|	将文件描述符为 n 的文件以追加的方式重定向到 file。
|n >& m	|将输出文件 m 和 n 合并。
|n <& m	|将输入文件 m 和 n 合并。
|<< tag	|将开始标记 tag 和结束标记 tag 之间的内容作为输入。


* 一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：
  * 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
  * 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
  * 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。
* 默认情况下，`command > file `将 stdout 重定向到 file，`command < file `将stdin 重定向到 file。
* 如果希望 stderr 重定向到 file，可以这样写：`command 2 > file`
* 如果希望 stderr 追加到 file 文件末尾，可以这样写：`command 2 >> file `
* 如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：` command > file 2>&1`或者 `$ command >> file 2>&1`
* 如果希望对 stdin 和 stdout 都重定向，可以这样写：` command < file1 >file2`

### 2.4.2 Here Document

&emsp;&emsp;Here Document 是 Shell 中的一种特殊的重定向方式，它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command。它的基本的形式如下：
```bash
command << delimiter
    document
delimiter
```

&emsp;&emsp;注意：结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。开始的delimiter前后的空格会被忽略掉。


### 2.4.3 /dev/null 文件
&emsp;&emsp;如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：
```bash
$ command > /dev/null
```
&emsp;&emsp;`/dev/null `是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 `/dev/null` 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。
如果希望屏蔽 stdout 和 stderr，可以这样写：
```bash
$ command > /dev/null 2>&1
```

 
## 2.5 文件包含
&emsp;&emsp;和其他语言一样，Shell 也可以包含外部脚本。这样可以很方便的封装一些公用的代码作为一个独立的文件。Shell 文件包含的语法格式如下：
```bash
. filename   # 注意点号(.)和文件名中间有一空格

#或

source filename
```


## 2.6 表达式格式

运算符（赋值符除外）与操作数之间要有空格。

## 2.7 shell通配符（不是正则表达式）
* `*`——任意个字符
* `?`——任意一个字符
* `[abc]`——在范围内的任意单个字符
* `[!abc]`——不在范围内的任意单个字符
* `[[:class:]]`——在集合内的任意单个字符，calss的取值请参考[《正则表达式详解》](https://blog.csdn.net/liao20081228/article/details/79719359#t5)
* `[![:class:]]`——不在集合内的任意单个字符，calss的取值请参考[《正则表达式详解》](https://blog.csdn.net/liao20081228/article/details/79719359#t5)

# 3 shell 变量
&emsp;&emsp;shell变量没有类型，都是字符串。
## 3.1  变量分类
### 3.1.1 自定义变量
```bash
#定义变量：
    [属性] [选项] 变量名称[=值]

#使用变量：
    $变量名 
#或 
    ${变量名}
```

* 标识符命名规则
  * 变量名不加美元符号
  * 变量名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
  * 变量名中间不能有空格
  * 变量名不能使用bash里的关键字（可用help命令查看保留关键字）。
  * 如果变量有初值，则等号两侧不能有空格。
* 关键字
  * unset ：删除变量
  * read ： 变量值从键盘输入
     * -p : 设置提示字符
     * -t  : 设置超时时间
     * -n : 输入的最大长度
  * readonly ：只读变量
  * export ：设为环境变量
  * unsetenv: 取消环境变量
  * local ：局部变量，尤其是在递归中防止出错，**只能用在函数中**。shell中除了函数中用local声明的变量是局部变量，其余都是全局变量。
  * declare： 显式声明变量，语法如下：
     * `declare [+/-][raxi] 变量名[=值]`
         * +/- 　"-"可用来指定变量的属性，"+"则是取消变量所设的属性。
         * r 　将变量设置为只读。
         * x 　将变量设置为环境变量，可供shell以外的程序来使用。
         * i 　将变量设置为整数，如果为其他，则转为0。
         * a   将变量设置为一维数组。

     * `declare [-r/a/x/i/f]`
         * 列出具有该属性的变量或函数
         * 如果不带任何参数则列出所有的变量和函数




### 3.1.2 环境变量
&emsp;&emsp;系统预定义的变量，一般在/etc/profile、/etc/bashrc、\~/.bashrc、~/.profile中进行定义 。
|环境变量| 描述|
|:--|:--|
|BASH_SUBSHELL | 记录当前子shell的层次。BASH_SUBSHELL是从0开始计数的整数。
|BASH_VERSINFO|  是一个数组包含六个元素，这六个元素显示bash的版本信息。
|BASH_VERSION|   显示shell版本的信息
|DIRSTACK|        记录了栈顶的目录值，初值为空
|EUID |            有效用户id
|GLOBLGNORE|    是由冒号分割的模式列表，表示通配时忽略的文件名集合。
|GROUPS |        记录当前用户所属的组
|HOME|           用户主目录 
|HISTSIZE|　      历史记录数
|HOSTNAME|      主机名
|HOSTTYPE|       记录系统的硬件架构。
|	IFS  |      用于设置指定shell域分隔符，默认情况下为空格。
|LANG |　　        语言相关的环境变量，多语言可以修改此环境变量
|LC_ADDRESS|       地址书写方式
|LC_COLLATE|      比较和排序习惯
|LC_CTYPE |       用户所使用的语言符号及其分类
|LC_IDENTIFICATION|  locale对自身包含信息的概述
|LC_MEASUREMENT| 度量衡表达方式()
|LC_MESSAGES |    信息主要是提示信息,错误信息, 状态信息, 标题, 标签, 按钮和菜单等
|LC_MONETARY|    货币单位
|LC_NAME  |        姓名书写方式
|LC_NUMERIC |    数字
|LC_PAPER |         默认纸张尺寸大小 ()
|LC_TELEPHONE |   电话号码书写方式 
|LC_TIME |         时间显示格式()，
|LOGNAME |      登录名
|LESSCLOSE|
|LS_COLORS|	      颜色配置
|MACHTYPE |      记录硬件架构和操作系统类型
|MAIL|　　　　　　 当前用户的邮件存放目录
|OLDPWD |         用户上一次工作目录
|OSTYPE |         记录操作系统类型。
|PATH |            文件搜索路径，决定了shell将到哪些目录中寻找命令或程序
|PPID |			  父进程ID
|PS1　|　　　　　   基本提示符，对于root用户是#，对于普通用户是\$
|PS2　|　　　　　   附属提示符，默认是“>”
|PWD  |           用户当前工作目录 
|SECONDS|      记录脚本从开始到结束耗费的时间。
|SHELL| 　　　　   前用户Shell类型
|SHELLOPTS|      记录了处于“开”状态的shell选项列表，它只是一个只读变量。
|SHLVL |         记录了bash嵌套的层次，第一个Shell。  \$SHLVL=1如果在这个Shell中执   行脚本，脚本中的$SHLVL=2 
|SSH_TTY|
|SSH_CLIENT |         ssh协议客户端ip和端口
|SSH_CONNECTION   | ssh协议客户端和服务器端的ip与端口
|TERM |			  颜色模式，16级或256级
|UID|				 用户ID
|USER  |          用户名
|XDG_SESSION：_ID | 终端会话id
|XDG_RUNTIME_DIR|系统用户独立的运行目录
|LINENO|LINENO出现时的当前行号|
|BASH_LINENO[]|函数被调用时的行号，0表示所在函数的调用语句在源文件中的行号|
|BASH_SOURCE[]|函数所在文件，0表示当前函数所在文件，1表示调用函数所在文件，以此类推|
|FUNCNAME[]|函数调用数组,0表示当前函数，1表示调用函数，依此类推|





### 3.1.3 参数变量
&emsp;&emsp;参数用来处理传递的参数。包括命令行参数和函数参数。
|变量| 描述|
|:--|:--|
|\$1,\$2…\$n |命令行或函数调用传入的参数，相当于C语言中的argv[i]。不算程序本身。当n>=10，要使用\${n}
|\$0 |表示shell程序名称，每一项相当于 C语言中 argv[0]
|\$# | 传递到脚本的参数个数，不包含\$0 
|\$* | 把全部参数显示为一个字符串，以”\$1 \$2 … &emsp;&emsp;\$n”的形式输出所有参数。
|\$@ | 传入脚本的全部参数，argv[1] 到argv[n-1] ,每个参数作为一个字符串。
|\$? | 前个命令或者函数返回值。0表示没有错误，其他任何值表明有错误。
|\$$|  脚本运行的当前进程号 
|\$!|  后台运行的最后一个进程的ID号
|\$-|	显示Shell使用的当前选项，与set命令功能相同。


### 3.1.4 字符串

* 字符串：字符串是shell编程中最常用最有用的数据类型，字符串可以用单引号，也可以用双引号，也可以不用引号。习惯数值不加引号，字符串加引号。
 * 单引号，如：`str='this is a var'`
     * 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
     * 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。
 * 双引号，如：`str="Hello, I know your are \"$your_name\"! \n"`
     * 双引号里可以有变量，命令替换，单双引号。
     * 双引号里可以出现转义字符。
 * 无论赋值还是取变量值时，只要**字符串有空格则必须用引号括起来 或 进行转义**。 如` a=“hello world” `
 * 如果要使用字符\$、”、’、空格、\，则必须用转义符，如：`a=What\'s\ your\ \"topic\"\?` 
 * 变量是字符串时在取值时最好加上引号，否则可能出错。

### 3.1.5 数组
* bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小（与 PHP 类似）。与大部分编程语言类似，数组元素的下标由0开始。数组定义格式： 
```bash
array_name=(value1 value2  ... valuen) #下标从0开始， 各个数据之间用空格隔开 

array_name[index1]=value1; array_name[index3]=value3;...;array_name[indexn]=valuen; #下标可以不连续

array_name=([index1]=value1 [index3]=value3 ... [indexn]=valuen) 

```
* 访问某个数组元素  ：`${array_name[index]} `
* 数组的所有元素：`${array_name[@]}`或`${array_name[*]}`
* 数组的所有元素相当于变量名
  * 求数组长度：`${#array_name[@]}`或`${#array_name[*]}`
  * 截取部分数组元素：`${array_name[@]:pos:len}`
     * len可省略，表示到最后。
     * len省略时，pos可以为负数，但是前面必须有空格或正数，或将负数括起来



## 3.2 变量操作
|特殊表达式|描述|
|:--|:--|
|`${var}`|	变量var的值, 与`$var`相同,，加花括号是为了帮助解释器识别变量的边界，推荐加上。
|`${var-DEFAULT}`|如果没有声明var，或`declare var`,则返回`DEFAULT`，否则返回var。
|`${var:-DEFAULT}`|	如果var没有被声明或值为空, 则返回`DEFAULT`，否则返回var。
|`${var=DEFAULT}`|如果没有声明var，或`declare var`,则返回`DEFAULT`并赋值给var，否则返回var。
|`${var:=DEFAULT}`|如果var没有被声明或值为空,,则返回`DEFAULT`并赋值给var，否则返回var。
|`${var+OTHER}`|如果没有声明var，或`declare var`,则返回`null`，否则返回`OTHER`。
|`${var:+OTHER}`|如果var没有被声明或值为空, 则返回`null`，否则返回`OTHER`。
|`${var?ERR_MSG}`|如果没被声明var，或`declare var`, 那么就打印`ERR_MSG` 
|`${var:?ERR_MSG}`|如果var没有被声明或值为空, 那么就打印`ERR_MSG` 
|`${!varprefix*}`|匹配之前所有以varprefix开头进行声明的变量，不包括`declare var`
|`${!varprefix@}`|匹配之前所有以varprefix开头进行声明的变量，不包括`declare var`
|**变量拼接**||
|`${str1}${str2}`|将两个字符串合成一个字符串
|**变量替换**||
|`${var/pattern1/pattern2}`|使用pattern2, 来代替var中第一个与pattern1匹配的子串，支持shell通配符。
|`${var//pattern1/pattern2}`|使用pattern2, 来代替var中所有与pattern1匹配的子串，支持shell通配符。
|`${var/#pattern1/pattern2}`|如果var的开始与pattern1匹配,则用pattern2来代替与pattern1匹配的子串，支持shell通配符。
|`${var/%pattern1/pattern2}`|如果var的结束与pattern1匹配,则用pattern2来代替与pattern1匹配的子串，支持shell通配符。
|**变量截取**||
|`${var:pos}`|提取第pos个字符到末尾的所有字符。若pos为正数，从左边0处开始；若num为负数，从右边开始提取字串，但必须在冒号后面加空格或正数或整个pos加上括号
|`${var:pos:len}`|从var的第`$pos`个字符开始提取长度为`len`的子串。不能为负数。
|`${var#pattern}`|如果var以pattern开头，就把var中最短的匹配内容去掉。支持shell通配符。
|`${var##pattern}`|如果var以pattern开头，就把var中最长的匹配内容去掉。支持shell通配符。
| `${var%pattern}`|如果var以pattern结尾，就把var中最短的匹配内容去掉。支持shell通配符。
|`${var%%pattern}`|如果var以pattern结尾，就把var中最长的匹配内容去掉。支持shell通配符。
|**查找子串**|
|`expr index var subvar`|查找子出现的位置
|**变量长度**|
|`${#str}`|获取字符串长度


#  运算符与表达式
* Shell 和其他编程语言一样，支持多种运算符，包括：
  * 算数运算符
  * 关系运算符
  * 布尔运算符
  * 字符串运算符
  * 文件测试运算符
  * 位运算符
* 原生bash不支持数学运算，但是可以通过其他方式来实现。
  * 借助命令替换， \`expr 表达式\` 或者 `$(expr 表达式)` 。
  * 借助C语言，常用`$((表达式))`或`$[表达式]`
  * 借助let关键字，`let var=表达式`。
* 表达式和运算符之间要有空格，但是C语言风格不需要。
* 条件表达式要放在方括号之间，并且要有空格，例如:` [$a==$b] `是错误的，必须写成` [ $a == $b ]`。
* 使用[[ ... ]]条件判断结构，而不是[ ... ]，能够防止脚本中的许多逻辑错误。
  * 使用[ ... ] 作为条件判断时，如果变量是字符串应该加上引号
  * 使用[[ ... ]]，不能使用逻辑运算符-a 和 -o，但可以使用||和&&；使用[ ... ]可以使用逻辑运算符-a 和 -o，但不可以使用||和&&

## 4.1 算术运算符 

* 乘号`*`前边必须加反斜杠`\`才能实现乘法运算；
* `$((表达式))`，此处表达式中的 `*` 不需要转义符号 `\`。

|运算符|	说明|变量 a 为 10，变量 b 为 20|
|:--|:--|:--|
|+|	加法|	`expr $a + $b` 结果为 30。
|-|	减法|	`expr $a - $b` 结果为 -10。
|*|	乘法|	`expr $a * $b` 结果为  200。
|/|	除法|	`expr $b / $a` 结果为 2。
|%|	取余|	`expr $b % $a` 结果为 0。



## 4.2 关系运算符

* 关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

|运算符|	说明|	变量 a 为 10，变量 b 为 20|
|:--|:--|:--|
|-eq|	检测两个数是否相等，相等返回 true。|	`[ $a -eq $b ]` 返回 false。
|-ne|	检测两个数是否相等，不相等返回 true。|	`[ $a -ne $b ]` 返回 true。
|-gt|	检测左边的数是否大于右边的，如果是，则返回 true。|	`[ $a -gt $b ]` 返回 false。
|-lt|	检测左边的数是否小于右边的，如果是，则返回 true。|	`[ $a -lt $b ]` 返回 true。
|-ge|	检测左边的数是否大于等于右边的，如果是，则返回 true。|	`[ $a -ge $b ]` 返回 false。
|-le|	检测左边的数是否小于等于右边的，如果是，则返回 true。|	`[ $a -le $b ] `返回 true。

## 4.3 逻辑运算符

|运算符	|说明	|假定变量 a 为 10，变量 b 为 20|
|:--|:--|:--|
|!|	非运算，表达式为 true 则返回 false，否则返回 true。	|`[ ! false ]` 返回 true。
|-o或&#124;&#124;|或运算，有一个表达式为 true 则返回 true。|	`[ $a -lt 20 -o $b -gt 100 ] `返回 true。
|-a或&&	|与运算，两个表达式都为 true 才返回 true。|	`[ $a -lt 20 -a $b -gt 100 ]` 返回 false。

>注：-o 、-a使用在test内部，而|| 、&&使用在test之间

## 4.4 位运算符
|运算符	|说明	|
|:--|:--|
|<<|左移|
| \>>|右移|
|\||按位或|
|&|按位且|
|~|按位取反|
|^|按位异或|

## 4.5 字符串运算符

|运算符	|说明|	变量 a 为 "abc"，变量 b 为 "efg"|
|:--|:--|:--|
|= 或==|	检测两个字符串是否相等，相等返回 true。|	`[ $a = $b ]` 返回 false。
|!= |	检测两个字符串是否相等，不相等返回 true。|	`[ $a != $b ]` 返回 true。
|-z var|	检测字符串长度是否为0，为0返回 true。|	`[ -z $a ]` 返回 false。
|-n var |	检测字符串长度是否为0，不为0返回 true。|	`[ -n $a ]` 返回 true。
|var	|检测字符串是否为空，不为空返回 true。|	`[ $a ]` 返回 true。

## 4.6 文件测试运算符

|操作符	|说明|	举例file=~/.bashrc|
|:--|:--|:--|
|-a file|	检测文件是否存在，如果是，则返回 true。|`[ -a $file ]`返回true|
|-b file|	检测文件是否是块设备文件，如果是，则返回 true。|	`[ -b $file ]` 返回 false
|-c file|	检测文件是否是字符设备文件，如果是，则返回 true。|	`[ -c $file ]` 返回 false
|-d file|	检测文件是否是目录，如果是，则返回 true。|	`[ -d $file ]` 返回 false。
|-e file|	检测文件（包括目录）是否存在，如果是，则返回 true。	|`[ -e $file ]` 返回 true
|-f file|	检测文件是否是普通文件，如果是，则返回 true。|	`[ -f $file ]` 返回 true。
|-h file|	检测文件是否是符号链接，如果是，则返回true	|`[ -h $file ]`返回false
|-g file|	检测文件是否设置了 SGID 位，如果是，则返回 true。|	`[ -g $file ]` 返回 false
|-k file|	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。|`[ -k $file ]` 返回 false。
|-p file|	检测文件是否是有名管道，如果是，则返回 true。|	`[ -p $file ]` 返回 false。
|-r file|	检测文件是否可读，如果是，则返回 true。	|`[ -r 0 ]` 返回 true。
|-s file|	检测文件是否为空（文件大小是否大于0），不为空返回 true。|	`[ -s $file ]` 返回 true。
|-t fd| 	如果文件描述符 FD 打开且指向一个终端则为真。|`[ -t $file ]` 返回 false。	
|-u file|	检测文件是否设置了 SUID 位，如果是，则返回 true。|	`[ -u $file ] 返回 false。
|-w file|	检测文件是否可写，如果是，则返回 true。|`[ -w $file ]` 返回 truee
|-x file|	检测文件是否可执行，如果是，则返回 true。|	`[ -x $file ]` 返回 true
|-O FILE|	如果 FILE 存在且属有效用户ID则为真。|	
|-G FILE|	如果 FILE 存在且属有效用户组则为真。|	
|-L FILE|	如果 FILE 存在且是一个符号连接则为真。|	
|-N FILE|	如果 FILE存在且自上次访问过后发生了修改则为真。	|
|-S FILE|	如果 FILE 存在且是一个套接字则为真。|	
|FILE1 -nt FILE2|	如果 FILE1 比 FILE2 要新, 或者 FILE1存在且 FILE2 不存在则为真。|	
|FILE1 -ot FILE2|	如果 FILE1 比 FILE2 要老, 或者 FILE2 存在且 FILE1 不存在则为真。|	
|FILE1 -ef FILE2|	如果 FILE1 和 FILE2 指向相同的设备和节点号则为真。	|
|-o OPTION|	如果 shell选项 “OPTIONNAME” 开启则为真。|	

# 5 控制语句
## 5.1 选择语句
### 5.1.1 条件语句
```bash
#单分支：

if [ condition ]
then  
    action
fi 

#双分支：

if [ condition ]
then  
    action1
else
    action2
fi

#多分支：

if [ condition1 ]
then 
    action1
elif [ condition2 ]
then  
    action2 
elif [ condition3 ]
then 
    ... ...
else 
    action 
fi 

```

### 5.1.2 开关语句
```bash
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
*）                             #默认情况，实际是shell通配符
    command1
    command2
    ...
    commandN
    ;;
esac
```

## 5.2 循环语句
&emsp;&emsp;同c语言一样shell循环可用**break**跳出循环，用**continue**继续下一循环。

### 5.2.1  for 循环 
```bash
for 变量 in [取值列表]   
do 
      循环体
done

#或

for ((循环变量初值;循环条件；循环增量))
do 
      循环体
done
```
* 当变量值在列表里，for循环即执行一次所有命令，使用变量名获取列表中的当前取值。命令可为任何有效的shell命令和语句。in列表可以包含命令替换、字符串、文件名和目录。in列表是可选的，如果不用它，for循环使用命令行的位置参数。注意如果是目录则应该使用`for 循环变量 in dir/* `或者 for 循环变量 in `ls dirname`
* Linux shell中，定义的函数变量，默认是全局global的，即使是在函数中定义的变量，也是全局的。所以，递归函数中的直接定义的变量，由于是全局变量变量，导致：下一层调用中修改了某个变量后，返回上一层时，变量的值不会恢复。所以导致运行结果不对。解决办法就是，把（递归函数中的）所有的变量之前加上local定义，表示局部变量，这样运行的结果，和函数执行逻辑，就和C等其他语言类似，结果也就都正确了。



### 5.2.2 While 语句 
```bash
循环变量=初值
while [ 循环条件 ]
do
   循环体
done

#或

while ((C语言写法))
do
   循环体
done
```



### 5.3.3 until 语句 
```bash   
循环变量=初值
until [循环条件]
do
    循环体
done 

#或

unitl（（c语言写法））
do
    循环体
done 
```

### 5.3.4 select 语句
```
select 循环变量 in [取值列表]
do
    循环体
done
```
* select是交互式循环，shell为取值列表中的每一个元素生成一个选项，用户可以进行选择，然后根据选择的值执行相应的操作。
* select是永久循环，必须使用break跳出。

# 6 shell 函数 
```bash
# 函数定义

[function] 函数名() //funciton可省略
{
    ......
    [return val]
} 

#函数调用

函数名 参数1 参数2 ...... 参数n

```

* 在函数中使用参数变量来访问参数


# 7 常用命令
## 7.1 表达式计算命令——`expr`
* **命令**：`expr [--help] [--version]`
&emsp;&emsp;&emsp;`expr EXPRESS`
* **描述**：计算表达式。

  * 许多云算法需要为shell进行转义或引用。如果两个ARG都是数字，则比较是算术的，否则是词典的。模式匹配返回`\( 和 \)`之间匹配的字符串或null;如果未使用`\(和\）`，则返回匹配的字符数或0。只有第一个'\(...\)'会引用返回的值；其余的'\(...\)'只在正则表达式分组时有意义。在正则表达式中，'\+'，'\?'和'\|'分表代表匹配一个或多个，0个或1个以及两端任选其一的意思。
  * 如果EXPRESSION既不为null也不为0，则退出状态为0;如果EXPRESSION为空或0，则退出状态为1;如果EXPRESSION在语法上无效，则退出状态为2;如果发生错误，则退出状态为3。

|EXPRESSION|作用|
|:--|:--|
|ARG1 &#124; ARG2 | 如果ARG1既不是null也不是0，则返回ARG1，否则返回ARG2
|ARG1 & ARG2 | 如果ARG1不为null或0，则返回ARG1，否则返回0
|ARG1 < ARG2|如果ARG1小于ARG2，返回true，否则返回false
|ARG1 <= ARG2|如果ARG1小于等于ARG2，返回true，否则返回false
|ARG1 = ARG2|如果ARG1等于ARG2，返回true，否则返回false
|ARG1 != ARG2|如果ARG1不等于ARG2，返回true，否则返回false
|ARG1 >= ARG2|如果ARG1大于等于ARG2，返回true，否则返回false
|ARG1 > ARG2|如果ARG1大于ARG2，返回true，否则返回false
|ARG1 + ARG2|返回ARG1和ARG2的算术和
|ARG1 - ARG2|返回ARG1和ARG2的算术差
|ARG1 * ARG2|返回ARG1和ARG2的算术乘积
|ARG1 / ARG2|返回ARG1的算术商除以ARG2
|ARG1 ％ ARG2|返回ARG1的算术余数除以ARG2
|var ：REGEXP|执行模式匹配。两端参数会转换为字符格式，且第二个参数被视为正则表达式(**GNU基本正则**)，它默认会隐含前缀"^"。
|match var REGEXP|与var相同
|substr var POS len|返回var字符串中从pos开始，长度最大为len的子串。如果pos或len为负数，0或非数值，则返回空字符串。从1开始计数。
|index var CHARS|CHARSET中任意单个字符在var中最前面的字符位置。如果在var中完全不存在CHARSET中的字符，则返回0。
|len var|返回var的字符长度。
|\+ TOKEN|将TOKEN解析为普通字符串，即使TOKEN是像match或操作符"/"一样的关键字。
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|



## 7.2  标准输出命令
### 7.2.1 标准格式化输出命令——`printf`
* **命令**：printf FORMAT [ARGS]...
&emsp;&emsp;：printf LONG-OPTION
* **描述**：根据指定格式在屏幕上显示指定内容。格式控制字符和C函数printf类似。


|长选项|作用|
|:--|:--|
|--help|帮助
|--version|版本

|转义字符|描述|转义字符|描述|转义字符|描述|
|:--|:--|:--|:--|:--|:--|
|\"|双引号|\\\ |     \\ |\a |   响铃|
|\\b |退格|\c|输出时不自动添加换行符|\e|溢出|\f |换页
|\n| 换行 |\r |回车 |\t |水平制表符 
|\v| 垂直制表符|\NNN|八进制ASCII字符|\xHH| 表示十六进ASII字符|
|\uHHHH|十六进制Unicode字符|\UHHHHHHHH|十六进制Unicode字符|%%|%
|%b|解释ARGS时加上\ |%q|ARGS输出时可用作shell输入 |

|控制字符|描述|控制字符|描述|
|:--|:--|:--|:--|
|%d	|带符号int整数|	%c	|字符
|%i|带符号int整数|%s	|一串字符，遇到\0就停止输出
|%u|无符号int整|%f|	浮点数
|%o|八进制int整数|%e |	科学计数法, 使用小写"e"|
|%x|无符号十六进制int整数, 用小写字母|%E |	科学计数法, 使用大写"E"|
|%X|无符号十六进制int整数, 用大写字母|%g	|使用%e或%f中较短的一个	|
|%n	|参数是一个指向一个整数的指针.指向的是字符数放置的位置	|%G	|使用%E或%f中较短的一个|

|控制字符|描述|控制字符|描述|
|:--|:--|:--|:--|
|-|左对齐|#|十六进制加上前缀0x，八进制加上前缀0
|m.n|m域宽，n小数位数，默认6位|


### 7.2.2 标准输出命令——`echo`
* **命令**：`echo [SHORT-OPTION]... [var]...`
&emsp;&emsp;&emsp;`echo LONG-OPTION`
* **描述**：在屏幕上显示指定内容。
  * 可以指定彩色颜色。 
  * 参数可以加引号也可以不加引号。
  * 参数可以是字符串、变量、命令替换。
  * 默认自动添加换行符。

|短选项|长选项|描述|
|:--|:--|:--|
|-e||允许解释特定的转义字符
|-n||输出时不自动添加换行符
|-E||默认禁用解释特定的转义字符
||--help|帮助
||--version|版本

|转义字符|描述|转义字符|描述|转义字符|描述|
|:--|:--|:--|:--|:--|:--|
|\\\ |\\ |\a |   响铃|\\b |退格
|\c|输出时不自动添加换行符|\e|溢出|\f |换页
|\n| 换行 |\r |回车 |\t |水平制表符 |\v| 垂直制表符 、
|\0NNN|八进制ASCII字符|\xHH| 表示十六进ASII字符|

```bash
echo -e "\e[特殊格式;前景;背景色m"
echo -e "\033[特殊格式;前景;背景色m"
```
|前景色|背景色 |深前景色|颜色|
|:--|:--|:--|:--|
|30|40|90 |黑色|
|31 |41|91 |红色|
|32 |42 |92|绿色|
|33 |43 |93|黄色|
|34 |44 |94|蓝色|
|35 |45| 95|紫红色|
|36 |46 |96|青蓝色|
|37 |47| 97|白色| 

|特殊格式|描述|
|:--|:--|:--|:--|:--|:--|
| 0 |OFF关闭所有显示效果 | 
|1 |高亮显示 |
|4 |显示下划线 |
|5 |闪烁显示 |
|7 |反色显示 
|8 |字符颜色将会与背景颜色相同'


|特殊格式|描述|
|:--|:--|:--|:--|:--|:--|
|\e\[nA |光标上移n行|
|\e\[nB| 光标下移n行|
|\e\[nC |光标右移n行|
|\e\[nD |光标左移n行|
|\e\[y;xH| 设置光标位置
|\e\[2J |清屏
|\e\[K |清除从光标到行尾的内容
|\e\[s| 保存光标位置|
|\e\[u| 恢复光标位置|
|\e\[?25l| 隐藏光标|
|\e\[?25h |显示光标|
## 7.3  条件测试命令——`test`

* **命令**：`test [EXPRESSION]`
&emsp;&emsp;&emsp;`test [OPTIONS]`
* **描述**：和条件测试运算符[]或[[]]的功能一样。

## 7.4 指定状态码命令——`:`、`true`、`false`
* **命令**：`:`
* **描述**：空命令，退出状态为成功。

&emsp;

* **命令**：`true`
* **描述**：返回成功结果，退出状态为成功。

&emsp;

* **命令**：`false`
* **描述**：返回失败结果，退出状态为失败



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
