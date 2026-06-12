---
title: vmstat 
tags: man8，系统管理
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《vmstat manpages》。</font>当前版本为4.0.4，手册更新时间为2024-07-19。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称
mount——挂载一个文件系统。

# 概览
vmstat \[options] \[<u>delay</u> \[<u>count</u>]]
# 说明
vmstat 报告有关进程、内存、分页、块 I/O、陷阱、磁盘和 CPU 活动的信息。

生成的第一个报告会给出自上次重启以来的平均值。后续报告则会提供长度为<u>delay</u>的采样周期内的信息。无论哪种情况，进程和内存报告都是即时生成的。

# 选项
- <u>delay</u> 
更新间的延迟时间（以秒为单位）。如果未指定<u>delay</u>时间，则仅打印一次报告，其中包含自启动以来的平均值。  
- <u>count</u> 
更新次数。若未指定 <u>count</u>，且定义了<u>delay</u>时间时，默认值为无限。 
- -a，--active 
显示活动和非活动内存，前提是内核版本为2.5.41或更高。
- -f，--forks
-f 开关显示自启动以来的forks数量。这包括 fork、vfork 和 clone 系统调用，等同于创建的任务总数。每个进程由一个或多个任务表示，具体取决于线程的使用情况。此显示不会重复。 
- -m，-slabs
显示slabinfo信息。
- -n，-one-header
仅显示一次标题头，而不是周期显示
- -s，--stats
显示各种事件计数器和内存统计信息的表格。此显示不会重复。
- -d，-disk
 报告磁盘统计信息（需要 2.5.70 或更高版本）。
- -D，--disk-sum
报告一些关于磁盘活动的汇总统计数据。  
- -p，--partition <u>device</u>
- 分区详细统计信息（需要2.5.70及以上版本）。  
- -S，--unit <u>character</u>
在1000（<u>k</u>）、1024（<u>K</u>）、1000000（<u>m</u>）或1048576（<u>M</u>）字节之间切换输出。请注意，这不会改变块（bi/bo）字段。|
- -t，--timestamp
 在每行末尾添加时间戳
- -w，--wide
宽输出模式（适用于内存较大的系统，因为默认输出模式可能会出现不必要的列断行问题）。该模式的输出每行宽度超过80个字符。|
- -y，--no-first 
省略自系统启动以来的第一个统计报告。  
- -V，--version
显示版本信息并退出。
- -h，--help
显示帮助并退出。 


# VM模式下的字段说明
Procs
- r: 可运行进程（正在运行或等待运行时间）的数量。
- b: 因等待 I/O 完成而阻塞的进程数量。

Memory：这些值受 --unit 选项影响。
- swpd: 交换内存使用量。
- free: 空闲内存量。
- buff: 用作缓冲区的内存量。
- cache: 用作缓存的内存量。
- inact: 不活跃内存量。（-a 选项）
- active: 活跃内存量。（-a 选项）

Swap: 这些值受 --unit 选项影响。
- si: 从磁盘交换入的内存量（每秒）。
- so: 交换到磁盘的内存量（每秒）。
  
IO
- bi: 从块设备接收的千字节数（每秒千字节）。
- bo: 发送到块设备的千字节数（每秒千字节）。
 
System
- in: 每秒中断次数，包括时钟中断。
- cs: 每秒上下文切换次数。

CPU: 这些是 CPU 总时间的百分比。
- us: 运行非内核代码所花费的时间。（用户时间，包括 nice 时间）
- sy: 运行内核代码所花费的时间。（系统时间）
- id: 空闲时间。在 Linux 2.5.41 之前，此值包含 IO 等待时间。
- wa: 等待 IO 所花费的时间。在 Linux 2.5.41 之前，此值包含在空闲时间中。
- st: 从虚拟机中被窃取的时间。在 Linux 2.6.11 之前，此值未知。
- gu: 运行 KVM 客户机代码所花费的时间（客户机时间，包括客户机 nice 时间）。

# DISK模式下的字段说明
Reads
- total: 成功完成的总读取次数
- merged: 合并的读取次数（合并为一次 I/O）
- sectors: 成功读取的扇区数
- ms: 读取所花费的毫秒数
 
Writes
- total: 成功完成的总写入次数
- merged: 合并的写入次数（合并为一次 I/O）
- sectors: 成功写入的扇区数
- ms: 写入所花费的毫秒数

I/O
- cur: 正在进行的 I/O 数量
- s: I/O 所花费的秒数
- 
# DISK PARTITION模式下的字段说明
- reads：向该分区发出的总读取次数 
- read sectors：该分区的总读取扇区数 
- writes：向该分区发出的总写入次数 
- requested writes：为该分区发出的总写入请求次数
# SLAB模式下的字段说明
Slab 模式显示每个 slab 的统计信息，有关此信息的更多详情请参阅 slabinfo(5) 

- cache: 缓存名称 
- num: 当前活动对象的个数 
- total: 可用对象的总数 
- size: 每个对象的大小 
- pages: 至少包含一个活动对象的页面数量

# 注意
vmstat 需要读取 `/proc` 下的文件。-m 选项需要读取 `/proc/slabinfo` 的权限，而标准用户可能无法访问该文件。`/proc` 的挂载选项（如 `subset=pid`）也可能影响可见内容。

# 另请参阅
free(1)， iostat(1)， mpstat(1)， ps(1)， sar(1)， top(1)， slabinfo(5)

# 报告BUGS
请将错误报告发送至 procps@freelists.org。


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《vmstat manpages》。</font>当前版本为4.0.4，手册更新时间为2024-07-19。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
