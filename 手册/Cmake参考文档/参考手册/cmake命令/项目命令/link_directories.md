---
title: link_directories
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------


添加用于链接器查找库文件的目录。

```cmake
link_directories([AFTER|BEFORE] directory1 [directory2 ...])
```

添加建议链接器搜索库文件的路径。传递给该命令的相对路径会被解释为相对于当前源码目录，详见策略文档 [CMP0015](https://cmake.org/cmake/help/latest/policy/CMP0015.html#policy:CMP0015 "CMP0015")。

该命令仅对调用该命令之后创建的目标生效。

*3.13 版本新增*：目录会被添加到当前 `CMakeLists.txt` 文件的目录属性 [LINK_DIRECTORIES](https://cmake.org/cmake/help/latest/prop_dir/LINK_DIRECTORIES.html#prop_dir:LINK_DIRECTORIES) 中，必要时会将相对路径转换为绝对路径。关于构建系统属性定义的更多内容，请查阅 [cmake‑buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7)) 手册。

*3.13 版本新增*：默认情况下，指定的目录会追加到现有目录列表的末尾。可以将变量 [CMAKE_LINK_DIRECTORIES_BEFORE](https://cmake.org/cmake/help/latest/variable/CMAKE_LINK_DIRECTORIES_BEFORE.html#variable:CMAKE_LINK_DIRECTORIES_BEFORE) 设置为 `ON` 来修改该默认行为。通过显式使用 `AFTER` 或 `BEFORE`，可以不受默认配置影响，选择追加或者前置目录。

*3.13 版本新增*：`link_directories` 的参数可以使用生成器表达式，语法格式为 `"$<...>"`。可用的生成器表达式请查阅 [cmake‑generator‑expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7)) 手册。

>注意：
该命令很少有使用必要，在有其他可选方案时建议避免使用。尽可能优先传入库的完整绝对路径，这样可以保证始终链接到正确的库。[find_library](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) 命令可以获取库的完整路径，获取的路径通常可以直接用于 [target_link_libraries](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) 调用中。需要提供库搜索路径的场景包括：
>- [Xcode](https://cmake.org/cmake/help/latest/generator/Xcode.html#generator:Xcode) 这类工程生成器：用户可以在构建阶段切换目标架构，但库文件只包含单一架构（不是用二进制文件），因此无法使用库的完整路径。
>- 库自身存在私有库依赖，期望通过 `RPATH` 机制完成查找，但部分链接器无法完整解析这些路径（例如存在 `$ORIGIN` 这类特殊标识）。
>
>如果必须提供库搜索路径，优先使用 [target_link_directories](https://cmake.org/cmake/help/latest/command/target_link_directories.html#command:target_link_directories) 而非 `link_directories()`，尽可能将作用域限定在目标上。这个面向目标的命令还可以控制搜索目录向其他依赖目标的传递规则。

# 另请参阅
- [target_link_directories()](https://cmake.org/cmake/help/latest/command/target_link_directories.html#command:target_link_directories)
- [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries)

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------