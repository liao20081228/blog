---
title: target_link_directories
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
*3.13 版本新增*。 
为目标添加链接目录。
```cmake
target_link_directories(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```
指定链接器在链接给定目标时应该搜索库的路径。每项可以是绝对路径或相对路径，后者将被解释为相对于当前源码目录的路径。这些条目将被添加到链接命令中。

指定的 `<target>` 必须是通过 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable") 或 [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library") 等命令创建的，并且不能是[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)。

`INTERFACE`、`PUBLIC` 和 `PRIVATE` 关键字用于指定其后的条目的范围。`PRIVATE` 和 `PUBLIC` 项将填充 `<target>` 的 [LINK_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/LINK_DIRECTORIES.html#prop_tgt:LINK_DIRECTORIES "LINK_DIRECTORIES")属性。`PUBLIC` 和 `INTERFACE` 项将填充 `<target>` 的 [INTERFACE_LINK_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_DIRECTORIES.html#prop_tgt:INTERFACE_LINK_DIRECTORIES "INTERFACE_LINK_DIRECTORIES") 属性（[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)仅支持 `INTERFACE` 项）。每项指定一个链接目录，并且在添加到相关属性之前，如果必要的话，将转换为绝对路径。对同一个 `<target>` 重复调用此命令时，项目将按调用顺序追加。

如果指定了 `BEFORE`，则内容将被添加到相关属性的开头，而不是末尾。

`target_link_directories` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参阅 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)") 手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)")手册。

>注意：此命令很少需要使用，应尽量避免使用，尤其是在有其他选择的情况下。尽可能传递库的完整绝对路径，因为这可以确保始终链接到正确的库。[find_library()](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library "find_library")命令会返回完整路径，通常可以直接在 [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries") 调用中使用。需要指定库搜索路径的情况包括：
>- 像 [Xcode](https://cmake.org/cmake/help/latest/generator/Xcode.html#generator:Xcode "Xcode") 这样的项目生成器，用户可以在构建时切换目标架构，但由于库的完整路径只能提供一个架构（即它不是通用二进制文件），因此无法使用完整路径。
>- 库本身可能依赖于其他私有库，这些私有库期望通过 `RPATH` 机制来查找，但某些链接器无法完全解析这些路径（例如，由于存在 `$ORIGIN` 等特殊字符）。

# 另请参阅
- [link_directories()](https://cmake.org/cmake/help/latest/command/link_directories.html#command:link_directories "link_directories")
- [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions")
- [target_compile_features()](https://cmake.org/cmake/help/latest/command/target_compile_features.html#command:target_compile_features "target_compile_features")
- [target_compile_options()](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options "target_compile_options")
- [target_include_directories()](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories "target_include_directories")
- [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")
- [target_link_options()](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options "target_link_options")
- [target_precompile_headers()](https://cmake.org/cmake/help/latest/command/target_precompile_headers.html#command:target_precompile_headers "target_precompile_headers")
- [target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources "target_sources")
 
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------