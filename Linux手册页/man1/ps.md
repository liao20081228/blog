---
title: ps
tags: GNU开发工具,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《ps manpages》。</font>objdump的版本为4.0.4，手册更新时间为2023-08。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称

ps ——报告当前进程的快照。

# 描述
ps 用于显示所选活动进程的相关信息。若需要对所选进程及显示信息进行周期性更新，请改用 `top` 命令。 

本版本的 ps 支持多种类型的选项： 
1. UNIX 选项：可组合使用，**必须**以单个短横线（-）开头。 
2. BSD 选项：可组合使用，**不得**搭配短横线使用。 
3. GNU 长选项：以两个短横线（--）开头。 

不同类型的选项**允许**自由混用，但可能出现冲突。由于本 ps 兼容多种标准与实现版本，因此存在部分功能完全相同的同义选项。 

默认情况下，ps 会选择与当前用户拥有相同有效用户 ID（euid=EUID）、且与调用者关联至同一终端的所有进程。它会显示进程 ID（`pid=PID`）、进程关联的终端（`tname=TTY`）、以 `[DD-]hh:mm:ss` 格式呈现的累计 CPU 时间（`time=TIME`），以及可执行文件名称（`ucmd=CMD`）。默认输出不进行排序。 

使用 BSD 风格的选项时，会在默认显示内容中增加进程状态（`stat=STAT`），并显示命令参数（`args=COMMAND`）而非可执行文件名。你可通过环境变量 `PS_FORMAT` 覆盖该行为。同时，BSD 风格选项还会改变进程筛选规则，将你在其他终端（TTY）上的进程纳入显示范围；换而言之，这相当于筛选出所有进程（排除其他用户的进程以及无终端关联的进程）。下文将选项描述为“等效”时，不考虑上述附加效果，例如 `-M` 与 `Z` 会被视为等效，以此类推。 

除下文特别说明外，进程筛选选项为叠加生效。默认筛选规则会被舍弃，随后将符合条件的进程加入待显示进程集合。只要进程满足任一给定的筛选条件，就会被显示出来。

# 示例
使用标准语法查看系统上的所有进程： 
```sh
ps -e 
ps -ef 
ps -eF 
ps -ely 
``` 


使用 BSD 语法查看系统上的所有进程： 
``` 
ps ax 
ps axu 
``` 
打印进程树： 
``` 
ps -ejH 
ps axjf 
``` 
获取线程相关信息： 
``` 
ps -eLf 
ps axms 
``` 
获取安全相关信息： 
``` 
ps -eo euser,ruser,suser,fuser,f,comm,label 
ps axZ 
ps -eM 
``` 
以用户格式查看所有以 root 身份运行的进程（真实ID与有效ID）：
``` 
ps -U root -u root u 
``` 
使用自定义格式查看所有进程： 
``` 
ps -eo pid,tid,class,rtprio,ni,pri,psr,pcpu,stat,wchan:14,comm 
ps axo stat,euid,ruid,tty,tpgid,sess,pgrp,ppid,pid,pcpu,comm 
ps -Ao pid,tt,user,fname,tmout,f,wchan 
``` 
仅打印 syslogd 进程的进程ID： 
``` 
ps -C syslogd -o pid= 
``` 
仅打印 PID 为 42 的进程名称： 
``` 
ps -q 42 -o comm= 
```

# 简易进程选择
- a
解除 BSD 风格的 “仅自身进程” 限制。当使用部分 BSD 风格（不带 `-`）选项，或 `ps` 的个性化设置为类 BSD 时，该限制会作用于所有进程集合。 以此方式选中的进程，会追加到通过其他方式选中的进程集合中。换一种表述：该选项会让 `ps` 列出所有关联终端（tty）的进程；若与 `x` 选项一同使用，则会列出所有进程。
- -A
选择所有进程。与 `-e` 等效。
- -a
选择除会话组长进程（参见 <u>getsid </u>(2)）以及未关联终端的进程之外的所有进程。
- -d
选择除会话组长进程之外的所有进程。
- --deselect 
选择不满足指定条件的所有进程（对选择条件取反）。与 `-N` 等效。
- -e
选择所有进程。与 `-A` 等效。
- g
选择真正的全部进程，包括会话组长进程。该标记已废弃，未来版本可能移除。通常 `a` 选项已隐含此效果，仅在 sunos4 个性化下有用。
- -N
选择不满足指定条件的所有进程（对选择条件取反）。与 `--deselect` 等效。
- T 
选择与当前终端关联的所有进程。与不带参数的 `t` 选项等效。
- r 
仅选择处于运行状态的进程。
- x 
解除 BSD 风格的 “必须关联 tty” 限制。当使用部分 BSD 风格（不带 `-`）选项，或 `ps` 的个性化设置为类 BSD 时，该限制会作用于所有进程集合。 以此方式选中的进程，会追加到通过其他方式选中的进程集合中。换一种表述：该选项会让 `ps` 列出当前用户的所有进程（与 `ps` 自身 EUID 相同）；若与 `a` 选项一同使用，则会列出所有进程。

