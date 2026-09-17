---
title: target_compile_features
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
*在3.1版本中添加*。 

为目标添加预期的编译器特性。

```cmake
target_compile_features(<target> <PRIVATE|PUBLIC|INTERFACE> <feature> [...])
```
指定在编译给定`<target>`时所需的编译器特性。如果该特性未列在变量 [CMAKE_C_COMPILE_FEATURES](https://cmake.org/cmake/help/latest/variable/CMAKE_C_COMPILE_FEATURES.html#variable:CMAKE_C_COMPILE_FEATURES "CMAKE_C_COMPILE_FEATURES")、[CMAKE_CUDA_COMPILE_FEATURES](https://cmake.org/cmake/help/latest/variable/CMAKE_CUDA_COMPILE_FEATURES.html#variable:CMAKE_CUDA_COMPILE_FEATURES "CMAKE_CUDA_COMPILE_FEATURES") 或 [CMAKE_CXX_COMPILE_FEATURES](https://cmake.org/cmake/help/latest/variable/CMAKE_CXX_COMPILE_FEATURES.html#variable:CMAKE_CXX_COMPILE_FEATURES "CMAKE_CXX_COMPILE_FEATURES") 中，CMake 将会报告错误。如果使用该特性需要额外的编译器标志（例如 `-std=gnu++11`），则会自动添加该标志。

需要使用 `INTERFACE`、`PUBLIC` 和 `PRIVATE` 关键字来指定特性的作用范围。`PRIVATE` 和 `PUBLIC` 项将填充 `<target>` 的 [COMPILE_FEATURES](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_FEATURES.html#prop_tgt:COMPILE_FEATURES "COMPILE_FEATURES") 属性。`PUBLIC` 和 `INTERFACE` 项将填充 `<target>` 的 [INTERFACE_COMPILE_FEATURES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_COMPILE_FEATURES.html#prop_tgt:INTERFACE_COMPILE_FEATURES "INTERFACE_COMPILE_FEATURES") 属性。对同一个 `<target>` 的重复调用会追加这些项。

*从 3.11 版本开始*：允许在[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)上设置 `INTERFACE` 项。

指定的 `<target>` 必须是通过 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable") 或 [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library") 等命令创建的，并且不能是[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)。

`target_compile_features` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参阅 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")手册。有关编译特性以及支持的编译器列表的信息，请参阅 [cmake-compile-features(7)](https://cmake.org/cmake/help/latest/manual/cmake-compile-features.7.html#manual:cmake-compile-features(7) "cmake-compile-features(7)") 手册。

# 另请参阅
- [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions")
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