---
title: add_subdirectory
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
向构建中添加一个子目录。
```cmake
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL] [SYSTEM])
```
该命令用于向构建中添加一个子目录。`source_dir` 指定了包含源`CMakeLists.txt` 和源代码文件的目录。如果它是一个相对路径，则相对于当前目录进行解析（这是典型用法），但也可以是一个绝对路径。`binary_dir` 指定了输出文件存放的目录。如果它是一个相对路径，则相对于当前输出目录进行解析，但也可以是一个绝对路径。如果未指定 `binary_dir`，则将使用 `source_dir` 的值（在展开任何相对路径之前）作为输出目录（这是典型用法）。指定源目录中的 `CMakeLists.txt` 文件将立即被 CMake 处理，然后继续处理当前输入文件中的其他命令。

如果提供了 `EXCLUDE_FROM_ALL` 参数，则会在添加的目录上设置 [EXCLUDE_FROM_ALL](https://cmake.org/cmake/help/latest/prop_dir/EXCLUDE_FROM_ALL.html#prop_dir:EXCLUDE_FROM_ALL "EXCLUDE_FROM_ALL")属性。这将使该目录在默认构建中被排除。有关详细信息，请参见目录属性  [EXCLUDE_FROM_ALL](https://cmake.org/cmake/help/latest/prop_dir/EXCLUDE_FROM_ALL.html#prop_dir:EXCLUDE_FROM_ALL "EXCLUDE_FROM_ALL")。

从 3.25 版本开始：如果提供了 `SYSTEM` 参数，则会将子目录的 [SYSTEM](https://cmake.org/cmake/help/latest/prop_dir/SYSTEM.html#prop_dir:SYSTEM "SYSTEM") 目录属性设置为 `true`。该属性用于初始化在该子目录中创建的每个非导入目标的[SYSTEM](https://cmake.org/cmake/help/latest/prop_tgt/SYSTEM.html#prop_tgt:SYSTEM "SYSTEM")属性。

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------