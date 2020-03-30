---
title: Linux文本文件查看与编辑
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 1 查看文件内容
## 1.1 查看未压缩文件
<style>table{word-break:initial;}</style>
### 1.1.1 `cat`
**命令**：cat [OPTION]... [FILE]...
**描述**：顺序将FILE输出到标准输出。没有FILE或FILE为-则读取标准输入。
**版本**：8.25
|短选项|长选项|描述|
|:---|:--|:--|
|-A|--show-all|等效于 -vET
|-b|--number-nonblank|列出行号，仅针对非空白行做行号显示，覆盖-n选项
|-e ||等效于 -vE
|-E|--show-ends|在每一行后添加\$
|-n|--number|显示行号
|-s|--squeeze-blank|当遇到有连续两行以上的空白行，就代换为一行的空白行
|-t||等效于 -vT
|-T|--show-tabs|以^I显示TAB字符
|-u||忽略该选项
|-v|--show-nonprinting|运用 ^ 和 M- 标记，除了 LFD 和 TAB 之外|
||--help|帮助
||--version|显示版本
|**更多信息**||<http://linux.51yip.com/search/> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;||<http://man.linuxde.net/>|


### 1.1.2 `tac`
**命令**：tac [OPTION]... [FILE]...
**描述**：按行逆序将FILE输出到标准输出。 没有FILE或FILE为-则读取标准输入。
**版本**：8.25
|短选项|长选项|描述|
|:--|:--|:--|
| -b|--before|将末尾的换行符显示在开始
| -r|--regex|将换行符解析为正则表达式|
| -s STRING|--separator=STRING|使用STRING代替\n作为换行符|
||--help|帮助
||--version|显示版本
|**更多信息**||<http://linux.51yip.com/search/> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;||<http://man.linuxde.net/>|

### 1.1.3 `nl`
**命令**： nl [OPTION]... [FILE]...
**描述**：将FILE输出到标准输出，并加上行号。
|常用选项|作用|
|:----|:--|
|-b STYLE|按照STYLE显示行号，a全部显示行号，t空行不显示行号，n不显示行号，pBRE含有正则表达式BRE匹配的行显示行号|
|-n FORMAT|根据FORMAT设置行号样式，ln向左对齐，前面不加零补位，rn 向右对齐，前面不加零补位，rz 向右对齐，前面加零补位
|-w NUMBER|设置行号的位数
|**更多信息**||<http://linux.51yip.com/search/nl> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||<http://man.linuxde.net/nl>|

### 1.1.4 `more`
**命令**：more [options] file...
**描述**：将FILE按页输出到标准输出。
|常用选项|作用|
|:----|:--|
|-d |提示使用者，在画面下方显示 [Press space to continue, 'q' to quit.] ，如果使用者按错键，则会显示 [Press 'h' for instructions.] 而不是 '哔' 声
|-s  | 当遇到有连续两行以上的空白行，就代换为一行的空白行 
|-number|一页可显示的行数 
| +number|从第 num 行开始显示
| +/string|在每个文档显示前搜寻该字串（pattern），然后从该字串之后开始显示
|**更多信息**|<http://linux.51yip.com/search/more> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/more>|

|常用交互命令|作用|
|:----|:--   |          
|h or ? | 显示帮助
| SPACE |显示接下来的k行，默认一屏 
| z|显示接下来的k，默认一屏，若有参数则按照参数
| RETURN| 向下滚动k行，默认1行，若有参数则按照参数
|d or ^D | 滚动k行，默认为当前滚动值， 初始为11，若有参数则按照参数
|q or Q |退出
| s  | 向后跳过接下来的k行，默认为1 
| f   | 向后跳过接下来的k屏，默认为1  
|b or ^B  | 向前跳过k屏，默认为1 ，不能用于管道
| ' |  去上一次搜索开始的地方
|=   | 显示当前行号
|/pattern |查找第k个与正则表达式匹配的内容，默认为1
| n   |搜索上次正则表达式的第k次出现。 默认为1。
|!command |调用Shell，并执行命令
|:!command|调用Shell，并执行命令
| v |根据环境变量 VISUAL或EDITOR启动编辑器，如果没有设置则启用vi
| ^L | 重绘屏幕。
|:n  | 跳转到第k行，默认为1,
| :p |  跳转到第k个前面的文件，默认为1,
|:f  | 显示当前文件名和行号
| .  |   重复上一个命令
|**更多信息**|<http://linux.51yip.com/search/more> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/more>|

