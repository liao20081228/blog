---
title: Linux文件与目录管理
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 管理目录与文件
## 1.1 相对路径与绝对路径

* 绝对路径：由根目录 / 写起，例如： /usr/share/doc 这个目录。
* 相对路径：不是由 /写起，例如../man，相对路径意指相对于目前工作目录的路径！
## 1.2 特殊目录符号
|符号|意义|
|:----|:--|
|.| 代表此层目录
|  \.\. | 代表上一层目录
|-| 代表前一个工作目录
|~ |代表『目前用户身份』所在的家目录
|~account |代表 account 这个用户的家目录(account 是个账号名称)

## 1.3 查看目录项——`ls`
**命令**： ls [OPTION]... [FILE]...
**描述**：列出有关FILE的信息（默认为当前目录）。 

|常用选项|作用|
|:----|:--|
|-a|列出所有目录项，包括.和..和隐藏项
|-A|列出所有目录项，包括隐藏项，不包括.和..
|-d |仅列出目录本身，而不是列出目录内的文件数据(常用)
|-f |直接列出结果，而不进行排序 (ls 预设会以name排序！)
|-F |根据文件、目录等信息，给予附加数据结构，例如：*代表可执行文件； /代表目录； =代表 socket 文件； \|代表 FIFO 文件；
|-h|和-l或-s一起使用。以容易理解的格式列出文件大小 (例如 1K 234M 2G)
|-i|打印出每个文件的 inode 号|
|-l|列出文件的详细信息。
|-n |列出 UID 与 GID 而非使用者与群组的名称
|-r|将排序结果反向输出，例如：原本档名由小到大，反向则为由大到小；
|-R |连同子目录内容一起列出来，等于该目录下的所有文件都会显示出来；
|-S |以文件容量大小排序，而不是用name排序；
|-t|依时间排序，而不是用档名。
|--color=WHEN |WHEN为auto\|always\|never，自动\|始终\|从不彩色输出
|--full-time |以完整时间模式 (包含年、月、日、时、分) 输出
|--time=TIME|TIME为atime或ctime，输出atime或ctime,默认为mtime
|**更多信息**|<http://linux.51yip.com/search/ls> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/ls>|


## 1.4 改变当前目录——`cd`
**命令**：cd [相对路径 或 绝对路径] 目录名
**描述**：改变工作目录。将当前工作目录改变到指定的目录下。

|常用选项|作用|
|:----|:--|
|无|
|**更多信息**|<http://linux.51yip.com/search/cd> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/cd>|

## 1.5 查看当前工作目录——`pwd`
**命令**：pwd [options]
**描述**：显示当前工作目录。
|常用选项|作用|
|:----|:--|
|-P|以符号链接所指的目录代替符号链接|
|**更多信息**|<http://linux.51yip.com/search/pwd> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/pwd>|

## 1.6 新建空目录——`mkdir`
**命令**： mkdir [OPTION]... DIRECTORY...
**描述**：如果目录不存在就创建目录。 
|常用选项|作用|
|:----|:--|
|-m MODE|按给定权限(数字表示)创建目录
|-p|根据需要创建路径上的所有目录
|**更多信息**|<http://linux.51yip.com/search/mkdir> 和 man 手册|
|&emsp;&emsp;|<http://man.linuxde.net/mkdir>|

## 1.7 删除空目录——`rmdir`
**命令**：rmdir [OPTION]... DIRECTORY...
**描述**：移除空文件夹。
|常用选项|作用|
|:----|:--|
|-p|删除路径上的所有空目录
|**更多信息**|<http://linux.51yip.com/search/rmdir> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/rmdir>|

## 1.8 拷贝目录或文件——`cp`
**命令**：cp [OPTION]... [-T] SOURCE DEST
&emsp;&emsp;&emsp;cp [OPTION]... SOURCE... DIRECTORY
&emsp;&emsp;&emsp;cp [OPTION]... -t DIRECTORY SOURCE...
**描述**：拷贝SOURCE到DEST，或多个SOURCE到DIRECTORY。长选项必须的参数短选项也必须。

