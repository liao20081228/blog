---
title: CMake 
tags: cmake
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译至<font color=blue>[《Cmake Reference Documentation v3.16》](https://cmake.org/cmake/help/v3.16/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
# 1 命令行工具

## 1.1 cmake
### 1.1.1 概要
```shell
#生成一个构建系统
 cmake [<options>] <path-to-source>
 cmake [<options>] <path-to-existing-build>
 cmake [<options>] -S <path-to-source> -B <path-to-build>

#构建项目
 cmake --build <dir> [<options>] [-- <build-tool-options>]

#安装项目
 cmake --install <dir> [<options>]
 
#打开项目
 cmake --open <dir>

#允许一个脚本
 cmake [{-D <var>=<value>}...] -P <cmake-script-file>

#运行命令行工具
 cmake -E <command> [<options>]

#运行Find-Package工具
 cmake --find-package [<options>]

#查看帮助
 cmake --help[-<topic>]
```
### 1.1.2 描述

**cmake**可执行文件是跨平台构建系统生成器CMake的命令行界面。 上面的综述列出了该工具可以执行的各种操作，如下节所述。

要使用CMake构建软件项目，请生成项目构建系统。 有选择地使用cmake构建项目，安装项目或直接运行相应的构建工具（例如make）。 cmake也可以用于查看帮助。

其它操作用于软件开发人员使用CMake语言编写脚本来支持他们的构建工作。

有关可以代替cmake的图形用户界面，请参见ccmake和cmake-gui。 有关CMake测试和打包工具的命令行界面，请参阅ctest和cpack。

有关CMake的更多信息，请参阅本手册末尾的链接。

### 1.2.3 CMake构建系统介绍

构建系统描述了使用构建工具从源代码中如何构建可执行文件和库，以及自动执行该过程。 例如，构建系统可以是与命令行生成工具make一起使用的Makefile，也可以是集成开发环境（IDE）的项目文件。 为了避免维护多个此类构建系统，项目可以使用以CMake语言编写的文件抽象地指定其构建系统。 从这些文件中，CMake通过一个称为生成器的后端为每个用户在本地生成一个首选的构建系统。

要使用CMake生成构建系统，必须选择以下内容：

* 源码树
包含项目提供的源文件的顶级目录。 该项目使用cmake-language（7）手册中描述的文件指定其构建系统，该文件从名为CMakeLists.txt的顶级文件开始。 这些文件指定了构建目标及其依赖性，如cmake-buildsystem（7）手册中所述。

* 构建树
用于存储构建系统文件和构建输出文件(例如可执行文件和库)的顶级目录。CMake将编写一个CMakeCache.txt文件将该目录标识为构建树，并存储诸如构建系统配置选项之类的持久信息。
为了维护干净的源码树，请使用单独的专用构建树执行源码外构建。也支持源码内构建，其中构建树与源码树位于相同的目录中，但不建议这样做。

* 生成器
这选择了要生成的构建系统的类型。 有关所有生成器的文档，请参见cmake-generators（7）手册。 运行`cmake --help`以查看本地可用的生成器列表。 可选择使用下面的-G选项来指定生成器，或者简单地接受当前平台的默认CMake选择。
当使用命令行构建工具生成器之一时，CMake期望编译工具链所需的环境已在shell中配置好。 而使用IDE构建工具生成器时，不需要特定的环境。

### 1.2.4 生成项目的构建系统
按以下命令样本之一运行CMake，以指定源代码和构建树并生成构建系统。

**`cmake [<options>] <path-to-source>`**
使用当前工作目录作为构建树，<path-to-source\> 作为源码树。指定的路径可以是绝对路径，也可以是相对于当前工作目录的路径。 源码树必须包含CMakeLists.txt文件，并且不能包含CMakeCache.txt文件，因为后者标识了已存在的构建树。 例如：
```shell
$ mkdir build ; cd build
$ cmake ../src
```
**`cmake [<options>] <path-to-existing-build>`**
使用\<path-to-existing-build>作为构建树，并从其CMakeCache.txt文件加载到源码树的路径，该文件必须已由先前的CMake运行生成。 指定的路径可以是绝对路径，也可以是相对于当前工作目录的路径。 例如：
```shell
$ cd build
$ cmake .
```

**`cmake [<options>] -S <path-to-source> -B <path-to-build>`**
使用\<path-to-build>作为构建树，并使用\<path-to-source>作为源码树。 指定的路径可以是绝对路径，也可以是相对于当前工作目录的路径。 源码树必须包含CMakeLists.txt文件。 如果构建树尚不存在，则会自动创建。 例如：
```shell
$ cmake -S src -B build
```
在所有情况下，\<options>可能是下面的零个或多个选项。

生成构建系统后，可以使用相应的本机构建工具来构建项目。 例如，使用Unix Makefiles生成器后，可以直接运行make：
```shell
$ make
$ make install
```
或者，可以使用cmake通过自动选择并调用适当的本机构建工具来构建项目。


### 1.2.5 选项
* `-S <path-to-source>`
要构建的CMake项目的根目录的路径。

* `-B <path-to-build>`
构建目录的路径，CMake将使用该目录作为构建目录的根目录。如果该目录不存在，CMake将创建它。

* `-C <initial-cache>`
预加载脚本以填充缓存。
当CMake首次在空的构建树中运行时，它将创建CMakeCache.txt文件，并使用项目的自定义设置填充该文件。 此选项可用于指定在第一次通过项目的CMake列表文件之前从中加载缓存项的文件。 加载的条目优先于项目的默认值。 给定的文件应该是含有使用了`CACHE`选项的`set（）`命令的CMake脚本，而不是缓存格式的文件。
脚本中对CMAKE_SOURCE_DIR和CMAKE_BINARY_DIR的引用评估为顶级源码树和构建树。

* `-D <var>:<type>=<value>, -D <var>=<value>`
创建或更新CMake `CACHE`条目。
当CMake首次在空的构建树中运行时，它将创建CMakeCache.txt文件，并使用项目的可自定义设置填充该文件。 此选项可用于指定优先于项目默认值的设置。 可以根据需要对尽可能多的CACHE条目重复该选项。
如果指定了:\<type>部分，则该部分必须是set()命令文档为其`CACHE`签名（**所谓的签名就是模板的意思**）指定的类型之一。 如果省略:\<type>部分，则该条目将以无类型被创建（如果该类型不与已存在的类型有关）。 如果项目中的命令将类型设置为`PATH`或`FILEPATH`，则\<value>将转换为绝对路径。
此选项也可以作为单个参数给出：`-D<var>：<type> = <value>或-D<var> = <value>`。

* `-U <globbing_expr>`
从CMake缓存中删除匹配的条目。
此选项可用于从CMakeCache.txt文件中删除一个或多个变量，支持通配符\*和\？。 可以根据需要对尽可能多的CACHE条目重复该选项。
谨慎使用，可能使CMakeCache.txt不工作。

* `-G <generator-name>`
指定一个构建系统生成器。
CMake可能在某些平台上支持多个本机构建系统。 生成器负责生成特定的构建系统。 可能的生成器名称在cmake-generators（7）手册中指定。
如果未指定，则CMake检查CMAKE_GENERATOR环境变量，否则回退到内置默认选择。

* `-T <toolset-spec>`
生成器的工具集规范(如果支持)。
一些CMake生成器支持工具集规范，以告诉本机生成系统如何选择编译器。 有关详细信息，请参见CMAKE_GENERATOR_TOOLSET变量。

* `-A <platform-name>`
如果生成器支持，请指定平台名称。
一些CMake生成器支持将平台名称提供给本机构建系统以选择编译器或SDK。 有关详细信息，请参见CMAKE_GENERATOR_PLATFORM变量。

* `-Wno-dev`
禁止开发者警告。
禁止发给CMakeLists.txt文件作者的警告。 默认情况下，这将关闭已弃用警告。

* `-Wdev`
允许开发者警告。
允许发给CMakeLists.txt文件作者的警告。 默认情况下，这将打开已弃用警告。

* `-Werror=dev`
将开发者警告视为错误。
将发给CMakeLists.txt文件作者的警告视为错误。默认情况下，这将打开把已弃用警告视为错误。

* `-Wno-error=dev`
将开发者警告不视为错误。
将发给CMakeLists.txt文件作者的警告不视为错误。默认情况下，这将关闭把已弃用警告视为错误。

* `-Wdeprecated`
启用已弃用的功能警告。
启动使用已废弃功能的警告，这些警告是针对CMakeLists.txt文件的作者的。

* `-Wno-deprecated`
禁止已弃用的功能警告。
禁止使用已废弃功能的警告，这些警告是针对CMakeLists.txt文件的作者的。

* `-Werror=deprecated`
将已弃用功能和宏的警告视为错误
将使用已弃用功能和宏的警告视为错误，这些警告是针对CMakeLists.txt文件的作者的。

* `-Wno-error=deprecated`
将已弃用功能和宏的警告不视为错误
将使用已弃用功能和宏的警告不视为错误，这些警告是针对CMakeLists.txt文件的作者的。

* `-L[A][H]`
列出非高级缓存变量。
列出CACHE变量将运行CMake，并列出CMake CACHE中所有未标记为INTERNAL或ADVANCED的变量。 这将有效显示当前的CMake设置，其可以使用-D选项进行更改。 更改某些变量可能会导致创建更多变量。 如果指定了A，则还将显示高级变量。 如果指定了H，它还将显示每个变量的帮助信息。

* `-N`
只是观察模式。
只加载缓存，而不实际运行配置和生成步骤。

* `--graphviz=[file]`
生成依赖关系的graphviz，有关更多信息，请参见CMakeGraphVizOptions。
生成一个graphviz输入文件，其中将包含项目中的所有库和可执行文件依赖项。 有关更多详细信息，请参见CMake GraphViz Options的文档。

* `--system-information [file]`
转储关于此系统的信息。
转储有关当前系统的各种信息。 如果从CMake项目的二进制树的顶部运行，它将转储其他信息，例如缓存，日志文件等。

* `--log-level=<ERROR|WARNING|NOTICE|STATUS|VERBOSE|DEBUG|TRACE>`
设置日志级别。
message（）命令将仅输出指定日志级别或更高级别的消息。 默认日志级别为STATUS。
出于向后兼容的原因，`--loglevel`也可以作为这个选项的同义词。

* `--debug-trycompile`
不要删除try_compile()构建树。 一次仅对一个try_compile()有用。
不要删除为try_compile()调用创建的文件和目录。 这对于调试失败的try_compiles非常有用。 但是，它可能会更改try-compiles的结果，因为来自前一个try-compile的旧垃圾可能会导致其他测试不正确的通过或失败。 此选项最好一次只用于一个try-compile调，且仅在调试时使用。

* `--debug-output`
将cmake置于调试模式。
在cmake运行期间打印额外的信息，例如message（SEND_ERROR）调用的堆栈跟踪。

* `--trace`
将cmake置于跟踪模式。
打印所有调用的轨迹，以及从哪里发出的。

* `--trace-expand`
将cmake置于跟踪模式。
类似于--trace，但是具备变量展开。

* `--trace-source=<file>`
将cmake置于跟踪模式，但只输出指定文件的行。
允许多个选项。

* `--trace-redirect=<file>`
将cmake置于跟踪模式，并将跟踪输出重定向到文件而不是stderr。

* `--warn-uninitialized`
警告未初始化的值。
使用未初始化的变量时，打印警告。

* `--warn-unused-vars`
警告未使用的变量。
发现已声明或设置但未使用的变量时打印警告。

* `--no-warn-unused-cli`
不要警告命令行选项中的未使用变量。
找到在命令行上声明但未使用的变量时不要警告。

* `--check-system-vars`
发现系统文件中变量使用的问题。
通常，仅在CMAKE_SOURCE_DIR和CMAKE_BINARY_DIR中搜索未使用和未初始化的变量。 该标志告诉CMake其他文件也要发出警告。

### 1.2.6 构建系统
Make提供一个命令行签名来构建一个已经生成的项目二进制树
```
cmake --build <dir> [<options>] [-- <build-tool-options>]
```
它抽象了一个本地构建工具的命令行界面，并提供了以下选项:

* `--build <dir>`
要构建的项目二进制目录。 这是必需的，必须在前面。

* `--parallel [<jobs>], -j [<jobs>]`
构建时要使用的最大并发进程数。 如果省略\<jobs>，则使用本机构建工具的默认数。
如果设置了CMAKE_BUILD_PARALLEL_LEVEL环境变量，则在未指定此选项时指定默认的并行级别。
一些本地构建工具总是并行构建。 \<jobs>值为1的可用于限制单个作业。

* `--target <tgt>..., -t <tgt>...`
构建\<tgt>而不是默认目标。 可以指定多个目标，以空格分隔。

* `--config <cfg>`
对于多配置工具，选择configuration \<cfg>。

* `--clean-first`
* 首先构建目标clean，然后构建。（只是clean的话，使用 --target clean）

* `--use-stderr`
忽略。在CMake >= 3.0，该行为是默认。

* `--verbose, -v`
启用详细输出(如果支持)，包括要执行的构建命令。
如果设置了VERBOSE环境变量或CMAKE_VERBOSE_MAKEFILE缓存变量，则可以省略此选项。
* `--`
将剩余的选项传递给本机工具

不带选项运行cmake --build，打开快速帮助。

### 1.2.7  安装项目

CMake提供一个命令行签名来安装已经生成的项目二进制树:
```
cmake --install <dir> [<options>]
```
可以在构建项目以后安装该项目，而无需使用生成的构建系统或本机构建工具。 选项包括：

* `--install <dir>`
要安装的项目二进制目录。 这是必需的，必须在前面。

* `--config <cfg>`
对于多配置生成器，选择配置 \<cfg>。

* `--component <comp>`
基于组件安装。 仅安装组件\<comp>。

* `--prefix <prefix>`
覆盖安装前缀CMAKE_INSTALL_PREFIX。

* `--strip`
在安装之前将其剥离

* `-v, --verbose`
启用详细输出。
如果设置了VERBOSE环境变量，则可以省略此选项。

不带选项运行cmake --install，打开快速帮助。

### 1.2.8 打开项目
```
cmake --open <dir>
```
在关联的应用程序中打开生成的项目。 仅某些生成器支持。

### 1.2.9 运行脚本
```
cmake [{-D <var>=<value>}...] -P <cmake-script-file>
```
将给定的cmake文件作为以CMake语言编写的脚本进行处理。 不执行配置或生成步骤，并且不修改缓存。 如果使用-D定义变量，则必须在-P参数之前。

### 1.2.10 运行命令行工具

CMake通过签名提供了内置命令行工具
```
cmake -E <command> [<options>]
```
运行cmake -E或cmake -E help来获得命令总结。可用的命令是:

* `capabilities`
 报告JSON格式的cmake功能。输出是一个JSON对象，具有以下key：
	* version 带有版本信息的JSON对象。key是:
		* string cmake --version所显示的完整版本字符串
		* major 整数形式的主版本号
		* minor 整数形式的次版本号
		* patch 整数形式的补丁级别
		* suffix cmake版本后缀字符串
		* isDirty 如果cmake构建来自脏树，则设置为bool。
	* generators 可用生成器的列表。每个生成器都是一个JSON对象，具有以下key：
		* name 包含生成器名称的字符串
		* toolsetSupport 如果生成器支持工具集，则为true；否则为false
		* platformSupport 如果生成器支持平台，则为true；否则为false。
		* extraGenerators 具有与生成器兼容的所有其他生成器的字符串列表。
	* fileApi 当cmake-file-api（7）可用时存在的可选成员。 该值是具有一个成员的JSON对象：
		* requests 包含零个或多个受支持的file-api请求的JSON数组。 每个请求都是一个具有成员的JSON对象：
			* kind 指定受支持的对象类型之一。
			* version 一个JSON数组，其元素都是一个JSON对象，包含主要成员和次要成员，这些成员指定了非负整数版本组件。
	* serverMode 如果cmake支持服务器模式，则为true；否则为false。

* `chdir <dir> <cmd> [<arg>...]`
更改当前工作目录并运行命令。

* `compare_files [--ignore-eol] <file1> <file2>`
检查\<file1>是否与\<file2>相同。 如果文件相同，则返回0，否则返回1。--ignore-eol选项表示逐行比较，并且忽略LF/ CRLF差异。

* `copy <file>... <destination>`
将文件复制到\<destination>（文件或目录）。 如果指定了多个文件，则<\destination>必须是目录并且必须存在。 不支持通配符。 copy跟随符号链接。 这意味着它不会复制符号链接，但是会复制它指向的文件或目录。

* `copy_directory <dir>... <destination>`
将\<dir> ...目录的内容复制到\<destination>目录。 如果\<destination>目录不存在，将创建该目录。 copy_directory确实跟随符号链接。

* `copy_if_different <file>... <destination>`
如果文件已更改，则将它们复制到\<destination>（文件或目录）。 如果指定了多个文件，则\<destination>必须是目录并且必须存在。 copy_if_different确实跟随符号链接。

* `create_symlink <old> <new>`
创建一个符号链接\<new>，命名为\<old>。
注意：创建\<new>符号链接的路径必须预先存在。

* `echo [<string>...]`
将参数显示为文本。

* `echo_append [<string>...]`
以文本形式显示参数，但不显示新行。

* `env [--unset=NAME]... [NAME=VALUE]... COMMAND [ARG]...`
在修改后的环境中运行命令

* `environment`
显示当前环境变量。

* `false`
 什么也不做，退出码为1。
 
* `make_directory <dir>...`
创建\<dir>目录。 如有必要，还创建父目录。 如果目录已经存在，它将被静默忽略。

* `md5sum <file>...`
以md5sum兼容格式创建文件的MD5校验和:
```
351abe79cd3800b38cdfb25d45015a15  file1.txt
052f86c15bbde68af55c7f7b340ab639  file2.txt
```
* `sha1sum <file>..`
以sha1sum兼容格式创建文件的SHA1校验和:
```
4bb7932a29e6f73c97bb9272f2bdc393122f86e0  file1.txt
1df4c8f318665f9a5f2ed38f55adadb7ef9f559c  file2.txt
```
* `sha1sum <file>..`
以sha1sum兼容格式创建文件的SHA1校验和:
```
4bb7932a29e6f73c97bb9272f2bdc393122f86e0  file1.txt
1df4c8f318665f9a5f2ed38f55adadb7ef9f559c  file2.txt
```
* `sha224sum <file>..`
以sha224sum兼容格式创建文件的SHA224校验和:
```
b9b9346bc8437bbda630b0b7ddfc5ea9ca157546dbbf4c613192f930  file1.txt
6dfbe55f4d2edc5fe5c9197bca51ceaaf824e48eba0cc453088aee24  file2.txt
```
* `sha256sum <file>..`
以sha256sum兼容格式创建文件的SHA256校验和:
```
76713b23615d31680afeb0e9efe94d47d3d4229191198bb46d7485f9cb191acc  file1.txt
15b682ead6c12dedb1baf91231e1e89cfc7974b3787c1e2e01b986bffadae0ea  file2.txt
```
* `sha384sum <file>..`
以sha384sum兼容格式创建文件的SHA384校验和:
```
acc049fedc091a22f5f2ce39a43b9057fd93c910e9afd76a6411a28a8f2b8a12c73d7129e292f94fc0329c309df49434  file1.txt
668ddeb108710d271ee21c0f3acbd6a7517e2b78f9181c6a2ff3b8943af92b0195dcb7cce48aa3e17893173c0a39e23d  file2.txt
```
* `sha512sum <file>..`
以sha512sum兼容格式创建文件的SHA512校验和:
```
2a78d7a6c5328cfb1467c63beac8ff21794213901eaadafd48e7800289afbc08e5fb3e86aa31116c945ee3d7bf2a6194489ec6101051083d1108defc8e1dba89  
7a0b54896fe5e70cca6dd643ad6f672614b189bf26f8153061c4d219474b05dad08c4e729af9f4b009f1a1a280cb625454bf587c690f4617c27e3aebdf3b7a2d  
```
* `remove [-f] <file>...`
删除文件。 如果任何列出的文件尚不存在，该命令将返回非零的退出代码，但不会记录任何消息。 在这种情况下-f选项更改行为以返回零退出代码（即成功）。 remove不跟随符号链接。 这意味着它将仅删除符号链接，而不删除其指向的文件。

* `remove_directory <dir>...`
删除\<dir>目录及其内容。 如果目录不存在，它将被静默忽略。 如果\<dir>是指向目录的符号链接，则仅符号链接将被删除。
* `rename <oldname> <newname>`
重命名文件或目录（在一个卷上）。 如果具有\<newname>名称的文件已经存在，则它将被静默替换。

* `server`
启动cmake-server(7)模式。

* `sleep <number>...`
睡眠一定的时间。

* `tar [cxt][vf][zjJ] file.tar [<options>] [--] [<pathname>...]`
创建或解压tar或zip存档。选项是：
	* c 创建一个包含指定文件的新存档。如果使用，则\<pathname>…参数是强制性的。
	* x 从存档中提取到磁盘。 \<pathname> ...参数可用于仅提取选定的文件或目录。 提取选定的文件或目录时，必须提供其确切名称，例如由列表（-t）打印的路径。
	* t 列出存档内容。 \<pathname> ...自变量可用于仅列出选定的文件或目录。
	* v 生成详细输出。
	* z 使用gzip压缩。
	* j 使用bzip2压缩。
	* J 使用XZ压缩。
	* --zstd 使用Zstandard压缩
	* --files-from=\<file> 从给定文件中读取文件名，每行读取一个。 空行将被忽略。 除了--add-file=\<name>以外，行不能以-开头。
	* --format=\<format> 指定要创建的归档的格式。 支持的格式为：7zip，gnutar，pax，paxr（受限的pax，默认）和zip。
	* --mtime=\<date>  指定记录在tarball条目中的修改时间。
	* -- 停止解释选项，并将所有剩余参数视为文件名，即使它们以-开头。
* `time <command> [<args>...]`
运行命令并显示运行时间。

* `touch <file>...`
如果文件不存在，则创建\<file>。 如果\<file>存在，则更改\<file>的访问和修改时间。

* `touch_nocreate <file>...`
创建文件（如果存在）,反之不创建它。 如果文件不存在，它将被静默忽略。

* `true`
什么也不做，退出码为0。

### 1.2.11 windows特定命令行工具
下面的cmake -E命令仅在Windows上可用

* `delete_regv <key>`
删除Windows注册表值。
* `env_vs8_wince <sdkname>`
显示一个批处理文件，该文件为VS2005中安装的提供的Windows CE SDK设置了环境。

* `env_vs9_wince <sdkname>`
显示一个批处理文件，该文件为VS2008中安装的提供的Windows CE SDK设置了环境。

* `write_regv <key> <value>`
写入Windows注册表值。

### 1.2.12 运行Find-Package工具
CMake为基于Makefile的项目提供了一个类似于pkg-config的帮助器：
```
cmake --find-package [<options>]
```
它使用find_package（）搜索软件包，并将结果标志输出到stdout。 可以使用它代替pkg-config来在基于简单Makefile的项目或基于autoconf的项目（通过share/acloca/cmake.m4）中找到已安装的库。

注意由于某些技术限制，此模式不受很好的支持。 保留它是出于兼容性考虑，但不应在新项目中使用。

### 1.2.13 查看帮助

要打印CMake文档中选定的页面，请使用：
```
cmake --help[-<topic>]
```
以下其中一个选项:

* `--help,-help,-usage,-h,-H,/?` 
打印用法信息并退出。用法描述了基本命令行接口及其选项。

* `--version,-version,/V [<f>]` 
显示程序名称/版本标题并退出。
如果指定了文件，则将版本写入其中。如果给定f，则将帮助打印到指定的文件f中。

* `--help-full [<f>]`
打印所有帮助手册并退出。
所有手册均以人类可读的文本格式打印。 如果给定f，则将帮助打印到指定的文件f中。

* `--help-manual <man> [<f>]`
打印一份帮助手册并退出。
指定的手册以易于阅读的文本格式打印。如果给定f，则将帮助打印到指定的文件f中。

* `--help-manual-list [<f>]`
列出可用的帮助手册并退出。
该列表包含所有可以通过在--help-manual选项后加上手册名称来获得帮助的手册。如果给定f，则将帮助打印到指定的文件f中。

* `--help-command <cmd> [<f>]`
打印一个命令的帮助并退出。
\<cmd>的cmake-commands（7）手册将以人类可读的文本格式打印。 如果给定f，则将帮助打印到指定的文件f中。

* `--help-command-list [<f>]`
列出可用帮助的命令并退出。
该列表包含所有可通过使用--help-command选项后跟命令名称获得帮助的命令。 如果给定f，则将帮助打印到指定的文件f中。
* `--help-commands [<f>]`
打印cmake-commands手册并退出。
cmake-commands（7）手册以人类可读的文本格式打印，如果给定f，则将帮助打印到指定的文件f中。
* `--help-module <mod> [<f>]`
打印一个模块的帮助并退出。
\<mod>的cmake-modules（7）手册以人类可读的文本格式打印。如果给定f，则将帮助打印到指定的文件f中。

* `--help-module-list [<f>]`
列出可用帮助的模块并退出。
该列表包含所有可通过使用--help-module选项后跟模块名称获得帮助的模块。 如果给定f，则将帮助打印到指定的文件f中。

* `--help-modules [<f>]`
打印cmake-modules手册并退出。
cmake-modules（7）手册以人类可读的文本格式打印。 如果给定f，则将帮助打印到指定的文件f中。

* `--help-policy <cmp> [<f>]`
打印一个策略的帮助并退出。
\<cmp>的cmake-policies（7）手册以人类可读的文本格式打印。如果给定f，则将帮助打印到指定的文件f中。

* `--help-policy-list [<f>]`
列出可用帮助的策略并退出。
该列表包含所有可通过使用--help-policy选项后跟策略名称获得帮助的模块。 如果给定f，则将帮助打印到指定的文件f中。

* `--help-policies [<f>]`
打印cmake-policies手册并退出。
cmake-policies（7）手册以人类可读的文本格式打印。 如果给定f，则将帮助打印到指定的文件f中。

* `--help-property <prop> [<f>]`
打印一个的属性的帮助并退出。
\<prop>的 cmake-properties（7）手册以人类可读的文本格式打印。如果给定f，则将帮助打印到指定的文件f中。

* `--help-property-list [<f>]`
列出可用帮助的属性并退出。
该列表包含所有可通过使用--help-property选项后跟属性名称获得帮助的模块。 如果给定f，则将帮助打印到指定的文件f中。


* `--help-properties [<f>]`
打印cmake-properties手册并退出。
cmake-properties（7）手册以人类可读的文本格式打印。 如果给定f，则将帮助打印到指定的文件f中。

* `--help-variable <var> [<f>]`
打印一个的变量的帮助并退出。
\<var>的 cmake-variables（7）手册以人类可读的文本格式打印。如果给定f，则将帮助打印到指定的文件f中。

* `--help-variable-list [<f>]`
列出可用帮助的变量并退出。
该列表包含所有可通过使用--help-variable选项后跟变量名称获得帮助的模块。 如果给定f，则将帮助打印到指定的文件f中。


* `--help-variables [<f>]`
打印cmake-variables手册并退出。
cmake-variables（7）手册以人类可读的文本格式打印。 如果给定f，则将帮助打印到指定的文件f中。

### 1.2.14 另请参阅
以下资源可用于获得有关CMake的帮助：

* 主页 
<https://cmake.org>
学习CMake的主要起点。

* 在线文档和社区资源 
<https://cmake.org/documentation>可
以在此网页上找到指向可用文档和社区资源的链接。

* 话语论坛
<https://discourse.cmake.org?
话语论坛主持有关CMake的讨论和问题。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译至<font color=blue>[《Cmake Reference Documentation v3.16》](https://cmake.org/cmake/help/v3.16/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