### 1.1.5 `less`
**命令**：less [options] file...
**描述**：将FILE按页输出到标准输出。
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/less> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/less>|


|常用交互命令|作用|
|:----|:--|  
|  空格键 |向下翻动一页；
|  [pagedown]|向下翻动一页；
|  [pageup] |向上翻动一页；
|  /字符串 |向下搜寻『字符串』的功能；
|  ?字符串 |向上搜寻『字符串』的功能；
|  n |重复前一个搜寻 (与 / 或 ? 有关！)
|  N |反向的重复前一个搜寻 (与 / 或 ? 有关！)
|  g |前进到这个资料的第一行去；
|  G |前进到这个数据的最后一行去 (注意大小写)；
|  q |离开 less 这个程序；
|**更多信息**|<http://linux.51yip.com/search/less> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/less>|

### 1.1.6 `head`
**命令**：head [OPTION]... [FILE]...
**描述**：默认将每个FILE的前10行输出到标准输出。
|常用选项|作用|
|:----|:--|
|-c [-]NUM|输出每个文件的前NUM字节，如果带有-，则输出除最后NUM字节以外的所有内容，NUM可以有前缀单位，如KB，GB
|-n [-]NUM|输出每个文件的前NUM行，如果带有-，则输出除最后NUM行以外的所有内容，NUM可以有前缀单位，如K，G
|**更多信息**|<http://linux.51yip.com/search/head> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/head>|

### 1.1.7 `tail`
**命令**：tail [OPTION]... [FILE]...
**描述**：默认将每个FILE的后10行输出到标准输出。
|常用选项|作用|
|:----|:--
|-c [+]NUM|输出每个文件的后NUM字节，如果带有+，则输出除前NUM字节以外的所有内容，NUM可以有前缀单位，如KB
|-n [+]NUM|输出每个文件的后NUM行，如果带有+，则输出除前NUM行以外的所有内容，NUM可以有前缀单位，如K
|--pid=PID|与-f联用，当指定的进程号的进程终止后，自动退出tail命令
|**更多信息**|<http://linux.51yip.com/search/tail> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/tail>| 




## 1.2 查看压缩文件

### 1.2.1 查看压缩文件——`zcat`、`bzcat`、`xzcat`
**命令**：zcat [ -fhLV ] [ name ...  ]

**描述**：等同于`压缩工具 -d -c`。不解压文件查看文件内容。
|常用选项|作用|备注|
|:--|:--|:--|
|-c|将压缩的数据输出到屏幕上
|-d|解压缩；默认选项
|-f| 目标存在时覆盖，或原文件是符号链接|
|-k|不删除原文件|
|-q|不显示警告信息
|-t|检验一个压缩文件的一致
|-v|压缩时显示相关信息
|-z|压缩|
|-#|#为压缩等级，1-9，默认6
|-l|查看压缩文件的压缩比列等信息|bzcat不支持|
|-r|递归压缩或解压目录中的每个文件，但不是打包|仅zcat支持|
|-s|使用小内存|仅bzcat支持|
|-S SUF|添加后缀SUF|仅zcat支持|
|-e|极限模式|仅xzcat支持|
|**更多信息**|<http://linux.51yip.com/search/zcat> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/zcat>|
||<http://linux.51yip.com/search/bzcat>|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/bzcat>|
||<http://linux.51yip.com/search/xzcat> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/xzcat>|
||man 手册|

