---
title: vim插件——YouCompleteMe 
tags: vim插件=
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《YouCompleteMe 官方文档》](https://github.com/Valloric/YouCompleteMe "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>




# 1 简介

* **插件介绍**：YouCompleteMe是一款针对Vim的快速，即用型的模糊搜索代码补全引擎。包括以下几种引擎：
  * 基于标识符的引擎，可与每种编程语言一起使用
  * 基于[Clang](http://clang.llvm.org/)的引擎，为C / C ++ / Objective-C / Objective-C ++（C系列）提供本地语义代码补全
  * 基于[Jedi](https://github.com/davidhalter/jedi)的Python 2和3的补全引擎
  * 基于[OmniSharp的](https://github.com/OmniSharp/omnisharp-server)C＃补全引擎
  * Go的[Gocode](https://github.com/nsf/gocode)和[Godef](https://github.com/Manishearth/godef)语义引擎的组合，
  * 基于[TSServer](https://github.com/Microsoft/TypeScript/tree/master/src/server)的TypeScript完成引擎，
  * 基于[Tern](http://ternjs.net/)的JavaScript补全引擎，
  * 基于[racer](https://github.com/phildawes/racer)的Rust补全引擎，
  * 基于[jdt.ls](https://github.com/eclipse/eclipse.jdt.ls)的Java实验性补全引擎。
  * 基于omnifunc的补全引擎，它使用来自Vim的omnicomplete系统的数据为许多其他语言（Ruby，PHP等）提供语义补全。
  * 基于UltiSnips的代码片补全引擎
* **仓库地址**：<https://github.com/Valloric/YouCompleteMe>

# 2 安装(Ubuntu 16.04)
* 首先确保Vim版本至少为7.4.1578，并且支持Python 2或Python 3。
  * `vim --version | grep python`，显示`+python`则支持，否则安装支持python2的vim-nox-py2，或者使用源码重新编译vim
* 使用Vundle安装YCM，以支持python2或python3
  * `git clone https://github.com/Valloric/YouCompleteMe.git ~/.vim/bundle/YouCompleteMe`
  * `cd ~/.vim/bundle/YouCompleteMe`
  * `git submodule update --init --recursive`
  * `vim ~/.vimrc`
  * 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'Valloric/YouCompleteMe'`
  * `:wq`
  * `vim`
  * `:PluginInsttall`
* 安装编译环境
  * `sudo apt install build-essential cmake`
  * `sudo apt install python-dev python3-dev`
  * `sudo apt install ctags`   
  * `sudo apt install gcc g++`
  * `sudo apt install clang libclang-dev`
* 编译YCM 
  * `cd ~/.vim/bundle/YouCompleteMe`
  * `./install.py`
  * 语言支持：
     * C/C++/Objective-C/Objective-C++：编译时添加`--clang-completer`
     * C#：先安装[Mono](http://www.mono-project.com/docs/getting-started/install/linux/#debian-ubuntu-and-derivatives)，编译时添加`--cs-completer`
     * Go：先安装[Go](https://golang.org/doc/install)，编译时添加`--go-completer`
     * TypeScript：先安装[ Node.js and npm](https://docs.npmjs.com/getting-started/installing-node)，然后使用`npm install -g typescript`安装TypeScript SDK
     * JavaScript: 先安装[ Node.js and npm](https://docs.npmjs.com/getting-started/installing-node)，编译时添加`--js-completer`
     * Rust: 先安装[Rust](https://www.rust-lang.org/)，编译时添加`--rust-completer`
     * Java: 先安装[JDK8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，编译时添加` --java-completer`
     * 全部支持：先安装所有的依赖环境，编译时添加`--all`
     
>注意：`如果提示fatal: unable to access 'https://go.googlesource.com/tools/': Failed to connect to go.googlesource.com 那就是GFW做的，自己想办法吧`
# 3 C族语义补全配置
* `vim ~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/.ycm_extra_conf.py`
* 在flags[]列表中添加头文件路径，编译选项:
```python
'-isystem',
'C/C++ 头文件绝对路径',
'-I',
'C/C++ 头文件绝对路径',
'-std',
'C/C++ 标准',
'-x',
'目标语言',
```
* 如果不想因为c++11新特性发出警告就添加'-Wc\+\+11-compat'，并删除' \-Wc\+\+98-compat'（如果有的话）

* 也可以考虑使用[YCM-Generator](https://github.com/rdnetto/YCM-Generator)生成ycm_extra_conf.py文件。
* 我的设置如下：
```python
flags = [
'-Wall',
'-Wextra',
'-Werror',
'-Wno-long-long',
'-Wno-variadic-macros',
'-fexceptions',
'-DNDEBUG',
#'-Wc++11-compat',
#'-Wc++98-compat',
# THIS IS IMPORTANT! Without the '-x' flag, Clang won't know which language to
# use when compiling headers. So it will guess. Badly. So C++ headers will be
# compiled as C headers. You don't want that so ALWAYS specify the '-x' flag.
# For a C project, you would set this to 'c' instead of 'c++'.
# use clang -v to show header path
'-x',
'c++',
'-isystem',
'/usr/include/c++/5',
'-isystem',
'/usr/include/c++/5/backward',
'-isystem',
'/usr/include/x86_64-linux-gnu/c++/5',
'-isystem',
'/usr/include/x86_64-linux-gnu',
'-isystem',
'/usr/include',
'-isystem',
'/usr/local/include',
'-isystem',
'/usr/include/clang/6.0/include',
]

```

# 4 命令
* `:YcmRestartServer`
  如果ycmd补全服务器由于某种原因突然停止，则可以使用此命令重新启动它
  
* `:YcmForceCompileAndDiagnostics`
 调用此命令将强制YCM立即重新编译文件并显示遇到的任何新诊断信息
* `:YcmDiags`
 如果在文件中检测到任何错误或警告，则调用此命令将用齐填充Vim的位置列表，然后将其打开。如果给定的错误或警告可以通过调用：YcmCompleter FixIt来修复，那么（FixIt available）会。有关更多附加到错误或警告文本中信息，请参阅FixIt完成程序子命令。`g:ycm_open_loclist_on_ycm_diags`可以禁止打开位置列表，但仍然会填充它。
 
* `:YcmShowDetailedDiagnostic`
 当用户的光标在出现诊断信息的代码行时，该命令显示完整的诊断文本
 
* `:YcmDebugInfo`
 打印出当前文件的各种调试信息。如果您使用语义补全引擎，那么可以查看该文件将使用哪些编译命令。
 
* `:YcmToggleLogs`
 会显示YCM、ycmd服务器和当前文件类型的语义引擎服务器创建的日志文件列表，如果有的话。
 
* `:YcmCompleter`
 通过该命令，可以访问YCM中类似于IDE的其他功能，例如语义GoTo，类型信息，FixIt和 refactoring等。如果无其他参数则列出当前可以使用的子命令列表

## 4.1 YcmCompleter子命令
&emsp;&emsp;被调用的子命令会自动发送到当前激活的语义补全引擎，因此：如果当前文件是Python文件，`YcmCompleter GoToDefinition`将调用Python语义补全器上的`GoToDefinition`子命令，如果当前文件是C /C ++/Objective-C之一，则发送到clang补全引擎。可以将子命令映射到。例如，`nnoremap <leader> jd：YcmCompleter GoTo <CR>`将`<leader> jd`映射到`YcmCompleter GoTo` 。

### 4.1.1 GoTo系列
&emsp;&emsp;GoTo系列命令用于浏览代码时，进行跳转，当移动光标时，子命令会进入到Vim的跳转列表中，可以使用CTRL-O跳回到原来的位置，CTRL-I向前跳转。如果有多个目标，quickfix列表将填充所有可用位置，并在屏幕底部打开。您可以使用`YcmQuickFixOpened`自动命令来更改此行为。

* `:YcmCompleter GoToInclude`
  查找当前行的头文件并跳转到它。文件类型支持：c，cpp，objc，objcpp
  
* `:YcmCompleter GoToDeclaration`
 查找光标下的标识符并跳转到其声明，文件类型支持：c，cpp，objc，objcpp，cs，go，java，python，rust，typescript
 
* `:YcmCompleter GoToDefinition`
 查找光标下的标识符并跳转到其定义。注意：对于C族语言，这仅适用于符号的定义位于当前翻译单元中时。翻译单元由您正在编辑的文件以及引入的头文件组成。文件类型支持：c，cpp，objc，objcpp，cs，go，java，javascript，python，rust，typescript
 
* `:YcmCompleter GoTo `
 该命令尝试执行它可以执行的“最明智的”GoTo操作。目前，这意味着它尝试查找光标下的符号并在可能的情况下跳转到其定义;如果定义不能从当前翻译单元访问，则跳转到符号的声明。对于C/C++/Objective-C，它首先尝试查找当前行的头文件并跳转到它。对于C＃，先尝试查找当前标识符的实现。文件类型中支持：c，cpp，objc，objcpp，cs，go，java，javascript，python，rust，typescript
 
* `:YcmCompleter GoToImprecise`
 与GoTo命令相同，只是在AST中查找节点前，不会使用libclang重新编译文件。文件类型支持：c，cpp，objc，objcpp。
 
* `:YcmCompleter GoToReferences`
 这个命令试图找到项目中所有文件中引用光标下标识符的位置，并在quickfix列表列出这些位置。文件类型支持：java，javascript，python，typescript
 
* `:YcmCompleter GoToImplementation`
 查找光标下的标识符并跳转到其实现（而不是接口）。如果有多个实现，则提供可供选择的实现列表。文件类型支持: cs。
 
* `:YcmCompleter GoToImplementationElseDeclaration`
 查找光标下的标识符并跳转到其实现，否则跳转到它的声明。如果有多个实现，则提供可供选择的实现列表。文件类型支持：cs
 
* `:YcmCompleter GoToType`
 查找光标下的标识符并跳转到其类型的定义，例如如果标识符是一个对象，则转到其类的定义。文件类型支持：typescript

### 4.1.2 语义信息命令
&emsp;&emsp;这些命令对于查找有关代码的静态信息很有用，例如变量类型，查看声明和文档字符串

* `:YcmCompleter GetType`
  显示光标下的变量或方法的类型，以及父类型。文件类型支持：c，cpp，objc，objcpp，java，javascript，typescript
  
* `:YcmCompleter GetTypeImprecise`
 与GetType命令相同，只是它在AST中查找节点前不会使用libclang重新编译文件。文件类型支持：c，cpp，objc，objcpp
 
* `:YcmCompleter GetParent`
  显示光标下标识符的语义父项。语义父项指语义上包含给定标识符的项目。，在类中声明，但在类外实现的函数，语义父项是类。对于全局声明，语义父项是翻译单元。文件类型支持：c，cpp，objc，objcpp

* `:YcmCompleter GetDoc`
  显示预览窗口，其中包含有关光标下标识符的快速信息。根据文件类型的不同，包括如下内容：标识符的类型或声明，Doxygen /javadoc注释，Python文档，等等。文件类型支持：c，cpp，objc，objcpp，cs，java，javascript，python，typescript，rust
  
* `:YcmCompleter GetDocImprecise`
  与GetDoc命令相同，只是它在查找AST节点之前不用libclang重新编译文件。文件类型支持：c，cpp，objc，objcpp

### 4.1.3 重构命令
&emsp;&emsp;这些命令会更改源代码以执行重构或代码更正。YouCompleteMe不会执行任何无法撤消的操作，也不会将文件保存或写入磁盘。

* `:YcmCompleter FixIt`
 在可用的情况下，尝试更改缓冲区以纠正当前行上的诊断错误或警告。如果有多个建议可用，则会显示选项并可以选择一个。提供诊断的补全引擎可以对源进行微小的修改以纠正诊断。例如语法错误，如缺少后缀分号等。如果没有 FixIt可用，或者当前行上没有诊断信息，则此命令对当前缓冲区没有影响。如果进行了任何修改，则对缓冲区所做的更改数量将被显示，用户可以使用编辑器的撤消命令进行恢复。当诊断可用时，并且`g：ycm_echo_current_diagnostic`设置为1，则当补全引擎能够添加此指示时，文本（FixIt）会附加到显示的诊断信息后。当补全引擎能够添加此指示时，文本（FixIt可用）也附加到`:YcmDiags`命令输出的诊断文本中。文件类型支持：c，cpp，objc，objcpp，cs，java，typescript

* `:YcmCompleter RefactorRename <new name> `
 在支持的文件类型中，此命令尝试执行光标下的标识符的语义重命名。这包括重命名声明，标识符的定义和用法，或任何其他语言适当的操作。具体行为由使用中的语义引擎定义。与FixIt类似，此命令对源文件自动应用修改。重命名操作可能涉及对多个文件的更改，这些文件可能不会在Vim缓冲区中打开。YouCompleteMe为你处理所有的文件。文件类型支持：java，javascript（仅限变量），typescript
* 多文件重构
  * 当重构或FixIt命令触及多个文件时，YouCompleteMe会尝试将这些修改应用到当前选项卡中任何现有的、打开的可见缓冲区。如果找不到这样的缓冲区，则YouCompleteMe将在当前窗口顶部的一个新的水平窗口中打开文件，应用更改，然后隐藏该窗口。注意：之后的缓冲区仍保持打开状态，必须手动保存。这样做之前会打开确认对话框提醒您即将发生的动作。
  * 修改完成后，quickfix列表将列出所有修改的位置。用于查看使用`:copen`进行的所有自动更改。通常，使用`CTRL-W <enter>`在新列视图中打开选定的文件。
  * 缓冲区不会自动保存。也就是说，您必须在查看quickfix列表中的更改后手动保存修改后的缓冲区。使用Vim强大的撤消功能可以撤消更改。请注意，Vim的undo是针对每个缓冲区，因此要撤消所有更改，必须分别在每个修改的缓冲区中应用撤消命令。
  * 在应用修改时，Vim可能会找到已经打开并且具有交换文件的文件。如果您在任何此类提示中选择“中止”或“退出”，该命令将中止。这会使重构操作部分完成，必须使用Vim的撤销功能手动更正。在这种情况下，quickfix列表未被填充。用`:buffers`查看由命令打开的缓冲区。
  
* `:YcmCompleter Format`
  该命令根据Vim选项`shiftwidth`和`expandtab`的值来格式化整个缓冲区或其一部分。要格式化文档的特定部分，您可以使用Vim的可视模式进行选择或直接在命令前输入范围。文件类型支持：java，typescript
  
* `:YcmCompleter OrganizeImports`
该命令将删除未使用的imports并对当前文件中的imports进行排序。它还可以将来自TypeScript中相同模块的导入进行分组和解析Java中的导入。

### 4.1.4 杂项命令
&emsp;&emsp;这些命令用于一般管理，而不是IDE功能。它们涵盖了语义引擎服务器实例和编译标志等内容。

* `:YcmCompleter RestartServer`
 为与YCM对话的独立服务器的语义引擎重新启动以作为本地主机服务器，在编辑JavaScript项目中的文件以切换到该项目时，请参阅此子命令。可以为Python提供一个额外的可选参数，指定用于重启Python语义引擎的python二进制文件。文件类型支持：cs，go，java，javascript，python，rust, typescript

* `:YcmCompleter ClearCompilationFlagCache`
 YCM将缓存从`ycm_extra_conf.py`文件中的`FlagsForFile`函数获得的标志，除非返回它们时，将do_cache参数设置为False。它还缓存从编译数据库中提取的标志。该缓存在内存中，永远不会失效（除非使用：YcmRestartServer命令重新启动服务器）。该命令完全清除该缓存。YCM将在未来根据需要重新查询您的FlagsForFile函数或编译数据库。文件类型支持：c，cpp，objc，objcpp

* `:YcmCompleter ReloadSolution`
 指示Omnisharp服务器清除其缓存并从磁盘重新加载所有文件。文件类型支持：cs


# 5 函数
* `call youcompleteme#GetErrorCount()`
 获取YCM诊断为错误的数量。如果不存在错误，则此函数返回0

* `call youcompleteme#GetWarningCount()`
 获取YCM诊断为警告的数量。如果不存在警告，则此函数返回0

# 6 自动命令
* `YcmLocationOpened`
当YCM打开位置列表窗口以响应YcmDiags命令时，会触发此自动命令。默认情况下，位置列表窗口当前窗口的底部打开，其高度自适应。`YcmLocationOpened`可以用于修改设置。例如：

```
function! s:CustomizeYcmLocationWindow()
  " Move the window to the top of the screen.
  wincmd K
  " Set the window height to 5.
  5wincmd _
  " Switch back to working window.
  wincmd p
endfunction

autocmd User YcmLocationOpened call s:CustomizeYcmLocationWindow()
```
*  `YcmQuickFixOpened`
当YCM打开quickfix窗口以响应GoTo\*和RefactorRename子命令时，会触发此自动命令。默认情况下，quickfix窗口在屏幕底部打开并且其高度设置为自适应。`YcmQuickFixOpened`自动命令可以用来修改该设置。例如

```
function! s:CustomizeYcmQuickFixWindow()
  " Move the window to the top of the screen.
  wincmd K
  " Set the window height to 5.
  5wincmd _
endfunction

autocmd User YcmQuickFixOpened call s:CustomizeYcmQuickFixWindow()
```
# 7 选项
* `let g:ycm_min_num_of_chars_for_completion = 2`
 设置触发标识符补全的最小字符数，设置为99或更大的数字，则等效于关闭标识符补全功能，但保留语义补全功能
 
* `let g:ycm_min_num_identifier_candidate_chars = 0`
 设置要在标示符补全列表中显示的候选项的最小字符数，0表示没有限制，对语义补全无影响
 
* `let g:ycm_max_num_candidates = 50`
 设置语义补全的最大候选项数量，0表示没有限制
 
* `let g:ycm_max_num_identifier_candidates = 10`
 设置标识符补全的最大候选项数量，0表示没有限制
 
* `let g:ycm_auto_trigger=1`
 0表示关闭ycm语义补全和标识符补全触发器，但仍可以用ctrl+space 打开语义补全，1表示打开
 
* `let g:ycm_filetype_whitelist = { '*': 1 }`
 文件类型白名单，vim打开这些类型文件时会开启YCM。`*`表示所有文件类型
 
* `let g:ycm_filetype_blacklist = {'tagbar' : 1, 'qf' : 1,'notes' : 1, 'markdown' : 1, 'unite' : 1, 'text' : 1, 'vimwiki' : 1, 'pandoc' : 1, 'infolog' : 1, 'mail' : 1}`
 文件类型黑名单，vim打开这些类型文件时会关闭YCM
 
* `let g:ycm_filetype_specific_completion_to_disable={'gitcommit': 1}`
 语义补全黑名单，vim打开这些类型文件时会关闭YCM语义补全，但标识符补全仍可用
 
* `let g:ycm_filepath_blacklist = {'html' : 1, 'jsx' : 1,'xml' : 1,}`
 对特定文件类型禁用文件路径补全
 
* `let g:ycm_show_diagnostics_ui = 1`
 开启YCM的显示诊断信息的功能，0表示关闭
 
* `let g:ycm_error_symbol = '>>'`
 "设置错误标志为>>
 
* `let g:ycm_warning_symbol = '>>'`
 "设置警告标志为>>
 
* `let g:ycm_enable_diagnostic_signs = 1`
 在代码中高亮显示YCM诊断对应的内容，如果关闭，则会关闭错误和警告高亮功能，0表示关闭
 
* `let g:ycm_enable_diagnostic_highlighting = 1`
 高亮显示代码中与诊断信息有关的文本或代码，0表示关闭
 
* `let g:ycm_echo_current_diagnostic = 1`
 当光标移到所在行时显示诊断信息
 
* `let g:ycm_filter_diagnostics={}`
 诊断信息过滤器，此选项控制YCM将呈现哪些诊断，针对特定文件类型，用正则表达式控制要显示的内容，用level控制消息的级别，`{}`表示显示所有诊断信息
 
* `let g:ycm_always_populate_location_list = 0`
 每次获取新诊断数据时自动填充位置列表，1表示打开，默认关闭以免干扰可能已放置在位置列表中的其他数据。在vim中用`:help location-list`命令查看位置列表的具体解释
 
* `let g:ycm_open_loclist_on_ycm_diags = 1`
 在强制编译后自动打位置列表并用诊断信息填充，所谓位置列表就是标出各错误或警告对应在哪些行的小窗口，可以实现直接跳转到错误行
 
* `let g:ycm_complete_in_comments=0`
 补全功能在注释中同样有效，1表示打开
 
* `let g:ycm_complete_in_strings = 1`
 打开字符串自动补全功能。0代表关闭。这用于c系语言中#include后列出头文件很有用，如果设置为0则关闭文件名补全功能
 
* `let g:ycm_collect_identifiers_from_comments_and_strings = 0`
 让YCM可以收集注释中的文字来分析以用于补全，默认为0，只能收集代码中的文字来分析
 
* `let g:ycm_collect_identifiers_from_tags_files = 0`
 开启tags补全引擎 ，在vim中用:h 'tags'命令来查看相关信息，只支持ctags，切必须使用`--fields=+l`选项
 
* `let g:ycm_seed_identifiers_with_syntax = 0`
 使用vim的语法标识符来建立标识符数据库  
 
* `let g:ycm_extra_conf_vim_data = []`
 将数据从Vim发送到`.ycm_extra_conf.py`文件中的FlagsForFile函数。
 
* `let g:ycm_server_python_interpreter = ''`
 为ycm服务器指定特定的python解释器，默认为空表示在系统上搜索适当的Python解释器
 
* `let g:ycm_keep_logfiles = 0`
 YCM关闭时保存日志，0表示关闭
 
* `let g:ycm_log_level = 'info'`
 设置YCM的日志记录级别，可以是debug，info，warning，error或critical。debug是最详细的
 
* `let g:ycm_auto_start_csharp_server = 1`
 设置当vim打开c#文件时，OmniSharp server自动开启，0代表不自动开启
 
* `let g:ycm_auto_stop_csharp_server = 1`
 设置当vim关闭c#文件时，OmniSharp server自动关闭，0代表不自动关闭
 
* `let g:ycm_csharp_server_port = 0`
 指定OmniSharp server的监视端口，0表示使用os自动提供的未使用的端口
 
* `let g:ycm_csharp_insert_namespace_expr = ''`
 默认c#中插入命名空间时自动在最近的using语句下插入using语句，如要插入到其他地方则设置该选项
 
* `let g:ycm_add_preview_to_completeopt = 0`
 为当前补全选项在vim顶部增加预览窗口，用来显示函数原型等信息，如果vim的completeopt已经设置为prieview则不会有影响，`:h completeopt`查看相关信息，用`:set completeopt?`查看当前vim的设置，默认为0
 
* `let g:ycm_autoclose_preview_window_after_completion = 0`
 选中补全选项后自动关闭预览窗口，当`g:ycm_add_preview_to_completeopt`设为1时或者vim的`completeopt`设为preview有效
 
* `let g:ycm_autoclose_preview_window_after_insertion = 0`
 离开插入模式后自动关闭预览窗口，当`g:ycm_add_preview_to_completeopt`设为1时或者vim的`completeopt`设为preview有效
 
* `let g:ycm_max_diagnostics_to_display = 30`
此选项控制在文件中检测到错误或警告时向用户显示的最大诊断数

* `let g:ycm_key_list_select_completion = ['<TAB>', '<Down>']`
 设置用于选择补全列表中的第一个选项以及进入补全列表后向下选择的快捷键，默认为tab键和方向下键
 
* `let g:ycm_key_list_previous_completion = ['<S-TAB>', '<Up>']`
 设置用于向上选择补全列表中的选项的快捷键，默认为shift+tab，和方向上键
 
* `let g:ycm_key_list_stop_completion = ['<C-y>']`
 设置用于关闭补全列表的快捷键，默认为ctrl+y
 
* `let g:ycm_key_invoke_completion = '<C-Space>'`
 设置强制启用语义补全的快捷键，有些系统函数如fopen, strcpy如果不智能提示，可以按<Ctrl>+<Space>键 
 
* `let g:ycm_key_detailed_diagnostics = '<leader>d'`
 设置查看光标停留处的错误诊断详细信息的快捷键,默认为\d
 
* `let g:ycm_global_ycm_extra_conf = ''`
 设置`.ycm_extra_conf.py`的全局路径，避免每次都需要复制到当前目录.若为空则每次都需复制`.ycm_extra_conf.py`文件到当前目录
 
* `let g:ycm_confirm_extra_conf = 1`
  允许自动加载`.ycm_extra_conf.py`，不再提示 ，设置为1，则每次都提示用于确认该文件是否安全 
  
* `let g:ycm_extra_conf_globlist = []`
  设置加载 `.ycm_extra_conf.py`的路径，`*`表示匹配任何字符，`?`匹配任何单个字符，[seq] 匹配seq中的任何单个字符，[!seq] 匹配不在seq中的任何单个字符，路径前加`！`表示不加载所有改路径上匹配的文件
  
* `let g:ycm_filepath_completion_use_working_dir = 0`
  设置YCM的文件名补全时，相对路径是按照vim的当前工作目录还是活动缓冲区中的文件所在目录来解释。0是按照文件所在目录。
  
* `let g:ycm_semantic_triggers =  {'c' : ['->', '.'],'objc' : ['->', '.', 're!\[[_a-zA-Z]+\w*\s', 're!^\s*[^\W\d]\w*\s','re!\[.*\]\s'],'ocaml' : ['.', '#'],'cpp,objcpp' : ['->', '.', '::'],'perl' : ['->'],'php' : ['->', '::'],'cs,java,javascript,typescript,d,python,perl6,scala,vb,elixir,go' : ['.'],'ruby' : ['.', '::'],'lua' : ['.', ':'],'erlang' : [':'],}`
  设置YCM的语义触发器的关键字
  
* `let g:ycm_cache_omnifunc = 1`
  某些omni补全引擎引起与YCM缓存不适配，可能不会为给定的前缀产生所有可能的结果，如果关闭该选项则每次都重新查询omni补全引擎生成匹配项 ，默认为1代表开启
  
* `let g:ycm_use_ultisnips_completer = 1`
 启用ultisnips补全，1代表允许
 
* `let g:ycm_goto_buffer_command = 'same-buffer'`
 设置使用goto跳转快捷键时，新窗口的打开方式可以设置为'same-buffer', 'horizontal-split', 'vertical-split', 'new-tab'或 'new-or-existing-tab'
 
* `let g:ycm_disable_for_files_larger_than_kb = 1000`
  设置YCM的作用的文件大小上限，单位为Kb，0表示无上限
  
* `let g:ycm_python_binary_path = 'python'`
  指定用来运行jedi补全库的Python解释器。默认情况下与ycm服务器使用相同的解释器
  




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《YouCompleteMe 官方文档》](https://github.com/Valloric/YouCompleteMe "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
