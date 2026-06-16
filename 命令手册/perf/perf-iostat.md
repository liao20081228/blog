---
title: perf-iostat 
tags: perf,性能
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------

# 名称
perf-iostat- 显示 I/O 性能指标

# 概览
<u>perf</u> iostat</u>  list
<u>perf</u> iostat</u>  \<ports> -- \<command> \[\<options>]

# 说明
模式旨在为每个PCIe根端口提供四个I/O性能指标：
- 入站读取 - 根端口下的I/O设备从主机内存读取数据，单位为MB
- 入站写入 - 根端口下的I/O设备向主机内存写入数据，单位为MB
- 出站读取 - CPU从根端口下的I/O设备读取数据，单位为MB
- 出站写入 - CPU向根端口下的I/O设备写入数据，单位为MB

# 选项
- \<command>...
您可以在 shell 中指定的任何命令。
- list
 列出所有 PCIe 根端口
- \<ports>
选择要监控的根端口。支持以逗号分隔的列表。

# 示例

# 说明
Linux性能计数器是一个基于内核的新型子系统，为所有性能分析相关功能提供了一个框架。它涵盖了硬件层面（CPU/PMU，性能监控单元）和软件层面（软件计数器、跟踪点）的特性。

# 另请参阅
perf-stat(1)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
