---
title: add_dependencies
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
在顶层目标之间添加依赖关系。
```cmake
add_dependencies(<target> <target-dependency>...)
```

使一个顶层目标 `<target>` 依赖于其他顶层目标，以确保这些目标在 `<target>`之前被构建。顶层目标是指通过 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable")、[add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "add_library") 或 [add_custom_target()](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target "add_custom_target") 命令创建的目标（不包括由 CMake 生成的目标，如 `install`）。

添加到[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)或[接口库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#interface-libraries)的依赖关系被传递性的遵循，因为目标本身不会被构建。

*3.3 版本新增*：允许向接口库添加依赖关系。

*3.8 版本新增*：依赖关系将填充 `<target>` 的 [MANUALLY_ADDED_DEPENDENCIES](https://cmake.org/cmake/help/latest/prop_tgt/MANUALLY_ADDED_DEPENDENCIES.html#prop_tgt:MANUALLY_ADDED_DEPENDENCIES "MANUALLY_ADDED_DEPENDENCIES") 属性。

*3.9 版本更改*：[Ninja 生成器](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html#ninja-generators)使用比其他生成器更弱的顺序规则，以提高并发性。它们仅保证在 `<target>` 开始编译之前，依赖项的自定义命令已经完成；这确保了生成的源文件可用。

*4.1 版本更改*：该命令可以不带任何依赖项调用。之前，至少需要一个依赖项。

# 另请参阅
- [add_custom_target()](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target "add_custom_target") 和 [add_custom_command()](https://cmake.org/cmake/help/latest/command/add_custom_command.html#command:add_custom_command "add_custom_command") 命令的 `DEPENDS` 选项，用于在自定义规则中添加文件级别的依赖关系。

- [OBJECT_DEPENDS](https://cmake.org/cmake/help/latest/prop_sf/OBJECT_DEPENDS.html#prop_sf:OBJECT_DEPENDS "OBJECT_DEPENDS") 源文件属性，用于向对象文件添加文件级别的依赖关系。
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------