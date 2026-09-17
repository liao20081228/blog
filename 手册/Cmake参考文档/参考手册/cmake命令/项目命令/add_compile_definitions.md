---
title: add_compile_definitions
tags: cmake
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
[TOC]
# 项目命令
这些命令仅在CMake项目中可用。
## add_compile_definitions
*自版本 3.12 起添加*。

将预处理器定义添加到源文件的编译过程中。
```cmake
add_compile_definitions（<definition>...）
```
将预处理器定义添加到编译器命令行中。

这些预处理器定义会被添加到当前 `CMakeLists` 文件的 [COMPILE_DEFINITIONS](https://cmake.org/cmake/help/latest/prop_dir/COMPILE_DEFINITIONS.html#prop_dir:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS") *目录属性*中。它们也会被添加到当前 `CMakeLists` 文件中每个目标的 [COMPILE_DEFINITIONS](https://cmake.org/cmake/help/latest/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS "COMPILE_DEFINITIONS") *目标属性*中。

`<definition>`的语法为 `VAR` 或 `VAR=value`。不支持函数式定义。CMake 会自动为原生构建系统正确转义值（请注意，CMake 语言语法可能需要转义某些值）。

*自版本 3.26 起添加*：任何项前导的 `-D` 都会被移除。

`add_compile_definitions` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用表达式的详细信息，请参阅 [cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)") 手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "cmake-buildsystem(7)") 手册。

### 另请参阅
命令 [target_compile_definitions()](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions "target_compile_definitions") 用于添加针对特定目标的宏定义。

## add_compile_options
向源文件的编译过程中添加选项。
```cmake
add_compile_options(<option>...)
```
将选项添加到 [COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_dir/COMPILE_OPTIONS.html#prop_dir:COMPILE_OPTIONS "COMPILE_OPTIONS") *目录属性*中。这些选项在从当前目录及其子目录编译目标时使用。
>注意：这些选项在链接时不会被使用。有关链接选项，请参见 [add_link_options()](https://cmake.org/cmake/help/latest/command/add_link_options.html#command:add_link_options "add_compile_options") 命令。

### 参数

`add_compile_options` 的参数可以使用生成器表达式，其语法为 `$<...>`。有关可用的表达式，请参见 [cmake-generator-expressions(7) ](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions")手册。有关定义构建系统属性的更多信息，请参阅 [cmake-buildsystem(7)](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#manual:cmake-buildsystem(7) "manual:cmake-buildsystem") 手册。

### 选项去重
最终用于目标的选项集是通过从当前目标及其依赖项的使用要求中累积选项而构建的。选项集会进行去重处理，以避免重复。

*从 3.12 版本开始*：虽然对单个选项来说，去重是有益的，但去重步骤可能会破坏选项组。例如，`-option A -option B `会变成 `-option A B`。可以使用类似 shell 的引号且带有`SHELL:` 前缀来指定一组选项。`SHELL:` 前缀会被移除，其余选项字符串将使用 [separate_arguments()](https://cmake.org/cmake/help/latest/command/separate_arguments.html#command:separate_arguments "separate_arguments") `UNIX_COMMAND` 模式进行解析。例如，`"SHELL:-option A" "SHELL:-option B"` 会变成 `-option A -option B`。


### 示例
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

### 另请参阅
- 该命令可用于添加任何选项。然而，对于添加预处理器定义和包含目录，建议使用更具体的命令 [add_compile_definitions()](https://cmake.org/cmake/help/latest/command/add_compile_definitions.html#command:add_compile_definitions "add_compile_definitions") 和 [include_directories()](https://cmake.org/cmake/help/latest/command/include_directories.html#command:include_directories "include_directories")。
- 命令 [target_compile_options()](https://cmake.org/cmake/help/latest/command/target_compile_options.html#command:target_compile_options "target_compile_options") 用于添加针对特定目标的选项。
- 该命令为所有语言添加编译选项。使用 [COMPILE_LANGUAGE](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:COMPILE_LANGUAGE "COMPILE_LANGUAGE") 生成器表达式来指定针对特定语言的编译选项。
- 源文件属性 [COMPILE_OPTIONS](https://cmake.org/cmake/help/latest/prop_sf/COMPILE_OPTIONS.html#prop_sf:COMPILE_OPTIONS "COMPILE_OPTIONS") 用于向单个源文件添加选项。
- [add_link_options()](https://cmake.org/cmake/help/latest/command/add_link_options.html#command:add_link_options "add_link_options") 用于添加链接选项。
- [CMAKE_\<LANG>_FLAGS](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS.html#variable:CMAKE_%3CLANG%3E_FLAGS "CMAKE_<LANG>_FLAGS") 和 [CMAKE_\<LANG>_FLAGS_\<CONFIG>](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_FLAGS_CONFIG.html#variable:CMAKE_%3CLANG%3E_FLAGS_%3CCONFIG%3E "CMAKE_<LANG>_FLAGS_<CONFIG>") 用于添加传递给语言范围内调用的所有编译器的标志。这包括驱动编译和链接的调用。

## add_custom_target
添加一个没有输出的目标，这样它将始终被构建。
```cmake
add_custom_target(Name [ALL] [command1 [args1...]]
                  [COMMAND command2 [args2...] ...]
                  [DEPENDS depend depend depend ...]
                  [BYPRODUCTS [files...]]
                  [WORKING_DIRECTORY dir]
                  [COMMENT comment]
                  [JOB_POOL job_pool]
                  [JOB_SERVER_AWARE <bool>]
                  [VERBATIM] [USES_TERMINAL]
                  [COMMAND_EXPAND_LISTS]
                  [SOURCES src1 [src2...]])
```
添加一个具有指定名称的目标，该目标执行指定的命令。该目标没有输出文件，即使这些命令尝试创建与目标同名的文件，该目标也*始终被视为过时*。使用 [add_custom_command()](https://cmake.org/cmake/help/v4.2/command/add_custom_command.html#command:add_custom_command "add_custom_command") 命来生成一个带有依赖项的文件。默认情况下，没有任何目标依赖于这个自定义目标。若要添加对其他目标的依赖或从其他目标添加依赖，请使用 [add_dependencies()](https://cmake.org/cmake/help/v4.2/command/add_dependencies.html#command:add_dependencies "add_dependencies") 命令。

可用选项包括：

- `ALL`
指示此目标应添加到默认构建目标中，以便每次构建时都会运行它（不能将命令命名为`ALL`）。
- `BYPRODUCTS`
*添加于 3.2 版本*。

	指定命令预期生成的文件，但这些文件的修改时间在后续构建中可能会或可能不会被更新。如果副产品的名称是相对路径，则该路径将按相对于当前源代码目录对应的构建树目录进行解释。每个副产品文件将自动标记为具有[GENERATED](https://cmake.org/cmake/help/v4.2/prop_sf/GENERATED.html#prop_sf:GENERATED "GENERATED")源文件属性。

	有关此功能背后的动机，请参阅策略 [CMP0058](https://cmake.org/cmake/help/v4.2/policy/CMP0058.html#policy:CMP0058 "CMP0058")。
	
	显式指定副产品被[Ninja](https://cmake.org/cmake/help/v4.2/generator/Ninja.html#generator:Ninja "ninja") 生成器支持，以便告诉 `ninja` 构建工具在副产品缺失时如何重新生成它们。当其他构建规则（例如自定义命令）依赖于这些副产品时，此功能也很有用。`Ninja` 要求为任何生成的有其他规则依赖的文件（即使存在仅限顺序的依赖关系）提供一个构建规则，以确保副产品在其依赖项构建之前可用。

	在执行 `make clean` 时[Makefile 生成器](https://cmake.org/cmake/help/v4.2/manual/cmake-generators.7.html#makefile-generators)会移除 `BYPRODUCTS`和其他 [GENERATED](https://cmake.org/cmake/help/v4.2/prop_sf/GENERATED.html#prop_sf:GENERATED "GENERATED") 文件。

	添加于 3.20 版本*：`BYPRODUCTS` 的参数可以使用一组受限的[生成器表达式](https://cmake.org/cmake/help/v4.2/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")。不允许使用[依赖于目标的表达式](https://cmake.org/cmake/help/v4.2/manual/cmake-generator-expressions.7.html#target-dependent-expressions)。

	*从 3.28 版本开始更改*：在使用[文件集](https://cmake.org/cmake/help/v4.2/command/target_sources.html#file-sets)的自定义目<a class="xsj_anchor xsj_anchor_range xsj_anchor_range_end" name="xsj_1770885411257"></a>标中，副产品现在被视为私有，除非它们被列在非私有的文件集中。请参阅策略 [CMP0154](https://cmake.org/cmake/help/v4.2/policy/CMP0154.html#policy:CMP0154 "CMP0154")。

- `COMMAND`
指定在构建时执行的命令行。如果指定了多个 `COMMAND`，它们将按顺序执行，但*不一定*被组合成一个有状态的 shell 或批处理脚本。（若要运行完整的脚本，可以使用 [configure_file()](https://cmake.org/cmake/help/v4.2/command/configure_file.html#command:configure_file "configure_file")命令或 [file(GENERATE)](https://cmake.org/cmake/help/v4.2/command/file.html#generate "file(GENERATE)") 命令来创建该脚本，然后指定一个 `COMMAND` 来启动它。）

	如果 `COMMAND` 指定的是一个可执行目标名称（由 [add_executable()](https://cmake.org/cmake/help/v4.2/command/add_executable.html#command:add_executable "add_executable") 命令创建），并且满足以下任一条件，则该名称将自动被构建时生成的可执行文件的位置所替换：

	- 该目标不是交叉编译的（即 [CMAKE_CROSSCOMPILING](https://cmake.org/cmake/help/v4.2/variable/CMAKE_CROSSCOMPILING.html#variable:CMAKE_CROSSCOMPILING "CMAKE_CROSSCOMPILING") 变量未设置为 true）。
	- 在版本3.6添加：该目标正在交叉编译，并且提供了模拟器（即其 [CROSSCOMPILING_EMULATOR](https://cmake.org/cmake/help/v4.2/prop_tgt/CROSSCOMPILING_EMULATOR.html#prop_tgt:CROSSCOMPILING_EMULATOR "CROSSCOMPILING_EMULATOR") 目标属性已设置）。在这种情况下，[CROSSCOMPILING_EMULATOR](https://cmake.org/cmake/help/v4.2/prop_tgt/CROSSCOMPILING_EMULATOR.html#prop_tgt:CROSSCOMPILING_EMULATOR "CROSSCOMPILING_EMULATOR") 的内容将被添加到命令的前面，然后再加上目标可执行文件的位置。

	如果上述条件均不满足，则假定该命令名称是一个可在构建时通过 `PATH` 找到的程序。

	`COMMAND` 的参数可以使用[生成器表达式](https://cmake.org/cmake/help/v4.2/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")。可以使用 [TARGET_FILE](https://cmake.org/cmake/help/v4.2/manual/cmake-generator-expressions.7.html#genex:TARGET_FILE "TARGET_FILE") 生成器表达式来引用目标文件在命令行中的位置（即作为命令参数，而不是作为要执行的命令）。

	当以下基于目标的生成器表达式被用作要执行的命令或被作为命令参数提及时，将自动添加一个目标级别的依赖项，以确保所提及的目标在此自定义目标之前被构建（参见策略 [CMP0112](https://cmake.org/cmake/help/v4.2/policy/CMP0112.html#policy:CMP0112 "CMP0112")）：
	- `TARGET_FILE`
	- `TARGET_LINKER_FILE`
	- `TARGET_SONAME_FILE`
	- `TARGET_PDB_FILE`

	命令和参数是可选的，如果未指定这些内容，则将创建一个空的目标。
- `COMMENT`
在构建时执行命令之前显示给定的消息。

	*3.26 版本新增*：`COMMENT`的参数可以使用[生成器表达式](https://cmake.org/cmake/help/v4.2/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")。

- `DEPENDS`
引用文件和使用[add_custom_command()]命令的自定义命令创建的输出调用在同一目录（即 `CMakeLists.txt `文件）。当构建目标时，这些文件将被更新。

	*从 3.16 版本开始更改*：如果任何依赖项是同一目录中某个目标或其构建事件的副产品，则在此目标构建之前会添加一个目标级别的依赖项，以确保副产品在此目标构建之前可用。
	
	使用 [add_dependencies()](https://cmake.org/cmake/help/v4.2/command/add_dependencies.html#command:add_dependencies "add_dependencies") 命令来添加对其他目标的依赖。

- `COMMAND_EXPAND_LISTS`
*从 3.8 版本开始添加。*

	`COMMAND` 参数的列表将被展开，包括那些通过[生成器表达式](https://cmake.org/cmake/help/v4.2/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")创建的列表，这使得诸如 `${CC} "-I$<JOIN:$<TARGET_PROPERTY:foo,INCLUDE_DIRECTORIES>,;-I>" foo.cc` 这样的 `COMMAND` 参数能够被正确展开。
	
- `JOB_POOL`
*从 3.15 版本开始添加。*

	为 [Ninja](https://cmake.org/cmake/help/v4.2/generator/Ninja.html#generator:Ninja "Ninja") 生成器指定一个[池子](https://cmake.org/cmake/help/v4.2/prop_gbl/JOB_POOLS.html#prop_gbl:JOB_POOLS "JOB_POOLS")。该选项与 `USES_TERMINAL` 不兼容，因为 `USES_TERMINAL` 会隐含使用`console`池。使用未通过 [JOB_POOLS](https://cmake.org/cmake/help/v4.2/prop_gbl/JOB_POOLS.html#prop_gbl:JOB_POOLS "JOB_POOLS") 定义的池子会导致 ninja 在构建时出错。

- `JOB_SERVER_AWARE`
*从 3.28 版本开始添加*。

	指定该命令是 GNU Make 作业服务器感知的。

	对于 [Unix Makefiles](https://cmake.org/cmake/help/v4.2/generator/Unix%20Makefiles.html#generator:Unix%20Makefiles "Unix Makefiles")、[MSYS Makefiles](https://cmake.org/cmake/help/v4.2/generator/MSYS%20Makefiles.html#generator:MSYS%20Makefiles "MSYS Makefiles") 和 [MinGW Makefiles](https://cmake.org/cmake/help/v4.2/generator/MinGW%20Makefiles.html#generator:MinGW%20Makefiles "MinGW Makefiles") 生成器，这会在配方行前添加 `+` 前缀。有关更多信息，请参阅 [GNU Make 文档](https://www.gnu.org/software/make/manual/html_node/MAKE-Variable.html)。

	其他生成器会静默忽略此选项。

- `SOURCES`
指定要包含在自定义目标中的额外源文件。指定的源文件将被添加到 IDE 项目文件中，以便于编辑，即使这些文件没有构建规则。
- `VERBATIM`
命令的所有参数将被正确转义，以便构建工具能够接收到每个参数而不会改变其内容。请注意，在 `add_custom_target` 命令看到这些参数之前，CMake 语言处理器仍会对其进行一层转义。因此，建议使用 `VERBATIM` 选项，以确保正确的行为。当未指定 `VERBATIM` 时，行为是平台特定的，因为没有对工具特定的特殊字符提供保护。
- `USES_TERMINAL`
*从 3.2 版本开始添加*。

	如果可能，该命令将直接获得对终端的访问权限。在使用 [Ninja](https://cmake.org/cmake/help/v4.2/generator/Ninja.html#generator:Ninja "Ninja") 生成器时，这会将命令放入`console`池。

- `WORKING_DIRECTORY`
以指定的当前工作目录执行该命令。如果该路径是相对路径，则相对于当前源代码目录对应的构建树目录进行解释。如果未指定，则默认为 [CMAKE_CURRENT_BINARY_DIR]( CMAKE_CURRENT_BINARY_DIR "CMAKE_CURRENT_BINARY_DIR")。

	*从 3.13 版本开始添加*：`WORKING_DIRECTORY` 的参数可以使用[生成器表达式](https://cmake.org/cmake/help/v4.2/manual/cmake-generator-expressions.7.html#manual:cmake-generator-expressions(7) "cmake-generator-expressions(7)")。
	
### Ninja Multi-Config
*3.20 版本新增*：`add_custom_target` 现在支持 [Ninja Multi-Config](https://cmake.org/cmake/help/v4.2/generator/Ninja%20Multi-Config.html#generator:Ninja%20Multi-Config "Ninja Multi-Config")生成器的跨配置功能。更多信息请参阅生成器文档。
### 另请参阅

- [add_custom_command()](https://cmake.org/cmake/help/v4.2/command/add_custom_command.html#command:add_custom_command "add_custom_command")

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
