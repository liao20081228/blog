---
title: include_directories
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------


向构建过程添加头文件包含目录。

```cmake
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
```

将给定目录添加到编译器用于搜索头文件的目录列表中。相对路径会被解析为相对于当前源码目录。

这些包含目录会被添加到当前 `CMakeLists` 文件的 [INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_dir/INCLUDE_DIRECTORIES.html#prop_dir:INCLUDE_DIRECTORIES) 目录属性。同时也会添加到当前 `CMakeLists` 文件内每个目标的 [INCLUDE_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES) 目标属性。生成器实际使用的是目标属性的值。

默认情况下，指定的目录会追加到现有目录列表的末尾。可以通过将 [CMAKE_INCLUDE_DIRECTORIES_BEFORE](https://cmake.org/cmake/help/latest/variable/CMAKE_INCLUDE_DIRECTORIES_BEFORE.html#variable:CMAKE_INCLUDE_DIRECTORIES_BEFORE) 设置为 `ON` 来修改该默认行为。显式使用 `AFTER` 或 `BEFORE`，可以不受默认设置影响，选择追加还是前置。

如果给定 `SYSTEM` 选项，在部分平台上会告知编译器这些目录属于系统头文件目录。该标记可能带来的效果包括编译器跳过部分警告，或者这些已固定安装的系统文件不参与依赖计算，具体请查阅编译器文档。

`include_directories` 的参数可以使用 `$<...>` 语法的生成器表达式。可用的表达式参见 [cmake‑generator‑expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7)) 手册。关于定义构建系统属性的更多内容参见 [cmake‑buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7)) 手册。


> **注意**： 优先使用 [target_include_directories](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories) 命令为各个目标添加包含目录，并可选择将其传递 / 导出给依赖方。


# 另请参阅
- [target_include_directories()](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories)


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------