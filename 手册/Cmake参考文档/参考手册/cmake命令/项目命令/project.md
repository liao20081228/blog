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

  *3.12 版本新增*：如果在顶层 CMakeLists.txt 中调用 project() 命令，该版本号还会保存到变量 [CMAKE_PROJECT_VERSION](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_VERSION.html#variable:CMAKE_PROJECT_VERSION)。



- `COMPAT_VERSION <version>`
*4.3 版本新增*。

  可选；必须同时设置 `VERSION` 才能使用。 
  
  接收由若干非负整数组成的版本参数`<version>`，格式为 `<主版本>[.<次版本>[.<补丁版本>[.<微调版本>]]]`，并设置变量： 
  - [PROJECT_COMPAT_VERSION](https://cmake.org/cmake/help/latest/variable/PROJECT_COMPAT_VERSION.html#variable:PROJECT_COMPAT_VERSION)、[<PROJECT‑NAME>_COMPAT_VERSION](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_COMPAT_VERSION.html#variable:%3CPROJECT-NAME%3E_COMPAT_VERSION)。 
   
  如果在顶层 `CMakeLists.txt` 中调用 `project()` 命令，该兼容版本号还会保存到变量 [CMAKE_PROJECT_COMPAT_VERSION](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_COMPAT_VERSION.html#variable:CMAKE_PROJECT_COMPAT_VERSION)。

- `SPDX_LICENSE <license‑string>`
*4.3 版本新增*。 

  可选。将变量 
  - [PROJECT_SPDX_LICENSE](https://cmake.org/cmake/help/latest/variable/PROJECT_SPDX_LICENSE.html#variable:PROJECT_SPDX_LICENSE)、[<PROJECT‑NAME>_SPDX_LICENSE](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_SPDX_LICENSE.html#variable:%3CPROJECT-NAME%3E_SPDX_LICENSE) 
  
  设置为 `<license‑string>`。该参数必须为 [SPDX](https://spdx.dev/) [许可证表达式](https://spdx.github.io/spdx-spec/v3.0.1/annexes/spdx-license-expressions/)，用来描述整个项目的许可证，除软件制品外，还包含文档、资源以及随项目一同分发的其他资料。可查阅SPDX网站[许可证列表](https://spdx.org/licenses/)获取常用许可证及其标识符。可通过目标属性 [SPDX_LICENSE](https://cmake.org/cmake/help/latest/prop_tgt/SPDX_LICENSE.html#prop_tgt:SPDX_LICENSE) 为单个软件制品指定许可证。

> **注意** 项目许可证不会用于初始化各个独立目标的 [SPDX_LICENSE](https://cmake.org/cmake/help/latest/prop_tgt/SPDX_LICENSE.html#prop_tgt:SPDX_LICENSE) 属性。这一设计可以让导出包信息时所指定的包许可证、组件默认许可证具备实际意义。只有通用包规范导出功能会使用该信息。
> 
>部分场景下项目许可证会被继承为包许可证。更多细节请查阅 [export()](https://cmake.org/cmake/help/latest/command/export.html#command:export)、[install()](https://cmake.org/cmake/help/latest/command/install.html#command:install)命令文档中关于 `PROJECT` 选项的相关内容。

- `DESCRIPTION <description‑string>`
*3.9 版本新增*。 

  可选。将变量
  - [PROJECT_DESCRIPTION](https://cmake.org/cmake/help/latest/variable/PROJECT_DESCRIPTION.html#variable:PROJECT_DESCRIPTION)、[<PROJECT‑NAME>_DESCRIPTION](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_DESCRIPTION.html#variable:%3CPROJECT-NAME%3E_DESCRIPTION) 
   
  设置为 `<description‑string>`。建议描述文本尽量简短，通常不超过若干个单词。 
  
  如果在顶层 `CMakeLists.txt` 中调用 `project()`命令，该描述文本还会保存到变量 [CMAKE_PROJECT_DESCRIPTION](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_DESCRIPTION.html#variable:CMAKE_PROJECT_DESCRIPTION)。 
  
  *3.12 版本新增* : `<PROJECT‑NAME>_DESCRIPTION` 变量。

- `HOMEPAGE_URL <url‑string>`
*3.12 版本新增*。 

  可选。将变量 
  - [PROJECT_HOMEPAGE_URL](https://cmake.org/cmake/help/latest/variable/PROJECT_HOMEPAGE_URL.html#variable:PROJECT_HOMEPAGE_URL)、[<PROJECT‑NAME>_HOMEPAGE_URL](https://cmake.org/cmake/help/latest/variable/PROJECT-NAME_HOMEPAGE_URL.html#variable:%3CPROJECT-NAME%3E_HOMEPAGE_URL) 

  设置为 `<url‑string>`，该地址应为项目的标准主页链接。 
  
  如果在顶层 `CMakeLists.txt` 中调用 `project()` 命令，该链接还会保存到变量 [CMAKE_PROJECT_HOMEPAGE_URL](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_HOMEPAGE_URL.html#variable:CMAKE_PROJECT_HOMEPAGE_URL)。

- `LANGUAGES <language‑name>...`
可选。也可以使用简短调用形式，省略 `LANGUAGES` 关键字。 

  指定构建项目所需要启用的编程语言。

  支持的语言：

  - C
  - CXX（即 C++）
  - CSharp：*3.8版本新增*，C#
  - CUDA：*3.8版本新增*
  - OBJC：*3.16版本新增*，Objective‑C
  - OBJCXX：*3.16版本新增*，Objective‑C++
  - Fortran
  - HIP：*3.21版本新增*
  - ISPC：*3.18版本新增*
  - Swift：*3.15版本新增*
  - ASM：由C编译器提供支持的汇编语言。启用ASM时请放在列表末尾，以便CMake检测C或CXX编译器是否支持汇编。
  - ASM_NASM：Netwide 汇编器
  - ASM_MARMASM：*3.26版本新增*，面向ARM、ARM64的微软汇编器
  - ASM_MASM：面向x86、x64的微软汇编器
  - ASM_POASM：*4.4版本新增*，Pelles C工具链汇编器
  - ASM‑ATT

不给出任何语言选项时，默认启用 `C` 和 `CXX`。指定语言为 `NONE`，或者使用 `LANGUAGES` 关键字但后面不写任何语言，则不会启用任何编程语言。

通过 `VERSION`、`COMPAT_VERSION`、`SPDX_LICENSE`、`DESCRIPTION`、`HOMEPAGE_URL` 选项设置的变量，用作软件包元数据与文档的默认值。在生成通用包规范的包描述时，[export()](https://cmake.org/cmake/help/latest/command/export.html#command:export)和 [install()](https://cmake.org/cmake/help/latest/command/install.html#command:install)命令会相应使用这些变量。

# 代码注入

用户可以定义多个变量，用于指定在执行 `project()` 命令的不同阶段需要引入的文件。下文概述调用 `project()` 期间执行的步骤：


- *3.15 版本新增*：每一次调用 `project()`，无论项目名称是什么，如果 [CMAKE_PROJECT_INCLUDE_BEFORE](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_INCLUDE_BEFORE.html#variable:CMAKE_PROJECT_INCLUDE_BEFORE) 已设置，则引入该变量指定的文件与模块。

- *3.17 版本新增*：如果 `project()` 命令的项目名为 `<PROJECT‑NAME>`，若 [CMAKE_PROJECT_<PROJECT‑NAME>_INCLUDE_BEFORE](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_PROJECT-NAME_INCLUDE_BEFORE.html#variable:CMAKE_PROJECT_%3CPROJECT-NAME%3E_INCLUDE_BEFORE) 已设置，则引入该变量指定的文件与模块。

- 设置前文 [synopsis](https://cmake.org/cmake/help/latest/command/project.html#synopsis) 与 [options](https://cmake.org/cmake/help/latest/command/project.html#options) 章节所详述的各类项目专属变量。

- 仅针对第一次 `project()` 调用：

  - 如果 [CMAKE_TOOLCHAIN_FILE](https://cmake.org/cmake/help/latest/variable/CMAKE_TOOLCHAIN_FILE.html#variable:CMAKE_TOOLCHAIN_FILE) 已设置，至少读取一次该工具链文件。该文件可能被多次读取，后续启用编程语言时也可能再次读取（见下文）。
  - 设置描述主机与目标平台的相关变量。此时不一定会设置语言专属变量。首次运行时，仅可能存在工具链文件所定义的语言相关变量；后续运行时，会加载上一次运行缓存的语言专属变量。
  - *3.24 版本新增*：如果 [CMAKE_PROJECT_TOP_LEVEL_INCLUDES](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_TOP_LEVEL_INCLUDES.html#variable:CMAKE_PROJECT_TOP_LEVEL_INCLUDES) 已设置，则依次引入该变量列出的每一个文件。CMake 在此之后会忽略该变量。

- 启用本次调用所指定的编程语言；未指定语言则启用默认语言。首次启用某一门编程语言时，工具链文件可能会被重新读取。

- *3.15 版本新增*：每一次调用 `project()`，无论项目名称是什么，如果 [CMAKE_PROJECT_INCLUDE](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_INCLUDE.html#variable:CMAKE_PROJECT_INCLUDE) 已设置，则引入该变量指定的文件与模块。

- 如果 `project()` 命令的项目名为 `<PROJECT‑NAME>`，若 [CMAKE_PROJECT_<PROJECT‑NAME>_INCLUDE](https://cmake.org/cmake/help/latest/variable/CMAKE_PROJECT_PROJECT-NAME_INCLUDE.html#variable:CMAKE_PROJECT_%3CPROJECT-NAME%3E_INCLUDE) 已设置，则引入该变量指定的文件与模块。

# 用法

项目的顶层 `CMakeLists.txt` 文件必须直接显式调用 `project()` 命令；通过 [include](https://cmake.org/cmake/help/latest/command/include.html#command:include) 命令加载该调用是不够的。如果不存在此类调用，CMake 会输出一条警告，并在文件顶部隐式生成 `project(Project)`，以此启用默认编程语言（C 和 CXX）。


> 注意 应当在顶层 `CMakeLists.txt` 的靠前位置调用 `project()` 命令，但要放在 [cmake_minimum_required](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html#command:cmake_minimum_required) 命令之后。 需要优先确定版本与策略设置，之后再调用会受这些设置影响的其他命令，这点十分重要。因此如果不遵守该顺序，`project()` 命令会产生警告。另可参阅策略 [CMP0000](https://cmake.org/cmake/help/latest/policy/CMP0000.html#policy:CMP0000)。



------

<a class="xsj_anchor xsj_anchor_range xsj_anchor_range_end" name="xsj_1789718102368"></a>
***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------