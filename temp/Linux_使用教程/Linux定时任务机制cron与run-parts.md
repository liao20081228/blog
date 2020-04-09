---
title: Linux定时任务机制cron与run-parts
tags: Linux教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------



<style>table{word-break:initial;}</style>


# 1 简介
**cron**是Unix系统的一个配置定期任务的工具，用于定期或者以一定的时间间隔执行一些命令或者脚本；可执行的任务范围可以是每天夜里自动备份用户的home文件夹，也可以每个小时记录CPU的信息日志。 ubuntu中cron位于`/usr/sbin/cron`。crond守护进程是在系统启动时由init进程启动的，受init进程的监视，如果它不存在了，会被init进程重新启动。这个守护进程每分钟唤醒一次，并通过检查crontab文件判断需要做什么。 


**crontab命令**用于编辑执行中的定期任务列表，并且操作是基于每个用户的，每一个用户（包括root用户）都拥有自己的crontab。 crontab位于`/usr/bin/crontab`。 

**crontab文件**每个用户有一个以用户名命名的crontab文件，存放在`/var/spool/cron/crontabs`目录里。若管理员允许或者禁止其他用户拥有crontab文件，则应编辑/etc/下面的`cron.deny`和`cron.allow`这两个文件来禁止或允许用户拥有自己的crontab文件。每一个用户都可以有自己的crontab文件，但在一个较大的系统中，系统管理员一般会禁止这些文件，而只在整个系统保留一个这样的文件。 

系统首先判断是否有cron.allow这个文件，如果有这个文件的话，系统会判断这个使用者有没有在cron.allow的名单里面，如果在名单里面的话，就可以使用cron机制。如果这个使用者没有在cron.allow名单里面的话，就不能使用cron机制。

如果系统里面没有cron.allow这个文件的话，系统会再判断是否有cron.deny这个文件，如果有cron.deny这个文件的话，就会判断这个使用者有没有在cron.deny这个名单里面，如果这个使用者在cron.deny名单里面的话，将不能使用cron机制。如果这个使用者没有在cron.deny这个名单里面的话就可以使用cron机制。

如果系统里这两个文件都没有的话，就可以使用cron机制

# 1 cron命令
**命令**：`cron [-f] [-l] [-L loglevel]`
**描述**：在进入多用户运行级别时，cron会自动从/etc/init.d启动

|选项|描述|
|:--|:--|
|-f|保持前台模式，不要守护进程。|
|-l |为/etc/cron.d文件启用LSB兼容名称。 但是，此设置不会影响`/etc/cron.hourly,/etc/cron.daily,/etc/cron.weekly或/etc/cron.monthly`下的文件解析。|
|-n|发送邮件时在主题中包含FQDN。 默认情况下，cron将缩写主机名。|
|-L loglevel|cron按照以下值的总和记录有关作业的内容（无论此值如何都记录错误）：<br />1 将记录所有cron作业的开始<br />2 将记录所有cron作业的结束<br />4 将记录所有失败的作业（退出状态!= 0）<br />8 记录所有cron作业的进程号<br />默认值为1。如果设置为0则禁止生成日志。设置为15则记录所有日志。
 
 
 **注意**：

* cron在其假脱机区域（`/var/spool/cron/crontabs`）中搜索crontab文件（以`/etc/passwd`中的帐户命名）;并将发现的crontab文件加载到内存中。请注意，不应直接访问此目录中的crontabs文件，应使用crontab命令来访问和更新它们。

* cron还读取`/etc/crontab`，它的格式略有不同（参见crontab(5)）。在Debian中，`/etc/crontab`的内容被预定义为运行`/etc/cron.hourly,/etc/cron.daily,/etc/cron.weekly和/etc/cron.monthly`下的程序。此配置特定用于于Debian，请参阅下面的DEBIAN 规范下的注释。

* 另外，在Debian中，cron读取`/etc/cron.d`目录中的文件。 cron以与`/etc/crontab`文件相同的方式处理`/etc/cron.d`中的文件（它们遵循该文件的特殊格式，即它们包括用户字段）。但是，它们独立于`/ etc/crontab`：例如，它们不会从中继承环境变量设置。此更改特定于Debian，请参阅下面的DEBIAN SPECIFIC下的注释。

