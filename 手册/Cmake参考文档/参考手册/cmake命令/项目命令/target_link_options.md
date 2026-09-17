---
title: target_link_options
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
*3.13 版本新增*。

为可执行文件、共享库或模块库目标添加链接阶段的选项。

```cmake
target_link_options(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
指定的`<target>`必须是通过 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable") 或 [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library") 等命令创建的，并且不能是[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)。

此命令可用于添加任何链接选项，但也可以使用其他命令来添加库[target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries)或 [link_libraries()](https://cmake.org/cmake/help/latest/command/link_libraries.html#command:link_libraries)。有关详细信息，请参阅[目录](https://cmake.org/cmake/help/latest/prop_dir/LINK_OPTIONS.html#prop_dir:LINK_OPTIONS)和[目标](https://cmake.org/cmake/help/latest/prop_tgt/LINK_OPTIONS.html#prop_tgt:LINK_OPTIONS)`LINK_OPTIONS`属性的文档。

>注意：此命令不能用于为静态库目标添加选项，因为静态库不使用链接器。要添加归档工具或 MSVC 库工具的编译标志，请参见 [STATIC_LIBRARY_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/STATIC_LIBRARY_OPTIONS.html#prop_tgt:STATIC_LIBRARY_OPTIONS) 目标属性。

如果指定了 `BEFORE`，则内容将被添加到属性之前，而不是追加到属性末尾。

`INTERFACE`、`PUBLIC` 和 `PRIVATE` 关键字用于指定随后参数的[作用范围](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#target-command-scope)。`PRIVATE` 和 `PUBLIC` 项将填充 `<target>` 的 `LINK_OPTIONS` 属性。`PUBLIC` 和 `INTERFACE` 项将填充 `<target>` 的 `INTERFACE_LINK_OPTIONS` 属性。随后参数指定链接选项。对同一 `<target>` 重复调用该命令时，参数将按调用顺序追加。

>注意：[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)仅支持 `INTERFACE` 项。

`target_link_options` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参阅 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7)) 手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7)) 手册。

# 主机与设备特定链接选项
*在3.18版本中新增*：当涉及设备链接步骤时（该步骤由[CUDA_SEPARABLE_COMPILATION](https://cmake.org/cmake/help/latest/prop_tgt/CUDA_SEPARABLE_COMPILATION.html#prop_tgt:CUDA_SEPARABLE_COMPILATION)和[CUDA_RESOLVE_DEVICE_SYMBOLS](https://cmake.org/cmake/help/latest/prop_tgt/CUDA_RESOLVE_DEVICE_SYMBOLS.html#prop_tgt:CUDA_RESOLVE_DEVICE_SYMBOLS)属性以及[CMP0105](https://cmake.org/cmake/help/latest/policy/CMP0105.html#policy:CMP0105)策略控制），原始选项将被传递给主机和设备链接步骤（设备链接步骤中的选项会使用`-Xcompiler`或等效的参数进行包装）。使用[$<DEVICE_LINK:...>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:DEVICE_LINK)生成器表达式包装的选项仅用于设备链接步骤；而使用[$<HOST_LINK:...>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:HOST_LINK)生成器表达式包装的选项则仅用于主机链接步骤。

# 选项去重
用于目标的最终选项集是通过从当前目标及其依赖项的使用要求中累积选项而构建的。为了避免重复，这些选项会被去重处理。 


*从 3.12 版本新增*：虽然对单个选项来说，去重是有益的，但去重步骤可能会破坏选项组。例如，`-option A -option B `会变成 `-option A B`。可以使用类似 shell 的引号且带有`SHELL:` 前缀来指定一组选项。`SHELL:` 前缀会被移除，其余选项字符串将使用 [separate_arguments()](https://cmake.org/cmake/help/latest/command/separate_arguments.html#command:separate_arguments "separate_arguments") `UNIX_COMMAND` 模式进行解析。例如，`"SHELL:-option A" "SHELL:-option B"` 会变成 `-option A -option B`。

# 处理编译器驱动程序差异
为了向链接器工具传递选项，每个编译器驱动程序都有自己特定的语法。可以使用 `LINKER:` 前缀和 `,` 分隔符以一种可移植的方式指定要传递给链接器工具的选项。`LINKER:` 会被替换为相应的驱动程序选项，而 `,` 则会被替换为相应的驱动程序分隔符。驱动程序前缀和分隔符的值由变量 [CMAKE\_\<LANG\>\_LINKER\_WRAPPER\_FLAG](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_LINKER_WRAPPER_FLAG.html#variable:CMAKE_%3CLANG%3E_LINKER_WRAPPER_FLAG) 和  [CMAKE\_\<LANG\>\_LINKER\_WRAPPER\_FLAG\_SEP](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_LINKER_WRAPPER_FLAG_SEP.html#variable:CMAKE_%3CLANG%3E_LINKER_WRAPPER_FLAG_SEP) 决定。

例如，对于 `Clang`，`"LINKER:-z,defs"` 会被替换为 `-Xlinker -z -Xlinker defs`；而对于 GNU GCC，则会替换为 `-Wl,-z,defs`。

`LINKER:` 前缀可以作为 `SHELL:` 前缀表达式的一部分来指定。

`LINKER:` 前缀还支持另一种语法，即使用 `SHELL:` 前缀、用空格作为分隔符来指定参数。因此，前面的例子也可以写成 `"LINKER:SHELL:-z defs"`。

>注意：不支持在 `LINKER:` 前缀的开头以外的任何位置指定 `SHELL:` 前缀。

# 另请参阅
- [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions")
- [target_compile_features()](https://cmake.org/cmake/help/latest/command/target_compile_features.html#command:target_compile_features "target_compile_features")
- [target_compile_options()](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options "target_compile_options")
- [target_include_directories()](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")
- [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")
- [target_link_directories()](https://cmake.org/cmake/help/latest/command/target_link_directories.html#command:target_link_directories)
- [target_precompile_headers()](https://cmake.org/cmake/help/latest/command/target_precompile_headers.html#command:target_precompile_headers "target_precompile_headers")
- [target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources "target_sources")
- [CMAKE\_\<LANG\>\_FLAGS](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html#variable:CMAKE_%3CLANG%3E_FLAGS) 和 [CMAKE\_\<LANG\>\_FLAGS\_\<CONFIG\>](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS_CONFIG.html#variable:CMAKE_%3CLANG%3E_FLAGS_%3CCONFIG%3E) 用于添加传递给所有编译器调用的全局语言标志。这包括驱动编译的调用以及驱动链接的调用。 
- [CheckLinkerFlag](https://cmake.org/cmake/help/latest/module/CheckLinkerFlag.html#module:CheckLinkerFlag) 模块用于检查链接器标志是否被编译器支持。
 
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------