---
title: add_compile_definitions
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

向源文件的编译过程中添加选项。
```cmake
add_compile_options(<option>...)
```
将选项添加到 [COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_dir/COMPILE_OPTIONS.html#prop_dir:COMPILE_OPTIONS "COMPILE_OPTIONS") *目录属性*中。这些选项在从当前目录及其子目录编译目标时使用。
>注意：这些选项在链接时不会被使用。有关链接选项，请参见 [add_link_options()](https://cmake.org/cmake/help/latest/command/add_link_options.html#command:add_link_options "add_compile_options") 命令。

# 参数

`add_compile_options` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用的表达式，请参见 [cmake-generator-expressions(7) ](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions")手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "manual:cmake-buildsystem") 手册。

# 选项去重
最终用于目标的选项集是通过从当前目标及其依赖项的使用要求中累积选项而构建的。选项集会进行去重处理，以避免重复。

*从 3.12 版本开始*：虽然对单个选项来说，去重是有益的，但去重步骤可能会破坏选项组。例如，`-option A -option B `会变成 `-option A B`。可以使用类似 shell 的引号且带有`SHELL:` 前缀来指定一组选项。`SHELL:` 前缀会被移除，其余选项字符串将使用 [separate_arguments()](https://cmake.org/cmake/help/latest/command/separate_arguments.html#command:separate_arguments "separate_arguments") `UNIX_COMMAND` 模式进行解析。例如，`"SHELL:-option A" "SHELL:-option B"` 会变成 `-option A -option B`。


# 示例
由于不同的编译器支持不同的选项，因此该命令的典型用法是在编译器特定的条件语句中：
```cmake
if (MSVC)
    # warning level 4
    add_compile_options(/W4)
else()
    # additional warnings
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()
```
要设置针对特定语言的选项，请使用 [$<COMPILE_LANGUAGE>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:COMPILE_LANGUAGE "COMPILE_LANGUAGE") 或 [$<COMPILE_LANGUAGE:languages>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:COMPILE_LANGUAGE:languages "COMPILE_LANGUAGE:languages") 生成器表达式。

# 另请参阅
- 该命令可用于添加任何选项。然而，对于添加预处理器定义和包含目录，建议使用更具体的命令 [add_compile_definitions()](https://cmake.org/cmake/help/latest/command/add_compile_definitions.html#command:add_compile_definitions "add_compile_definitions") 和 [include_directories()](https://cmake.org/cmake/help/latest/command/include_directories.html#command:include_directories "include_directories")。
- 命令 [target_compile_options()](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options "target_compile_options") 用于添加针对特定目标的选项。
- 该命令为所有语言添加编译选项。使用 [COMPILE_LANGUAGE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:COMPILE_LANGUAGE "COMPILE_LANGUAGE") 生成器表达式来指定针对特定语言的编译选项。
- 源文件属性 [COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_sf/COMPILE_OPTIONS.html#prop_sf:COMPILE_OPTIONS "COMPILE_OPTIONS") 用于向单个源文件添加选项。
- [add_link_options()](https://cmake.org/cmake/help/latest/command/add_link_options.html#command:add_link_options "add_link_options") 用于添加链接选项。
- [CMAKE_\<LANG>_FLAGS](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html#variable:CMAKE_%3CLANG%3E_FLAGS "CMAKE_<LANG>_FLAGS") 和 [CMAKE_\<LANG>_FLAGS_\<CONFIG>](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS_CONFIG.html#variable:CMAKE_%3CLANG%3E_FLAGS_%3CCONFIG%3E "CMAKE_<LANG>_FLAGS_<CONFIG>") 用于添加传递给语言范围内调用的所有编译器的标志。这包括驱动编译和链接的调用。



------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
