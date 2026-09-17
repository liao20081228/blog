---
title: link_libraries
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------



为后续新增的所有目标链接库。

```
link_libraries([item1 [item2 [...]]] [[debug|optimized|general] <item>] ...)

```

指定库或链接标志，用于当前目录及其子目录中，后续由 `add_executable`、`add_library` 等命令创建的全部目标的链接过程。 参数含义请参考 `target_link_libraries` 命令文档。

### 注意

**只要条件允许，建议优先使用 `target_link_libraries`。** 库依赖会自动链式传递，因此几乎不需要在目录级别全局指定链接库。



------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------