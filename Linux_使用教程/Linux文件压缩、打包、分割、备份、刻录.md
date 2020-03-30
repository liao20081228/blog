---
title: Linux文件压缩、打包、分割、备份、刻录
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



# 1 压缩
## 1.1 压缩的概念
目前计算机系统中都是使用bytes单位来计量的！而计算机最小的计量单位应该是 bits 才对啊。但是如果今天我们只是记忆一个数字1， 在最右边占据 1 个 bit ，而其他的7bits将会自动的被填上0，而一些聪明的计算机工程师就利用一些复杂的计算方式，将这些没有使用到的空间丢出来，以让文件占用的空间变小！这就是压缩的技术啦！

另外一种压缩技术也很有趣，他是将重复的数据进行统计记录的。举例来说，如果你的数据为111....，共有 100 个 1 时，那么压缩技术会记录为“100 个 1”而不是真的有 100 个 1 的位存在！这样也能够精简文件记录的容量呢！

|常见格式|压缩程序|解压程序|备注|
|:--:|:--:|:--:|:--:|
|*.Z |compress |uncompress|被gzip取代|
|*.zip|zip |unzip|
|*.gz |gzip |gunzip|兼容\*.Z和\*.zip
|*.bz2 |bzip2|bunzip
|*.xz |xz |unxz
|*.tar| tar |tar
|*.tar.\* |tar+压缩|

## 1.2 压缩工具——`zip`、`gzip`、`bzip2`、`xz`
**命令**：gzip [ -acdfhklLnNrqtvV19 ] [--rsyncable] [-S suffix] [filenames...]
&emsp;&emsp;&emsp;bzip2 [ -cdfkhqstvzVL123456789 ] [ filenames ...]
&emsp;&emsp;&emsp;xz [ -cedfkhqtvzV123456789 ] [ filenames ...]
**描述**：压缩或者解压缩文件，查看压缩文件信息。默认分别自动添加.gz，.bz2，.xz后缀。

|常用选项|作用|备注|
|:--|:--|:--:|
|-c|将压缩的数据输出到屏幕上
|-d|解压缩；
|-f| 目标存在时覆盖，或原文件是符号链接|
|-k|不删除原文件|
|-q|不显示警告信息
|-t|检验一个压缩文件的一致
|-v|压缩时显示相关信息
|-#|#为压缩等级，1-9，默认6
|-l|查看压缩文件的压缩比列等信息|bzip2不支持|
|-r|递归压缩或解压目录中的每个文件，但不是打包|仅gzip支持|
|-z|压缩|gzip不支持|
|-s|使用小内存|仅bzip2支持|
|-S SUF|添加后缀SUF|仅gzip支持|
|-e|极限模式|仅xz支持|
|**更多信息**|<http://linux.51yip.com/search/gzip> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/gzip>|
||<http://linux.51yip.com/search/bzip2>|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/bzip2>|
||<http://linux.51yip.com/search/xz> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/xz>|
||man 手册|

## 1.2 解压缩工具——`unzip`、`gunzip`、`bunzip2`、`unxz`
**命令**：gunzip [ -acfhklLnNrtvV ] [-S suffix]  [filenames...]
&emsp;&emsp;&emsp;bunzip2 [ -fkvsVL ] [ filenames ...  ]
&emsp;&emsp;&emsp;unxz [ -cdfkhqtvzV123456789 ] [ filenames ...  ]
**描述**：解压缩文件，查看压缩文件信息。
|常用选项|作用|备注
|:--|:--|:--:|
|-c|将压缩的数据输出到屏幕上
|-d|解压缩；默认选项
|-f| 目标存在时覆盖，或原文件是符号链接|
|-k|不删除原文件，默认删除执行完后删除源文件|
|-q|不显示警告信息
|-t|检验一个压缩文件的一致性；
|-v|解压缩时显示相关信息；
|-#|#为表压缩等级，1最快，压缩比最差，预设是6
|-l|查看压缩文件的压缩比列等信息|bunzip2不支持|
|-r|递归压缩或解压目录中的每个文件，但不是打包|仅gunzip支持|
|-S SUF|添加后缀SUF|仅gunzip支持|
|-z|压缩|gunzip不支持|
|-e|极限模式|仅unxz支持
|**更多信息**|<http://linux.51yip.com/search/gunzip> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/gunzip>|
||<http://linux.51yip.com/search/bunzip2>|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/bunzip2>|
||<http://linux.51yip.com/search/unxz> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/unxz>|
||man 手册|