|短选项|长选项|描述（V8.25）|
|:----|:--|:--|
|-a| --archive|等效于 -dR --preserve=all
||--attributes-only|只拷贝属性|
||--backup[=CONTROL]|设置备份方式，可为none、numbered、existing、simple
|-b||类似--backup，但没有参数|
||--copy-contents|在递归时拷贝文件目录|
|-d ||等效于 --no-dereference --preserve=links|
|-f|--force|如果无法打开已存在的目标文件，会删除该文件并再尝试打开，与-n选项一起使用时会失效|
|-H||跟随SOURCE中的命令行符号链接|
|-i| --interactive|重写文件前需要确认，覆盖前面的-n选项|
|-l|--link|使用硬链接代替拷贝
|-L|--dereference|拷贝源文件而不是符号链接|
|-n|--no-clobber|不要覆盖已存在的文件
|-P|--no-dereference|拷贝符号链接而不是源文件|
|-p | |等效于--preserve=mode,ownership,timestamps|
||--preserve[=ATTR]|保留指定的文件属性，(默认值为：mode，ownership，timestamps)， 若可能，额外的属性：context, links, xattr, all
||--parents|复制前先在<目录>创建来源文件路径中的所有目录
|-R, -r|--recursive|复制目录及目录内的所有项目
|| --reflink[=WHEN]|控制拷贝方式。always为写时复制，auto为标准复制|
||--remove-destination|删除已存在的DEST
||--sparse=WHEN|设置稀疏文件的复制方式。可以为auto，always，never
||--strip-trailing-slashes|移除SOURCE末尾的斜杠
|-s|--symbolic-link| 只创建符号链接而不是复制文件
|-S|--suffix=SUFFIX|指定备份后缀，默认为~|
|-t|--target-directory=DIRECTORY|将所有的SOURCE复制到DIRECTORY
|-T|--no-target-directory|将DEST作为普通文件|
|-u|--update|只在SOURCE比DEST新，或DEST不存在时才进行复制
|-v|--verbose|显示正在做的事|
|-x|--one-file-system|保留文件系统
|-Z||将DEST文件的SELinux安全环境设为默认类型|
||--context[=CTX]|类似-Z，设置SELinux或SMACK安全环境为CTX|
||--help|显示帮助信息
||--version|显示版本信息
|&emsp;&emsp;&emsp;|

## 1.9 删除目录或文件——`rm`
**命令**：rm [OPTION]... [FILE]...
**描述**：rm删除每个指定的文件。默认情况下，它不会删除目录。
|常用选项|作用|
|:----|:--|
|-f, --force|强制删除，忽略不存在的文件，从不给出提示。
|-i| 交互模式删除文件，删除文件前给出提示。
|-I |在删除多于三个文件之前或递归删除时提示， 
| -r, -R, --recursive|递归删除
|-d, --dir|删除空目录
|**更多信息**|<http://linux.51yip.com/search/rm> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/rm>|

## 1.10 新建文件
### 1.10.1 `touch`
**命令**：touch [OPTION]... [FILE]...
**描述**：如果文件不存在则创建文件。
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/touch> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/touch>| 

### 1.10.2 `vim`
**命令**：vim filename
**描述**：如果文件不存在则创建文件。
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/vim> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/vim>| 

## 1.11 移动目录或文件——`mv`
**命令**：mv [OPTION]... [-T] SOURCE DEST
&emsp;&emsp;&emsp;mv [OPTION]... SOURCE... DIRECTORY
&emsp;&emsp;&emsp;mv [OPTION]... -t DIRECTORY SOURCE...
**描述**：移动SOURCE到DEST，或重命名。
|常用选项|作用|
|:----|:--|
|-f |force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；
|-i |若目标文件 (destination) 已经存在时，就会询问是否覆盖！
|-u|若目标文件已经存在，且 source 比较新，才会更新 (update)
|**更多信息**|<http://linux.51yip.com/search/mv> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/mv>|

## 1.12 获取基名和目录名
### 1.12.1 `basename`
**命令**：basename NAME [SUFFIX]
&emsp;&emsp;&emsp;basename OPTION... NAME...
**描述**：打印NAME中的基本名。
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/basename> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/basename>|

### 1.12.1 `dirname`
**命令**： dirname [OPTION] NAME...
**描述**：输出NAME中的目录名。
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/dirname> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/dirname>|




