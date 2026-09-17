---
title: add_link_options
tags: cmake,cmake命令
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

指定用于链接给定目标及其依赖的库或标志。被链接的库目标的目标[使用要求](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#target-usage-requirements)将会传递传播。一个目标的依赖项的使用要求会影响该目标自身源码的编译。

# 概述
本命令存在多种调用形式，详见后续小节。通用形式如下：

```cmake
target_link_libraries(<target> ... <item>... ...)
```

`<target>` 必须已经通过 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable)、[add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) 这类命令创建，禁止为[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)。若策略 [CMP0079](https://cmake.org/cmake/help/latest/policy/CMP0079.html#policy:CMP0079) 未设置为 `NEW`，则该目标必须在当前目录内创建。对同一个 `<target>` 多次调用本命令，会按调用顺序追加条目。

*3.13 版本新增*：`<target>` 不再强制要求和 `target_link_libraries` 调用处在同一个目录。

每一个 `<item>` 可以是下面其中一类：

- **库目标名**：生成的链接行会带上该目标对应可链接库文件的完整路径。构建系统会生成依赖关系：当库文件发生变更时，会重新链接 `<target>`。
	
	该目标必须是项目内通过 [add_library()](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) 创建，或是[导入库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)。如果是项目内创建的库，构建系统会自动增加顺序依赖，保证在链接 `<target>` 之前，该库目标已经为最新版本。
	
	若导入库设置了 [IMPORTED_NO_SONAME](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_NO_SONAME.html#prop_tgt:IMPORTED_NO_SONAME) 目标属性，CMake 会让链接器去搜索库，而不是直接使用完整路径（例如 `/usr/lib/libfoo.so` 会变成 `-lfoo`）。 
	
	目标产物的完整路径会自动做 Shell 引号转义处理。

- **库文件的完整路径**：生成的链接行一般直接保留文件完整路径。库文件变更时，构建系统会触发 `<target>` 的重新链接。
	
	部分场景下 CMake 仍会改为让链接器搜索库（e.g. `/usr/lib/libfoo.so` becomes `-lfoo`），例如检测到共享库没有 SONAME 字段。CMake 4.0 之前版本，另一种场景参见策略 [CMP0060](https://cmake.org/cmake/help/latest/policy/CMP0060.html#policy:CMP0060)。
	
	如果库文件是 macOS Framework ，框架的 `Headers` 目录会被当作目标[使用要求](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#target-usage-requirements)处理，效果等同于把框架目录作为头文件包含目录。
	
	*3.28 版本新增*：Apple 平台下，库文件允许指向 `.xcframework` 文件夹；此时目标会将选中库的 Headers 目录作为使用要求。
	
	*3.8 版本新增*：VS2010 及以上的 [Visual Studio 生成器](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html#visual-studio-generators)，后缀为 `.targets` 的库文件会被识别为 MSBuild 目标文件，并导入生成的工程文件；其他生成器不支持该行为。 
	
	库文件完整路径会自动完成 Shell 引号转义。

- **普通库名称**：生成的链接行会要求链接器搜索该库（例如 `foo` 变为 `-lfoo` 或者 `foo.lib`）。
 	
 	库名/标志直接作为命令行片段使用，不会额外增加引号与转义。

- **链接标志**：以 `-` 开头，但不是 `-l`、`-framework`的条目，会被当作链接器标志。注意：在传递依赖逻辑中，这类标志会被当作普通库链接条目处理。因此建议仅作为 `PRIVATE` 私有链接项，禁止传递给依赖方。
	
	此处填入的链接标志，会插入在链接命令链接库文件所在位置。受链接器行为影响，该位置不一定正确。建议使用目标属性 [LINK_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/LINK_OPTIONS.html#prop_tgt:LINK_OPTIONS) 或者命令 [target_link_options](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options) 显式添加链接标志，标志会被放置在工具链规定的正确位置。
	
	*3.13 版本新增*：目标属性 [LINK_OPTIONS](https://cmake.org/cmake/help/latest/prop_tgt/LINK_OPTIONS.html#prop_tgt:LINK_OPTIONS) 或者命令 [target_link_options](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options) ；更早 CMake 版本请使用旧属性 [LINK_FLAGS](https://cmake.org/cmake/help/latest/prop_tgt/LINK_FLAGS.html#prop_tgt:LINK_FLAGS)。 
	
	链接标志直接作为命令行片段，不会额外引号转义。

- **生成器表达式**：`$<...>` [生成器表达式](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7))求值结果可以是上面任意一种条目或它们的[分号分隔的条目列表](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#cmake-language-lists)。 如果`...`求值后含有分号 `;`（例如变量 `${list}`），必须用引号 `"$<...>"` 包裹，保证本命令将其识别为单个 `<item>`。 
	
	生成器表达式也可以作为上述条目的片段，例如 `foo$<1:_d>`。

- `debug` / `optimized` / `general` 关键字，后跟一个 `<item>`。这种`<item>`仅对对应构建配置有用。`debug`：用于 Debug 配置（或则若设置全局属性 [DEBUG_CONFIGURATIONS](https://cmake.org/cmake/help/latest/prop_gbl/DEBUG_CONFIGURATIONS.html#prop_gbl:DEBUG_CONFIGURATIONS)，则对应该属性定义的配置集）。`optimized`：用于除 `Debug` 之外其余全部配置。`general`：适用于全部配置，可以省略。该关键字由本命令直接解析；如果关键字是生成器表达式求值得到，则失去特殊含义。
	
	更细粒度的按配置链接，推荐使用 [\$\<CONFIG:...\>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:CONFIG) 生成器表达式。更结构化的方式是使用设置了 [IMPORTED\_CONFIGURATIONS](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_CONFIGURATIONS.html#prop_tgt:IMPORTED_CONFIGURATIONS) 的[导入库目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)做链接，在 Find‑Module 模块中尤为常用。

包含 `::` 的条目（如 `Foo::Bar`） 会被识别为[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)或者[别名库目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)；不存在该目标时直接报错。参见策略 [CMP0028](https://cmake.org/cmake/help/latest/policy/CMP0028.html#policy:CMP0028)。

CMake 在链接命令行上排布直接依赖的顺序规则，请参见变量 [CMAKE_LINK_LIBRARIES_STRATEGY](https://cmake.org/cmake/help/latest/variable/CMAKE_LINK_LIBRARIES_STRATEGY.html#variable:CMAKE_LINK_LIBRARIES_STRATEGY) 以及目标属性 [LINK_LIBRARIES_STRATEGY](https://cmake.org/cmake/help/latest/prop_tgt/LINK_LIBRARIES_STRATEGY.html#prop_tgt:LINK_LIBRARIES_STRATEGY)。

更多构建系统属性定义，参见手册 [cmake‑buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7))。

# 编译器驱动差异处理
*4.0 版本新增*。

要向链接器工具传递选项，各个编译器驱动拥有各自的语法。可以使用 `LINKER:` 前缀配合逗号分隔符，以可移植的方式指定需要传递给链接器工具的选项。`LINKER:` 会被替换为对应的编译器驱动包装选项，逗号会被替换为对应的驱动分隔符。该驱动前缀与驱动分隔符由变量 [\<LANG\>\_LINKER\_WRAPPER\_FLAG](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_LINKER_WRAPPER_FLAG.html#variable:CMAKE_%3CLANG%3E_LINKER_WRAPPER_FLAG)和 [\<LANG>\_LINKER\_WRAPPER\_FLAG\_SEP](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_LINKER_WRAPPER_FLAG_SEP.html#variable:CMAKE_%3CLANG%3E_LINKER_WRAPPER_FLAG_SEP) 的取值提供。

示例：`"LINKER:-z,defs"`
* Clang：展开为 `-Xlinker -z -Xlinker defs`
* GCC：展开为 `-Wl,-z,defs`

`LINKER:` 支持另一种语法，内部嵌套 `SHELL:`、空格分隔参数，上例等价写法： `"LINKER:SHELL:-z defs"`

> **注意**：禁止在 `LINKER:` 前缀以外的位置写 `SHELL:`。


# 目标 和/或 其依赖项的库

```cmake
target_link_libraries(<target> {INTERFACE|PUBLIC|PRIVATE} <item>... [{INTERFACE|PUBLIC|PRIVATE} <item>...]...)
```

`PUBLIC`、`PRIVATE`、`INTERFACE` [作用域](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#target-command-scope)关键字，可以在同一条命令同时指定链接依赖与链接接口。

`PUBLIC`：库会链接到当前目标，同时纳入链接接口，传递给依赖当前目标的上层。`PRIVATE`：库会链接到当前目标，但不纳入链接接口，不会向外传递。`INTERFACE`：追加到链接接口向外传递，不会用于链接当前目标。

# 目标和其依赖项的库

```cmake
target_link_libraries(<target> <item>...)
```

该调用形式默认库依赖具备传递性。当别的目标链接本目标时，本目标链接的库也会出现在对方链接行。 传递的“链接接口”保存在目标属性 [INTERFACE_LINK_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_LIBRARIES.html#prop_tgt:INTERFACE_LINK_LIBRARIES)，也可以直接修改该属性覆盖。

CMake4.0 之前，若策略 [CMP0022](https://cmake.org/cmake/help/latest/policy/CMP0022.html#policy:CMP0022) 不为 `NEW`，传递链接行为依然生效，但会受旧属性 [LINK_INTERFACE_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_LIBRARIES.html#prop_tgt:INTERFACE_LINK_LIBRARIES) 覆盖。调用该命令的其他形式会改写该属性，使得仅由此形式添加的库变为私有。

# 目标 和/或 其依赖项的库（遗留）
此用法仅用于兼容性。请优先使用 `PUBLIC` 或 `PRIVATE` 关键字。
```cmake
target_link_libraries(<target> <LINK_PRIVATE|LINK_PUBLIC> <lib>... [<LINK_PRIVATE|LINK_PUBLIC> <lib>...]...)
```

`LINK_PUBLIC`、`LINK_PRIVATE` 可在一个命令中同时设置链接依赖与链接接口。

跟在 `LINK_PUBLIC` 后面的库与目标会被链接，同时会被加入到 [INTERFACE_LINK_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_LIBRARIES.html#prop_tgt:INTERFACE_LINK_LIBRARIES) 中。 

在 CMake 4.0 之前的版本，如果策略 [CMP0022](https://cmake.org/cmake/help/latest/policy/CMP0022.html#policy:CMP0022) 未设置为 `NEW`，它们也会被加入到 [LINK_INTERFACE_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/LINK_INTERFACE_LIBRARIES.html#prop_tgt:LINK_INTERFACE_LIBRARIES)属性。 跟在 `LINK_PRIVATE` 后面的库与目标会参与本目标的链接，但不会被加入到  [INTERFACE_LINK_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_LIBRARIES.html#prop_tgt:INTERFACE_LINK_LIBRARIES) （以及旧属性 [LINK_INTERFACE_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/LINK_INTERFACE_LIBRARIES.html#prop_tgt:LINK_INTERFACE_LIBRARIES)）之中。

# 仅依赖项的库（遗留）
此用法仅用于兼容性。请优先使用 `INTERFACE` 关键字。

```cmake
target_link_libraries(<target> LINK_INTERFACE_LIBRARIES <item>...)
```


`LINK_INTERFACE_LIBRARIES` 模式会将库追加到目标属性 [INTERFACE_LINK_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_LIBRARIES.html#prop_tgt:INTERFACE_LINK_LIBRARIES)，而不会将这些库用于当前目标的链接过程。

在 CMake 4.0 之前的版本中，如果策略 [CMP0022](https://cmake.org/cmake/help/latest/policy/CMP0022.html#policy:CMP0022) 未设置为 `NEW`，该模式还会把库追加到 [LINK_INTERFACE_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/LINK_INTERFACE_LIBRARIES.html#prop_tgt:LINK_INTERFACE_LIBRARIES) 属性，以及其对应各构建配置。

# 链接对象库

*3.12 版本新增*。

[对象库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#object-libraries)（Object Libraries）可以作为 `target_link_libraries` 的第一个参数（即`<target>`），用于指定其源码对其他库的依赖关系。例如下面代码：

```cmake
add_library(A SHARED a.c)
target_compile_definitions(A PUBLIC A)
add_library(obj OBJECT obj.c)
target_compile_definitions(obj PUBLIC OBJ)
target_link_libraries(obj PUBLIC A)
```

编译 `obj.c` 时会带上 `-DA -DOBJ`，并且为 `obj` 建立会传递给其依赖方的使用要求。

普通库和可执行文件可以链接[对象库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#object-libraries)，从而获取它的目标文件以及使用要求。延续上面示例，执行代码：

```cmake
add_library(B SHARED b.c)
target_link_libraries(B PUBLIC obj)
```

编译 `b.c` 会带上 `-DA -DOBJ`；生成共享库 `B`，包含来自 `b.c` 和 `obj.c` 的目标文件，并且将 `B` 链接到 `A`。进一步看下面代码：

```cmake
add_executable(main main.c)
target_link_libraries(main B)
```

编译 `main.c` 带上 `-DA -DOBJ`，可执行文件 `main` 会链接 `B` 和 `A`。对象库的使用要求会经由 `B` 传递下去，但对象库本身的目标文件不会跟着传递。

[对象库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#object-libraries)也可以“链接”其他对象库以获取使用要求；但对象库本身没有链接步骤，因此不会处理对方的目标文件。接上面示例：

```cmake
add_library(obj2 OBJECT obj2.c)
target_link_libraries(obj2 PUBLIC obj)
add_executable(main2 main2.c)
target_link_libraries(main2 obj2)
```

编译 `obj2.c` 带上 `-DA -DOBJ`；生成可执行文件 `main2`，包含 `main2.c` 与 `obj2.c` 的目标文件，同时 `main2` 链接到 `A`。

换言之： 当[对象库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#object-libraries)出现在某个目标的 [INTERFACE_LINK_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_LINK_LIBRARIES.html#prop_tgt:INTERFACE_LINK_LIBRARIES) 属性中时，它会被当作[接口库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#interface-libraries)（Interface Libraries）处理； 而当对象库出现在某个目标的 [LINK_LIBRARIES](https://cmake.org/cmake/help/latest/prop_tgt/LINK_LIBRARIES.html#prop_tgt:LINK_LIBRARIES) 属性中时，它的目标文件也会一并参与链接。


# 通过 `$<TARGET_OBJECTS>` 链接对象库

*3.21 版本新*增。

对象库对应的目标文件可以通过生成器表达式 [\$\<TARGET_OBJECTS>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:TARGET_OBJECTS) 进行引用。这类目标文件会被放置在链接行上所有库的前面，不其相对顺序影响。此外，构建系统会增加顺序依赖，保证依赖目标执行链接前，该对象库已经编译为最新版本。例如如下代码

```cmake
add_library(obj3 OBJECT obj3.c)
target_compile_definitions(obj3 PUBLIC OBJ3)
add_executable(main3 main3.c)
target_link_libraries(main3 PRIVATE a3 $<TARGET_OBJECTS:obj3> b3)
```

链接可执行文件 `main3`时 ，先链接 `main3.c` 和 `obj3.c` 的目标文件，之后再链接库 `a3`、`b3`。编译 `main3.c` 时不会带入 `obj3` 的使用要求，例如 `-DOBJ3`。

该方式被用于实现在链接行中传递目标文件的包含关系作为使用要求。延续上例，如下代码

```cmake
add_library(iface_obj3 INTERFACE)
target_link_libraries(iface_obj3 INTERFACE obj3 $<TARGET_OBJECTS:obj3>)
```

创建接口库 `iface_obj3`，它会转发 `obj3` 的使用要求，同时将 `obj3` 的目标文件添加到依赖方的链接行。如下代码

```camke
add_executable(use_obj3 use_obj3.c)
target_link_libraries(use_obj3 PRIVATE iface_obj3)
```

编译 `use_obj3.c` 时会带上 `-DOBJ3`；链接`use_obj3.c` 与 `obj3.c` 的目标文件为可执行文件 `use_obj3`。

该机制也可以通过静态库传递生效。由于静态库本身不执行链接步骤，它不会消耗以此方式引用的对象库的目标文件。取而代之，这些目标文件会成为该静态库的传递链接依赖。延续示例, 如下代码

```cmake
add_library(static3 STATIC static3.c)
target_link_libraries(static3 PRIVATE iface_obj3)
add_executable(use_static3 use_static3.c)
target_link_libraries(use_static3 PRIVATE static3)
```

编译 `static3.c`时会带上 `-DOBJ3`，生成的 `libstatic3.a` 仅包含它自身的目标文件。 由于是 `static3`的私有依赖，使用要求不会向外传递，因此编译 `use_static3.c` 时不会定义 `-DOBJ3`。 但是 `static3` 的链接依赖会被传递，其中包含 `iface_obj3` 对 `$<TARGET_OBJECTS:obj3>` 的引用。最终生成的可执行文件 `use_static3` 包含 `use_static3.c` 和 `obj3.c` 的目标文件，同时链接 `libstatic3.a`。

使用该方案时，项目需要自行保证不要将多个二进制程序同时链接到 `iface_obj3`，否则每一个二进制的链接行都会带入 `obj3` 的目标文件。

> **注意**: 在 3.21 之前的 CMake 版本中，部分场景下也可以在 `target_link_libraries` 中使用 [\$\<TARGET_OBJECTS>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:TARGET_OBJECTS)，但并未得到完整支持：
>- 不会把目标文件放在链接行所有库的前面；
>- 不会增加针对对象库的编译顺序依赖；
>- 在 Xcode 的多架构编译场景无法正常工作。

# 静态库循环依赖



------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------