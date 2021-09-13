---
title: gcc
tags: 编译, 开发
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《stty manpages》</font>， stty当前版本为8.30，手册更新时间为2019.9。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
# 1 名称
stty——更改和打印终端行设置。
# 2 概要
```bash
stty [-F DEVICE | --file=DEVICE] [SETTING]...
stty [-F DEVICE | --file=DEVICE] [-a|--all]
stty [-F DEVICE | --file=DEVICE] [-g|--save]
```
# 3 描述
打印或更改终端特性。

长选项的强制性参数对于短选项也是强制性的。
|短选项|长选项|描述|
|:--|:--|:--|
|-a|--all|  以人类可读的形式打印所有当前设置。|
|-g |--save| 以 stty 可读的形式打印所有当前设置。|
| -F <u>DEVICE</u>| --file=<u>DEVICE</u>|打开并使用指定的 <u>DEVICE</u> 而不是标准输入 stdin。|
||--help| 显示此帮助并退出。|
||--version|输出版本信息并退出。|

在 <u>SETTING</u> 之前的可选的 - 表示否定。 \* 标记非 POSIX 设置。 底层系统定义了哪些设置可用。

## 3.1 特殊字符
|非POSIX设置|特殊字符|描述|
|:--|:--|:--|
|\*| discard CHAR|CHAR 将切换丢弃输出。|
| |eof CHAR| CHAR 将发送文件结尾（终止输入）。|
| |eol CHAR|CHAR将结束行。|
|\*| eol2 CHAR| 用于结束行的备用 CHAR。|
| | erase CHAR| CHAR 将删除最后输入的字符。|
| |intr CHAR|   CHAR 会发送中断信号。|
||kill CHAR| CHAR 将擦除当前行。|
| \*| lnext CHAR|  CHAR 将输入引用的下一个字符。|
||quit CHAR|  CHAR 将发送退出信号。|
|\* |rprnt CHAR|CHAR 将重绘当前行。|
| |start CHAR|CHAR 将在停止输出后重新启动输出。|
||stop CHAR|CHAR 将停止输出。|
||susp CHAR|    CHAR 将发送终端停止信号。|
|\*| swtch CHAR|  CHAR 将切换到不同的shell层。|
|\*| werase CHAR|  CHAR 将删除最后输入的单词。|
## 3.2 特殊设置
|非POSIX设置|特殊设置|描述|
|:--|:--|:--|
||N|将输入和输出速度设置为 N 波特。|
|\* |cols N|  告诉内核终端有 N 列。|
|\* |columns N|与 cols N 相同。|
|\* |\[-]drain|  在应用设置之前等待传输（默认开启）。|
| |ispeed N|将输入速度设置为 N。|
| \* |line N| 使用线规 N。|
||min N -icanon|为完整读取，设置最少 N 个字符。|
| |ospeed N| 将输出速度设置为 N。|
|\* |rows N|告诉内核终端有 N 行。|
| * |size|  根据内核打印行数和列数。|
| |speed|打印终端速度。|
| |time N -icanon|  将读取超时设置为十分之一秒 。|
## 3.3 控制设置
|非POSIX设置|控制设置|描述|
|:--|:--|:--|
||\[-]clocal| 禁用调制解调器控制信号|
||\[-]cread|   允许接收输入|
|\* | \[-]crtscts|启用 RTS/CTS 握手|
| | csN|将字符大小设置为 N 位，N 在 \[5..8]中|
||\[-]cstopb|每个字符使用两个停止位（加“-”表示一个停止位）|
|| \[-]hup|当最后一个进程关闭 tty 时发送挂断信号|
||\[-]hupcl|     与 \[-]hup 相同|
||\[-]parenb|在输出中生成奇偶校验位并在输入中期望奇偶校验位|
| |\[-]parodd|设置奇校验（带“-”为偶校验）|
|\*| \[-]cmspar|使用“stick”（标记/空格）奇偶校验 |

## 3.4 输入设置
|非POSIX设置|输入设置|描述|
|:--|:--|:--|
|| \[-]brkint|打断导致一个中断信号。|
||\[-]icrnl|将回车转换为换行符。|
||\[-]ignbrk| 忽略中断字符。|
|| \[-]igncr| 忽略回车。|
||\[-]ignpar|忽略具有奇偶校验错误的字符。|
|* | \[-]imaxbel|   发出哔哔声，不要刷新字符上的完整输入缓冲区。|
||\[-]inlcr|将换行符转换为回车。|
||\[-]inpck|启用输入奇偶校验。|
||\[-]istrip|   清除输入字符的高（第 8）位。|
|* | \[-]iutf8| 假设输入字符是 UTF-8 编码的。|
|* |  \[-]iuclc|将大写字符转换为小写。|
| * |  \[-]ixany|让任何字符重新开始输出，而不仅仅是开始字符。|
||\[-]ixoff| 启用发送 开始/停止 字符。|
||\[-]ixon|  启用 XON/XOFF 流量控制。|
||\[-]parmrk| 标记奇偶校验错误（具有 255-0 个字符的序列）。|
||\[-]tandem|与 \[-]ixoff 相同。 |
## 3.5 输出设置
|非POSIX设置|输出设置|描述|
|:--|:--|:--|
|\* | bsN | 退格延迟风格，N 在 \[0..1]中。|
|\* |crN|回车延迟风格，N 在 \[0..3]中。|
|\* |ffN |  换页延迟风格，N 在 \[0..1]中。|
|\* |nlN | 换行延迟风格，N 在 \[0..1]中。|
|\* | \[-]ocrnl|将回车转换为换行符。|
|\* |\[-]ofdel|使用删除字符代替 NUL 字符进行填充。|
|\* |\[-]ofill|使用填充（填充）字符代替延迟时间。|
|\* |\[-]olcuc|将小写字符转换为大写。|
|\* | \[-]onlcr|将换行符转换为回车换行符。|
|\* |\[-]onlret|换行符表现为回车。|
|\* | \[-]onocr|不要在第一列打印回车。|
|  |\[-]opost|后处理输出。|
|\* | tabN |水平制表符延迟风格，N 在 \[0..3]中。|
|\* |tabs|与tab0一样。|
|\* |-tabs|与tab3一样。|
|\* |vtN| 垂直制表符延迟风格，N 在 \[0..3]中。|