# 2 查找文件或目录
## 2.1 在PATH中查找命令文件——`which`
**命令**：which [-a] filename ...
**描述**：根据当前环境变量PATH查找命令文件所在路径。
|常用选项&emsp;|作用|
|:----|:--|
|-a|列出所有匹配的路径，而不是第一个
|**更多信息**|<http://linux.51yip.com/search/which> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/which>| 

## 2.2 在特定目录查找文件——`whereis`
**命令**：whereis [options] [-BMS directory... -f] name...
**描述**： 在特定目录查找文件。
|常用选项&emsp;|作用|
|:----|:--|
|-l |可以列出 whereis 会去查询的几个主要目录
|-b |只找 binary 格式的文件
|-m |只找在说明文件 manual 路径下的文件
|-s |只找 source 来源文件
|-u |搜寻不在上述三个项目当中的其他特殊文件
|-a|列出所有匹配的路径，而不是第一个
|**更多信息**|<http://linux.51yip.com/search/whereis> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/whereis>|

## 2.3 利用数据库中查找文件——`locate`
**命令**：locate [OPTION]... PATTERN...
**描述**：利用updatedb创建的数据库中查找匹配PATTERN的文件，如果没有指定-r，则PATTERN可以使用通配符。
|常用选项&emsp;|作用|
|:----|:--|
|-i |忽略大小写的差异；
|-c |不输出文件名，仅计算找到的文件数量
|-l |仅输出几行的意思，例如输出五行则是 -l 5
|-S |输出 locate 所使用的数据库文件的相关信息，包括该数据库纪录的文件/目录数量等
|-r |后面可接正则表达式
|**更多信息**|<http://linux.51yip.com/search/locate·> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/locate>|

## 2.4 查找文件——`find`
**命令**：find 起始目录 查找条件 操作
**描述**：在指定目录结构中搜索问件，并执行指定的操作。该命令的查找条件可以是一个逻辑运算符 not、and、or 组成的复合条件:

|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/find> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/find>|


* 逻辑运算符
  * -a——逻辑与
  * -o——逻辑或
  * !——逻辑非
* 常用的查找条件有：
  * 根据名称和文件属性查找。
     * -name '字串'—— 查找文件名匹配所给字串的所有文件，字串内可用通配符*、?、[ ]。
     * -gid n ——查找属于 ID 号为 n 的用户组的所有文件。
     * -uid n ——查找属于 ID 号为 n 的用户的所有文件。
     * -group '字串' ——查找属于用户组名为所给字串的所有的文件。
     * -user '字串' ——查找属于用户名为所给字串的所有的文件。
     * -empty ——查找大小为 0 的目录或文件。
     * -perm [-|/]MODE ——查找具有指定权限的文件和目录，-表示目标权限要包括MODE，/表示目标权限要包括MODE中的任意权限。
     * -size [+|-]n[bckw] ——查找指定文件大小的文件，n 后面的字符表示单位，缺省为b，代表 512字节的块。+表示大于，-表示小于，省略表示等于。
     * -type x ——查找类型为 x 的文件，x 为下列字符之一：-	l c	d p	s b 	
  * 根据时间查找
     * -amin [+|-]n ——查找 n 分钟以前被访问过的所有文件。（+表示 n+1分钟之前，-表示n分钟之内，省略表示n到n+1分钟）
     * -cmin [+|-]n ——查找 n 分钟以前文件状态被修改过的所有文件。
     * -mmin [+|-]n ——查找 n 分钟以前文件内容被修改过的所有文件。
     * -atime [+|-]n ——查找 n 天以前被访问过的所有文件。
     * -ctime [+|-]n ——查找 n 天以前文件状态被修改过的所有文件。
     * -mtime [+|-]n ——查找 n 天以前文件内容被修改过的所有文件。
* 可执行的操作。
  * -exec 命令名称 {}——对符合条件的文件执行所给的Linux命令，而不询问用户是否需要执行该命令。{}表示命令的参数即为所找到的文件；命令的末尾必须以“ \;”结束。例如，在“/home/user“目录下查找名为 main.c文件并显示这些文件的详细信息，则使用find /home/user –name main.c –exec ls –l {} \;或者
  * -ok 命令名称 {} ——对符合条件的文件执行所给的 Linux命令，与 exec 不同的是，它会询问用户是否需要执行该命令。


# 3 管理文件权限与属性
Linux中目录也是一种特殊的文件。
## 3.1 文件权限与属性
`ls -al`显示信息如下：