# 按列表选择进程 
这些选项接受单个参数，参数格式为空格分隔或逗号分隔的列表。此类选项可多次使用。 示例：`ps -p "1 2" -p 3,4`。

- 123
等同于 `--pid 123`。 
- +123 
等同于 `--sid 123`。 
- -123
按进程组 ID（PGID）筛选进程。 
- -C <u>cmdlist</u> 
按命令名称筛选。 筛选可执行文件名称在 <u>cmdlist </u>列表中的进程。 注意：命令名称与命令行参数不同。旧版 procps 工具与内核会将命令名截断为 15 个字符，目前两者均已取消该限制。若你之前依赖仅匹配前 15 个字符，可能会出现匹配失败。
- -G <u>grplist </u>
按真实组 ID（RGID）或组名筛选。 筛选真实组名或组 ID 在 <u>grplist </u>列表中的进程。 真实组 ID 标识创建该进程的用户所属的组，参见 <u>getgid</u>(2)。
- -g <u>grplist</u>
按会话或有效组名筛选。 按会话筛选是多项标准规定的行为，但按有效组筛选是其他部分操作系统采用的逻辑行为。 本版本 ps 在列表全为数字时（与会话格式一致），会按会话筛选； 仅当同时指定部分组名时，组 ID 数字才会生效。 参见 `-s` 与 `--group` 选项。 
- --Group <u>grplist</u> 
按真实组 ID（RGID）或组名筛选。等同于 `-G`。 
- --group <u>grplist </u>
按有效组 ID（EGID）或组名筛选。 筛选有效组名或组 ID 在<u>grplist</u>列表中的进程。 有效组 ID 决定进程使用哪一组的文件访问权限，参见 <u>getegid</u>(2)。 `-g` 选项常作为本选项的替代用法。 
- p <u>pidlist</u>
按进程 ID 筛选。等同于 `-p` 和 `--pid`。 
- -p <u>pidlist</u> 
按 PID 筛选。 筛选进程 ID 在 <u>pidlist</u> 列表中的进程。等同于 `p` 和 `--pid`。
- --pid <u>pidlist</u> 
按进程 ID 筛选。等同于 `-p` 和 `p`。 
- --ppid <u>pidlist </u>
按父进程 ID筛选。 筛选父进程 ID 在 <u>pidlist </u>列表中的进程，即筛选指定进程的子进程。
- q <u>pidlist </u>
按进程 ID 筛选（快速模式）。等同于 `-q` 和 `--quick-pid`。 
- -q <u>pidlist </u>
按 PID 筛选（快速模式）。 仅读取 <u>pidlist</u>中列出 PID 的必要信息，不应用额外过滤规则；PID 顺序保持原样、不排序。 该模式下不允许使用其他筛选选项、排序选项或树形展示格式。 等同于 `q` 和 `--quick-pid`。 
- --quick-pid <u>pidlist</u>
按进程 ID 筛选（快速模式）。等同于 `-q` 和 `q`。 
- -s <u>sesslist</u> 
按会话 ID筛选。 筛选会话 ID 在 <u>sesslist</u> 列表中的进程。 
- --sid <u>sesslist </u>
按会话 ID 筛选。等同于 `-s`。 
- t <u>ttylist </u>
按终端 tty 筛选。 与 `-t`、`--tty` 基本等效，还可使用空的 <u>ttylist</u>表示与 ps 关联的当前终端。 推荐使用 `T` 选项，比用空列表的 `t` 更规范。 
- -t <u>ttylist</u>
按终端 tty 筛选。 筛选与 <u>ttylist </u>中指定终端关联的进程。 终端（tty/文本输出终端）可写为多种格式：`/dev/ttyS1`、`ttyS1`、`S1`。 使用 `-` 可筛选未关联任何终端的进程。 
- --tty <u>ttylist</u>
按终端筛选。等同于 `-t` 和 `t`。 
- U <u>userlist </u>
按有效用户 ID（EUID）或用户名筛选。 筛选有效用户名或用户 ID 在 <u>userlist </u>列表中的进程。 有效用户 ID 决定进程使用哪一用户的文件访问权限，参见 <u>geteuid</u>(2)。 等同于 `-u` 和 `--user`。 
- -U <u>userlist</u>
 按真实用户 ID（RUID）或用户名筛选。 筛选真实用户名或用户 ID 在 <u>userlist</u>列表中的进程。 真实用户 ID 标识创建该进程的用户，参见 <u>getuid</u>(2)。 
- -u <u>userlist </u>
按有效用户 ID（EUID）或用户名筛选。 筛选有效用户名或用户 ID 在 <u>userlist</u>列表中的进程。 有效用户 ID 决定进程使用哪一用户的文件访问权限，参见 <u>geteuid</u>(2)。 等同于 `U` 和 `--user`。 
- --User <u>userlist</u>
按真实用户 ID（RUID）或用户名筛选。等同于 `-U`。 
- --user userlist
按有效用户 ID（EUID）或用户名筛选。等同于 `-u` 和 `U`。

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《ps manpages》。</font>objdump的版本为4.0.4，手册更新时间为2023-08。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
