---
title: add_compile_definitions
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 项目命令
这些命令仅在CMake项目中可用。
## add_compile_definitions
*自版本 3.12 起添加*。

将预处理器定义添加到源文件的编译过程中。
```cmake
add_compile_definitions（<definition>...）
```
将预处理器定义添加到编译器命令行中。

这些预处理器定义会被添加到当前 `CMakeLists` 文件的 [COMPILE_DEFINITIONS](https://cmake.org/cmake/help/latest/prop_dir/COMPILE_DEFINITIONS.html#prop_dir:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS") *目录属性*中。它们也会被添加到当前 `CMakeLists` 文件中每个目标的 [COMPILE_DEFINITIONS](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS") *目标属性*中。

`<definition>`的语法为 `VAR` 或 `VAR=value`。不支持函数式定义。CMake 会自动为原生构建系统正确转义值（请注意，CMake 语言语法可能需要转义某些值）。

*自版本 3.26 起添加*：任何项前导的 `-D` 都会被移除。

`add_compile_definitions` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参阅 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)") 手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)") 手册。

### 另请参阅
命令 [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions") 用于添加针对特定目标的宏定义。



------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