|权限与文件类型| 链接数| 文件所有者|  文件所属组 | 文件大小| 最近修改时间| 文件名字
|:----|:--|:----|:--|:----|:--|:--|
|drwxr-xr-x   |   2   |   liao20081228|  liao20081228| 4096   |  Jan 15 20:25  | Music
|-rw------- |      1  |    liao20081228 | liao20081228 |82 |      Jan 17 04:37|  .xsession

### 3.1.1 文件类型
使用 `ls–l`命令显示的信息中，开头是由10个字母构成的字符串，其中第一个字符表示文件类型，它可以是下列类型之一：

|符号	|意义|	符号|	意义|
|:----|:--|:----|:--|
|-	|普通文件|d|	目录|
|l |	符号链接|	
|c	|字符设备，数据以单个字符为单位|b |	块设备文件，数据以块为单位
|p|	命名管道|s | 	Socket文件 	|		

### 3.1.2 文件权限
* 一般权限
  * 使用 `ls –l`命令显示的信息中，开头是由10个字母构成的字符串，后面的 9 个字符表示文件的访问权限，分为 3 组，每组 3位。
  * 第一组表示文件创建者的权限，，第二组表示同组用户的权限 ， 第三组表示其他用户的权限。
  * 每一组的三个字符分别表示对文件的读、写、执行权限。每一组用一个由三种权限代表的数字相加来表示，例如 r_x :5，rw_:6，R__:4，那么这三组就可以用3个数字表示，例如rwxr_xr_x:755 ，rw_r__r__:644。如下：

|权限|对文件的意义|对目录的意义|代码
|:----|:--|:--|:--|
|r|读取文件内容|读取目录项|4
|w|修改文件内容|修改目录及目录项|2
|x|执行文件|进入目录|1
|\_|没有任何权限|没有任何权限|0

* 特殊权限
  * SUID 
     * SUID 权限仅对二进制程序有效；
     * 执行者对于该程序需要具有可执行权限；
     * 本权限仅在执行该程序的过程中有效 (run-time)；
     * 执行者将具有该程序拥有者 (owner) 的权限。 
     * 若具有x权限时，所有者权限的x变为s，否则变为S。
  * SGID
     * 对于文件
         * SGID 对二进制程序有用；
         * 程序执行者对于该程序来说，需具备 x 的权限；
         * 执行者将具有该程序所属组的权限。
         * 若具有x权限时，所属组x变为s，否则变为S。
     * 对于目录
         * 用户若对于此目录具有 r 与 x 的权限时，该用户能够进入此目录；
         * 用户在此目录下的有效群组(effective group)将会变成该目录的群组；
         * 用途：若用户在此目录下具有w的权限(可以新建文件)，则使用者所建立的新文件，该新文件的群组与此目录的群组相同。
     * 若具有x权限时，所属组的x变为s，否则变为S
  * SBIT
     * 目前只针对目录有效，对于文件已经没有效果了。
     * 当用户对于此目录具有w,x权限，亦即具有写入的权限时，当用户在该目录下建立文件或目录时，仅有自己与root才有权力删除该文件，哪怕是目录的拥有者具有w权限也不行。
     * 若具有x权限时，其他人的x变为t，否则变为T。

### 3.1.3 默认权限与权限掩码

* 默认权限
  * 系统创建文件时默认权限为0666，也就是文件默认不具有执行权限。
  * 系统创建目录时默认权限为0777，也就是目录默认具有所有权限。
* 权限掩码mask
  * 在创建文件或目录时，要从默认权限中去除的权限。
  * 对于普通用户为0002，表示去除其他用户的写入权限
  * 对于超级用户为0022，表示去除所有组和其它用户的写入权限。

### 3.1.4 文件时间
Linux为每个文件保存三个时间：

* mtime——当该文件的内容数据变更时，就会更新这个时间！
* ctime——当该文件的状态改变时，就会更新这个时间，如：权限与属性更改。
* atime——当该文件的内容被取用时，就会更新这个读取时间 (access)。

### 3.1.5  文件名与扩展名
* Linux文件名的长度不能超过255字节，一般不建议使用非打印字符和特殊字符。
* Linux中没有所谓的扩展名，扩展名只是为了让用户，了解该文件的用途。