### 1.2.2 查看压缩文件——`zmore`、`bzmore` `xzmore`
**命令**：zmore  [file...]
&emsp;&emsp;&emsp;bzmore  [file...]
&emsp;&emsp;&emsp;xzmore  [file...]
**描述**：查看压缩文件，但只能向后查看。
|常用选项|无|
|:--|:--
|**更多信息**|<http://linux.51yip.com/search/zmore> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/zmore>|
||<http://linux.51yip.com/search/bzmore>|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/bzmore>|
||<http://linux.51yip.com/search/xzmore> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/xzmore>|
||man 手册|

### 1.2.3 查看压缩文件——`zless`、`bzless` `xzless`
**命令**：zless  [file...]
&emsp;&emsp;&emsp;bzless  [file...]
&emsp;&emsp;&emsp;xzless  [file...]
**描述**：查看压缩文件，可以向前向后查看。
|常用选项|无|
|:--|:--
|**更多信息**|<http://linux.51yip.com/search/zless> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/zless>|
||<http://linux.51yip.com/search/bzless>|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/bzless>|
||<http://linux.51yip.com/search/xzless> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/xzless>|
||man 手册|








# 2 查看两个文件的差异
## 2.1 查看两个普通文件的差异——`diff`
**命令**：diff [-bBi]  file1 file2
**描述**：将file1与file2按行比较，以查看两个文件的差异。
|常用选项|作用|
|:--|:--
|-b|忽略空白的差异。
|-B|忽略空白行的差异|
|-i|忽略大小写的差异
|**更多信息**|<http://linux.51yip.com/search/diff> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/diff>|

## 2.2 查看两个压缩文件差异——`zdiff`、`bzdiff`、`xzdiff`
**命令**：zdiff [ diff_options ] file1 [ file2 ]
&emsp;&emsp;&emsp;bzdiff [ diff_options ] file1 [ file2 ]
&emsp;&emsp;&emsp;xzdiff [ diff_options ] file1 [ file2 ]
**描述**：*Zdiff用于在压缩文件上调用diff程序。指定的所有选项都直接传递给diff。如果只指定file1，则将其与file1.gz的未压缩内容进行比较。如果指定了两个文件，则它们的内容（如果需要则先解压）会被传送到diff。输入文件不会被修改。 diff的退出状态将被保留。
|常用选项|无|
|:--|:--
|**更多信息**|<http://linux.51yip.com/search/diff> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/diff>|



# 3 比较两个文件是否相同
## 3.1 比较两个普通文件是否相同——`cmp`

**命令**：cmp [-s]  file1 file2
**描述**：将file1与file2按字节比较，以查看两个文件的差异。
|常用选项|作用|
|:--|:--
|-s|将所有的不同的字节都列出来。因为cmp预设仅会输出第一个发现的不同点。
|**更多信息**|<http://linux.51yip.com/search/cmp> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/cmp>|


## 3.2 比较两个压缩文件是否相同——`zcmp`、`bzcmp`、`xzcmp`
**命令**：zcmp [ cmp_options ] file1 [ file2 ]
&emsp;&emsp;&emsp;bzcmp [ cmp_options ] file1 [ file2 ]
&emsp;&emsp;&emsp;xzcmp [ cmp_options ] file1 [ file2 ]
**描述**：*cmp用于在压缩文件上调用cmp程序。指定的所有选项都直接传递给cmp。如果只指定file1，则将其与file1.gz的未压缩内容进行比较。如果指定了两个文件，则它们的内容（如果需要则先解压）会被传送到cmp。输入文件不会被修改。 cmp的退出状态将被保留。只显示第一个不同字节
|常用选项|作用|
|:--|:--
|-b|显示不同的字节|
|-i skip|输入的文件都跳过前skip字节
|-i skip1:skip2|第一个文件跳过skip1字节，第二个文件跳过skip2字节
|-l|显示所有不同字节的，行号和ascii
|-n limt|只比较前limt字节
|**更多信息**|<http://linux.51yip.com/search/cmp> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/cmp>|


