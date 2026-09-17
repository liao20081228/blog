---
title: add_library
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
使用指定的源文件将库添加到项目中。

### 普通库
```cmake
add_library(<name> [<type>] [EXCLUDE_FROM_ALL] <sources>...)
```
添加一个名为 `<name>` 的库目标，该目标将从命令调用中列出的源文件构建而成。 

可选的 `<type>` 指定要创建的库的类型： 

- `STATIC `
[静态库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#static-libraries)：目标文件的归档，用于链接其他目标时使用。 
- `SHARED`
[共享库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#shared-libraries)：一个动态库，可以被其他目标链接并在运行时加载。 
- `MODULE` 
模块库：一个插件，不能被其他目标链接，但可以在运行时使用类似 dlopen 的功能动态加载。 

如果未指定 `<type>`，则默认值根据 [BUILD_SHARED_LIBS](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS "BUILD_SHARED_LIBS") 变量的值设置为 `STATIC` 或 `SHARED`。 

选项包括： 
- `EXCLUDE_FROM_ALL`
自动设置 [EXCLUDE_FROM_ALL](https://cmake.org/cmake/help/latest/prop_tgt/EXCLUDE_FROM_ALL.html#prop_tgt:EXCLUDE_FROM_ALL "EXCLUDE_FROM_ALL") 目标属性。有关该目标属性的详细信息，请参见相关文档。 

`<name>` 对应于逻辑目标名称，并且必须在项目中全局唯一。实际构建的库的文件名是根据本地平台的约定（例如 `lib<name>.a` 或 `<name>.lib`）生成的。 

*从 3.1 版本开始*：`add_library` 的源文件参数可以使用语法为 `$<...>` 的“生成器表达式”。有关可用表达式的详细信息，请参见 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)") 手册。

*从 3.11 版本开始*：如果源文件稍后通过 [target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources "target_sources") 添加，则可以省略这些源文件。

对于 `SHARED` 和 `MODULE` 库，[POSITION_INDEPENDENT_CODE](https://cmake.org/cmake/help/latest/prop_tgt/POSITION_INDEPENDENT_CODE.html#prop_tgt:POSITION_INDEPENDENT_CODE "POSITION_INDEPENDENT_CODE") 目标属性会自动设置为 `ON`。`SHARED` 库还可以标记为[FRAMEWORK](https://cmake.org/cmake/help/latest/prop_tgt/FRAMEWORK.html#prop_tgt:FRAMEWORK "FRAMEWORK") 目标属性，以创建一个 macOS 框架。 

*从 3.8 版本开始*：`STATIC` 库也可以标记为 [FRAMEWORK](https://cmake.org/cmake/help/latest/prop_tgt/FRAMEWORK.html#prop_tgt:FRAMEWORK "FRAMEWORK") 目标属性，以创建一个静态框架。

如果一个库不导出任何符号，则不能将其声明为 `SHARED` 库。例如，Windows 资源 DLL 或者不导出任何非托管符号的托管 C\+\+/CLI DLL 应该被声明为 `MODULE` 库。这是因为 CMake 期望在 Windows 上，`SHARED` 库总是有一个关联的导入库。 

默认情况下，库文件将创建在构建目录目录中，该目录对应于命令调用时所处的源码目录目录。要更改此位置，请参见 [ARCHIVE_OUTPUT_DIRECTORY](https://cmake.org/cmake/help/latest/prop_tgt/ARCHIVE_OUTPUT_DIRECTORY.html#prop_tgt:ARCHIVE_OUTPUT_DIRECTORY "ARCHIVE_OUTPUT_DIRECTORY")、[LIBRARY_OUTPUT_DIRECTORY](https://cmake.org/cmake/help/latest/prop_tgt/LIBRARY_OUTPUT_DIRECTORY.html#prop_tgt:LIBRARY_OUTPUT_DIRECTORY "LIBRARY_OUTPUT_DIRECTORY") 和 [RUNTIME_OUTPUT_DIRECTORY](https://cmake.org/cmake/help/latest/prop_tgt/RUNTIME_OUTPUT_DIRECTORY.html#prop_tgt:RUNTIME_OUTPUT_DIRECTORY "RUNTIME_OUTPUT_DIRECTORY") 目标属性的文档。要更改最终文件名中的 `<name>` 部分，请参见 [OUTPUT_NAME](https://cmake.org/cmake/help/latest/prop_tgt/OUTPUT_NAME.html#prop_tgt:OUTPUT_NAME "OUTPUT_NAME") 目标属性的文档。 

有关定义构建系统属性的更多信息，请参见 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)") 手册。 

另请参阅 [HEADER_FILE_ONLY](https://cmake.org/cmake/help/latest/prop_sf/HEADER_FILE_ONLY.html#prop_sf:HEADER_FILE_ONLY "HEADER_FILE_ONLY")，了解如果某些源文件是预处理的，并且你希望在 IDE 中能够访问原始源文件时，应该如何处理。 

*从 3.30 版本开始*：在不支持共享库的平台上，`add_library` 现在会在调用创建`SHARED` 库时失败，而不是像以前那样自动将其转换为 `STATIC` 库。有关此更改的详细信息，请参见策略 [CMP0164](https://cmake.org/cmake/help/latest/policy/CMP0164.html#policy:CMP0164 "CMP0164")。

### 目标库
```cmake
add_library(<name> OBJECT <sources>...)
```
添加一个[目标库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#object-libraries)，以便在不将目标文件归档或链接到库文件中的情况下编译源文件。 

由 `add_library` 或 [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "add_executable") 创建的其他目标可以使用形如 [$<TARGET_OBJECTS:objlib>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:TARGET_OBJECTS "TARGET_OBJECTS") 的表达式来引用这些目标文件作为源文件，其中 `objlib` 是对象库的名称。例如：
```cmake
add_library(... $<TARGET_OBJECTS:objlib> ...)
add_executable(... $<TARGET_OBJECTS:objlib> ...)
```
将把 objlib 目标文件包含在一个库中，并与从它们自身源代码编译而来的目标文件一起生成可执行文件。目标库只能包含能够编译的源文件、头文件以及其他不会影响普通库链接的文件（例如`.txt`）。它们可以包含用于生成这些源文件的自定义命令，但不能包含 `PRE_BUILD`、`PRE_LINK` 或 `POST_BUILD` 命令。某些原生构建工具（如 [Xcode](https://cmake.org/cmake/help/latest/generator/Xcode.html#generator:Xcode "Xcode")）可能不喜欢仅包含目标文件的构建目标，因此建议在任何引用 [$<TARGET_OBJECTS:objlib>](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:TARGET_OBJECTS "TARGET_OBJECTS") 的目标中至少添加一个真实的源文件。

*自 3.12 版本起*：目标库可以通过 [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries "target_link_libraries") 函数进行链接。

### 接口库
```cmake
add_library(<name> INTERFACE)
```
创建一个接口库（interface‑library）目标：该目标可为依赖它的使用者指定使用要求，但不会编译任何源码，也不会在磁盘上生成库产物。

无源码的接口库，不会作为构建目标出现在生成的构建系统中。但依然允许为它设置属性，也支持安装与导出。一般通过下面这些命令为接口目标填充 `INTERFACE_*` 系列属性：
- [set_property()](https://cmake.org/cmake/help/latest/command/set_property.html#command:set_property)
- [target_link_libraries(INTERFACE)](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries)
- [target_link_options(INTERFACE)](https://cmake.org/cmake/help/latest/command/target_link_options.html#command:target_link_options)
- [target_include_directories(INTERFACE)](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories)
- [target_compile_options(INTERFACE)](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options)
- [target_compile_definitions(INTERFACE)](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions)
- [target_sources(INTERFACE)](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources)。

之后就可以像普通目标一样，把该接口库作为参数传入 [target_link_libraries()](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) 使用。

*版本 3.15 新增*：接口库支持 [PUBLIC_HEADER](https://cmake.org/cmake/help/latest/prop_tgt/PUBLIC_HEADER.html#prop_tgt:PUBLIC_HEADER)、[PRIVATE_HEADER](https://cmake.org/cmake/help/latest/prop_tgt/PRIVATE_HEADER.html#prop_tgt:PRIVATE_HEADER) 目标属性。通过这两个属性指定的头文件，可以使用 [install(TARGETS)](https://cmake.org/cmake/help/latest/command/install.html#targets) 命令完成安装。

```cmake
add_library( INTERFACE [EXCLUDE_FROM_ALL] ...)
```
*版本 3.19 新增*

创建携带源文件的[接口库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#interface-libraries)目标（除[上述](https://cmake.org/cmake/help/latest/command/add_library.html#interface)文档中的普通接口库具备的使用要求、属性能力之外）。源文件允许直接写在 `add_library` 参数里，也可以后续通过 [target_sources()](https://cmake.org/cmake/help/latest/command/target_sources.html#command:target_sources)，配合 `PRIVATE` / `PUBLIC` 关键字追加。

如果接口库配置了源文件（即设置了目标属性 [SOURCES](https://cmake.org/cmake/help/latest/prop_tgt/SOURCES.html#prop_tgt:SOURCES)），或者配置了头文件集合（设置了 [HEADER_SETS](https://cmake.org/cmake/help/latest/prop_tgt/HEADER_SETS.html#prop_tgt:HEADER_SETS) 属性），它会作为构建目标出现在生成的构建系统，行为类似 [add_custom_target()](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target) 创建的自定义目标。它本身不会编译源码，但可以容纳由 [add_custom_command()](https://cmake.org/cmake/help/latest/command/add_custom_command.html#command:add_custom_command) 创建的自定义构建规则。

可选参数：
- `EXCLUDE_FROM_ALL`
自动为目标设置 [EXCLUDE_FROM_ALL](https://cmake.org/cmake/help/latest/prop_tgt/EXCLUDE_FROM_ALL.html#prop_tgt:EXCLUDE_FROM_ALL)属性，参考该属性文档了解详情。

> 
> 注意：
> 多数命令里出现 `INTERFACE` 关键字时，关键字后面的内容仅属于对外暴露的使用要求，不属于目标自身配置。但本版 `add_library` 语法中，`INTERFACE` 仅代表库类型；写在其后的源文件属于该接口库的`PRIVATE`内容，不会进入 [INTERFACE_SOURCES](https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_SOURCES.html#prop_tgt:INTERFACE_SOURCES) 属性。

```cmake
add_library( INTERFACE SYMBOLIC)
```
*版本 4.2 新增*

创建符号型[接口库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#interface-libraries)目标。符号接口库用于表示软件包内可选组件、可选功能。它没有使用要求、不编译源码、不输出磁盘库文件，但支持导出和安装；也允许使用常规的 [if(TARGET)](https://cmake.org/cmake/help/latest/command/if.html#target) 语法判断该目标是否存在。

符号接口库允许作为可链接目标，用来强制校验依赖项必须具备某个可选组件。
举例：库 `libgui` 可能提供、也可能不提供 `widget` 组件；上层使用者链接 `widget` 目标，以此声明自己依赖该组件。这样 [find_package()](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) 在声明为必须组件时，可以通过链接对应符号目标完成校验。

符号接口库的 [SYMBOLIC](https://cmake.org/cmake/help/latest/prop_tgt/SYMBOLIC.html#prop_tgt:SYMBOLIC) 目标属性会被置为 true。

### 导入库
```cmake
add_library(<name> <type> IMPORTED [GLOBAL])
```
创建一个名为`<name>`的[导入库目标（imported‑targets）](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)。该目标允许像项目内部构建出来的普通目标一样被引用；但默认情况下，该目标仅在其被创建的目录及其下级目录内可见。

`<type>`必须取以下值之一：
- `STATIC`、`SHARED`、`MODULE`、`UNKNOWN`
用于引用项目外部的库文件。目标属性 [IMPORTED_LOCATION](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_LOCATION.html#prop_tgt:IMPORTED_LOCATION)（或者每个配置变体 [IMPORTED_LOCATION_\<CONFIG>](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_LOCATION_CONFIG.html#prop_tgt:IMPORTED_LOCATION_%3CCONFIG%3E)）指定磁盘上主库文件的路径：
	- 在绝大多数非 Windows 平台下的共享库（`SHARED`）：主库文件是链接器与动态加载器共同使用的 `.so` 或 `.dylib` 文件。如果被引用的库文件设置了 `SONAME`（macOS 上为以 `@rpath/` 开头的 `LC_ID_DYLIB`），应当将该字段的值设置到目标属性 [IMPORTED_SONAME](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_SONAME.html#prop_tgt:IMPORTED_SONAME)。若被引用库文件没有 `SONAME`，但当前平台支持 SONAME 机制，则应当设置目标属性 [IMPORTED_NO_SONAME](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_NO_SONAME.html#prop_tgt:IMPORTED_NO_SONAME)。
	- Windows 平台下的共享库（`SHARED`）：目标属性 [IMPORTED_IMPLIB](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_IMPLIB.html#prop_tgt:IMPORTED_IMPLIB)（或者每个配置变体 [IMPORTED_IMPLIB_\<CONFIG>](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_IMPLIB_CONFIG.html#prop_tgt:IMPORTED_IMPLIB_%3CCONFIG%3E)）指定磁盘上 DLL 导入库文件（`.lib` 或 `.dll.a`）的路径；`IMPORTED_LOCATION` 填写运行时 `.dll` 文件路径（该属性为可选，但生成器表达式 [TARGET_RUNTIME_DLLS](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:TARGET_RUNTIME_DLLS) 需要依赖它）。

	额外的使用要求可以通过 `INTERFACE_*` 系列属性进行声明。

	`UNKNOWN` 库类型一般仅用于 [Find‑Module](https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html#find-modules)（查找模块）的内部实现。它允许直接使用导入库路径（通常由 [find_library](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) 命令找到），而无需预先知晓该库的具体类型。该特性在 Windows 上尤其有用：Windows 下静态库与 DLL 的导入库文件扩展名完全相同。

- `OBJECT`
引用项目外部的一组目标文件。目标属性 [IMPORTED_OBJECTS](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_OBJECTS.html#prop_tgt:IMPORTED_OBJECTS)（或者其每个配置变体 [IMPORTED_OBJECTS_\<CONFIG>](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED_OBJECTS_CONFIG.html#prop_tgt:IMPORTED_OBJECTS_%3CCONFIG%3E)）指定磁盘上若干目标文件的路径。额外的使用依赖要求可以通过 `INTERFACE_*` 系列属性声明。

- `INTERFACE`
不引用磁盘上任何库文件或目标文件，仅可通过 `INTERFACE_*` 系列属性声明使用要求。

可选参数：
- `GLOBAL`
令该目标名称全局可见。

导入目标不会生成任何构建规则，目标属性 [IMPORTED](https://cmake.org/cmake/help/latest/prop_tgt/IMPORTED.html#prop_tgt:IMPORTED) 会被置为`true`。导入库方便在诸如 `target_link_libraries` 等命令中直接引用。

导入库的详细信息依靠设置以 `IMPORTED_`、`INTERFACE_` 开头的属性来配置，更多细节查阅对应属性文档。
### 别名库
```cmake
add_library(<name> ALIAS <target>)
```
创建一个[别名目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#alias-targets)，使得后续命令中可以使用 `<name>` 来指代 `<target>`。`<name>` 不会作为 make 目标出现在生成的构建系统中。`<target>` 本身不能是一个别名目标。

*3.11 版本新增*：别名可以指向带有 `GLOBAL` 属性的[导入目标](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#imported-targets)。

*3.18 版本新增*：别名可以指向非全局的导入目标。这类别名的作用域仅限于创建该别名的目录及其子目录。可以通过目标属性 [ALIAS_GLOBAL](https://cmake.org/cmake/help/latest/prop_tgt/ALIAS_GLOBAL.html#prop_tgt:ALIAS_GLOBAL) 判断该别名是否为全局可见。

别名目标可以用作可链接目标，也可以用来读取目标属性。还可以使用常规的 [if(TARGET)](https://cmake.org/cmake/help/latest/command/if.html#target) 子命令检测其是否存在。不能使用别名 `<name>` 去修改原目标 `<target>` 的属性：即不可以将别名作为 [set_property()](https://cmake.org/cmake/help/latest/command/set_property.html#command:set_property)、[set_target_properties](https://cmake.org/cmake/help/latest/command/set_target_properties.html#command:set_target_properties)、[target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) 等命令的操作对象。
别名目标**不能被安装（install），也不能被导出（export）**。

# 另请参阅
- [add_executable()](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable)
  
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------