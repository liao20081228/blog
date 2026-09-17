---
title: add_link_options
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

*3.13 版本新增*。

为本命令调用之后，在当前目录及子目录中新增的可执行文件、共享库或模块库目标，向其链接阶段添加选项。

```cmake
add_link_options(<option> ...)
```

本命令可用于添加任意链接选项。若要添加库文件，请使用替代命令：[target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) 或 [link_libraries](https://cmake.org/cmake/help/latest/command/link_libraries.html#command:link_libraries)。参见[directory](https://cmake.org/cmake/help/latest/prop_dir/LINK_OPTIONS.html#prop_dir:LINK_OPTIONS) 与[target](https://cmake.org/cmake/help/latest/prop_tgt/LINK_OPTIONS.html#prop_tgt:LINK_OPTIONS) `LINK_OPTIONS`属性的文档。

>**注意**：本命令不能用于为静态库目标添加选项，因为静态库不会调用链接器。如需为归档工具或 MSVC 库管理器设置标志，请查阅目标属性 [STATIC_LIBRARY_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/STATIC_LIBRARY_OPTIONS.html#prop_tgt:STATIC_LIBRARY_OPTIONS)。

`add_link_options` 的参数可以使用生成器表达式，语法为 `$<...>`。可用生成器表达式参见 [cmake‑generator‑expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7)) 手册；关于构建系统属性定义的更多内容参见 [cmake‑buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7)) 手册。

# 主机与设备专属链接选项

*3.18 版本新增*。 当涉及设备链接步骤时（由目标属性 [CUDA_SEPARABLE_COMPILATION](https://cmake.org/cmake/help/latest/prop_tgt/CUDA_SEPARABLE_COMPILATION.html#prop_tgt:CUDA_SEPARABLE_COMPILATION)、[CUDA_RESOLVE_DEVICE_SYMBOLS](https://cmake.org/cmake/help/latest/prop_tgt/CUDA_RESOLVE_DEVICE_SYMBOLS.html#prop_tgt:CUDA_RESOLVE_DEVICE_SYMBOLS) 以及策略 [CMP0105](https://cmake.org/cmake/help/latest/policy/CMP0105.html#policy:CMP0105) 控制），原始选项会同时传递给主机与设备链接步骤（设备链接时会被包装为 `-Xcompiler` 或等效参数）。 被生成器表达式[\$\<DEVICE_LINK:...>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:DEVICE_LINK) 包裹的选项仅用于设备链接步骤。 
被生成器表达式 [\$\<HOST_LINK:...>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:HOST_LINK) 包裹的选项，仅用于主机链接步骤。

# 选项去重

目标最终使用的链接选项，由目标自身选项及其依赖的使用要求累积得到。选项集合会执行去重，避免重复。

*3.12 版本新增*：虽然去重对单独选项有益，但该过程可能拆散选项组。 例如 `-option A -option B` 会被处理成 `-option A B`。 可以使用类似 Shell 的引号配合 `SHELL:` 前缀来指定一组选项。`SHELL:` 前缀会被丢弃，剩余字符串按照 [separate_arguments](https://cmake.org/cmake/help/latest/command/separate_arguments.html#command:separate_arguments) 的 `UNIX_COMMAND` 模式解析。 示例：`"SHELL:-option A" "SHELL:-option B"` 最终得到 `-option A -option B`。

# 处理不同编译器驱动的差异

向链接器传递选项时，不同编译器驱动拥有各自语法。可使用 `LINKER:` 前缀加逗号分隔符，以可移植方式指定要传递给链接器的选项。 `LINKER:` 会被替换为对应编译器驱动的包装标志，逗号会替换为驱动对应的分隔符。该包装标志与分隔符取自变量 [\<LANG>\_LINKER\_WRAPPER\_FLAG](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_LINKER_WRAPPER_FLAG.html#variable:CMAKE_%3CLANG%3E_LINKER_WRAPPER_FLAG)、[\<LANG>\_LINKER\_WRAPPER\_FLAG\_SEP](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_LINKER_WRAPPER_FLAG_SEP.html#variable:CMAKE_%3CLANG%3E_LINKER_WRAPPER_FLAG_SEP)。

示例：`"LINKER:-z,defs"`

* 对于 Clang，展开为：`-Xlinker -z -Xlinker defs`
* 对于 GNU GCC，展开为：`-Wl,-z,defs`

`LINKER:` 前缀内部可以嵌套使用 `SHELL:` 前缀表达式。 `LINKER:` 支持另一种语法：搭配 `SHELL:` 前缀、以空格作为参数分隔。上面示例等价写法：`"LINKER:SHELL:-z defs"`。


> 注意：不支持在 `LINKER:` 前缀以外的任意位置写 `SHELL:` 前缀。

# 另请参阅

*   `link_libraries`
*   `target_link_libraries`
*   `target_link_options`
*   变量 `<LANG>_FLAGS`、`<LANG>_FLAGS_<CONFIG>`：添加语言全局标志，作用于编译器的全部调用，包含编译阶段与链接阶段。

4.3 版本新增： 变量 `<LANG>_LINK_FLAGS`、`<LANG>_LINK_FLAGS_<CONFIG>`：添加语言全局标志，仅作用于编译器执行链接的调用。


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------