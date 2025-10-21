---
title: cmake
tags: cmake参考手册
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

[TOC]
# 概要
[<span id="Synopsis">生</span>成项目构建系统](#Generate_a_Project_Buildsystem)
```bash
 cmake [<options>] -B <path-to-build> [-S <path-to-source>]
 cmake [<options>] <path-to-source | path-to-existing-build>
```
[构建项目](#Build_a_Project)
```bash
cmake --build <dir> [<options>] [-- <build-tool-options>]
```
[安装项目](#Install_a_Project)
```bash
cmake --install <dir> [<options>]
```

[打开项目](#Open_a_Project)
```bash
cmake --open <dir>
```
[运行脚本](#Run_a_Script)
```bash
cmake [-D <var>=<value>]... -P <cmake-script-file>
```
[运行命令行工具](#Run_a_Command-Line_Tool)
```bash
 cmake -E <command> [<options>]
```
[运行包查找工具](#Run_the_Find-Package_Tool)
```bash
cmake --find-package [<options>]
```
[运行工作流预设](#Run_a_Workflow_Preset)
```bash
cmake --workflow <options>
```
[查看帮助信息](#View_Help)
```bash
cmake --help[-<topic>]
```
# 描述
**cmake** 可执行文件是跨平台构建系统生成器 CMake 的命令行界面。上述[概要](#Synopsis)列出了该工具可以执行的各种操作，具体描述见下文各节。

要使用 CMake 构建软件项目，首先需要[生成项目构建系统](#Generate_a_Project_Buildsystem)。可以选择使用 **cmake** 来[构建项目](#Build_a_Project)、[安装项目](#Install_a_Project)，或者直接运行相应的构建工具（例如 `make`）。**cmake** 还可以用于[查看帮助信息](#View_Help)。

其他操作主要用于软件开发人员以[Cmake语言](#cmake-language)编写脚本以支持其构建过程。

如需使用图形用户界面替代 **cmake**，请参阅 [ccmake](#ccmake) 和 [cmake-gui](#cmake-gui)。如需使用命令行界面访问 CMake 的测试和打包功能，请参阅 [ctest](#ctest) 和 [cpack](#cpack)。

有关 CMake 的更多信息，[另请参阅](#see_also)本手册末尾的链接。

# CMake构建系统介绍
构建系统描述了如何使用构建工具从源代码自动生成项目的可执行文件和库。例如，构建系统可以是用于命令行`make`工具的`Makefile`，或者是一个集成开发环境（IDE）的项目文件。为了避免维护多个这样的构建系统，项目可以使用[Cmake语言](#cmake_language)编写的文件来抽象地指定其构建系统。根据这些文件，CMake通过一个称为*生成器*的后端工具，为每个用户本地生成一个首选的构建系统。

要使用CMake生成构建系统，必须选择以下内容：

源码树
: 包含项目提供的源文件的顶级目录。项目通过[cmake-language(7)](#cmake_language)手册中的文件来指定其构建系统，这些文件以名为`CMakeLists.txt`的顶级文件开始。这些文件指定了[cmake-buildsystem(7)](#cmake-buildsystem)描述的构建目标及其依赖关系。

构建树
: 用于存储构建系统文件和构建输出产物（例如可执行文件和库）的顶级目录。CMake会写入一个`CMakeCache.txt`文件，以标识该目录为构建树，并存储持久信息，如构建系统配置选项。

: 为了保持源代码树的原始状态，应使用单独的专用构建树进行源代码目录外构建。源代码目录内构建（即构建树与源代码树位于同一目录）也是支持的，但不推荐使用。


生成器
: 这决定了要生成的构建系统的类型。有关所有生成器的文档，请参见[cmake-generators(7)](#cmake-generators)手册。运行`cmake --help`可查看本地可用的生成器列表。可以选择使用下面的`-G`选项来指定生成器，或者简单地接受CMake为当前平台选择的默认生成器。

: 当使用命令行构建工具生成器时，CMake期望编译器工具链所需的环境已经在shell中配置好。当使用IDE构建工具生成器时，不需要特定的环境配置。

# 生成构建系统

<span id="Generate_a_Project_Buildsystem">使</span>用以下命令用法之一运行 CMake，以指定源代码目录和构建目录并生成构建系统：

* `cmake [<options>] -B <path-to-build> [-S <path-to-source>]`
V3.13添加。
使用 `<path-to-build>` 作为构建树，`<path-to-source>` 作为源码树。指定的路径可以是绝对路径，也可以是相对于当前工作目录的相对路径。源码目录必须包含一个 `CMakeLists.txt` 文件。如果构建目录尚不存在，系统会自动创建它。例如：
	```
	$ cmake -S src -B build
	```

* `cmake [<options>] <path-to-source>`
使用当前工作目录作为构建树，并将 `<path-to-source>` 作为源码树。指定的路径可以是绝对路径，也可以是相对于当前工作目录的相对路径。源代码树中必须包含一个 `CMakeLists.txt` 文件，但不能包含 `CMakeCache.txt` 文件，因为后者标识了已存在的构建树。例如：
	```
	$ mkdir build ; cd build
	$ cmake ../src
	```

* `cmake [<options>] <path-to-existing-build>`
使用 `<path-to-existing-build>` 作为构建树，并从其 `CMakeCache.txt` 文件中加载源码树的路径。该文件必须已经由之前的 CMake 运行生成。指定的路径可以是绝对路径，也可以是相对于当前工作目录的相对路径。例如：
	```
	$ cd build
	$ cmake .
	```

在所有情况下，`<options>`可以是以下[选项](#Options)中的一个或多个。 

上述指定源码树和构建树的样式可以混合使用。使用 [-S](#-S) 或 [-B](#-B) 指定的路径始终分别被归类为源码树或构建树。使用普通参数指定的路径则根据其内容以及之前给出的路径类型进行分类。如果只给出了一种类型的路径，则另一种类型将使用当前工作目录（cwd）作为默认值。例如：

| 命令行                 | 源目录   | 构建目录 |
| :--------------------- | :------- | :------- |
| cmake -B build         | *cwd*    | build    |
| cmake -B build src     | src      | build    |
| cmake -B build -S src  | src      | build    |
| cmake src              | src      | *cwd*    |
| cmake build (existing) | *loaded* | build    |
| cmake -S src           | src      | *cwd*    |
| cmake -S src build     | src      | build    |
| cmake -S src -B build  | src      | build    |


从3.23版本开始，CMake会在指定多个源代码路径时发出警告。虽然这一功能从未被正式记录或支持过，但旧版本会意外地接受多个源代码路径，并使用最后指定的路径。因此，应避免传递多个源代码路径参数。

在生成构建系统后，可以使用相应的原生构建工具来构建项目。例如，在使用[Unix Makefiles](#Unix_Makefiles)生成器之后，可以直接运行`make`命令：
```
make
make install
```
或者，也可以使用`cmake`来自动选择并调用适当的原生构建工具来[构建项目](#Build_a_Project)。

## 选项
- <span id="-S">**-S**</span> \<<span id="Options">path</span>-to-source>
要构建的CMake项目的根目录路径。

- <span id="-B">**-B**</span> \<path-to-build>
CMake 将使用此目录路径作为构建目录的根目录。

	如果该目录尚不存在，CMake 将会创建它。
- **-C** \<initial-cache>
预加载一个脚本来填充缓存。

	当首次在一个空的构建目录中运行CMake时，它会创建一个名为`CMakeCache.txt`的文件，并为该项目填充可自定义的设置。此选项可用于指定一个文件，以便在第一次遍历项目的CMake列表文件之前从中加载缓存条目。加载的条目优先于项目的默认值。所给定的文件应是一个包含使用`CACHE`选项的[set()](#set)命令的CMake脚本，而不是缓存格式的文件。

	脚本中对[CMAKE_SOURCE_DIR](#CMAKE_SOURCE_DIR)和[CMAKE_BINARY_DIR](#CMAKE_BINARY_DIR)的引用将解析为源码树和构建树的顶层目录。

- <span id="-D">**-D**</span> \<var>:\<type>=\<value>, -D \<var>=\<value>
创建或更新一个CMake`CACHE`条目。 

	当CMake首次在一个空的构建树中运行时，它会创建一个名为`CMakeCache.txt`的文件，并在其中填充项目的可自定义设置。此选项可用于指定一个优先于项目的默认值的设置。如果需要，可以重复使用此选项来指定任意数量的`CACHE`条目。 

	如果指定了`:<type>`部分，则必须是[set()](#set)命令文档中为其`CACHE`用法指定的类型之一。如果省略了`:<type>`部分，且该条目尚未存在类型，则将创建一个无类型的条目。如果项目中的某个命令将类型设置为`PATH`或`FILEPATH`，则`<value>`将被转换为绝对路径。 

	此选项也可以作为单个参数给出：`-D<var>:<type>=<value>` 或 `-D<var>=<value>`。 

	需要注意的是，`-C`和`-D`参数的顺序是重要的。它们将按照列出的顺序执行，后一个参数将优先于前一个参数。例如，如果你先指定了`-DCMAKE_BUILD_TYPE=Debug`，然后又指定了一个`-C`参数，该参数指向一个包含以下内容的文件： 
	``` 
	set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE) 
	``` 
	那么`-C`参数将优先执行，`CMAKE_BUILD_TYPE`将被设置为`Release`。然而，如果`-D`参数出现在`-C`参数之后，它将优先执行，`CMAKE_BUILD_TYPE`将被设置为`Debug`。 

	如果`-C`文件中`set(... CACHE...)`调用没有使用`FORCE`选项，而`-D`参数设置了相同的变量，那么无论顺序如何，`-D`参数都将优先执行，因为非`FORCE`的`set(... CACHE...)`调用具有这种特性。

- **-U** \<globbing_expr>
从CMake `CACHE`中移除匹配项。 

	此选项可用于从`CMakeCache.txt`文件中移除一个或多个变量，支持使用`*`和``进行通配符匹配。可以根据需要重复使用此选项以移除任意数量的`CACHE`条目。 

	请谨慎使用，否则可能导致`CMakeCache.txt`文件无法正常工作。

- **-G** \<generator-name>
指定一个构建系统生成器。

	在某些平台上，CMake 可能支持多种原生构建系统。生成器负责生成特定的构建系统。可能的生成器名称可以在 [cmake-generators(7)](#cmake-generators) 手册中查看。 
	
	如果未指定生成器，CMake 会检查 [CMAKE_GENERATOR](#CMAKE_GENERATOR) 环境变量，否则将回退到内置的默认选择。

- **-T** \<toolset-spec>
生成器的工具集规范（如果支持）。

	一些CMake生成器支持一个工具集规范来告诉原生构建系统如何选择编译器。有关详细信息，请参阅[CMAKE_GENERATOR_TOOLSET](#CMAKE_GENERATOR_TOOLSET)变量。

- <span id="-A">**-A**</span> \<platform-name>
如果生成器支持，则指定平台名称。

	一些CMake生成器支持为本机构建系统提供平台名称，以选择编译器或SDK。有关详细信息，请参阅[CMAKE_GENERATOR_PLATFORM](#CMAKE_GENERATOR_PLATFORM)变量。

- **--toolchain** \<path-to-file>
3.21版本新增。

	指定交叉编译工具链文件，相当于设置[CMAKE_TOOLCHAIN_FILE](#CMAKE_TOOLCHAIN_FILE)变量。相对路径解释为相对于构建目录，如果找不到，则解释为相对于源目录。

- **--install-prefix** \<directory>
3.21版本新增。

	指定[CMAKE_INSTALL_PREFIX](#CMAKE_INSTALL_PREFIX)变量使用的安装目录。必须是绝对路径。
	
- **--project-file** \<project-file-name>
4.0版本新增。

	指定备用项目文件名。

	这决定了配置项目时CMake处理的顶层文件，以及[add_subdirectory()](#add_subdirectory)处理的文件。

	默认情况下，这是`CMakeLists.txt`。如果设置为其他任何值，则在项目子目录中找不到指定文件时，`CMakeLists.txt`将用作回退。

	>注意：此功能旨在供开发人员在增量过渡期间临时使用，而不是用于发布最终产品。当项目文件不是`CMakeLists.txt`时，CMake将始终发出警告。

- **-Wno-dev**
抑制开发人员警告。

	抑制针对`CMakeLists.txt`文件作者的警告。默认情况下，这也将关闭已弃用警告。

- **-Wdev**
启用开发者警告。

	启用针对`CMakeLists.txt`文件作者的警告。默认情况下，这也将打开已弃用警告。

- **-Wdeprecated**
启用已弃用功能警告。

	启用使用已弃用功能的警告，这些警告是针对`CMakeLists.txt`文件的作者。  

- **-Wno-deprecated**
抑制弃用功能警告。

	抑制使用弃用功能的警告，这些警告是针对`CMakeLists.txt`文件的作者。 

- **-Werror**=\<what>
将CMake警告视为错误。`<what>`必须是以下值之一：
	- `dev`
将开发人员警告视为错误。

		将针对`CMakeLists.txt`文件作者的警告视为错误。默认情况下，这也会将已弃用警告作为错误。
	- `deprecated`
将已弃用宏和函数警告视为错误。

		将针对 `CMakeLists.txt` 文件作者的使用已弃用宏和函数警告视为错误。

- **-Wno-error**=\<what>
不要将CMake警告视为错误。`<what>`必须是以下值之一：
	- `dev`
不将针对 `CMakeLists.txt` 文件作者的警告视为错误。默认情况下，这也会关闭将已弃用警告视为错误。
	- `deprecated`
不将针对 `CMakeLists.txt` 文件作者的使用已弃用宏和函数警告视为错误。

- <span id="--fresh">**--fresh**</span>
3.24版本新增。

	执行构建目录的全新配置。这将删除任何已存在的`CMakeCache.txt`文件和关联的`CMakeFiles/`目录，并从头开始重新创建它们。

	在版本3.30中更改：对于之前由[FetchContent](#FetchContent)填充的具有策略[CMP0168](#CMP0168)的`NEW`设置的依赖项，其来自任何先前运行的戳记和脚本文件将删除。因此，下载、更新和补丁步骤将被强制重新执行。
- **-L\[A]\[H]**
列出非高级缓存变量。 

	列出`CACHE`变量将运行CMake并列出CMake `CACHE`中所有未标记为`INTERNAL`或[ADVANCED](#ADVANCED)的变量。这将有效地显示当前的CMake设置，然后可以使用[-D](#-D)选项更改这些设置。更改某些变量可能会导致创建更多的变量。如果指定了`A`，那么它也将显示高级变量。如果指定了`H`，它还将显示每个变量的帮助。	

- **-LR\[A]\[H]** \<regex>
3.31版本新增。

	显示特定的非高级缓存变量

	从CMake `CACHE`中显示与给定正则表达式匹配的非INTERNAL或[ADVANCED](#ADVANCED)变量。如果指定了`A`，那么它也将显示高级变量。如果指定了`H`，它还将显示每个变量的帮助。
	
- **-N**
仅查看模式。

	只加载缓存。不要实际运行配置和生成步骤。  

- **--graphviz**=\<file>
生成依赖关系的graphviz，更多信息请参见[CMakeGraphVizOptions](#CMakeGraphVizOptions)。

	生成一个graphviz输入文件，该文件将包含项目中的所有库和可执行依赖项。有关更多详细信息，请参阅[CMakeGraphVizOptions](#CMakeGraphVizOptions)的文档。
	
- **--system-information** \[file]
转储有关此系统的信息。

	转储有关当前系统的大量信息。如果从CMake项目的二叉树的顶部运行，它将转储额外的信息，如缓存，日志文件等。

- **--print-config-dir**
3.31版本新增。

	打印用户范围的FileAPI查询的CMake配置目录。

	详细信息，请参见[CMAKE_CONFIG_DIR](#CMAKE_CONFIG_DIR)。 

- **--log-level**=\<level>
3.16版本新增。

	设置日志`<level>`。

	[message()](#message)命令只会输出指定日志级别或更高级别的消息。有效的日志级别为`ERROR`、`WARNING`、`NOTICE`、`STATUS`（默认）、`VERBOSE`、`DEBUG`或`TRACE`。

	要使日志级别在CMake运行之间保持不变，请将[CMAKE_MESSAGE_LOG_LEVEL](#CMAKE_MESSAGE_LOG_LEVEL)设置为缓存变量。如果同时给出命令行选项和变量，则命令行选项优先。

	出于向后兼容的原因，`--loglevel`也被接受为此选项的同义词。

	3.25版中添加：有关[查询当前消息日志级别](#Getting_current_message_log_level)的方法，请参阅[cmake_language()](#cmake_language)命令。

- **--log-context**
启用[message()](#message)命令输出附带到每个消息的上下文。

	此选项打开仅显示当前CMake运行的上下文。要使显示上下文对所有后续CMake运行持久化，请将[CMAKE_MESSAGE_CONTEXT_SHOW](#CMAKE_MESSAGE_CONTEXT_SHOW)设置为缓存变量。当给出此命令行选项时，[CMAKE_MESSAGE_CONTEXT_SHOW](#CMAKE_MESSAGE_CONTEXT_SHOW)将被忽略。

- **--sarif-output**=\<path>
4.0版本新增。

	启用以SARIF格式记录CMake生成的诊断消息。

	将诊断消息写入指定路径下的SARIF文件。项目也可以将[CMAKE_EXPORT_SARIF](#CMAKE_EXPORT_SARIF)设置为`ON`来为构建目录启用此功能。  

- **--debug-trycompile**
不要删除为[try_compile()](#try_compile)/[try_run()](#try_run)调用创建的文件和目录。这在调试失败的检查时很有用。

	请注意，[try_compile()](#try_compile)的某些用法可能会使用相同的构建目录，如果项目执行多个[try_compile()](#try_compile)，这将限制此选项的用处。例如，这样的使用可能会更改结果，因为先前的try-compile中的工件可能会导致不同的测试通过或错误地失败。此选项最好仅在调试时使用。

	（相对于上面，[try_run()](#try_run)命令是一个高效的[try_compile()](#try_compile)。两者的任何组合都会受到描述的潜在问题的影响。)

	3. 25版中添加：当启用此选项时，每个try-compile检查都会打印一条日志消息，报告执行检查的目录。

- **--debug-output**
将cmake置于调试模式。

	在cmake运行期间打印额外的信息，如带有 [message(SEND_ERROR)](#message)调用的堆栈跟踪。

- <span id="--debug-find">**--debug-find**</span>
3.17版本新增。

	cmake find命令置于调试模式。

	cmake运行过程中打印额外的find调用信息到标准错误。输出是为人类而设计的，而不是用于解析。另请参见[CMAKE_FIND_DEBUG_MODE](#CMAKE_FIND_DEBUG_MODE)变量以调试项目的更本地的部分。

- **--debug-find**-pkg=\<pkg>[,...]
3.23版本新增。

	在[find_package(\<pkg>)](#find_package)调用下运行时，将cmake find命令置于调试模式，其中`<pkg>`是给定的区分大小写的包名逗号分隔列表中的条目。

	类似于[--debug-find](#--debug-find)，但将作用域限制到指定的包。

- **--debug-find-var**=\<var>[,...]
3.23版本新增。

	当调用 cmake find 命令时，如果 `<var>` 作为结果变量，并且 `<var>` 是给定逗号分隔列表中的一个条目，则将 cmake find 命令置于调试模式。。

	类似于[--debug-find](#--debug-find)，但将作用域限制为指定的变量名。
	
- <span id="--trace">**--trace**</span>
将cmake置于跟踪模式。

	打印所有调用及其来源的跟踪信息。

- **--trace-expand**
将cmake置于跟踪模式。

	类似于[--trace](#--trace)，但扩展了变量。

- **--trace-format**=\<format>
3.17版本新增。

	将cmake置于跟踪模式并设置跟踪输出格式。

	`<format>`可以是以下值之一。

	- `human`
	以人类可读的格式打印每个跟踪行。这是默认格式。

	- `json-v1`
将每一行打印为单独的JSON文档。每个文档由换行符（`\n`）分隔。保证JSON文档中不会出现换行符。
		```json
		{
		  "file": "/full/path/to/the/CMake/file.txt",
		  "line": 0,
		  "cmd": "add_executable",
		  "args": ["foo", "bar"],
		  "time": 1579512535.9687231,
		  "frame": 2,
		  "global_frame": 4
		}
		```
		成员是：
		- `file`
		调用函数的 CMake 源文件的完整路径。
		- `line`
		`file`中函数调用开始的行。
		- `line_end`
		如果函数调用跨越多行，则该字段将被设置为函数调用结束的行。如果函数调用跨越单行，则此字段将被取消设置。在`json-v1`格式的小版本2中增加了该字段。
		- `defer`
		可选成员，当函数调用被[cmake_language(DEFER)](#cmake_language)时出现。如果出现，则其值是包含延迟调用`<id>`的字符串。
		- `cmd`
		被调用的函数的名称。
		- `args`
		所有函数参数的字符串列表。
		- `time`
		函数调用的时间戳（自epoch以来的秒数）。
		- `frame`
		被调用的函数的栈帧深度，在当前正在处理的`CMakeLists.txt`的上下文中。
		- `global_frame`
		被调用的函数的栈帧深度，在跟踪中涉及的所有`CMakeLists.txt`文件中进行全局跟踪。在`json-v1`格式的小版本2中增加了该字段。

		此外，输出的第一个JSON文档包含当前主版本和次版本的版本关键字
		```json
		JSON version format
		{
		  "version": {
		    "major": 1,
		    "minor": 2
		  }
		}
		```
		成员包括：
		- `verison`
		指示JSON格式的版本。该版本具有遵循语义版本约定的主要组件和次要组件。

- **trace-source**=\<file>
将cmake置于跟踪模式，但只输出指定文件的行。

	允许多个选项。

- **trace-redirect**=\<file>
将 cmake 置于跟踪模式，并将跟踪输出重定向到文件而不是 stderr。

- **--warn-uninitialized**
警告未初始化的值。

	使用未初始化的变量时打印警告。 

- **--warn-unused-vars**
什么都不做。在CMake版本3.2及以下版本中，这启用了关于未使用变量的警告。在CMake版本3.3到3.18中，该选项被打破。在CMake 3.19及更高版本中，该选项已被删除。  

- **--no-warn-unused-cli**
不要警告命令行选项。

	不要查找在命令行中声明但未使用的变量。
	
- **--check-system-vars**
查找系统文件中变量使用的问题。

	正常情况下，只在[CMAKE_SOURCE_DIR](#CMAKE_SOURCE_DIR)和[CMAKE_BINARY_DIR](#CMAKE_BINARY_DIR)中查找未使用和未初始化的变量。这个标志告诉CMake也警告其他文件。 

- **--compile-no-warning-as-error**
3.24版本新增。

	忽略目标属性[COMPILE_WARNING_AS_ERROR](#COMPILE_WARNING_AS_ERROR)和变量[CMAKE_COMPILE_WARNING_AS_ERROR](#CMAKE_COMPILE_WARNING_AS_ERROR)，以防止在编译时将警告视为错误。

- **--link-no-warning-as-error**
4.0版本新增。

	忽略目标属性[LINK_WARNING_AS_ERROR](#LINK_WARNING_AS_ERROR)和变量[CMAKE_LINK_WARNING_AS_ERROR](#CMAKE_LINK_WARNING_AS_ERROR)，以防止将警告视为链接上的错误。 
	
- **--profiling-output**=\<path>
3.18版本新增。

	与[--profiling-format](#--profiling-format)一起使用，输出到给定的路径。  
	
- <span id="--profiling-format">**--profiling-format**</span>=\<file>
启用以给定格式输出 CMake 脚本的性能分析数据。

	这有助于对执行的CMake脚本进行性能分析。应使用第三方应用程序将输出处理为人类可读的格式。

	目前支持的值有：`google-trace` 以Google Trace Format输出，可以通过Google Chrome的`about:trace`选项卡进行解析，也可以使用Trace Compass等工具的插件进行解析。

- **--preset** \<preset>, **--preset**=\<preset>
从`CMakePresets.json`和`CMakeUserPresets.json`文件中读取[预设](#cmake-preset)，这些文件必须与顶级`CMakeLists.txt`文件位于同一目录中。预设可以指定生成器、构建目录、变量列表和其他要传递给CMake的参数。`CMakePresets.json`或`CMakeUserPresets.json`中至少有一个必须存在。[CMake GUI](#cmake-gui)也识别并支持`CMakePresets.json`和`CMakeUserPresets.json`文件。有关这些文件的完整详细信息，请参见[cmake-presets(7)](#cmake-preset)。

	在所有其他命令行选项之前读取预设值，尽管[-S](#-S)选项可用于指定包含`CMakePresets.json`和`CMakeUserPresets.json`文件的源目录。如果没有给出[-S](#-S)，则假定当前目录是顶级源码目录，并且必须包含预设文件。通过在命令行上手动指定，可以覆盖所选预设（变量、生成器等）指定的选项。例如，如果预设将名为`MYVAR`的变量设置为`1`，但用户使用`-D`参数将其设置为`2`，则首选值`2`。

- **--list-presets**\[=\<type>]
列出指定\<type>的可用预设。\<type>的有效值为`configure`、`build`、`test`、`package`或`all`。如果省略`<type>`，则假定为`configure`。当前工作目录必须包含CMake预设文件，除非使用[-S](#-S)选项指定了不同的顶级源目录。

- <span id="--debugger">**--debugger**</span>
打开CMake语言的交互式调试开关。CMake在以[--debugger-pipe](#--debugger-pipe)命名的管道上公开了一个符合[调试适配器协议](#https://msdocs.cn/debug-adapter-protocol/)规范的调试接口，并进行了以下修改。 

	`initialize`响应包括一个名为`cmakeVersion`的附加字段，它指定正在调试的CMake的版本。
	```json
	{
	  "cmakeVersion": {
	    "major": 3,
	    "minor": 27,
	    "patch": 0,
	    "full": "3.27.0"
	  }
	}
	```
	成员包括：

	- major
指定主版本号的整数。

	- minor
指定次要版本号的整数。

	- patch
指定补丁版本号的整数。

	- full
指定完整CMake版本的字符串。

- <span id="--debugger-pipe">**--debugger-pipe**</span> \<pipe name>, **--debugger-pipe**=\<pipe name>
用于调试器通信的管道（在Windows上）或域套接字（在Unix上）的名称。

- **--debugger-dap-log** \<log path>, **--debugger-dap-log**=\<log path>
将所有调试器通信记录到指定的文件中。 
# 构建项目

<span id="Build_a_Project">CMake 提供</span>了一个命令行用法来构建已生成的项目二进制树：
```
cmake --build <dir>             [<options>] [-- <build-tool-options>]
cmake --build --preset <preset> [<options>] [-- <build-tool-options>]
```
该工具抽象了一个原生构建工具的命令行界面，提供了以下选项：

- **--build** \<dir>
<span id="cmake_--build">要构建</span>的项目二进制目录。这是必需的（除非指定了预设），并且必须放在第一个参数位置。

- **--preset** \<preset>, --preset=\<preset>
使用构建预设来指定构建选项。项目二进制目录将从 `configurePreset` 关键字中推断出来。当前工作目录必须包含 CMake 预设文件。有关更多详细信息，请参见[preset](#cmake-preset)部分。

- **--list-presets**
列出可用的构建预设。当前工作目录必须包含 CMake 预设文件。

- **-j** [\<任务数>], **--parallel** [\<任务数>]
自 3.12 版本起添加。

	指定在构建时使用的最大并发进程数。如果省略了 `<jobs>`，则使用原生构建工具的默认值。
	
	如果设置了 [CMAKE_BUILD_PARALLEL_LEVEL](#CMAKE_BUILD_PARALLEL_LEVEL)` 环境变量，当未指定此选项时，该变量将指定默认的并行级别。
	
	某些原生构建工具总是并行构建的。可以使用 `<jobs>` 值为 1 来限制为单任务构建。

- **-t** \<tgt>..., **--target** \<tgt>...
<span id="--target_clean">构建</span>指定的 `<target>` 而不是默认目标。可以指定多个目标，用空格分隔。

- **--config** <cfg>
对于多配置构建工具，选择配置 `<cfg>`。

- **--clean-first**
先构建目标`clean`，再构建。（仅清理时，请使用 [--target clean](#--target_clean)。）

- **--resolve-package-references**=\<value>
自 3.23 版本起添加。

	在构建之前，从外部包管理器（例如 NuGet）解析远程包引用。当 `<value>` 设置为 `on`（默认值）时，将在构建目标之前恢复包。当 `<value>` 设置为 `only` 时，将恢复包但不执行构建。当 `<value>` 设置为 `off` 时，将不恢复任何包。

	如果目标未定义任何包引用，则此选项不会产生任何效果。

	此设置可以在构建预设中指定（使用 `resolvePackageReferences`）。如果命令行中指定了此选项，则预设中的设置将被忽略。
	
	如果未提供命令行参数或预设选项，则将评估环境特定的缓存变量以决定是否执行包恢复。
	
	当使用 [Visual Studio 生成器时](#Visual_Studio_Generators,)，包引用是通过 [VS_PACKAGE_REFERENCES](#VS_PACKAGE_REFERENCES) 属性定义的。包引用通过 NuGet 进行恢复。可以通过将 [CMAKE_VS_NUGET_PACKAGE_RESTORE](#CMAKE_VS_NUGET_PACKAGE_RESTORE) 变量设置为 `OFF` 来禁用此功能。

- **--use-stderr**
忽略。在 CMake >= 3.0 中，此选项的行为是默认的。

- **-v**, **--verbose**
启用详细输出（如果支持），包括要执行的构建命令。

	如果设置了 [VERBOSE](#VERBOSE) 环境变量或 [CMAKE_VERBOSE_MAKEFILE](#CMAKE_VERBOSE_MAKEFILE) 缓存变量，则可以省略此选项。

- **--**
将剩余的选项传递给原生构建工具。

不带任何选项运行 [cmake --build](#cmake_--build)以获取快速帮助。

## 生成器特定的构建工具行为
`cmake --build` 在某些生成器下具有特殊行为：

- [Xcode](#Xcode)
自 4.1 版本起添加：如果第三方工具在 CMake 生成的 `.xcodeproj` 旁边写入了 `.xcworkspace` 文件，`cmake --build` 将通过工作区来驱动构建。

# 安装项目

CMake 提供了一个命令行用法来<span id="Install_a_Project">安装</span>已生成的项目二进制树：
```
cmake --install <dir> [<options>]
```
这可以在构建项目后使用以便在不使用生成的构建系统或原生构建工具的情况下运行安装。选项包括：

- <span id="cmake_--install">**--install**</span> \<dir>
项目二进制目录安装路径。这是必需的，且必须首先指定。  

- **--config** \<cfg>
对于多配置生成器，选择配置`<cfg>`。

- **--component** \<comp>
基于组件的安装。仅安装组件`<comp>`。

- **--default-directory-permissions** \<permissions>
默认目录安装权限。权限格式为`<u=rwx,g=rx,o=rx>`。

- **--prefix** \<prefix>
覆盖安装前缀，即 [MAKE_INSTALL_PREFIX](#MAKE_INSTALL_PREFIX)。

- **--strip**
安装前先剥离符号表。

- **-v**, **--verbose**
启用详细输出。

	如果设置了[VERBOSE](#VERBOSE)环境变量，则可以省略此选项。

- **-j** \<jobs>, **--parallel** \<jobs>
添加于版本3.31。 

	使用指定的作业数并行安装。仅在启用  [INSTALL_PARALLEL](#INSTALL_PARALLEL) 时可用。如果未提供此选项，则 [CMAKE_INSTALL_PARALLEL_LEVEL](#CMAKE_INSTALL_PARALLEL_LEVEL) 环境变量会指定默认的并行级别。 

不带任何选项运行 [cmake --install](#cmake_--install) 可快速获取帮助信息。

# 打开项目
```
cmake --open <dir>
```
<span id="Open_a_Project">在</span>相关应用程序中打开生成的项目。此功能仅被部分生成器支持。

# 运行脚本
```
cmake [-D <var>=<value>]... -P <cmake-script-file> [-- <unparsed-options>...]
```
- **-D** \<var>=\<value>
<span id="Run_a_Script">定义</span>一个用于脚本模式的变量。

- **-P** \<cmake-script-file>
将给定的CMake文件作为用CMake语言编写的脚本来处理。不会执行配置或生成步骤，也不会修改缓存。如果使用了`-D`选项定义变量，则必须在`-P`参数之前进行此操作。

`--`之后的任何选项不会被CMake解析，但它们仍会被包含在传递给脚本的[CMAKE_ARGV\<n>](#CMAKE_ARGV)变量集合中（包括`--`本身）。

# 运行命令行工具
CMake通过用法提供了内置的<span id="Run_a_Command-Line_Tool">命令行工具</span>。
```
cmake -E <command> [<options>]
```
- **-E** [help]
运行`cmake -E或cmake -E help`以获取命令摘要。

可用的命令有: 

- **capabilities**
3.7版本新增。

	以JSON格式上报cmake能力。输出是一个具有以下键值的JSON对象：
	- `version`
一个带有版本信息的JSON对象。键值包括：
		- `string`
		cmake [--version](#--version)显示的完整版本字符串。
		- `major`
		整数形式的主版本号。
		- `minor`
		整数形式的次版本号。
		- `patch`
		整数形式的补丁级别。
		- `suffix`
		cmake版本后缀字符串。
		- `isDirty`
		如果cmake构建来自脏目录，则设置一个`true`的布尔值。
	- `generators`
	可用生成器列表。每个生成器都是一个具有以下键的JSON对象：
		- `name`
		包含生成器名称的字符串。
		- `toolsetSupport`
		如果生成器支持工具集，则为`true`，否则为`false`。
		- `platformSupport`
		如果生成器支持平台，则为`true`，否则为`false`。
		- `supportedPlatforms`
3.21版本新增。

			当生成器通过[CMAKE_GENERATOR_PLATFORM](#CMAKE_GENERATOR_PLATFORM) ([-A ...](#-A))支持平台规范时，可能出现的可选成员。该值是已知支持的平台列表。
		- `extraGenerators`
		包含与生成器兼容的所有[额外生成器](#extraGenerators)的字符串列表。
	- `fileApi`
可选成员，当[cmake-file-api(7)](#cmake-file-api)可用时出现。值是一个包含一个成员的JSON对象：
		- `requests`
包含零个或多个支持的`file-api`请求的JSON数组。每个请求都是一个包含以下成员的JSON对象：
			 - `kind`
指定支持的[对象种类](#object_kinds)之一。

			 - `version`
一个JSON数组，其每个元素都是一个JSON对象，包含指定非负整数版本组件的`major`和`minor`。

	- `serverMode`
	如果cmake支持服务器模式，则为`true`，否则为`false`。从CMake 3.20开始一直为`false`。
	- `tls`
	3.25版本新增。

		如果启用了TLS支持，则为`true`，否则为`false`。
	- `debugger`
3.27版本新增。

		如果支持[--debugger](#--debugger)模式，则为`true`，否则为`false`。
- **cat** \[--] \<files>...
3.18版本新增。

	连接文件并在标准输出上打印。

	- **--**
3.24版本新增。

		添加了对双破折号参数的支持`--`。cat的这个基本实现不支持任何选项，因此使用以`-`开头的选项将导致错误。如果文件以`-`开头，请使用`--`来指示选项的结尾。

	在3.29版中添加：cat现在可以通过传递`-`参数来打印标准输入。

- **chdir** \<dir> \<cmd> \[\<arg>...]
更改当前工作目录并运行命令。

- **compare_files** \[--ignore-eol] \<file1> \<file2>
检查`<file1>`是否与`<file2>`相同。如果文件相同，则返回0，否则返回1。如果参数无效，则返回2。

	- **--ignore-eol**
		3.14版本新增。

		该选项表示逐行比较，忽略LF/CRLF差异。
- **copy** \<file>... \<destination>, copy -t \<destination> \<file>...
将文件复制到`<destination>`（文件或目录）。如果指定了多个文件，或者指定了-t，则`<destination>`必须是目录，并且必须存在。如果未指定`-t`，则假定最后一个参数是`<destination>`。不支持通配符。`copy`跟随符号链接。这意味着它不复制符号链接，而是复制它指向的文件或目录。

	3.5版本中新增：支持多个输入文件。

	3.26版本中添加：支持`-t`参数。  
- **copy_directory** \<dir>... \<destination>
复制`<dir>...`目录的内容到`<destination>`目录。如果`<destination>`目录不存在，它将被创建。`copy_directory`跟随符号链接。

	3.5版本中新增：支持多个输入目录。

	在3.15版中添加：现在，当源目录不存在时，该命令将失败。以前，它是通过创建一个空的目标目录来成功的。
- **copy_directory_if_different** \<dir>... \<destination>
3.26版本新增。

	复制`<dir>...`目录的更改内容到`<destination>`目录。如果`<destination>`目录不存在，它将被创建。

	`copy_directory_if_different`跟随符号链接。当源目录不存在时，命令将失败。

- **copy_directory_if_newer** \<dir>... \<destination>
4.2版本新增。

	如果源文件比目标文件新（基于文件时间戳），复制`<dir>...`目录的内容到`<destination>`目录。如果`<destination>`目录不存在，它将被创建。

	`copy_directory_if_newer`跟随符号链接。当源目录不存在时，命令将失败。这比`copy_directory_if_different`快，因为它只比较文件时间戳，而不是文件内容。

- **copy_if_different** \<file>... \<destination>
如果文件已更改，则将文件复制到`<destination>`（文件或目录）。如果指定了多个文件，则`<destination>`必须是目录，并且必须存在。`copy_if_different`跟随符号链接。

	3.5版本中新增：支持多个输入文件。  

- **copy_if_newer** \<file>... \<destination>
4.2版本新增。

	如果源文件比目标文件新（基于文件时间戳），则将文件复制到`<destination>`（文件或目录）。如果指定了多个文件，则`<destination>`必须是目录，并且必须存在。`copy_if_newer`跟随符号链接。这比`copy_if_different` 快，因为它只比较文件时间戳，而不是文件内容。 
- **create_symlink** \<old> \<new>
创建一个符号链接<new>，命名为<old>。

	3.13版本中新增：支持在Windows上创建符号链接。

	>注意要创建`<new>`符号链接的路径必须事先存在。 
- **create_hardlink** \<old> \<new>
3.19版本新增。

	创建一个指向`<old>`的硬链接`<new>`。

	>注意要创建`<new>`硬链接的路径必须事先存在。`<old>`必须事先存在。
- **echo** \[\<string>...]
将参数显示为文本。
- **echo_append** \[\<string>...]
将参数显示为文本，但不换行。
- **env** \[\<options>] \[--] \<command> \[\<arg>...]
3.1版本新增。

	在修改后的环境中执行命令。选项包括：
	- **NAME**=VALUE
将`NAME`的当前值替换为`value`。
	- **--unset**=NAME
取消设置`NAME`的当前值。
	- **--modify** ENVIRONMENT_MODIFICATION
3.25版本新增。

		对修改后的环境应用单个[ENVIRONMENT_MODIFICATION](#ENVIRONMENT_MODIFICATION)操作。

		`NAME=VALUE`和`--unset=NAME`选项分别等效于`--modify NAME=set:VALUE`和`--modify NAME=unset:`。请注意，`--modify NAME=reset:`将`NAME`重置为cmake启动时（或取消设置）的值，而不是最新的`NAME=VALUE`选项。
	- **--**
3.24版本新增。

		添加了对双破折号参数的支持`--`。使用`--`停止解释选项/环境变量，并将下一个参数视为命令，即使它以`-`开头或包含`=`。
- **environment**
显示当前环境变量。
- **false**
3.16版本新增。

	不执行任何操作，退出代码为1。
- **make_directory** \<dir>...
创建`<dir>`目录。如果需要，也可以创建父目录。如果目录已经存在，它将被静默忽略。

	3.5版本中新增：支持多个输入目录。
- **md5sum** \<file>...
以`md5sum`兼容格式创建文件的MD5校验和：
	```
	351abe79cd3800b38cdfb25d45015a15  file1.txt
	052f86c15bbde68af55c7f7b340ab639  file2.txt
	```
- **sha1sum** \<file>...
3.10版本新增。

	以`sha1sum`兼容格式创建文件的SHA1校验和：
	```
	4bb7932a29e6f73c97bb9272f2bdc393122f86e0  file1.txt
	1df4c8f318665f9a5f2ed38f55adadb7ef9f559c  file2.txt
	```
- **sha224sum**  \<file>...
3.10版本新增。

	以`sha224sum`兼容格式创建文件的 SHA224 校验和：
	```
	b9b9346bc8437bbda630b0b7ddfc5ea9ca157546dbbf4c613192f930  file1.txt
	6dfbe55f4d2edc5fe5c9197bca51ceaaf824e48eba0cc453088aee24  file2.txt
	```
- **sha256sum**  \<file>...
3.10版本新增。

	以`sha256sum`兼容格式创建文件的 SHA256 校验和：
	```
	76713b23615d31680afeb0e9efe94d47d3d4229191198bb46d7485f9cb191acc  file1.txt
	15b682ead6c12dedb1baf91231e1e89cfc7974b3787c1e2e01b986bffadae0ea  file2.txt
	```
- **sha384sum**  \<file>...
3.10版本新增。

	以`sha384sum`兼容格式创建文件的 SHA384 校验和：
	```
	acc049fedc091a22f5f2ce39a43b9057fd93c910e9afd76a6411a28a8f2b8a12c73d7129e292f94fc0329c309df49434  file1.txt
	668ddeb108710d271ee21c0f3acbd6a7517e2b78f9181c6a2ff3b8943af92b0195dcb7cce48aa3e17893173c0a39e23d  file2.txt
	```
- **sha512sum**  \<file>...
3.10版本新增。

	以`sha512sum`兼容格式创建文件的 SHA512 校验和：
	```
	2a78d7a6c5328cfb1467c63beac8ff21794213901eaadafd48e7800289afbc08e5fb3e86aa31116c945ee3d7bf2a6194489ec6101051083d1108defc8e1dba89   file1.txt
	7a0b54896fe5e70cca6dd643ad6f672614b189bf26f8153061c4d219474b05dad08c4e729af9f4b009f1a1a280cb625454bf587c690f4617c27e3aebdf3b7a2d   file2.txt
	```
- **remove** [-f] \<file>...
自版本3.17起已弃用。

	删除文件。计划的行为是，如果列出的任何文件已经不存在，则命令返回非零退出代码，但不记录任何消息。`-f`选项会将行为更改为在这种情况下返回零退出代码（即成功）。`remove`不会跟随符号链接。这意味着它只删除符号链接，而不删除它指向的文件。

	实现有bug，总是返回0。如果不打破向后兼容性，就无法修复它。改用rm。
- **remove_directory** \<dir>...
	自版本3.17起已弃用。

	删除`<dir>`目录及其内容。如果目录不存在，它将被静默忽略。改用rm。

	3.15版本新增：支持多个目录。

	在版本3.16中添加：如果`<dir>`是指向目录的符号链接，则仅删除符号链接。
- **rename** \<oldname> \<newname>
	重命名文件或目录（在一个卷上）。如果具有`<newname>`名称的文件已经存在，则它将被静默替换。  
- **rm** [-rRf] [--] \<file|dir>...
	3.17版本新增。

	删除文件`<file>`或目录`<dir>`。使用`-r`或`-R`递归删除目录及其内容。如果列出的任何文件/目录不存在，命令将返回非零退出代码，但不会记录任何消息。`-f`选项会将行为更改为在这种情况下返回零退出代码（即成功）。使用`--`停止解释选项并将所有剩余参数视为路径，即使它们以`-`开头。
- **sleep** \<number>
3.0版本新增。

	睡眠`<number>`秒。`<number>`可以是浮点数。由于启动/停止CMake可执行文件的开销，实际的最小值大约为0.1秒。这在CMake脚本中可以很有用地插入延迟：
	```
	#睡眠约0.5秒
	execute_process(COMMAND ${CMAKE_COMMAND} -E sleep 0.5)
	```
- **tar** [cxt][vf][zjJ] file.tar [\<options>] [--] [\<pathname>...]
创建或解压缩tar或zip存档。选项包括：
	- **c**
	创建包含指定文件的新存档。如果使用，则`<pathname>`。参数是必需的。
	- **x**
	从归档文件中提取到磁盘。

		在3.15版本中添加：`<pathname>...`参数可用于仅提取选定的文件或目录。当提取选定的文件或目录时，您必须提供它们的确切名称，包括路径，如列表（`-t`）打印的。
	- **t**
列出归档内容。
在3.15版本中添加：`<pathname>...`参数可用于仅列出选定的文件或目录。
	- **v**
生成详细的输出。
	- **z**
使用gzip压缩生成的归档文件。
	- **j**
使用bzip2压缩生成的归档文件。
	- **J**
3.1版本新增。

		使用XZ压缩生成的归档文件。
	- **--zstd**
3.15版本新增。

		使用ZStandard压缩生成的归档文件。
	- **--files-from**=\<file>
3.1版本新增。

	从给定文件中读取文件名，每行一个。空行将被忽略。除了`--add-file=<name>`可以添加名称以`-`开头的文件之外，行不能以`-`开头。
	- **--format**=\<format>
3.3版本新增。

		指定要创建的归档文件的格式。支持的格式有：7zip、gnutar、pax、paxr（默认pax）和zip。
	- **--mtime**=\<date>
3.1版本新增。

		指定tarball条目中记录的修改时间。
	- **--touch**
3.24版本新增。

		使用当前本地时间戳，而不是从归档文件中提取文件时间戳。
	- **--**
3.1版本新增。

		停止解释选项并将所有剩余参数视为文件名，即使它们以`-`开头。

	在版本3.1中添加：LZMA (7zip)支持。

	3.15版中添加：现在，即使某些文件不可读，命令也会继续向归档文件中添加文件。这种行为更符合经典的tar工具。命令现在还解析所有标志，如果提供了无效的标志，则会发出警告。

- **time** \<command> [\<args>...]
运行`<command>`并显示运行时间（包括CMake前端的开销）。

	3.5版中添加：命令现在可以正确地将带有空格或特殊字符的参数传递给子进程。这可能会破坏那些通过自己的额外引用或转义来解决此错误的脚本。

- **touch** \<file>...
如果文件不存在，则创建`<file>`。如果`<file>`存在，则它正在改变`<file>`的访问和修改时间。
- **touch_nocreate** \<file>...
如果文件存在，则点击文件，但不创建该文件。如果文件不存在，它将被静默忽略。
- **true**
3.16版本新增。

	不执行任何操作，退出代码为0。

# 运行包查找工具
CMake为基于Makefile的项目提供了一个类似pkg-config的助手：
```
cmake --find-package [<options>]
```
>注意：由于一些技术限制，此模式不被很好地支持。它是为了兼容而保留的，但不应该在新项目中使用。

- **--find-package**
它使用[find_package()](#find_package)命令搜索包，并将结果标记打印到stdout。这可以替代 pkg-config，用于在基于 Makefile 的纯项目中或在基于 Autoconf 的项目中使用系统上安装在 share/aclocal/cmake.m4 中的辅助宏查找已安装的库。

使用此选项时，需要以下变量：

- `NAME`
`find_package(<PackageName>)`中调用的包的名称。
- `COMPILER_ID`
用于搜索包的[编译器ID](#CMAKE_<LANG>_COMPILER_ID)，即GNU/Intel/Clang/MSVC等。
- `LANGUAGE`
用于搜索包的语言，如C/CXX/Fortran/ASM等。
- `MODE`
包搜索模式。值可以是以下值之一：
	`EXIST`
仅检查给定包是否存在。
	`COMPILE`
打印编译使用给定包的对象文件所需的标志。
	`LINK`
打印使用给定包时链接所需的标志。
- `SILENT`
（可选）如果为TRUE，则不打印查找结果消息。

例如：
```
cmake --find-package -DNAME=CURL -DCOMPILER_ID=GNU -DLANGUAGE=C -DMODE=LINK
```

# 运行工作流预设
3.25版本新增。

[CMake Presets](#cmake-preset)提供了一种按顺序执行多个构建步骤的方法：
```
cmake --workflow <options>
```
选项有：

- **--workflow**
使用以下选项之一选择[工作流预设](#workflow_preset)。

- **--preset** \<preset>, **--preset**=\<preset>
使用工作流预设指定工作流。项目二进制目录是从初始配置预设中推断出来的。当前工作目录必须包含CMake预设文件。有关更多详细信息，请参阅[预设](#cmake_preset)。

	在3.31版中更改：当紧跟在`--workflow`选项之后时，可以省略`--prese`t参数，只提供`<preset>`名称。这意味着以下语法有效：
	```
	$ cmake --workflow my-preset
	```
- **--list-presets**
列出可用的工作流预设。当前工作目录必须包含CMake预设文件。

- **--fresh**
执行构建目录的全新配置，这与[cmake --fresh](#--fresh)具有相同的效果。

# 查看帮助信息
要打印CMake文档中的选定页面，请使用
```
cmake --help[-<topic>]
```
具有以下选项之一：

- **-version** [\<file>], **--version** [\<file>], **/V** [\<file>]
显示程序名称/版本横幅并退出。如果给定`<file>`，输出将打印到`<file>`。

 - **-h**, **-H**, **--help**, **-help**, **-usage**, **/?**
打印用法信息并退出。

	用法介绍了基本命令行界面及其选项。

- **--help** \<keyword> [\<file>]
打印一个CMake关键字的帮助。

	\<keyword>可以是属性、变量、命令、策略、生成器或模块。

	\<keyword>的相关手册项以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

	在版本3.28中更改：在CMake 3.28之前，此选项仅支持命令名称。

- **--help-full** [\<file>]
打印所有帮助手册并退出。

	所有手册都以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-manual** \<man> [\<file>]
打印一个帮助手册并退出。

	指定的手册以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-manual-list** [\<file>]
列出可用的帮助手册并退出。

	该列表包含可通过使用`--help-manual`选项后跟手册名称来获取其帮助的所有手册。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-command** \<cmd> [\<file>]
打印一条命令的帮助并退出。

	`<cmd>`的[cmake-commands(7)](#cmake-commands)手册条目以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-command-list** [<文件>]
列出帮助可用的命令的并退出。

	该列表包含可以通过使用`--help-command`选项后跟命令名称获得帮助的所有命令。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-commands** [<file>]
打印cmake-commands手册并退出。

	[cmake-commands(7)](#cmake-commands)手册条目以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-module** <mod> [<file>]
打印一个模块的帮助并退出。

	`<mod>`的[cmake-modules(7)](#cmake-modules)手册条目以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-module-list** [\<file>]
列出帮助可用的模块并退出。

	该列表包含可以通过使用`--help-module`选项后跟模块名称来获取其帮助的所有模块。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-modules** [\<file>]
打印cmake-modules手册并退出。

	[cmake-modules(7)](#cmake-modules)手册以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-policy** \<cmp> [<file>]
打印一个策略的帮助并退出。

	`<cmp>`的[cmake-policy(7)](cmake-policy)的手册条目以以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-policy-list** [\<file>]
列出帮助可用的策略并退出。

	该列表包含所有可以通过使用`--help-policy`选项后跟策略名称获取帮助的策略。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-policys** [\<file>]
打印cmake-policy手册并退出。

	[cmake-policy(7)](#cmake-policy)手册以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-property** \<prop>[\<file>]
打印一个属性的帮助并退出。

	`<prop>`的[cmake-properties(7)](#cmake-properties)的手册条目以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-property-list** \[\<文件>]
列出帮助可用的属性并退出。

	该列表包含所有可以通过使用`--help-property`选项后跟属性名称来获得帮助的属性。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-properties** \[\<文件>]
打印cmake-properties手册并退出。

	[cmake-properties(7)](#cmake-properties)手册以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-variable** \<var> [<file>]
打印一个变量的帮助并退出。

	`<var>`的[cmake-variables(7)](#cmake-variables)的手册条目以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-variable-list** \[\<文件>]
列出帮助可用的变量并退出。

	该列表包含所有可以通过使用`--help-variable`选项后跟变量名获得帮助的变量。如果给定`<file>`，输出将打印到命名的`<file>`。

- **--help-variables** \[\<file>]
手动打印cmake-variables并退出。

	[cmake-variables(7)](#cmake-variables)手册以人类可读的文本格式打印。如果给定`<file>`，输出将打印到命名的`<file>`。

要查看项目可用的预设，请使用
```
cmake <source-dir> --list-presets
```
# 返回值（退出码）
在正常终止时，cmake可执行文件返回退出代码0。

如果终止是由命令消息[message(FATAL_ERROR)](#message)或其他错误条件引起的，则返回非零退出代码。

# 另请参阅
以下资源可用于获取使用CMake的帮助：

- 首页
<https://cmake.org>

	是学习CMake的主要起点。

- 在线文档和社区资源
<https://cmake.org/documentation>

	可在此网页上找到可用文档和社区资源的链接。

- 互动论坛
<https://disprocess.cmake.org>

	互动论坛举办关于CMake的讨论和提问。

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[cmake 参考手册](https://cmake.org/cmake/help/latest/index.html)》。</font>版本为V4.2.0，手册更新时间为2025-06。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------