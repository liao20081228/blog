---
title: perf-list
tags: perf,性能
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------

# 名称
perf-list —— 列出所有符号事件类型。

# 概览
<u>perf </u><u>list </u>\[\<options>] \[hw|sw|cache|tracepoint|pmu|sdt|metric|metricgroup|event_glob]

# 说明
该命令显示在各种perf命令中使用-e选项时可选择的符号事件类型。
# 选项
- -d, --desc
 打印额外的事件描述。（默认）
- --no-desc
 不打印描述。
- -v, --long-desc
 打印更详细的事件描述以及所有具有字母数字后缀的类似PMU。
- --debug
 启用调试输出。
- --details
 打印命名事件如何在内部解析为perf事件，以及perf stat计算的任何额外表达式。
- --deprecated
 打印已弃用的事件。默认情况下，已弃用的事件会被隐藏。
- --unit
 打印仅限于特定PMU名称的PMU事件和度量值。（例如：--unit cpu、--unit msr、--unit cpu_core、--unit cpu_atom）
- -j, --json
 以JSON格式输出。
- -o, --output=
 输出文件名。默认情况下，输出会写入标准输出。






# 说明
Linux性能计数器是一个基于内核的新型子系统，为所有性能分析相关功能提供了一个框架。它涵盖了硬件层面（CPU/PMU，性能监控单元）和软件层面（软件计数器、跟踪点）的特性。

# 另请参阅
perf(1) \[1] [Common Trace Format](http://www.efficios.com/ctf)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
