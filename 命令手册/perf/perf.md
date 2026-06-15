---
title: perf 
tags: perf,性能
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------

# 名称
perf - Linux性能分析工具

# 概览
<u>perf</u> \[--version] \[--help] \[OPTIONS] COMMAND \[ARGS]

# 选项
- -h, --help
 运行 perf help 命令。
- -v, --version
 显示 perf 版本。
- -vv
 打印编译时库的状态。
- --exec-path
 显示或设置可执行文件路径。
- --html-path
 显示 HTML 文档路径。
- -p, --paginate
 设置分页器。
- --no-pager
 不设置分页器。
- --buildid-dir
 设置 buildid 缓存目录。其优先级高于 buildid.dir 配置文件选项。
- --list-cmds
 列出最常用的 perf 命令。
- --list-opts
 列出可用的 perf 选项。
- --debugfs-dir
 设置 debugfs 目录或设置环境变量 PERF_DEBUGFS_DIR。
- --debug
 在取值范围（0, 10）内设置debug变量（见下方列表）。使用方法如下：
	``` 
	--debug verbose # 设置 verbose = 1
	--debug verbose=2 # 设置 verbose = 2
	```
	允许设置的调试变量列表：
  - verbose - 一般调试信息
  - ordered-events - 有序事件对象的调试信息
  - data-convert - 数据转换命令的调试信息
  - stderr - 在浏览模式下将调试输出（选项 -v）写入标准错误输出
  - perf-event-open - 打印 perf_event_open() 的参数和返回值
  - kmaps - 打印内核和模块映射（不带浏览的 perf script 和 perf report）
- --debug-file
写debug输出到特定文件。
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《perf manpages》</font>。当前版本为7.0.0，手册更新时间为2026-05-25。<font color=red>本文与原文档采用相同的版权许可</font>。<font color=blue>转载请注明出处！！！</font>***

------