* 与`/etc/crontab`一样，`/etc/cron.d`目录中的文件变化也被监视。通常，系统管理员不应使用`/etc/cron.d/`，而是使用标准系统crontab文件：`/etc/crontab`。

 * `/etc/crontab`和`/etc/cron.d`中的文件必须由root拥有，并且同组或其他用户不能拥有写权限。与假脱机区域相反，`/ etc/cron.d`下的文件或`/etc/cron.hourly,/etc/cron.daily,/etc/cron.weekly和/etc/cron.monthly`下的文件也可能是符号链接，只要符号链接和它指向的文件都由root拥有。 当它们由run-parts运行时（有关更多信息，请参阅 run-parts(8)），`/etc/cron.d`下的文件不需要是可执行的，而/etc/cron.hourly,/etc/cron.daily,/etc/cron.weekly和/etc/cron.monthly下的文件需要可执行权限。

*  之后后每分钟唤醒一次，检查所有存储的crontabs文件，检查每个命令以查看它是否应该在当前运行。执行命令时，任何输出都将邮寄给crontab的所有者（或者crontab的MAILTO环境变量中指定的用户，如果存在的话）。运行这些进程的cron的子副本的名称被强制为大写，如syslog和ps输出中所示。

* 另外，cron每一分钟检查一次其假脱机目录的修改时间（或`/ tc/crontab`文件的modtime）是否已更改，如果有，cron将检查所有crontabs文件的修改时间并重新加载已更改的那些。因此，**无论何时修改crontab文件，都不需要重新启动cron**。请注意，只要crontab(1)命令更改了crontab文件，它就会更新假脱机目录的modtime。 

* 当时钟变化少于3小时时，例如在夏令时的开始和结束时，存在特殊考虑因素。 如果时间已经向前移动，那些在跳过的时间内运行的作业将在更改后立即运行。 相反，如果时间向后移动不到3小时，那些落入重复时间的工作将不会重新运行。

* 只有在特定时间运行的作业（未指定为@hourly，也不是在小时或分钟说明符前有`*`）才会受到影响。 使用通配符指定的作业将立即根据新时间运行。

* 超过3小时的时钟变化被认为是对时钟的校正，将立即使用新时间。

* cron将其操作记录到syslog工具'cron'，并且可以使用标准syslogd(8)工具控制日志记录。

**环境**

* 如果在Debian系统中的`/etc/default/cron`中配置，则可以通过使用`/etc/environment`或使用`/etc/default/locale`来管理cron守护程序本地化设置环境，其中后者的值覆盖来自前者的。 读取这些文件，它们将用于设置LANG，LC_ALL和LC_CTYPE环境变量。 然后使用这些变量来设置邮件的字符集，默认为“C”。

* 这不会影响在cron下运行的任务的环境。 有关如何修改任务环境的更多信息，请参阅crontab（5）

* 守护进程将使用`/etc/timezone`中的定义（如果存在）作为时区。

* 可以在用户的crontab定义中重新定义环境，但cron只能在单个时区中处理任务。