## 3.5 本地设置
|非POSIX设置|本地设置|描述|
|:--|:--|:--|
|  |\[-]crterase| 回显擦除字符，就像退格-空格-退格。|
|\* |crtkill|通过遵守 echoprt 和 echoe 设置kill所有行。|
|\* |-crtkill|通过遵守 echoctl 和 echok 设置kill所有行。|
|\* |\[-]ctlecho| 以帽子表示法（'^c'）回显控制字符。|
|  |\[-]echo| 回显输入字符。|
|\* |\[-]echoctl|与 \[-]ctlecho 相同。|
|  |\[-]echoe|与 \[-]crterase 相同。|
|  |\[-]echok|在kill字符后回显换行符。|
|\* |\[-]echoke|与 \[-]crtkill 相同。|
|  |\[-]echonl|即使不回显其他字符，也回显换行符。|
|\* |\[-]echoprt|向后会先已擦除字符，在 '\\' 和 '/' 之间。 |
|\* |\[-]extproc|启用“LINEMODE”； 对高延迟链接有用。|
|\* |\[-]flusho|丢弃输出。|
|  |\[-]icanon| 启用特殊字符：erase、kill、werase、rprnt。|
|  |\[-]iexten|启用非 POSIX 特殊字符|
|  |\[-]isig|启用 interrupt、quit和suspend 特殊字符。|
|  |\[-]noflsh|中断后禁用冲刷并停止特殊字符。| 
|\* |\[-]prterase|与 \[-] echort 相同。|
|\* |\[-]tostop|停止尝试写入终端的后台作业。|
|\* |\[-]xcase|与 icanon一起使用，用 '\\' 转义大写字符|


## 3.6 组合设置
|非POSIX设置|组合设置|描述|
|:--|:--|:--|
|\* |\[-]LCASE|与\[-]lcase相同。|
||cbreak| 与-icanon相同。|
||-cbreak|与icanon相同。|
||cooked| 与 brkint ignpar istrip icrnl ixon opost isig icanon, eof 和 eol 字符相同到它们的默认值。|
||-cooked|与 raw相同。|
||crt|与 echoe echoctl echoke相同。|
||dec|与 echoe echoctl echoke -ixany intr ^c erase 0177 kill ^u相同。|
|\*| \[-]decctlq|与 \[-]ixany相同。|
||ek |erase和kill字符到它们的默认值。|
||evenp |与  parenb -parodd cs7相同。|
||-evenp|与 -parenb cs8相同。|
|\*| \[-]lcase|与 xcase iuclc olcuc相同。|
||litout|与 -parenb -istrip -opost cs8相同。|
||-litout|与parenb istrip opost cs7相同。|
||nl |与   -icrnl -onlcr相同。|
||-nl |与 icrnl -inlcr -igncr onlcr -ocrnl -onlret相同。|
||oddp|与   parenb parodd cs7相同。|
||-oddp |与  -parenb cs8相同。|
||\[-]parity|与 \[-]evenp相同。|
||pass8 |与  -parenb -istrip cs8相同。|
||-pass8| 与 parenb istrip cs7相同。|
||raw |与 -ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr -icrnl -ixon -ixoff -icanon -opost -isig -iuclc -ixany -imaxbel -xcase min 1 time 0相同。|
||-raw|与cooked相同。|
||sane |与cread -ignbrk brkint -inlcr -igncr icrnl icanon iexten echo echoe echok -echonl -noflsh -ixoff -iutf8 -iuclc -ixany imaxbel -xcase -olcuc -ocrnl opost -ofill onlcr -onocr -onlret nl0 cr0 tab0 bs0 vt0 ff0 isig -tostop -ofdel  -echoprt  echoctl  echoke  -extproc -flusho相同，所有特殊字符为其默认值。|

处理连接到标准输入的 tty 行。 不带参数，打印波特率、行规则和与 stty sane 的偏差。在设置中，CHAR 是按字面意思理解的，或者像 ^c、0x37、0177 或 127 那样编码； 特殊值 ^- 或 undef 用于禁用特殊字符。

# 4 作者
David MacKenzie。
# 5 报告错误
GNU coreutils 在线帮助：<https://www.gnu.org/software/coreutils/>
报告 stty 翻译错误：<https://translationproject.org/team/> 

# 6 版权
版权所有 © 2018 自由软件基金会, Inc. 许可证 GPLv3+：GNU GPL 版本 3 或更高版本 <https://gnu.org/licenses/gpl.html>。

这是免费软件：您可以自由更改和重新分发它。 在法律允许的范围内，不提供任何保证。
# 7 参见
完整文档位于：<https://www.gnu.org/software/coreutils/stty> 或通过本地获取：info '(coreutils) stty invocation'。

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《stty manpages》</font>， stty当前版本为8.30，手册更新时间为2019.9。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
