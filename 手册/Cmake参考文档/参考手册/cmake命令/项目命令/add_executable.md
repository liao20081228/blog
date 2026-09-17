---
title: add_executable
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
使用指定的源文件将可执行文件添加到项目中。

# 普通可执行文件
```cmake
add_executable(<name> <options>... <sources>...)
```

添加一个名为`<name>`的[可执行目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#executables)，该目标将由命令调用中列出的源文件构建而成。


选项包括：

- `WIN32`
自动设置[WIN32_EXECUTABLE](https://cmake.org/cmake/help/latest/prop_tgt/WIN32_EXECUTABLE.html#prop_tgt:WIN32_EXECUTABLE "WIN32_EXECUTABLE")目标属性。有关该目标属性的详细信息，请参见相关文档。
- `MACOSX_BUNDLE`
自动设置[MACOSX_BUNDLE](https://cmake.org/cmake/help/latest/prop_tgt/MACOSX_BUNDLE.html#prop_tgt:MACOSX_BUNDLE "MACOSX_BUNDLE")目标属性。有关该目标属性的详细信息，请参见相关文档。
- `EXCLUDE_FROM_ALL`
自动设置[EXCLUDE_FROM_ALL](https://cmake.org/cmake/help/latest/prop_tgt/EXCLUDE_FROM_ALL.html#prop_tgt:EXCLUDE_FROM_ALL "EXCLUDE_FROM_ALL")目标属性。有关该目标属性的详细信息，请参见相关文档。

`<name>`对应于逻辑目标名称，必须在项目中全局唯一。实际生成的可执行文件的名称将根据本地平台的约定（例如`<name>.exe`或仅`<name>`）来构建。

*3.1版本新增*：`add_executable`的源文件参数可以使用“生成器表达式”，其语法为`$<...>`。有关可用表达式的详细信息，请参见[cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")手册。

*3.11版本新增*：如果稍后通过[target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources "target_sources")添加了源文件，则可以省略这些源文件。

默认情况下，可执行文件将创建在构建目录目录中，该目录对应于命令调用时所处的源树目录。要更改此位置，请参见[RUNTIME_OUTPUT_DIRECTORY](https://cmake.org/cmake/help/latest/prop_tgt/RUNTIME_OUTPUT_DIRECTORY.html#prop_tgt:RUNTIME_OUTPUT_DIRECTORY "RUNTIME_OUTPUT_DIRECTORY")目标属性的文档。要更改最终文件名中的`<name>`部分，请参见[OUTPUT_NAME](https://cmake.org/cmake/help/latest/prop_tgt/OUTPUT_NAME.html#prop_tgt:OUTPUT_NAME "OUTPUT_NAME")目标属性的文档。

有关定义构建系统属性的更多信息，请参见[cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)")手册。

另请参阅[HEADER_FILE_ONLY](https://cmake.org/cmake/help/latest/prop_sf/HEADER_FILE_ONLY.html#prop_sf:HEADER_FILE_ONLY "HEADER_FILE_ONLY")，了解如果某些源文件经过预处理，并且你希望在IDE中能够访问原始源文件时，应如何处理。

# 导入的可执行文件

```cmake
add_executable(<name> IMPORTED [GLOBAL])¶
```
添加一个[导入的可执行目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)以引用位于项目之外的某个可执行文件。该目标名称可以像项目内构建的任何目标一样被引用，但默认情况下，它仅在创建它的目录及其子目录中可见。

选项包括：

- `GLOBAL`
使目标名称全局可见。

不会生成用于构建导入目标的规则，并且 [IMPORTED](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED "IMPORTED")目标的属性为 `True`。导入的可执行文件对于方便地从诸如 [add_custom_command](https://cmake.org/cmake/help/latest/command/add_custom_command.html#command:add_custom_command "add_custom_command")等命令中引用非常有用。

导入可执行文件的详细信息通过设置以 `IMPORTED_` 开头的属性来指定。其中最重要的属性是 [IMPORTED_LOCATION](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_LOCATION.html#prop_tgt:IMPORTED_LOCATION "IMPORTED_LOCATION")（以及其针对不同配置的版本 [IMPORTED_LOCATION_\<CONFIG>](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_LOCATION_CONFIG.html#prop_tgt:IMPORTED_LOCATION_%3CCONFIG%3E "IMPORTED_LOCATION_<CCONFIG></CCONFIG>")），它指定了磁盘上主要可执行文件的位置。有关更多信息，请参阅 `IMPORTED_*` 属性的文档。

# 别名的可执行文件
```cmake
add_executable(<name> ALIAS <target>)
```
创建一个[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)，使得在后续命令中可以使用`<name>`来引用`<target>`。生成的构建系统中不会将`<name>`作为make目标出现。注意，`<target>`本身不能是`ALIAS `。

*3.11版本新增*：`ALIAS` 可以针对`GLOBAL`[导入的目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)。

*3.18版本新增*：`ALIAS` 可以针对`non-GLOBAL`导入的目标。此类别名仅在其创建目录及其子目录中有效。可以使用 [ALIAS_GLOBAL](https://cmake.org/cmake/help/latest/prop_tgt/ALIAS_GLOBAL.html#prop_tgt:ALIAS_GLOBAL "ALIAS_GLOBAL")目标属性来检查该别名是否为全局别名。

`ALIAS`目标可以用作读取属性的目标、自定义命令和自定义目标的可执行文件。它们还可以通过常规的 [if(TARGET)](https://cmake.org/cmake/help/latest/command/if.html#target "if(TARGET)") 子命令来测试是否存在。需要注意的是，`<name>` 不能用于修改 `<target>` 的属性，也就是说，它不能作为 [set_property()](https://cmake.org/cmake/help/latest/command/set_property.html#command:set_property "set_property")、[set_target_properties()](https://cmake.org/cmake/help/latest/command/set_target_properties.html#command:set_target_properties "set_target_properties")、[target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries") 等命令的操作数。`ALIAS` 目标不能被安装或导出。

# 另请参阅
 
 - [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library")
  
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------