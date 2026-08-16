---
title: Linux首次登陆与帮助
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 1 基本操作
&emsp;&emsp;图形界面下查看隐藏文件：和windows类似，在资源管理器勾选显示隐藏目录即可。
## 1.1 图形界面
&emsp;&emsp;注销与退出：注销是退出当前用户但并不关机。

&emsp;&emsp;图形界面切换文本模式：`[Ctrl] + [Alt] + [F1]~[F6]`
&emsp;&emsp;文本模式切换图形界面： `startx`
&emsp;&emsp;图形界面切换到终端模式：`CTRL+ALT+T`

&emsp;&emsp;tty：每个tty对应一个文本模式和一个图形界面，[Ctrl] + [Alt]+[F2]~[F6] 每次会建立一个tty。
## 1.2 基本命令
&emsp;&emsp;在Linux当中，默认root的提示字符为#，而一般身份用户的提示字符为 $。

&emsp;&emsp;命令行格式：`$  程序名 [-options] parameters 回车`，命令开始执行，空格要用\转义，命令换行用\，区分大小写。

&emsp;&emsp;（乱码时）更改语言支持：
```shell
$ LANG=目标语言
$ export LC_ALL=目标语言
```
&emsp;&emsp;文件补全或命令补全：TAB键
&emsp;&emsp;终止程序：CTRL+c
&emsp;&emsp;终止程序并产生core文件：CTRL+\
&emsp;&emsp;结束键盘输入：CTRL+d
&emsp;&emsp;向前/后翻屏：shift+ PageUp/PageDown

&emsp;&emsp;显示日期工具：date
&emsp;&emsp;查看日历工具：cal
&emsp;&emsp;简单计算器：bc

&emsp;&emsp;将数据同步写入硬盘中的指令： sync
&emsp;&emsp;惯用的关机指令： shutdown
&emsp;&emsp;重新启动，关机： reboot, halt, poweroff
>**shutdown**
>&emsp;&emsp;shutdown命令安全地将系统关机。shutdown执行它的工作是送信号给init程序﹐要求它改变runlevel。
>&emsp;&emsp;假定没有-h也没有-r参数给shutdown。要想了解在停机〔halt〕或>者重新开机〔reboot〕过程中做了哪些动作﹐你可以在这个文件/etc/inittab里>看到这些runlevels相关的资料。
>
>* Runlevel 0被用来停机〔halt〕。
>* runlevel 6是用来重新激活〔reboot〕系统。
>* runlevel 1则是被用来让系统进入管理工作可以进行的状态﹔这是预设的﹐
>
>>**shutdown 参数说明**
>>
>>* [-t] 在改变到其它runlevel之前﹐告诉init多久以后关机。
>>* [-r] 重启计算器。
>>* [-k] 并不真正关机﹐只是送警告信号给每位登录者〔login〕。
>>* [-h] 关机后关闭电源〔halt〕。
>>* [-n] 不用init﹐而是自己来关机。不鼓励使用这个选项﹐而且该选项所>>>产生的后果往往不总是你所预期得到的。
>>* [-c] cancel current >>process取消目前正在执行的关机程序。所以这个选项当然没有时间参数﹐但是>>可以输入一个用来解释的讯息﹐而这信息将会送到每位使用者。
>>* [-f] 在重启计算器〔reboot〕时忽略fsck。
>>* [-F] 在重启计算器〔reboot〕时强迫fsck。
>>* [-time] 设定关机〔shutdown〕前的时间。**例如**：
>>```shell
>>$ shutdown －h now #立即关机；
>>$ shutdown －h 20：49 #20：49分关机
>>$ shutdown －h ＋10 #10分钟后关机
>>$ shutdown －r now  #立即重启
>>$ shutdown －r ＋10 'string' #10分钟后系统重启并给每个登录用户发通知
>>$ shutdown －k now 'string ' #仅给每个登录用户发通知并不真关机
>>```
>
>**halt**
>&emsp;&emsp;其实halt就是调用shutdown -h。halt执行时﹐杀死应用进程﹐执行sync系统调用﹐文件系统写操作完成后就会停止内核。
>>**参数说明**:
>>
>>* [-n] 防止sync系统调用﹐它用在用fsck修补根分区之后﹐以阻止内核用老版本的超级块〔superblock〕覆盖修补过的超级块。
>>* [-w] 并不是真正的重启或关机﹐只是写wtmp〔/var/log/wtmp〕纪录。
>>* [-d] 不写wtmp纪录〔已包含在选项[-n]中〕。
>>* [-f] 没有调用shutdown而强制关机或重启。
>>* [-i] 关机〔或重启〕前﹐关掉所有的网络接口。
>>* [-p] 该选项为缺省选项。就是关机时调用poweroff。
>>
>**reboot**
>&emsp;&emsp;假定没有-h也没有-r参数给shutdown。要想了解在停机〔halt〕或>者重新开机〔reboot〕过程中做了哪些动作﹐你可以在这个文件/etc/inittab里>看到这些runlevels相关的资料。reboot的工作过程差不多跟halt一样﹐不过它是引发主机重启﹐而halt是关机。它的参数与halt相差不多。
>**init**
>&emsp;&emsp;init是所有进程的祖先﹐它的进程号始终为1﹐所以发送TERM信号给init会终止所有的用户进程>﹑守护进程等。shutdown就是使用这种机制。init定义了8个运行级别(runlevel)，init 0为关机﹐init1为重启。关>于init可以长篇大论﹐这里就不再叙述。另外还有telinit命令可以改变init的运行级别﹐比如﹐telinit -iS可使系统>进入单用户模式﹐并且得不到使用shutdown时的信息和等待时间。
>
>**poweroff**
&emsp;&emsp;关机,在多用户方式下(Runlevel 3)不建议使用.
      
