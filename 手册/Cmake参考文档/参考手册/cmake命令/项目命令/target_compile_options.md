---
title: target_compile_options
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
为目标添加编译选项。
```cmake
target_compile_options(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
将选项添加到目标属性 [COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_OPTIONS.html#prop_tgt:COMPILE_OPTIONS "COMPILE_OPTIONS") 或 [INTERFACE_COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_COMPILE_OPTIONS.html#prop_tgt:INTERFACE_COMPILE_OPTIONS "INTERFACE_COMPILE_OPTIONS") 中。这些选项在编译指定`<target>`时会被使用，该目标必须是通过 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable") 或 [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library") 等命令创建的，并且不能是[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)。

>注意：这些选项不会在链接目标时使用。如需链接时使用选项，请参见 [target_link_options()](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options "target_link_options") 命令。
# 参数
如果指定了 `BEFORE`，则内容将被添加到属性的开头而不是末尾。有关在某些情况下是否忽略 `BEFORE` 的情况，请参见策略 [CMP0101](https://cmake.org/cmake/help/latest/policy/CMP0101.html#policy:CMP0101 "CMP0101")。

`INTERFACE`、`PUBLIC` 和 `PRIVATE` 关键字用于指定后续参数的作用域。`PRIVATE` 和 `PUBLIC` 项将填充 `<target>` 的 [COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_OPTIONS.html#prop_tgt:COMPILE_OPTIONS "COMPILE_OPTIONS") 属性。`PUBLIC` 和 `INTERFACE` 项将填充 `<target>` 的 [INTERFACE_COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_COMPILE_OPTIONS.html#prop_tgt:INTERFACE_COMPILE_OPTIONS "INTERFACE_COMPILE_OPTIONS") 属性。后续参数用于指定编译选项。对同一个 `<target>` 重复调用这些参数时，会按调用顺序追加各项。

*3.11 版本新增功能*：允许在[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)上设置 `INTERFACE` 项。

`target_compile_options` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参见 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)") 手册。有关定义构建系统属性的更多信息，请参见 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)") 手册。
# 选项去重
最终用于目标的选项集是通过从当前目标及其依赖项的使用要求中累积选项而构建的。选项集会进行去重处理，以避免重复。

*从 3.12 版本开始*：虽然对单个选项来说，去重是有益的，但去重步骤可能会破坏选项组。例如，`-option A -option B `会变成 `-option A B`。可以使用类似 shell 的引号且带有`SHELL:` 前缀来指定一组选项。`SHELL:` 前缀会被移除，其余选项字符串将使用 [separate_arguments()](https://cmake.org/cmake/help/latest/command/separate_arguments.html#command:separate_arguments "separate_arguments") `UNIX_COMMAND` 模式进行解析。例如，`"SHELL:-option A" "SHELL:-option B"` 会变成 `-option A -option B`。
# 另请参阅
- 此命令可用于添加任何选项。然而，对于添加预处理器定义和包含目录，建议使用更具体的命令 [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions") 和 [target_include_directories()](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")。
- 对于整个目录的设置，可以使用命令 [add_compile_options()](https://cmake.org/cmake/help/latest/command/add_compile_options.html#command:add_compile_options "add_compile_options")。
- 对于特定文件的设置，可以使用源文件属性 [COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_sf/COMPILE_OPTIONS.html#prop_sf:COMPILE_OPTIONS "COMPILE_OPTIONS")。
- 此命令为目标中的所有语言添加编译选项。使用[COMPILE_LANGUAGE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:COMPILE_LANGUAGE "COMPILE_LANGUAGE")生成器表达式来指定针对每种语言的编译选项。
- [target_compile_features()](https://cmake.org/cmake/help/latest/command/target_compile_features.html#command:target_compile_features "target_compile_features")
- [target_include_directories()](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")
- [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")
- [target_link_directories()](https://cmake.org/cmake/help/latest/command/target_link_directories.html#command:target_link_directories "target_link_directories")
- [target_link_options()](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options "target_link_options")
- [target_precompile_headers()](https://cmake.org/cmake/help/latest/command/target_precompile_headers.html#command:target_precompile_headers "target_precompile_headers")
- [target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources "target_sources")
- [CMAKE\_\<LANG>_FLAGS](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html#variable:CMAKE_%3CLANG%3E_FLAGS "CMAKE_<LANG>_FLAGS") 和 [CMAKE\_\<LANG>_FLAGS_\<CONFIG>](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS_CONFIG.html#variable:CMAKE_%3CLANG%3E_FLAGS_%3CCONFIG%3E "CMAKE_<LANG>_FLAGS_<CONFIG>") 用于添加传递给所有编译器调用的语言范围的编译器标志。这包括驱动编译和链接的调用。
- 可以使用 [CheckCompilerFlag](https://cmake.org/cmake/help/latest/module/CheckCompilerFlag.html#module:CheckCompilerFlag "CheckCompilerFlag") 模块来检查编译器是否支持某个给定的标志。
 
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------