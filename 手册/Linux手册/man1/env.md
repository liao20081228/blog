---
title: env
tags: 用户命令man1,通用命名手册
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《env manpages》。</font>journalctl的版本为0.10.0，手册更新时间为2026-09。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称
 env——在环境中将每个 NAME 设置为 VALUE，并运行 COMMAND

# 概要
**env** \[-**i**|--**ignore-environment**] \[-**C**|--**chdir**] \[-**0**|--**null**] \[-**f**|--**file**] \[-**u**|--**unset**] \[-**v**|--**debug**]... \[-**S**|--**split-string**] \[-**a**|--**argv0**] \[--**ignore-signal**] \[--**default-signal**] \[--**block-signal**] \[--**list-signal-handling**] \[-**h**|--**help**] \[-**V**|--**version**] \[vars]

**#描述**

在环境中将每个 NAME 设置为 VALUE，并运行 COMMAND

**选项**
- **-i, --ignore-environment**
以空环境开始
- **-C, --chdir \<DIR>**
将工作目录更改为 DIR
- **-0, --null**
每个输出行以 0 字节而不是换行符结束（仅在打印环境时有效）
- **-f, --file \<PATH>**
从“.env”风格的配置文件中读取并设置变量（在任何 unset 和/或 set 之前）
- **-u, --unset \<NAME>**
从环境中移除变量
- **-v, --debug**
为每个处理步骤打印详细信息
- **-S, --split-string \<S>**
处理 S 并将其拆分为单独的参数；用于在 shebang 行上传递多个参数
- **-a, --argv0 \<a>**
覆盖传递给被执行命令的第零个参数。未使用此选项时，默认值为 `command`。
- **--ignore-signal\[=\<SIG>]**
将 SIG 信号的处理设置为不执行任何操作
- **--default-signal\[=\<SIG>]**
 将 SIG 信号的处理重置为默认操作
- **--block-signal\[=\<SIG>]**
在运行 COMMAND 时阻止 SIG 信号递送
-- **--list-signal-handling**
列出前面选项请求的信号处理更改
- **-h, --help**
打印帮助
- **-V, --version**
打印版本
- \[vars]

**补充**
单独的 `-` 意味着 `-i`。如果没有 COMMAND，则打印结果环境。
# 版本
       v(uutils coreutils) 0.10.0

# 示例
显示环境：
```bash
env
```
运行程序。常用于脚本中 shebang（#!）之后，用于查找程序的路径：
```bash
env program
```
清空环境并运行程序：
```bash
env [-i|--ignore-environment] program
```
从环境中移除变量并运行程序：
```bash
env [-u|--unset] variable program
```
设置一个变量并运行程序：
```bash
env variable=value program
```
设置一个或多个变量并运行程序：
```bash
env variable1=value variable2=value variable3=value ... program
```
以不同名称运行程序：
```bash
env [-a|--argv0] custom_name program
```
 这些示例由 tldr-pages 项目 <https://tldr.sh> 提供，采用 CC BY 4.0 许可证。请注意，由于 uutils 仍在开发中，某些示例可能会失败。


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《env manpages》。</font>journalctl的版本为0.10.0，手册更新时间为2026-09。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