## 1.3 自解压工具——`gzexe`、`bzexe`
**命令**：gzexe [ -d ]   [exe_file...]
&emsp;&emsp;&emsp;bzexe [ -d ]   [exe_file...]
**描述**：压缩可执行文件为运行时自解压文件，解压缩自解压文件。输出文件会在文件名后加上后缀~。
|常用选项|作用|
|:--|:--|
|-d|解压缩；
|**更多信息**|<http://linux.51yip.com/search/gzexe> |
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/gzexe>|
||<http://linux.51yip.com/search/gzexe>|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/gzexe>|
||man 手册|


## 1.4 *.Z转为\*.gz格式——`znew`
**命令**：znew [ -ftv9PK] [ name.Z ...  ]
**描述**：将\*.Z文件重新压缩为*.gz文件。
|常用选项|作用|
|:--|:--|
|-f|强制执行转换操作，即是目标“.gz”已经存在；
|-t|删除原文件前测试新文件；
|-v|显示文件名和每个文件的压缩比；
|-9|食用油花的压缩比，速度较慢；
|-P|使用管道完成转换操作，以降低磁盘空间使用；
|-K|当“.Z”文件比“.gz”文件小时，保留“.Z”文件。
|**更多信息**|<http://linux.51yip.com/search/znew> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/znew>|

## 1.5 强制添加*.gz后缀——`zforce`
**命令**：zforce [ name...  ]
**描述**：强制gzip压缩文件加上后缀.gz，以避免压缩两次。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/zforce> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/zforce>|

## 1.6 压缩文件修复工具——`bzip2recover`
**命令**：bzip2recover  [filename.bz2...]
**描述**：修复损坏的bz2文件。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/bzip2recover> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/bzip2recover>|**




# 2 打包——`tar`
**命令**：tar [ -z|j|J ] [ -x|c|t ] [ -v ] -f   [ dest ] srv
&emsp;&emsp;&emsp;tar [ -Adr ] [ --delete ] -f [ dest ] srv
**描述**：将多个目录或文件打包成一个大文件，同时还可以通过 gzip/bzip2/xz 的支持，将该文件同时进行压缩！ 如果只想解打包单独的文件，只需目标文件为该文件即可。
|常用选项|作用|
|:--|:--|
|-A|新增文件到已存在的打包文件；
|-c|建立新的打包文件；
|-C dir|将打包文件拆放在dir目录下
|-d| 找出归档文件和文件系统之间的差异
|-x|解打包文件
|-t|列出打包文件的内容
|-z|通过gzip指令处理备份文件；
|-j|通过bzip2处理文件
|-J|通过xz处理文件
|-f 文件|指定要处理的文件,该选项只能在最末尾
|-v|显示指令执行过程；
|-r|添加文件到打包文件末尾
|-p|保留原本权限与属性，常用于备份(-c)重要的配置文件
|-P|保留绝对路径，即允许备份数据中含有根目录；
|--delete|从打包文件删除文件
|--exclude=FILE|在打包过程中，不要将FILE 打包！
|--newer TIME| 仅打包mtime 与 ctime 新于TIME的文件
|--newer-mtime TIME|仅打包mtime新于TIME的文件
|**更多信息**|<http://linux.51yip.com/search/tar> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/tar>|


# 3 分割文件—`split`
**命令**：split [OPTION]... [FILE [PREFIX]] 
**描述**：将一个文件分割为多个文件，合并分割文件则是使用重定向。

|常用选项|	作用|
|:--|:--|
|-a N|	后缀长度N，默认为2
|-b SIZE|	每个分割文件的大小
|-C SIZE|	分割文件中单行最大字节
|-d	|使用数字而不是字母作为切割后的小文件的后缀
|-l	|每个分割文件的行数
|PREFIX	|分割文件名的前缀为PREFIX
|更多信息|	<http://linux.51yip.com/search/split> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|	<http://man.linuxde.net/split>