**debian规范**
* Debian的cron引入了一些变化，这些变化最初不是上游可用的。引入的最重要的变化是：
	*  支持 将 `/etc/crontab`预定义为`/etc/cron.{hourly,daily,weekly,monthly}`,
	*  支持`/etc/cron.d`(包crontabs文件的drop-in目录），
	* PAM 支持,
	* SELinux 支持,
	* 审计日志支持，
	* 夏令时和其他与时间相关的变更/修复，
	* SGID crontab（1）而不是SUID root，
	* Debian特定的文件位置和命令，
	* 特定于Debian的配置（`/etc/default/cron`），
	* 许多其他较小的功能和修复。

* 在Debian中通过`/etc/crontab`文件的默认设置提供对`/etc/cron.hourly,/etc/cron.daily,/etc/cron.weekly和/etc/cron.monthly`的支持（请参阅 crontab（5）中的系统范围示例）。默认的系统范围的crontab包含四个任务：每小时，每天，每周和每月运行。这些任务中的每一个都将执行run-parts，并把每个目录作为run-parts的参数。如果安装了anacron（每小时任务除外），则会禁用这些任务，以防止两个守护程序之间发生冲突。

* 如上所述，这些目录下的文件必须通过一些健全性检查，包括以下内容：可执行，由root拥有，不可由组或其他写入，如果符号链接，则指向root拥有的文件。此外，文件名必须符合run-parts的文件名要求：它们必须完全由字母，数字组成，并且只能包含特殊符号下划线（`'_'`）和连字符（`' - '`）。任何不符合这些要求的文件都不会被运行部件执行。例如，任何包含点的文件都将被忽略。这样做是为了防止cron安把`/etc/cron.d/`中的Debian软件包管理系统留下的任何文件作为配置文件运行（即以.dpkg-distorig, dpkg-new，.dpkg-orig结尾的文件）。

* 系统管理员和程序包可以使用此功能来引入将按定义的时间间隔运行的任务。由这些目录中的包创建的文件应以提供它们的包命名。

* 对`/etc/cron.d`的支持包含在cron守护程序本身中，它将此位置作为系统范围的`crontab spool`处理。此目录可以包含按照`/ etc / crontab`中使用的格式定义任务的任何文件，即与用户cron spool不同，这些文件必须提供用户名以在任务定义中运行任务。

* 此目录中的文件必须由root拥有，不需要是可执行的（它们是配置文件，就像`/etc/crontab`一样），并且必须符合run-parts（8）所使用的相同命名约定：它们必须仅由大写和小写字母，数字，下划线和连字符组成。这意味着它们不能包含任何点。如果为cron指定了-l选项（此选项可以通过`/etc/default/cron`设置，请参见下文），那么它们必须符合LSB命名空间规范，与run-parts中的--lsbsysinit选项完全相同。

* 此功能的目的是允许那些需要比`/etc/cron.{hourly,daily,weekly,monthly}`目录更精细地控制其调度的程序包将crontab文 添加到`/etc/cron.d`。此类文件应以提供它们的包命名。

* 此外，cron的默认配置由`/etc/default/cron`控制，它由启动cron守护程序的`init.d`脚本读取。此文件确定cron是否将读取系统的环境变量，并且可以在执行之前向cron程序添加其他选项，以配置其日志记录或定义如何处理`/etc/cron.d`下的文件。

# 2 crontab命令

**命令**：`crontab [ -u user ] file`
&emsp;&emsp;&emsp;`crontab [ -u user ] [ -i ] { -e | -l | -r }`
**描述**：
* crontab是用于安装，卸载或列出用于驱动Vixie Cron中的cron(8)守护程序的表。每个用户都可以拥有自己的crontab，虽然这些是`/var/spool/cron/crontabs`中的文件，但它们不能直接编辑。

*  如果`/etc/cron.allow`文件存在，则用户必须在其中列出（每行一个用户）才能允许使用此命令。如果`/etc/cron.allow`文件不存在但`/etc/cron.deny`文件确实存在，则在`/etc/cron.deny`文件中列出的不得使用此命令。

*  如果这两个文件都不存在，那么根据站点相关的配置参数，只允许超级用户使用此命令，或者所有用户都可以使用此命令。

*  如果两个文件都存在，则`/etc/cron.allow`优先。这意味着不考虑`/etc/cron.deny`，并且必须列在`/etc/cron.allow`中才能使用crontab。

* 论是否存在任何这些文件，始终允许root管理用户设置crontab。对于标准Debian系统，所有用户都可以使用此命令。

|选项|描述|
|--|--|
|-u|指定要使用crontab（列出时）或修改（编辑时）的用户的名称。如果没有给出这个选项，crontab会检查“你的”crontab，即执行命令的人的crontab。请注意，su(8)可能会混淆crontab，如果你在su(8)中运行，为了安全起见，你应该总是使用-u选项。
|-|如果给出伪文件名“ - ”，则此命令的第一种形式用于从某些命名文件或标准输入安装新的crontab。
|-l|选项使当前的crontab显示在标准输出上。请参阅下面的DEBIAN SPECIFIC下的注释。
|-r|选项导致删除当前的crontab。
|-e|选项用于使用VISUAL或EDITOR环境变量指定的编辑器编辑当前的crontab。退出编辑器后，将自动安装修改过的crontab。如果未定义任何环境变量，则使用默认编辑器`/usr/bin/editor`。
|-i|在实际删除crontab之前提示用户输入“y / Y”响应。

**DEBIAN规范**：
crontab -l 会显示安装时位于crontab开头的三行“DO NOT EDIT THIS FILE”标头。 问题是使得管道命令`crontab -l | crontab  -`  的结果与原始crontab文件不再相等 ，其将不断添加标头的副本。 这会导致使用sed编辑crontab的脚本时感到痛苦。 因此，-l选项的默认行为已更改为不输出此类标头。 您可以通过将环境变量CRONTAB_NOHEADER设置为'N'来获取原始行为，这将导致crontab -l命令发出无关的标头。

 # 3 crontable文件格式
**描述**

* crontab文件包含对cron（8）守护程序的指令的一般形式：`run this command at this time on this date`。每个用户都有自己的crontab，任何给定的crontab中的命令都将作为拥有crontab的用户执行。 Uucp和News通常都有自己的crontabs，不需要显式运行su（1）作为cron命令的一部分。

* 空行、前导空格和制表符将被忽略。第一个非空格字符是哈希符号（＃）的行是注释，也将被忽略。请注意，注释不能与cron命令在同一行上，因为它们将被视为命令的一部分。同样，注释不允许与环境变量设置在同一行。

* crontab中的活动行是环境设置或cron命令。 crontab文件被从上到下进行解析，因此任何环境设置都只会影响文件中位于它们下面的cron命令。环境设置的形式，
&emsp;&emsp;`name = value`
等号（=）周围的空格是可选的，value中任何连续的非前导空格都是分配给name的值的一部分。value字符串可以放在引号中（单引号或双引号，但必须是匹配）以保留前导或尾随空白。要定义空变量，必须使用引号。对于环境替换或变量替换，将不解析value字符串，因此，类似
&emsp;&emsp;`PATH = $HOME/bin$PATH`
不会像你期望的那样工作。接下来的例子也不会奏效
&emsp;&emsp;`A = 1`
&emsp;&emsp;`B = 2`
&emsp;&emsp;`C = $A $B`
最后一个值中的定义变量不会有任何替代。

* 设置命令路径的另一种方法是使用这样的事实，即许多shell会将代字号（~）视为`$HOME`的替换，因此如果您使用bash执行任务，则可以使用：
&emsp;&emsp;`SHELL=/bin/bash`
&emsp;&emsp;`PATH=~/bin:/usr/bin/:/bin`

* cron（8）守护进程自动设置了几个环境变量。 SHELL设置为`/bin/sh`，LOGNAME和HOME是从crontab所有者的`/etc/passwd`行设置的。 PATH设置为`"/usr/bin:/bin"`。 HOME，SHELL和PATH可能会被crontab中的设置覆盖; LOGNAME是作业运行的用户，可能不会更改。

* 另一个注意事项：LOGNAME变量有时在BSD系统上称为USER ......在这些系统上，也会设置USER。

* 除了LOGNAME，HOME和SHELL之外，如果由于在“this”crontab中运行命令而有任何理由发送邮件，cron（8）将查看MAILTO。如果定义了MAILTO（并且非空），则将邮件发送给如此命名的用户。 MAILTO还可用于通过用逗号分隔收件人用户来将邮件定向到多个收件人。如果MAILTO已定义但为空（MAILTO =""），则不会发送任何邮件。否则邮件将发送给crontab的所有者。

* 在Debian GNU/Linux系统上，cron支持pam_env模块，并加载`/etc/environment`和`/etc/security/pam_env.conf`指定的环境。它还从`/etc/default/locale`读取本地化设置信息。但是，PAM设置不会覆盖上述设置，也不会覆盖crontab文件本身的任何设置。请特别注意，如果您想要`"/usr/bin:/bin"`以外的PATH，则需要在crontab文件中进行设置。

* 默认情况下，cron将使用“Content-Type"邮件：如果未设置LC\_\* 环境变量，或者由LC\_\*环境变量指定本地化设置（请参阅locale(7)），含有"charset="的纯文本头部将本地化的charmap/codeset 设置为crond(8)启动时的语言环境，即默认的系统本地化设置。通过将crontabs中的CONTENT_TYPE和CONTENT_TRANSFER_ENCODING变量设置为这些用户的邮件头的正确值，可以对邮寄的cron作业输出使用不同的字符编码。

* cron命令的格式非常符合V7标准，具有许多向上兼容的扩展。每行有五个时间和日期字段，后跟一个命令，后跟一个换行符（'\ n'）。系统crontab（`/etc/crontab`）使用相同的格式，但命令的用户名在时间和日期字段之后，命令之前指定。字段可以用空格或制表符分隔。命令字段的最大允许长度为998个字符。

* 当分钟、小时和月份字段与当前时间匹配时，以及当“两天”（一个月的第几天或星期几）字段中的至少一个与当前时间匹配时，命令将由cron(8)执行（请参阅下面的“注意”）。 cron(8)每分钟检查一次cron条目。时间和日期字段是：
&emsp;&emsp;字段    &emsp;&emsp;     允许的值
&emsp;&emsp;------       &emsp;&emsp;   ----------
&emsp;&emsp;分钟     &emsp;&emsp;    0-59
&emsp;&emsp;小时            &emsp;&emsp;      0-23
&emsp;&emsp;几号   &emsp;&emsp; 1-31
&emsp;&emsp;月份     &emsp;&emsp;      1-12 (或者名字)
&emsp;&emsp;星期几&emsp;&emsp;  0-7 (或者缩写名字或全名)
字段可以是星号（\*），它总是代表“第一个到最后一个”，也就是任意。

* 允许数字范围。范围是用连字符分隔的两个数字。指定的范围包括在内。例如，8-11表示“小时”条目指定在8,9,10和11小时执行。

*  列表是允许的。列表是由逗号分隔的一组数字（或范围）。例如：`1,2,5,9`或`0-4,8-12`。

* 步长值可与范围结合使用。在范围后跟随“/\<number\>”，指定在该范围内跳过数字值。例如，可以在小时字段中使用“0-23/2”来指定每隔一小时执行一次命令（V7标准中的替代方法是“0,2,4,6,8,10,12,14,16,18,20,22 ''）。星号后也允许跟随步长，所以如果你想说“每两个小时”，只需使用“\*/2”。

* 名称也可用于“月”和“星期几”字段。使用特定日期或月份的前三个字母（大小写无关紧要）。不允许使用范围或名称列表。

* “第六个”字段（行的其余部分）指定要运行的命令。该行的整个命令部分（直到换行符或％字符）将由`/bin/sh`或crontab文件的SHELL变量中指定的shell执行。除非使用反斜杠（\）进行转义，否则命令中的百分号（％）将更改为换行符，并且第一个％之后的所有数据将作为标准输入发送给命令。无法像shell尾部的“\”那样将单个命令行拆分为多行，。

* 注意：命令执行的日子可以由两个字段指定 ：月中的某天和星期几。如果两个字段都受到限制（即不是*），则当任一字段与当前时间匹配时，将运行该命令。例如，“30 4 1,15 * 5”将导致命令在每个月的1日和15日凌晨4:30以及每个星期五运行。但是，可以通过向命令添加测试来得到所需的结果（请参阅下面的EXAMPLE CRON FILE 文件中的最后一个示例）。

* 可能会出现以下八个特殊字符串中的一个而不是上诉的五个字段：
&emsp;&emsp;字符串&emsp;&emsp;含义
&emsp;&emsp;------&emsp;&emsp; -------
&emsp;&emsp;@reboot&emsp;&emsp;在启动时运行一次。
&emsp;&emsp;@yearly&emsp;&emsp;每年运行一次，`“0 0 1 1 *”`。
&emsp;&emsp;@annually&emsp;&emsp;（和@yearly一样）
&emsp;&emsp;@monthly&emsp;&emsp;每月运行一次，`“0 0 1 * *”`。
&emsp;&emsp;@weekly&emsp;&emsp;每周运行一次，`“0 0 * * 0”`。
&emsp;&emsp;@daily&emsp;&emsp;每天运行一次，`“0 0 * * *”`。
&emsp;&emsp;@midnight&emsp;&emsp;`（与@daily相同）`
&emsp;&emsp;@hourly&emsp;&emsp;每小时运行一次，`“0 * * * *”`。

* 请注意，就@reboot而言，启动是cron(8)守护进程启动的时间。特别的，他可能在某些系统守护程序或其他工具启动之前。这是由于机器的引导顺序。


**CRON文件示例**


```shell
       # use /bin/bash to run commands, instead of the default /bin/sh
       SHELL=/bin/bash
       # mail any output to `paul', no matter whose crontab this is
       MAILTO=paul
       #
       # run five minutes after midnight, every day
       5 0 * * *       $HOME/bin/daily.job >> $HOME/tmp/out 2>&1
       # run at 2:15pm on the first of every month -- output mailed to paul
       15 14 1 * *     $HOME/bin/monthly
       # run at 10 pm on weekdays, annoy Joe
       0 22 * * 1-5    mail -s "It's 10pm" joe%Joe,%%Where are your kids?%
       23 0-23/2 * * * echo "run 23 minutes after midn, 2am, 4am ..., everyday"
       5 4 * * sun     echo "run at 5 after 4 every sunday"
       # Run on every second Saturday of the month
       0 4 8-14 * *    test $(date +\%u) -eq 6 && echo "2nd Saturday"
```
**系统CRON文件示例**
```
#以下列出了普通系统crontab文件的内容。 不像用户crontab文件，该文件具有/etc/crontab使用的用户名字段。

# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow usercommand
17 * * * *  root  cd / && run-parts --report /etc/cron.hourly
25 6 * * *  root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6 * * 7  root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6 1 * *  root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```



# 4 run-parts命令

**命令**：`run-parts  [--test] [--verbose] [--report] [--lsbsysinit] [--regex=RE] [--umask=umask] [--arg=argument] [--exit-on-error] [--help] [--version] [--list] [--reverse] [--] DIRECTORY`
&emsp;&emsp;&emsp;`run-parts -V`
**描述**:
 * run-parts运行在目录DIRECTORY中找到的所有符合下述约束的所有可执行文件。其他文件和目录会被忽略。
 
 * 如果既没有给出--lsbsysinit选项也没有给出--regex选项，则名称必须完全由ASCII大写和小写字母，ASCII数字，ASCII下划线和ASCII减号连字符组成。
 
 * 如果给出了--lsbsysinit选项，则名称不得以.dpkg-old、.dpkg-dist、.dpkg-new、.dpkg-tmp结尾，并且必须属于以下一个或多个名称空间：LANANA -assigned名称空间` (^[a-z0-9]+$)`; LSB分层和保留名称空间`(^_?([a-z0-9_.]+-)+[a-z0-9]+$)`;和Debian cron脚本命名空间`（^[a-zA-Z0-9 _-]+$）`。

* 如果给出了--regex选项，则名称必须与该选项的参数的指定的自定义扩展正则表达式匹配。

* 除非给出--reverse选项，否则文件以其名称的词法排序顺序（根据C/POSIX语言环境字符排序规则）运行，在这种情况下，它们以相反的顺序运行。


|短选项|长选项|描述|
|:--|:--|:--|
||--test|打印将要运行的脚本的名称，但不实际运行它们。
||--list|打印所有匹配文件的名称（不限于可执行文件），但实际上并不运行它们。此选项不能与--test一起使用。
| -v| --verbose|在运行之前将每个脚本的名称打印到stderr。
||--report|类似于--verbose，但只打印产生输出的脚本的名称。脚本的名称将打印到脚本首先生成输出的stdout或stderr中的任何一个。
|| --reverse|反转脚本的执行顺序。
||--exit-on-error|一旦脚本以非零退出代码返回，则退出。
||--lsbsysinit| 使用LSB名称空间而不是经典行为。
||--new-session|在单独的进程会话中运行每个脚本。 如果使用此选项，则终止run-parts不会终止当前运行的脚本，它将一直运行直到完成。
||--regex=RE| 根据自定义扩展正则表达式RE验证文件名。 有关示例，请参见“示例”部分。
|-u|--umask = umask| 在运行脚本之前将umask设置为umask。 umask应该以八进制指定。 默认情况下，umask设置为022。
|-a|--arg=argument|将参数传递给脚本。 对于要传递的每个参数，都要使用--arg一次。
|--||指定这是选项的结尾。 在 - 之后的任何文件名都不会被解释为选项，即使它以连字符开头。
|-h| --help|显示使用信息并退出。
-V| --version|显示版本和版权并退出
   

   
   
------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
