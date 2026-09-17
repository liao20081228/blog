---
title: add_definitions
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
将 `-D` 定义标志添加到源文件的编译过程中。 
```cmake
add_definitions(-DFOO -DBAR...) 
```

该命令会将定义添加到当前目录中目标编译器的命令行参数中(无论是在调用此命令之前还是之后添加的)，在这之后添加到子目录中的目标也会受到影响。此命令可用于添加任何标志，但其主要目的是添加预处理器定义。 

>注意：此命令已被以下替代方案取代： 
使用 [add_compile_definitions()](https://cmake.org/cmake/help/latest/command/add_compile_definitions.html#command:add_compile_definitions "add_compile_definitions") 来添加预处理器定义。 
使用 [include_directories()](https://cmake.org/cmake/help/latest/command/include_directories.html#command:include_directories "include_directories") 来添加包含目录。 
使用 [add_compile_options()](https://cmake.org/cmake/help/latest/command/add_compile_options.html#command:add_compile_options "add_compile_options")来添加其他选项。 

以 `-D` 或 `/D` 开头且看起来像是预处理器定义的标志会自动添加到当前目录的 [COMPILE_DEFINITIONS](https://cmake.org/cmake/help/latest/prop_dir/COMPILE_DEFINITIONS.html#prop_dir:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS") 目录属性中。出于向后兼容性的考虑，具有非简单值的定义可能会保留在标志集中，而不是被转换。有关如何将预处理器定义添加到特定作用域和配置的详细信息，请参阅[目录](https://cmake.org/cmake/help/latest/prop_dir/COMPILE_DEFINITIONS.html#prop_dir:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS")、[目标](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS")和[源文件](https://cmake.org/cmake/help/latest/prop_sf/COMPILE_DEFINITIONS.html#prop_sf:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS")的 `COMPILE_DEFINITIONS` 属性的文档。 

# 另请参阅 
有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)") 手册。

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------