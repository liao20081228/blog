---
title: perf-stat
tags: perf,性能
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------

# 名称
perf-data - 运行命令并收集性能计数器统计信息

# 概览
<u>perf</u> <u>stat</u> \[-e \<EVENT> | --event=EVENT] \[-a] \<command>
<u>perf</u> <u>stat</u> \[-e \<EVENT> | --event=EVENT] \[-a] -- \<command> \[\<options>]
<u>perf</u> <u>stat</u> \[-e \<EVENT> | --event=EVENT] \[-a] record \[-o file] -- \<command> \[\<options>]
<u>perf</u> <u>stat</u> report \[-i file]

# 说明
该命令用于运行一个命令并从中收集性能计数器统计数据。

# 选项
- \<command>...
 任何你可以在 shell 中指定的命令。
- record
 参见 STAT RECORD。
- report
 参见 STAT REPORT。
- -e, --event=
选择 PMU 事件。选择可以是：
  - 一个符号事件名称（使用 <u>perf</u> <u>list</u>列出所有事件）
  - 一个原始的PMU事件，形式为rN，其中N是一个表示原始寄存器编码的十六进制值，其事件控制寄存器的布局描述在`/sys/bus/event_source/devices/cpu/format/*`的条目中。
  - 一个符号事件或原始 PMU 事件，后跟一个可选的冒号和事件修饰符列表，例如 `cpu-cycles:p`。有关事件修饰符的详细信息，请参阅 perf-list(1) 手册页。
  - 一个符号化事件，例如 `pmu/param1=0x3,param2/`，其中 `param1` 和 `param2` 被定义为`/sys/bus/event_source/devices/<pmu>/format/* `中的PMU的格式。`'percore'` 是一个事件限定符，用于汇总一个核心中两个硬件线程的事件计数。例如：
 `perf stat -A -a -e cpu/event,percore=1/,otherevent...`
  - 一个符号化的事件，例如 `pmu/config=M,config1=N,config2=K/`，其中 M、N、K 是数字（十进制、十六进制、八进制格式）。每个 <u>config</u>、<u>config1 </u>和 <u>config2 </u>参数的合法取值
 由 `/sys/bus/event_source/devices/<pmu>/format/*` 中对应的条目定义

  注意，最后两种语法支持在 PMU 名称中使用前缀和通配符匹配，以简化在大型系统中跨多个相同类型 PMU 实例（例如内存控制器 PMU）创建事件的过程。非核心 PMU 通常会有多个实例，因此在此匹配过程中也会忽略前缀'uncore_'。
- -i, --no-inherit
 子任务不继承计数器
- -p, --pid=\<pid>
 对现有进程 ID（逗号分隔列表）进行统计事件
- -t, --tid=\<tid>
 对现有线程 ID（逗号分隔列表）进行统计事件
 - -b, --bpf-prog
在现有的BPF程序ID上统计事件（以逗号分隔的列表），需要root权限。可以使用bpftool-prog来查找系统中所有BPF程序的ID。例如： 
    ```
    bpftool prog | head -n 1
    17247: tracepoint  name sys_enter  tag 192d548b9d754067  gpl
    
    # perf stat -e cycles,instructions --bpf-prog 17247 --timeout 1000
    Performance counter stats for 'BPF program(s) 17247':
    85,967      cycles
    28,982      instructions              #    0.34  insn per cycle
    1.102235068 seconds time elapsed
    ```
- --bpf-counters
使用 BPF 程序来聚合来自 perf_events 的读数。这使得多个 perf-stat 会话（它们正在统计相同的指标，如周期、指令等）可以共享硬件计数器。 要默认在常见事件上使用 BPF 程序，请使用 `"perf config stat.bpf-counter-events=<事件列表>"`。
- --bpf-attr-map、

# 说明
Linux性能计数器是一个基于内核的新型子系统，为所有性能分析相关功能提供了一个框架。它涵盖了硬件层面（CPU/PMU，性能监控单元）和软件层面（软件计数器、跟踪点）的特性。

# 另请参阅
perf(1) \[1] [Common Trace Format](http://www.efficios.com/ctf)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