# 2 帮助与信息
&emsp;&emsp;几乎所有的命令都带有`--help`选项，来显示帮助信息。
## 2.1 man
&emsp;&emsp;几乎所有的系统自带命令都在man  page中有详细帮助信息。man文档一般放在`/usr/share/man/类别`，查阅man手册的命令格式：
```shell
$ man n command #查询指定命令的说明，其中n为命令类别,下面的表格

$ man -f [指令或者是数据] #查询指定文件的说明，等效于$whatis [指令或者是数据]，需要建立whatis库：$ mandb 或者 $ makewhatis

$ man -k [指令或者是数据] #查询说明文件包含关键词的命令，等效于apropos [指令或者是数据]
```
|代号| 代表内容|
|:--|--|
|1 |用户在 shell 环境中可以操作的指令或可执行文件
|2 |系统核心可呼叫的函数与工具等
|3 |一些常用的函数(function)与函式库(library)，大部分为 C 的函式库(libc)
|4 |装置文件的说明，通常在/dev 下的文件
|5 |配置文件或者是某些文件的格式
|6 |游戏(games)
|7 |惯例与协议等，例如 Linux 文件系统、网络协议、ASCII code 等等的说明
|8|系统管理员可用的管理指令
|9 |跟 kernel 有关的文件

&emsp;&emsp; man page 的内容大致包括：
|代号 |内容说明|
|:--|:--|
|NAME |简短的指令、数据名称说明
|SYNOPSIS| 简短的指令下达语法(syntax)简介
|DESCRIPTION |较为完整的说明，这部分最好仔细看看！
|OPTIONS| 针对 SYNOPSIS 部分中，有列举的所有可用的选项说明
|COMMANDS |当这个程序(软件)在执行的时候，可以在此程序(软件)中下达的指令
|FILES |这个程序或数据所使用或参考或连结到的某些文件
|SEE ALSO| 可以参考的，跟这个指令或数据有相关的其他说明！
|EXAMPLE |一些可以参考的范例
|Authors |作者
|Copyright |版权

&emsp;&emsp; man page 的支持的操作：
|按键 |进行工作
|:--|--|
|空格键 |向下翻一页
|[Page Down] |向下翻一页
|[Page Up] |向上翻一页
|[Home] |去到第一页
|[End]| 去到最后一页
|/string| 向『下』搜寻 string 这个字符串，如果要搜寻 vbird 的话，就输入 |/vbird|
|?string| 向『上』搜寻 string 这个字符串
|n, N|利用 / 或 ? 来搜寻字符串时，可以用 n 来继续下一个搜寻 (不论是 / 或 ?) ，可以利用 N 来进行『反向』搜寻。举例来说，我以 /vbird 搜寻 vbird 字符串， 那么可以 n 继续往下查询，用 N 往上查询。若以 ?vbird 向上查询 vbird 字符串， 那我可以用 n 继续『向上』查询，用 N 反向查询。
|q |结束这次的 man page

## 2.2 info
&emsp;&emsp;在Linux里面额外提供了一种求助方法info。info与man的用途其实差不多，都是用来查询指令的用法或者是文件的格式。但是与man一口气输出一堆信息不同的是，info将文件数据拆成一个一个的段落，每个段落用自己的页面来撰写， 并在各个页面中还有类似于超链接的东西来跳到各不同的页面中，每个独立的页面称为一个节点(node)。
&emsp;&emsp;info指令的文件默认是放置在/usr/share/info/这个目录，并且以info格式编写。

&emsp;&emsp;info页面格式：
|名称|含义|
|:--|--|
|File|代表资料是来自哪个文件；
|Node|代表目前的这个页面是属于哪个节点。
|Next|下一个节点，你也可以按『N』到下个节点去；
|Up|上一层的节点总揽画面，你也可以按下『U』回到上一层；
|Prev|前一个节点。但由于Top是info.info的第一个节点，所以上面没有前一个节点的信息。

&emsp;&emsp;info支持的操作：
|按键| 进行工作|
|:--|:--|
|空格键 |向下翻一页
|[Page Down] |向下翻一页
|[Page Up] |向上翻一页
|[tab] |在 node 之间移动，有 node 的地方，通常会以 * 显示。
|[Enter] |当光标在 node 上面时，按下 Enter 可以进入该 node 。
|b |移动光标到该 info 画面当中的第一个 node 处
|e |移动光标到该 info 画面当中的最后一个 node 处
|n |前往下一个 node 处
|p |前往上一个 node 处
|u |向上移动一层
|s(/) |在 info page 当中进行搜寻
|h, ? |显示求助选单
|q |结束这次的 info page

## 2.3 其它帮助文件
&emsp;&emsp;在/usr/share/doc中或者开源项目的doc目录中。



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
