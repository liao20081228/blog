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
- --bpf-attr-map
使用“--bpf-counters”选项时，不同的perf-stat会话通过一个固定的哈希表共享关于共享BPF程序和映射的信息。使用“--bpf-attr-map”来指定这个固定哈希表的路径。默认路径是/sys/fs/bpf/perf_attr_map。
- -a, --all-cpus
系统范围内从所有CPU收集（如果未指定目标则为默认设置）
- --no-scale
不要缩放/标准化计数器值。
- -d, --detailed
打印更详细的统计数据，最多可指定3次。
-d:          详细事件，L1和LLC数据缓存。
-d -d:     更详细的事件，dTLB 和 iTLB 事件
 -d -d -d:     非常详细的事件，添加了预取事件。
- -r, --repeat=\<n>
重复执行命令并打印平均值和标准差（最多100次）。0表示无限循环。
- -B, --big-num
根据本地化设置，使用千位分隔符打印大数字。默认启用。使用“--no-big-num”可禁用。默认设置可通过“perf config stat.big-num=false”更改。
- -C, --cpu=
所提供的CPU列表计数。多个CPU可以以逗号分隔的形式提供，中间不加空格：`0,1`。CPU范围使用“-”指定：`0-2`。在单线程CPU模式下，此选项将被忽略。仍需使用-a选项来激活系统范围的监控。默认情况下，将对所有CPU进行计数。
- -A, --no-aggr
不要汇总所有被监控CPU的计数。
- -n, --null
空运行 - 不要启动任何计数器。
这可以用于仅测量经过的墙上时钟时间，或者在不运行任何计数器的情况下评估perf stat本身的原始开销。
- -v, --verbose
更详细地显示（例如显示计数器打开错误等）
- -x SEP, --field-separator SEP
使用CSV风格的输出打印计数，以便直接导入电子表格。列之间由SEP中指定的字符串分隔。 
- --table
每次运行（-r 选项）显示时间，以表格格式呈现，例如：
    ```
    $ perf stat --null -r 5 --table perf bench sched pipe

               Performance counter stats for 'perf bench sched pipe' (5 runs):

               # Table of individual measurements:
               5.189 (-0.293) #
               5.189 (-0.294) #
               5.186 (-0.296) #
               5.663 (+0.181) ##
               6.186 (+0.703) ####

               # Final result:
               5.483 +- 0.198 seconds time elapsed  ( +-  3.62% )

    ```
- -G name, --cgroup name
仅在名为“name”的容器（cgroup）中进行监控。此选项仅在 per-cpu 模式下可用。必须挂载 cgroup 文件系统。当属于容器“name”的所有线程在被监控的 CPU 上运行时，它们将被监控。可以提供多个 cgroup。每个 cgroup 应用于相应的事件，即第一个 cgroup 应用于第一个事件，第二个 cgroup 应用于第二个事件，依此类推。可以使用例如 `-G foo,,bar` 来提供一个空的 cgroup（始终监控所有内容）。cgroup 必须具有对应的事件，即它们始终引用在命令行中先前定义的事件。如果用户希望跟踪特定 cgroup 的多个事件，可以使用 `-e e1 -e e2 -G foo,foo`，或仅使用 `-e e1 -e e2 -G foo`。

    如果想要监控某个cgroup的周期计数以及系统全局的周期计数，可以使用以下命令行：`perf stat -e cycles -G cgroup_name -a -e cycles`。
- --for-each-cgroup name
扩展每个使用 "name"（允许以逗号分隔的多个 cgroup）的 cgroup 的事件列表。它还支持正则表达式模式以匹配多个组。此选项与为每个事件 x 名称重复使用 -e 选项和 -G 选项具有相同效果。此选项不能与 -G/--cgroup 选项一起使用。
- -o file, --output file
将输出打印到指定文件。
- --append
将内容追加到使用 -o 选项指定的输出文件中。如果未指定 -o，则忽略此选项。
- --log-fd
将日志输出到文件描述符，而不是标准错误输出。与 --output 选项互补，且二者互斥。此处可使用 --append 选项。示例： 
`3>results perf stat --log-fd 3 -- $cmd `
`3>>results perf stat --log-fd 3 --append -- $cmd`
- --control=fifo:ctl-fifo\[,ack-fifo], --control=fd:ctl-fd\[,ack-fd]
ctl-fifo / ack-fifo 被打开并用作 ctl-fd / ack-fd，如下所示。在 ctl-fd 描述符上监听控制测量的命令（<u>enable</u>：启用事件，<u>disable</u>：禁用事件）。可以使用 `--delay=-1` 选项在禁用事件的情况下启动测量。可选地，向 ack-fd 描述符发送控制命令完成（ack\n）以与控制进程同步。以下是一个 bash shell 脚本示例，用于在测量期间启用和禁用事件：

    ````
               #!/bin/bash
               ctl_dir=/tmp/
               ctl_fifo=${ctl_dir}perf_ctl.fifo
               test -p ${ctl_fifo} && unlink ${ctl_fifo}
               mkfifo ${ctl_fifo}
               exec {ctl_fd}<>${ctl_fifo}
               ctl_ack_fifo=${ctl_dir}perf_ctl_ack.fifo
               test -p ${ctl_ack_fifo} && unlink ${ctl_ack_fifo}
               mkfifo ${ctl_ack_fifo}
               exec {ctl_fd_ack}<>${ctl_ack_fifo}
               perf stat -D -1 -e cpu-cycles -a -I 1000       \
                         --control fd:${ctl_fd},${ctl_fd_ack} \
                         \-- sleep 30 &
               perf_pid=$!
               sleep 5  && echo 'enable' >&${ctl_fd} && read -u ${ctl_fd_ack} e1 && echo "enabled(${e1})"
               sleep 10 && echo 'disable' >&${ctl_fd} && read -u ${ctl_fd_ack} d1 && echo "disabled(${d1})"
               exec {ctl_fd_ack}>&-
               unlink ${ctl_ack_fifo}
               exec {ctl_fd}>&-
               unlink ${ctl_fifo}
               wait -n ${perf_pid}
    exit $?
    ```

- --pre, --post







# 说明
Linux性能计数器是一个基于内核的新型子系统，为所有性能分析相关功能提供了一个框架。它涵盖了硬件层面（CPU/PMU，性能监控单元）和软件层面（软件计数器、跟踪点）的特性。

# 另请参阅
perf(1) \[1] [Common Trace Format](http://www.efficios.com/ctf)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
