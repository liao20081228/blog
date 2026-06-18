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

# 事件修饰符
事件可以通过附加冒号和一个或多个修饰符来可选地添加修饰符。修饰符允许用户限制要统计的事件。以下是存在的修饰符：

 - u - 用户空间计数
 - k - 内核计数
 - h - 虚拟机监控程序计数
 - I - 非空闲计数
 - G - 客户机计数（在KVM客户机中）
 - H - 主机计数（不在KVM客户机中）
 - p - 精确级别
 - P - 使用检测到的最大精确级别
 - S - 读取采样值（PERF_SAMPLE_READ）
 - D - 将事件固定到PMU
 - W - 组是弱组，如果不可调度，将退化为非组
 - e - 组或事件是独占的，不共享PMU
 - b - 使用BPF聚合（参见perf stat --bpf-counters）
 - R - 事件退休延迟值
 - X - 不重新分组事件以匹配PMU
 - 
p修饰符可用于指定指令地址的精确度。p修饰符可以多次指定：
 - 0 - SAMPLE_IP可以有任意的滑移
 - 1 - SAMPLE_IP必须具有恒定的滑移
 - 2 - SAMPLE_IP请求具有0滑移
 - 3 - SAMPLE_IP必须具有0滑移，或者使用随机化来避免样本阴影效应。

在Intel系统中，精确事件采样是通过PEBS实现的，它支持精确度级别2，在某些特殊情况下支持精确度级别3。 


在AMD系统中，它是通过IBS OP实现的（精确度级别2）。与Intel PEBS提供不同精确度级别不同，AMD核心PMU本质上是非精确的，而IBS本质上是精确的。 （即ibs_op//、ibs_op//p、ibs_op//pp和ibs_op//ppp都是相同的）。精确修饰符适用于事件类型0x76（CPU周期，CPU时钟未停止）和0xC1（微操作退休）。

这两个事件都映射到IBS执行采样（IBS op），分别设置了IBS Op计数器控制位（IbsOpCntCtl）（参见【AMD处理器编程参考（PPR）】中与处理器家族、型号和步进相关的“核心复合体（CCX）→处理器x86核心→基于指令的采样（IBS）”部分）。 

《手册第二卷：系统编程》13.3节“基于指令的采样”。使用IBS的示例如下：
```
perf record -a -e cpu-cycles:p... # 使用ibs op计数周期
perf record -a -e r076:p... # 与-e cpu-cycles:p相同
perf record -a -e r0C1:p... # 使用ibs op计数微操作
```



# 说明
Linux性能计数器是一个基于内核的新型子系统，为所有性能分析相关功能提供了一个框架。它涵盖了硬件层面（CPU/PMU，性能监控单元）和软件层面（软件计数器、跟踪点）的特性。

# 另请参阅
perf(1) \[1] [Common Trace Format](http://www.efficios.com/ctf)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