# 4 搜索文件内容
## 4.1 搜索普通文件内容——`grep`、`fgrep`、`egrep`
**命令**：grep [-acinv] [-e] [-EF][ --color=auto] 'PATTERN' filename 
&emsp;&emsp;&emsp;fgrep [-acinv] [-e] [-E][ --color=auto] 'PATTERN' filename 
&emsp;&emsp;&emsp;egrep [-acinv] [-e] [-F][ --color=auto] 'PATTERN' filename 
**描述**：取出含有PATTERN的行。

|常用选项|	作用|
|:----|:--|
|-a|	将 binary 文件以 text 文件的方式搜寻数据
|-c	|计算找到 '搜寻字符串' 的次数
|-i	|忽略大小写的不同，所以大小写视为相同
|-n|	顺便输出行号
|-v	|反向选择，亦即显示出没有 '搜寻字符串' 内容的行！
|-e|	使用正则表达式
|-E	|使用扩展正则表达式，等同于egrep
|-F	|将PATTER解释为字符串，而不是正则表达式，等同于fgrep
|更多信息|	<<http://linux.51yip.com/search/grep> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/grep>
||<<http://linux.51yip.com/search/fgrep> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/fgrep>
||<<http://linux.51yip.com/search/egrep> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/egrep>
## 4.2 搜索压缩文件内容
### 4.2.1 `zgrep`、`bzgrep`、`xzgrep`
**命令**：zgrep [ grep_options ] [ -e ] pattern filename...
&emsp;&emsp;&emsp;bzgrep [ grep_options ] [ -e ] pattern filename...
&emsp;&emsp;&emsp;xzgrep [ grep_options ] [ -e ] pattern filename...
**描述**：\*Zgrep用于在通过gzip压缩的文件上调用grep。选项 -[drRzZ] | --di\* | --exc* | --inc\* | --rec* | --nu\*导致 zgrep以错误代码终止。所有其他选项直接传递给grep。如果没有输入文件就读取标准输入。
|常用选项|无|
|:--|:--
|**更多信息**|<http://linux.51yip.com/search/grep> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/grep>|

### 4.2.2 `zegrep`、`bzegrep`、`xzegrep`
**命令**：zegrep [ grep_options ] pattern filename...
&emsp;&emsp;&emsp;bzegrep [ grep_options ] pattern filename...
&emsp;&emsp;&emsp;xzegrep [ grep_options ] pattern filename...
**描述**：*Zegrep等同于*zgrep -E，使用扩展的正则表达式对文本进行搜索。
|常用选项|无|
|:--|:--
|**更多信息**|<http://linux.51yip.com/search/egrep> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/egrep>|

### 4.2.3 `zfgrep`、`bzfgrep`、`xzfgrep`
**命令**：zfgrep [ grep_options ] pattern filename...
&emsp;&emsp;&emsp;bzfgrep [ grep_options ] pattern filename...
&emsp;&emsp;&emsp;xzfgrep [ grep_options ] pattern filename...
**描述**：*Zfgrep等同于*zgrep -F，利用固定的字符串来对文本进行搜索，但不支持正则表达式的引用，所以此命令的执行速度也最快。
|常用选项|无|
|:--|:--
|**更多信息**|<http://linux.51yip.com/search/fgrep> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/fgrep>|




