---
title: cmake-buildsystem(7)
tags: cmake参考手册
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
[TOC] 

# 介绍


基于<span id="manual:cmake-buildsystem(7)">CMake的构建系统</span>被组织为一组高层次的逻辑目标。每个目标对应一个可执行文件或库，或者是一个包含自定义命令的自定义目标。目标之间的依赖关系在构建系统中被明确表示出来，以确定构建顺序以及响应变化时的重新生成规则。

# 二进制目标
可执行文件和库是通过 [add_executable()](#add_executable)和 [add_library()](#add_library) 命令定义的。生成的二进制文件会根据目标平台具有相应的 [PREFIX](#PREFIX)、[SUFFIX](#SUFFIX) 和扩展名。二进制目标之间的依赖关系通过 [target_link_libraries()](#target_link_libraries) 命令来表达：

```cmake
add_library(archive archive.cpp zip.cpp lzma.cpp)
add_executable(zipapp zipapp.cpp)
target_link_libraries(zipapp archive)
```
`archive` 被定义为一个`STATIC`库——一个包含从`archive.cpp`、`zip.cpp`和`lzma.cpp`编译而来的对象的归档文件。`zipapp`被定义为由编译并链接`zipapp.cpp`生成的可执行文件。在链接`zipapp`可执行文件时，会链接该静态库`archive `。

## 可执行文件
<span id="Executables">可执行文件</span>是通过将目标文件链接在一起生成的二进制文件，其中一个目标文件包含程序的入口点，例如 `main`。 

[add_executable()](#add_executable) 命令用于定义一个可执行目标： 

```cmake
add_executable(mytool mytool.cpp) 
``` 

CMake 会生成构建规则以将源文件编译为目标文件，并将它们链接成一个可执行文件。 

可以使用 [target_link_libraries()](#target_link_libraries)命令指定可执行文件的链接依赖项。链接器首先从可执行文件自身源文件编译生成的目标文件开始，然后通过搜索链接库来解析剩余的符号依赖关系。 

诸如 [add_custom_command()](#add_custom_command) 之类的命令（该命令会生成在构建时运行的规则）可以透明地使用[EXECUTABLE](#prop_tgt:TYPE)目标作为`COMMAND`命令的可执行文件。构建系统规则会确保在尝试运行命令之前，先构建该可执行文件。
## 静态库
静态库是目标文件的归档文件。它们是由归档工具而非链接器生成的。[可执行文件](#Executables)、[共享库](#Shared_Libraries)和[模块库](#Module_Libraries)可以将静态库作为依赖项进行链接。链接器根据需要从静态库中选择目标文件的子集，以解析符号并将它们链接到最终的二进制文件中。每个链接到静态库的二进制文件都会获得该库符号的独立副本，因此静态库本身在运行时是不需要的。

当使用 [add_library()](#add_library) 命令并指定 `STATIC` 库类型时，可以定义一个静态库目标：

```cmake
add_library(archive STATIC archive.cpp zip.cpp lzma.cpp)
```

或者，当 [BUILD_SHARED_LIBS](#BUILD_SHARED_LIBS) 变量为 `false` 时，且命令没有指类型时：

```cmake
add_library(archive archive.cpp zip.cpp lzma.cpp)
```

CMake 会生成构建规则以将源文件编译为目标文件，并将它们归档为静态库。

静态库的链接依赖项可以通过 [target_link_libraries()](#target_link_libraries) 命令来指定。由于静态库是归档文件而非已链接的二进制文件，因此它们的链接依赖项中的目标文件不会包含在静态库本身中（除非目标库被指定为直接链接依赖项）。相反，CMake 会记录静态库的链接依赖项，以便在链接最终二进制文件时进行传递使用。
## 动态库
## APP框架
## 模块库
## 目标库

# 构建规范与使用要求
## 目标命令
## 目标构建规范
### 目标编译属性 
### 目标链接属性
## 目标用法要求
### 传递编译属性
### 传递链接属性
## 自定义传递属性 
## 兼容接口属性 
## 属性来源调试 
## 使用生成器表达式构建规范 
### 包含目录和使用要求 
## 链接库和生成器表达式 
## 输出产物 
### 运行时输出产物 
### 库输出产物 
### 归档输出产物 
## 目录范围命令 
# 构建配置 
## 大小写敏感性 
## 默认和自定义配置 
# 伪目标 
## 导入目标 
## 别名目标 
## 接口库 
### 允许在接口库上设置的属性



------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
