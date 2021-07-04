---
title:  stty
tags: 用户命令
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
||min N -icanon|为完整读取设置最少 N 个字符。|
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
|* | \[-]imaxbel|   发出哔哔声，不要刷新字符上的完整输入缓冲区|
||\[-]inlcr|
||\[-]inpck|
||\[-]istrip|
|* | \[-]iutf8|
|* |  \[-]iuclc|
| * |  \[-]ixany|
||\[-]ixoff|
||\[-]ixon|
||\[-]parmrk|
||\[-]tandem|
## 3.5 输出设置

## 3.5 本地设置

## 3.6 组合设置

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《stty manpages》</font>， stty当前版本为8.30，手册更新时间为2019.9。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