### 3.1.6 隐藏属性
文件除了权限、文件类型、链接数、文件所有者、 文件所属组、 文件大小、最近修改时间、文件名字之外，还有许多隐藏属性。

## 3.2 管理文件权限与属性

### 3.2.1 权限预设——`umask`
**命令**：umask [mask]
**描述**：查看或者设置权限掩码。
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/umask> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/umask>|




### 3.2.2 改变所属组——`chgrp`
**命令**：chgrp [OPTION]... GROUP FILE...
&emsp;&emsp;&emsp;chgrp [OPTION]... --reference=RFILE FILE...
**描述**：改变子目录和文件的所有组。
|常用选项|作用|
|:----|:--|
|-R| 递归式地改变指定目录及其下的所有子目录和文件的拥有者。
|-H|与-R联用，如果一个命令行参数是一个目录的符号链接，遍历它
|-L|与-R联用，遍历每个遇到的目录的符号链接
| -P|与-R联用，不会遍历任何符号链接（默认）
|**更多信息**|<http://linux.51yip.com/search/chgrp> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/chgrp>|

### 3.2.3 改变所有者——`chown`
**命令** ：chown [OPTION]... [OWNER][:[GROUP]] FILE...
&emsp;&emsp;&emsp; chown [OPTION]... --reference=RFILE FILE
**描述**：chown更改每个给定文件的用户和/或组的所有权。

|常用选项|作用|
|:----|:--|
|-R| 递归式地改变指定目录及其下的所有子目录和文件的拥有者。
|-H|与-R联用，如果一个命令行参数是一个目录的符号链接，遍历它
|-L|与-R联用，遍历每个遇到的目录的符号链接
| -P|与-R联用，不会遍历任何符号链接（默认）
|**更多信息**|<http://linux.51yip.com/search/chown> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/chown>|

### 3.2.3 改变权限——`chmod`
只有文件所有者或者超级用户可以修改文件权限。
**命令**： chmod [OPTION]... MODE[,MODE]... FILE...
&emsp;&emsp;&emsp;       chmod [OPTION]... OCTAL-MODE FILE...
&emsp;&emsp;&emsp;       chmod [OPTION]... --reference=RFILE FILE...
**描述**：根据指定模式改变给定文件的文件模式位。该模式可以是符号表示，也可以是八进制数。chmod永远不会改变符号链接的权限，因为从不使用符号链接的权限。但是，对于命令行上列出的每个符号链接，chmod将更改指向文件的权限。 但是，chmod忽略在目录递归遍历期间遇到的符号链接。

* 文字设定法，模式之间可以用逗号分隔，模式格式：[who][+|-|=][right]
  * who 操作对象，可是下述字母中的任一个或者它们的组合：
     * u  表示“用户（user）”，即文件或目录的所有者。
     * g  表示“同组（group）用户”，即与文件所有者有相同组 ID 的所有用户。
     * o  表示“其他（others）用户”。
     * a  表示“所有（all）用户”。它是系统默认值。
     * 省略 默认为a，但是umask中设置的位不受影响。
  * 操作符号可以是：
     * \+ 添加某个权限。
     * \- 取消某个权限。
     * = 赋予给定权限并取消其他所有权限（如果有的话）。
  * right所表示的权限可用下述字母的任意组合：
     * r 可读 
     * w 可写
     * x 可执行
     * s SUID和SGID权限，让执行者享有文件所有者或所属组的权限，有x权限时为大写，否则为小写
     * t SBIT权限，只有所有者和root可以删除或重命名，有x权限为大写，否则为小写
  * 文件名：以空格分开的要改变权限的文件列表，支持通配符。
* 数字设定法
  * mode由4个八进制数表示：
     * 第一位，SUID，SGID，SBIT权限
     * 第二位，所有者权限
     * 第三位，所属组权限
     * 第四位，其他人权限
  * 每个八进制数由0,1,2,4相加得到：
     * 0 表示没有权限
     * 1 在第一位表示SBIT权限，在后三位表示可执行权限
     * 2 在第一位表示SGID权限，在后三位表示可写权限
     * 4 在第一位表示SUID权限，在后三位表示可读权限，

|常用选项|作用|
|:----|:--|
|-R| 递归式地改变指定目录及其下的所有子目录和文件的拥有者。
|**更多信息**|<http://linux.51yip.com/search/chmod> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/chmod>|

