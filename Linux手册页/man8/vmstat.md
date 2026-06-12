---
title: vmstat 
tags: man8,系统管理
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

<u>delay</u> 更新之间的延迟时间（以秒为单位）。如果未指定延迟时间，则仅打印一次报告，其中包含自启动以来的平均值。 

<u>count</u> 更新次数。若未指定 count，且定义了延迟时间时，默认值为无限。




------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《vmstat manpages》。</font>当前版本为4.0.4，手册更新时间为2024-07-19。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

**strong text**