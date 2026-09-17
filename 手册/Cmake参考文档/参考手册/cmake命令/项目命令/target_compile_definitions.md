---
title: target_compile_definitions
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
为目标添加编译定义。
```cmake
target_compile_definitions(<target>
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
指定在编译给定`<target>`时要使用的编译定义。指定的目标必须是通过诸如 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable") 或 [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library") 等命令创建的，并且不能是[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)。

需要使用 `INTERFACE`、`PUBLIC` 和 `PRIVATE` 关键字来指定后续参数的作用范围。`PRIVATE` 和 `PUBLIC` 项将填充`<target>`的 [COMPILE_DEFINITIONS](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS") 属性。`PUBLIC` 和 `INTERFACE` 项将填充`<target>`的 [INTERFACE_COMPILE_DEFINITIONS](https://cmake.com.cn/cmake/help/latest/prop_tgt/INTERFACE_COMPILE_DEFINITIONS.html#prop_tgt:INTERFACE_COMPILE_DEFINITIONS "INTERFACE_COMPILE_DEFINITIONS") 属性。以下参数用于指定编译定义。对同一个`<target>`重复调用该命令时，会按调用顺序追加各项。

*从 3.11 版本开始*：允许在[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)上设置 `INTERFACE` 项。

`target_compile_definitions` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参阅 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)") 手册。

任何参数项前导的 `-D` 将被移除。空项将被忽略。例如，以下所有形式都是等价的：
```cmake
target_compile_definitions(foo PUBLIC FOO)
target_compile_definitions(foo PUBLIC -DFOO)  # -D removed
target_compile_definitions(foo PUBLIC "" FOO) # "" ignored
target_compile_definitions(foo PUBLIC -D FOO) # -D becomes "", then ignored
```
定义可选地可以包括值：
```cmake
target_compile_definitions(foo PUBLIC FOO=1)
```
请注意，许多编译器将 `-DFOO` 视为等同于 `-DFOO=1`，但其他工具可能无法在所有情况下识别这种用法（例如 IntelliSense）。

# 另请参阅
- [add_compile_definitions()](https://cmake.org/cmake/help/latest/command/add_compile_definitions.html#command:add_compile_definitions "add_compile_definitions")
- [target_compile_features()](https://cmake.org/cmake/help/latest/command/target_compile_features.html#command:target_compile_features "target_compile_features")
- [target_compile_options()](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options "target_compile_options")
- [target_include_directories()](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")
- [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")
- [target_link_directories()](https://cmake.org/cmake/help/latest/command/target_link_directories.html#command:target_link_directories "target_link_directories")
- [target_link_options()](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options "target_link_options")
- [target_precompile_headers()](https://cmake.org/cmake/help/latest/command/target_precompile_headers.html#command:target_precompile_headers "target_precompile_headers")
- [target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources "target_sources")
- 
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------