### 3.2.4 改变atime和mtime——`touch`

**命令**：touch [OPTION]... [FILE]...
**描述**：将文件的atime和mtime更改为当前时间。
|常用选项&emsp;|作用|
|:----|:--|
|-a |仅修订 atime；
|-c |不创建文件；
|-d STRING |用STRING代替当前时间
|-m |仅修改 mtime ；
|-t STAMP | 使用 [[CC]YY]MMDDhhmm[.ss] 得时间来代替当前时间
|**更多信息**|<http://linux.51yip.com/search/touch> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/touch>|

### 3.2.5 查看文件类型——`file`
**命令**：file [OPTION]... [FILE]...
**描述**：显示文件类型的相关信息，不仅仅是系统文件类型分类。
|常用选项&emsp;|无|
|:----|:--|
|无|
|**更多信息**|<http://linux.51yip.com/search/file> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/file>|

### 3.2.6 设置文件隐藏属性——`chattr`
**命令**： chattr [ -RVf ] [ -v version ] [ mode ] files...
**描述**：显示文件类型的相关信息，不仅仅是系统文件类型分类。
|常用选项|作用|
|:----|:--|
|-R|递归地修改目录以及其下内容的属性.如果在递归目录时遇到了符号链接,遍历将跳过.
|**更多信息**|<http://linux.51yip.com/search/chattr> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/chattr>|

|常用属性|作用|
|:----|:--|
|A|当设定了 A 这个属性时，若你有存取此文件(或目录)时，他的访问时间 atime 将不会被修改，可避免I/O较慢的机器过度的存取磁盘。(目前建议使用文件系统挂载参数处理这个项目)
|S|一般文件是异步写入磁盘的(原理请参考前一章 sync 的说明)，如果加上 S 这个属性时，当你进行任何文件的修改，该更动会同步写入磁盘中。
|a |当设定 a 之后，这个文件将只能增加数据，而不能删除也不能修改数据，只有 root 才能设定这属性
|c |这个属性设定之后，将会自动的将此文件压缩，在读取的时候将会自动解压缩，但是在储存的时候，将会先进行压缩后再储存。
|d |当 dump 程序被执行的时候，设定 d 属性将可使该文件(或目录)不会被 dump 备份
|i |这个 i 可就很厉害了！他可以让一个文件不能被删除、改名、设定连结也无法写入或新增数据！对于系统安全性有相当大的帮助！只有 root 能设定此属性
|s |当文件设定了s属性时，如果这个文件被删除，他将会被完全的移除出这个硬盘空间，所以如果误删了，完全无法救回来了喔！
|u |与 s 相反的，如果该文件被删除了，则数据内容其实还存在磁盘中，可以用恢复该文件。
|**更多信息**|<http://linux.51yip.com/search/chattr> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/chattr>|

### 3.2.7 查看文件隐藏属性——`lsattr`
**命令**： lsattr [-RVadv] [ files...]
**描述**：显示文件类型的相关信息，不仅仅是系统文件类型分类。
|常用选项&emsp;|作用|
|:----|:--|
|-a |将隐藏文件的属性也显示出来；
|-d |如果参数是目录，仅列出目录本身的属性而非目录内的文件名；
|-R|连同子目录的数据也一并列出来！
|**更多信息**|<http://linux.51yip.com/search/lsattr> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/lsattr>|

# 4 环境变量PATH
**环境变量PATH**用于设定执行文件路径。当执行一个命令时，系统会依照PATH变量中的每个目录下搜寻相应的可执行文件， 如果在 PATH 定义的所有目录中含有多个同名可执行文件，那么先搜寻到的同名指令先被执行！

PATH(一定是大写)这个变量的内容是由一堆目录所组成的，每个目录中间用冒号(:)来隔开， 每个目录是有顺序之分的。

* 不同身份使用者预设的 PATH 不同，默认能够随意执行的指令也不同(如 root 与 dmtsai)；
* PATH 是可以修改的；
* 使用绝对路径或相对路径直接指定某个指令的文件名来执行，会比搜寻 PATH 来的正确；
* 指令应该要放置到正确的目录下，执行才会比较方便；
* 本目录(.)最好不要放到 PATH 当中。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
