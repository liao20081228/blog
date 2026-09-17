---
title: target_include_directories
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
为目标添加包含目录。
```cmake
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

指定在编译给定目标时要使用的包含目录。指定的 `<target>` 必须是由通过 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable") 或 [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library") 等命令创建的，并且不能是[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)。

通过显式使用 `AFTER` 或 `BEFORE`，你可以选择是在末尾追加还是开头插入，而不受默认行为的影响。

`INTERFACE`、`PUBLIC` 和 `PRIVATE` 关键字用于指定后续参数的作用范围。`PRIVATE` 和 `PUBLIC` 项将填充 `<target>` 的 [INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES "INCLUDE_DIRECTORIES") 属性。`PUBLIC` 和 `INTERFACE` 项将填充 `<target>` 的 [INTERFACE_INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES "INTERFACE_INCLUDE_DIRECTORIES") 属性。后续参数用于指定包含目录。

*从 3.11 版本开始*：允许在[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)上设置 `INTERFACE` 项。

对同一个 `<target>` 重复调用该命令时，会按照调用顺序追加项目。

如果指定了 `SYSTEM`，编译器将被告知这些目录在某些平台上应被视为系统包含目录。这可能会产生一些效果，例如抑制警告或在依赖关系计算中跳过包含的头文件（请参阅编译器文档）。此外，系统包含目录的搜索顺序将优先于普通包含目录，无论指定的顺序如何。

如果 `SYSTEM` 与 `PUBLIC` 或 `INTERFACE` 一起使用，[INTERFACE_SYSTEM_INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_SYSTEM_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_SYSTEM_INCLUDE_DIRECTORIES "INTERFACE_SYSTEM_INCLUDE_DIRECTORIES") 目标属性将被填充为指定的目录。

`target_include_directories` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参阅 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)") 手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)")手册。

指定的包含目录可以是绝对路径或相对路径。相对路径将被解释为相对于当前源码目录（即 [CMAKE_CURRENT_SOURCE_DIR](https://cmake.org/cmake/help/latest/variable/CMAKE_CURRENT_SOURCE_DIR.html#variable:CMAKE_CURRENT_SOURCE_DIR "CMAKE_CURRENT_SOURCE_DIR")），并在存储到相关目标属性之前转换为绝对路径。如果路径以生成器表达式开头，则它将始终被视为绝对路径（下面有一个例外情况），并且将直接使用该路径而不会进行任何修改。

在构建目录和安装目录之间，包含目录的使用要求通常有所不同。可以使用 [BUILD_INTERFACE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:BUILD_INTERFACE "BUILD_INTERFACE") 和 [INSTALL_INTERFACE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:INSTALL_INTERFACE "INSTALL_INTERFACE") 生成器表达式来描述基于使用位置的独立使用要求。[INSTALL_INTERFACE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:INSTALL_INTERFACE "INSTALL_INTERFACE")  表达式中允许使用相对路径，这些路径将被解释为相对于安装前缀的路径。不应在 [BUILD_INTERFACE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:BUILD_INTERFACE "BUILD_INTERFACE") 表达式中使用相对路径，因为它们不会被转换为绝对路径。例如：
```cmake
target_include_directories(mylib PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/mylib>
  $<INSTALL_INTERFACE:include/mylib>  # <prefix>/include/mylib
)
```
# 创建可重定位的包
需要注意的是，不建议在目标的 [INTERFACE_INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES "INTERFACE_INCLUDE_DIRECTORIES")的 [INSTALL_INTERFACE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:INSTALL_INTERFACE "INSTALL_INTERFACE") 中使用依赖项的包含目录的绝对路径。这样做会将依赖项的包含目录路径硬编码到已安装的包中，而这些路径**是构建该包时所在机器上的路径**。

[INTERFACE_INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES "INTERFACE_INCLUDE_DIRECTORIES")的 [INSTALL_INTERFACE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:INSTALL_INTERFACE "INSTALL_INTERFACE") 仅适用于指定目标本身提供的头文件所需的包含目录，而不适用于通过其 [INTERFACE_LINK_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_LIBRARIES.html#prop_tgt:INTERFACE_LINK_LIBRARIES "INTERFACE_LINK_LIBRARIES") 目标属性列出的传递依赖项所提供的。这些依赖项本身应该是在 [INTERFACE_INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES "INTERFACE_INCLUDE_DIRECTORIES") 中指定自己的头文件位置的目标。

有关在创建可重新分发的包时，指定使用要求时必须采取的额外注意事项的讨论，请参阅 [cmake-packages(7)](https://cmake.org/cmake/help/latest/manual/cmake-packages.7.html#manual:cmake-packages(7) "cmake-packages(7)")手册中的[创建可重定位的包](https://cmake.org/cmake/help/latest/manual/cmake-packages.7.html#creating-relocatable-packages)部分。

# 另请参阅
- [include_directories()](https://cmake.org/cmake/help/latest/command/include_directories.html#command:include_directories "include_directories")
- [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions")
- [target_compile_features()](https://cmake.org/cmake/help/latest/command/target_compile_features.html#command:target_compile_features "target_compile_features")
- [target_compile_options()](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options "target_compile_options")
- [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries")
- [target_link_directories()](https://cmake.org/cmake/help/latest/command/target_link_directories.html#command:target_link_directories "target_link_directories")
- [target_link_options()](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options "target_link_options")
- [target_precompile_headers()](https://cmake.org/cmake/help/latest/command/target_precompile_headers.html#command:target_precompile_headers "target_precompile_headers")
- [target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources "target_sources")
 
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------