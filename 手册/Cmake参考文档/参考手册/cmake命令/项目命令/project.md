---
title: project
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
# 概要
```cmake

project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [COMPAT_VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [SPDX_LICENSE <license-string>]
        [DESCRIPTION <description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])

```

设置项目的名称，并将其存入变量 [PROJECT_NAME]。当在顶层 `CMakeLists.txt` 中调用该命令时，还会将项目名称存入变量 [CMAKE_PROJECT_NAME](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_NAME.html#variable:CMAKE_PROJECT_NAME)。


同时会设置如下变量：

[`PROJECT_SOURCE_DIR`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/variable/PROJECT_SOURCE_DIR.html#variable:PROJECT_SOURCE_DIR%5D(https://cmake.org/cmake/help/latest/variable/PROJECT_SOURCE_DIR.html#variable:PROJECT_SOURCE_DIR))、[`<PROJECT‑NAME>_SOURCE_DIR`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/variable/PROJECT-NAME_SOURCE_DIR.html#variable:%3CPROJECT-NAME%3E_SOURCE_DIR%5D(https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_SOURCE_DIR.html#variable:%3CPROJECT-NAME%3E_SOURCE_DIR))

> 
> 
> 项目源码目录的绝对路径。
> 
> 

[`PROJECT_BINARY_DIR`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/variable/PROJECT_BINARY_DIR.html#variable:PROJECT_BINARY_DIR%5D(https://cmake.org/cmake/help/latest/variable/PROJECT_BINARY_DIR.html#variable:PROJECT_BINARY_DIR))、[`<PROJECT‑NAME>_BINARY_DIR`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/variable/PROJECT-NAME_BINARY_DIR.html#variable:%3CPROJECT-NAME%3E_BINARY_DIR%5D(https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_BINARY_DIR.html#variable:%3CPROJECT-NAME%3E_BINARY_DIR))

> 
> 
> 项目构建输出目录的绝对路径。
> 
> 

[`PROJECT_IS_TOP_LEVEL`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/variable/PROJECT_IS_TOP_LEVEL.html#variable:PROJECT_IS_TOP_LEVEL%5D(https://cmake.org/cmake/help/latest/variable/PROJECT_IS_TOP_LEVEL.html#variable:PROJECT_IS_TOP_LEVEL))、[`<PROJECT‑NAME>_IS_TOP_LEVEL`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/variable/PROJECT-NAME_IS_TOP_LEVEL.html#variable:%3CPROJECT-NAME%3E_IS_TOP_LEVEL%5D(https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_IS_TOP_LEVEL.html#variable:%3CPROJECT-NAME%3E_IS_TOP_LEVEL))

> 
> 
> 3.21 版本新增。 布尔值，标识当前项目是否为顶层项目。
> 
> 

其余变量由下文 [`project()` 命令选项](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/command/project.html#options%5D(https://cmake.org/cmake/help/latest/command/project.html#options)) 中描述的可选参数进行设置。如果未传入某个选项，则对应的变量会被置为空字符串。

注意：形如 `<name>_SOURCE_DIR`、`<name>_BINARY_DIR` 的变量，也可能在调用 `project()` 之前被其他命令设置（例如 [`FetchContent_MakeAvailable`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/module/FetchContent.html#command:fetchcontent_makeavailable%5D(https://cmake.org/cmake/help/latest/module/FetchContent.html#command:fetchcontent_makeavailable)) 就是一例）。项目代码**不应当**指望 `<PROJECT‑NAME>_SOURCE_DIR`、`<PROJECT‑NAME>_BINARY_DIR` 在脱离 `project()` 调用作用域、以及它的子作用域之外仍保持某个特定值。

3.30 版本变更： 当执行 `project(<PROJECT‑NAME> ...)` 时，如果 `<PROJECT‑NAME>_SOURCE_DIR`、`<PROJECT‑NAME>_BINARY_DIR`、`<PROJECT‑NAME>_IS_TOP_LEVEL` 已经作为普通变量存在，则该调用会更新它们。同名的缓存条目依旧会照常设置。详细参见 3.30.3、3.30.4、3.30.5 的版本说明文档。

3.31 版本变更： `project(<PROJECT‑NAME> ...)` 命令总会把 `<PROJECT‑NAME>_SOURCE_DIR`、`<PROJECT‑NAME>_BINARY_DIR`、`<PROJECT‑NAME>_IS_TOP_LEVEL` 设置为普通变量，参见策略 [`CMP0180`](https://www.doubao.com/chat/%5Bhttps://cmake.org/cmake/help/latest/policy/CMP0180.html#policy:CMP0180%5D(https://cmake.org/cmake/help/latest/policy/CMP0180.html#policy:CMP0180))。同名的缓存条目依旧会照常设置。

* * *




------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------