# 5 文本处理工具——`sed`
**命令**：sed [options] ' [n1[,n2]]动作' file(s)
&emsp;&emsp;&emsp;sed [options] -f scriptfile file(s)
**描述**：文本处理。
|常用选项|作用|
|:--|:--
|-n| 使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到屏幕上。但如果加上 -n 参数后，则只有经过 sed 特殊处理的那一行(或者动作)才会被列出来。
|-e|直接在命令行模式上进行 sed 的动作编辑；
|-f| -f  scriptfile 可以执行  scriptfile 内的 sed 动作；
|-r|sed 的动作支持的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
|-i|直接修改文件内容，而不是由屏幕输出。
|n1, n2 |不一定会存在，表示要处理的文本范围。
|动作||
|a|新增， a 的后面可以接字符串，而这些字符串会在新的一行出现(目前的下一行)～
|c|替换， c 的后面可以接字符串，这些字符串可以取代 n1,n2 之间的行！
|d|删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
|i|插入， i 的后面可以接字符串，而这些字符串会在新的一行出现(目前的上一行)；
|p|打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运作～
|s| 替换，可以直接进行取代的工作！通常这个s的动作可以搭配正规表示法！例如 1,20s/old/new/g 
|**更多信息**|<http://linux.51yip.com/search/sed> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/sed>|



# 6 文本处理工具——`awk`

**命令**：awk '条件1{动作1 } 条件2{动作2}...' filename
**描述**：文本处理。awk主要是处理每一行的字段内的数据，而默认的字段的分隔符为 "空格键" 或 "[tab]键"。若动作需要多个命令辅助时可用分号隔开。awk支持逻辑运算符<、<=、=、!=、>、>=。
|变量名称|作用|
|:----|:--|
|\$0|整行数据|
|\$n|n为大于0的整数，代表当前行的第n个字段的内容|、
|NF |每一行 (\$0) 拥有的字段总数|
|NR |目前 awk 所处理的是第几行数据
|FS| 目前的分隔字符，默认是空格键
|**更多信息**|<http://linux.51yip.com/search/awk> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/awk>|

# 7 文件打印准备——`pr`
**命令**：pr [OPTION]... [FILE]...
**描述**：pr命令用来将文本文件转换成适合打印的格式，它可以把较大的文件分割成多个页面进行打印，并为每个页面添加标题。
|常用选项|作用|
|:--|:--
|-h<标题>|为页指定标题；
|-l<行数>|指定每页的行数。
|**更多信息**|<http://linux.51yip.com/search/pr> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/pr>|


# 8 格式化打印——`printf`
**命令**：printf '打印格式 '  实际内容
**描述**：格式化输出。
|格式参数|作用|
|:--|:--
|\a| 警告声音输出
|\b| 退格键(backspace)
|\f |清除屏幕 (form feed)
|\n| 输出新的一行
|\r| 亦即 Enter 按键
|\t |水平的 [tab] 按键
|\v |垂直的 [tab] 按键
|\xNN |NN 为两位数的数字，可以转换数字成为字符。
|%ns| 那个 n 是数字， s 代表 string ，亦即多少个字符；
|%ni| 那个 n 是数字， i 代表 integer ，亦即多少整数字数；
|%N.nf| 那个 n 与 N 都是数字， f 代表 floating (浮点)，如果有小数字数，假设我共要十个位数，但小数点有两位，即为 %10.2f 啰！
|**更多信息**|<http://linux.51yip.com/search/printf> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/printf>|

# 9 选取行中指定内容——`cut`
**命令**：cut -d [-s] 'delimiter' -f 范围 
&emsp;&emsp;&emsp;cut -c|b 范围 
**描述**：从输入数据的每一行中取出指定的数据。

|常用选项|	作用|
|:--|:--|
|-d	|设置用来将输入数据截断为几段的单个字符
|-s	|不显示不包含分隔符的行
|-f	|以段为单位
|-c	|以字符为单位
|-b	|以字节为单位
|选取范围|	
|n1,n2,...|	显示n1,n2,...字节/字符/段
|[n]-[m]|	显示n-m字节/字符/段,n缺省为1，m缺省为最后一行行号
|更多信息|	<<http://linux.51yip.com/search/cut> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/cut>



# 10 排序——`sort`
**命令**：sort [-fbMnrtuk] [file or stdin] 
**描述**：排序。

