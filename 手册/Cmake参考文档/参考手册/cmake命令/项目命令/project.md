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

设置项目的名称，并将其存入变量[PROJECT_NAME](https://cmake.org/cmake/help/latest/variable/PROJECT_NAME.html#variable:PROJECT_NAME "PROJECT_NAME") 。当在顶层 `CMakeLists.txt` 中调用该命令时，还会将项目名称存入变量 [CMAKE_PROJECT_NAME](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_NAME.html#variable:CMAKE_PROJECT_NAME)。


同时会设置如下变量：

- [PROJECT\_SOURCE\_DIR](https://cmake.org/cmake/help/latest/variable/PROJECT_SOURCE_DIR.html#variable:PROJECT_SOURCE_DIR "PROJECT_SOURCE_DIR")， [\<PROJECT\-NAME\>\_SOURCE_DIR](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_SOURCE_DIR.html#variable:%3CPROJECT-NAME%3E_SOURCE_DIR "<PROJECT-NAME>_SOURCE_DIR")
项目源码目录的绝对路径。

- [PROJECT\_BINARY\_DIR](https://cmake.org/cmake/help/latest/variable/PROJECT_BINARY_DIR.html#variable:PROJECT_BINARY_DIR "PROJECT_BINARY_DIR")， [\<PROJECT-NAME\>\_BINARY\_DIR](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_BINARY_DIR.html#variable:%3CPROJECT-NAME%3E_BINARY_DIR "<PROJECT-NAME>_BINARY_DIR")
项目二进制目录的绝对路径。


- [PROJECT_IS_TOP_LEVEL](https://cmake.org/cmake/help/latest/variable/PROJECT_IS_TOP_LEVEL.html#variable:PROJECT_IS_TOP_LEVEL "PROJECT_IS_TOP_LEVEL"), [\<PROJECT\-NAME\>\_IS\_TOP\_LEVEL](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_IS_TOP_LEVEL.html#variable:%3CPROJECT-NAME%3E_IS_TOP_LEVEL "<PROJECT-NAME>_IS_TOP_LEVEL")
*3.21 版本新增*。 
布尔值，标识当前项目是否为顶层项目。

其余变量由下文 [选项](https://cmake.org/cmake/help/latest/command/project.html#options) 中描述的可选参数进行设置。如果未传入某个选项，则对应的变量会被置为空字符串。

注意：形如 `<name>_SOURCE_DIR`、`<name>_BINARY_DIR` 的变量，也允许在调用 `project()` 之前被其他命令设置（例如[FetchContent_MakeAvailable()](https://cmake.org/cmake/help/latest/module/FetchContent.html#command:fetchcontent_makeavailable "fetchcontent_makeavailable") 就是一例）。项目代码不应当指望 `<PROJECT‑NAME>_SOURCE_DIR`、`<PROJECT‑NAME>_BINARY_DIR` 在脱离 `project()` 调用作用域、以及它的子作用域之外仍保持某个特定值。

*3.30 版本变更*： 当执行 `project(<PROJECT‑NAME> ...)` 时，如果 `<PROJECT‑NAME>_SOURCE_DIR`、`<PROJECT‑NAME>_BINARY_DIR`、`<PROJECT‑NAME>_IS_TOP_LEVEL` 已经设置为普通变量，则该调用会更新它们。同名的缓存条目依旧会照常设置。详细参见 3.30.3、3.30.4、3.30.5 的版本说明文档。

*3.31 版本变更*： `project(<PROJECT‑NAME> ...)` 命令总会把 `<PROJECT‑NAME>_SOURCE_DIR`、`<PROJECT‑NAME>_BINARY_DIR`、`<PROJECT‑NAME>_IS_TOP_LEVEL` 设置为普通变量，参见策略[CMP0180](https://cmake.org/cmake/help/latest/policy/CMP0180.html#policy:CMP0180 "CMP0180")。同名的缓存条目依旧会照常设置。

# 选项
可用选项：

- `VERSION <version>` 
可选；允许不使用，除非策略 [CMP0048](https://cmake.org/cmake/help/latest/policy/CMP0048.html#policy:CMP0048) 设置为 `NEW`。 

  接收由若干非负整数组成的版本参数`<version>`，格式为 `<主版本>[.<次版本>[.<补丁版本>[.<微调版本>]]]`，并设置如下变量：

  - [PROJECT_VERSION](https://cmake.org/cmake/help/latest/variable/PROJECT_VERSION.html#variable:PROJECT_VERSION)、[\<PROJECT‑NAME>_VERSION](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_VERSION.html#variable:%3CPROJECT-NAME%3E_VERSION) 
  - [PROJECT_VERSION_MAJOR](https://cmake.org/cmake/help/latest/variable/PROJECT_VERSION_MAJOR.html#variable:PROJECT_VERSION_MAJOR)、[<PROJECT‑NAME>_VERSION_MAJOR](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_VERSION_MAJOR.html#variable:%3CPROJECT-NAME%3E_VERSION_MAJOR) 
  - [PROJECT_VERSION_MINOR](https://cmake.org/cmake/help/latest/variable/PROJECT_VERSION_MINOR.html#variable:PROJECT_VERSION_MINOR)、[<PROJECT‑NAME>_VERSION_MINOR](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_VERSION_MINOR.html#variable:%3CPROJECT-NAME%3E_VERSION_MINOR) 
  - [PROJECT_VERSION_PATCH](https://cmake.org/cmake/help/latest/variable/PROJECT_VERSION_PATCH.html#variable:PROJECT_VERSION_PATCH)、[<PROJECT‑NAME>_VERSION_PATCH](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_VERSION_PATCH.html#variable:%3CPROJECT-NAME%3E_VERSION_PATCH) 
  - [PROJECT_VERSION_TWEAK](https://cmake.org/cmake/help/latest/variable/PROJECT_VERSION_TWEAK.html#variable:PROJECT_VERSION_TWEAK)、[<PROJECT‑NAME>_VERSION_TWEAK](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_VERSION_TWEAK.html#variable:%3CPROJECT-NAME%3E_VERSION_TWEAK)

  3.12 版本新增：如果在顶层 CMakeLists.txt 中调用 project() 命令，该版本号还会保存到变量 CMAKE_PROJECT_VERSION。



**COMPAT_VERSION** 4.3 版本新增。 可选；必须同时设置 VERSION 才能使用。 接收由若干非负整数组成的版本参数，格式为 `<主版本>[.<次版本>[.<补丁版本>[.<微调版本>]]]`，并设置变量： PROJECT_COMPAT_VERSION、<PROJECT‑NAME>_COMPAT_VERSION。 如果在顶层 CMakeLists.txt 中调用 project() 命令，该兼容版本号还会保存到变量 CMAKE_PROJECT_COMPAT_VERSION。

**SPDX_LICENSE <license‑string>** 4.3 版本新增。 可选。将变量 PROJECT_SPDX_LICENSE、<PROJECT‑NAME>_SPDX_LICENSE 设置为 <license‑string>。该参数必须为 SPDX 许可证表达式，用来描述整个项目的许可证，除软件制品外，还包含文档、资源以及随项目一同分发的其他资料。可查阅SPDX网站获取常用许可证及其标识符。可通过目标属性 SPDX_LICENSE 为单个软件制品指定许可证。

> 
> 
> 注意 项目许可证不会用于初始化各个独立目标的 SPDX_LICENSE 属性。这一设计可以让导出包信息时所指定的包许可证、组件默认许可证具备实际意义。只有通用包规范导出功能会使用该信息。 部分场景下项目许可证会被继承为包许可证。更多细节请查阅 export、install 命令文档中关于 PROJECT 选项的相关内容。
> 
> 

**DESCRIPTION <description‑string>** 3.9 版本新增。 可选。将变量 PROJECT_DESCRIPTION、<PROJECT‑NAME>_DESCRIPTION 设置为 <description‑string>。建议描述文本尽量简短，通常不超过若干个单词。 如果在顶层 CMakeLists.txt 中调用 project() 命令，该描述文本还会保存到变量 CMAKE_PROJECT_DESCRIPTION。 3.12 版本新增 <PROJECT‑NAME>_DESCRIPTION 变量。

**HOMEPAGE_URL <url‑string>** 3.12 版本新增。 可选。将变量 PROJECT_HOMEPAGE_URL、<PROJECT‑NAME>_HOMEPAGE_URL 设置为 <url‑string>，该地址应为项目的标准主页链接。 如果在顶层 CMakeLists.txt 中调用 project() 命令，该链接还会保存到变量 CMAKE_PROJECT_HOMEPAGE_URL。

**LANGUAGES <language‑name>...** 可选。也可以使用简短调用形式，省略 LANGUAGES 关键字。 指定构建项目所需要启用的编程语言。

支持的语言：

*   C
*   CXX（即 C++）
*   CSharp：3.8版本新增，C#
*   CUDA：3.8版本新增
*   OBJC：3.16版本新增，Objective‑C
*   OBJCXX：3.16版本新增，Objective‑C++
*   Fortran
*   HIP：3.21版本新增
*   ISPC：3.18版本新增
*   Swift：3.15版本新增
*   ASM：由C编译器提供支持的汇编语言。启用ASM时请放在列表末尾，以便CMake检测C或CXX编译器是否支持汇编。
*   ASM_NASM：Netwide 汇编器
*   ASM_MARMASM：3.26版本新增，面向ARM、ARM64的微软汇编器
*   ASM_MASM：面向x86、x64的微软汇编器
*   ASM_POASM：4.4版本新增，Pelles C工具链汇编器
*   ASM‑ATT

不给出任何语言选项时，默认启用 C 和 CXX。指定语言为 NONE，或者使用 LANGUAGES 关键字但后面不写任何语言，则不会启用任何编程语言。

通过 VERSION、COMPAT_VERSION、SPDX_LICENSE、DESCRIPTION、HOMEPAGE_URL 选项设置的变量，用作软件包元数据与文档的默认值。在生成通用包规范的包描述时，export 和 install 命令会相应使用这些变量。



------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------