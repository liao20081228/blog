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
1 列出所有PCIe根端口（以2-S平台为例）：
```
$ perf iostat list
S0-uncore_iio_0<0000:00>
S1-uncore_iio_0<0000:80>
S0-uncore_iio_1<0000:17>
S1-uncore_iio_1<0000:85>
S0-uncore_iio_2<0000:3a>
S1-uncore_iio_2<0000:ae>
S0-uncore_iio_3<0000:5d>
S1-uncore_iio_3<0000:d7>
```
2 收集所有PCIe根端口的指标：
```
$ perf iostat -- dd if=/dev/zero of=/dev/nvme0n1 bs=1M oflag=direct
357708+0 records in
357707+0 records out
375083606016 bytes (375 GB, 349 GiB) copied, 215.974 s, 1.7 GB/s

Performance counter stats for 'system wide':

   port    Inbound Read(MB)  Inbound Write(MB) Outbound Read(MB) Outbound Write(MB)
0000:00     1     0     2     3
0000:80     0     0     0     0
0000:17352552    43     0    21
0000:85     0     0     0     0
0000:3a     3     0     0     0
0000:ae     0     0     0     0
0000:5d     0     0     0     0
0000:d7     0     0     0     0

```
3 收集以下逗号分隔的PCIe根端口列表的指标：
```
$ perf iostat 0000:17,0:3a -- dd if=/dev/zero of=/dev/nvme0n1 bs=1M oflag=direct
357708+0 records in
357707+0 records out
375083606016 bytes (375 GB, 349 GiB) copied, 197.08 s, 1.9 GB/s

Performance counter stats for 'system wide':

   port             Inbound Read(MB)    Inbound Write(MB)    Outbound Read(MB)   Outbound Write(MB)
0000:17358559    44     0    22
0000:3a     3     2     0     0

197.081983474 seconds time elapsed
```
# 说明
Linux性能计数器是一个基于内核的新型子系统，为所有性能分析相关功能提供了一个框架。它涵盖了硬件层面（CPU/PMU，性能监控单元）和软件层面（软件计数器、跟踪点）的特性。

# 另请参阅
perf-stat(1)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