|常用选项|	作用|
|:----|:--|
|-f|	忽略大小写的差异，例如 A 与 a 视为编码相同；
|-b|	忽略最前面的空格符部分；
|-M|	以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
|-n	|使用『纯数字』进行排序(默认是以文字型态来排序的)；
|-r|	反向排序；
|-u	|就是 uniq ，相同的数据中，仅出现一行代表；
|-t|	分隔符，预设是用 [tab] 键来分隔；
|-k|	用哪个区间 (field) 来进行排序
|-o|	输入文件
|更多信息|<<http://linux.51yip.com/search/sort> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/sort>

# 11 过滤重复行——`uniq`
**命令**：uniq [-cdiu] [-fsw n] input [output] 
**描述**：用于报告或忽略文件中的连续的重复行。

|常用选项|	作用|
|:----|:--|
|-c|	显示每行重复次数
|-d|	只显示重复行
|-f|	跳过哪几段
|-i	|忽略大小写
|-s	|不比较每行的前n个字符
|-u	|只显示唯一行
|-w	|最多比较N个字符
|更多信息|	<<http://linux.51yip.com/search/uniq> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/uniq>

# 12 统计文件的字数——`wc`
**命令**：wc [-cmlLw] file... 
**描述**：计算文件行数、词数、字节数。

|常用选项|	作用|
|:----|:--|
|-c	|显示字节数
|-m	|显示字符数
|-l	|显示行数
|-L	|显示最大宽度
|-w	|显示词数
|更多信息|	<<http://linux.51yip.com/search/wc> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/wc>

#13 字符替换、去重、删除——`tr`
**命令**：tr [-cdst] SET1 [SET2] 
**描述**：对来自标准输入的字符进行替换、去重、删除。

|常用选项|	作用|
|:----|:--|
|-c|	使用SET1的补集
|-d|	删除 SET1 ；
|-s|	把连续重复的字符以SET1中的单个字符表示
|-t|	先删除第一字符集较第二字符集多出的字符。
|更多信息|	<http://linux.51yip.com/search/tr> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<http://man.linuxde.net/tr>

# 14 过滤控制字符——`col`
**命令**：col [-bfhpx] [-l num] 
**描述**：过滤控制字符。

|常用选项|	作用|
|:----|:--|
|-b| 不输出任何退格符,在每列的位置上只打印最后写的那个字符.
|-h|	输出TAB符而不是空格
|-x	输出空格来代替TAB符
|-l| num	设置缓冲行数
|更多信息|	<http://linux.51yip.com/search/col> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<http://man.linuxde.net/col>


# 15 合并相同行——`join`
**命令**：join [OPTION]... FILE1 FILE2 
**描述**：将文件1和文件2中指定数据相同的行合并为一行，且将相同字段放在第一个。

|常用选项|	作用|
|:----|:--|
|-t delimiter|	设置分隔字符，默认为空格
|-i|	忽略大小写的差异；
|-1 FIELD|	设置第一个文件用于比较的字段，默认1
|-2 FIELD|	设置第二个文件用于比较的字段，默认1
|更多信息|	<http://linux.51yip.com/search/join> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|	<http://man.linuxde.net/join>

# 16 合并对应行——`paste`
**命令**：paste [-ds ] FILE1 FILE2 
**描述**：将两行贴在一起，且中间以 TAB 键隔开。

|常用选项|	作用|
|:----|:--|
|-d|	设置分隔符，默认为TAB
|-s|	粘贴一个文件而不是对应行并删除换行符
|更多信息|	<http://linux.51yip.com/search/paste> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|	<http://man.linuxde.net/paste>

# 17 将TAB转为空格——`expand`
**命令**：expand [-t n] FILE... 
**描述**：将TAB转为空格。

|常用选项|	作用|
|:----|:--|
|-t n|	一个tab转为n个空格，缺省为8
|更多信息|	<http://linux.51yip.com/search/expand> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|	<http://man.linuxde.net/expand>







------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
