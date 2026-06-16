---
title: perf-data 
tags: perf,性能
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------

# 名称
perf-data - 数据文件相关处理

# 概览
<u>perf</u> <u>data</u>  \[\<common options>] \<command> \[\<options>]

# 说明
数据文件相关处理

# 命令
- convert
将perf数据文件转换为另一种格式。可以设置 data-convert debug 变量以获取转换过程中的调试信息，例如：perf --debug data-convert data convert...
# CONVERT选项
- --to-ctf
 触发 CTF 转换，指定 CTF 数据目录的路径。
- --to-json
 触发 JSON 转换。指定要输出的 JSON 文件名。
- --tod
 将时间转换为墙上时钟时间。
- -i
 指定输入性能数据文件的路径。
- -f, --force
 不要抱怨，直接执行。
- --time
仅转换给定时间窗口内的采样：`<开始>,<结束>`。时间格式为`秒.纳秒`。如果未给出开始时间（即时间字符串为`，x.y`），则分析从文件开头开始。如果未给出结束时间（即时间字符串为`x.y，`），则分析持续到文件末尾。多个时间范围可以用空格分隔，此时需要将参数用引号括起来，例如：
 `--time "1234.567,1234.789 1235,"`，也支持使用时间百分比来表示多个时间范围。时间字符串格式为`'a%/n,b%/m,...'` 或 `'a%-b%,c%-%d,...'`。例如：
&emsp;&emsp;选择第二个 10% 的时间片：
&emsp;&emsp;`perf data convert --to-json out.json --time 10%/2`
&emsp;&emsp;选择从 0% 到 10% 的时间片：
&emsp;&emsp;`perf data convert --to-json out.json --time 0%-10%`
&emsp;&emsp;选择第一个和第二个 10% 的时间片：
&emsp;&emsp;`perf data convert --to-json out.json --time 10%/1,10%/2`
&emsp;&emsp;选择从 0% 到 10% 和从 30% 到 40% 的时间片：
&emsp;&emsp;`perf data convert --to-json out.json --time 0%-10%,30%-40%`
- -v, --verbose
 显示更多信息（显示计数器打开错误等）。
- --all
 转换所有事件，包括非采样事件（comm、fork 等）到输出。默认关闭，仅转换采样事件。
# 说明
Linux性能计数器是一个基于内核的新型子系统，为所有性能分析相关功能提供了一个框架。它涵盖了硬件层面（CPU/PMU，性能监控单元）和软件层面（软件计数器、跟踪点）的特性。

# 另请参阅
perf(1) \[1] [Common Trace Format](http://www.efficios.com/ctf)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