# 4 文件系统备份与还原
## 4.1 xfs文件系统备份——`xfsdump`
**命令**：xfsdump [-L S_label] [-M M_label] [-l #] [-f  备 份名]  待备份文件
&emsp;&emsp;&emsp;xfsdump -I
**描述**：xfsdump是通过文件系统的UUID来分辨各个备份档的，因此不能备份两个具有相同UUID的文件系统，仅支持文件系统的备份，不支持特定目录的备份，不支持没有挂载的文件系统备份，必须使用root权限，只能备份XFS，备份文件只能用xfsrestore解析。
|常用选项|作用|
|:--|:--|
|-L |纪录每次备份的 session 标头，这里填写针对此文件系统的简易说明
|-M |纪录储存媒体的标头，这里可以填写此媒体的简易说明
|-l #|指定备份等级，0~9， 0完整备份，1-9仅备份与前一级备份的差异
|-f 备份名|产生的文件，/dev/st0装置文件名或其他一般文件档名等
|-I|从 /var/lib/xfsdump/inventory 列出目前备份的信息状态
|**更多信息**|<http://linux.51yip.com/search/xfsdump> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/xfsdump>|

## 4.2 xfs文件系统恢复——`xfsrestore`
**命令**：xfsrestore -I 
&emsp;&emsp;&emsp;xfsrestore [-f  备份文件 ] [-L S_label] 待复原目录 
&emsp;&emsp;&emsp;xfsrestore [-f  备份文件 ] [-L S_label] [-s] 单一目录或文件  待复原目录 
&emsp;&emsp;&emsp;xfsrestore [-f  备份文件 ] -r  待复原目录 
&emsp;&emsp;&emsp;xfsrestore [-f  备份文件 ] -i  待复原目录 
**描述**：文件系统恢复，恢复顺序与备份顺序一样。
|常用选项|作用|
|:--|:--|
|-I|查询备份数据，包括 Label 名称与备份时间等
|-f|后面接的就是备份文件！
|-L|就是 Session 的 Label name 喔！可用 -I 查询到数据
|-s|需要接某特定目录，即仅复原某一个文件或目录
|-r|如果是用文件来储存备份数据，那这个就不需要使用。如果是一个磁带内有多个文件，需要这东西来达成累积复原
|-i |进入互动模式，进阶管理员使用的！一般我们不太需要操作它！
|**更多信息**|<http://linux.51yip.com/search/tar> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/tar>|

## 4.3 ext文件系统备份——`dump`
**命令**：dump [- Suvj] [-level] [-f 备份文件] 待备份文件
&emsp;&emsp;&emsp;dump -w 
**描述**：备份文件系统或目录。当备份文件系统时可以使用dump的完整功能，和0-9级，可以使用挂载点或设备文件名；当备份目录时，所有待备份数据都要在该目录，只支持0级，不支持-u。

|常用选项|作用|
|:--|:--|
|-S|仅显示待备份数据需要多少磁盘空间
|-u|将这次dump时间记录到/etc/dumpdates|
|-v|显示dump过程
|-j|加入bzip2支持，将数据压缩，默认等级为2
|-level|备份等级，0-9,0是完整备份，1-9为差异备份|
|-f|产生的备份文件|
|-w|列出在/etc/fstab里面的具有dump设置的分区是否备份过|
|**更多信息**|<http://linux.51yip.com/search/dump> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/dump>|


## 4.4 ext文件系统恢复——`restore`
**命令**：restore -t [-f dumpfile] [-h]
&emsp;&emsp;&emsp;restore -C [-f dumpfile] [-D 挂载点]
&emsp;&emsp;&emsp;restore -i [-f dumpfile] [-h]
&emsp;&emsp;&emsp;restore -r [-f dumpfile] [-h]
**描述**：文件系统恢复，恢复顺序与备份顺序一样。
|常用选项|无|
|:--|:--|
|-t|查看备份文件中的重要数据
|-C|列出备份文件中与当前文件系统不一样的文件
|-i|互动模式。可以仅还原部分文件
|-r|将整个文件系统还原的一种模式，用于还原文件系统的备份
|-f|接要处理的备份文件。
|-D|与-C搭配，可以查出后面接的挂载点与dump内有不同的文件
|**更多信息**|<http://linux.51yip.com/search/restore> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/restore>|

# 5 光盘刻录
## 5.1 建立映像文件——`mkisofs`、`genisoimage`
**命令**：mkisofs|genisoimage [-o  映像文件 ] [-Jrv] [-V vol] [-m file]  待备份文件... -graft-point isodir=systemdir...
**描述**：制作映像文件。
|常用选项|作用|
|:--|:--|
|-o |后面接你想要产生的那个映像文件。
|-J |产生兼容windows机器的文件名结构，可增加文件名长度到64个unicode字符
|-r |通过Rock Ridge扩展来记录unix文件系统的数据，否则只支持dos系统
|-v |显示建置 ISO 文件的过程
|-V vol|建立 Volume，有点像Windows在文件管理器内看到的CD title的东西
|-m file|要排除的文件，后面的文件不备份到映像档中，克使用*通配符
|-graft-point|将实际路径与映像文件中的路径对应起来，否则直接加到映像文件的根目录
|**更多信息**|<http://linux.51yip.com/search/mkisofs> |
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/mkisofs>|
||<http://linux.51yip.com/search/genisoimage> |
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/genisoimage>|
||man 手册

## 5.2 刻录工具——`cdrecord`、`wodim`
**命令**：wodim  --devices dev=/dev/sr0... 
&emsp;&emsp;&emsp;wodim -v dev=/dev/sr0 blank=[fast|all]
&emsp;&emsp;&emsp;wodim -v dev=/dev/sr0 -format
&emsp;&emsp;&emsp;wodim -v dev=/dev/sr0 [可用选项功能] file.iso
&emsp;&emsp;&emsp;cdrecord -scanbuSdev=ATA 
&emsp;&emsp;&emsp;cdrecord -v dev=ATA:x,y,z blank=[fast|all]
&emsp;&emsp;&emsp;cdrecord -v dev=ATA:x,y,z -format
&emsp;&emsp;&emsp;cdrecord -v dev=ATA:x,y,z [可用选项功能] file.iso
**描述**：将映像文件刻录到光盘。
|常用选项|作用|
|:--|:--|
|-scanbuSdev|扫瞄磁盘总线并找出可用的刻录机，后续的装置为 ATA 接口
|--devices|扫瞄磁盘总线并找出可用的刻录机，后续的装置为 ATA 接口
|-v|在 cdrecord 运作的过程中，显示过程而已。
|dev=/dev/sr0|可以找出此光驱的 bus 地址，非常重要！
|dev=ATA:x,y,z|后续x,y,z为刻录机所有位置，非常重要！
|blank|blank 为抹除可重复写入的 CD/DVD-RW，使用 fast 较快，all 较完整
|-format |对光盘片进行格式化，但是仅针对 DVD+RW 这种格式的 DVD 而已；
|[可用选项] |主要是写入 CD/DVD 时可使用的选项，常见的选项包括有：
|-data |指定后面的文件以数据格式写入，不是以 CD 音轨(-audio)方式写入！
|speed=X |指定刻录速度，例如 CD 可用 speed=40 为 40 倍数，DVD 则可用 speed=4 之类
|-eject |指定刻录完毕后自动退出光盘
|fs=Ym |指定多少缓冲存储器，可用在将映像档先暂存至缓冲存储器。预设为 4m，一般建议可增加到 8m ，不过，还是得视你的刻录机而定。
|针对 DVD 的选项|
|driveropts=burnfree |打开 Buffer Underrun Free 模式的写入功能
|-sao |支持 DVD-RW 的格式
|**更多信息**|<http://linux.51yip.com/search/cdrecord> |
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/cdrecord>
||<http://linux.51yip.com/search/wodim> |
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/wodim>|
||man 手册

# 6 其他常见的压缩与备份工具
## 6.1 磁盘备份——`dd`
**命令**：dd if="input_file" of="output_file" bs="block_size" count="number"
**描述**：读取磁盘装置的内容，然后将整个装置备份成一个文件呢！此外还可以备份文件，制作指定大小的文件。dd是一个一个扇区去读/写的，即使没有用到的扇区也会倍写入备份文件中！因此这个文件会变得跟原本的磁盘一模一样大！不过，dd就是因为不理会文件系统，单纯有啥纪录啥，因此不论该磁盘内的文件系统你是否认识，它都可以备份、还原的！
|常用选项|作用|
|:--|:--|
|if |就是 input file 啰～也可以是装置喔！
|of |就是 output file 喔～也可以是装置；
|bs |一个 block 的大小，若未指定则预设是 512 bytes(一个 sector 的大小)
|count|多少个 bs 的意思。
|**更多信息**|<http://linux.51yip.com/search/dd> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/dd>|

## 6.2 标准IO备份——`cpio`
**命令**：cpio -ovcB > [file|device] 
&emsp;&emsp;&emsp;cpio -ivcdu < [file|device] 
&emsp;&emsp;&emsp;cpio -ivct < [file|device] 
**描述**：可以备份任何东西，包括装置设备文件。 cpio 不会主动的去找文件来备份！所以cpio得要配合类似find等可以找到文件名的指令来告知 cpio 该被备份的数据在哪里。
|常用选项|作用|
|:--|:--|
|-o |将数据 copy 输出到文件或装置上
|-B |让预设的Blocks可以增加至5120 bytes ，预设是 512 bytes ！让大文件的储存速度加快
|还原选项|
|-i |将数据自文件或装置 copy 出来系统当中
|-d |自动建立目录！使用cpio所备份的数据内容不见得会在同一层目录中，因此必须让cpio在还原时可以建立新目录
|-u |自动的将较新的文件覆盖较旧的文件！
|-t |需配合 -i 选项，可用在"察看"以 cpio 建立的文件或装置的内容
|共享选项|
|-v |让储存的过程中文件名可以在屏幕上显示
|-c |一种较新的 portable format 方式储存
|**更多信息**|<http://linux.51yip.com/search/dd> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/dd>|


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
